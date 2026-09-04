# Doğrulama

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
| `epsPas` | 0,2218 | 0,2219 | %0,03 |
| `sigmaBar` | 196,7 MPa | 196,8 MPa | %0,05 |
| `sigmaFOut` | 265,5 MPa | 265,7 MPa | %0,06 |
| **`sigmaD`** | **77,6 MPa** | **77,6 MPa** | %0,02 |
| **`safety`** | **0,292** | **0,292** | %0,05 |
| **`delta`** | **2,52** | **2,52** | %0,00 |
| `dT` (adyabatik) | 22,5 °C | 22,5 °C | %0,00 |
| `A1` | 40,26 mm² | 40,26 mm² | %0,01 |
| `F` | 3123 N | 3125 N | %0,07 |
| `v1` | 4,99 m/s | 4,99 m/s | %0,07 |

Tolerans %0,5; en büyük sapma %0,07.

`sigmaFIn = 70,0 MPa` çıkıyor — `SIGMA_Y0` tabanı çalışıyor demektir (Hollomon
`ε = 0`'da sıfır verirdi).

---

## Altı test

| # | Test | Ne yapar | Geçme koşulu | Sonuç |
|---|---|---|---|---|
| T1 | Sıfır limiti | `μ = 0`, `α = 0,001°` ile referans pası hesaplar | `sigmaD` ≈ ideal iş (±%1) | 43,66 / 43,66 MPa ✔ |
| T2 | Alt sınır | Referans pası normal hesaplar | `sigmaD` > ideal iş | 77,61 > 43,66 ✔ |
| T3 | Teorik maksimum | `r = 0,70` olacak pas dener | `R_TEORIK_ASIM` hatası | hata verdi ✔ |
| T4 | Hacim korunumu | `A0·v0` ile `A1·v1` karşılaştırır | sapma < %0,1 | %0,0000 ✔ |
| T5 | Otomatik N — eşit gerinim | Üç senaryo, `equalStrain` + otomatik | 16 · 11 · 25 | 16 · 11 · 25 ✔ |
| T6 | Otomatik N — eşit emniyet | Üç senaryo, `equalSafety` + otomatik | 11 · 8 · 17 | 11 · 8 · 17 ✔ |

**T1'in referansı bağımsızdır:** `sigma_ideal = K · ε^(n+1) / (n+1)` kapalı formundan
gelir, kodun kendi `sigmaBar`'ından değil. Böylece hem Siebel'in sıfır limiti hem de
ortalama akma gerilmesi integrali sınanmış olur.

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
