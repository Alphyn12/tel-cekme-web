# Kabuller ve Sınırlar

Bu araç bir **ön tasarım** aracıdır, üretim reçetesi değildir. Aşağıdaki maddeler
modelin bilerek dışarıda bıraktığı veya sabitlediği şeylerdir. Araç içindeki
"Kabuller ve sınırlar" bölümü bu dosyayla aynı içeriği taşır.

## Hesaba katılmayanlar

- Kalıp esnemesi ve elastik geri yaylanma
- Yağ filmi kalınlığı ve hidrodinamik yağlama rejimi (sürtünme katsayısı sabit alınır)
- Geri gerilim (back tension)
- Şekil değiştirme hızının akma gerilmesine etkisi
- Sıcaklığa bağlı akma gerilmesi değişimi (sıcaklık artışı hesaplanır ama akmaya
  geri beslenmez)
- Kalıba ve yağa giden ısı ile paslar arası soğutma (bkz. M-6)
- Kalıp aşınmasının pas boyunca geometriyi değiştirmesi
- Tel içindeki artık gerilmeler ve merkez–yüzey gerinim farkı

## Modelleme tercihleri

### M-1 · Akma gerilmesine 70 MPa taban

`σ = K·εⁿ` bağıntısı `ε → 0`'da sıfır verir; bu fiziksel değildir. Bu yüzden
**görüntülenen** akma gerilmelerine tavlanmış ETP bakır için `SIGMA_Y0 = 70 MPa`
taban uygulanır. `sigmaBar` (pas boyunca ortalama akma gerilmesi) bu tabandan
etkilenmez, analitik integralden gelir.

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

Emniyet oranı, pekleşme katsayısı `K`'dan **bağımsızdır** — `σ_d` ve `σ_f`
ifadelerinin ikisi de `K` ile doğrusal olduğu için oran sadeleşir (akma tabanının
devrede olmadığı, `ε > 0,02` bölgesinde). Bu nedenle `K`'nın literatürdeki geniş
aralığı (315–530 MPa) **kopma riskini etkilemez**, buna karşılık kuvvet, güç ve
sıcaklık artışını doğrudan oranlı biçimde etkiler.

Sayısal doğrulama (referans pas, `K` = 315 · 380 · 450 · 530):
emniyet oranı dördünde de **0,29214535** — sekiz basamağa kadar aynı.
`equalSafety` programının tamamı da değişmez (aynı N, aynı çaplar); değişen tek
şey güçtür: 212,0 → 356,6 kW. Akma tabanı yalnızca `ε < 0,014` altında devreye
girer; gerçek hiçbir pas oraya inmez.

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
