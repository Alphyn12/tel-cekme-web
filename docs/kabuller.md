# Kabuller ve Sınırlar

> **7 Eylül 2026 düzeltmesi:** Aşağıdaki tarihsel örnekler saha doğrulaması değildir.
> 0,50/0,60 oran, Δ 1,5–3 ve ΔT 100 °C eşikleri aracın seçimidir; evrensel kırılma
> ölçütleri değildir. %63,2 ideal, pekleşmesiz analizden alınan araç sınırıdır.
> **8 Eylül 2026:** 70 MPa tabanı artık ortalama akma integraline de uygulanır
> (bkz. K-15). Bunun sonucu olarak K değişmezliği koşulludur: taban devre dışıyken
> ve pasın tamamı taban üzerindeyken oran sadeleşir, aradaki geçiş bölgesinde
> sadeleşmez. ΔT mutlak sıcaklık değildir.
> Motor verimi bilinmeden %15–25 sabit güç farkı garanti edilemez. Kademeli azalan
> otomatik yöntem her pasta %20 sınırını artık gerçekten sağlar (otomotiv: 20 pas).
> Güncel kaynak kapsamı: [araştırma notları](arastirma-notlari.md).

Bu araç bir **ön tasarım** aracıdır, üretim reçetesi değildir. Aşağıdaki maddeler
modelin bilerek dışarıda bıraktığı veya sabitlediği şeylerdir. Araç içindeki
"Kabuller ve sınırlar" bölümü bu dosyayla aynı içeriği taşır.

## Hesaba katılmayanlar

- Kalıp esnemesi ve elastik geri yaylanma
- Yağ filmi kalınlığı ve hidrodinamik yağlama rejimi (sürtünme katsayısı sabit alınır)
- **Geri gerilimin belirsizliği.** Belirsizlik bandı `K`, `n` ve `μ` ile sınırlıdır;
  `σ_b` banda dahil değildir. Sebep: **σ_b ölçülen bir hat ayarıdır, kestirilen bir
  malzeme özelliği değil.** Bant malzeme belirsizliği için kuruldu, makine ayarı için
  değil. Geri gerilimin sonuca etkisini görmek isteyen `σ_b`'yi doğrudan değiştirip
  yeniden hesaplar; bu, bir aralık kestirmekten daha dürüsttür çünkü değer zaten
  makinede okunabilir.
- Geri gerilimin **faydası**: kalıp basıncını ve aşınmayı azaltması. Geri gerilim
  artık modelde (girdi `σ_b`), ama yalnızca **bedeli** görünür — çekme gerilmesini
  artırması. Kalıp aşınması hiç modellenmediği için kazanç tarafı yok; araç geri
  gerilimi her zaman olumsuz gösterir. Bu asimetri bilinçlidir ve arayüzde yazılıdır.
- Şekil değiştirme hızının akma gerilmesine etkisi
- Sıcaklığa bağlı akma gerilmesi değişimi (sıcaklık artışı hesaplanır ama akmaya
  geri beslenmez)
- Kalıba ve yağa giden ısı ile paslar arası soğutma (bkz. M-6)
- Kalıp aşınmasının pas boyunca geometriyi değiştirmesi
- Tel içindeki artık gerilmeler ve merkez–yüzey gerinim farkı

## Modelleme tercihleri

### M-1 · Akma gerilmesine 70 MPa taban

`σ = K·εⁿ` bağıntısı `ε → 0`'da sıfır verir; bu fiziksel değildir. Bu yüzden akma
gerilmelerine bir taban uygulanır. **Taban artık girdidir** (varsayılan tavlanmış
ETP bakır için 70 MPa); malzeme listesinden seçim yapılınca o malzemenin değeriyle
gelir — alüminyum 28, çelik 180, paslanmaz 250 MPa. Sabit kalsaydı malzeme listesi
bakırın tabanıyla çalışır ve düşük gerinimli paslarda yanlış sonuç verirdi.

Taban **iki yerde birden** geçerlidir: çıkış akma gerilmesinde (`sigmaFOut`) ve pas
boyunca ortalama akma gerilmesinde (`sigmaBar`). Gerekçe tutarlılıktır — malzeme
70 MPa'nın altında akamıyorsa, o pasta yaptığı iş de o tabanın altında hesaplanamaz.
Aksi hâlde payda "akamaz" derken pay akabiliyormuş gibi davranır.

İntegral parça parça alınır. `ε* = (70/K)^(1/n)` tabanın kesiştiği gerinimdir:

| Bölge | Koşul | `sigmaBar` |
|---|---|---|
| Pasın tamamı taban üzerinde | `ε_çıkış ≤ ε*` | `70 MPa` |
| Geçiş | `ε_giriş < ε* < ε_çıkış` | iki parçanın ağırlıklı ortalaması |
| Taban devre dışı | `ε_giriş ≥ ε*` | saf Hollomon integrali |

Tabanın çekme gerilmesine katkısı (`tabanPayi`) %0,5'i aşarsa arayüz uyarı gösterir.
Eşik projenin kendi sayısal toleransıdır: tabanın etkisi tolerans kadar büyüdüğünde
sayı artık bir modelleme tercihini taşıyor demektir. Varsayılan senaryolarda pay
%0,21 civarındadır (uyarı çıkmaz); pas başına kesit azalma %10'un altına inince
eşik aşılır.

### M-2 · İlk pasta `epsIn = 0` — gelen filmaşin şekil değiştirmemiş kabul edilir

Birikmiş gerinim hesabı 1. pasta `epsIn = 0` ile başlar. Bu, hatta giren **8 mm
filmaşinin sıcak haddelenmiş / tavlanmış durumda** olduğu ve daha önce soğuk şekil
değiştirmediği anlamına gelir.

Pratikte doğru kabul: sürekli döküm–haddeleme (Southwire, Contirod vb.) çıkışı
filmaşin sıcak haddelenmiş hâlde gelir ve pekleşmemiştir. Ancak filmaşin daha önce
soğuk çekilmiş veya sertleşmiş bir stoktan geliyorsa, gerçek akma gerilmesi
modelin verdiğinden **yüksek**, emniyet payı ise **dar** olur — bu durumda araç
iyimser taraftadır.

### M-3 · Ara tavlama gerinimi tam sıfırlar

Ara tavlama işaretlenen pastan sonra `epsIn = 0` yapılır; yani tavlama tam ve
malzeme başlangıç durumuna dönüyor kabul edilir. Kısmi tavlama (toparlanma,
kısmi yeniden kristalleşme) modellenmez.

### M-4 · Sürtünme ve kalıp geometrisi bütün paslarda aynı

`mu` ve `alphaDeg` bütün paslar için tek değerdir. Gerçek hatlarda kalıp açısı
pastan pasa değişebilir; araç bunu desteklemez.

Bunun görünür bir sonucu var: `tapered` (kademeli azalan) yönteminde son paslarda
kesit azalması düştükçe delta yükselir (`Δ = α(1+√(1−r))²/r`) ve üst sınırı (3,0)
aşabilir — sınamada 16 pas için delta aralığı 2,10–3,23 çıkıyor. Gerçek hatlarda bu,
**bitirme paslarında daha küçük yarı açılı kalıplar** kullanılarak çözülür; modelde
açı sabit olduğu için uyarı olarak görünür. Yani buradaki yüksek delta değerleri
modelin sabit açı kabulünden kaynaklanır, programın kendisinden değil.

### M-5 · Pas sayısı ve dizi kurulumu

`equalStrain` ve `tapered` yöntemlerinde otomatik pas sayısı
`MAX_R_PER_PASS = 0.20` (pas başına maks %20 kesit azalma) baş parmak kuralından
türetilir. Bu fiziksel bir sınır değil, sektörde yaygın bir başlangıç noktasıdır;
teorik sınır `r > 1 − 1/e = 0,632`'dir ve ayrıca kontrol edilir.

`equalSafety` yönteminin otomatik pas sayısı bu kuraldan **türetilmez**: emniyet
oranını hedefin (0,50) altında tutan **en küçük** pas sayısı 1'den başlanarak aranır.
Arama yalnızca emniyet ve teorik sınıra bakar; ortaya çıkan delta ve ΔT uyarıları
bastırılmadan gösterilir.

`equalSafety` yöntemi kullanıcı pas sayısını elle verdiğinde hedef emniyete
ulaşamayabilir; bu durumda araç hata vermez, o pas sayısıyla en dengeli dağıtımı
kurar ve riskli pasları işaretler.

### M-6 · ΔT adyabatik bir üst sınırdır

Pas başına sıcaklık artışı `ΔT = σ_D / (ρ·c_p)` bağıntısıyla hesaplanır. Bu,
**şekil verme işinin tamamının ısıya döndüğü ve hiç kayıp olmadığı** varsayımıdır.
Gerçekte ısının önemli bir kısmı kalıba ve yağa geçer, ayrıca paslar arasında
soğutma vardır — bu yüzden gerçek sıcaklık artışı modelin verdiğinden **düşüktür**.

Uyarı eşiği bu yüzden **100 °C**'dir: adyabatik üst sınırın 100 °C'yi aşması,
yağ/emülsiyon bozunması bakımından anlamlı bir işarettir. Arayüzde bu büyüklük
"sıcaklık artışı" değil **"adyabatik ΔT"** olarak etiketlenir.

### M-7 · Güç, kalıplarda harcanan mekanik güçtür

Tablodaki güç `P = F · v₁`, yani **kalıplarda harcanan mekanik güçtür**. Motor ve
aktarma verimi dahil **değildir**: şebekeden çekilen güç, aktarma ve motor kayıpları
nedeniyle bu değerin yaklaşık **%15–25 üstündedir**. Soğutma, yağ devri gibi yardımcı
tüketimler hiç hesaba katılmaz.

### M-8 · Hız hat boyunca zincirlenir, kütle debisi sabittir

Bir pasın giriş hızı, bir önceki pasın çıkış hızıdır: sürekli bir hatta bütün
kalıplardan aynı kütle geçer. `v0` girdisi **hat girişi** hızıdır (8 mm filmaşin);
son pasın çıkış hızı hattın çıkış hızıdır (Otomotiv ince için 2,0 → 67,2 m/s).

Kütle debisi `ṁ = ρ·A·v` bütün paslarda sabit kalır (900,8 g/s) ve `kWh/ton`
hesabı buna dayanır. T4 bunu raporlanan çap ve hız değerlerinden doğrular,
T7 ise toplam işin ideal şekil verme işine oranını (1,78) sınar.

### M-9 · Emniyet oranı pekleşme katsayısından bağımsızdır

Emniyet oranı, pekleşme katsayısı `K`'dan **koşullu olarak** bağımsızdır. `σ_d` ve
`σ_f` ifadelerinin ikisi de `K` ile doğrusal olduğu için oran sadeleşir — ama
yalnızca tabanın devrede olmadığı ya da tamamen baskın olduğu bölgelerde. Aradaki
geçiş bölgesinde sadeleşmez, çünkü `ε* = (70/K)^(1/n)` sınırının kendisi `K`'ya
bağlıdır (bkz. M-1 ve K-15).

Sayısal doğrulama (referans pas, `K` = 315 / 530):

| Bölge | Emniyet oranı farkı |
|---|---|
| Taban devre dışı (`ε_giriş = 0,05`) | `0` — on iki basamağa kadar aynı |
| Geçiş (`ε_giriş = 0`, referans pas) | `2,0 · 10⁻³` — **sadeleşmiyor** |
| Tamamen taban üzerinde (`r = %0,25`) | `0` — iki taraf da 70 MPa'ya sabit |

`equalSafety` programının yapısı `K` ile değişmez (aynı N = 11, aynı çaplar);
en yüksek emniyet oranı altıncı basamakta oynar (0,495384 → 0,495237), güç ise
doğrudan ölçeklenir: 212,0 → 356,6 kW.

Bunun araçtaki üç sonucu: **T8** bu değişmezliği sınar, duyarlılık analizi
**iki ayrı grafik** olarak sunulur (emniyet için K çubuğu sıfır, enerji için en
büyük), ve emniyet bandı yalnızca `n` ile `μ`'den beslenirken enerji bandı üçünden
birden beslenir.

## Sonuç

Model, kopma riskini **mertebe olarak** doğru gösterir ve paslar arası
karşılaştırma için güvenilirdir. Mutlak değerler `K`, `n` ve `mu` seçimine
duyarlıdır — literatürde ETP bakır için `K = 315–530 MPa`, `n = 0,35–0,54`
aralığı geçer, tek bir doğru değer yoktur. Bu yüzden araçta belirsizlik bandı
ve duyarlılık analizi bulunur.
