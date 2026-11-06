# Xiaomi Redmi 3S (xiaomi-land)

Snapdragon 430 (msm8937), Adreno 505, 720x1280, 2 or 3 GB of RAM. Built from
postmarketOS's generic msm89x7 port (`device-qcom-msm89x7`, category
testing), as pmOS itself runs land; its kernel `linux-postmarketos-qcom-msm89x7`
(msm89x7-mainline) picks land's device tree. Every ztOSIAL interface uses its
generic backend.

## Booting

lk2nd (`lk2nd-msm8952`) sits on `boot` and boots extlinux from the ext2 boot
partition inside `userdata`, so `flash.sh` writes only `userdata`, from
Xiaomi's fastboot (hold Vol-Down at power-on). land has no A/B slots: lk2nd
goes once on the one `boot` partition, `fastboot flash boot lk2nd.img`.

## State

Boots to ztMETRO; Wi-Fi, touch, the keys and USB networking work on the phone. From the device tree: display (three panels, BOE,
Tianma and EBBG, all in the initramfs), touch (FocalTech FT5306), Wi-Fi
(WCN3620), audio (PM8937 codec; speaker through an AW8738 amplifier on HPHR),
an RGB notification LED (AW2013), battery and charger (PMI8950).

**No modem**: msm89x7-mainline's land device tree does not enable the modem
(`&modem` stays disabled), so there are no calls, SMS or mobile data until it
does.

The capacitive keys under the glass are touches the FT5306 reports below the
panel (y = 2000; x = 180, 540, 900): ztOSMobile's `ztos-virtualkeyd` turns them
into menu, home and back from `etc/ztos/virtualkeys.conf`, and ztMETRO then
draws no nav bar.

## Where the overlay's files come from

| Files | From | Licence |
|---|---|---|
| `usr/bin/msm-firmware-loader*.sh`, `usr/lib/systemd/system/msm-firmware-loader*.service` | as `xiaomi-daisy` (postmarketOS's msm-firmware-loader); its lk2nd slot change only acts when a `modem_a` partition exists, and land has none | MIT |
| `usr/share/alsa/ucm2/` (`conf.d/xiaomi-land`, `Xiaomi/riva`, `codecs/msm8953-wcd`) | msm89x7-mainline's alsa-ucm-conf at `6078de44affd1023c1ba4f71ef1e21468e3282e9`, the commit pmaports' `soc-qcom-msm89x7` uses (land's profile is riva's) | BSD-3-Clause, `Xiaomi/riva/LICENSE.alsa-ucm-conf` |
| `etc/wireplumber/wireplumber.conf.d/51-qcom.conf` | pmaports `soc-qcom-msm89x7` | |
| `etc/modules-load.d/ledtrig-timer.conf` | the notification LED blinks with the timer trigger | |
| `etc/ztos/virtualkeys.conf` | the capacitive keys' boxes for ztos-virtualkeyd, measured on the phone | |
| `vendor/ztos/manifest.toml` | ztOSIAL manifest: generic backends | |
