# The Qualcomm userspace the Mi A2 Lite's modem needs and Arch Linux ARM does
# not package: rmtfs (the modem's remote file system) and qrtr (its IPC
# router library, and qrtr-lookup to list the modem's services), from
# linux-msm's release tarballs; and q6voiced (postmarketOS: opens the audio
# DSP's voice PCM while ModemManager has a call up, which starts call audio),
# with tinyalsa built into it. Built on aarch64 Arch Linux ARM (gcc, libudev,
# libdbus), into ../overlay/usr/local. qrtr's and tinyalsa's own builds are
# replaced by their library sources, so make and gcc are all it needs.
#
#   make            download, check and build
#   make install    into ../overlay (what is committed)

QRTR_VER  := 1.2
QRTR_SHA  := 7a82bf80246fe71287b13c66c0466208822e8337fcd4aaf302eee6c5fcb48a52
RMTFS_VER := 1.3
RMTFS_SHA := 4b6c398806a1046e2256a685d592833de0f37b7245b8adc4797e1791f0e117ee
TINYALSA_VER := 2.0.0
TINYALSA_SHA := 573ae0b2d3480851c1d2a12503ead2beea27f92d44ed47b74b553ba947994ef1
# q6voiced has no releases: a commit (its latest, 2024-06-07)
Q6V_VER   := 75ae4079fc40c1c555ce9129ee86780bf598aaf6
Q6V_SHA   := d2c25597564e1962a16bf104b56bf9d817d3720ca074949ee2bbd47073ff5953

CC      ?= gcc
CFLAGS  ?= -O2 -g -Wall
OVERLAY := ../overlay/usr/local
B       := build
QRTR    := $(B)/qrtr-$(QRTR_VER)
RMTFS   := $(B)/rmtfs-$(RMTFS_VER)
TINYALSA := $(B)/tinyalsa-$(TINYALSA_VER)
Q6V     := $(B)/q6voiced-$(Q6V_VER)

all: $(B)/libqrtr.so.1 $(B)/qrtr-lookup $(B)/rmtfs $(B)/q6voiced

$(B)/%.tar.gz:
	@mkdir -p $(B)
	curl -fsSL -o $@.tmp $(URL_$*)
	echo "$(SHA_$*)  $@.tmp" | sha256sum -c -
	mv $@.tmp $@

URL_qrtr-$(QRTR_VER)   := https://github.com/linux-msm/qrtr/archive/refs/tags/v$(QRTR_VER).tar.gz
SHA_qrtr-$(QRTR_VER)   := $(QRTR_SHA)
URL_rmtfs-$(RMTFS_VER) := https://github.com/linux-msm/rmtfs/archive/refs/tags/v$(RMTFS_VER).tar.gz
SHA_rmtfs-$(RMTFS_VER) := $(RMTFS_SHA)
URL_tinyalsa-$(TINYALSA_VER) := https://github.com/tinyalsa/tinyalsa/archive/refs/tags/v$(TINYALSA_VER).tar.gz
SHA_tinyalsa-$(TINYALSA_VER) := $(TINYALSA_SHA)
URL_q6voiced-$(Q6V_VER) := https://gitlab.com/postmarketOS/q6voiced/-/archive/$(Q6V_VER)/q6voiced-$(Q6V_VER).tar.gz
SHA_q6voiced-$(Q6V_VER) := $(Q6V_SHA)

$(QRTR)/.unpacked: $(B)/qrtr-$(QRTR_VER).tar.gz
	tar -C $(B) -xzf $<
	touch $@

$(RMTFS)/.unpacked: $(B)/rmtfs-$(RMTFS_VER).tar.gz
	tar -C $(B) -xzf $<
	touch $@

$(TINYALSA)/.unpacked: $(B)/tinyalsa-$(TINYALSA_VER).tar.gz
	tar -C $(B) -xzf $<
	touch $@

$(Q6V)/.unpacked: $(B)/q6voiced-$(Q6V_VER).tar.gz
	tar -C $(B) -xzf $<
	touch $@

$(B)/libqrtr.so.1: $(QRTR)/.unpacked
	$(CC) $(CFLAGS) -fPIC -shared -Wl,-soname,libqrtr.so.1 -I$(QRTR)/include -o $@ \
		$(QRTR)/lib/logging.c $(QRTR)/lib/qmi.c $(QRTR)/lib/qrtr.c

$(B)/qrtr-lookup: $(QRTR)/.unpacked $(B)/libqrtr.so.1
	$(CC) $(CFLAGS) -I$(QRTR)/include -o $@ $(QRTR)/src/lookup.c $(B)/libqrtr.so.1

$(B)/rmtfs: $(RMTFS)/.unpacked $(B)/libqrtr.so.1
	$(CC) $(CFLAGS) -I$(QRTR)/include -o $@ \
		$(addprefix $(RMTFS)/,qmi_rmtfs.c rmtfs.c rproc.c sharedmem.c storage.c util.c) \
		$(B)/libqrtr.so.1 -ludev -lpthread

# q6voiced uses four of tinyalsa's calls: its PCM and plugin sources, linked in
$(B)/q6voiced: $(Q6V)/.unpacked $(TINYALSA)/.unpacked
	$(CC) $(CFLAGS) -I$(TINYALSA)/include $$(pkg-config --cflags dbus-1) -o $@ $(Q6V)/q6voiced.c \
		$(addprefix $(TINYALSA)/src/,pcm.c pcm_hw.c pcm_plugin.c snd_card_plugin.c limits.c) \
		$$(pkg-config --libs dbus-1) -ldl

install: all
	install -Dm755 $(B)/libqrtr.so.1 $(OVERLAY)/lib/libqrtr.so.1
	install -Dm755 $(B)/qrtr-lookup $(OVERLAY)/bin/qrtr-lookup
	install -Dm755 $(B)/rmtfs $(OVERLAY)/bin/rmtfs
	install -Dm644 $(QRTR)/LICENSE ../overlay/usr/share/licenses/qrtr/LICENSE
	install -Dm644 $(RMTFS)/LICENSE ../overlay/usr/share/licenses/rmtfs/LICENSE
	install -Dm755 $(B)/q6voiced $(OVERLAY)/bin/q6voiced
	install -Dm644 $(Q6V)/LICENSE ../overlay/usr/share/licenses/q6voiced/LICENSE
	install -Dm644 $(TINYALSA)/NOTICE ../overlay/usr/share/licenses/tinyalsa/NOTICE

clean:
	rm -rf $(B)

.PHONY: all install clean
