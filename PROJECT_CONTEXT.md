# PROJECT_CONTEXT.md — Türkiye Taraftar Koordinasyon

> Gün sonu özeti · 17 Haziran 2026

---

## Projenin Amacı

Stadyumdaki taraftarların tezahüratları (besteleri) gerçek zamanlı koordine etmesi için tasarlanmış tek sayfalı bir web uygulaması. Hedef etkinlik: **Türkiye vs Paraguay · World Cup 2026**.

Bir amigo besteyi yönetir; taraftarlar sırası gelince kendi sözlerini telefon ekranında büyük görür ve birlikte okur.

---

## Teknoloji

| Katman | Araç |
|--------|------|
| Frontend | Saf HTML + CSS + JS (derleme yok, bağımlılık yok) |
| Realtime DB | Firebase Realtime Database |
| Auth | Firebase Authentication (anonim + e-posta/şifre) |
| Hosting | GitHub Pages |
| QR | qrcodejs CDN |

---

## Dosya Yapısı

```
index.html          → Taraftar + Amigo uygulaması (tek dosya, ~1460 satır)
admin.html          → Yönetici paneli (amigo onay, katılımcı listesi)
database.rules.json → Firebase güvenlik kuralları
README.md           → Kullanıcıya yönelik kısa açıklama
GUVENLIK-KURULUM.md → Firebase kurulum adımları
PROJECT_CONTEXT.md  → Bu dosya
```

---

## Roller ve Akışlar

### 👥 Taraftar
1. Ad + soyad gir → isim çakışma kontrolü (`isimler/` node)
2. Tribün seç (Batı / Güney / Doğu / Kuzey)
3. Canlı ekranda bekle → amigo yayın başlatınca sözler görünür
4. Sayfa yenilemede oturum korunur (`localStorage: tt_oturum`)

### 🎤 Amigo
1. Ad + soyad → şifre gir (`config/amigo_sifre` ile Firebase kural sunucusunda karşılaştırılır)
2. Yönetici onayı bekle (`amigolar/$uid/durum: 'bekliyor' → 'onaylandi'`)
3. Besteyi seç → "Sırayla Başlat" veya "Herkes Birlikte Başlat"
4. Tribünler arası "Sıradaki →" ile ilerle, "Durdur" ile bitir
5. Kendi bestelerini ekle / düzenle / sil

### 🛡️ Yönetici (admin.html)
- Firebase e-posta/şifre ile giriş (anonim değil)
- Bekleyen amigo isteklerini onaylar/reddeder
- Katılımcı listesi ve tribün dağılımını görür
- Admin UID: `fys1pLjvIKg85t4CHlMDaX6zf303` (database.rules.json'da hardcoded)

---

## Beste Türleri

| Tür | Davranış |
|-----|----------|
| 🔁 Sırayla | Batı → Güney → Doğu → Kuzey → Herkes (Birlikte) finali |
| 🔴 Herkes Birlikte | Tüm tribünler aynı anda aynı sözü görür |

Sıralı bestede 5. adım (Herkes finalesi) ayrı bir section: `sozler[4]`.  
Herkes türünde tüm sözler tek kısımda: `sozler[0]`.

---

## Firebase Veri Yapısı

```
/canli
  caliyor: boolean          → yayın aktif mi
  aktifBeste: string        → aktif beste id'si
  aktifBolgeIdx: number     → sıralıda hangi tribün sırası
  herkesAktif: boolean      → herkes birlikte modunda mı
  herkesTumu: boolean       → standalone herkes mi, finale mi
  siralamaBolgeIdx: [0,1,2,3] → tribün sırası
  baslayanAmigo: string|null  → yayını başlatan amigo UID (kilit mekanizması)
  besteler: [...]            → o anki besteler snapshot'ı (taraftar için)
  ts: number                 → başlama timestamp'i (geri sayım için)

/besteler/$besteId
  id, isim, tur, meta, sozler[][], sahip (uid), sahipAd

/amigolar/$uid
  isim, soyisim, durum ('bekliyor'|'onaylandi'|'reddedildi')
  amigoSifre (kural sunucuda karşılaştırır, clientta okunmaz)

/katilimcilar/$uid
  isim, soyisim, bolge, bolgeIsim, girisSaati, rol

/isimler/$isimAnahtar
  uid, rol     → çift kayıt engellemek için (isimAnahtar: isim__soyisim normalize)

/config/amigo_sifre     → amigo girişte kural tarafında kontrol edilir
```

---

## Kritik State (`let state = {...}` — index.html)

```javascript
state = {
  uid,                  // Firebase anonymous uid
  mod,                  // aktif ekran: 'hosgeldin'|'taraftar'|'amigo'|...
  kullaniciIsim/Soyisim,
  taraftarBolge,        // 'bati'|'guney'|'dogu'|'kuzey'
  amigo,                // boolean
  besteler,             // güncel beste listesi
  aktifBeste,           // id
  aktifBolgeIdx,        // -1: başlamadı
  herkesAktif,          // herkes birlikte modu
  herkesTumu,           // standalone herkes (vs finale)
  caliyor,              // boolean — yayın aktif
  baslayanAmigo,        // uid|null — amigo kilidi için
  siralamaBolgeIdx,     // [0,1,2,3] — tribün sırası
}
```

---

## Amigo Kilidi Mekanizması

**Sorun:** Birden fazla amigo aynı anda yayın başlatabiliyordu.

**Çözüm (bugün eklendi):**

- `canli/baslayanAmigo` → yayını başlatan amigonun UID'si; yayın biterken `null`
- `amigoBaslat()` / `herkesBaslat()`: `if (state.caliyor && state.baslayanAmigo !== state.uid) return;`
- `amigoDurdur()`: `if (state.baslayanAmigo && state.baslayanAmigo !== state.uid) return;`
- UI: `benimBaslattim = !state.baslayanAmigo || state.baslayanAmigo === state.uid` → başkası yayındayken "🎤 Başka amigo yayında — durdurması bekleniyor" mesajı
- `state.baslayanAmigo` ve `state.caliyor` artık `canli` dinleyicisinde **her modda** (taraftar, amigo, besteler, qr...) güncelleniyor — stale state sorunu giderildi
- `gidAmigo()` girişinde `.get()` ile anlık okuma yapılıyor (onay-bekle → amigo geçişi için)

---

## Edit/Sil Yetki Kontrolü

```javascript
const benim = b.sahip && b.sahip === state.uid;
const duzenlenebilir = !state.caliyor && !state.baslayanAmigo && benim;
```

- `benim`: Firebase'de kayıtlı `sahip` alanı == kendi UID'i
- `!state.baslayanAmigo`: herhangi bir yayın devam ediyorsa gizli (çift güvence)
- Firebase kural tarafında da enforce: `data.child('sahip').val() === auth.uid`

---

## Önemli Fonksiyonlar

| Fonksiyon | Ne yapar |
|-----------|----------|
| `abonelikleriBaslat()` | Firebase `canli`, `besteler` listener'larını kurar |
| `amigoDurumYayinla()` | `canli` node'una tüm yayın state'ini yazar |
| `amigoEkraniGuncelle()` | Amigo panelini state'e göre render eder |
| `taraftarEkraniGuncelle()` | Taraftar ekranını render eder (sıra, sözler) |
| `besteleriDiziyeCevir()` | Firebase map → JS dizi (eski dizi formatını da tolere eder) |
| `isimAnahtar()` | İsim+soyisim → normalize key (çift kayıt için) |
| `bildirimGoster()` | Beste başlayınca taraftara overlay + geri sayım + ses |
| `gidAmigo()` | Amigo paneline geçer, canli'den `.get()` ile state tazeler |
| `oturumGeriYukle()` | Sayfa açılışında localStorage'dan oturumu kurtarır |

---

## Bilinen Kısıtlar

1. **Aynı tarayıcı testi:** Firebase anonymous auth UID tarayıcı başına sabittir. Aynı tarayıcıda iki amigo `oturumKapat()` → yeniden giriş yapsa bile aynı UID alır (Firebase localStorage cache). Gerçek kullanımda (farklı cihaz/tarayıcı) sorun yok.

2. **Admin UID hardcoded:** `database.rules.json` içinde admin UID sabittir (`fys1pLjvIKg85t4CHlMDaX6zf303`). Farklı Firebase projesi için güncellenmeli.

3. **Tek amigo kilidi:** Yalnızca bir amigo aynı anda yayın yapabilir. Bu tasarım gereği.

4. **`canli/besteler` snapshot:** Amigo yayın başlatırken `state.besteler`'i `canli`'ye de yazar (taraftarların offline bestesiz kalmaması için). Bu snapshot `besteleriDiziyeCevir` geçmez; array olarak gelir.

---

## Bugün Yapılan İşler

1. **Amigo kilidi eklendi** (`f8421d8`): `baslayanAmigo` alanı Firebase'e eklendi, başlatma/durdurma fonksiyonları kilitlendi, UI blok mesajı eklendi.

2. **Kilit hataları düzeltildi** (`92d7d54`):
   - `state.caliyor` tüm modlarda güncellenecek şekilde listener refactor edildi
   - `gidAmigo()` girişinde anlık Firebase okuma eklendi
   - Edit/sil koşuluna `!state.baslayanAmigo` eklendi

3. **Otomatik test:** İki bağımsız Playwright context ile 5 senaryo test edildi — tümü PASS.

---

## Geliştirme Notları

- `index.html` tek dosyadır; CSS, HTML ve JS aynı dosyada.
- Firebase SDK compat versiyonu (v10.12.0) kullanılıyor.
- GitHub Pages'te deploy edilir; `main` branch = production.
- Yeni özellik eklenirken `amigoDurumYayinla()` payload'ına alan eklemeyi ve dinleyicilerde `else if (state.mod === 'amigo')` ile `else if (state.mod === 'taraftar')` branch'lerinin ikisini de güncellemeyi unutma.
