.PHONY: createdirs ndk qemu limbo ffi glib ffi-clean glib-clean qemu qemu-clean pixman pixman-clean

include android-config.mak

CFLAGS=$(ARCH_CFLAGS)

ifeq ($(NDK_ENV), windows)
	NDK_BUILD = ndk-build.cmd
else ifeq ($(NDK_ENV), windows-x86_64)
	NDK_BUILD = ndk-build.cmd
else
	NDK_BUILD = ndk-build
endif

ndkbuild=cd $(NDK_PROJECT_PATH) && $(NDK_BUILD) -j $(BUILD_THREADS)
ndkclean=cd $(NDK_PROJECT_PATH) && $(NDK_BUILD) clean
ndkcleanobj=rm -rf $(NDK_PROJECT_PATH)/obj/

ifeq ($(NDK_ENV), windows)
	ndkgdb=cd $(NDK_PROJECT_PATH) && ndk-gdb.cmd
else ifeq ($(NDK_ENV), windows-x86_64)
	ndkgdb=cd $(NDK_PROJECT_PATH) && ndk-gdb.cmd
else
	ndkgdb=cd $(NDK_PROJECT_PATH) && ndk-gdb
endif

CREATE_OBJDIR=mkdir -p $(NDK_PROJECT_PATH)/obj/local/$(APP_ABI)/
CREATE_LIBSDIR=mkdir -p $(NDK_PROJECT_PATH)/libs/$(APP_ABI_DIR)/
DEL_LIBSDIR=rm -rf $(NDK_PROJECT_PATH)/libs/armeabi \
	$(NDK_PROJECT_PATH)/libs/armeabi-v7a \
	$(NDK_PROJECT_PATH)/libs/x86 \
	$(NDK_PROJECT_PATH)/libs/mips \
	$(NDK_PROJECT_PATH)/jniLibs/*


INSTALL_GLIB_LIBS=cp -f glib/glib/.libs/*.so \
    $(NDK_PROJECT_PATH)/obj/local/$(APP_ABI)/

INSTALL_FFI_LIBS=cp -f libffi/$(GNU_HOST)/.libs/*.a \
    $(NDK_PROJECT_PATH)/obj/local/$(APP_ABI)/

INSTALL_PIXMAN_LIBS=cp -f pixman/pixman/.libs/*.so \
    $(NDK_PROJECT_PATH)/obj/local/$(APP_ABI)/

MOD_QEMU_LIBS=$(OBJCOPY) --redefine-sym open=android_open  \
    qemu/x86_64-softmmu/qemu-system-x86_64

RELINK_QEMU_LIBS=$(LNK) $(SYS_ROOT) -L$(LIMBO_JNI_ROOT)/../obj/local/$(APP_ABI) \
    -lcompat-limbo -shared qemu/x86_64-softmmu/qemu-system-x86_64 \
    -o qemu/x86_64-softmmu/qemu-system-x86_64.so

INSTALL_LIBS=cp -f $(NDK_PROJECT_PATH)/obj/local/$(APP_ABI)/*.so \
	../libs/$(APP_ABI_DIR)/

REMOVE_GDBSETUP=rm -f ../libs/$(APP_ABI_DIR)/gdb.setup
REMOVE_GDBSERVER=rm -f ../libs/$(APP_ABI_DIR)/gdbserver

SYNC_LIBS=rsync -av --delete --delete-excluded --exclude libqemu-system-* ../libs/* ../jniLibs/

SYNC_LIBS_X86=rsync -av --include='libqemu-system-x86*.so' \
    --include='gdbserver' --include='gdb.setup' \
    --include='*/' --exclude='*' \
    ../libs/ ../../../../limbo-android-x86/src/main/jniLibs/
SYNC_LIBS_ARM=rsync -av --include='libqemu-system-arm*.so' \
    --include='gdbserver' --include='gdb.setup' \
    --include='*/' --exclude='*' \
    ../libs/ ../../../../limbo-android-arm/src/main/jniLibs/
SYNC_LIBS_PPC=rsync -av --include='libqemu-system-ppc*.so' \
    --include='gdbserver' --include='gdb.setup' \
    --include='*/' --exclude='*' \
    ../libs/ ../../../../limbo-android-ppc/src/main/jniLibs/
SYNC_LIBS_SPARC=rsync -av --include='libqemu-system-sparc*.so' \
    --include='gdbserver' --include='gdb.setup' \
    --include='*/' --exclude='*' \
    ../libs/ ../../../../limbo-android-sparc/src/main/jniLibs/

COPY_ROMS=-mkdir -p ../assets/roms && cp -rf ./qemu/pc-bios/*.rom \
./qemu/pc-bios/*.bin \
./qemu/pc-bios/*.img \
./qemu/pc-bios/*.bmp \
./qemu/pc-bios/*.ico \
./qemu/pc-bios/*.svg \
./qemu/pc-bios/*.rsrc \
./qemu/pc-bios/openbios-* \
./qemu/pc-bios/README \
../assets/roms/

COPY_KEYMAPS=cp -rf ./qemu/pc-bios/keymaps ../assets/roms/
COPY_RESOLV_CONF=mkdir -p ../assets/roms/etc/ && printf "nameserver 8.8.8.8\n\n" > ../assets/roms/etc/resolv.conf

COPY_ASSETS_X86=mkdir -p ../../../../limbo-android-x86/src/main/assets/ && \
    cp -rf ../assets/* ../../../../limbo-android-x86/src/main/assets/
COPY_ASSETS_ARM=mkdir -p ../../../../limbo-android-arm/src/main/assets/ && \
    cp -rf ../assets/* ../../../../limbo-android-arm/src/main/assets/
COPY_ASSETS_PPC=mkdir -p ../../../../limbo-android-ppc/src/main/assets/ && \
    cp -rf ../assets/* ../../../../limbo-android-ppc/src/main/assets/
COPY_ASSETS_SPARC=mkdir -p ../../../../limbo-android-sparc/src/main/assets/ && \
    cp -rf ../assets/* ../../../../limbo-android-sparc/src/main/assets/

#RENAME_TUNCTL=mv -f $(NDK_PROJECT_PATH)/libs/$(APP_ABI)/tunctl \
	$(NDK_PROJECT_PATH)/libs/$(APP_ABI)/libtunctl.so
#RENAME_PARPROUTED=mv -f $(NDK_PROJECT_PATH)/libs/$(APP_ABI)/parprouted \
	$(NDK_PROJECT_PATH)/libs/$(APP_ABI)/libparprouted.so
	
#GDB needs the gdb.setup under ./libs
COPY_GDB_SETUP_LIBS=-cp -f $(NDK_PROJECT_PATH)/libs/$(APP_ABI)/gdb.setup \
	$(NDK_PROJECT_PATH)/libs/gdb.setup
	
ifneq ($(NDK_DEBUG),1)
	STRIP_LIBS=$(STRIP) --strip-unneeded  $(subst /cygdrive/c,c:,$(NDK_PROJECT_PATH))/libs/$(APP_ABI_DIR)/*.so
endif

DEBUG =
ifeq ($(NDK_DEBUG), 1)
	LIBFFI_DEBUG = --enable-debug
	LIBGLIB_DEBUG = --enable-debug
	LIBPIXMAN_DEBUG =
else
	LIBFFI_DEBUG = --disable-debug
    LIBGLIB_DEBUG = --disable-debug
    LIBPIXMAN_DEBUG =
endif

export

#debug is default
# use "make release" for production version
all: debug
	
	
createdirs:
	$(CREATE_OBJDIR)
	$(DEL_LIBSDIR)
	$(CREATE_LIBSDIR)

ndk:
	$(ndkbuild)

ndk-debug:
	$(ndkbuild) NDK_DEBUG=1

ndk-clean:
	$(ndkclean)
	$(ndkcleanobj)

ffi-clean:
	-cd $(LIMBO_JNI_ROOT)/libffi && $(MAKE) distclean
	-cd $(LIMBO_JNI_ROOT)/libffi && $(MAKE) clean

ffi-config: ndk
	cd $(LIMBO_JNI_ROOT)/libffi && ./configure --host=$(GNU_HOST) $(LIBFFI_DEBUG) \
	CFLAGS="$(ARCH_CFLAGS) -I$(NDK_INCLUDE)  " \
	LDFLAGS="$(ARCH_LD_FLAGS) $(SYS_ROOT) -nostdlib -fno-unwind-tables "

ffi:
	cd $(LIMBO_JNI_ROOT)/libffi && $(MAKE) -j $(BUILD_THREADS) \
	CFLAGS="$(ARCH_CFLAGS) -I$(NDK_INCLUDE) " \
	LDFLAGS="$(ARCH_LD_FLAGS) $(SYS_ROOT) -nostdlib -fno-unwind-tables "
	$(INSTALL_FFI_LIBS)

glib-config: ndk ffi
	cd $(LIMBO_JNI_ROOT)/glib && NOCONFIGURE=1 ./autogen.sh
	cd $(LIMBO_JNI_ROOT)/glib && cp -rf $(LIMBO_JNI_ROOT)/patches/android.cache . \
	&& chmod a-w android.cache && ./configure --host=$(GNU_HOST) --cache-file=android.cache \
	--disable-libmount  --disable-dtrace  --disable-systemtap --with-pcre=internal \
	--with-sysroot=$(SYSROOT) $(LIBGLIB_DEBUG) \
	CFLAGS="$(ARCH_CFLAGS) -I$(NDK_INCLUDE) \
	-I${SYSROOT}/usr/include $(SYS_ROOT) -I${NDK_ROOT}/sources/android/support/include \
	-I${LIMBO_JNI_ROOT}/compat -I${LIMBO_JNI_ROOT}/compat/intl " \
	LDFLAGS="$(ARCH_LD_FLAGS) $(SYS_ROOT) -nostdlib \
	-L${LIMBO_JNI_ROOT}/../obj/local/$(APP_ABI) -lc -lcompat-intl -lcompat-iconv -llog -lcompat-limbo " \
	LIBFFI_CFLAGS="-I${LIMBO_JNI_ROOT}/libffi/$(GNU_HOST)/include " \
	LIBFFI_LIBS="${LIMBO_JNI_ROOT}/../obj/local/$(APP_ABI)/libffi.a"

glib: ffi
	cd $(LIMBO_JNI_ROOT)/glib/glib && $(MAKE) -j $(BUILD_THREADS) CFLAGS="$(ARCH_CFLAGS) \
	-I$(NDK_INCLUDE)  -I${SYSROOT}/usr/include $(SYS_ROOT) \
	-I${NDK_ROOT}/sources/android/support/include -I${LIMBO_JNI_ROOT}/compat \
	-I${LIMBO_JNI_ROOT}/compat/intl " \
	LDFLAGS="$(ARCH_LD_FLAGS) $(SYS_ROOT) -nostdlib  -L${LIMBO_JNI_ROOT}/../obj/local/$(APP_ABI) \
	-lc -lcompat-intl -lcompat-iconv -llog -lcompat-limbo " \
	LIBFFI_CFLAGS="-I${LIMBO_JNI_ROOT}/libffi/$(GNU_HOST)/include " \
	LIBFFI_LIBS="${LIMBO_JNI_ROOT}/libffi/$(GNU_HOST)/.libs/libffi.a "
	$(INSTALL_GLIB_LIBS)

glib-clean:
	-find $(LIMBO_JNI_ROOT)/glib/ -name "*.lo" -exec rm -rf {} \;
	-cd $(LIMBO_JNI_ROOT)/glib && $(MAKE) distclean
	-cd $(LIMBO_JNI_ROOT)/glib && $(MAKE) clean
	-rm -rf $(LIMBO_JNI_ROOT)/glib/android.cache

pixman-config: ndk
	cd $(LIMBO_JNI_ROOT)/pixman && ./configure --host=$(GNU_HOST) --with-sysroot=$(SYSROOT) \
	--disable-mmx --disable-arm-iwmmxt --disable-arm-iwmmxt2 $(LIBPIXMAN_DEBUG) \
	CFLAGS="$(ARCH_CFLAGS) -I$(NDK_INCLUDE) \
	-I${SYSROOT}/usr/include $(SYS_ROOT) -I${NDK_ROOT}/sources/android/support/include \
	-I${NDK_ROOT}/sources/android/cpufeatures -I${LIMBO_JNI_ROOT}/compat " \
	LDFLAGS="$(ARCH_LD_FLAGS) $(SYS_ROOT) \
	-nostdlib -L${LIMBO_JNI_ROOT}/../obj/local/$(APP_ABI) -lc -llog \
	-lcompat-limbo "

pixman:
	cd $(LIMBO_JNI_ROOT)/pixman && $(MAKE) -j $(BUILD_THREADS) \
	CFLAGS="$(ARCH_CFLAGS) -I$(NDK_INCLUDE)  -I${SYSROOT}/usr/include \
	$(SYS_ROOT) -I${NDK_ROOT}/sources/android/support/include \
	-I${NDK_ROOT}/sources/android/cpufeatures -I${LIMBO_JNI_ROOT}/compat " \
	LDFLAGS="$(ARCH_LD_FLAGS) $(SYS_ROOT) -nostdlib  \
	-L${LIMBO_JNI_ROOT}/../obj/local/$(APP_ABI) -lc -llog \
	-lcompat-limbo "
	$(INSTALL_PIXMAN_LIBS)

pixman-clean:
	-find $(LIMBO_JNI_ROOT)/pixman -name "*.lo" -exec rm -rf {} \;
	-cd $(LIMBO_JNI_ROOT)/pixman && $(MAKE) distclean
	-cd $(LIMBO_JNI_ROOT)/pixman && $(MAKE) clean

qemu: ndk ffi glib pixman
	cd $(LIMBO_JNI_ROOT)/qemu && $(MAKE) all -j $(BUILD_THREADS)

qemu-config: glib pixman
	$(MAKE) -f $(LIMBO_JNI_ROOT)/android-qemu-config.mak

qemu-clean:
	-cd $(LIMBO_JNI_ROOT)/qemu && $(MAKE) distclean
	-cd $(LIMBO_JNI_ROOT)/qemu && $(MAKE) clean

config: ffi-config glib-config pixman-config qemu-config

config-debug:
	$(MAKE) config NDK_DEBUG=1

build: qemu
	$(INSTALL_LIBS)
	$(RENAME_TUNCTL)
	$(RENAME_PARPROUTED)
	$(COPY_ROMS)
	$(COPY_KEYMAPS)
	$(COPY_RESOLV_CONF)

debug:
	$(MAKE) build -j $(BUILD_THREADS) NDK_DEBUG=1

release:
	$(MAKE) build -j $(BUILD_THREADS)
	$(STRIP_LIBS)
	$(REMOVE_GDBSETUP)
	$(REMOVE_GDBSERVER)

install-x86:
	$(SYNC_LIBS)
	$(SYNC_LIBS_X86)
	$(COPY_ASSETS_X86)

install-arm:
	$(SYNC_LIBS)
	$(SYNC_LIBS_ARM)
	$(COPY_ASSETS_ARM)

install-sparc:
	$(SYNC_LIBS)
	$(SYNC_LIBS_SPARC)
	$(COPY_ASSETS_SPARC)

install-ppc:
	$(SYNC_LIBS)
	$(SYNC_LIBS_PPC)
	$(COPY_ASSETS_PPC)

install-all: install-x86 install-arm install-ppc install-sparc

ndk-gdb:
	-echo $(APP_ABI)
	$(COPY_GDB_SETUP_LIBS)
	$(ndkgdb) --attach $(PKG_NAME) --verbose -t

distclean: qemu-clean ffi-clean glib-clean pixman-clean

clean: distclean ndk-clean
	-find . -name "*.d" -exec rm -rf {} \;
	-find . -name "*.a" -exec rm -rf {} \;
	-find . -name "*.o" -exec rm -rf {} \;
	-find . -name "*.so" -exec rm -rf {} \;
	$(MAKE) createdirs
	
