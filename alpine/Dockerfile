# FROM quay.io/pires/docker-jre:8u92
# FROM alpine:edge
FROM alpine:3.4
MAINTAINER tangfx <tangfx128@gmail.com>

ENV NEXUS_DATA /nexus-data

ENV NEXUS_VERSION 3.0.1-01

ENV JAVA_HOME /opt/java
ENV JAVA_VERSION_MAJOR 8
ENV JAVA_VERSION_MINOR 91
ENV JAVA_VERSION_BUILD 14

ENV PATH=${PATH}:/opt/java/bin:/opt/sonatype/nexus/bin \
    GLIBC_VERSION=2.23-r3 \
    LANG=C.UTF-8

# https://github.com/anapsix/docker-alpine-java
RUN apk add --update bash ca-certificates curl sudo && \
    for pkg in glibc-${GLIBC_VERSION} glibc-bin-${GLIBC_VERSION} glibc-i18n-${GLIBC_VERSION}; do curl -sSL https://github.com/andyshinn/alpine-pkg-glibc/releases/download/${GLIBC_VERSION}/${pkg}.apk -o /tmp/${pkg}.apk; done && \
    apk add --allow-untrusted /tmp/*.apk && \
    rm -v /tmp/*.apk && \
    ( /usr/glibc-compat/bin/localedef --force --inputfile POSIX --charmap UTF-8 C.UTF-8 || true ) && \
    echo "export LANG=C.UTF-8" > /etc/profile.d/locale.sh && \
    /usr/glibc-compat/sbin/ldconfig /lib /usr/glibc-compat/lib

COPY *.tar.gz /tmp/
WORKDIR /tmp

# install Oracle JRE
RUN mkdir -p /opt && \
  if [[ -f jre-${JAVA_VERSION_MAJOR}u${JAVA_VERSION_MINOR}-linux-x64.tar.gz ]]; then \
    tar -C /opt -zxf jre-${JAVA_VERSION_MAJOR}u${JAVA_VERSION_MINOR}-linux-x64.tar.gz && \
    mv /opt/jre1.${JAVA_VERSION_MAJOR}.0_${JAVA_VERSION_MINOR} ${JAVA_HOME} && \
    rm jre-${JAVA_VERSION_MAJOR}u${JAVA_VERSION_MINOR}-linux-x64.tar.gz; \
  else \
    curl --fail --silent --location --retry 3 \
    --header "Cookie: oraclelicense=accept-securebackup-cookie; " \
    http://download.oracle.com/otn-pub/java/jdk/${JAVA_VERSION_MAJOR}u${JAVA_VERSION_MINOR}-b${JAVA_VERSION_BUILD}/server-jre-${JAVA_VERSION_MAJOR}u${JAVA_VERSION_MINOR}-linux-x64.tar.gz \
    | gunzip | tar -x -C /opt \
    && ln -s /opt/jdk1.${JAVA_VERSION_MAJOR}.0_${JAVA_VERSION_MINOR} ${JAVA_HOME}; \
  fi

# install nexus
RUN mkdir -p /opt/sonatype && \
  if [[ -f nexus-${NEXUS_VERSION}-unix.tar.gz ]]; then \
    tar -C /opt/sonatype -zxf nexus-${NEXUS_VERSION}-unix.tar.gz && \
    rm nexus-${NEXUS_VERSION}-unix.tar.gz; \
  else \
    curl --fail --silent --location --retry 3 \
    https://download.sonatype.com/nexus/3/nexus-${NEXUS_VERSION}-unix.tar.gz \
    | gunzip | tar -x -C /opt/sonatype; \
  fi && \
  mv /opt/sonatype/nexus-${NEXUS_VERSION} /opt/sonatype/nexus && \
  chown -R root:root /opt/sonatype/nexus

# Patch nexus
# https://support.sonatype.com/hc/en-us/articles/218729178-Nexus-Repository-Manager-3-0-0-03-Docker-Rollup-Patch
# RUN curl --fail --silent --retry 3 \
#    https://support.sonatype.com/hc/en-us/article_attachments/208186848/nexus-repository-docker-3.0.0-03-patch2.jar \
#    -o /opt/sonatype/nexus/system/com/sonatype/nexus/plugins/nexus-repository-docker/3.0.0-03/nexus-repository-docker-3.0.0-03.jar

## configure nexus runtime env
RUN sed \
    -e "s|karaf.home=.|karaf.home=/opt/sonatype/nexus|g" \
    -e "s|karaf.base=.|karaf.base=/opt/sonatype/nexus|g" \
    -e "s|karaf.etc=etc|karaf.etc=/opt/sonatype/nexus/etc|g" \
    -e "s|java.util.logging.config.file=etc|java.util.logging.config.file=/opt/sonatype/nexus/etc|g" \
    -e "s|karaf.data=data|karaf.data=${NEXUS_DATA}|g" \
    -e "s|java.io.tmpdir=data/tmp|java.io.tmpdir=${NEXUS_DATA}/tmp|g" \
    -i /opt/sonatype/nexus/bin/nexus.vmoptions

RUN adduser -u 200 -G users -g "nexus role account" -D -h ${NEXUS_DATA} -s /bin/false nexus \
  && echo "nexus ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers \
  && chmod -R ugo+rwx /opt/sonatype/nexus/etc \
  && apk del curl glibc-i18n && \
    find /opt/jdk/ -maxdepth 1 -mindepth 1 | grep -v jre | xargs rm -rf && \
    rm -rf /opt/jdk/jre/plugin \
           /opt/jdk/jre/bin/javaws \
           /opt/jdk/jre/bin/jjs \
           /opt/jdk/jre/bin/keytool \
           /opt/jdk/jre/bin/orbd \
           /opt/jdk/jre/bin/pack200 \
           /opt/jdk/jre/bin/policytool \
           /opt/jdk/jre/bin/rmid \
           /opt/jdk/jre/bin/rmiregistry \
           /opt/jdk/jre/bin/servertool \
           /opt/jdk/jre/bin/tnameserv \
           /opt/jdk/jre/bin/unpack200 \
           /opt/jdk/jre/lib/javaws.jar \
           /opt/jdk/jre/lib/deploy* \
           /opt/jdk/jre/lib/desktop \
           /opt/jdk/jre/lib/*javafx* \
           /opt/jdk/jre/lib/*jfx* \
           /opt/jdk/jre/lib/amd64/libdecora_sse.so \
           /opt/jdk/jre/lib/amd64/libprism_*.so \
           /opt/jdk/jre/lib/amd64/libfxplugins.so \
           /opt/jdk/jre/lib/amd64/libglass.so \
           /opt/jdk/jre/lib/amd64/libgstreamer-lite.so \
           /opt/jdk/jre/lib/amd64/libjavafx*.so \
           /opt/jdk/jre/lib/amd64/libjfx*.so \
           /opt/jdk/jre/lib/ext/jfxrt.jar \
           /opt/jdk/jre/lib/ext/nashorn.jar \
           /opt/jdk/jre/lib/oblique-fonts \
           /opt/jdk/jre/lib/plugin.jar \
           /tmp/* /var/cache/apk/* && \
    echo 'hosts: files mdns4_minimal [NOTFOUND=return] dns mdns4' >> /etc/nsswitch.conf

VOLUME ${NEXUS_DATA}

EXPOSE 8081
USER nexus
WORKDIR /opt/sonatype/nexus

ENV JAVA_MAX_MEM 1200m
ENV JAVA_MIN_MEM 1200m
ENV EXTRA_JAVA_OPTS ""

CMD bin/nexus run
