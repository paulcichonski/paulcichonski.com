FROM node:4.0

## based on https://hub.docker.com/r/danlynn/ember-cli/

ENV EMBER_CLI_VERSION 1.13.8
ENV WATCHMAN_VERSION=v3.7.0

RUN npm install -g --unsafe-perm ember-cli@$EMBER_CLI_VERSION
RUN npm install -g bower
RUN npm install -g phantomjs

RUN cd /tmp && \
    git clone https://github.com/facebook/watchman.git && \
    cd watchman && \
    git checkout $WATCHMAN_VERSION && \
    ./autogen.sh && \
    ./configure --without-python && \
    make -j2 && \
    make install && \
    cd / && \
    rm -rf /tmp/watchman

RUN mkdir /opt/app
WORKDIR "/opt/app"
CMD ["ember", "help"]

