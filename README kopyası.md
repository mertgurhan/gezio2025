# Sisteme Yönelik Potansiyel Güvenlik Açıkları ve Önleme Teknikleri
Kapsam ve Tehdit Modeli
Varlıklar (korunacak şeyler):
Kullanıcı hesapları (e-posta, kullanıcı adı), oturum token’ları
Favoriler, yorumlar, puanlamalar (kullanıcıya bağlı veriler)
Harita/3. parti API anahtarları
Uygulama kaynak kodu, deploy ayarları, environment secrets
AI planlayıcı (prompta girilen kişisel planlar/tercihler)
Muhtemel saldırgan profilleri:
Rastgele botlar (spam, brute force)
Kötü niyetli kullanıcı (XSS/IDOR denemeleri)
parti bağımlılık/supply-chain (CDN değişikliği)
Yanlış yapılandırma (public DB bucket, açık kurallar)
1) v1 (Statik Tek Dosya) Güvenlik Riskleri
1.1 XSS (Cross-Site Scripting) – DOM Tabanlı
Senaryo:
Arama kutusu, filtre, modal içeriği gibi alanları JS ile üretirken innerHTML kullanıp kullanıcı girdisini doğrudan basarsan:
Kullanıcı "<img src=x onerror=alert(1)>" gibi bir değerle sayfada script çalıştırabilir.
Etkisi:
Sayfa üzerinde çalışan JS üzerinden:
kullanıcı adına işlem yapma (v2’de kritik)
token çalma (v2’de oturum ele geçirme)
sahte içerik gösterme
Önleme (v1 + v2):
innerHTML yerine textContent kullan.
Template üretmen gerekiyorsa:
kullanıcıdan gelen alanları escape/sanitize et.
Content Security Policy (CSP) uygula:
Inline script’leri azalt, mümkünse “nonce” kullan (v2’de daha kolay).
Kullanıcıdan gelen data ile URL üretirken allowlist uygula.
1.2 CDN / Supply-Chain Riski (Tailwind CDN)
Senaryo:
CDN kaynağı değişir / saldırıya uğrar / DNS hijack vs. olursa istemciye kötü niyetli dosya servis edilebilir.
Etkisi:
Her ziyaretçi etkilenebilir (toplu kompromize).
Önleme:
CDN yerine yerel derlenmiş Tailwind (v2’de ideal).
CDN kalacaksa:
Sabit versiyon kullan
SRI (Subresource Integrity) ekle
CSP ile script-src/style-src kısıtla
1.3 Güvenlik Başlıkları Eksikliği (Security Headers)
Senaryo:
Default hosting ayarlarında güvenlik header’ları yoksa clickjacking, MIME sniffing gibi riskler artar.
Önleme:
En azından şunları ekle (host/Next.js config üzerinden):
Content-Security-Policy
X-Frame-Options: DENY (veya CSP frame-ancestors)
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Strict-Transport-Security (HSTS) (HTTPS varsa)
1.4 Açık Redirect / Güvensiz Linkleme
Senaryo:
?redirect= gibi parametreleri kontrol etmeden kullanıcıyı yönlendirirsen phishing’e yardım eder.
Önleme:
Redirect URL’lerini allowlist ile sınırla (sadece kendi domain’in).
2) v2 (Üyelik + DB + Favori/Yorum) ile Gelen Kritik Riskler
2.1 Kimlik Doğrulama Zafiyetleri (Auth)
Riskler:
Brute force, credential stuffing
Şifre sıfırlama akışında açıklar
Token sızıntısı (XSS ile birleşince felaket)
Önleme:
Firebase/Supabase Auth kullan → standart ve sağlam.
Rate limit: login/register/reset endpoint’lerine IP + kullanıcı bazlı limit.
Şifre sıfırlamada:
tek kullanımlık link, kısa süre, throttling
Oturum yönetimi:
Token’ları güvenli sakla (mümkünse HttpOnly cookie yaklaşımı)
Logout’ta refresh token iptali/rotasyonu
2.2 Yetkilendirme Hataları (Authorization) – IDOR / Broken Access Control
En kritik başlık bu.
Çünkü kullanıcı verisi geldiğinde “kim neyi görebilir/silebilir?” hataları en yaygın saldırı yüzeyi.
Senaryolar:
Kullanıcı A, favori kaydının id’sini değiştirip Kullanıcı B’nin verisini silebilir (IDOR).
Yorum silme endpoint’i “owner check” yapmazsa herkes herkesin yorumunu silebilir.
Önleme (Supabase):
RLS (Row Level Security) aç ve kural yaz:
SELECT/INSERT/UPDATE/DELETE hepsinde auth.uid() ile sahiplik kontrolü
Örnek mantık:
Favoriler tablosu: user_id = auth.uid()
Önleme (Firebase):
Firestore Rules:
request.auth.uid == resource.data.userId
Uygulama prensibi:
UI’de “buton gizlemek” yetmez → asıl kontrol DB/Server’da olmalı.
2.3 Veri Doğrulama Hataları (Input Validation)
Riskler:
XSS’ye zemin hazırlayan “kirli veri”
Beklenmedik tipler (string yerine obje vs.) ile logic bypass
Çok uzun input → maliyet/DoS
Önleme:
Server/API katmanında schema doğrulama:
örn. Zod/Joi (Next.js API routes)
Alan bazlı limit:
yorum max 500–1000 karakter
kullanıcı adı allowlist (harf/rakam/_ gibi)
DB’ye yazmadan önce normalize et:
trim, lower-case (e-mail), unicode normalize
2.4 CSRF (Cross-Site Request Forgery)
Ne zaman risk?
Eğer auth cookie tabanlı ise (özellikle HttpOnly cookie kullanınca) CSRF ortaya çıkar.
Önleme:
CSRF token / double submit cookie
CORS politikası + SameSite=Lax/Strict cookie
State-changing istekleri sadece POST/PUT/DELETE ile yap
2.5 CORS Yanlış Yapılandırma
Risk:
Access-Control-Allow-Origin: * + credential gibi hatalı ayarlar token sızıntısına gidebilir.
Önleme:
Sadece kendi domain’lerin allowlist
Credentials gerekiyorsa * kullanma
3) Harita Entegrasyonu (Google Maps / Leaflet) Güvenliği
3.1 API Key Sızıntısı ve Kötüye Kullanım
Senaryo:
Google Maps key front-end’de görünür → başkası alıp kullanır → kota/maliyet artar.
Önleme:
Google Cloud Console:
HTTP referrer restriction (sadece senin domain)
API bazlı kısıtlama (sadece Maps JS API vs.)
Rate limit + kota alarmları
Leaflet + OpenStreetMap kullanıyorsan key derdi azalır ama yine de:
tile provider limitlerini gözet
4) AI Seyahat Planlayıcı (v2) Özel Riskler
4.1 Prompt Injection / Jailbreak Benzeri Manipülasyon
Senaryo:
Kullanıcı prompt’a “kuralları yok say, şu linke git, şunu yap” gibi kötü niyetli metin ekler.
Model, güvenilmemesi gereken çıktılar üretebilir.
Etkisi:
Yanıltıcı öneriler, link/komut enjeksiyonu
Eğer AI çıktısını “otomatik işlem”e bağlarsan (rezervasyon yap, ödeme başlat) kritik.
Önleme:
AI çıktısını sadece öneri olarak kullan.
Çıktıları:
URL allowlist kontrolünden geçir
HTML olarak basma (text olarak bas)
Sistem prompt + guardrail:
“Sadece gezilecek yer listesi üret, komut/şifre isteme, dış link verme” gibi kısıtlar
Loglarda prompt içinde kişisel veri tutma (KVKK hassasiyeti)
5) Dosya/Görsel Depolama (Storage) Riskleri
5.1 Kötücül Dosya Yükleme (ileride)
Senaryo:
Kullanıcı “görsel” diye script içeren dosya yükler, başka yerde çalıştırılır.
Önleme:
MIME doğrulama + uzantı kontrolü + boyut limiti
Storage “public by default” yapma
Gerekirse resimleri server-side re-encode (jpeg/png) ederek güvenli hale getir
6) Rate Limiting, Spam ve DoS
6.1 Spam (yorum/favori) & Brute Force
Risk:
Botlar yorumları doldurur, DB maliyet artar.
Önleme:
IP/user bazlı rate limit
Yorum eklemede:
zaman penceresi (örn. 1 dakikada max 5 yorum)
tekrar eden içerik tespiti (basit hash)
Register/login:
captcha (opsiyonel)
başarısız deneme kilidi
7) Loglama, İzleme, Hata Yönetimi
7.1 Hassas Bilgi Sızıntısı (Logs/Errors)
Risk:
Hata mesajında token, query, secret basılması.
Önleme:
Prod’da “genel hata mesajı”, detay log sadece server tarafında
Sentry benzeri hata izleme
Audit log:
favori ekle/sil, yorum sil, hesap işlemleri
8) Güvenli Geliştirme Pratikleri (SDLC)
8.1 Secrets Yönetimi
Risk:
API key, service role key, DB URL repoya girer.
Önleme:
.env + .gitignore
GitHub secret scanning / pre-commit hook
Supabase “service_role” key’ini client’ta asla kullanma
8.2 Bağımlılık Güvenliği
Önleme:
npm audit / Dependabot
Sürüm pinleme, lockfile
README’ye Eklemek için Kısa “Kontrol Listesi” (İstersen aynen koy)
v1 için Minimum Güvenlik Checklist
 innerHTML yerine textContent
 CSP + temel security headers
 CDN için sabit versiyon + SRI
 Harici link/redirect allowlist
v2 için Kritik Güvenlik Checklist
 Auth: rate limit + güvenli oturum yönetimi
 DB: RLS (Supabase) / Rules (Firebase) zorunlu
 API: schema validation + input limitleri
 CSRF/CORS doğru yapılandırma
 Harita key restriction + kota alarmı
 AI: çıktılar text olarak bas, allowlist, otomatik işlem yok
 Logging: hassas veri maskleme + audit log
