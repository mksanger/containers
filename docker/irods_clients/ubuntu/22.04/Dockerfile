# syntax=docker/dockerfile:1

ARG BASE_IMAGE=ubuntu:22.04
FROM $BASE_IMAGE as installer

ARG IRODS_VERSION="4.3-nightly"
ARG BATON_VERSION="4.1.0"
ARG HTSLIB_VERSION="1.18"
ARG SAMTOOLS_VERSION="1.18"
ARG BCFTOOLS_VERSION="1.18"
ARG HTSLIB_PLUGINS_VERSION="201712"

COPY . /opt/docker/irods_clients

RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections && \
    apt-get update && \
    apt-get install -q -y --no-install-recommends \
    apt-utils \
    ca-certificates \
    curl \
    dirmngr \
    gpg \
    gpg-agent \
    lsb-release \
    locales && \
    locale-gen en_GB en_GB.UTF-8 && \
    localedef -i en_GB -c -f UTF-8 -A /usr/share/locale/locale.alias en_GB.UTF-8

ENV LANG=en_GB.UTF-8 \
    LANGUAGE=en_GB \
    LC_ALL=en_GB.UTF-8

RUN curl -sSL https://packages.irods.org/irods-signing-key.asc | apt-key add - && \
    echo "deb [arch=amd64] https://packages.irods.org/apt/ $(lsb_release -sc) main" |\
    tee /etc/apt/sources.list.d/renci-irods.list && \
    apt-get update

ENV NIGHTLY_URL_BASE=https://github.com/wtsi-npg/irods_development_environment/releases/download/nightly/

RUN curl -sSL \
    -O ${NIGHTLY_URL_BASE}irods-dev_4.3.0-1.jammy_amd64.deb \
    -O ${NIGHTLY_URL_BASE}irods-runtime_4.3.0-1.jammy_amd64.deb \
    -O ${NIGHTLY_URL_BASE}irods-icommands_4.3.0-1.jammy_amd64.deb

RUN ls -l && apt-get install -y ./*.deb

RUN apt-get update && \
    apt-get install -q -y --no-install-recommends \
    autoconf \
    automake \
    build-essential \
    less \
    libtool \
    pkg-config \
    python3-sphinx \
    ssh \
    libdeflate-dev \
    libjansson-dev \
    libbz2-dev \
    libcurl3-dev \
    liblzma-dev \
    zlib1g-dev

ENV CPPFLAGS="-I/usr/include/irods"

RUN cd /tmp && \
    curl -sSL -O "https://github.com/wtsi-npg/baton/releases/download/${BATON_VERSION}/baton-${BATON_VERSION}.tar.gz" && \
    tar xfz baton-${BATON_VERSION}.tar.gz && \
    cd baton-${BATON_VERSION} && \
    ./configure && \
    make install

RUN cd /tmp && \
    curl -sSL -O "https://github.com/samtools/htslib/releases/download/${HTSLIB_VERSION}/htslib-${HTSLIB_VERSION}.tar.bz2" && \
    tar xfj htslib-${HTSLIB_VERSION}.tar.bz2 && \
    cd htslib-${HTSLIB_VERSION} && \
    ./configure --enable-plugins --without-curses && \
    make install && \
    ldconfig

RUN cd /tmp && \
    curl -sSL -O "https://github.com/samtools/samtools/releases/download/${SAMTOOLS_VERSION}/samtools-${SAMTOOLS_VERSION}.tar.bz2" && \
    tar xfj samtools-${SAMTOOLS_VERSION}.tar.bz2 && \
    cd samtools-${SAMTOOLS_VERSION} && \
    ./configure --with-htslib=system --without-curses && \
    make install

RUN cd /tmp && \
    curl -sSL -O "https://github.com/samtools/bcftools/releases/download/${BCFTOOLS_VERSION}/bcftools-${BCFTOOLS_VERSION}.tar.bz2" && \
    tar xfj bcftools-${BCFTOOLS_VERSION}.tar.bz2 && \
    cd bcftools-${BCFTOOLS_VERSION} && \
    ./configure --with-htslib=system --without-curses && \
    make install

RUN cd /tmp && \
    git clone --depth 1 --branch ${HTSLIB_PLUGINS_VERSION} "https://github.com/samtools/htslib-plugins.git" && \
    cd htslib-plugins && \
    make install

FROM $BASE_IMAGE 

ARG DOCKER_IMAGE
ARG DOCKER_TAG
ARG IRODS_VERSION="4.3-nightly"

RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections && \
    apt-get update && \
    apt-get install -q -y --no-install-recommends \
    apt-utils \
    ca-certificates \
    curl \
    dirmngr \
    gpg \
    gpg-agent \
    lsb-release \
    locales && \
    locale-gen en_GB en_GB.UTF-8 && \
    localedef -i en_GB -c -f UTF-8 -A /usr/share/locale/locale.alias en_GB.UTF-8

ENV LANG=en_GB.UTF-8 \
    LANGUAGE=en_GB \
    LC_ALL=en_GB.UTF-8

RUN curl -sSL https://packages.irods.org/irods-signing-key.asc | apt-key add - && \
    echo "deb [arch=amd64] https://packages.irods.org/apt/ $(lsb_release -sc) main" |\
    tee /etc/apt/sources.list.d/renci-irods.list && \
    apt-get update
    
ENV NIGHTLY_URL_BASE=https://github.com/wtsi-npg/irods_development_environment/releases/download/nightly/

RUN curl -sSL \
    -O ${NIGHTLY_URL_BASE}irods-runtime_4.3.0-1.jammy_amd64.deb \
    -O ${NIGHTLY_URL_BASE}irods-icommands_4.3.0-1.jammy_amd64.deb

RUN ls -l && apt-get install -y ./*.deb

RUN apt-get install -q -y --no-install-recommends \
    gnuplot \
    jq \
    libdeflate0 \
    libjansson4 \
    libbz2-1.0 \
    libcurl4 \
    liblzma5 \
    perl \
    zlib1g \
    unattended-upgrades && \
    unattended-upgrade -v && \
    apt-get remove -q -y unattended-upgrades && \
    apt-get autoremove -q -y && \
    apt-get clean -q -y && \
    rm -rf /var/lib/apt/lists/*

# Copy the installed baton and samtools applications
COPY --from=installer /usr/local /usr/local

# Copy the singularity-wrapper scripts
COPY --from=singularity ./scripts/* /usr/local/bin/
COPY --from=installer /opt/docker/irods_clients/manifest.json /usr/local/irods_clients/etc/manifest.json
COPY --from=installer /opt/docker/irods_clients/scripts/docker-entrypoint.sh /usr/local/irods_clients/bin/docker-entrypoint.sh

# Configure the singularity-wrapper script
ENV MANIFEST_PATH="/usr/local/irods_clients/etc/manifest.json"
ENV DOCKER_IMAGE="$DOCKER_IMAGE"
ENV DOCKER_TAG="$DOCKER_TAG"

RUN ldconfig

ENTRYPOINT ["/usr/local/irods_clients/bin/docker-entrypoint.sh"]

CMD ["/bin/bash"]