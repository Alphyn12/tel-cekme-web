# Karar Kaydı

## 7 Eylül 2026 inceleme kararları

Görünen girdi ile hesap girdisi aynı hassasiyette tutulur; URL sessizce yuvarlamaz.
Geçersiz girişte son hesap uyarıyla korunur, dışa aktarma ve sabitleme kapanır.
Belirsizlik alt/üst girdileri ayrı doğrulanır, nominal sonuç bantla birlikte gösterilir.
K/μ/n/hız/açı değişikliği ara tavlamaları silmez; çap/pas sayısı değişikliği sıfırlar.
Manuel programın duyarlılığı aynı çap dizisinde hesaplanır; pas sayısı çubuğu yoktur.
0,50/0,60 eşikleri fiziksel kopma sınırı olarak sunulmaz; 70 MPa tabanı ve
köşe taraması sınırları görünür biçimde belirtilir. Önceki kararlar aşağıda korunmuştur.

Bu araçta, ilk şartnamede yazmayan ya da geliştirme sırasında değişen mühendislik
kararları ve gerekçeleri. Kod bu kararlara uyar; bir karar değişirse burası da
güncellenir.

Kararların çoğu bir ölçümden çıktı: bir sayı beklenenden farklı geldi, sebebi
araştırıldı, model ya da arayüz ona göre değişti. Ölçümler kararların içinde.

## K-01 · Pas sayısı — otomatik N + kullanıcı ezmesi
**Tarih:** 2026-09-03 · K-03 ve K-04 ile ayrıntılandırıldı

`equalStrain` ve `tapered` bir pas sayısı (N) olmadan dizi kuramaz; girdi panelinde
böyle bir alan yoktu.

**Karar:** N varsayılan olarak otomatik hesaplanır, kullanıcı isterse ezebilir.
Girdi panelinde "Pas sayısı" alanı ve yanında "Otomatik'e dön" düğmesi bulunur.
Manuel çap dizisi girildiğinde alan devre dışı — N diziden gelir.

**N ne zaman otomatiğe döner:**

| Olay | N |
|---|---|
| Hazır senaryo düğmesi | otomatiğe döner |
| Başlangıç veya hedef çap elle değiştirildi | otomatiğe döner |
| Pas dağıtım yöntemi değiştirildi | **korunur** |

K-16'dan sonra bu kural taslak girdiye uygulanır: N burada sayılan olaylarda
hemen değişir, ama yeni N'e göre hesap HESAPLA'ya basılınca yapılır.

Yöntem değişiminde N korunur, çünkü manuel N'in varlık sebebi sabit kalıp sayısını
sabit tutup yöntemleri karşılaştırmaktır. Kullanıcı 13 girip yöntem değiştirdiğinde
13 kaybolursa alanın işlevi kalmaz.

## K-02 · Yayın: private çalışma deposu + temiz public depo
**Tarih:** 2026-09-03, 2026-09-04 ve 2026-09-15'te güncellendi

Geliştirme boyunca depo private tutuldu. Yayında ikiye ayrıldı: bütün geçmişi ve
çalışma notlarını taşıyan **private arşiv**, ve teslim edilen hâli taşıyan
**public depo** (`index.html`, `og.png`, `favicon.svg`, README, LICENSE ve dört
teknik doküman). Site public depodan Vercel ile yayınlanır; `og:url` ve
`og:image` mutlak adrestir, çünkü önizleme robotları göreli yolu okumaz.

İki depolu düzenin bedeli, eşitlemenin elle yapılmasıdır: 4–15 Eylül arasında
yedi commit çalışma deposunda kaldı, site eski sürümü sunmaya devam etti. Hata
sessizdi, çünkü push her iki depoya da sorunsuz gidiyordu ve fark yalnızca
canlı sayfada görünüyordu. Eşitleme artık `tools/yayinla.sh` ile yapılır: yayın
dosyalarının listesi betikte sabittir, işlenmemiş değişiklik veya itilmemiş
commit varsa durur, private yolların listeye sızmasını denetler ve itişten sonra
canlı sayfanın gerçekten yeni sürümü sunduğunu doğrular.

## K-03 · Dağıtım yöntemleri yeniden tanımlandı
**Tarih:** 2026-09-03

`equalReduction` ile `equalStrain` matematiksel olarak aynı şeydi
(`epsPas = ln(1/(1-r))`). Yöntemler şu hâlini aldı:

| Eski | Yeni |
|---|---|
| `equalReduction` | `equalStrain` — eşit gerinim |
| `equalStrain` | `tapered` — kademeli azalan, `TAPER_RATIO = 0.65` |
| `equalSafety` | `equalSafety` — değişmedi |

Sabitler, otomatik N formülü, üç yöntemin dizi kurulumu, ortak kontroller ve
pas sayısı doğrulama tablosu planın BÖLÜM 1'inde
"Pas dağıtım yöntemleri ve pas sayısı" başlığı altında.
Doğrulama testi sayısı dörtten **beşe** çıktı (T5: otomatik N = 16 / 11 / 25).

### K-03a · "Çok ince" senaryosunun 1. pas çıkışı düzeltildi

Kaynak tabloda bu hücre 7,169 mm veriliyordu; doğrusu **7,160 mm**.
Hesap: `epsTotal = 5,5452`, `N = 25`, `epsPas = 0,221807`,
`d1 = 8 · exp(−0,110904) = 7,1602 mm`. Aynı satırdaki `r = %19,89` bununla tutarlı.
T5 yalnızca N'i (16 / 11 / 25) kontrol ettiği için testi etkilemiyor.

## K-04 · `equalSafety` manuel N'de hata vermez
**Tarih:** 2026-09-03

Kullanıcı `TARGET_SAFETY`'ye ulaşmaya yetmeyecek kadar küçük bir N girerse
(örneğin 8,00 → 1,38 için 13), yöntem iterasyonu bırakır, **o N ile en dengeli
dağıtımı kurar** ve yüksek emniyet oranlarını kırmızı gösterir. Hata vermez.

Gerekçe: aracın en çok sorulacak sorusu *"13 kalıpla 1,38'e inebilir miyim?"*
Cevap "hayır" ise araç bunu hesaplayıp göstermeli, reddetmemeli.
N'i artırma iterasyonu **yalnızca otomatik modda** çalışır; manuel modda N sabittir.
`r > R_THEORETICAL_MAX` sert hatası bundan bağımsız, her durumda geçerli.

## K-05 · Manuel N'de otomatik değer de görünür
**Tarih:** 2026-09-03

Pas sayısı alanının altında sessiz bir satır: *"Otomatik: 16 pas"*. Kullanıcı 13
girdiğinde aracın önerisinin 16 olduğunu görür, farkın sebebini tabloda okur.

## K-06 · Kimlik başlığı — gerçek bilgiler, yer tutucu yok
**Tarih:** 2026-09-03

FAZ 6'daki "yer tutucu" ifadesi geçersiz; BÖLÜM 2'nin "yer tutucu metin yok"
kuralı kazanır. Sayfaya gömülecekler:

- **Ad:** Barış Kırlı
- **Unvan:** Makine Mühendisi / Mechanical Engineer (dile göre değişir)
- **E-posta:** kirlibaris12@gmail.com
- **LinkedIn:** https://www.linkedin.com/in/bariskirli
- **Telefon numarası sayfaya konmaz.** Sayfa herkese açık ve taranabilir olacak.

## K-07 · `equalSafety` otomatik pas sayısı yeniden tanımlandı
**Tarih:** 2026-09-04

**Sorun:** `equalSafety`'nin otomatik N'i, tamamen başka bir yöntemin baş parmak
kuralından (`MAX_R_PER_PASS = 0,20`) türetiliyordu. Sonuç: üçüncü yöntem eşit
gerinimle **aynı** pas sayısını veriyor, yani yeni bir şey söylemiyordu.

**Yeni tanım:** `N = 1`'den başlayarak yukarı ara; hem **max emniyet ≤ TARGET_SAFETY**
hem de **her pasta `r ≤ R_THEORETICAL_MAX`** koşulunu sağlayan **en küçük** N seçilir.
`MAX_PASSES`'a kadar çözüm yoksa hata verilir.

Üç ayrıntı:

1. Arama yalnızca emniyet ve teorik sınır üzerinden yapılır. Ortaya çıkan programda
   delta veya ΔT uyarısı varsa **bastırılmaz, gösterilir** — az pas, pas başına daha
   çok ısı demektir ve kullanıcının görmesi gereken şey tam olarak bu ödünleşimdir.
2. Manuel N davranışı değişmedi; K-04 aynen geçerli.
3. "Otomatik: X pas" satırı artık yönteme göre farklı değer gösterir
   (eşit gerinim 16, eşit emniyet 11 gibi). Bu doğru ve bilgilendiricidir.

**Sonuç (üç senaryo):** eşit gerinim 16 · 11 · 25 → eşit emniyet **11 · 8 · 17**.
Doğrulama testi ikiye bölündü: T5 eşit gerinim, **T6 eşit emniyet**.

Bu, karşılaştırma modunun manşetidir: *"Eşit gerinim 16 pas istiyor; aynı güvenlik
seviyesini 11 pasla da tutturabilirsin — 5 kalıp az."*

## K-08 · Sıcaklık uyarı eşiği 60 → 100 °C, etiket "adyabatik ΔT"
**Tarih:** 2026-09-04

`DT_WARNING = 60` normal senaryoların hepsinde tetikleniyordu (78 · 67 · 76 · 94 ·
100 °C) — her satır sarı yanacak, uyarı hiçbir şeyi ayırt etmeyecekti. Eşik
şartnameye gerekçesiz konmuştu.

**Karar:** `DT_WARNING = 100`. Model adyabatik artışı hesaplar (bütün iş ısıya
döner, kayıp yok), yani bir **üst sınır** verir; arayüzde etiket "sıcaklık artışı"
değil **"adyabatik ΔT"** olur. Gerekçe `kabuller.md` M-6'da.

Yan fayda: yeni eşikle eşit emniyet N=11 (100,3 °C) uyarı verir, N=12 (93,8 °C)
vermez — araç *"11 pas kopma açısından yeterli ama ısıl olarak tam sınırda"* diyor.

## K-09 · Kanonik kırmızı vaka
**Tarih:** 2026-09-04

Eşik renklerini sınamak için her zaman kırmızı üreten bilinen bir senaryo:
`8,00 → 1,38` · **`μ = 0,15`** · `α = 8°` · `K = 450` · `n = 0,35` ·
yöntem `equalStrain` · **pas sayısı elle 12** → 12 pasın 11'i kırmızı,
en yüksek emniyet **0,690**. Ayrıntı ve komşu değerler `docs/dogrulama.md`.

Tek başına `μ = 0,15` yetmiyor (otomatik 16 pasla 0,543, yani sarı); kontrol
listesindeki madde bu yüzden "μ = 0,15 + 12 pas" olarak güncellendi.

## K-10 · Delta uyarısı aksiyona çevrilecek (FAZ 3)
**Tarih:** 2026-09-04

Delta uyarısı `equalSafety` otomatik modda üç senaryoda da çıkıyor ve bu **yapısal**:
pas sayısını azaltmak pas başına kesit azalmasını büyütür, `Δ = α(1+√(1−r))²/r`
bağıntısında `r` büyüdükçe delta düşer. Eşik gevşetilmeyecek — varsayılan yöntemde
(eşit gerinim) delta 2,55 ve uyarı yok, yani eşik ayırt ediyor.

**Karar:** Delta `α` ile doğrusal olduğu için gereken kalıp açısı tam hesaplanır:

```
alphaGerekli = alphaMevcut * DELTA_MIN / deltaMevcut
```

Uyarı metni bilgi değil aksiyon versin, örnek (Otomotiv ince, `equalSafety`, N=11):

> *"11. pasta delta 1,32 (alt sınır 1,5) — aşırı sürtünme ve kalıp aşınması.
> Kalıp yarı açısını 9,1°'ye çıkarmak bu pası sınıra döndürür."*

Gerçek fabrika pratiğiyle örtüşür: bu sorun pas eklenerek değil, kalıp açısı
büyütülerek çözülür.

## K-11 · Hız hat boyunca zincirlenir
**Tarih:** 2026-09-04

Şu an `buildSchedule` her pasa **aynı** `v0`'ı veriyor; `calcPass` da
`v1 = v0·(d0/d1)²` hesaplıyor. Sonuç: her pasta çıkış hızı 2,49 m/s ve
**kütle debisi pastan pasa düşüyor** (900 → 33 g/s). Sürekli bir hatta bu
fiziksel olarak imkânsız: bütün kalıplardan aynı kütle geçer.

Zincirlenmiş hâlde (pas *i*'nin giriş hızı = pas *i−1*'in çıkış hızı):
kütle debisi sabit 900,8 g/s, son hız 67,2 m/s, toplam güç 75,8 → **325,8 kW**.

Bu **BÖLÜM 1'deki hiçbir formülü değiştirmez**; yalnızca her pasa hangi `v0`'ın
verildiğini değiştirir — `epsIn`'in birikimli taşınmasıyla birebir aynı mantık.
FAZ 3'teki `kWh/ton` hesabı kütle debisine dayanır.

**Uygulandı.** Doğrulanan değerler (Otomotiv ince, eşit gerinim, 16 pas):
`ṁ = 900,8 g/s` sabit · `v₁₆ = 67,2 m/s` · toplam mekanik güç `325,8 kW` ·
`100,5 kWh/ton`. İdeal şekil verme işi 56,4 kWh/ton, oran **1,78** (şekil verme
verimi %56 — tel çekmede tipik aralık %50–70).

Referans pas etkilenmedi (tek pas hesabı; hız yalnızca `v1` ve `P`'ye girer):
`sigmaD = 77,61` · `safety = 0,2921` · `delta = 2,520`.

Beraberinde:
- **T4 yeniden yazıldı:** artık hacim korunumu değil **kütle korunumu** sınanıyor ve
  kütle debisi formülden değil **raporlanan çap ve hız** değerlerinden hesaplanıyor
  (formülden hesaplansa test tanım gereği geçer, tam da bu hatayı kaçırırdı).
- **T7 eklendi:** toplam iş / ideal iş oranı, geçme koşulu 1,30 – 2,50.
- Güç etiketi **"Mekanik güç"** oldu (bkz. `kabuller.md` M-7), `v0` etiketi
  **"Giriş hızı — hat girişi"**.
- Zincirleme öncesi ölçülen toplam güç değerleri (75,8 / 75,2 / 74,1 kW) geçersiz;
  Faz 3 karşılaştırmaları yeni modelle üretilecek.

## K-12 · Emniyet oranının K'dan bağımsızlığı Faz 5'i şekillendirdi
**Tarih:** 2026-09-04

Cebirsel bulgu (`kabuller.md` M-9) doğrulandıktan sonra Faz 5 şu şekilde kuruldu:

1. **T8 · K değişmezliği** testi eklendi — `K = 315` ile `K = 530` arasında emniyet
   oranı farkı `< 1e-6`. Ucuz test, `passStress` içindeki cebirsel bir hatayı anında
   yakalar. Toplam test sayısı sekiz.
2. **İki ayrı tornado grafiği.** Emniyet için sıralama `r · μ · n · α · K`
   (K çubuğu "etkisiz" yazar — grafiğin kendi doğrulaması); özgül enerji için
   `K · μ · pas sayısı · α`. Tek grafik yapılsaydı K sıfır görünür ve kullanıcı
   aracı bozuk sanırdı.
3. **Belirsizlik ile duyarlılık ayrıldı.** Belirsizlik = bilmediklerimiz (K, n, μ);
   duyarlılık = değiştirebildiklerimiz (α, r, pas sayısı). Arayüzde ayrı bölümler.
4. **Enerji manşette bant olarak.** Özet şeridinde enerji, güç ve en yüksek emniyet
   bant açıkken aralık gösterir; eşik rengi bandın **üst ucuna** göre seçilir.

## K-13 · Köşe taraması çap dizisini sabit tutar
**Tarih:** 2026-09-04

Köşe taramasında parametre değişince `equalSafety` ve `tapered` **pas programını da**
değiştirir (N ve çaplar). O zaman köşeler farklı programları karşılaştırmış olur ve
bant, malzeme belirsizliğiyle program değişimini birbirine karıştırır.

**Karar:** Önce merkez parametrelerle program kurulur, sonra **o programın çap dizisi
sabit tutularak** sekiz köşede yeniden hesaplanır. `equalStrain`'de fark etmez
(program yalnızca geometriden gelir), `equalSafety` ve `tapered`'da eder.

Otomotiv ince · `equalSafety` · varsayılan bant ile ölçüldü:

| Yöntem | Pas sayısı | Enerji | En yüksek emniyet |
|---|---|---|---|
| Çap dizisi sabit (**uygulanan**) | 11 (hepsinde) | 71,3 – 128,8 kWh/t | **0,451 – 0,571** |
| Program her köşede yeniden kurulsaydı | 10 – 13 arası değişir | 70,0 – 132,6 kWh/t | 0,478 – 0,492 |

Yanlış yöntem emniyet bandını **0,49'da düz gösterip riski gizliyor**: her köşe kendi
programını yeniden optimize ettiği için emniyet hep hedefe oturuyor. Doğru yöntemde
seçilen programın kötü sürtünmede 0,571'e çıktığı, yani 0,50 sınırını aştığı görünüyor.

Ayrıca bandın **hangi köşeden geldiği** arayüzde yazılır: sekiz köşenin tamamı
parametreleriyle listelenir, alt ve üst uç işaretlenir. Kullanıcı böylece önce hangi
varsayımı netleştirmesi gerektiğini görür.

## K-14 · Ekranda tek doğru kaynak — `ciz()` ya hep ya hiç
**Tarih:** 2026-09-07

Dört ayrı arayüz hatası bildirildi; dördü de aynı kökten geliyordu: **ekranda o an
ne gösterildiğinin tek bir doğru kaynağı yoktu.** `ciz()` hesapla DOM yazımını iç
içe yapıyordu, paneller birbirinden bağımsız yazılıyordu.

| Belirti | Görünen |
|---|---|
| Kalıp kesiti kaydırıcıları | Bir kez dokununca donuyor; senaryo değişse bile kesit tabloya ait olmayan sayıyı gösteriyor |
| `esitBolge` çökmesi | Negatif emniyette bölge boş kalıyor, istisna `ciz()`'i yarıda kesiyor: özet yeni, tablo ve grafikler eski |
| Hata durumu | Kalıp özeti, pas etiketi, kaydırıcı değerleri, tornado başlığı bir önceki hesaptan kalıyor |
| Geçersiz girdi | Sessizce yutuluyor, eski sonuç güncelmiş gibi duruyor |

Bu, aracın en çok savunması gereken şeyi — **tutarlılığı** — vuruyordu.

**Yapısal karar:** `ciz()` üç aşamaya ayrıldı ve aşamalar arasında sızıntı yok.

1. **Girdi denetimi** — sınır dışı alan varsa hesap hiç yapılmaz
2. **`gorunumHesapla()`** — ekranda görünecek her şeyi hesaplar, **DOM'a hiç dokunmaz**
3. **`gorunumuYaz(g)`** — saf DOM yazımı, içinde hesap yok

Herhangi bir aşama düşerse `hesaplanamadiDurumu()` bütün panelleri **birlikte**
boşaltır. 3. aşama da `try/catch` içindedir: beklenmedik bir çizim hatasında bile
ekran yarım kalmaz, tutarlı hata durumuna düşer. Yarısı yeni yarısı eski bir görünüm
artık üretilemez.

**Türev durum tek kapıdan:** kalıp kesiti kaydırıcılarını sıfırlayan `programDegisti()`
eklendi ve programı değiştiren dokuz yolun hepsi oradan geçiyor (girdi, senaryo,
yöntem, pas sayısı, otomatiğe dön, tavlama kutuları ×2, manuel hesapla, manuel
temizle). Kaydırıcı yalnızca kullanıcı ona dokunduğu sürece geçerlidir.

**Sınırını bilen model:** `GIRDI_SINIRLARI` çekirdeğe kondu (aralıklar
`dogrulama.md`'de). Siebel yaklaşımı ve Hollomon pekleşmesi bu aralıkların dışında
fiziksel anlamını yitirir — negatif sürtünme çekme gerilmesini eksiye düşürür ve
`esitBolge`'yi çökerten şey de buydu. Kök çözüm `esitBolge`'yi yamamak değil, kapıyı
girişte kapatmaktı; `esitBolge` yine de savunmacı hâle getirildi (bölge boşsa `null`).
Kural tek yerde durur: arayüz ve **T9** aynı `girdiDenetimi`'ni çağırır.

Geçersiz girdide **eski sonuçlar silinmez** — silinirse kullanıcı bir tuş hatasıyla
bütün ekranı kaybeder. Korunur, ama üstüne "güncel değil" uyarısı konur. Sessizce
eski değeri güncelmiş gibi göstermek en kötüsüdür.

**Performans bir tutarlılık meselesi:** `runValidation()` sabit girdilerle çalışır,
sonucu asla değişmez; her tuş vuruşunda koşuyordu ve çizimin %76'sını yiyordu.
Bir kez koşuyor. Formül ve kabul panelleri yalnızca dil değişince yeniden yazılıyor.
Tuş başına **38 ms → 11 ms**.

**Kanonik kırmızı vaka (K-09) doğrulandı:** bu tur sonrası `μ = 0,15` + 12 pas ile
12 pasın 11'i kırmızı, en yüksek emniyet **0,690**, ilk kırmızı pas 2, maks ΔT
139,8 °C — altı satırın tamamı ve komşu değerler birebir aynı. Çekirdek değişmedi,
değişmemeliydi de.

## K-15 · 70 MPa tabanı ortalama akma integraline de uygulanır
**Tarih:** 2026-09-08

Araç iki farklı akma gerilmesi tanımını aynı orana koyuyordu. `sigmaFOut` tabana
tabiydi (`max(K·εⁿ, 70)`), pas boyunca ortalama akma `sigmaBar` değildi. Yani payda
"bakır 70 MPa'nın altında akamaz" derken, pay o işi sanki akabilirmiş gibi
hesaplıyordu. Bu, aracın kendi M-1 kabulüyle çelişiyordu.

**Ne kadar önemliydi:** normal kullanımda küçük, kenarda büyük.

| Pas başına kesit azalma | `sigmaBar` eksik tahmini |
|---|---|
| %20 (varsayılan senaryolar) | %0,20 |
| %10 | %0,55 — projenin kendi %0,5 toleransını aşıyor |
| %5 | %1,45 |
| %2 | %4,93 |
| %0,5 | %25,4 |

**Karar:** taban her iki yerde de geçerlidir. `ε* = (70/K)^(1/n)` kesişim gerinimi
hesaplanır, integral parça parça alınır (kapalı form, iterasyon yok). Ayrıntı ve
bölge tablosu `kabuller.md` M-1'de.

**Bedeli — ve neden kabul edildi:** `K` değişmezliği artık koşulludur. `ε*` sınırının
kendisi `K`'ya bağlı olduğu için geçiş bölgesinde oran `K` ile oynar (referans pasta
`2,0 · 10⁻³`). Taban devre dışıyken ve pasın tamamı taban üzerindeyken tam olarak
sadeleşmeye devam eder. **T8 bu üç bölgeyi birden sınayacak biçimde yeniden yazıldı**;
eski hâli yalnız birinci bölgeye bakıyordu ve tutarsız çekirdeği de geçiriyordu.

Değişen sayılar: referans pas gerilme kaynaklı büyüklüklerde %0,20 yukarı
(`sigmaD` 77,61 → 77,77 MPa, `safety` 0,2921 → 0,2927, `F` 3125 → 3131 N).
**Değişmeyenler:** kanonik kırmızı vaka K-09 (0,690 · 11/12 kırmızı), üç hazır
senaryonun özet değerleri, otomatik pas sayıları, kütle korunumu.

**Arayüz uyarısı doğru yere taşındı.** Önceki uyarı pasın *çıkışına* bakıyordu
(`K·ε_çıkış^n < 70`); belirleyici olan ise integral aralığının `ε*` eşiğini kesip
kesmediği, yani *girişidir*. Giriş gerinimi sıfır olan her pas — her 1. pas ve her
tavlama sonrası pas — bu eşiği zaten keser. Çıkışa bakan koşul yalnızca `r < %0,25`
altında ateşleniyor, hatanın toleransı aştığı `%10 → %0,25` bandını tamamen
kaçırıyordu. Uyarı artık tabanın çekme gerilmesine katkısını (`tabanPayi`) doğrudan
ölçüyor ve %0,5'i aşınca çıkıyor; ölçülen büyüklük eşikle aynı büyüklük.

## K-16 · Hesap açık bir düğmeye bağlandı; girdi yazmak sonucu değiştirmez
**Tarih:** 2026-09-15

Araç her tuş vuruşunda baştan hesaplıyordu. Bu, tek başına bakıldığında hızlı bir
arayüz; kullanımda ise güveni bozuyor. Sebebi şu: bu modelde bazı girdiler bazı
sayıları hiç değiştirmez. `K` emniyet oranında sadeleşir (taban devre dışıyken),
hız gerilmeyi değil gücü değiştirir, bant uçları nominal girdilerden ayrıdır.
Kullanıcı bir sayı yazıp tablonun kıpırdamadığını gördüğünde iki açıklama arasında
ayrım yapamaz: "bu girdi bu sonucu gerçekten etkilemiyor" ile "arayüz girdiyi almadı".
Sonuç sürekli kendiliğinden değiştiği için de hangi sayının hangi girdiye ait
olduğu hiçbir an sabitlenmiyordu.

**Karar:** Ekrandaki sayıları değiştiren tek yol HESAPLA düğmesidir. İstisnası yoktur —
hazır senaryo düğmeleri, pas dağıtım yöntemi, pas sayısı, ara tavlama kutuları,
belirsizlik bandı ve manuel çap listesi dâhil, hepsi girdiyi değiştirir, hesabı değil.

**Nasıl uygulanır — iki nesne:**

| Nesne | Ne tutar | Ne zaman değişir |
|---|---|---|
| `taslak` | kullanıcının kutulara yazdığı değerler | yazarken, anında |
| `durum`  | ekranda gösterilen hesabın girdileri | yalnızca `hesapla()` içinde |

Sonuç boru hattının tamamı (`buildSchedule`, `bandHesapla`, `duyarlilik*`,
`csvUret`, `adresYaz`) `durum`'dan okur ve bu değişiklikte hiç dokunulmadı.
Böylece "ekrandaki sayı hangi girdiyle üretildi" sorusunun tek bir cevabı vardır:
`durum`. Adres çubuğu da yalnızca hesapla ile güncellenir; paylaşılan bağlantı
her zaman gerçekten hesaplanmış bir programı taşır.

**Ayrışma gizlenmez, gösterilir.** Taslak ile durum ayrıldığı anda üç gösterge
birlikte yanar ve üçü de tek fonksiyondan (`hesaplaDurumunuYaz`) yazılır:
düğmenin yanındaki durum satırı, hangi girdinin hangi değerden hangi değere
gittiğini tek tek sayan şerit, ve sayfa uzun olduğu için ekranın altında duran
yapışkan çubuk. Sonuç bölümleri bu sürede soluklaşır. "Değişiklikleri geri al"
taslağı ekrandaki hesaba döndürür.

**Görünüm değişimleri bu kuralın dışındadır** — çünkü sonucu değiştirmezler:
dil, tema, panel katlama, kalıp kesiti kaydırıcıları, tabloda pas seçme ve
A/B karşılaştırmasını sabitleme. Bunlar aynı `durum`'u yeniden yazar, aynı
sayıları üretir.

**Girdi geçersizken** düğme kapanır ve sebebi yazılır; bayat kutusu alan hatası
ile bant hatası için ortak tek kaynaktan yazılır.

**Klavye:** girdi kutusunda Enter, sayfanın her yerinde Ctrl/Cmd+Enter.

Açılış tek istisnadır: sayfa boş açılmaz, adresteki senaryo (ya da varsayılan
"Otomotiv ince") bir kez hesaplanmış olarak gelir.

## Varsayılan kabuller (aksi söylenmedikçe geçerli)

- `anneals` dizisi **1-indekslidir** ve içindeki numaranın ait olduğu pastan
  **sonra** `epsIn` sıfırlanır.
- **1. pasta `epsIn = 0`** — gelen 8 mm filmaşin sıcak haddelenmiş / tavlanmış,
  önceden soğuk şekil değiştirmemiş kabul edilir. Ayrıntısı `kabuller.md` M-2.
- Son pas hedef çapa `±0,001 mm` içinde oturur.
- `K = 450`, `n = 0.35`, `alphaDeg = 8`, `mu = 0.05` varsayılan girdilerdir
  (hazır senaryolardan gelir).
- `r > R_THEORETICAL_MAX` kontrolü **pas bazındadır**, hesabı durdurur ve hata
  mesajı hangi pas / hangi değer olduğunu söyler.
- Eşik ve sabitlerin tamamı tek bir adlandırılmış sabitler bloğunda toplanır.

## K-17 · Anlama katmanı: sayfa önce ne işe yaradığını söyler
**Tarih:** 2026-09-15

Sayfa doğrudan yedi uzman parametresiyle açılıyordu: `μ`, `K`, `n`, kalıp yarı açısı,
akma tabanı. Hesap doğruydu, terimlerin hiçbiri sayfada tanımlı değildi. Aracı
kullanabilmek için zaten Siebel yaklaşımını bilmek gerekiyordu; bilmeyen ziyaretçi
ilk ekranda kalıyordu. Eksik olan yorum değil — `oneriCumleleri()` sonucu zaten
sayıyla yorumluyor — **giriş ve sözlük** idi.

**Karar:** Hesap koduna dokunmadan dört parçalı bir anlama katmanı eklendi:

1. **Giriş kartı** (`.giris`): aracın ne yaptığı, hangi soruya cevap verdiği ve üç
   adımlık kullanım. Sonuca bağlı değildir, hesap beklemez.
2. **Karar satırı** (`.karar`): özet şeridinden önce gelen tek cümlelik sonuç —
   `8,00 → 1,38 mm · 16 kalıp`, en kritik pas, emniyet oranı, uyarı eşiği ve
   Güvenli / Sınırda / Riskli rozeti. Özet şeridi altı sayıyı eşit ağırlıkta
   gösteriyor, hangisinin karar sayısı olduğu görünmüyordu.
3. **Girdi açıklamaları** (`alanYardim`): her kutunun altında bir satır — alan ne
   demek ve tipik aralığı ne (`μ`: 0,03 iyi yağlanmış hat, 0,08 kötü).
4. **Sütun sözlüğü** (`sutunYardim`): tablo başlıklarının düz karşılığı. Aynı metin
   başlık ipucu (`title`) olarak da durur; ipuçları dokunmatik ekranda görünmediği
   için sözlük katlanır bir bölüm olarak tabloyla birlikte gelir.

Karar satırı ve rozet **sonuç** bölümünün içindedir: bekleyen değişiklikte diğer
sayılarla birlikte soluklaşır, hesaplanamadığında birlikte gizlenir (K-16, K-14).

**Yan sonuç — V3 tek dile indi.** Karar satırı ilk yazımında her zaman "en kritik pas
{n}" diyordu; oysa V3, tek bir belirgin tepe yoksa tek pas numarası vermenin
yanıltıcı olduğunu söylüyor ve öneri cümleleri bu ayrımı zaten yapıyordu. Şimdi
karar satırı, öneri cümleleri **ve** özet şeridi aynı ölçütü (`belirginTepe`,
`esitBolge`) kullanır: eşit gerinim programında üçü birden "6–16 aralığında 11 pas"
der. Önceden şerit aynı ekranda "pas 16" yazıyordu.

**Mobil:** Telefonda girdi paneli tek sütuna iner — iki sütunda açıklama satırı üçe
kırılıyor ve kutular hizasını kaybediyordu. Geniş ekranda etiket yüksekliği üç satıra
sabitlendi, çünkü İngilizcede `PEKLEŞME KATSAYISI + MPa` üç satıra taşıp aynı sıradaki
kutuları kaydırıyordu. 360 px'te yatay taşma yok, sözlük başlığı 44 px dokunma hedefi.

Giriş kartı ve sözlük baskıda gizlenir; karar satırı basılır.

## K-18 · Optimizasyon: kısıt tabanlı tarama, amaç fonksiyonu seçilir
**Tarih:** 2026-09-15

Araç pas sayısını zaten optimize ediyordu (eşit emniyet + otomatik), kesit azalmayı
yönteme göre dağıtıyordu, ama **kalıp yarı açısını hiç optimize etmiyordu** — tek
değer, bütün paslarda sabit (M-4). Daha tuhafı: düzeltici değerleri hesaplayan
fonksiyonlar (`gerekliAci`, `guvenliAzalma`) kodda vardı ve yalnızca öneri
cümlelerinde kullanılıyordu. Araç doğru açıyı söylüyor, uygulamıyordu.

**Karar:** Amaç fonksiyonu kullanıcı tarafından seçilen, kısıtları aracın kendi
eşikleri olan deterministik bir tarama eklendi. Değişkenler: kalıp yarı açısı
(tek değer — M-4 korunur), pas dağıtım yöntemi, pas sayısı. Kısıtlar: emniyet ≤ 0,50,
delta 1,5–3,0, ΔT ≤ 100 °C, `r` ≤ %63,2 — hepsi pas tablosunda görünen sayılar,
gizli ölçüt yok.

**Neden ızgara, neden sezgisel arama değil:** `buildSchedule` saf fonksiyon, tarama
yan etkisiz. Izgara deterministiktir — aynı girdi her zaman aynı programı verir,
yani sonuç test edilebilir. Üç test eklendi: çıktı bütün kısıtları sağlar, bağımsız
olarak yeniden taranan ızgaradaki en iyiden kötü değildir, ve iki koşu aynı sonucu
verir.

**Maliyet ölçüldü, tahmin edilmedi.** `equalSafety` çağrı başına 50–138 ms (kök bulma
iterasyonu), `equalStrain`/`tapered` ise ~0,05 ms. Tam ızgarada üç yöntem beş dakika
sürüyordu. Bu yüzden hızlı yöntemler tam ızgarada (81 açı × 60 pas), eşit emniyet
kaba (1°) sonra ince (±0,6° içinde 0,1°) aramayla ve yalnızca kendi otomatik pas
sayısıyla taranır — o yöntemde pas sayısı zaten hedefin sonucudur. Tarayıcıda
~9700 aday, 530 ms.

**K-16 korundu.** Optimizasyon ekrandaki hiçbir sayıyı değiştirmez: bulunan programı
girdilere alır, bekleyen değişiklik şeridi ne değiştiğini yazar, sonucu HESAPLA
getirir. Tarama 0,5 saniye tek iş parçacığında koştuğu için düğme önce "Taranıyor…"
yazar ve bir kare bekler; yoksa kullanıcı donmuş bir sayfa görür.

**Sınır — bu bir model optimumudur.** Model `μ`'yü `α`'dan bağımsız sabit alır; gerçekte
kalıp açısı değişince yağlama rejimi ve kalıp aşınması da değişir, ikisi de kabuller
listesinde "hesaba katılmayanlar" arasında. Optimizer "açıyı 9,3°'ye çıkar" derse bu
modelde kazançtır, sahada doğrulanması gerekir. Uyarı cümlesi sonucun altında sabit
durur, kapatılamaz.

## K-19 · Sürtünme kalibrasyonu: modeli ölçümle hatta bağlamak
**Tarih:** 2026-09-15

Araç `μ`'yü girdi olarak alıyordu ve kendi duyarlılık analizinde "en kritik
parametre budur" diyordu. İkisi birlikte tuhaf bir boşluk üretiyor: sonucu en çok
belirleyen sayı, kullanıcının en az bilebileceği sayı. `K` için literatür aralığı
var (315–530 MPa) ve zaten emniyet oranında sadeleşiyor; `μ` için literatür aralığı
bir şey söylemiyor, çünkü `μ` malzemenin değil **hattın** özelliğidir.

**Karar:** Ters problem eklendi. Kullanıcı ekrandaki programın bir pasını seçip o
pasta ölçülen çekme kuvvetini girer; araç `σ_d = F / A₁` ile gerilmeyi bulur ve
Siebel'i `μ` için çözer.

**Neden ikili arama:** `σ_d` ifadesinde sürtünme terimi `μ` ile doğrusaldır, yani
`σ_d` `μ`'de monoton artandır — kök tektir, türev gerekmez, yakınsama garantilidir.
Bir test bu monotonluğu ayrıca sınıyor: ters çözümün tekliği varsayım değil, kontrol
edilen bir özellik.

**Aralık dışında sayı uydurulmaz.** Ölçülen kuvvet sürtünmesiz alt sınırın (`μ = 0`)
altındaysa fizik değil ölçüm ya da girdi hatalıdır; üst sınırın üstündeyse sürtünme
modelin kapsadığı yerin dışındadır. İki durumda da aşılan sınır **kuvvet cinsinden**
yazılır, çünkü kullanıcının elindeki büyüklük odur.

**K-16 korundu:** çözülen `μ` girdilere alınır, ekrandaki sayılar HESAPLA'ya kadar
değişmez.

**Sınır — kalibrasyon bir kayıp toplayıcıdır.** Kasnak momentinden hesaplanan kuvvet
aktarma ve yatak kayıplarını içerir; o kayıplar `μ`'ye yıkılır ve sonuç sistematik
olarak yüksek çıkar. Daha derini: çözülen sayı modelin bütün ihmallerini (geri
gerilim, sıcaklık geri beslemesi, kalıp esnemesi) tek bir katsayıya yükler. Yani
"gerçek sürtünme katsayısı" değil, **modelin o pası açıklamak için ihtiyaç duyduğu
sürtünme katsayısıdır**. Uyarı bu ayrımı açıkça yazar.

## K-20 · Yoğunluk ve özgül ısı girdiye çıktı: sessiz bakır bağı kapatıldı
**Tarih:** 2026-09-15

`RHO = 8960` ve `CP = 385` kod içinde sabitti, ama `K` ve `n` serbestçe düzenlenebiliyordu.
Bu ikisi birlikte sessiz bir hata üretiyordu: biri alüminyum için `K = 140`, `n = 0,25`
girdiğinde çekme gerilmesi doğru çıkıyor, **kütle debisi, kWh/ton ve ΔT bakır sabitleriyle
hesaplanmaya devam ediyordu**. Ekranda hata görünmüyordu; sayı yanlış değil, *sessizce
yanlıştı*. Bu, yanlış sayıdan kötüdür — çünkü denetlenemez.

**Karar:** `rho` ve `cp` girdi alanı oldu (varsayılan bakır). Değer çekirdekten arayüze
kadar tek yoldan akar: `calcPass` → pas nesnesi → `massFlowOf` → özgül enerji ve ΔT.
Sınırlar `1000–20000 kg/m³` ve `100–2000 J/(kg·K)`; alüminyumdan tungstene kadar gerçek
malzemeleri kapsar, saçma değeri reddeder.

**Kök sebep, ilk denemede kaçtı ve tarayıcı yakaladı.** `HESAP_ALANLARI` listesi
taslak↔durum eşitlemesinin tek kaynağıdır; oraya eklemeyi unutunca bağlantıdan gelen
`rho` hesaba giriyor ama kutuda eski değer görünüyordu. Yani tam olarak K-16'nın
engellemek için var olduğu durum: ekrandaki girdi ile hesabın girdisi ayrışmıştı.
Ders: yeni bir hesap girdisi eklemek üç listeye birden dokunmayı gerektirir —
`ALANLAR` (çizim), `HESAP_ALANLARI` (eşitleme), `bekleyenDegisiklikler` (fark).

**Geri kalan bağ açıkça yazıldı.** `K`, `n` ve 70 MPa tabanı (M-1) hâlâ bakıra göredir.
Yoğunluk ya da özgül ısı varsayılandan ayrılınca yorum kutusu bunu söyler. Malzeme
kütüphanesi (açılır liste) bir sonraki adımdır; bu adım yalnızca **sessiz** olanı
**görünür** yaptı.

Üç test eklendi: yoğunluk iki katına çıkınca kütle debisi iki kat, özgül enerji ve ΔT
yarıya iner, güç değişmez; özgül ısı yalnızca sıcaklığı değiştirir; alüminyum
değerleriyle ΔT ve kWh/ton gerçekten değişir; sınır dışı değerler reddedilir ve
varsayılanlar eski sayıları birebir korur.

## K-21 · Kalıp çapına yuvarlama: ideal program ile depodaki program
**Tarih:** 2026-09-15

Araç `7,197 mm` gibi çaplar üretiyordu. Bu sayı doğru ama uygulanamaz: kalıplar
katalog adımlarında gelir. Kullanıcı ideal programı alıp elle yuvarlarsa emniyet
oranının nasıl değiştiğini göremez — oysa asıl soru odur.

**Karar:** Ekrandaki programın çapları seçilen kalıp adımına oturtulur ve manuel
program olarak girdilere alınır. Sonucu HESAPLA getirir (K-16).

**Uç çaplar yuvarlanmaz.** `d₀` hattın beslemesi, son çap ürünün kendisidir. Onları
adıma oturtmak, sorulan soruyu değiştirmek olur; araç cevabı değiştirmeli, soruyu
değil.

**Çakışma düzeltilmez, reddedilir.** Kaba bir adımda iki komşu çap aynı değere
düşebilir ya da sıra bozulabilir. Diziyi "en yakın geçerli hâle" itmek sessiz bir
tasarım kararı olurdu; bunun yerine hangi çiftin çakıştığı söylenir ve daha ince
adım istenir. Aracın her yerindeki kural burada da geçerli: sınırını bilen model,
sessizce saçmalayandan iyidir.

**Hangi çapın nereye gittiği tek tek yazılır.** Yuvarlama görünmez bir işlem
olmamalı; liste `7,197 → 7,200` biçiminde bütün kaymaları gösterir, en büyük sapma
ayrıca söylenir.

**Ölçülen sonuç:** genel amaçlı senaryoda 0,05 mm adımı on çapı kaydırıyor, en büyük
sapma 0,025 mm, ve en yüksek emniyet oranı 0,389'dan 0,410'a çıkıyor. Yuvarlama
ücretsiz değildir; bedeli sayıyla görünür.

Üç test eklendi: uç çaplar korunur ve ara çapların hepsi adımın tam katıdır, sapma
yarım adımı aşmaz, sıra korunur; yuvarlanmış dizi geçerli bir program kurar; kaba
adım reddedilir ve geçersiz girdiler ayrı durum kodlarıyla döner.
