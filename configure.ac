-*- mode: m4 -*-
AC_PREREQ(2.52)

# The version of python used is determined by the executable pointed to by
# the PYTHON environment variable.  For instance if your system installs
# Python 3 as python3 to configure to compile pygobject under Python 3
# you would do this:
# $> PYTHON=python3 ./configure
m4_define(python_min_ver, 2.5.2)
m4_define(python3_min_ver, 3.1)

dnl the pygobject version number
m4_define(pygobject_major_version, 3)
m4_define(pygobject_minor_version, 2)
m4_define(pygobject_micro_version, 2)
m4_define(pygobject_version, pygobject_major_version.pygobject_minor_version.pygobject_micro_version)

dnl versions of packages we require ...
m4_define(introspection_required_version, 1.31.20)
m4_define(py2cairo_required_version, 1.2.0)
m4_define(py3cairo_required_version, 1.10.0)
m4_define(glib_required_version, 2.31.0)
m4_define(gio_required_version, 2.31.0)

AC_INIT(pygobject, pygobject_version,
        [http://bugzilla.gnome.org/enter_bug.cgi?product=pygobject])
AC_SUBST(ACLOCAL_AMFLAGS, "-I m4 -I .")
AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_SRCDIR([gi/gimodule.c])

AC_DEFINE(PYGOBJECT_MAJOR_VERSION, pygobject_major_version, [pygobject major version])
AC_SUBST(PYGOBJECT_MAJOR_VERSION, pygobject_major_version)
AC_DEFINE(PYGOBJECT_MINOR_VERSION, pygobject_minor_version, [pygobject minor version])
AC_SUBST(PYGOBJECT_MINOR_VERSION, pygobject_minor_version)
AC_DEFINE(PYGOBJECT_MICRO_VERSION, pygobject_micro_version, [pygobject micro version])
AC_SUBST(PYGOBJECT_MICRO_VERSION, pygobject_micro_version)

AM_CONFIG_HEADER(config.h)
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES(yes)])
AM_INIT_AUTOMAKE([1.11 foreign no-dist-gzip dist-xz])
AM_MAINTAINER_MODE([enable])

dnl put the ACLOCAL flags in the makefile
ACLOCAL="$ACLOCAL $ACLOCAL_FLAGS"

AC_CANONICAL_BUILD
AC_CANONICAL_HOST
AC_MSG_CHECKING([for some Win32 platform])
case "$host" in
  *-*-mingw*|*-*-cygwin*)
    platform_win32=yes
    ;;
  *)
    platform_win32=no
    ;;
esac
AC_MSG_RESULT([$platform_win32])
AM_CONDITIONAL(PLATFORM_WIN32, test "$platform_win32" = "yes")

AC_MSG_CHECKING([for native Win32])
case "$host" in
  *-*-mingw*)
    os_win32=yes
    ;;
  *)
    os_win32=no
    ;;
esac
AC_MSG_RESULT([$os_win32])
AM_CONDITIONAL(OS_WIN32, test "$os_win32" = "yes")

AC_DISABLE_STATIC

dnl XXXX hack to kill off all the libtool tags ...
dnl it isn't like we are using C++ or Fortran.
dnl (copied from libglade/configure.in)
m4_define([_LT_AC_TAGCONFIG],[])

AC_LIBTOOL_WIN32_DLL
AC_PROG_LIBTOOL
dnl when using libtool 2.x create libtool early, because it's used in configure
m4_ifdef([LT_OUTPUT], [LT_OUTPUT])
AC_ISC_POSIX
AC_PROG_CC
AM_PROG_CC_STDC
AM_PROG_CC_C_O

# check that we have the minimum version of python necisary to build
JD_PATH_PYTHON(python_min_ver)

# check if we are building for python 3
JD_PYTHON_CHECK_VERSION([$PYTHON], [3.0],
			 build_py3k=true,
			 build_py3k=false)

# if building for python 3 make sure we have the minimum version supported
if test $build_py3k = true ; then
  AC_MSG_CHECKING([for $PYTHON >=] python3_min_ver)
  JD_PYTHON_CHECK_VERSION([$PYTHON], python3_min_ver,
			  [AC_MSG_RESULT(yes)],
			  [AC_MSG_ERROR(too old)])
fi

AM_CHECK_PYTHON_HEADERS(, AC_MSG_ERROR([Python headers not found]))
if test "x$os_win32" = "xyes"; then
  AM_CHECK_PYTHON_LIBS(, AC_MSG_ERROR([Python libs not found. Windows requires Python modules to be explicitly linked to libpython.]))
fi

AC_MSG_CHECKING([for PySignal_SetWakeupFd in Python.h])
old_CPPFLAGS=$CPPFLAGS
CPPFLAGS="-Wall -Werror $PYTHON_INCLUDES"
AC_TRY_COMPILE([#include <Python.h>],
  [PySignal_SetWakeupFd(0);],
  setwakeupfd_ok=yes,
  setwakeupfd_ok=no)
AC_MSG_RESULT($setwakeupfd_ok)
if test "x$setwakeupfd_ok" != xno; then
  AC_DEFINE(HAVE_PYSIGNAL_SETWAKEUPFD, 1,
                [Define to 1 if PySignal_SetWakeupFd is available])
fi
CPPFLAGS=$old_CPPFLAGS

PLATFORM=`$PYTHON -c "import sys; from distutils import util; sys.stdout.write(util.get_platform())"`
AC_SUBST(PLATFORM)

AC_ARG_ENABLE(thread,
  AC_HELP_STRING([--disable-thread], [Disable pygobject threading support]),,
  enable_thread=yes)

AM_CHECK_PYMOD(thread,,,enable_thread=check_for_py3_module)
if test "x$enable_thread" = xcheck_for_py3_module; then
  dnl Python 3 uses the _thread module so check for that
  AM_CHECK_PYMOD(_thread,,,enable_thread=no)
fi

AC_MSG_CHECKING(whether to enable threading in pygobject)
if test "x$enable_thread" != xno; then
  extra_mods=gthread
  THREADING_CFLAGS=
  AC_MSG_RESULT(yes)
else
  extra_mods=
  THREADING_CFLAGS="-DDISABLE_THREADING"
  AC_MSG_RESULT(no)
fi
AC_SUBST(THREADING_CFLAGS)
CPPFLAGS="${CPPFLAGS} $THREADING_CFLAGS"

dnl get rid of the -export-dynamic stuff from the configure flags ...
export_dynamic=`(./libtool --config; echo eval echo \\$export_dynamic_flag_spec) | sh`

dnl glib
AM_PATH_GLIB_2_0(glib_required_version,,[AC_MSG_ERROR(maybe you want the pygobject-2-4 branch?)],gobject $extra_mods)
if test -n "$export_dynamic"; then
  GLIB_LIBS=`echo $GLIB_LIBS | sed -e "s/$export_dynamic//"`
fi
PYTHON_BASENAME=`basename $PYTHON`
AC_SUBST([PYTHON_BASENAME])

AS_AC_EXPAND(DATADIR, $datadir)

dnl libffi
AC_MSG_CHECKING(for ffi)
AC_ARG_WITH(ffi,
  AC_HELP_STRING([--without-ffi], [Disable libffi support]),
  with_libffi=$withval,
  with_libffi=auto)
if test x"$with_libffi" = xno ; then
  have_libffi=false
else
  PKG_CHECK_MODULES(FFI, libffi >= 3.0, have_libffi=yes, have_libffi=no)
fi
if test x"$with_libffi" = xyes && test x"$have_libffi" = xno ; then
  AC_MSG_ERROR([ffi requested, but not found])
fi
if test x"$have_libffi" = xyes; then
  LIBFFI_PC=libffi
fi
AM_CONDITIONAL(HAVE_LIBFFI, test "$have_libffi" = "yes")
AC_SUBST(FFI_CFLAGS)
AC_SUBST(FFI_LIBS)
AC_SUBST(LIBFFI_PC)

dnl gio
PKG_CHECK_MODULES(GIO, gio-2.0 >= gio_required_version)

AC_ARG_ENABLE(cairo,
    AC_HELP_STRING([--enable-cairo], [Enable Cairo bindings using introspection information]),
    enable_cairo=$enableval,
    enable_cairo=yes)

PKG_CHECK_MODULES(GI,
    glib-2.0 >= glib_required_version
    gobject-introspection-1.0 >= introspection_required_version
)

GI_DATADIR=$($PKG_CONFIG --variable=gidatadir gobject-introspection-1.0)
AC_SUBST(GI_DATADIR)

if test "$enable_cairo" != no; then
    if test $build_py3k = true; then
        PKG_CHECK_MODULES(PYCAIRO,
            py3cairo >= py3cairo_required_version
        )
    else
        PKG_CHECK_MODULES(PYCAIRO,
            pycairo >= py2cairo_required_version
        )
    fi
fi
AM_CONDITIONAL(ENABLE_CAIRO, test "$enable_cairo" = "yes")

INTROSPECTION_SCANNER=`$PKG_CONFIG --variable=g_ir_scanner gobject-introspection-1.0`
INTROSPECTION_COMPILER=`$PKG_CONFIG --variable=g_ir_compiler gobject-introspection-1.0`

AC_SUBST(INTROSPECTION_SCANNER)
AC_SUBST(INTROSPECTION_COMPILER)

dnl add required cflags ...
if test "x$GCC" = "xyes"; then
  JH_ADD_CFLAG([-Wall])
  JH_ADD_CFLAG([-fno-strict-aliasing])

  case $host_os in
  solaris*)
    ;;
  *)
    JH_ADD_CFLAG([-std=c9x])
    ;;
  esac

fi

AC_CONFIG_FILES(
  Makefile
  pygobject-3.0.pc
  pygobject-3.0-uninstalled.pc
  gi/Makefile
  gi/repository/Makefile
  gi/overrides/Makefile
  gi/_glib/Makefile
  gi/_gobject/Makefile
  examples/Makefile
  tests/Makefile
  PKG-INFO)
AC_OUTPUT

echo
echo "libffi support: $have_libffi"
echo
