# Kaynaklar

Araçtaki her formülün nereden geldiği. Formüller sekmesinde de aynı eşleme
kısaltılmış hâliyle görünür.

## Kitaplar ve makaleler

| Kısaltma | Kaynak |
|---|---|
| **Dieter** | G. E. Dieter, *Mechanical Metallurgy*, 3. baskı, McGraw-Hill — şekil verme (plastic forming) ve tel çekme bölümleri; pekleşme (Hollomon) ve akma eğrisi bölümü |
| **Kalpakjian & Schmid** | S. Kalpakjian, S. R. Schmid, *Manufacturing Engineering and Technology* — çekme (drawing) bölümü: kalıp geometrisi, proses parametreleri, ısınma |
| **Avitzur** | B. Avitzur, *Metal Forming: Processes and Analysis*, McGraw-Hill — şekil değiştirme bölgesi geometrisi, merkezi çatlak (centre burst) |
| **Wistreich** | J. G. Wistreich, "The Fundamentals of Wire Drawing", *Metallurgical Reviews*, 3(1), 1958 — delta faktörü, sürtünme ve çekme gerilmesi ölçümleri |

## Formül eşlemesi

| # | Formül | Kaynak |
|---|---|---|
| 1 | `r = 1 − (d₁/d₀)²` | Dieter, tel çekme bölümü — kesit azalma tanımı |
| 2 | `ε = 2·ln(d₀/d₁)` | Dieter — gerçek gerinim, hacim korunumundan |
| 3 | `σ_f = max(K·εⁿ, 70 MPa)` | Dieter — Hollomon bağıntısı ve pekleşme katsayıları tablosu. 70 MPa tabanı bu aracın modelleme tercihidir (tavlanmış ETP bakır akma gerilmesi mertebesi), kaynakta yoktur. |
| 4 | `σ̄ = K·(ε_out^(n+1) − ε_in^(n+1)) / ((n+1)·ε_pas)` | Dieter — Hollomon'un pas boyunca integrali (ortalama akma gerilmesi) |
| 5 | `σ_d = σ̄·[(1 + μ/α)·ε_pas + (2/3)·α]` | **Siebel yaklaşımı**; Dieter ve Kalpakjian & Schmid'de bu biçimiyle verilir. Üç terim: şekil verme, sürtünme, fazlalık (redundant) iş. |
| 6 | `emniyet = σ_d / σ_f,çıkış` | Dieter (çekme gerilmesinin akma gerilmesini aşamaması); pratik 0,60 sınırı Wistreich'in ölçümleriyle uyumludur |
| 7 | `Δ = α·(1 + √(1−r))² / r` | Wistreich (1958) ve Avitzur — şekil değiştirme bölgesi biçim oranı; Δ < 1,5 aşırı sürtünme, Δ > 3,0 merkezi çatlak riski |
| 8 | `ΔT = σ_d / (ρ·c_p)` | Kalpakjian & Schmid — adyabatik sıcaklık artışı (bütün işin ısıya döndüğü kabulü) |

## Sabitler

| Sabit | Değer | Kaynak / gerekçe |
|---|---|---|
| `RHO` | 8960 kg/m³ | Bakırın yoğunluğu (standart değer) |
| `CP` | 385 J/(kg·K) | Bakırın özgül ısısı (standart değer) |
| `SIGMA_Y0` | 70 MPa | Tavlanmış ETP bakır akma gerilmesi mertebesi — modelleme tercihi, `kabuller.md` M-1 |
| `K`, `n` | 450 MPa, 0,35 | Dieter'in pekleşme katsayıları tablosunda ETP bakır için verilen aralığın (K = 315–530 MPa, n = 0,35–0,54) içinden seçilmiş varsayılan. Tek doğru değer yoktur; bu yüzden araçta belirsizlik bandı vardır. |
| `MAX_R_PER_PASS` | 0,20 | Sektörde yaygın baş parmak kuralı (pas başına maks %20 kesit azalma). Fiziksel sınır değildir; teorik sınır `1 − 1/e = 0,632`'dir ve ayrıca kontrol edilir. |
| `TAPER_RATIO` | 0,65 | Kademeli azalanda son pasın ilk pasa gerinim oranı — bu aracın tercihi, gerçek hatlarda azaltmanın sona doğru hafifletilmesi pratiğinden |
| `TARGET_SAFETY` | 0,50 | Eşit emniyet yönteminin hedefi; 0,60 kopma sınırının altında pay bırakır |

## Not

Aracın verdiği sayılar bu kaynaklardaki bağıntıların doğrudan uygulanmasıdır;
kaynaklarda geçmeyen tek şeyler `SIGMA_Y0` tabanı, `TAPER_RATIO` ve
`MAX_R_PER_PASS` kabulleridir. Üçü de `docs/kabuller.md` içinde ayrıca yazılıdır.
