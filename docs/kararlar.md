# Karar Kaydı

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

Yöntem değişiminde N korunur, çünkü manuel N'in varlık sebebi sabit kalıp sayısını
sabit tutup yöntemleri karşılaştırmaktır. Kullanıcı 13 girip yöntem değiştirdiğinde
13 kaybolursa alanın işlevi kalmaz.

## K-02 · Yayın: private çalışma deposu + temiz public depo
**Tarih:** 2026-09-03, 2026-09-04'te güncellendi

Geliştirme boyunca depo private tutuldu. Yayında ikiye ayrıldı: bütün geçmişi ve
çalışma notlarını taşıyan **private arşiv**, ve teslim edilen hâli taşıyan
**public depo** (`index.html`, `og.png`, `favicon.svg`, README, LICENSE ve dört
teknik doküman). Site public depodan Vercel ile yayınlanır; `og:url` ve
`og:image` mutlak adrestir, çünkü önizleme robotları göreli yolu okumaz.

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
- **E-posta:** bariskirli9@gmail.com
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
