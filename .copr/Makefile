# Copr "make_srpm" SCM source method entry point. Invoked by Copr's
# builder as `make -f .copr/Makefile srpm outdir=<dir> spec=<path>`,
# already inside a clone of this repo at the configured commit/branch.
# Produces an SRPM for `outdir` from HEAD, using the spec committed
# alongside this Makefile under .copr/. Mirrors what
# make-source-tarball.sh + build-all.sh's SRPM step do in the local/CI
# pipeline (see github.com/ykne/GNOME-X11-specs), scoped to this package.
#
# subprojects/gvc is a real git submodule (.gitmodules), not a meson wrap -
# `meson subprojects download` silently does nothing for it, so it's
# fetched via `git submodule update --init` instead. libgxdp *is* a wrap,
# fetched the normal way. Both are gitignored on disk, so `git archive`
# alone would drop them - vendored into the source tarball explicitly.
# make_srpm runs as root with network access, so meson is installed here
# rather than relying on any project-level buildroot package list.

PKG := gnome-control-center
VERSION := 50.4

.PHONY: srpm
srpm:
	set -e; \
	dnf install -y meson; \
	git submodule update --init -- subprojects/gvc; \
	meson subprojects download libgxdp; \
	hash=$$(git rev-parse --short=8 HEAD); \
	workdir=$$(mktemp -d); \
	mkdir -p "$$workdir/$(PKG)-$(VERSION)/subprojects" "$$workdir/SOURCES"; \
	git archive HEAD | tar -x -C "$$workdir/$(PKG)-$(VERSION)"; \
	cp -r subprojects/gvc subprojects/libgxdp "$$workdir/$(PKG)-$(VERSION)/subprojects/"; \
	tar -cJf "$$workdir/SOURCES/$(PKG)-$(VERSION).tar.xz" -C "$$workdir" "$(PKG)-$(VERSION)"; \
	cp .copr/SOURCES/* "$$workdir/SOURCES/" 2>/dev/null || true; \
	sed "s/^Release:.*\$$/Release:        1.g$${hash}%{?dist}/" "$(spec)" > "$$workdir/$(PKG).spec"; \
	rpmbuild -bs \
		--define "_topdir $$workdir" \
		--define "_sourcedir $$workdir/SOURCES" \
		--define "_specdir $$workdir" \
		--define "_builddir $$workdir/BUILD" \
		--define "_srcrpmdir $(outdir)" \
		"$$workdir/$(PKG).spec"; \
	rm -rf "$$workdir"
