AM_CFLAGS = $(WERROR_CFLAGS) $(WARN_CFLAGS) -Wno-unused-parameter -Wno-pedantic
AM_CPPFLAGS = -I$(top_srcdir)/src -I$(srcdir) -I$(top_builddir)/lib -I$(top_srcdir)/lib \
 -DSRCDIR=\"$(abs_srcdir)\"
LDADD = ../lib/libgnu.a \
 $(GETADDRINFO_LIB) $(HOSTENT_LIB) $(INET_NTOP_LIB) $(INET_PTON_LIB) \
 $(LIBSOCKET) $(LIB_CLOCK_GETTIME) $(LIB_CRYPTO) $(LIB_GETLOGIN) $(LIB_NANOSLEEP) $(LIB_POLL) \
 $(LIB_POSIX_SPAWN) $(LIB_PTHREAD_SIGMASK) $(LIB_SELECT) $(LTLIBICONV) $(LTLIBINTL) \
 $(LTLIBMULTITHREAD) $(LTLIBTHREAD) $(SERVENT_LIB) @INTL_MACOSX_LIBS@ \
 -ldl

WGET_TESTS = \
 wget_options_fuzzer$(EXEEXT)

if FUZZING
  bin_PROGRAMS = $(WGET_TESTS)
  LDADD += $(LIB_FUZZING_ENGINE)
  MAIN = fuzzer.h
#  AM_LDFLAGS = -no-install -all-static
else
  AM_CPPFLAGS += -DTEST_RUN
  AM_TESTS_ENVIRONMENT = export VALGRIND_TESTS"=@VALGRIND_TESTS@";
  TESTS = $(WGET_TESTS)
  check_PROGRAMS = $(WGET_TESTS)
  MAIN = main.c fuzzer.h
endif

# Make libunittest "PHONY" so we're always sure we're up-to-date.
.PHONY: ../src/libunittest.a
../src/libunittest.a:
	$(MAKE) $(AM_MAKEFLAGS) -C ../src libunittest.a

wget_options_fuzzer_SOURCES = wget_options_fuzzer.c $(MAIN)
wget_options_fuzzer_LDADD = ../src/libunittest.a $(LDADD)


#EXTRA_DIST = $(wildcard *.options) $(wildcard *.dict) \
# $(wildcard *.in) $(wildcard *.repro)

dist-hook:
	find $(srcdir) -name '*.options' -exec cp -v '{}' $(distdir) ';'
	find $(srcdir) -name '*.dict' -exec cp -v '{}' $(distdir) ';'
	find $(srcdir) -name '*.in' -exec cp -vr '{}' $(distdir) ';'
	find $(srcdir) -name '*.repro' -exec cp -vr '{}' $(distdir) ';'

clean-local:
	rm -rf *.gc?? *.log lcov

oss-fuzz:
	if test "$$OUT" != ""; then \
	XLIBS="-lpsl -lgnutls -lnettle -lhogweed -lidn2 -lunistring"; \
	for ccfile in wget*_fuzzer.c; do \
	fuzzer=$$(basename $$ccfile .c); \
	$$CXX $$CXXFLAGS -I$(top_srcdir)/src -I$(top_srcdir) \
	"$${fuzzer}.c" -o "$${fuzzer}" \
	../src/libunittest.a ../lib/libgnu.a $${LIB_FUZZING_ENGINE} \
	-Wl,-Bstatic $${XLIBS} -Wl,-Bdynamic; \
	done; \
	fi

.PHONY: oss-fuzz