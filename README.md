# Tel Çekme Pas Programı Hesaplayıcısı

Bakır tel çekmede kopma riskini belirleyen şeyin pekleşme katsayısı değil **sürtünme**
olduğunu gösteren; pas programını kuran ve sonucun belirsizliğini onunla birlikte
raporlayan bir ön tasarım aracı.

Bulgu şu: emniyet oranı `σ_d / σ_f` ifadesinde pekleşme katsayısı `K` sadeleşir
(iki terim de `K` ile doğrusaldır). Literatürdeki en geniş belirsizlik ETP bakırın
`K` değerindedir — 315–530 MPa, yani ±%26 — ama bu belirsizlik kopma riskini
**hiç** etkilemez; yalnızca kuvveti, gücü ve sıcaklık artışını oranlı biçimde
ölçekler. Riski belirleyen, çoğu hesapta sabit varsayılan sürtünme katsayısıdır.
Araç bunu köşe taraması tablosunda doğrudan gösterir: `μ` 0,03'ten 0,08'e çıkınca
aynı program 0,451'den 0,571'e, yani güvenli bölgeden sınırın üstüne geçer.

**[Aracı aç →](https://tel-cekme.vercel.app/)** · tek HTML
dosyası, sıfır bağımlılık, çevrimdışı çalışır.

![Ana grafik: malzeme dayanımı ve çekme gerilmesi eğrileri, aradaki emniyet payı](og.png)

## Ne hesaplar

8 mm filmaşinden hedef çapa inen bir çekme hattı için pas programı kurar ve her
pasta şunları verir: kesit azalma, birikmiş gerinim, ortalama akma gerilmesi,
çıkış dayanımı, çekme gerilmesi, **emniyet oranı**, delta faktörü, adyabatik
sıcaklık artışı, çıkış hızı ve mekanik güç.

Pas dağıtım yöntemleri:

| Yöntem | Ne yapar |
|---|---|
| Eşit gerinim | Her pasta aynı oransal kesit azalması — kitap standardı, karşılaştırma tabanı |
| Kademeli azalan | Başta ağır, sonda hafif — fabrikaların gerçekte yaptığı |
| Eşit emniyet | Her pasın emniyet oranını hedefe (0,50) eşitler; otomatik modda **hedefi tutan en küçük pas sayısını** bulur |
| Manuel | Kendi çap dizinizi yapıştırın |

Otomotiv ince senaryosunda (8,00 → 1,38 mm) eşit gerinim 16 pas ister; eşit emniyet
aynı güvenlik seviyesini **11 pasla** tutturur — 5 kalıp az, %7 daha az enerji.
Bedeli de görünür: 11 pasta delta 1,32'ye (aşırı sürtünme sınırı) ve adyabatik ΔT
100,3 °C'ye çıkar, araç ikisini de uyarı olarak gösterir ve gereken kalıp açısını
(9,1°) söyler.

## Yaklaşım

- **Çekme gerilmesi:** Siebel yaklaşımı — şekil verme + sürtünme + fazlalık iş
- **Pekleşme:** Hollomon (`σ = K·εⁿ`), birikmiş gerinim taşınır, ara tavlamada sıfırlanır
- **Hız:** hat boyunca zincirlenir, kütle debisi sabittir (`ṁ = 900,8 g/s`, çıkış 67,2 m/s)
- **Belirsizlik:** `K`, `n`, `μ` için alt/üst sınır, sekiz köşe taraması; çap dizisi
  sabit tutulur, böylece bant malzeme belirsizliğini gösterir, program değişimini değil
- **Duyarlılık:** iki ayrı tornado — emniyet için `r · μ · n · α` (K sıfır çıkar,
  grafiğin kendi doğrulaması), özgül enerji için `K · μ · pas sayısı · α`

## Doğrulama

Sekiz test sayfa her açıldığında canlı çalışır:

| # | Test | Beklenen |
|---|---|---|
| T1 | Sıfır limiti | `σ_d` ≈ ideal iş (±%1) |
| T2 | Alt sınır | `σ_d` > ideal iş |
| T3 | Teorik maksimum | `r > 0,632` reddedilmeli |
| T4 | Kütle korunumu | raporlanan çap ve hızlardan kütle debisi sabit |
| T5 | Otomatik pas sayısı — eşit gerinim | 16 · 11 · 25 |
| T6 | Otomatik pas sayısı — eşit emniyet | 11 · 8 · 17 |
| T7 | Toplam iş / ideal iş | 1,30 – 2,50 (bulunan 1,78) |
| T8 | K değişmezliği | emniyet oranı `K` ile değişmemeli |

Referans pas (`d0 = 8,00` → `d1 = 7,16`, `α = 8°`, `μ = 0,05`, `K = 450`, `n = 0,35`):
`σ_d = 77,6 MPa`, emniyet `0,292`, delta `2,52`, `ΔT = 22,5 °C`. En büyük sapma %0,07.
Ayrıntı: [`docs/dogrulama.md`](docs/dogrulama.md).

## Kabuller ve sınırlar

Bu bir **ön tasarım** aracıdır, üretim reçetesi değildir. Kalıp esnemesi, yağ filmi
rejimi, geri gerilim, şekil değiştirme hızı, sıcaklığın akmaya geri beslenmesi, kalıp
aşınması ve artık gerilmeler hesaba katılmaz. Tablodaki güç kalıplarda harcanan
mekanik güçtür; şebeke gücü aktarma ve motor kayıpları nedeniyle %15–25 daha yüksektir.
Tamamı: [`docs/kabuller.md`](docs/kabuller.md).

## Kaynaklar

Dieter (*Mechanical Metallurgy*) · Kalpakjian & Schmid (*Manufacturing Engineering and
Technology*) · Avitzur (*Metal Forming*) · Wistreich (*The Fundamentals of Wire Drawing*,
1958). Formül–kaynak eşlemesi: [`docs/kaynaklar.md`](docs/kaynaklar.md).

## Nasıl çalıştırılır

`index.html` dosyasını indirip çift tıklayın. Kurulum, sunucu veya internet bağlantısı
gerekmez; tek istisna yazı tiplerinin çevrimiçi yüklenmesidir, o da olmazsa sistem
yazı tipleriyle açılır.

Arayüz Türkçe ve İngilizce. Girdiler adres çubuğunda taşınır, yani bir senaryoyu
bağlantı olarak paylaşabilirsiniz. Pas tablosu CSV olarak indirilir; yazdırma çıktısı
iki sayfadır (mühendislik özeti + belirsizlik, doğrulama ve kabuller).

## Depo yapısı

```
index.html      aracın tamamı — tek dosya
og.png          paylaşım önizleme görseli
favicon.svg
docs/
  dogrulama.md  referans hesap, testler, kanonik kırmızı vaka
  kabuller.md   modelin sınırları, madde madde
  kaynaklar.md  formül–kaynak eşlemesi
  kararlar.md   tasarım kararları ve gerekçeleri
```

---

Barış Kırlı — Makine Mühendisi · [bariskirli9@gmail.com](mailto:bariskirli9@gmail.com) ·
[linkedin.com/in/bariskirli](https://www.linkedin.com/in/bariskirli)

Bağımsız bir mühendislik çalışmasıdır, herhangi bir şirketle resmî ilişkisi yoktur.
MIT lisansı.
