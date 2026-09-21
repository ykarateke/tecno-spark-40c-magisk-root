# Root Kaldırma / Stok'a Dönme

Root'u kaldırmak istediğinde 3 yol var.

---

## Yol 1 — Stok `init_boot`'u geri yaz (veri korunur)

En temiz yöntem. Sadece root gider, veriler kalır:

```powershell
adb reboot bootloader
fastboot flash init_boot stock_init_boot.img
fastboot reboot
```

Doğrulama:
```powershell
adb shell su -c id      # "su: inaccessible or not found" beklenir
```

Magisk uygulamasını da kaldır:
```powershell
adb uninstall com.topjohnwu.magisk
```

---

## Yol 2 — Magisk "Complete Uninstall" (veri korunur)

1. Magisk → **Başlat/kaldır (Uninstall)** → **Complete Uninstall**
2. Modülleri de kaldırır, stok `init_boot`'u otomatik geri yükler.
3. Reboot.

---

## Yol 3 — Tam stok firmware (veri silinir)

En kötü senaryo / brick kurtarma:

1. Board ile uyumlu stock firmware indir:
   `Tecno Spark 40C (KM4k-XK67JABCDEFGH-...-OP-...)`
2. **SP Flash Tool** ile `MT6768_Android_scatter.txt` kullanarak **Download Only** modda flashla.
3. Bu işlem **tüm veriyi siler** ve bootloader durumunu değiştirmez.

> ⚠️ SP Flash Tool yanlış partition yazarsa brick riski. Scatter dosyasını cihazla eşleştirdiğinden emin ol.

---

## Bootloader kilitleme (opsiyonel)

Root tamamen kaldırıldıktan sonra kilitlemek istersen:

```powershell
adb reboot bootloader
fastboot flashing lock
```

> ⚠️ **Kilitleme cihazı fabrika ayarlarına döndürür (veri silinir)!** Sadece stock + rootsuz durumda yap.
> Yanlış imajla kilitlersen cihaz açılmaz.

---

## OTA notu

Bootloader açık + root'lu cihaz OTA alırken sorun çıkarabilir:
- Güncelleme öncesi **Magisk → Kur → "Etkin olmayan slota kur"** (Install to inactive slot) yöntemi,
- Sonrasında tekrar `init_boot` yamalama gerekir.

Ya da Yol 1 ile root'u kaldırıp OTA yapıp yeniden rootlamak en garantisidir.
