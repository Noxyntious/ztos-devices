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
sensor, accelerometer, gyroscope, notification LED, Wi-Fi (2.4 and 5 GHz), Bluetooth
(controller up), speaker, USB networking and SSH. The modem: SIM, network
(LTE), calls both ways with sound (q6voiced) and SMS both ways, through
ztOSMobile's ztoscorenetworkingd-mm over ModemManager. Untested: mobile data.
Not started: cameras, GPS.

Waydroid is off: pmOS's msm8953 kernel has binder with fixed devices but
not binderfs (`CONFIG_ANDROID_BINDERFS`), which ztOS's Waydroid needs.

## Where the overlay's files come from

| Files | From | Licence |
|---|---|---|
| `usr/bin/msm-firmware-loader*.sh`, `usr/lib/systemd/system/msm-firmware-loader*.service` | postmarketOS's msm-firmware-loader, as in `samsung-j5`; two changes for lk2nd, marked in the script: slot `_a` when no slot suffix is passed (daisy's `modem_a` and `modem_b` carry the same firmware), and `mkdir` of `/run/msm-firmware-loader` | MIT |
| `usr/local/bin/rmtfs`, `usr/local/bin/qrtr-lookup`, `usr/local/lib/libqrtr.so.1` | built by `userspace/Makefile` from linux-msm's rmtfs 1.3 and qrtr 1.2 (Arch Linux ARM packages neither) | BSD-3-Clause, in `usr/share/licenses` |
| `etc/systemd/system/rmtfs.service` | as in `samsung-j5` | |
| `usr/lib/msm-modem-uim-selection`, its unit, `etc/systemd/system/rmtfs.service.d/` | postmarketOS's msm-modem 13 (selects the SIM's USIM application before ModemManager) | GPL-3.0-or-later, in `usr/share/licenses/msm-modem` |
| `usr/share/alsa/ucm2/` (`conf.d/xiaomi-daisy`, `Xiaomi/daisy`, `codecs/msm8953-wcd`) | msm8953-mainline's alsa-ucm-conf at `ed9334bda853fe032794751c34cea03ec0d7d4eb`, the commit pmaports' `soc-qcom-msm8953-ucm` uses | BSD-3-Clause, `Xiaomi/daisy/LICENSE.alsa-ucm-conf` |
| `etc/wireplumber/wireplumber.conf.d/51-qcom.conf` | pmaports `soc-qcom-msm8953` (S16LE, 48 kHz, larger periods for the DSP) | |
| `usr/local/bin/q6voiced`, `etc/systemd/system/q6voiced.service` | built by `userspace/Makefile` from postmarketOS's q6voiced (commit `75ae407`) with tinyalsa 2.0.0 linked in; opens the voice PCM (`hw:0,4`, VoiceMMode1, as pmOS's `soc-qcom-msm8953`) while ModemManager has a call up, which starts call audio. The UCM HiFi verb already routes VoiceMMode1 to the earpiece codec and from the microphone | MIT (q6voiced), BSD-3-Clause (tinyalsa), in `usr/share/licenses` |
| `etc/modules-load.d/ledtrig-timer.conf` | the notification LED blinks with the timer trigger, a module in this kernel | |
| `vendor/ztos/manifest.toml` | ztOSIAL manifest: generic backends, the battery and charger, the USB controller | |

To rebuild the userspace binaries (rmtfs, qrtr, q6voiced), on aarch64 Arch Linux ARM:
`make -C userspace install`.
