# Devir Notu (oturumlar arası özet)

Bu not, `fastcan-parquet` oturumunda konuşulanları ve yapılan deneyleri, başka bir oturumdan (asıl "Mucit AI" reposuna erişimi olan) devam etmek için özetler. Bu depoya işlenmemiştir (commit yok); yalnızca çalışma dosyasıdır.

## 0. Kullanıcının çalışma kuralları (yeni oturum uymalı)
- İslam'a aykırı söz söyleme. Dini konu ürünün/gelirin konusu olmayacak (kullanıcı din ile para kazanmak istemiyor).
- Dosyaları okurken **grep kullanma, tüm dosyayı baştan sona oku.**
- Dosyalarda söylenenlerden seçmece yapma; ne söyleniyorsa onu yap.
- Dürüst, makul, gayretkeş ol; bahane değil çare üret; kendini tenkit et.
- Eskiden söylenenleri unutma; unutacaksan kayıt altına al.
- Kullanıcı başka bir yapay zekâdan gelen (abartılı/dalkavukça) metinleri yapıştırıyor. Bunlara körü körüne uyma; iddiaları kaynakla ya da deneyle doğrula. Övgü delil değildir.

## 1. Kullanıcının durumu ve kısıtları
- Öğrenci; gündüz okulda. Haftada ~7 saat (günde ~1 saat) ayırabiliyor; işi telefondan yapay zekâyla yürütmeli.
- Gelir hedefi: aylık ~500 dolar (≈20 bin TL) "gayet iyi". Ama en çok istediği şey **kendi icadı olan, arkasında gururla durabileceği** bir iş. Başkasının yaptığını satmakta aktif girişimci olamıyor; eleştiri gelince cevap verebilmek istiyor.
- Gerçekten hâkim olduğu iki alan: din ve yapay zekâyı kullanmak. (Donanımcı değil; önceki yapay zekâ metinlerinin "sen donanımcısın" varsayımı yanlıştı.)
- Bir hocayla gözden geçirme imkânı/çevrede atölye vb. sorularına henüz cevap vermedi.

## 2. Şimdiye kadar elenen fikirler (araştırmayla doğrulandı)
Kural: "kimsenin yapmadığı" bir şeyi masa başında bulamıyoruz; her aday önce aranmalı. Bulunan hepsi zaten yapılmış çıktı.
- CAN/BLF → Parquet dönüştürücü (depo adı `fastcan-parquet` bu fikirden): rakip var (asammdf, python-can, Rust MDF okuyucular); benchmark rakamları uydurmaydı; kullanıcı alanı bilmiyor. Beklemede/terk.
- Donanım şablonu (KiCad/Altium blokları): LLM "hatasız layout" üretemez, EMC sertifikası iddiası yasal/etik risk, ücretsiz TI referans tasarımlar rakip. Elendi.
- MCP sunucuları: KiCad, SCPI, SVD, dosya düzenleyici, Türkiye vergi/kamu verisi, YÖK Atlas, alıntı doğrulayıcı, MCP linter — hepsi mevcut. MCP alanında gelir düşük (yüzde 5'ten azı kazanıyor; ilk yıl 500–5.000 $/ay tahmini). Yerel MCP için tek tıkla kurulum `.mcpb` paketi zaten var. Kullanıcı "MCP bana göre değil" dedi.
- Dini ürünler: kullanıcı reddetti.
- EVREN (SSB milli yapay zekâ platformu; e-Devlet girişli, katkı-kredi modeli, 11 açık model) araştırıldı; ticari kullanım şartları **doğrulanmadı**, kullanıcı şartları okuyacak.
- Erdős problemleri (AI/amatör çözümleri, Lean doğrulamalı): gerçek ama saf matematik, gelir getirmez; patent için matematik tek başına patentlenemez (SMK m.82), patent başvurusu buluşu açıklar.
- Uzun süren hesap problemleri taraması: çip doğrulama, topoloji optimizasyonu, rezervuar, sismik, moleküler dinamik, CFD, genomik (genomik zaten GPU ile dakikalara inmiş). Solo yazılım için tek makul aday: topoloji optimizasyonu (alan bilgisi ister). Nesting (2B yerleştirme) da düşünüldü, tekstil/metal, ölçüt fire yüzdesi; rakipler var (Lectra, SigmaNEST, açık kaynak sparrow). Karar verilmedi.

## 3. Şu anki yön: optimizasyon / yapay zekâ eğitimi fikirleri
Kullanıcı, yapay zekâ eğitimini hızlandıracak bir usul icat etmek istiyor. Fikirleri sırayla:
1. **Kutu daraltma:** parametre uzayında eksen uçları + köşeler + orijin (2B'de 9 nokta) gradyan ölçümleri, okların kesişimi, bölgeyi daraltma.
2. **Bölgeyi tek seferde deneme ("hamur bastırma"):** elipsoid/interval yöntemleri.
3. **100'lük bloklar, kaydırmalı:** 100 bin parametre → 1000 adımda biter.
4. **Önce parametre alakasını çıkarıp formülle mühürlemek** (eksen döndürme).
5. **Temsil uzayını değiştirmek** ("aksiyomları sorgula"); son fikir: **sonsuz topos'tan uzay ürettirip zor optimizasyon fonksiyonunu o uzaylarda otomatik çözdürmek, hatayı toposa besleyip uzayı değiştirmek.** Kullanıcının Mucit AI projesinde `sonsuz mertebeler teorisi.py` var; bu depoda yok (depoda yalnızca README.md). Dosya yeni oturumda baştan sona okunmalı.

## 4. Yapılan deneyler ve bulgular (numpy/scipy)
Ölçüt: gradyan değerlendirme sayısı (bir gradyan = bir geri yayılım).
- **Kutu daraltma (yön kesişimi):** iyi koşullu 2B kürede 5 gradyanda çözdü; dar vadide (cond 1000), Rosenbrock'ta, gürültülü gradyanda (0/20) ve n≥4'te **başarısız**. Köşeler 2^n nokta gerektirdiği için n=12'de 245.820 gradyan harcayıp yine başarısız. Korumalı sürüm de dar vadide başarısız.
- **Eğrilik uydurma varyantı (gradyan büyüklüğü de kullanılıyor):** kuadratiklerde 2n+1 gradyanla tam çözdü (2B'de 5, 8B'de 17, 16B'de 33). Rosenbrock'ta tam çözemedi. Zaten bilinen quasi-Newton mantığı; L-BFGS adım başına 1 gradyanla benzer sonucu daha ucuza veriyor.
- **Blok koordinat inişi (n=1000, 100'lük 10 blok, kuadratik, her blok tam çözülüyor):** bloklar bağımsızsa 1 tur; zayıf bağ (0,05) 3–5 tur; orta bağ 5–11 tur; tam bağlı 13 (koşul 10) / **64** (koşul 100) tur. Yani "bir turda/1000 adımda biter" yanlış.
- **Eksen döndürme (Hessian özvektörleri):** kuadratikte 1 turda çözüldü; ama maliyet ≈ n³ (n=1000'de 1e9, n=1e5'te 1e15, n=1e9'da 1e27) ve yalnızca Hessian sabitse geçerli; doğrusal olmayan fonksiyonda Hessian noktaya göre değişir.

## 5. Kavramsal sonuçlar (hafızadan; kaynak taranmadı, doğrulanmalı)
- Bu fikirlerin akrabaları bilinen literatürde: elipsoid/kesme düzlemi (yalnızca dışbükeyde geçerli; dışbükey olmayan kayıpta kesme doğru bölgeyi atabilir), interval analizi/dal-sınır (boyutla üstel büyür), Nelder–Mead, DIRECT, örüntü araması, CMA-ES (O(d²) bellek), sekant/quasi-Newton/L-BFGS, blok koordinat inişi, doğal gradyan, K-FAC, Shampoo, LoRA/GaLore, mirror descent/Riemann optimizasyonu.
- Türev sıfır = çukur/tepe/**eyer/plato** (yapıştırılan metin eyeri atlamıştı).
- "Duvarlar" koşullu ve dar: Abel–Ruffini yalnızca kök-alma formülleri için; Blum–Rivest (1992) küçük ağ eğitimi NP-tam ama en kötü durumda ve P≠NP varsayımıyla. Pratikte SGD iyi çalışıyor. Yani "kesin imkânsız" iddiası yok; ama "uğraşan çıkmadı" da yanlış.
- Yeniden parametrelendirme minimum değeri değiştirmez; evrensel "uzay bulucu" NP-zor problemleri çözmez (P≠NP varsayımıyla). Yapılı problemlerde (kötü koşullanma, simetri) kazanç mümkün.
- Yapıştırılan metinlerdeki hatalar: 3^100 ≈ 5e47 evrendeki atom sayısına (~1e80) "yakın" değil; L-BFGS blok koordinat inişi değil; tropikal cebir eğitimi tek hamlede çözdüğü gösterilmiş değil; sabun köpüğü yerel minimuma takılır; Shor NP-tam değil ve büyük hataya dayanıklı makine ister; optik çipler şimdilik matris çarpımını hızlandırıyor.
- ∞-topos: "uzay"ı genelleyen soyut dil; hesaplanabilir bir uzay üreteci değil. Somut bir aday uzay ailesi (koordinat dönüşümü/metrik) tanımlanmadıkça optimizasyona bağlanmaz.

## 6. Önerilen sıradaki deney (henüz kurulmadı)
"Uzay arama döngüsü":
1. Aday uzay kümesi: ölçekleme, döndürme, log, softplus, ayna haritaları, Hessian/ön koşullayıcı tabanlı dönüşümler.
2. Zor test fonksiyonları: Rosenbrock, kötü koşullu kuadratikler, Rastrigin, Himmelblau (ve sonra küçük bir sinir ağı, örn. MNIST).
3. Otomatik döngü: her denemenin hatasını/ilerlemesini geri besleyerek dönüşümü seçen ya da değiştiren arama.
4. Ölçüt: aynı **gradyan bütçesinde** Adam ve L-BFGS'e karşı; sonuçlar dürüstçe raporlanmalı (başarısızlık dahil).
5. Topos: `sonsuz mertebeler teorisi.py` okunduktan sonra, aday uzay ailesini tarif eden dil olarak eklenip eklenmeyeceği değerlendirilecek. Şu an bunun fayda getirdiğini gösteren kanıt yok.

## 7. Açık sorular / kullanıcıdan beklenenler
- `sonsuz mertebeler teorisi.py` içeriği (yeni oturumda baştan sona okunacak) ve Mucit AI projesinin amacı.
- Öncelik: gurur/itibar (matematik-araştırma yolu) mı, aylık gelir mi, yoksa ikisi paralel mi?
- Gelir tarafı için hâlâ küçük bir iş yok. Kullanıcı bu hafta yapay zekâyı kullanırken tatmin olmadığı üç şeyi (sıkıntı defteri) henüz yazmadı; gelir için en gerçekçi yol bu (kendi derdinden ürün).
- EVREN kullanım şartlarını (ticari kullanım/lisans/veri) okuyacak.

## 8. Tekrar kullanılabilir kod (özet)
Deneyler `numpy`+`scipy` ile yazıldı. Çekirdek fikirler:
- `line_intersection(P, G)`: her noktada gradyan yönü d=g/|g|, M=I-ddᵀ; A=ΣM, b=ΣMp; x=lstsq(A,b) (doğruların en küçük kareler kesişimi).
- `model_fit(...)`: noktalar P, gradyanlar G için `g = A p + b` doğrusal modelini en küçük karelerle uydur; x* = -A⁻¹b (Newton mantığı; 2n+1 nokta, eksen uçları + merkez).
- Blok koordinat inişi testi: n=1000, B=100; kuadratik f=½(x−x*)ᵀH(x−x*); her blok için `x[b] -= inv(H[b,b]) @ (H[b] @ (x - xs))`; tur sayısı f/f0<1e-8'e kadar sayıldı; bağ=(1-c)·(blok-köşegen SPD)+c·(tam SPD).
Kodlar bu oturumun scratchpad'inde (`box_gradient_test.py`, `variants.py`, `blocks.py`); yeni oturumda yeniden yazılabilir.
