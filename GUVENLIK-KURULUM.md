# 🔐 Güvenlik Kurulumu (Firebase)

Bu dosya, veritabanını dışarıya açık olmaktan çıkarmak için yapılması gereken
adımları içerir. Kod değişiklikleri tamamlandı; aşağıdaki Console adımları
**senin** tarafından yapılmalı.

## Neden?
Önceden veritabanı tamamen açıktı: kimlik doğrulaması olmadan herkes `curl` ile
tüm veriyi (amigo/admin şifreleri dahil) okuyabiliyordu. Yeni yapı:
- **Admin** → gerçek Firebase Auth hesabı (e-posta/şifre)
- **Taraftar + Amigo** → anonim kimlik (auth gerekli, token'sız erişim engellenir)
- **Şifreler** → istemciye hiç okunmaz; sunucu tarafı güvenlik kuralı doğrular
- `config` → tamamen kapalı

## Adımlar

### 1. Kimlik yöntemlerini aç
Firebase Console → **Authentication → Sign-in method**
- **Anonymous** → Enable
- **Email/Password** → Enable

### 2. Admin hesabını oluştur
Authentication → **Users → Add user**
- E-posta: `admin@turkiye-taraftar.web` (admin.html'deki `ADMIN_EMAIL` ile aynı olmalı)
- Şifre: güçlü, yeni bir şifre belirle
- Oluşturduktan sonra kullanıcının **UID**'sini kopyala.

### 3. Kuralları yükle
Realtime Database → **Rules** sekmesi
- `database.rules.json` içeriğini yapıştır.
- İçindeki **3 adet** `ADMIN_UID_BURAYA` ifadesini, adım 2'deki UID ile değiştir.
- **Publish.**

### 4. Şifreleri yenile (ifşa oldular!)
Realtime Database → **Data** → `config` düğümü
- `amigo_sifre` → yeni bir değer ver (eski `WC2026TR!` ifşa oldu).
- `admin_sifre` → artık kullanılmıyor (admin Firebase Auth ile giriyor); **silebilirsin**.

### 5. Test et
- `index.html` → Taraftar girişi: isim + bölge → çalışmalı.
- `index.html` → Amigo: doğru şifre ile onay bekleme ekranı; yanlış şifrede "Yanlış şifre".
- `admin.html` → adım 2'deki şifre ile giriş; amigo onaylama çalışmalı.
- Dışarıdan test: `curl https://turkiye-taraftar-default-rtdb.firebaseio.com/config.json`
  → artık `Permission denied` dönmeli.

## Not
Anonim kimlik, token'sız (rastgele internet) erişimi durdurur. `canli`/`besteler`
yazma yetkisi yalnızca **admin tarafından onaylanmış** amigolara verilir
(kural: `amigolar/{uid}/durum === 'onaylandi'`).
