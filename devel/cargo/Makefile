# Created by: Jean-Sébastien Pédron <dumbbell@FreeBSD.org>
# $FreeBSD$

PORTNAME=	cargo
PORTVERSION=	0.19.0
CATEGORIES=	devel

MAINTAINER=	rust@FreeBSD.org
COMMENT=	Rust's Package Manager
# ' <-- Help Vim syntax hilighting...

LICENSE=	APACHE20 \
		MIT
LICENSE_COMB=	dual
# APACHE20 license is standard, see Templates/Licenses/APACHE20
LICENSE_FILE_MIT=	${WRKSRC}/LICENSE-MIT

# The following files are required:
#
# Main source archive from GitHub
#   This is the source of Cargo, taken from GitHub.
#
# :cargo_bootstrap
#   Like Rust, this is a prebuilt "cargo" used to bootstrap it, because
#   Cargo is a standard Rust package itself.
#
# :registry
#   Cargo clones several Git repositories for its dependencies and
#   caches them in its "registry". This is an archive of this registry
#   so nothing is downloaded during the build.
#
# :rust_installer
#   This is a collection of mainly shell scripts which are used to
#   install Cargo.

MASTER_SITES=	https://static.rust-lang.org/dist/:cargo_bootstrap \
		https://s3.amazonaws.com/rust-lang-ci/cargo-builds/:cargo_bootstrap \
		LOCAL/dumbbell/rust:cargo_bootstrap
.if !defined(SKIP_CARGO_REGISTRY)
MASTER_SITES+=	LOCAL/dumbbell/rust:registry
DISTFILES+=	${CARGO_REGISTRY}:registry
.endif
DIST_SUBDIR?=	rust

USE_GITHUB=	yes
GH_ACCOUNT=	rust-lang
GH_PROJECT=	rust-installer:rust_installer
GH_TAGNAME=	4f99485:rust_installer
GH_SUBDIR=	src/rust-installer:rust_installer

ONLY_FOR_ARCHS=		aarch64 amd64 i386
ONLY_FOR_ARCHS_REASON=	requires prebuilt bootstrap cargo

# Rust's target arch string is different from *BSD arch strings
RUST_ARCH_aarch64=	aarch64
RUST_ARCH_amd64=	x86_64
RUST_ARCH_i386=		i686
RUST_ARCH_x86_64=	x86_64 # dragonfly
RUST_TARGET=		${RUST_ARCH_${ARCH}}-unknown-${OPSYS:tl}

CARGO_BOOTSTRAP_DIR?=		2017-04-27
CARGO_BOOTSTRAP_DIR_aarch64?=	2017-04-24
CARGO_BOOTSTRAP_VERSION?=	0.18.0
CARGO_BOOTSTRAP_VERSION_aarch64?=0.18.0
CARGO_BOOTSTRAP=		${CARGO_BOOTSTRAP_DIR_${ARCH}:U${CARGO_BOOTSTRAP_DIR}}/cargo-${CARGO_BOOTSTRAP_VERSION_${ARCH}:U${CARGO_BOOTSTRAP_VERSION}}-${RUST_TARGET}${EXTRACT_SUFX}
CARGO_REGISTRY=			${PORTNAME}-registry-${DISTVERSIONFULL}.tar.xz

# We don't USES=cmake here, because cmake is not Cargo's build system.
# It's used by a bundled dependency (libgit2).
BUILD_DEPENDS=	cmake:devel/cmake \
		${RUST_PORT:T}>=1.18.0:${RUST_PORT}
LIB_DEPENDS=	libssh2.so:security/libssh2 \
		libcurl.so:ftp/curl
RUN_DEPENDS=	rustc:${RUST_PORT}
RUST_PORT?=	lang/rust

USES=		gmake python:-2.7,build pkgconfig ssl
HAS_CONFIGURE=	yes
CONFIGURE_ARGS=	--prefix="${PREFIX}" --mandir="${MANPREFIX}/man"
MAKE_ENV=	ARGS="${CARGO_ARGS}" \
		OPENSSL_DIR="${OPENSSLBASE}"
CARGO_ARGS=	--jobs ${MAKE_JOBS_NUMBER}

MAKE_ARGS+=	VERBOSE=1

OPTIONS_DEFINE=	BOOTSTRAP
OPTIONS_DEFAULT=BOOTSTRAP

BOOTSTRAP_DESC=		Bootstrap using pre-built vendor snapshot
BOOTSTRAP_CONFIGURE_ON=	--cargo="${WRKDIR}/${PORTNAME}-${CARGO_BOOTSTRAP_VERSION_${ARCH}:U${CARGO_BOOTSTRAP_VERSION}}-${RUST_TARGET}/cargo/bin/cargo"
BOOTSTRAP_CONFIGURE_OFF=--cargo="$$(command -v cargo)" # respect PATH
BOOTSTRAP_DISTFILES=	${CARGO_BOOTSTRAP}:cargo_bootstrap
BOOTSTRAP_MAKE_ENV=	LD_LIBRARY_PATH="${WRKSRC}/target/snapshot/cargo/lib"

post-patch:
# Fix mandir and don't install licenses outside of ${_LICENSE_DIR}
	@${REINPLACE_CMD} -e 's,share/man,man,' \
		-e 's,$$(S)LICENSE-[^[:space:]]* ,,g' \
		${WRKSRC}/Makefile.in

# In case the previous "make stage" failed, this ensures rust's
# install.sh won't backup previously staged files before reinstalling
# new ones. Otheriwe, the staging directory is polluted with unneeded
# files.
pre-install:
	@if test -f ${STAGEDIR}${PREFIX}/lib/rustlib/manifest-cargo; then \
		${SED} -E -e 's,^(dir|file:),${STAGEDIR},' \
		< ${STAGEDIR}${PREFIX}/lib/rustlib/manifest-cargo \
		| ${XARGS} ${RM}; \
	fi
	@${RM} \
		${STAGEDIR}${PREFIX}/lib/rustlib/components \
		${STAGEDIR}${PREFIX}/lib/rustlib/manifest-cargo \
		${STAGEDIR}${PREFIX}/lib/rustlib/rust-installer-version \
		${STAGEDIR}${PREFIX}/lib/rustlib/uninstall.sh

post-install:
	@${RM} ${STAGEDIR}${PREFIX}/lib/rustlib/install.log
	@${REINPLACE_CMD} -i '' -e 's|${STAGEDIR}||' \
		${STAGEDIR}${PREFIX}/lib/rustlib/manifest-cargo
	@${STRIP_CMD} ${STAGEDIR}${PREFIX}/bin/cargo
	@${RM} \
		${STAGEDIR}${PREFIX}/lib/rustlib/components \
		${STAGEDIR}${PREFIX}/lib/rustlib/rust-installer-version \
		${STAGEDIR}${PREFIX}/lib/rustlib/uninstall.sh

# "make gen-registry" is a special target to ease this port update.
#
# After changing the version number and the Git revision, you can run
# "make gen-registry" to update the distinfo and create a new snapshot of
# the registry. The new registry is written to ${DISTDIR} and can be
# uploaded.

gen-registry:
	${MAKE} -C${.CURDIR} -DSKIP_CARGO_REGISTRY makesum
	${MAKE} -C${.CURDIR} -DSKIP_CARGO_REGISTRY
	${TAR} cJvf ${_DISTDIR}/${CARGO_REGISTRY} -C${WRKDIR} \
		--uid 0 --gid 0 \
		--exclude libgit2/tests/ \
		--exclude curl/tests/ \
		--exclude 'index/github.com-*/.git/' \
		.cargo
	${MAKE} -C${.CURDIR} makesum

.include <bsd.port.mk>
