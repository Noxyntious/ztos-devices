# The Qualcomm userspace the Mi A2 Lite's modem needs and Arch Linux ARM does
# not package: rmtfs (the modem's remote file system) and qrtr (its IPC
# router library, and qrtr-lookup to list the modem's services). Built from
# linux-msm's release tarballs on aarch64 Arch Linux ARM (gcc, libudev), into
# ../overlay/usr/local. qrtr's meson build is replaced by its three library
# sources, so make and gcc are all it needs.
#
#   make            download, check and build
#   make install    into ../overlay (what is committed)

QRTR_VER  := 1.2
QRTR_SHA  := 7a82bf80246fe71287b13c66c0466208822e8337fcd4aaf302eee6c5fcb48a52
RMTFS_VER := 1.3
RMTFS_SHA := 4b6c398806a1046e2256a685d592833de0f37b7245b8adc4797e1791f0e117ee

CC      ?= gcc
CFLAGS  ?= -O2 -g -Wall
OVERLAY := ../overlay/usr/local
B       := build
QRTR    := $(B)/qrtr-$(QRTR_VER)
RMTFS   := $(B)/rmtfs-$(RMTFS_VER)

all: $(B)/libqrtr.so.1 $(B)/qrtr-lookup $(B)/rmtfs

$(B)/%.tar.gz:
	@mkdir -p $(B)
	curl -fsSL -o $@.tmp $(URL_$*)
	echo "$(SHA_$*)  $@.tmp" | sha256sum -c -
	mv $@.tmp $@

URL_qrtr-$(QRTR_VER)   := https://github.com/linux-msm/qrtr/archive/refs/tags/v$(QRTR_VER).tar.gz
SHA_qrtr-$(QRTR_VER)   := $(QRTR_SHA)
URL_rmtfs-$(RMTFS_VER) := https://github.com/linux-msm/rmtfs/archive/refs/tags/v$(RMTFS_VER).tar.gz
SHA_rmtfs-$(RMTFS_VER) := $(RMTFS_SHA)

$(QRTR)/.unpacked: $(B)/qrtr-$(QRTR_VER).tar.gz
	tar -C $(B) -xzf $<
	touch $@

$(RMTFS)/.unpacked: $(B)/rmtfs-$(RMTFS_VER).tar.gz
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

install: all
	install -Dm755 $(B)/libqrtr.so.1 $(OVERLAY)/lib/libqrtr.so.1
	install -Dm755 $(B)/qrtr-lookup $(OVERLAY)/bin/qrtr-lookup
	install -Dm755 $(B)/rmtfs $(OVERLAY)/bin/rmtfs
	install -Dm644 $(QRTR)/LICENSE ../overlay/usr/share/licenses/qrtr/LICENSE
	install -Dm644 $(RMTFS)/LICENSE ../overlay/usr/share/licenses/rmtfs/LICENSE

clean:
	rm -rf $(B)

.PHONY: all install clean
