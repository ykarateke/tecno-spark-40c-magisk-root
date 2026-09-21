# Root Gizleme — Shamiko + PlayIntegrityFork + Tricky Store

Rootladıktan sonra bazı uygulamalar (banka, POS, kurumsal) root/bootloader açık olduğunu algılayıp
kendini kapatır. Bu doküman, bunları gizlemek için doğru kurulumu anlatır.

---

## Neden gerekli?

Bu cihazda root sonrası:
- `ro.boot.verifiedbootstate = orange` (bootloader açık)
- Magisk/Zygisk izleri
- `su` binary'si

Uygulamalar bunları **Play Integrity**, `su` yolu kontrolü, mount kontrolü gibi yöntemlerle tespit eder.

---

## Gerekli bileşenler

| Modül | Kaynak | Görev |
|---|---|---|
| **Magisk Zygisk** | Magisk (yerleşik) | Zygisk motoru |
| **Shamiko** | [LSPosed releases](https://github.com/LSPosed/LSPosed.github.io/releases) | Root/Zygisk gizleme |
| **PlayIntegrityFork** | [osm0sis/PlayIntegrityFork](https://github.com/osm0sis/PlayIntegrityFork) | Play Integrity düzeltme |
| **Tricky Store** | [5ec1cff/TrickyStore](https://github.com/5ec1cff/TrickyStore) | Sahte kilitli-bootloader kimliği + keybox |

---

## ⚠️ EN ÖNEMLİ TUZAK: Shamiko, ReZygisk ile ÇALIŞMAZ

Shamiko, **Magisk'in kendi Zygisk API'sini** bekler. `ReZygisk` / `Zygisk Next` gibi alternatif
Zygisk implementasyonlarıyla kullanıldığında şu hatayı verir:

```
description=[❌ Unsupported environment] これで勝ったと思うなよ―――!!
```

**Doğru kurulum:**
1. **ReZygisk'i devre dışı bırak / kaldır:**
   ```
   su -c touch /data/adb/modules/rezygisk/disable
   # veya tamamen kaldir:
   su -c rm -rf /data/adb/modules/rezygisk
   ```
2. **Magisk Zygisk'i AÇ:**
   ```sql
   -- Magisk veritabani
   UPDATE settings SET value=1 WHERE key='zygisk';
   ```
3. **Reboot.**
4. Kontrol:
   ```
   su -c cat /data/adb/modules/zygisk_shamiko/module.prop
   ```
   Beklenen:
   ```
   description=[😋 Shamiko is working as blacklist mode]
   ```

---

## Adım adım kurulum

### 1) Modülleri kur
Magisk → **Modüller** → **Depolamadan kur**:

1. `Shamiko-vX.X.X-release.zip`
2. `PlayIntegrityFork-vXX.zip`
3. `Tricky-Store-vX.X.X-release.zip`

### 2) Magisk ayarları
- **Ayarlar → Zygisk:** AÇIK
- **Ayarlar → Red Listesini Uygula (Enforce DenyList):** **KAPALI**
  (Shamiko, DenyList'i kendisi okur; enforce açıkken çalışmaz)
- **Ayarlar → Magisk uygulamasını gizle:** İsteğe bağlı ama önerilir
  (uygulama adı rastgele paketle yeniden paketlenir)

### 3) Red Listesi (DenyList)
Ayarlar → **Red Listesini Yapılandır** → root sezen uygulamaları işaretle:

```
com.android.vending          (Play Store)
com.google.android.gms       (Google Play Services)
com.example.banking          (root sezen uygulama - ornek)
com.example.pos              (POS / odeme uygulamasi - ornek)
```

> Kendi cihazındaki root sezen uygulamaların **paket adlarını** buraya ekle.

ADB'den ekleme:
```powershell
adb shell su -c "magisk --denylist add com.example.banking"
adb shell su -c "magisk --denylist ls"
```

### 4) Tricky Store hedefleri

`/data/adb/tricky_store/target.txt` — root gizlenecek **tüm** paketler burada listelenmeli.
Ayrıca `com.android.vending` ve `com.google.android.gms` mutlaka olmalı.

Örnek içerik:
```
com.android.vending
com.google.android.gms
io.github.vvb2060.keyattestation
com.example.banking
com.example.pos
```

Güncelleme:
```powershell
adb push target.txt /sdcard/target.txt
adb shell su -c "cp /sdcard/target.txt /data/adb/tricky_store/target.txt"
adb shell su -c "chmod 644 /data/adb/tricky_store/target.txt"
```

Doğru **keybox.xml** dosyasının `Tricky Store` klasöründe olduğundan emin ol.

### 5) Reboot
Modül/Zygisk değişiklikleri sonrası **mutlaka yeniden başlat**.

### 6) Test
- Play Store → **Play Integrity API Checker** → **Check**
  - `MEETS_BASIC_INTEGRITY` ✅
  - `MEETS_DEVICE_INTEGRITY` ✅
  - `MEETS_STRONG_INTEGRITY` (keybox/cihaza bağlı, bazen ✅)
- İlgili uygulamayı açıp dene.

---

## Bilinen gerçekler / sınırlar

- **Bootloader açıkken bazı uygulamalar ekstra katı** olabilir; yazılımla %100 gizleme garanti değildir.
- **Play Integrity** verdict'leri Google tarafından sık değiştirilir; PIF/Tricky Store güncel tutulmalı.
- Bazı uygulamalar hâlâ **direkt kapanabilir** (kendi yazılım güvenlik katmanları nedeniyle).
- **Geliştirici seçenekleri / ADB** açıkken bazı uygulamalar reddeder:
  ```powershell
  adb shell settings put global development_settings_enabled 0
  adb shell settings put global adb_enabled 0
  ```
  (ADB'yi kapatınca uzaktan erişim biter; testleri telefondan yap.)

---

## Kontrol komutları

```powershell
# Root aktif mi
adb shell su -c id

# Shamiko calisiyor mu
adb shell su -c cat /data/adb/modules/zygisk_shamiko/module.prop

# Magisk ayarlari (zygisk=1, denylist=0 olmali)
adb shell su -c "magisk --sqlite 'SELECT * FROM settings'"

# DenyList
adb shell su -c "magisk --denylist ls"

# Tricky Store hedefleri
adb shell su -c cat /data/adb/tricky_store/target.txt
```
