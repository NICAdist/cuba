# Process this file with autoconf to produce a configure script.

AC_PREREQ(2.59)
AC_INIT([Cuba], [4.2.2], [hahn@feynarts.de])
AC_CONFIG_SRCDIR([cuba.h])

LIBS=-lm
USER_CFLAGS="$CFLAGS"

AC_ARG_VAR(MCFLAGS, [MathLink C compiler flags])

AC_PROG_CC
AC_PROG_FC
AC_PROG_RANLIB
AC_PROG_INSTALL

AS_IF([test "$GCC" = yes],
  [AS_CASE([`$CC --version 2>&1 < /dev/null`],
    [*LLVM*], [opt=-O3],
    [*gcc*4.2* | *gcc*4.4.3*], [opt=-O0],
    [opt=-O3])
   CFLAGS=${USER_CFLAGS:-$opt -fomit-frame-pointer -ffast-math -Wall}],
  [CFLAGS=${USER_CFLAGS:--O}])

AC_C_CONST
AC_C_INLINE
AC_C_LONG_DOUBLE

AC_TYPE_SIZE_T
AC_TYPE_SSIZE_T
AC_TYPE_PID_T

AC_CHECK_FUNCS([powl])
AC_CHECK_FUNCS([erf])
AC_FUNC_FORK
AC_FUNC_ALLOCA

AC_DEFUN([chk_shmget], [dnl
AC_REQUIRE([AC_CANONICAL_HOST])
AS_CASE([$host_os],
  [*cygwin*], [],
  [AC_CHECK_FUNCS([shmget])])
])
chk_shmget
AC_CHECK_FUNCS([getloadavg])

MAXDIM=${MAXDIM:-16}
AC_ARG_WITH(maxdim,
  [AS_HELP_STRING([--with-maxdim=N],
    [the maximum dimension for integration,
     if variable-size array are not supported])],
  [MAXDIM=$withval])

MAXCOMP=${MAXCOMP:-4}
AC_ARG_WITH(maxcomp,
  [AS_HELP_STRING([--with-maxcomp=N],
    [the maximum number of components of the integrand,
     if variable-size array are not supported])],
  [MAXCOMP=$withval])

REALSIZE=${REALSIZE:-8}
AC_ARG_WITH(real,
  [AS_HELP_STRING([--with-real=N],
    [use N-byte floating-point numbers])],
  [REALSIZE=$withval])
AS_CASE([$REALSIZE],
  [8],  [],
  [10], [SUFFIX=${SUFFIX:-l}],
  [16], [SUFFIX=${SUFFIX:-q}
         LIBS="-lquadmath $LIBS"],
  [REALSIZE=8
   AC_MSG_WARN([Unavailable real size])])
AC_SUBST([REALSIZE])
AC_SUBST([SUFFIX])

AC_MSG_CHECKING([for variable-size arrays])
AC_COMPILE_IFELSE([AC_LANG_SOURCE(,[[
	void test(int n) {
	  char s[n];
	}
  ]])],
  [AC_MSG_RESULT([yes])],
  [AC_MSG_RESULT([no, using MAXDIM=$MAXDIM and MAXCOMP=$MAXCOMP])
   AC_DEFINE_UNQUOTED([NDIM], [$MAXDIM], [Maximum number of components])
   AC_DEFINE_UNQUOTED([NCOMP], [$MAXCOMP], [Maximum number of dimensions])]
)

AC_MSG_CHECKING([for MathLink])
cat > conftest.tm << _EOF_
:Begin:
:Function: inc
:Pattern: Inc[i_Integer]
:Arguments: {i}
:ArgumentTypes: {Integer}
:ReturnType: Integer
:End:

#include "mathlink.h"

static int inc(const int i) { return i + 1; }

int main(int argc, char **argv) {
  return MLMain(argc, argv);
}
_EOF_
FCC="$srcdir/tools/fcc"
rm -f conftest$EXEEXT
MCSTDCPP=
for MCSTDCPP in "" -stdlib=libstdc++ ; do
  REALCC="$CC" CC="$FCC" CXX="$FCC $MCSTDCPP" PATH="$PATH:$srcdir/tools" \
    mcc $MCFLAGS $CFLAGS -o conftest$EXEEXT conftest.tm > /dev/null 2>&1
  AS_IF([test -x conftest$EXEEXT], [break])
done
AS_IF([test -x conftest$EXEEXT],
  [AC_MSG_RESULT([yes])
   MATH_DEFAULT=math],
  [AC_MSG_RESULT([no])])
AC_SUBST([MCSTDCPP])
AC_SUBST([MATH_DEFAULT])

AC_CHECK_PROGS([HAVE_QMAKE], [qmake])
AS_IF([test -n "$HAVE_QMAKE"],
  [AS_CASE([`qmake -v`],
    [*Qt?version?4*], [TOOLS_DEFAULT=tools])])
AC_SUBST([TOOLS_DEFAULT])

test -n "$FC" && DEMO_FORTRAN_DEFAULT=demo-fortran
AC_SUBST([DEMO_FORTRAN_DEFAULT])

AC_CONFIG_HEADERS([config.h])
AC_CONFIG_FILES([makefile])

AC_OUTPUT

