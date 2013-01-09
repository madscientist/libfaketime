#
# Notes:
#
#   * Variables controlling behavior.  You can set these to empty on the
#     make command line to disable them (e.g., "make USE_CACHING="):
#
#     USE_FAKE_STAT
#         - Enables time faking also for files' timestamps.
#
#     USE_FAKE_ATFILE
#         - Enables time faking for the fstatat() group of functions
#
#     USE_FAKE_INTERNAL_CALLS
#         - Also intercept libc internal __functions, e.g. not just time(),
#           but also __time(). Enhances compatibility with applications
#           that make use of low-level system calls, such as Java Virtual
#           Machines.
#
#     USE_CACHING
#         - Disables the caching of the fake time offset. Only disable caching
#           if you change the fake time offset during program runtime very
#           frequently. Disabling the cache may negatively influence the
#           performance.
#
#     USE_LIMITEDFAKING
#         - Support environment variables that limit time faking to certain
#           time intervals or number of function calls.
#
#     USE_SPAWNSUPPORT
#         - Enable support for spawning an external process at a given
#           timestamp.
#
#     USE_PTHREAD_SINGLETHREADED_TIME
#         - Define this if you want to single-thread time() ... there ARE
#           possibile caching side-effects in a multithreaded environment
#           without this, but the performance impact may require you to
#           try it unsynchronized.
#
#     USE_FAKE_PTHREAD_COND_TIMEDWAIT
#         - Intercept pthread_cond_timedwait().  Some implementations of glibc
#           call the clock_gettime() system call directly, rather than using
#           gettimeofday() or similar, when computing the timed wait.  If your
#           implementation does this set this value to reset the timeout
#           appropriately based on the fake time.
#
#   * Compilation addition: second libMT target added for building the
#     pthread-enabled library as a separate library
#
#   * Compilation switch change: previous versions compiled using
#     '-nostartfiles' This is no longer the case since there is a 'startup'
#     constructor for the library which is used to activate the start-at times
#     when specified. This also initializes the dynamic disabling of the
#     FAKE_STAT calls.
#

USE_CACHING= -DCACHING
USE_FAKE_STAT = -DFAKE_STAT
USE_FAKE_ATFILE = -DFAKE_ATFILE
USE_FAKE_INTERNAL_CALLS = -DFAKE_INTERNAL_CALLS
USE_POSIX_REALTIME = -DPOSIX_REALTIME
USE_LIMITEDFAKING = -DLIMITEDFAKING
USE_SPAWNSUPPORT = -DSPAWNSUPPORT
USE_PTHREAD_SINGLETHREADED_TIME = -DPTHREAD_SINGLETHREADED_TIME
USE_FAKE_PTHREAD_COND_TIMEDWAIT = -DFAKE_PTHREAD_COND_TIMEDWAIT

# Where to install
PREFIX = /usr/local

# Shared library version
SONAME = 1

# Allow the user to override the default compilation flags
CFLAGS = -O2 -g

# Utilities
CC = gcc
INSTALL = install

# --------------------------------------------------
# The rest of this should not be changed

_CFLAGS += -std=gnu99 -Wall -fPIC
_CPPFLAGS = $(USE_CACHING) $(USE_FAKE_ATFILE) $(USE_FAKE_STAT) $(USE_FAKE_INTERNAL_CALLS) $(USE_POSIX_REALTIME) $(USE_LIMITEDFAKING) $(USE_SPAWNSUPPORT)
_MTFLAGS = $(USE_PTHREAD_SINGLETHREADED_TIME) $(USE_FAKE_PTHREAD_COND_TIMEDWAIT)
LDFLAGS += -shared -ldl -lm -lpthread

SRC = faketime.c
OBJ = faketime.o faketimeMT.o

LIBS = libfaketime.so.${SONAME} libfaketimeMT.so.${SONAME}

COMPILE.o = ${CC} ${CPPFLAGS} ${_CPPFLAGS} ${CFLAGS} ${_CFLAGS}

all: ${LIBS}

lib%.so.${SONAME}: %.o
	${CC} -o $@ -Wl,-soname,$@ $^ ${LDFLAGS}

%MT.o : %.c
	${COMPILE.o} -DPTHREAD ${_MTFLAGS} -o $@ -c $<

%.o : %.c
	${COMPILE.o} -o $@ -c $<

clean:
	@rm -f ${OBJ} ${LIBS}

distclean: clean
	@echo

_INST = ${DESTDIR}${PREFIX}

install: ${LIBS}
	@echo
	@echo "Copying the faketime libraries to ${_INST}/lib/faketime and the faketime wrapper script to ${_INST}/bin ..."
	$(INSTALL) -dm0755 "${_INST}/lib/faketime/"
	$(INSTALL) -m0644 ${LIBS} "${_INST}/lib/faketime/"
	$(INSTALL) -Dm0755 faketime "${_INST}/bin/faketime"

uninstall:
	for f in ${LIBS}; do rm -f "${_INST}/lib/faketime/$$f"; done
	rmdir "${_INST}/lib/faketime"
	rm -f "${_INST}/bin/faketime"

.PHONY: all clean distclean install uninstall
.PRECIOUS: ${OBJ} ${LIBS}
