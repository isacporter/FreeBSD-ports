# Created by: Dmitry Marakasov <amdmi3@amdmi3.ru>
# $FreeBSD$

PORTNAME=	vor
PORTVERSION=	0.5.7
CATEGORIES=	games
MASTER_SITES=	https://jasonwoof.com/downloads/vor/ \
		http://mirror.amdmi3.ru/distfiles/

MAINTAINER=	amdmi3@FreeBSD.org
COMMENT=	Dodge the rocks until you die

LICENSE=	GPLv2+
LICENSE_FILE=	${WRKSRC}/COPYING

USES=		gmake pkgconfig tar:tgz
USE_SDL=	mixer image sdl
GNU_CONFIGURE=	yes

PLIST_FILES=	bin/${PORTNAME} \
		share/applications/${PORTNAME}.desktop
PORTDOCS=	*
PORTDATA=	*

OPTIONS_DEFINE=	DOCS

post-install:
	@${STRIP_CMD} ${STAGEDIR}${PREFIX}/bin/${PORTNAME}
	${INSTALL_DATA} ${WRKSRC}/${PORTNAME}.desktop ${STAGEDIR}${PREFIX}/share/applications

post-install-DOCS-on:
	@${MKDIR} ${STAGEDIR}${DOCSDIR}
	${INSTALL_DATA} ${WRKSRC}/README ${STAGEDIR}${DOCSDIR}

.include <bsd.port.mk>
