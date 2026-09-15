# Doğrulama

## 7 Eylül 2026 ek incelemesi

Yeni testler `node tests/audit.cjs` ile ağsız ve bağımlılıksız çalışır. Yerleşik
9 test; girdi/sonuç tutarlılığı, parametre duyarlılığı, URL hassasiyeti, manuel
çap dizisi, belirsizlik sınırları, kütle ve enerji korunumu ile tamamlandı.
Kademeli azalan otomatik program her pasta %20 sınırını sağlar (otomotiv: 20 pas).

Tarayıcıda hedef=9, ardından başlangıç=10 akışı sınandı: eski sürüm 1,38 hedefiyle
hesaplıyordu; yeni sürüm görünen 9 mm hedefini kullanır. Geçersiz girişte dışa
aktarma ve karşılaştırma sabitlemesi pasifleştirilir; bant açıkken nominal sonuç
ayrıca gösterilir. Eski 15.725 girdilik tarama aşağıdaki geçmiş kayıttır; bu
incelemede yeniden yapılmış gibi kabul edilmemelidir. Yeni tarama 500 girdidir.

Hesap çekirdeğinin sınandığı referans vakalar. Araçtaki **Doğrulama sekmesi** bu
testleri her açılışta canlı çalıştırır; bu dosya beklenen değerlerin kaydıdır.

Bütün sayılar `index.html` içindeki çekirdeğin Node ile koşturulmasıyla üretildi.

---

## Referans pas

Girdiler: `d0 = 8,00` · `d1 = 7,16` · `epsIn = 0` · `α = 8°` · `μ = 0,05` ·
`K = 450 MPa` · `n = 0,35` · `v0 = 4,0 m/s`

| Büyüklük | Beklenen | Hesaplanan | Sapma |
|---|---|---|---|
| `r` | 0,1990 | 0,1990 | %0,01 |
| `epsPas` | 0,2219 | 0,2219 | %0,00 |
| `sigmaBar` | 197,2 MPa | 197,2 MPa | %0,01 |
| `sigmaFOut` | 265,7 MPa | 265,7 MPa | %0,01 |
| **`sigmaD`** | **77,8 MPa** | **77,8 MPa** | %0,00 |
| **`safety`** | **0,293** | **0,293** | %0,03 |
| **`delta`** | **2,52** | **2,52** | %0,00 |
| `dT` (adyabatik) | 22,5 °C | 22,5 °C | %0,00 |
| `A1` | 40,26 mm² | 40,26 mm² | %0,01 |
| `F` | 3131 N | 3131 N | %0,01 |
| `v1` | 4,99 m/s | 4,99 m/s | %0,07 |

Tolerans %0,5; en büyük sapma %0,03.

> **8 Eylül 2026:** Bu değerler, 70 MPa tabanının ortalama akma integraline de
> uygulanmasıyla (K-15) %0,20 yukarı taşındı. Önceki kayıt: `sigmaD` 77,6 ·
> `safety` 0,292 · `F` 3123 N.

`sigmaFIn = 70,0 MPa` çıkıyor — `SIGMA_Y0` tabanı çalışıyor demektir (Hollomon
`ε = 0`'da sıfır verirdi).

---

## Dokuz test

| # | Test | Ne yapar | Geçme koşulu | Sonuç |
|---|---|---|---|---|
| T1 | Sıfır limiti | `μ = 0`, `α = 0,001°` ile referans pası hesaplar | `sigmaD` ≈ ideal iş (±%1) | 43,66 / 43,66 MPa ✔ |
| T2 | Alt sınır | Referans pası normal hesaplar | `sigmaD` > ideal iş | 77,61 > 43,66 ✔ |
| T3 | Teorik maksimum | `r = 0,70` olacak pas dener | `R_TEORIK_ASIM` hatası | hata verdi ✔ |
| T4 | Kütle korunumu | Raporlanan çap ve hızlardan her pasın kütle debisini hesaplar | sapma < %0,01 | %0,0000 (900,8 g/s) ✔ |
| T5 | Otomatik N — eşit gerinim | Üç senaryo, `equalStrain` + otomatik | 16 · 11 · 25 | 16 · 11 · 25 ✔ |
| T6 | Otomatik N — eşit emniyet | Üç senaryo, `equalSafety` + otomatik | 11 · 8 · 17 | 11 · 8 · 17 ✔ |
| T7 | Toplam iş / ideal iş | Hattın özgül enerjisini ideal şekil verme işine oranlar | 1,30 – 2,50 | 1,78 (56,4 → 100,5 kWh/ton) ✔ |
| T8 | K değişmezliğinin koşulu | Referans pası `K = 315` / `K = 530` ile üç bölgede hesaplar | A < 1e-9 · B > 1e-6 · C < 1e-9 | A 0,0 · B 2,0e-3 · C 0,0 ✔ |
| T9 | Sınır denetimi | Aralık dışı dört girdiyi ve geçerli tabanı `girdiDenetimi`'ne verir | 5 / 5 | 5 / 5 ✔ |

**T1'in referansı bağımsızdır:** `sigma_ideal = K · ε^(n+1) / (n+1)` kapalı formundan
gelir, kodun kendi `sigmaBar`'ından değil. Böylece hem Siebel'in sıfır limiti hem de
ortalama akma gerilmesi integrali sınanmış olur.

**T4 çıktı listesinden okur:** kütle debisi formülden değil, raporlanan çap ve hız
değerlerinden hesaplanır. Aksi hâlde test tanım gereği geçer ve hız zincirlemesindeki
bir hatayı kaçırırdı.

**T8 üç bölgeyi birden sınar.** 70 MPa tabanı hem ortalama akmada hem çıkış
akmasında geçerli olunca `K` sadeleşmesi koşullu hâle gelir:

| | Bölge | Beklenen |
|---|---|---|
| A | taban devre dışı (`ε_giriş ≥ ε*`) | sadeleşir |
| B | geçiş (`ε_giriş < ε* < ε_çıkış`) | **sadeleşmez** — `ε*` `K`'ya bağlı |
| C | pasın tamamı taban üzerinde | sadeleşir (iki taraf da 70 MPa) |

Yalnız A'ya bakan bir test, tabanı yalnızca paydaya uygulayan tutarsız bir
çekirdeği de geçerdi; testin değeri üçünü birden bağlamasındadır.

**T9 geçerli tabanı da sınar:** yalnızca reddedilenlere bakılsaydı "her şeyi reddet"
diyen bozuk bir denetim de testi geçerdi. Sınanan vakalar: geçerli taban (kabul),
negatif `μ`, `K = 0`, `n = 1,5`, hedef çap > başlangıç çapı (dördü de red).

### Girdi geçerlilik aralıkları

T9'un dayandığı sınırlar çekirdekte `GIRDI_SINIRLARI` altında tanımlıdır; arayüz ve
test aynı kaynağı okur (bkz. `kararlar.md` K-14).

Çekirdeğin kendi savunması (`CEKIRDEK_SINIRLARI`) bundan **bilerek daha geniştir**:
üst uçları buradan alır, ama alt uçları yalnızca fiziksel gerekliliktir. Sebebi
doğrulama testlerinin sınırları yoklamasıdır — T1 sıfır limitini `μ = 0` ve
`α = 0,001°` ile sınar; arayüzün kabul etmediği bu değerler çekirdekte geçerlidir.

| Alan | Aralık |
|---|---|
| `d0` | 0,1 – 50 mm |
| `dTarget` | 0,01 mm – `d0` (`d0`'a eşit olamaz) |
| `alphaDeg` | 1 – 20° |
| `mu` | 0,005 – 0,30 |
| `K` | 100 – 1000 MPa |
| `n` | 0,05 – 0,80 |
| `v0` | 0,1 – 20 m/s |

Aralık dışı girdiyle **hesap yapılmaz**: alanın altında sınırı söyleyen bir satır
çıkar, ekrandaki son geçerli hesap "güncel değil" uyarısıyla korunur. Kanonik kırmızı
vakanın `μ = 0,15`'i bu aralığın içindedir; sınır denetimi vakayı kırmaz.

---

## Otomatik pas sayısı — iki yöntemin farkı

| Senaryo | `equalStrain` | `equalSafety` | Kazanç |
|---|---|---|---|
| Otomotiv ince (8,00 → 1,38) | 16 pas | **11 pas** | 5 kalıp az |
| Genel amaçlı (8,00 → 2,50) | 11 pas | **8 pas** | 3 kalıp az |
| Çok ince (8,00 → 0,50) | 25 pas | **17 pas** | 8 kalıp az |

Eşit emniyet yönteminin verdiği programlarda emniyet oranı **bütün paslarda aynı**
(yayılma %0,00) ve hedefin hemen altında: 0,4952 · 0,4703 · 0,4977.

Az pas bedelsiz değildir — Otomotiv ince için 11 pas iki uyarı birden verir:
adyabatik ΔT 100,3 °C (eşik 100) ve delta 1,32 (alt sınır 1,5). Bu uyarılar
**bastırılmaz**; aracın işi kısıtları gizlemek değil ortaya çıkarmaktır.

---

## Kanonik kırmızı vaka

Eşik renklerini sınamak için elimizde her zaman kırmızı üreten bilinen bir senaryo olsun.

**Girdiler:** `d0 = 8,00` · `dHedef = 1,38` · **`μ = 0,15`** · `α = 8°` ·
`K = 450` · `n = 0,35` · `v0 = 2,0` · yöntem `equalStrain` · **pas sayısı elle 12**

**Beklenen:** 12 pasın **11'i kırmızı** (emniyet > 0,60), en yüksek emniyet
**0,690** (pas 12), ilk kırmızı satır **pas 2**, pas başına `r = %25,4`,
maks adyabatik ΔT 139,8 °C.

| pas | giriş → çıkış (mm) | emniyet | delta | ΔT (°C) |
|---|---|---|---|---|
| 1 | 8,000 → 6,910 | 0,519 | 1,91 | 44,1 |
| 2 | 6,910 → 5,969 | **0,631** | 1,91 | 68,2 |
| 3 | 5,969 → 5,156 | **0,656** | 1,91 | 81,8 |
| 6 | 3,847 → 3,323 | **0,679** | 1,91 | 108,0 |
| 9 | 2,479 → 2,141 | **0,687** | 1,91 | 125,8 |
| 12 | 1,598 → 1,380 | **0,690** | 1,91 | 139,8 |

(Ara satırlar atlandı; emniyet pas 2'den sonra 0,63–0,69 bandında düz seyreder.)

**Neden bu kombinasyon:** Yalnızca `μ = 0,15` yetmiyor — otomatik 16 pasla en yüksek
emniyet 0,543 çıkıyor, yani sarı. Kırmızıyı görmek için pas sayısının da 12'ye
düşürülmesi gerekiyor. Sınırdaki komşu değerler: `μ = 0,15` + 14 pas → 0,606 (kırmızı,
sınırda), `μ = 0,10` + 12 pas → 0,587 (sarı).

Faz 3'te eşik renkleri bu vakayla sınanacak.

---

## Diğer sınamalar

| Durum | Beklenen | Sonuç |
|---|---|---|
| `8,00 → 4,50` tek pas | `R_TEORIK_ASIM` (%68,4) | ✔ hata, sınır %63,2 |
| `8,00 → 1,38`, N = 2 | 1. pasta %82,8 → hata | ✔ mesaj pas numarasını söylüyor |
| N = 0 / N = 3,5 | `GECERSIZ_PAS_SAYISI` | ✔ |
| Hedef çap ≥ başlangıç | `HEDEF_CAP_BUYUK` | ✔ |
| Tavlama pas 8'de | sonraki pasların dayanımı düşer | ✔ `sigmaFOut` 548 → 265 MPa |
| Manuel `8,00 7,16 6,40` | ondalık virgül okunur | ✔ |
| Manuel `8.00,7.16,6.40` | virgül ayırıcı okunur | ✔ |
| Manuel `8,00,7,16` | belirsiz → `MANUEL_BELIRSIZ` | ✔ |
| Manuel artan dizi | hangi değerin hatalı olduğu söylenir | ✔ |
| Eşit emniyet yayılması (N = 13…60) | paslar arası fark ≈ 0 | ✔ %0,00 |

## Arayüz tutarlılığı (K-14)

Ekranda birbirine ait olmayan sayıların yan yana durmaması, hesap kadar sınanır.

| Durum | Beklenen | Sonuç |
|---|---|---|
| Aralık dışı girdi (negatif `μ`) | hesap yapılmaz, alan sınırı yazılır, son geçerli hesap "güncel değil" uyarısıyla korunur | ✔ |
| Okunamayan metin (`abc`, yarım ondalık) | aynı davranış, sessizce yutulmaz | ✔ |
| Hesap hatası (`N = 2`) | on beş panelin tamamı birlikte boşalır, bayat etiket kalmaz | ✔ 0 bayat |
| Kalıp kesiti kaydırıcısı + girdi değişimi | kesit pasın kendi değerlerine döner, tabloyla birebir aynı | ✔ 0,305 / 0,305 |
| Kaydırıcı + senaryo değişimi | aynı | ✔ 0,279 / 0,279 |
| Hata durumunda dışa aktarma | CSV / yazdır / bağlantı düğmeleri kapanır | ✔ |
| Dil değişimi | doğrulama paneli iki dilde de 9/9 yazar | ✔ |
| 360 px, dört durum | yatay kaydırma yok | ✔ |

## Rapor üretimi

Teknik inceleme raporu elle yazılmaz, **koddan üretilir**:

```
node tools/rapor-veri.cjs   # sayıları index.html'in canlı kodundan JSON olarak çıkarır
python tools/rapor.py       # testleri koşar ve output/pdf/tel-cekme-inceleme.pdf yazar
```

`rapor.py` içinde tek bir mühendislik sayısı elle yazılı değildir: referans pas değerleri,
kanonik kırmızı vaka, senaryo özetleri, taban payı tablosu, K değişmezliğinin üç bölgesi ve
test sonuçlarının tamamı üretim anında yeniden hesaplanır. Veri çıkarma ya da test koşumu
düşerse rapor **hiç üretilmez** — bayat sayı basmaktansa çıktı olmaması yeğlenir. Bu, bu
projede tekrar tekrar görülen belge–kod kaymasına karşı yapısal önlemdir.

**Çekirdek taraması:** beyan edilen geçerlilik aralığı içinde 15.725 rastgele girdi
kümesi, toplam 276.153 pas hesaplandı — çökme, `NaN`/`Inf` ve negatif emniyet yok.
445 girdi temiz `DrawingError` ile reddedildi (`R_TEORIK_ASIM` 376,
`PAS_SAYISI_YAKINSAMADI` 69), yani hata kutusuna düşen bilinen sınır durumları.
