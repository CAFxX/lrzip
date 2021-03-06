##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##
##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##
m4_define([v_maj], [0])
m4_define([v_min], [6])
m4_define([v_mic], [08])
##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##
m4_define([v_v], m4_join([], v_min, v_mic))
m4_define([v_ver], [v_maj.v_v])
m4_define([lt_rev], m4_eval(v_maj + v_min))
m4_define([lt_cur], v_mic)
m4_define([lt_age], v_min)
##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##
##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##


dnl Process this file with autoconf to produce a configure script.
AC_INIT([lrzip],[v_ver],[kernel@kolivas.org])
AC_PREREQ([2.59])
AC_CONFIG_SRCDIR([configure.ac])
AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_HEADERS([config.h])

AM_INIT_AUTOMAKE([1.6 dist-bzip2 foreign])
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])
AC_GNU_SOURCE

AC_PROG_LIBTOOL
##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##
##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##
m4_ifdef([v_rev], , [m4_define([v_rev], [0])])
m4_ifdef([v_rel], , [m4_define([v_rel], [])])
AC_DEFINE_UNQUOTED(LRZIP_MAJOR_VERSION, [v_maj], [Major version])
AC_DEFINE_UNQUOTED(LRZIP_MINOR_VERSION, [v_min], [Minor version])
AC_DEFINE_UNQUOTED(LRZIP_MINOR_SUBVERSION, [v_mic], [Micro version])
version_info="lt_rev:lt_cur:lt_age"
release_info="v_rel"
AC_SUBST(version_info)
AC_SUBST(release_info)
##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##
##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##
VMAJ=v_maj
AC_SUBST(VMAJ)

AC_CANONICAL_BUILD
AC_CANONICAL_HOST

dnl Checks for programs.
AC_PROG_CC
AC_PROG_CXX
PKG_PROG_PKG_CONFIG
AC_PROG_INSTALL
AC_PROG_LN_S
AC_SUBST(SHELL)
AC_SYS_LARGEFILE

AC_CHECK_PROG([HAVE_POD2MAN], [pod2man], [yes])
AS_IF([test "$HAVE_POD2MAN" != "yes"],
	AC_MSG_FAILURE([pod2man is needed to generate manual from POD]))


 AC_ARG_ENABLE(
		asm,
		[AC_HELP_STRING([--enable-asm],[Enable native Assembly code])],
		ASM=$enableval,
		ASM=yes
             )
if test x"$ASM" = xyes; then
	AC_CHECK_PROG( ASM_PROG, nasm, yes, no )
	if test x"$ASM_PROG" = x"no "; then
		ASM=no
	fi
fi

static=no
AC_ARG_ENABLE([static-bin],
	[AC_HELP_STRING([--enable-static-bin],[Build statically linked binary @<:@default=no@:>@])],
	[static=$enableval]
	)
AM_CONDITIONAL([STATIC], [test "x$static" = "xyes"])

AC_CHECK_HEADERS(fcntl.h sys/time.h unistd.h sys/mman.h)
AC_CHECK_HEADERS(ctype.h errno.h sys/resource.h)
AC_CHECK_HEADERS(endian.h sys/endian.h arpa/inet.h)

AC_TYPE_OFF_T
AC_TYPE_SIZE_T
AC_C___ATTRIBUTE__
AC_CHECK_SIZEOF(int)
AC_CHECK_SIZEOF(long)
AC_CHECK_SIZEOF(short)

if test $ac_cv_sizeof_long != 4 -a "x$ASM" = "xyes" ; then
	AC_MSG_WARN([64bit arch detected, disabling ASM])
	ASM=no
fi


AC_CACHE_CHECK([for large file support],rzip_cv_HAVE_LARGE_FILES,[
AC_RUN_IFELSE([AC_LANG_SOURCE([[
#include <stdio.h>
#include <sys/types.h>
main() { return (sizeof(off_t) == 4); }]])],[rzip_cv_HAVE_LARGE_FILES=yes],[rzip_cv_HAVE_LARGE_FILES=no],[rzip_cv_HAVE_LARGE_FILES=cross])])
if test x"$rzip_cv_HAVE_LARGE_FILES" = x"yes"; then
    AC_DEFINE(HAVE_LARGE_FILES, 1, [ ])
fi

AC_C_INLINE

AC_CHECK_LIB(pthread, pthread_create, ,
        AC_MSG_ERROR([Could not find pthread library - please install libpthread]))
AC_CHECK_LIB(m, sqrt, ,
        AC_MSG_ERROR([Could not find math library - please install libm]))
AC_CHECK_LIB(z, compress2, ,
	AC_MSG_ERROR([Could not find zlib library - please install zlib-dev]))
AC_CHECK_LIB(bz2, BZ2_bzBuffToBuffCompress, ,
        AC_MSG_ERROR([Could not find bz2 library - please install libbz2-dev]))
AC_CHECK_LIB(lzo2, lzo1x_1_compress, ,
        AC_MSG_ERROR([Could not find lzo2 library - please install liblzo2-dev]))

AC_CHECK_FUNCS(mmap strerror)
AC_CHECK_FUNCS(getopt_long)

# final checks for x86 and/or assembler
if test x"$ASM" = x"no"; then
	ASM_OBJ=7zCrc.o
	ASM=no
else
	case $host in
		i?86-*)
			ASM_OBJ="7zCrcT8.o 7zCrcT8U.o"
			ASM_CMD="$ASM_PROG -f elf" ;;
# x86_64 code is broken still
#		x86_64-*)
#			ASM_OBJ="7zCrcT8.o 7zCrcT8U_64.o"
#			ASM_CMD="$ASM_PROG -f elf64" ;;
		*) ASM_OBJ=7zCrc.o ;;
	esac
fi
AM_CONDITIONAL([USE_ASM], [test "x$ASM" != "xyes" -a "x$ASM" != "xno"])
AC_SUBST([ASM_OBJ])
AC_SUBST([ASM_CMD])

AC_CONFIG_FILES([
Makefile
lzma/Makefile
lzma/C/Makefile
doc/Makefile
man/Makefile
])
AC_OUTPUT

echo
echo
echo
echo "------------------------------------------------------------------------"
echo "$PACKAGE $VERSION"
echo "------------------------------------------------------------------------"
echo
echo
echo "Configuration Options Summary:"
echo
echo "  ASM.(32 bit only)..: $ASM"
echo "  Static binary......: $static"
echo
echo "Compilation............: make (or gmake)"
echo "  CPPFLAGS.............: $CPPFLAGS"
echo "  CFLAGS...............: $CFLAGS"
echo "  CXXFLAGS.............: $CXXFLAGS"
echo "  LDFLAGS..............: $LDFLAGS"
echo
echo "Installation...........: make install (as root if needed, with 'su' or 'sudo')"
echo "  prefix...............: $prefix"
echo
