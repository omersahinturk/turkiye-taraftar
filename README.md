# 🇹🇷 Türkiye Taraftar Koordinasyon

Stadyumdaki taraftarların **bestelerini (tezahürat) gerçek zamanlı koordine etmesi** için
tasarlanmış web uygulaması. Bir amigo besteyi yönetir; her tribün sırası geldiğinde kendi
sözlerini telefon ekranında büyük görür ve birlikte okur.

> Örnek etkinlik: **Türkiye vs Paraguay · World Cup 2026**

## Roller

- **👥 Taraftar** — Ad-soyad ve tribün (Batı / Güney / Doğu / Kuzey) seçerek katılır.
  Sırası gelince sözleri ekranda görür. Tüm besteleri "Besteler" kataloğundan ezberleyebilir.
- **🎤 Amigo** — Şifreyle giriş yapıp yönetici onayından sonra besteyi başlatır/yönetir,
  yeni beste ekleyip kendi bestelerini düzenler/siler.
- **🛡️ Yönetici** — Ayrı panelden (`admin.html`) amigo isteklerini onaylar; katılımcı ve
  tribün dağılımını izler.

## Beste türleri

- **🔁 Sırayla** — Batı → Güney → Doğu → Kuzey tribünleri sırayla okur, en sonda
  **Herkes (Birlikte)** finali ile herkes aynı sözü söyler.
- **🔴 Herkes Birlikte** — Tribün fark etmeksizin herkes aynı sözleri aynı anda, ekranda
  büyük görerek söyler.

## Özellikler

- Firebase ile **gerçek zamanlı** yayın (amigo → tüm taraftarlar anında).
- Sayfa yenilemede oturum korunur (taraftar tribününde, amigo panelinde kalır).
- Beste kataloğu (Sırayla / Herkes Birlikte olarak iki bölüm).
- QR kod ve link ile kolay paylaşım.
- Bir cihaz = bir kayıt (mükerrer katılımı önler).

## Teknoloji

- Statik **HTML + CSS + JavaScript** (derleme/bağımlılık yok).
- **Firebase Realtime Database** (canlı durum) + **Firebase Authentication**
  (taraftar/amigo için anonim, yönetici için e-posta/şifre).
- **GitHub Pages** üzerinde barınır.

## Dosyalar

| Dosya | Açıklama |
|-------|----------|
| `index.html` | Taraftar ve amigo uygulaması |
| `admin.html` | Yönetici paneli |
| `database.rules.json` | Firebase güvenlik kuralları |
| `GUVENLIK-KURULUM.md` | Firebase güvenlik/kurulum adımları |

## Kurulum

Güvenlik kurallarını ve Firebase Authentication yapılandırmasını kurmak için
[`GUVENLIK-KURULUM.md`](GUVENLIK-KURULUM.md) dosyasındaki adımları izleyin.
