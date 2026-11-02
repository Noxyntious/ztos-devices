# Xiaomi Mi A2 Lite (xiaomi-daisy)

Snapdragon 625 (msm8953), Adreno 506, 1080x2280. Built from postmarketOS's
generic msm8953 port (`device-qcom-msm8953`), as pmOS itself does since it
archived `device-xiaomi-daisy`; its kernel `linux-postmarketos-qcom-msm8953`
picks daisy's device tree. Every ztOSIAL interface uses its generic backend.

## Booting

lk2nd (`lk2nd-msm8953`) sits on `boot` and boots extlinux from the ext2 boot
partition inside `userdata`, so `flash.sh` writes only `userdata`. A phone
that has run postmarketOS already has lk2nd: hold Vol-Down at power-on for
lk2nd's fastboot. A stock phone needs lk2nd once, from Xiaomi's fastboot:
`fastboot flash boot lk2nd.img` (the active slot; `fastboot getvar
current-slot`).

## What works

Tested on the phone: display and GPU (freedreno, OpenGL ES 3.1), touch,
power and volume keys, vibration, battery and charging, backlight, light
sensor, accelerometer, gyroscope, Wi-Fi (2.4 and 5 GHz), Bluetooth
(controller up), speaker, USB networking and SSH. The modem starts and
serves QMI; with no SIM inserted ModemManager reports `sim-missing`, so SIM,
calls, SMS and mobile data are untested. Not started: call audio (q6voiced),
cameras, GPS.

## Where the overlay's files come from

| Files | From | Licence |
|---|---|---|
| `usr/bin/msm-firmware-loader*.sh`, `usr/lib/systemd/system/msm-firmware-loader*.service` | postmarketOS's msm-firmware-loader, as in `samsung-j5`; two changes for lk2nd, marked in the script: slot `_a` when no slot suffix is passed (daisy's `modem_a` and `modem_b` carry the same firmware), and `mkdir` of `/run/msm-firmware-loader` | MIT |
| `usr/local/bin/rmtfs`, `usr/local/bin/qrtr-lookup`, `usr/local/lib/libqrtr.so.1` | built by `userspace/Makefile` from linux-msm's rmtfs 1.3 and qrtr 1.2 (Arch Linux ARM packages neither) | BSD-3-Clause, in `usr/share/licenses` |
| `etc/systemd/system/rmtfs.service` | as in `samsung-j5` | |
| `usr/lib/msm-modem-uim-selection`, its unit, `etc/systemd/system/rmtfs.service.d/` | postmarketOS's msm-modem 13 (selects the SIM's USIM application before ModemManager) | GPL-3.0-or-later, in `usr/share/licenses/msm-modem` |
| `usr/share/alsa/ucm2/` (`conf.d/xiaomi-daisy`, `Xiaomi/daisy`, `codecs/msm8953-wcd`) | msm8953-mainline's alsa-ucm-conf at `ed9334bda853fe032794751c34cea03ec0d7d4eb`, the commit pmaports' `soc-qcom-msm8953-ucm` uses | BSD-3-Clause, `Xiaomi/daisy/LICENSE.alsa-ucm-conf` |
| `etc/wireplumber/wireplumber.conf.d/51-qcom.conf` | pmaports `soc-qcom-msm8953` (S16LE, 48 kHz, larger periods for the DSP) | |
| `vendor/ztos/manifest.toml` | ztOSIAL manifest: generic backends, the battery and charger, the USB controller | |

To rebuild the userspace binaries, on aarch64 Arch Linux ARM:
`make -C userspace install`.
