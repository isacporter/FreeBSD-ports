# $FreeBSD$

PORTNAME=	babeltrace
PORTVERSION=	1.5.3
DISTVERSIONPREFIX=	v
CATEGORIES=	devel

MAINTAINER=	luca.pizzamiglio@gmail.com
COMMENT=	Tool and library to read, write and convert execution traces

# license is complicated
LICENSE=	MIT GPLv2 LGPL21
LICENSE_COMB=	multi
LICENSE_FILE_MIT=	${WRKSRC}/mit-license.txt
LICENSE_FILE_GPLv2=	${WRKSRC}/gpl-2.0.txt
LICENSE_FILE_LGPL21=	${WRKSRC}/lgpl-2.1.txt
LICENSE_FILE=	${WRKSRC}/LICENSE

LIB_DEPENDS=	libpopt.so:devel/popt \
		libuuid.so:misc/e2fsprogs-libuuid

USE_GITHUB=	yes
GH_ACCOUNT=	efficios

USES=		autoreconf:build bison gmake libtool pathfix pkgconfig
GNU_CONFIGURE=	yes
CONFIGURE_ARGS=	--disable-debug-info
USE_GNOME=	glib20
LDFLAGS+=	-L${LOCALBASE}/lib
USE_LDCONFIG=	yes

PORTDOCS=	ChangeLog std-ext-lib.txt API.txt lttng-live.txt

post-patch:
	@${REINPLACE_CMD} -e 's|LICENSE mit-license.txt gpl-2.0.txt||' \
		${WRKSRC}/Makefile.am
	@${REINPLACE_CMD} -e 's|machine\/endian|sys\/endian|' \
		${WRKSRC}/include/babeltrace/endian.h

pre-configure:
	(cd ${WRKSRC}; ./bootstrap )

post-install:
	${STRIP_CMD} ${STAGEDIR}${PREFIX}/lib/libbabeltrace*so.1.0.0

.include <bsd.port.mk>
