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

Tested on the phone: display (EBBG panel) and GPU (freedreno, Adreno 505),
touch, the capacitive keys (below), power and volume keys, vibration,
suspend and resume, battery and charging, backlight, the RGB notification
LED (under the home key), Wi-Fi, Bluetooth (headsets: ztOS pairs as
NoInputNoOutput, so pairing another phone fails), USB networking and SSH.
Sensors through ztOSIAL: accelerometer, gyroscope and proximity (qcom-smgr,
the sensor DSP; read as streams); the magnetometer the DSP lists reads 0.
No light sensor. Untested: speaker, microphone, headphones. Not working:
cameras.

**Modem: started, not yet usable.** msm89x7-mainline's land device tree
leaves it off; `device.toml`'s `dtb_enable`/`dtb_set` switch it on as land's
siblings do. It boots (MPSS.JO.1.2, 2016), serves every QMI service and reads
the SIM (slot 2 tested), and `msm-modem-wwan-port` opens the bam-dmux data
port; the modem then powers bam-dmux but never opens a data channel, so
there is no wwan netdev, and ModemManager makes no QMI modem without one:
no calls, SMS or mobile data yet.

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
| `usr/local/bin/{rmtfs,qrtr-lookup}`, `usr/local/lib/libqrtr.so.1`, `etc/systemd/system/rmtfs.service*`, `usr/lib/msm-modem-uim-selection*` | as `xiaomi-daisy` (its `userspace/Makefile` builds rmtfs and qrtr) | BSD-3-Clause, GPL-3.0-or-later, in `usr/share/licenses` |
| `usr/lib/msm-modem-wwan-port`, its unit | postmarketOS's msm-modem 13 (`5f504e06`): opens the bam-dmux data port with qmicli before ModemManager | GPL-3.0-or-later |
| `vendor/ztos/manifest.toml` | ztOSIAL manifest: generic backends | |
