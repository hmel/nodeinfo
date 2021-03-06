dnl require autoconf 2.60 (AS_ECHO/AS_ECHO_N)
AC_PREREQ([2.60])
define(_CLIENT_VERSION_MAJOR, 0)
define(_CLIENT_VERSION_MINOR, 1)
define(_CLIENT_VERSION_REVISION, 1)
define(_CLIENT_VERSION_BUILD, 0)
define(_CLIENT_VERSION_RC, 0)
define(_CLIENT_VERSION_IS_RELEASE, false)
define(_COPYRIGHT_YEAR, 2019)
define(_COPYRIGHT_HOLDERS,[The %s developers])
define(_COPYRIGHT_HOLDERS_SUBSTITUTION,[[NodeInfo]])
AC_INIT([Bitcoin Core],m4_join([.], _CLIENT_VERSION_MAJOR, _CLIENT_VERSION_MINOR, _CLIENT_VERSION_REVISION, m4_if(_CLIENT_VERSION_BUILD, [0], [], _CLIENT_VERSION_BUILD))m4_if(_CLIENT_VERSION_RC, [0], [], [rc]_CLIENT_VERSION_RC),[https://github.com/jonasschnelli/nodeinfo/issues],[bitcoin],[https://jonasschnelli.dev/])
AC_CONFIG_SRCDIR([src/main.cpp])
AC_CONFIG_HEADERS([src/config/bitcoin-config.h])
AC_CONFIG_AUX_DIR([build-aux])
AC_CONFIG_MACRO_DIR([build-aux/m4])

AC_CANONICAL_HOST

AH_TOP([#ifndef NODEINFO_CONFIG_H])
AH_TOP([#define NODEINFO_CONFIG_H])
AH_BOTTOM([#endif //NODEINFO_CONFIG_H])

dnl faketime breaks configure and is only needed for make. Disable it here.
unset FAKETIME

dnl Automake init set-up and checks
AM_INIT_AUTOMAKE([no-define subdir-objects foreign])

dnl faketime messes with timestamps and causes configure to be re-run.
dnl --disable-maintainer-mode can be used to bypass this.
AM_MAINTAINER_MODE([enable])

dnl make the compilation flags quiet unless V=1 is used
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])

dnl Compiler checks (here before libtool).
if test "x${CXXFLAGS+set}" = "xset"; then
  CXXFLAGS_overridden=yes
else
  CXXFLAGS_overridden=no
fi
AC_PROG_CXX

dnl By default, libtool for mingw refuses to link static libs into a dll for
dnl fear of mixing pic/non-pic objects, and import/export complications. Since
dnl we have those under control, re-enable that functionality.
case $host in
  *mingw*)
     lt_cv_deplibs_check_method="pass_all"
  ;;
esac
dnl Require C++11 compiler (no GNU extensions)
AX_CXX_COMPILE_STDCXX([11], [noext], [mandatory], [nodefault])
dnl Check if -latomic is required for <std::atomic>
CHECK_ATOMIC

dnl Unless the user specified OBJCXX, force it to be the same as CXX. This ensures
dnl that we get the same -std flags for both.
m4_ifdef([AC_PROG_OBJCXX],[
if test "x${OBJCXX+set}" = "x"; then
  OBJCXX="${CXX}"
fi
AC_PROG_OBJCXX
])

dnl Libtool init checks.
LT_INIT([pic-only])

dnl Check/return PATH for base programs.
AC_PATH_TOOL(AR, ar)
AC_PATH_TOOL(RANLIB, ranlib)

AC_LANG_PUSH([C++])
AX_CHECK_COMPILE_FLAG([-Werror],[CXXFLAG_WERROR="-Werror"],[CXXFLAG_WERROR=""])

if test "x$enable_debug" = xyes; then
  # Clear default -g -O2 flags
  if test "x$CXXFLAGS_overridden" = xno; then
	CXXFLAGS=""
  fi
  # Prefer -Og, fall back to -O0 if that is unavailable.
  AX_CHECK_COMPILE_FLAG(
    [-Og],
    [[DEBUG_CXXFLAGS="$DEBUG_CXXFLAGS -Og"]],
    [AX_CHECK_COMPILE_FLAG([-O0],[[DEBUG_CXXFLAGS="$DEBUG_CXXFLAGS -O0"]],,[[$CXXFLAG_WERROR]])],
    [[$CXXFLAG_WERROR]])

  # Prefer -g3, fall back to -g if that is unavailable.
  AX_CHECK_COMPILE_FLAG(
    [-g3],
    [[DEBUG_CXXFLAGS="$DEBUG_CXXFLAGS -g3"]],
    [AX_CHECK_COMPILE_FLAG([-g],[[DEBUG_CXXFLAGS="$DEBUG_CXXFLAGS -g"]],,[[$CXXFLAG_WERROR]])],
    [[$CXXFLAG_WERROR]])

  AX_CHECK_PREPROC_FLAG([-DDEBUG],[[DEBUG_CPPFLAGS="$DEBUG_CPPFLAGS -DDEBUG"]],,[[$CXXFLAG_WERROR]])
  AX_CHECK_PREPROC_FLAG([-DDEBUG_LOCKORDER],[[DEBUG_CPPFLAGS="$DEBUG_CPPFLAGS -DDEBUG_LOCKORDER"]],,[[$CXXFLAG_WERROR]])
  AX_CHECK_COMPILE_FLAG([-ftrapv],[DEBUG_CXXFLAGS="$DEBUG_CXXFLAGS -ftrapv"],,[[$CXXFLAG_WERROR]])
fi

if test "x$CXXFLAGS_overridden" = "xno"; then
  AX_CHECK_COMPILE_FLAG([-Wall],[WARN_CXXFLAGS="$WARN_CXXFLAGS -Wall"],,[[$CXXFLAG_WERROR]])
  AX_CHECK_COMPILE_FLAG([-Wextra],[WARN_CXXFLAGS="$WARN_CXXFLAGS -Wextra"],,[[$CXXFLAG_WERROR]])
  AX_CHECK_COMPILE_FLAG([-Wformat],[WARN_CXXFLAGS="$WARN_CXXFLAGS -Wformat"],,[[$CXXFLAG_WERROR]])
  AX_CHECK_COMPILE_FLAG([-Wvla],[WARN_CXXFLAGS="$WARN_CXXFLAGS -Wvla"],,[[$CXXFLAG_WERROR]])
  AX_CHECK_COMPILE_FLAG([-Wformat-security],[WARN_CXXFLAGS="$WARN_CXXFLAGS -Wformat-security"],,[[$CXXFLAG_WERROR]])
  AX_CHECK_COMPILE_FLAG([-Wthread-safety-analysis],[WARN_CXXFLAGS="$WARN_CXXFLAGS -Wthread-safety-analysis"],,[[$CXXFLAG_WERROR]])
  AX_CHECK_COMPILE_FLAG([-Wrange-loop-analysis],[WARN_CXXFLAGS="$WARN_CXXFLAGS -Wrange-loop-analysis"],,[[$CXXFLAG_WERROR]])
  AX_CHECK_COMPILE_FLAG([-Wredundant-decls],[WARN_CXXFLAGS="$WARN_CXXFLAGS -Wredundant-decls"],,[[$CXXFLAG_WERROR]])

  ## Some compilers (gcc) ignore unknown -Wno-* options, but warn about all
  ## unknown options if any other warning is produced. Test the -Wfoo case, and
  ## set the -Wno-foo case if it works.
  AX_CHECK_COMPILE_FLAG([-Wunused-parameter],[NOWARN_CXXFLAGS="$NOWARN_CXXFLAGS -Wno-unused-parameter"],,[[$CXXFLAG_WERROR]])
  AX_CHECK_COMPILE_FLAG([-Wself-assign],[NOWARN_CXXFLAGS="$NOWARN_CXXFLAGS -Wno-self-assign"],,[[$CXXFLAG_WERROR]])
  AX_CHECK_COMPILE_FLAG([-Wunused-local-typedef],[NOWARN_CXXFLAGS="$NOWARN_CXXFLAGS -Wno-unused-local-typedef"],,[[$CXXFLAG_WERROR]])
  AX_CHECK_COMPILE_FLAG([-Wdeprecated-register],[NOWARN_CXXFLAGS="$NOWARN_CXXFLAGS -Wno-deprecated-register"],,[[$CXXFLAG_WERROR]])
  AX_CHECK_COMPILE_FLAG([-Wimplicit-fallthrough],[NOWARN_CXXFLAGS="$NOWARN_CXXFLAGS -Wno-implicit-fallthrough"],,[[$CXXFLAG_WERROR]])
fi

use_pkgconfig=yes
case $host in
  *mingw*)

     #pkgconfig does more harm than good with MinGW
     use_pkgconfig=no

     TARGET_OS=windows
     AC_CHECK_LIB([mingwthrd],      [main],, AC_MSG_ERROR(libmingwthrd missing))
     AC_CHECK_LIB([kernel32],      [main],, AC_MSG_ERROR(libkernel32 missing))
     AC_CHECK_LIB([user32],      [main],, AC_MSG_ERROR(libuser32 missing))
     AC_CHECK_LIB([gdi32],      [main],, AC_MSG_ERROR(libgdi32 missing))
     AC_CHECK_LIB([comdlg32],      [main],, AC_MSG_ERROR(libcomdlg32 missing))
     AC_CHECK_LIB([winspool],      [main],, AC_MSG_ERROR(libwinspool missing))
     AC_CHECK_LIB([winmm],      [main],, AC_MSG_ERROR(libwinmm missing))
     AC_CHECK_LIB([shell32],      [main],, AC_MSG_ERROR(libshell32 missing))
     AC_CHECK_LIB([comctl32],      [main],, AC_MSG_ERROR(libcomctl32 missing))
     AC_CHECK_LIB([ole32],      [main],, AC_MSG_ERROR(libole32 missing))
     AC_CHECK_LIB([oleaut32],      [main],, AC_MSG_ERROR(liboleaut32 missing))
     AC_CHECK_LIB([uuid],      [main],, AC_MSG_ERROR(libuuid missing))
     AC_CHECK_LIB([rpcrt4],      [main],, AC_MSG_ERROR(librpcrt4 missing))
     AC_CHECK_LIB([advapi32],      [main],, AC_MSG_ERROR(libadvapi32 missing))
     AC_CHECK_LIB([ws2_32],      [main],, AC_MSG_ERROR(libws2_32 missing))
     AC_CHECK_LIB([mswsock],      [main],, AC_MSG_ERROR(libmswsock missing))
     AC_CHECK_LIB([shlwapi],      [main],, AC_MSG_ERROR(libshlwapi missing))
     AC_CHECK_LIB([iphlpapi],      [main],, AC_MSG_ERROR(libiphlpapi missing))
     AC_CHECK_LIB([crypt32],      [main],, AC_MSG_ERROR(libcrypt32 missing))

     # -static is interpreted by libtool, where it has a different meaning.
     # In libtool-speak, it's -all-static.
     AX_CHECK_LINK_FLAG([[-static]],[LIBTOOL_APP_LDFLAGS="$LIBTOOL_APP_LDFLAGS -all-static"])

     AC_PATH_TOOL(WINDRES, windres, none)
     if test x$WINDRES = xnone; then
       AC_MSG_ERROR("windres not found")
     fi

     CPPFLAGS="$CPPFLAGS -D_MT -DWIN32 -D_WINDOWS -DBOOST_THREAD_USE_LIB -D_WIN32_WINNT=0x0601"
     LEVELDB_TARGET_FLAGS="-DOS_WINDOWS"
     if test "x$CXXFLAGS_overridden" = "xno"; then
       CXXFLAGS="$CXXFLAGS -w"
     fi
     case $host in
       i?86-*) WINDOWS_BITS=32 ;;
       x86_64-*) WINDOWS_BITS=64 ;;
       *) AC_MSG_ERROR("Could not determine win32/win64 for installer") ;;
     esac
     AC_SUBST(WINDOWS_BITS)

     dnl libtool insists upon adding -nostdlib and a list of objects/libs to link against.
     dnl That breaks our ability to build dll's with static libgcc/libstdc++/libssp. Override
     dnl its command here, with the predeps/postdeps removed, and -static inserted. Postdeps are
     dnl also overridden to prevent their insertion later.
     dnl This should only affect dll's.
     archive_cmds_CXX="\$CC -shared \$libobjs \$deplibs \$compiler_flags -static -o \$output_objdir/\$soname \${wl}--enable-auto-image-base -Xlinker --out-implib -Xlinker \$lib"
     postdeps_CXX=

     ;;
  *darwin*)
     TARGET_OS=darwin
     LEVELDB_TARGET_FLAGS="-DOS_MACOSX"
     if  test x$cross_compiling != xyes; then
       BUILD_OS=darwin
       AC_PATH_PROGS([RSVG_CONVERT], [rsvg-convert rsvg],rsvg-convert)
       AC_CHECK_PROG([BREW],brew, brew)
       if test x$BREW = xbrew; then
         dnl These Homebrew packages may be keg-only, meaning that they won't be found
         dnl in expected paths because they may conflict with system files. Ask
         dnl Homebrew where each one is located, then adjust paths accordingly.
         dnl It's safe to add these paths even if the functionality is disabled by
         dnl the user (--without-wallet or --without-gui for example).

         qt5_prefix=`$BREW --prefix qt5 2>/dev/null`
         if test x$qt5_prefix != x; then
           PKG_CONFIG_PATH="$qt5_prefix/lib/pkgconfig:$PKG_CONFIG_PATH"
           export PKG_CONFIG_PATH
         fi
       fi
     else
       case $build_os in
         *darwin*)
           BUILD_OS=darwin
           ;;
         *)
           AC_PATH_TOOL([INSTALLNAMETOOL], [install_name_tool], install_name_tool)
           AC_PATH_TOOL([OTOOL], [otool], otool)
           AC_PATH_PROGS([GENISOIMAGE], [genisoimage mkisofs],genisoimage)
           AC_PATH_PROGS([RSVG_CONVERT], [rsvg-convert rsvg],rsvg-convert)
           AC_PATH_PROGS([IMAGEMAGICK_CONVERT], [convert],convert)
           AC_PATH_PROGS([TIFFCP], [tiffcp],tiffcp)

           dnl libtool will try to strip the static lib, which is a problem for
           dnl cross-builds because strip attempts to call a hard-coded ld,
           dnl which may not exist in the path. Stripping the .a is not
           dnl necessary, so just disable it.
           old_striplib=
           ;;
       esac
     fi

     AX_CHECK_LINK_FLAG([[-Wl,-headerpad_max_install_names]], [LDFLAGS="$LDFLAGS -Wl,-headerpad_max_install_names"])
     CPPFLAGS="$CPPFLAGS -DMAC_OSX"
     OBJCXXFLAGS="$CXXFLAGS"
     ;;
   *android*)
     dnl make sure android stays above linux for hosts like *linux-android*
     LEVELDB_TARGET_FLAGS="-DOS_ANDROID"
     ;;
   *linux*)
     TARGET_OS=linux
     LEVELDB_TARGET_FLAGS="-DOS_LINUX"
     ;;
   *kfreebsd*)
     LEVELDB_TARGET_FLAGS="-DOS_KFREEBSD"
     ;;
   *freebsd*)
     LEVELDB_TARGET_FLAGS="-DOS_FREEBSD"
     ;;
   *openbsd*)
     LEVELDB_TARGET_FLAGS="-DOS_OPENBSD"
     ;;
   *netbsd*)
     LEVELDB_TARGET_FLAGS="-DOS_NETBSD"
     ;;
   *dragonfly*)
     LEVELDB_TARGET_FLAGS="-DOS_DRAGONFLYBSD"
     ;;
   *solaris*)
     LEVELDB_TARGET_FLAGS="-DOS_SOLARIS"
     ;;
   *hpux*)
     LEVELDB_TARGET_FLAGS="-DOS_HPUX"
     ;;
   *)
     AC_MSG_ERROR(Cannot build leveldb for $host. Please file a bug report.)
     ;;
esac

if test x$use_pkgconfig = xyes; then
  m4_ifndef([PKG_PROG_PKG_CONFIG], [AC_MSG_ERROR(PKG_PROG_PKG_CONFIG macro not found. Please install pkg-config and re-run autogen.sh.)])
  m4_ifdef([PKG_PROG_PKG_CONFIG], [
  PKG_PROG_PKG_CONFIG
  if test x"$PKG_CONFIG" = "x"; then
    AC_MSG_ERROR(pkg-config not found.)
  fi
  ])
fi

if test x$TARGET_OS != xwindows; then
  # All windows code is PIC, forcing it on just adds useless compile warnings
  AX_CHECK_COMPILE_FLAG([-fPIC],[PIC_FLAGS="-fPIC"])
fi

if test x$use_hardening != xno; then
  use_hardening=yes
  AX_CHECK_COMPILE_FLAG([-Wstack-protector],[HARDENED_CXXFLAGS="$HARDENED_CXXFLAGS -Wstack-protector"])
  AX_CHECK_COMPILE_FLAG([-fstack-protector-all],[HARDENED_CXXFLAGS="$HARDENED_CXXFLAGS -fstack-protector-all"])

  AX_CHECK_PREPROC_FLAG([-D_FORTIFY_SOURCE=2],[
    AX_CHECK_PREPROC_FLAG([-U_FORTIFY_SOURCE],[
      HARDENED_CPPFLAGS="$HARDENED_CPPFLAGS -U_FORTIFY_SOURCE"
    ])
    HARDENED_CPPFLAGS="$HARDENED_CPPFLAGS -D_FORTIFY_SOURCE=2"
  ])

  AX_CHECK_LINK_FLAG([[-Wl,--dynamicbase]], [HARDENED_LDFLAGS="$HARDENED_LDFLAGS -Wl,--dynamicbase"])
  AX_CHECK_LINK_FLAG([[-Wl,--nxcompat]], [HARDENED_LDFLAGS="$HARDENED_LDFLAGS -Wl,--nxcompat"])
  AX_CHECK_LINK_FLAG([[-Wl,--high-entropy-va]], [HARDENED_LDFLAGS="$HARDENED_LDFLAGS -Wl,--high-entropy-va"])
  AX_CHECK_LINK_FLAG([[-Wl,-z,relro]], [HARDENED_LDFLAGS="$HARDENED_LDFLAGS -Wl,-z,relro"])
  AX_CHECK_LINK_FLAG([[-Wl,-z,now]], [HARDENED_LDFLAGS="$HARDENED_LDFLAGS -Wl,-z,now"])
  AX_CHECK_LINK_FLAG([[-fPIE -pie]], [PIE_FLAGS="-fPIE"; HARDENED_LDFLAGS="$HARDENED_LDFLAGS -pie"],, [[$CXXFLAG_WERROR]])

  case $host in
    *mingw*)
       AC_CHECK_LIB([ssp],      [main],, AC_MSG_ERROR(libssp missing))
    ;;
  esac
fi

dnl this flag screws up non-darwin gcc even when the check fails. special-case it.
if test x$TARGET_OS = xdarwin; then
  AX_CHECK_LINK_FLAG([[-Wl,-dead_strip]], [LDFLAGS="$LDFLAGS -Wl,-dead_strip"])
fi

AC_CHECK_HEADERS([endian.h sys/endian.h byteswap.h stdio.h stdlib.h unistd.h strings.h sys/types.h sys/stat.h sys/select.h sys/prctl.h])

AC_CHECK_DECLS([getifaddrs, freeifaddrs],,,
    [#include <sys/types.h>
    #include <ifaddrs.h>]
)
AC_CHECK_DECLS([strnlen])

# Check for daemon(3), unrelated to --with-daemon (although used by it)
AC_CHECK_DECLS([daemon])

AC_CHECK_DECLS([le16toh, le32toh, le64toh, htole16, htole32, htole64, be16toh, be32toh, be64toh, htobe16, htobe32, htobe64],,,
		[#if HAVE_ENDIAN_H
                 #include <endian.h>
                 #elif HAVE_SYS_ENDIAN_H
                 #include <sys/endian.h>
                 #endif])

AC_CHECK_DECLS([bswap_16, bswap_32, bswap_64],,,
		[#if HAVE_BYTESWAP_H
                 #include <byteswap.h>
                 #endif])

AC_CHECK_DECLS([__builtin_clz, __builtin_clzl, __builtin_clzll])

dnl Check for malloc_info (for memory statistics information in getmemoryinfo)
AC_MSG_CHECKING(for getmemoryinfo)
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[#include <malloc.h>]],
 [[ int f = malloc_info(0, NULL); ]])],
 [ AC_MSG_RESULT(yes); AC_DEFINE(HAVE_MALLOC_INFO, 1,[Define this symbol if you have malloc_info]) ],
 [ AC_MSG_RESULT(no)]
)

dnl Check for mallopt(M_ARENA_MAX) (to set glibc arenas)
AC_MSG_CHECKING(for mallopt M_ARENA_MAX)
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[#include <malloc.h>]],
 [[ mallopt(M_ARENA_MAX, 1); ]])],
 [ AC_MSG_RESULT(yes); AC_DEFINE(HAVE_MALLOPT_ARENA_MAX, 1,[Define this symbol if you have mallopt with M_ARENA_MAX]) ],
 [ AC_MSG_RESULT(no)]
)

AC_MSG_CHECKING([for visibility attribute])
AC_LINK_IFELSE([AC_LANG_SOURCE([
  int foo_def( void ) __attribute__((visibility("default")));
  int main(){}
  ])],
  [
    AC_DEFINE(HAVE_VISIBILITY_ATTRIBUTE,1,[Define if the visibility attribute is supported.])
    AC_MSG_RESULT(yes)
  ],
  [
    AC_MSG_RESULT(no)
    if test x$use_reduce_exports = xyes; then
      AC_MSG_ERROR([Cannot find a working visibility attribute. Use --disable-reduce-exports.])
    fi
  ]
)

TEMP_LDFLAGS="$LDFLAGS"
LDFLAGS="$TEMP_LDFLAGS $PTHREAD_CFLAGS"
AC_MSG_CHECKING([for thread_local support])
AC_LINK_IFELSE([AC_LANG_SOURCE([
  #include <thread>
  static thread_local int foo = 0;
  static void run_thread() { foo++;}
  int main(){
  for(int i = 0; i < 10; i++) { std::thread(run_thread).detach();}
  return foo;
  }
  ])],
  [
    AC_DEFINE(HAVE_THREAD_LOCAL,1,[Define if thread_local is supported.])
    AC_MSG_RESULT(yes)
  ],
  [
    AC_MSG_RESULT(no)
  ]
)
LDFLAGS="$TEMP_LDFLAGS"

# Check for different ways of gathering OS randomness
AC_MSG_CHECKING(for Linux getrandom syscall)
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[#include <unistd.h>
  #include <sys/syscall.h>
  #include <linux/random.h>]],
 [[ syscall(SYS_getrandom, nullptr, 32, 0); ]])],
 [ AC_MSG_RESULT(yes); AC_DEFINE(HAVE_SYS_GETRANDOM, 1,[Define this symbol if the Linux getrandom system call is available]) ],
 [ AC_MSG_RESULT(no)]
)

AC_MSG_CHECKING(for getentropy)
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[#include <unistd.h>]],
 [[ getentropy(nullptr, 32) ]])],
 [ AC_MSG_RESULT(yes); AC_DEFINE(HAVE_GETENTROPY, 1,[Define this symbol if the BSD getentropy system call is available]) ],
 [ AC_MSG_RESULT(no)]
)

AC_MSG_CHECKING(for getentropy via random.h)
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[#include <unistd.h>
 #include <sys/random.h>]],
 [[ getentropy(nullptr, 32) ]])],
 [ AC_MSG_RESULT(yes); AC_DEFINE(HAVE_GETENTROPY_RAND, 1,[Define this symbol if the BSD getentropy system call is available with sys/random.h]) ],
 [ AC_MSG_RESULT(no)]
)

AC_MSG_CHECKING(for sysctl KERN_ARND)
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[#include <sys/types.h>
  #include <sys/sysctl.h>]],
 [[ static const int name[2] = {CTL_KERN, KERN_ARND};
    sysctl(name, 2, nullptr, nullptr, nullptr, 0); ]])],
 [ AC_MSG_RESULT(yes); AC_DEFINE(HAVE_SYSCTL_ARND, 1,[Define this symbol if the BSD sysctl(KERN_ARND) is available]) ],
 [ AC_MSG_RESULT(no)]
)

AC_MSG_CHECKING(for if type char equals int8_t)
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[#include <stdint.h>
  #include <type_traits>]],
 [[ static_assert(std::is_same<int8_t, char>::value, ""); ]])],
 [ AC_MSG_RESULT(yes); AC_DEFINE(CHAR_EQUALS_INT8, 1,[Define this symbol if type char equals int8_t]) ],
 [ AC_MSG_RESULT(no)]
)

# Check for reduced exports
if test x$use_reduce_exports = xyes; then
  AX_CHECK_COMPILE_FLAG([-fvisibility=hidden],[RE_CXXFLAGS="-fvisibility=hidden"],
  [AC_MSG_ERROR([Cannot set default symbol visibility. Use --disable-reduce-exports.])])
fi


BITCOIN_QT_INIT

dnl sets $bitcoin_enable_qt, $bitcoin_enable_qt_test, $bitcoin_enable_qt_dbus
BITCOIN_QT_CONFIGURE([$use_pkgconfig])

AM_CONDITIONAL([TARGET_DARWIN], [test x$TARGET_OS = xdarwin])
AM_CONDITIONAL([BUILD_DARWIN], [test x$BUILD_OS = xdarwin])
AM_CONDITIONAL([TARGET_WINDOWS], [test x$TARGET_OS = xwindows])
AM_CONDITIONAL([ENABLE_QT],[test x$bitcoin_enable_qt = xyes])

AC_DEFINE(CLIENT_VERSION_MAJOR, _CLIENT_VERSION_MAJOR, [Major version])
AC_DEFINE(CLIENT_VERSION_MINOR, _CLIENT_VERSION_MINOR, [Minor version])
AC_DEFINE(CLIENT_VERSION_REVISION, _CLIENT_VERSION_REVISION, [Build revision])
AC_DEFINE(CLIENT_VERSION_BUILD, _CLIENT_VERSION_BUILD, [Version Build])
AC_DEFINE(CLIENT_VERSION_IS_RELEASE, _CLIENT_VERSION_IS_RELEASE, [Version is release])
AC_DEFINE(COPYRIGHT_YEAR, _COPYRIGHT_YEAR, [Copyright year])
AC_DEFINE(COPYRIGHT_HOLDERS, "_COPYRIGHT_HOLDERS", [Copyright holder(s) before %s replacement])
AC_DEFINE(COPYRIGHT_HOLDERS_SUBSTITUTION, "_COPYRIGHT_HOLDERS_SUBSTITUTION", [Replacement for %s in copyright holders string])
define(_COPYRIGHT_HOLDERS_FINAL, [patsubst(_COPYRIGHT_HOLDERS, [%s], [_COPYRIGHT_HOLDERS_SUBSTITUTION])])
AC_DEFINE(COPYRIGHT_HOLDERS_FINAL, "_COPYRIGHT_HOLDERS_FINAL", [Copyright holder(s)])
AC_SUBST(CLIENT_VERSION_MAJOR, _CLIENT_VERSION_MAJOR)
AC_SUBST(CLIENT_VERSION_MINOR, _CLIENT_VERSION_MINOR)
AC_SUBST(CLIENT_VERSION_REVISION, _CLIENT_VERSION_REVISION)
AC_SUBST(CLIENT_VERSION_BUILD, _CLIENT_VERSION_BUILD)
AC_SUBST(CLIENT_VERSION_IS_RELEASE, _CLIENT_VERSION_IS_RELEASE)
AC_SUBST(COPYRIGHT_YEAR, _COPYRIGHT_YEAR)
AC_SUBST(COPYRIGHT_HOLDERS, "_COPYRIGHT_HOLDERS")
AC_SUBST(COPYRIGHT_HOLDERS_SUBSTITUTION, "_COPYRIGHT_HOLDERS_SUBSTITUTION")
AC_SUBST(COPYRIGHT_HOLDERS_FINAL, "_COPYRIGHT_HOLDERS_FINAL")

AC_SUBST(RELDFLAGS)
AC_SUBST(DEBUG_CPPFLAGS)
AC_SUBST(WARN_CXXFLAGS)
AC_SUBST(NOWARN_CXXFLAGS)
AC_SUBST(DEBUG_CXXFLAGS)
AC_SUBST(COMPAT_LDFLAGS)
AC_SUBST(ERROR_CXXFLAGS)
AC_SUBST(GPROF_CXXFLAGS)
AC_SUBST(GPROF_LDFLAGS)
AC_SUBST(HARDENED_CXXFLAGS)
AC_SUBST(HARDENED_CPPFLAGS)
AC_SUBST(HARDENED_LDFLAGS)
AC_SUBST(PIC_FLAGS)
AC_SUBST(PIE_FLAGS)
AC_CONFIG_FILES([Makefile])

AC_OUTPUT

echo "done"
echo
