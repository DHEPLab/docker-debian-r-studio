FROM debian:13.3

# OCI Labels to link container to repository
LABEL org.opencontainers.image.source="https://github.com/DHEPLab/docker-debian-r-studio"
LABEL org.opencontainers.image.description="R development container"
LABEL org.opencontainers.image.licenses="MIT"

# Install R Studio
RUN apt-get update \
  && apt-get install -y \
      gdebi-core \
      gnupg2 \
      libcairo2-dev \
      libcurl4-openssl-dev \
      libfontconfig1-dev \
      libgdal-dev  \
      libgeos-dev \
      libproj-dev  \
      libssl-dev \
      libudunits2-dev \
      libxml2-dev \
      pandoc \
      r-base \
      r-base-dev \
      r-cran-tinytex \
      wget \
  && wget https://s3.amazonaws.com/rstudio-ide-build/server/jammy/amd64/rstudio-server-2026.02.0-daily-95-amd64.deb \
  && gdebi -n rstudio-server-2026.02.0-daily-95-amd64.deb \
  && rm -rf rstudio-server-2026.02.0-daily-95-amd64.deb
 
# Clean Up apt-get cache
RUN apt-get clean \
  && rm -rf /var/lib/apt/lists/* \
  && rm -rf /var/lib/rstudio-server/r-versions

COPY build/sssd.conf /etc/sssd/sssd.conf
COPY build/logging.conf /etc/rstudio/logging.conf
  
# Set default env values
ENV RSW_LICENSE=""
ENV RSW_LICENSE_SERVER=""
ENV RSW_TESTUSER=rstudio
ENV RSW_TESTUSER_PASSWD=rstudio
ENV RSW_TESTUSER_UID=10000
ENV RSW_LAUNCHER=true
ENV RSW_LAUNCHER_TIMEOUT=10
ENV DIAGNOSTIC_DIR=/var/log/rstudio
ENV DIAGNOSTIC_ENABLE=false
ENV DIAGNOSTIC_ONLY=false
  
RUN useradd -m rstudio \
  && echo "rstudio:rstudio" | chpasswd \
  && mkdir -p /home/rstudio/.rstudio/monitored/user-settings \
  && chown -R rstudio:rstudio /home/rstudio/.rstudio
  
# Create log dir
RUN mkdir -p /var/lib/rstudio-server/monitor/log \
  && chown -R rstudio:rstudio /var/lib/rstudio-server/monitor \
  && chmod 600 /etc/sssd/sssd.conf \
  && echo '\n# allow home directory creation\nsession required pam_mkhomedir.so skel=/etc/skel umask=0022' >> /etc/pam.d/common-session

RUN echo "tail -f /dev/null" >> /usr/sbin/rstudio-server

RUN R -e "install.packages( \
  c( \
    'devtools' \
    , 'requiRements' \
  ), \
  dependencies = TRUE \
)"

COPY build/requirements.txt /tmp/requirements.txt

RUN R -e "requiRements::install(path_to_requirements = '/tmp/requirements.txt')" \
  && rm -rf /tmp/requirements.txt

  USER rstudio

  CMD ["rstudio-server","start"]
  