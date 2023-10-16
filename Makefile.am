AM_CPPFLAGS=-D_DEFAULT_SOURCE
AM_CFLAGS=-g -Wall -pedantic -fno-exceptions -D_BSD_SOURCE

bin_PROGRAMS = knock

if BUILD_KNOCKD
sbin_PROGRAMS = knockd
dist_sbin_SCRIPTS = src/knock_helper_ipt.sh
sysconf_DATA = knockd.conf
endif

dist_doc_DATA = README.md

knock_SOURCES = src/knock.c
knockd_SOURCES = src/knockd.c src/list.c src/list.h src/knock_helper_ipt.sh

%.1: %.1.in
	sed -e "s/#VERSION#/$(VERSION)/" $< > $@

EXTRA_DIST = knockd.conf
