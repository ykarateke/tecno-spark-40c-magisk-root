# Detaylı Root Rehberi — TECNO SPARK 40C (KM4k/KM4n)

Bu doküman, cihazı **veri silmeden** Magisk ile rootlama adımlarını ayrıntılı anlatır.

---

## 0. Ön koşullar

- Bootloader kilidi **açık**:
  ```powershell
  adb reboot bootloader
  fastboot getvar unlocked      # unlocked: yes olmali
  fastboot getvar current-slot  # a veya b
  fastboot reboot
  ```
- PC'de `adb` ve `fastboot` çalışıyor.
- Telefonda **USB hata ayıklama** açık ve bilgisayar ADB yetkilendirmesi yapılmış.
- `stock_init_boot.img` hazır (firmware'den çıkarılmış ya da bu repodan).

> Bu cihaz **A/B** olduğu için root hedefi `init_boot` (a/b) partition'ıdır. **`boot` partition'ı DEĞİL.**

---

## 1. Stock `init_boot.img`'i edinme

### Seçenek A — Bu repodaki hazır imaj

Repoda `stock_init_boot.img` zaten var; doğrudan kullan.

### Seçenek B — Firmware'den çıkarma

```powershell
# 7-Zip ile arsivden cikar
& 'C:\Program Files\7-Zip\7z.exe' e "TecnoSpark40C_KM4k.7z" `
    -o "C:\fw" -ir!"*init_boot.img"

# Dogrulama: magic = ANDROID!, header version 4, ramdisk ~2.6MB
python -c "import struct;d=open(r'C:\fw\init_boot.img','rb').read(64);print(d[:8], struct.unpack('<I',d[12:16])[0])"
```

Beklenen: `b'ANDROID!' 2638395` (ramdisk boyutu).

Doğru dosyanın hash'i repodaki `stock_init_boot.img` ile aynı olmalıdır.

---

## 2. Magisk'in stok imajı yamaması

1. [`tools/Magisk-v30.7.apk`](../tools/Magisk-v30.7.apk) dosyasını telefona kur:
   ```powershell
   adb install -r tools\Magisk-v30.7.apk
   ```
2. Stok imajı telefona at:
   ```powershell
   adb push stock_init_boot.img /sdcard/Download/stock_init_boot.img
   ```
3. Telefonda **Magisk** uygulamasını aç:
   - **Magisk** kartındaki **Kur** butonuna bas
   - **"Bir Dosya Seç ve Yamala"** (Select and Patch a File) seçeneğini seç
   - **Downloads → `stock_init_boot.img`** seç
   - **LET'S GO** → **"All done!"** beklenir
4. Çıktı dosyası: `/sdcard/Download/magisk_patched-XXXXX.img`

> ⚠️ **"Doğrudan Kur" (Direct Install) DEĞİL**, "Bir Dosya Seç ve Yamala" seçilmeli.

---

## 3. Yamalı imajı geri çekme

```powershell
adb shell ls /sdcard/Download/magisk_patched*.img
adb pull /sdcard/Download/magisk_patched-XXXXX.img magisk_patched_init_boot.img
```

Boyut stok imajla aynı olmalı (8.388.608 bayt).

> ⚠️ Yamalı imaj **cihaza ve build'e özeldir**; hash'i firmware/sürüme göre değişir ve bu repoda
> tutulmaz. Doğrulamak için stok imajla aynı boyutta olması yeterlidir. Başka bir cihazdan alınan
> yamalı imajı **asla flashlama**.

---

## 4. Fastboot ile flash

```powershell
adb reboot bootloader
fastboot devices                       # seri numarasi gorunmeli
fastboot getvar current-slot           # ornek: b

# Dogrudan init_boot'a yaz (aktif slot otomatik secilir)
fastboot flash init_boot magisk_patched_init_boot.img

fastboot reboot
```

Çıktı örneği:
```
Sending 'init_boot_b' (8192 KB)   OKAY
Writing 'init_boot_b'             OKAY
Finished. Total time: 0.328s
```

> **Not:** `fastboot flash init_boot` komutu aktif slota yazar. Belirli bir slota yazmak için
> `fastboot flash init_boot_a ...` veya `_b` kullanılabilir.

---

## 5. Doğrulama

```powershell
adb shell getprop sys.boot_completed       # 1
adb shell su -c id
# uid=0(root) gid=0(root) groups=0(root) context=u:r:magisk:s0
```

`u:r:magisk:s0` context'i = Magisk root aktif.

---

## 6. Root erişimi (shell)

Magisk ilk kurulumda `su` isteğini onay bekler. İki yol:

- **Telefondan:** Magisk → **Süper Kullanıcı** → `[SharedUID] Kabuk / com.android.shell` satırını aktif et → parmak izi/PIN doğrula.
- **Ayarlardan:** Magisk → **Ayarlar → Süper Kullanıcı → Otomatik Yanıt = İzin Ver**.

---

## 7. Sonrası

Root gizleme (banka/POS uygulamaları için): **[HIDE-ROOT.md](HIDE-ROOT.md)**

Root kaldırma / stok'a dönme: **[UNROOT.md](UNROOT.md)**

---

## Sık kullanılan komutlar

| Amaç | Komut |
|---|---|
| Aktif slot | `fastboot getvar current-slot` |
| Kilit durumu | `fastboot getvar unlocked` |
| init_boot partition boyutu | `fastboot getvar partition-size:init_boot_b` |
| Yamalı init_boot doğrulama | `adb shell sha256sum /sdcard/Download/magisk_patched-*.img` |
| Magisk sürüm | `adb shell su -c 'magisk -V'` |
