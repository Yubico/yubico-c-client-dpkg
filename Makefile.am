# Written by Simon Josefsson <simon@josefsson.org>.
# Copyright (c) 2008-2013 Yubico AB
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are
# met:
#
#   * Redistributions of source code must retain the above copyright
#      notice, this list of conditions and the following disclaimer.
#
#   * Redistributions in binary form must reproduce the above
#      copyright notice, this list of conditions and the following
#      disclaimer in the documentation and/or other materials provided
#      with the distribution.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
# LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
# A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
# OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
# SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
# LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
# DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
# THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

SUBDIRS = . tests

AM_CPPFLAGS = @LIBCURL_CPPFLAGS@
ACLOCAL_AMFLAGS = -I m4
AM_CFLAGS = $(WARN_CFLAGS)

# The library.

lib_LTLIBRARIES = libykclient.la
include_HEADERS = ykclient_server_response.h ykclient.h \
	ykclient_errors.h ykclient_version.h

libykclient_la_SOURCES = ykclient_server_response.h \
			 ykclient_server_response.c
libykclient_la_SOURCES += ykclient.h ykclient_errors.h	\
			  ykclient_version.h
libykclient_la_SOURCES += ykclient.c ykclient_version.c
libykclient_la_SOURCES += rfc4634/sha.h rfc4634/sha-private.h		\
			rfc4634/hmac.c rfc4634/sha1.c rfc4634/usha.c	\
			rfc4634/sha224-256.c rfc4634/sha384-512.c
libykclient_la_SOURCES += b64/cencode.h b64/cencode.c b64/cdecode.h	\
			b64/cdecode.c
libykclient_la_LIBADD = @LIBCURL@
libykclient_la_LDFLAGS = -no-undefined \
	-export-symbols-regex '^ykclient_.*' \
	-version-info $(LT_CURRENT):$(LT_REVISION):$(LT_AGE)

# The command line tools.

bin_PROGRAMS = ykclient

ykclient_SOURCES = tool.c
ykclient_LDADD = ./libykclient.la

# Man page

dist_man_MANS = ykclient.1
DISTCLEANFILES = $(dist_man_MANS)

ykclient.1: $(builddir)/ykclient $(srcdir)/tool.c $(srcdir)/configure.ac
	$(HELP2MAN) \
		--output=$@ $(builddir)/ykclient$(EXEEXT) \
		--name="YubiCloud One-Time-Password Validation Client" \
		--no-info

# Release

indent:
	indent *.c *.h *.h.in
	indent *.c *.h *.h.in

ChangeLog:
	cd $(srcdir) && git2cl > ChangeLog

PROJECT = yubico-c-client

release:
	@if test -z "$(KEYID)"; then \
		echo "Try this instead:"; \
		echo "  make release KEYID=[PGPKEYID]"; \
		echo "For example:"; \
		echo "  make release KEYID=2117364A"; \
		exit 1; \
	fi
	@if test ! -d "$(YUBICO_GITHUB_REPO)"; then \
		echo "yubico.github.com repo not found!"; \
		echo "Make sure that YUBICO_GITHUB_REPO is set"; \
		exit 1; \
	fi
	@head -3 $(srcdir)/NEWS | grep -q "Version $(VERSION) .released `date -I`" || \
		(echo 'error: You need to update date/version in $(srcdir)/NEWS'; exit 1)
	rm -f $(srcdir)/ChangeLog
	make ChangeLog distcheck
	gpg --detach-sign --default-key $(KEYID) $(PACKAGE)-$(VERSION).tar.gz
	gpg --verify $(PACKAGE)-$(VERSION).tar.gz.sig
	cd $(srcdir) && git push
	cd $(srcdir) && git tag -u $(KEYID)! -m $(VERSION) $(PACKAGE)-$(VERSION)
	cd $(srcdir) && git push --tags
	$(YUBICO_GITHUB_REPO)/publish $(PROJECT) $(VERSION) $(PACKAGE)-$(VERSION).tar.gz*
	@if test -d $(srcdir)/../wiki-$(PROJECT); then \
		cd $(srcdir) && \
		cp README ../wiki-$(PROJECT)/ReadMe.wiki && \
		cd ../wiki-$(PROJECT) && \
		git commit -m Sync. ReadMe.wiki; \
	fi