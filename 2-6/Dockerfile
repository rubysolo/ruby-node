FROM ruby:2-stretch

ENV LANG C.UTF-8

RUN gem install bundler -v 1.17.3

RUN groupadd --gid 1000 node \
  && useradd --uid 1000 --gid node --shell /bin/bash --create-home node

# disable ipv6
RUN mkdir ~/.gnupg && chmod 0700 ~/.gnupg
RUN echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf

# gpg keys listed at https://github.com/nodejs/node#release-team
# 6A010...97F5 key is for yarn, others are for node
RUN set -ex && \
    for key in \
      108F52B48DB57BB0CC439B2997B01419BD92F80A \
      114F43EE0176B71C7BC219DD50A3051F888C628D \
      4ED778F539E3634C779C87C6D7062848A1AB005C \
      56730D5401028683275BD23C23EFEFE93C4CFFFE \
      71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
      77984A986EBC2AA786BC0F66B01FBB92821C587A \
      7937DFD2AB06298B2293C3187D33FF9D0246406D \
      8FCCA13FEF1D0C2E91008E09770F7A9A5AE15600 \
      93C7E9E91B49E432C2F75674B0A78B0A6C481CF6 \
      94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
      9554F04D7259F04124DE6B476D5A82AC7E37093B \
      A48C2BEE680E841632CD4E44F07496B3EB3C1762 \
      B9AE9905FFD7803F25714661B63B535A4C206CA9 \
      B9E2F5981AA6E0CD28160D9FF13993A75599653C \
      C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
      C82FA3AE1CBEDC6BE46B9360C43CEC45C17AB93C \
      DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
      FD3A5288F042B6850C66B31F09FE44734EB7990E \
      6A010C5166006599AA17F08146C2130DFD2497F5 ; \
    do
      gpg --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys "$key" || \
      gpg --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" || \
      gpg --keyserver hkp://pgp.mit.edu:80 --recv-keys "$key"; \
    done

ENV NODE_MAJOR 6

RUN NODE_VERSION=$(curl -SL "https://nodejs.org/dist/index.tab" \
    | grep "^v$NODE_MAJOR" \
    | head -n 1 | awk '{ print substr($1,2) }') \
  ARCH= && dpkgArch="$(dpkg --print-architecture)" \
  && case "${dpkgArch##*-}" in \
    amd64) ARCH='x64';; \
    ppc64el) ARCH='ppc64le';; \
    s390x) ARCH='s390x';; \
    arm64) ARCH='arm64';; \
    armhf) ARCH='armv7l';; \
    i386) ARCH='x86';; \
    *) echo "unsupported architecture"; exit 1 ;; \
  esac \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-$ARCH.tar.xz" \
  && curl -SLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
  && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
  && grep " node-v$NODE_VERSION-linux-$ARCH.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
  && tar -xJf "node-v$NODE_VERSION-linux-$ARCH.tar.xz" -C /usr/local --strip-components=1 --no-same-owner \
  && rm "node-v$NODE_VERSION-linux-$ARCH.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
  && ln -s /usr/local/bin/node /usr/local/bin/nodejs

RUN YARN_VERSION=$(curl -sSL --compressed https://yarnpkg.com/latest-version) \
  && curl -fSLO --compressed "https://yarnpkg.com/downloads/$YARN_VERSION/yarn-v$YARN_VERSION.tar.gz" \
  && curl -fSLO --compressed "https://yarnpkg.com/downloads/$YARN_VERSION/yarn-v$YARN_VERSION.tar.gz.asc" \
  && gpg --batch --verify yarn-v$YARN_VERSION.tar.gz.asc yarn-v$YARN_VERSION.tar.gz \
  && mkdir -p /opt/yarn \
  && tar -xzf yarn-v$YARN_VERSION.tar.gz -C /opt/yarn --strip-components=1 \
  && ln -s /opt/yarn/bin/yarn /usr/local/bin/yarn \
  && ln -s /opt/yarn/bin/yarn /usr/local/bin/yarnpkg \
  && rm yarn-v$YARN_VERSION.tar.gz.asc yarn-v$YARN_VERSION.tar.gz
