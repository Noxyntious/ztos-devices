# ModemManager for the Redmi 3S: Arch Linux ARM's own version, with one request
# left out that land's modem firmware (MPSS.JO.1.2, 2016) crashes on: WDS
# "Configure Profile Event List", sent while enabling the modem since
# ModemManager 1.22 (the Quectel BG96 crashes the same way: ModemManager issue
# #795). Without this the modem restarts as soon as ModemManager enables it,
# again and again, and the SIM is never seen.
#
# Only the daemon is built and replaced: the plugins, libmm-glib, mmcli, the
# D-Bus and udev files stay Arch's. It goes to /usr/local/lib/ztos-modemmanager,
# started through ztos-modemmanager, which runs Arch's own daemon instead if
# pacman has since moved ModemManager to another version (then rebuild).
#
# ztConflo builds it in the phone's root under qemu ([userspace].build), with
# meson and glib2-devel from [userspace].build_deps; it can also be made by hand
# in an aarch64 Arch Linux ARM root:
#
#   make            download, check, patch and build
#   make install    into / (the drop-in, the daemon, the wrapper)

MM_VER := 1.24.2
MM_SHA := fbc75adcc0d7b0565f256e7ff4e8872b0a37c4413ff576665f7470932d9c1b68
MM_URL := https://gitlab.freedesktop.org/mobile-broadband/ModemManager/-/archive/$(MM_VER)/ModemManager-$(MM_VER).tar.gz

B    := build
SRC  := $(B)/ModemManager-$(MM_VER)
OUT  := $(B)/meson
DEST := /usr/local/lib/ztos-modemmanager

all: $(OUT)/src/ModemManager

# the daemon must match the plugins and libmm-glib Arch installed
check-version:
	@have=$$(pacman -Q modemmanager | cut -d' ' -f2); have=$${have%-*}; \
	if [ "$$have" != "$(MM_VER)" ]; then \
		echo "Arch Linux ARM has ModemManager $$have, this builds $(MM_VER): set MM_VER and MM_SHA" >&2; \
		exit 1; \
	fi

$(B)/ModemManager-$(MM_VER).tar.gz:
	@mkdir -p $(B)
	curl -fsSL -o $@.tmp $(MM_URL)
	echo "$(MM_SHA)  $@.tmp" | sha256sum -c -
	mv $@.tmp $@

$(SRC)/.patched: $(B)/ModemManager-$(MM_VER).tar.gz skip-profile-event-list.patch
	rm -rf $(SRC)
	tar -C $(B) -xzf $<
	patch -d $(SRC) -p1 < skip-profile-event-list.patch
	touch $@

# Arch's options (arch-meson, polkit permissive), less what only the parts
# left out need (introspection, docs, man pages, completion, tests)
$(OUT)/build.ninja: $(SRC)/.patched | check-version
	rm -rf $(OUT)
	CFLAGS="$${CFLAGS:--O2 -g}" meson setup $(OUT) $(SRC) \
		--prefix=/usr --libexecdir=lib --sbindir=bin --buildtype=plain \
		--wrap-mode=nodownload -Db_pie=true \
		-Ddbus_policy_dir=/usr/share/dbus-1/system.d \
		-Ddist_version='"$(MM_VER)-ztos"' -Dpolkit=permissive \
		-Dintrospection=false -Dvapi=false -Dgtk_doc=false -Dman=false \
		-Dbash_completion=false -Dtests=false -Dexamples=false

$(OUT)/src/ModemManager: $(OUT)/build.ninja
	ninja -C $(OUT) src/ModemManager

install: all
	install -Dm755 $(OUT)/src/ModemManager $(DEST)/ModemManager
	echo $(MM_VER) > $(DEST)/version
	install -Dm755 ztos-modemmanager /usr/local/bin/ztos-modemmanager
	install -Dm644 ztos-modemmanager.conf /etc/systemd/system/ModemManager.service.d/ztos-modemmanager.conf
	install -Dm644 $(SRC)/COPYING /usr/share/licenses/ztos-modemmanager/COPYING
	install -Dm644 skip-profile-event-list.patch /usr/share/licenses/ztos-modemmanager/skip-profile-event-list.patch

clean:
	rm -rf $(B)

.PHONY: all check-version install clean
