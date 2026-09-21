# TECNO SPARK 40C (KM4k / KM4n) — Magisk ile Root (Veri Kaybı Olmadan)

**🌐 Dil / Language:** **Türkçe** | [English 🇬🇧](README.en.md)

> Bootloader kilidi açık bir **TECNO SPARK 40C** (model `KM4k` / `KM4n`, MediaTek **Helio G81 / MT6768/MT6769**) cihazı, **veri silmeden** Magisk ile rootlama rehberi.

---

## Özet

| | |
|---|---|
| **Cihaz** | TECNO SPARK 40C |
| **Model kodu** | `KM4n` / `KM4k` (board: `km4k_xk67j`) |
| **SoC** | MediaTek MT6768 / MT6769 (Helio G81 / G85 ailesi) |
| **Android** | 15 (HiOS 15.1) |
| **Yöntem** | Magisk patched `init_boot` (A/B cihaz) |
| **Veri kaybı** | **YOK** |
| **Root** | Magisk **v30.7** (versionCode 30700) |
| **Yazılım sürümü** | `KM4n-15.1.2.155(TR001PF001AZ)` |

---

## Nasıl çalışır? (Kısa mantık)

Bu cihaz **A/B (seamless) partition** kullanır. Android 13+ GKI cihazlarda root için `boot` değil **`init_boot`** bölümü yamalanır:

1. Cihazdan **stock `init_boot.img`** alınır (firmware'den çıkarılır).
2. Magisk uygulaması bu imajı **yamalar**.
3. Yamalı imaj `fastboot flash init_boot ...` ile yazılır.
4. Bootloader zaten açık olduğu için **kullanıcı verisi silinmez**.

Standart Magisk yöntemidir; `init_boot` yamalandığı için `/data` dokunulmaz.

---

## Gereksinimler

- **Bootloader kilidi AÇIK** olmalı (`fastboot getvar unlocked` → `yes`).
  - Değilse: cihazda Geliştirici Seçenekleri → OEM Kilidini Aç → `fastboot flashing unlock` (bu işlem veriyi siler).
- PC: **ADB + Fastboot (platform-tools)**
- **7-Zip** (firmware arşivini açmak için)
- **Magisk v30.7 APK** → [`tools/Magisk-v30.7.apk`](tools/Magisk-v30.7.apk) (bu repoda)
- Stock firmware (içinden `init_boot.img` almak için) — aşağıya bak.

---

## Firmware ve `init_boot.img`

Board ile uyumlu stock firmware:
`Tecno Spark 40C (KM4k-XK67JABCDEFGH-V-OP-250711V1696)`

> **Not:** Tam firmware ~4.9 GB'dir ve bu repoda tutulmaz. İndirdikten sonra arşivden **sadece `init_boot.img`** çıkarılır. Bu repodaki [`stock_init_boot.img`](stock_init_boot.img) zaten o firmware'den çıkarılmıştır ve doğrudan kullanılabilir.

Arşivden `init_boot.img` çıkarma:

```powershell
7z e "firmware.7z" -o out -ir!"*init_boot.img"
```

Doğrulama (SHA-256):

```
stock_init_boot.img = 891197858105D3D57CFB6F2CFC7B658A21B0E842504ACA899D9005AF30EAD0FE
```

> ⚠️ **Yamalı imaj (`magisk_patched_init_boot.img`) bu repoda TUTULMAZ.** Sebep: Magisk'in
> yamadığı imaj **cihaza ve yazılım sürümüne özeldir** ve başka bir cihaza/başka bir build'e
> flashlanırsa **bootloop / brick** yapabilir. Bu yüzden herkes **kendi cihazı için kendi imajını
> yamalar**. Adımlar [docs/ROOT-GUIDE.md](docs/ROOT-GUIDE.md) içinde; yama işlemi 2 dakika sürer.

---

## Adım adım root

Detaylı rehber: **[docs/ROOT-GUIDE.md](docs/ROOT-GUIDE.md)**

Hızlı özet:

```powershell
# 0) Cihaz bagli ve ADB yetkili olsun
adb devices

# 1) Stock init_boot'u telefona at
adb push stock_init_boot.img /sdcard/Download/stock_init_boot.img

# 2) Telefonda: Magisk > Kur > "Bir Dosya Sec ve Yamala" > stock_init_boot.img
#    Cikti: /sdcard/Download/magisk_patched-*.img

# 3) Yamali imaji geri cek
adb pull /sdcard/Download/magisk_patched-XXXXX.img magisk_patched_init_boot.img

# 4) Fastboot'a gir ve yamali init_boot'u yaz
adb reboot bootloader
fastboot devices
fastboot flash init_boot magisk_patched_init_boot.img
fastboot reboot

# 5) Dogrula
adb shell su -c id     # uid=0(root) ... context=u:r:magisk:s0
```

---

## Root gizleme (banka / POS / kurumsal uygulamalar)

Bazı uygulamalar root/bootloader açık olduğunu algılayıp çalışmayı reddeder. Kullanılan set:

| Bileşen | Görev |
|---|---|
| **Magisk Zygisk** | Zygisk motoru (ReZygisk DEĞİL — Shamiko ile uyum için) |
| **Shamiko** | Root/Zygisk izlerini gizler (blacklist/denylist modu) |
| **PlayIntegrityFork** | Play Integrity DEVICE/STRONG düzeltir |
| **Tricky Store** | Sahte "kilitli bootloader" kimliği + keybox |

Detay ve tuzaklar: **[docs/HIDE-ROOT.md](docs/HIDE-ROOT.md)**

> ⚠️ **Kritik:** Shamiko, **ReZygisk ile çalışmaz**. Magisk'in kendi Zygisk'i açık olmalı ve ReZygisk devre dışı bırakılmalıdır. Aksi halde Shamiko `[❌ Unsupported environment]` verir.

---

## Repo içeriği

```
.
├── README.md
├── README.en.md                     # English version
├── LICENSE
├── .gitignore
├── stock_init_boot.img              # Firmware'den cikarilmis orijinal imaj
├── tools/
│   └── Magisk-v30.7.apk
├── docs/
│   ├── ROOT-GUIDE.md                # Detayli root adimlari (yamalama dahil)
│   ├── HIDE-ROOT.md                 # Root gizleme (Shamiko/PIF/TrickyStore)
│   ├── TROUBLESHOOTING.md           # Sorun giderme
│   └── UNROOT.md                    # Root kaldirma / stok'a donme
```

---

## Önemli uyarılar

- **OTA güncellemesi** gelirse root kaybolur. Güncelleme öncesi Magisk → Kur → **"Etkin olmayan slota kur"** yapılmalı; güncelleme sonrası tekrar yamalanmalı.
- Bootloader açık kaldığı sürece cihazın `verifiedbootstate=orange` olur; bu bazı sıkı uygulamalarca görülebilir.
- Yanlış partition flashlamak cihazı brickleyebilir. **Her zaman `stock_init_boot.img` yedeğini sakla.**
- Bu rehber eğitim amaçlıdır. Cihazına yaptığın işlemlerin sorumluluğu sana aittir.

> ⚠️ **YASAL UYARI:** Bu rehberi uygulayarak cihazının **brick olması, bootloop, veri kaybı,
> garanti dışı kalma** gibi tüm riskleri kabul etmiş sayılırsın. Depo yazarları/kullanıcıları
> hiçbir zarardan **sorumlu tutulamaz**. Tam metin: **[docs/DISCLAIMER.md](docs/DISCLAIMER.md)**

---

## Teşekkür / Kaynaklar

- [Magisk](https://github.com/topjohnwu/Magisk) — topjohnwu
- [Shamiko](https://github.com/LSPosed/LSPosed.github.io) — LSPosed
- [PlayIntegrityFork](https://github.com/osm0sis/PlayIntegrityFork) — osm0sis
- [Tricky Store](https://github.com/5ec1cff/TrickyStore) — 5ec1cff
- [Hovatek](https://www.hovatek.com) — firmware arşivi

---

## Lisans

[MIT](LICENSE) — Rehber ve dosyalar "olduğu gibi" sunulur, garanti verilmez.
