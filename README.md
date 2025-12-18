# 🌍 Gezio - Türkiye Keşif Rehberi (v1 MVP)

Gezio, kullanıcıların Türkiye'deki şehirleri, gezilecek yerleri ve restoranları keşfetmelerini sağlayan web tabanlı bir seyahat rehberi uygulamasıdır.Bu proje için artırımlı model tercih edilmiştir.Kullanıcılardan gelen feedbackler doğrultusunda websitesi güncellenecektir.

## 🚀 Proje Durumu: v1 (Tamamlandı)

Bu sürüm **"Single File" (Tek Dosya HTML)** yapısı ile geliştirilmiş olup, temel gösterim ve listeleme fonksiyonlarına odaklanmıştır.

### ✅ v1 Özellikleri
* **Şehir Seçimi:** 81 il listelenmiştir (Şu an için *İstanbul* ve *Muğla* aktif, diğerleri "Yakında" modundadır).
* **Kategorili Listeleme:**
    * 🏛️ **Gezilecek Yerler:** Müzeler, tarihi yapılar, plajlar vb.
    * 🍽️ **Restoranlar:** Yeme-içme mekanları, kafeler.
* **Detay Görünümü:** Seçilen yer hakkında detaylı bilgi, görsel ve puanlama modalı.
* **Arama ve Filtreleme:** Şehir bazlı içerik arama ve kategoriye göre süzme.

---

## 📅 Yol Haritası (Roadmap) - v2

Projenin bir sonraki aşamasında, tek dosya yapısından kaynaklanan teknik limitleri aşmak için modern bir framework'e (Next.js/React) geçiş yapılacaktır.

### 🚧 v2 İçin Planlanan Özellikler
1.  **Favorilere Ekleme Sistemi:**
    * *Not:* v1 sürümünde kod yapısının tek bir HTML dosyasında toplanması ve satır sayısının artmasıyla oluşan yönetim zorlukları (maintainability issues) nedeniyle, "Favorilere Ekle" özelliği stabiliteyi bozmamak adına **v2 sürümüne ertelenmiştir.**
2.  **Kullanıcı Üyeliği ve Veritabanı:** Firebase/Supabase entegrasyonu ile gerçek kullanıcı kayıtları.
3.  **Yapay Zeka Seyahat Planlayıcı:** Seçilen gün sayısına göre otomatik rota oluşturma.
4.  **Harita Entegrasyonu:** Google Maps veya Leaflet ile gerçek konum gösterimi.
5.  **Tema ve inglizce dil seçeneği**
6.  **Favoriye ekleme ve görüntüleme**
7.  **sözel rehber tutma rehber seçebileceksin**
8.  **Geri kalan şehirler aktifleştirilecek**

## 🛠️ Kullanılan Teknolojiler (v1)
* **HTML5 & CSS3**
* **JavaScript (Vanilla JS)**
* **Tailwind CSS (CDN)** - Hızlı UI tasarımı için.
