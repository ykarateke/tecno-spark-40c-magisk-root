# Sorun Giderme

## Bootloop / cihaz açılmıyor

**En sık sebep:** Yanlış/uyumsuz `init_boot` imajı flashlandı.

**Çözüm — stok imajı geri yaz:**
```powershell
fastboot flash init_boot stock_init_boot.img
fastboot reboot
```

Cihaz fastboot'a girmiyorsa:
- Ses Kısma + Güç ile fastboot'a zorla, ya da
- Recovery'den `adb reboot bootloader`.

---

## Fastboot cihazı görmüyor

- Windows'ta **fastboot sürücüsü** kurulu olmalı (Kod 28 hatası = sürücü yok).
- Cihaz bootloader'da `VID_0E8D&PID_201C` olarak görünür.
- Sürücü: Google USB Driver (`android_winusb.inf`) genelde yeterlidir.
- Farklı USB portu / kablo dene (USB 2.0 portu tercih et).

---

## `su` → "Permission denied"

Magisk kurulu ve root çalışıyor ama shell'e izin verilmemiş.
- Telefon → **Magisk → Süper Kullanıcı → `[SharedUID] Kabuk`** → aktif et (parmak izi ister).
- Veya **Ayarlar → Süper Kullanıcı → Otomatik Yanıt = İzin Ver**.

---

## Shamiko `[❌ Unsupported environment]`

**Sebep:** ReZygisk / Zygisk Next gibi alternatif Zygisk kullanılıyor.

**Çözüm:**
1. ReZygisk'i devre dışı bırak: `su -c touch /data/adb/modules/rezygisk/disable`
2. Magisk Zygisk'i aç: `UPDATE settings SET value=1 WHERE key='zygisk';`
3. Reboot.
4. `module.prop` → `[😋 Shamiko is working as blacklist mode]` görmelisin.

Detay: [HIDE-ROOT.md](HIDE-ROOT.md)

---

## Uygulama root algılıyor / kapanıyor

1. Uygulamayı hem **DenyList**'e hem **Tricky Store `target.txt`**'ye ekle.
2. **Red Listesini Uygula (enforce)** kapalı olsun (Shamiko için).
3. **Reboot** at.
4. **Geliştirici seçenekleri + ADB** açıksa kapat:
   ```powershell
   adb shell settings put global development_settings_enabled 0
   adb shell settings put global adb_enabled 0
   ```
5. Magisk uygulamasını gizle (rastgele paket adı).
6. Play Integrity sonucunu kontrol et; DEVICE/STRONG düşükse PIF/TrickyStore güncelle.

> Not: Bootloader açıkken bazı uygulamalar yazılımla tam gizlenemez.

---

## Magisk uygulaması silindi ama root duruyor

Sorun değil. `magiskd` `init_boot`'ta olduğu için root çalışır. APK'yı yeniden kur:
```powershell
adb install -r tools\Magisk-v30.7.apk
```
veya [Magisk releases](https://github.com/topjohnwu/Magisk/releases) son sürüm.

---

## BROM / preloader moduna girme (dump için)

Bu cihazda **tuş kombinasyonları ve `adb reboot edl` çalışmadı**. Denenenler:
- Ses Açma + Ses Kısma + USB
- Ses Açma + Ses Kısma + Güç + USB
- `adb reboot edl`, `fastboot oem edl`, `fastboot reboot edl` → hepsi desteklenmiyor

Bu yüzden `init_boot` **firmware arşivinden** çıkarıldı (dump yerine). Eğer BROM gerekirse:
- mtkclient + UsbDk + libusb kurulumu gerekir.
- Kombinasyon cihazda zor; test point gerekebilir.

---

## Firmware indirme (MEGA/Drive kota)

- **MEGA:** IP başına bant genişliği kotası. VPN / mobil hotspot / MEGAsync ile aşılır.
- **Google Drive:** "too many users" hatası IP bazlı; VPN ya da "kopya oluştur".
