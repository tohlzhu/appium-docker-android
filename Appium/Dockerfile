ARG BASE_IMAGE=ubuntu:22.04

FROM ${BASE_IMAGE}

ENV BASE_IMAGE=${BASE_IMAGE} \
    SDK_VERSION=commandlinetools-linux-8512546_latest \
    ANDROID_BUILD_TOOLS_VERSION=33.0.0 \
    APPIUM_VERSION=1.22.3

ENV DEBIAN_FRONTEND=noninteractive

#=============
# Set WORKDIR
#=============
WORKDIR /root

#==================
# General Packages
#------------------
# openjdk-8-jdk
#   Java
# ca-certificates
#   SSL client
# tzdata
#   Timezone
# zip
#   Make a zip file
# unzip
#   Unzip zip file
# curl
#   Transfer data from or to a server
# wget
#   Network downloader
# libqt5webkit5
#   Web content engine (Fix issue in Android)
# libgconf-2-4
#   Required package for chrome and chromedriver to run on Linux
# xvfb
#   X virtual framebuffer
# gnupg
#   Encryption software. It is needed for nodejs
# salt-minion
#   Infrastructure management (client-side)
# adb
#   for replacement of sdkmanager "platform-tools"
#==================
RUN apt-get -qqy update && \
    apt dist-upgrade -y && \
    apt-get -qqy --no-install-recommends install \
 #   openjdk-8-jdk \
    default-jdk \
    ca-certificates \
    tzdata \
    zip \
    unzip \
    curl \
    wget \
    libqt5webkit5 \
    libgconf-2-4 \
    xvfb \
    gnupg \
    salt-minion \
    adb \
    fastboot \
  && rm -rf /var/lib/apt/lists/*

# 1. Launch EC2 ARM based Instance (a1.metal / a1.2xlarge): (16 Gb RAM, 32Gb Disk), Ubuntu Server 22.04 LTS (HVM) ARM x64
# 2. sudo apt update && sudo apt upgrade
# 3. sudo apt install default-jdk python3-pip repo python-is-python3 unzip libpcre2-dev adb

#===============
# Set JAVA_HOME
#===============
# ENV JAVA_HOME="/usr/lib/jvm/java-8-openjdk-arm64/jre" \
ENV JAVA_HOME="/usr/lib/jvm/java-11-openjdk-arm64" \
    PATH=$PATH:$JAVA_HOME/bin

#=====================
# Install Android SDK
#=====================
#ENV ANDROID_DOWNLOAD_PATH=/root \
#    ANDROID_HOME=/opt/android \
#    ANDROID_TOOL_HOME=/opt/android/cmdline-tools
ENV ANDROID_DOWNLOAD_PATH=/root \
    ANDROID_HOME=/opt/android-sdk \
    ANDROID_TOOL_HOME=/opt/android-sdk/cmdline-tools

RUN wget -O tools.zip https://dl.google.com/android/repository/${SDK_VERSION}.zip && \
    unzip tools.zip && rm tools.zip && \
    chmod a+x -R ${ANDROID_DOWNLOAD_PATH} && \
    chown -R root:root ${ANDROID_DOWNLOAD_PATH} && \
    mkdir -p ${ANDROID_TOOL_HOME} && \
    mv cmdline-tools ${ANDROID_TOOL_HOME}/latest
ENV PATH=$PATH:${ANDROID_TOOL_HOME}/latest:${ANDROID_TOOL_HOME}/latest/bin

# https://askubuntu.com/questions/885658/android-sdk-repositories-cfg-could-not-be-loaded
RUN mkdir -p ~/.android && \
    touch ~/.android/repositories.cfg && \
    mkdir ${ANDROID_HOME}/platform-tools && \
    ln -s /usr/lib/android-sdk/platform-tools/* ${ANDROID_HOME}/platform-tools/ && \
    echo y | sdkmanager "build-tools;$ANDROID_BUILD_TOOLS_VERSION"
ENV PATH=$PATH:$ANDROID_HOME/platform-tools:$ANDROID_HOME/build-tools
#    echo y | sdkmanager "platform-tools" && \

#====================================
# Install latest nodejs, npm, appium
# Using this workaround to install Appium -> https://github.com/appium/appium/issues/10020 -> Please remove this workaround asap
#====================================
RUN curl -sL https://deb.nodesource.com/setup_16.x | bash && \
    apt-get -qqy install nodejs && \
    npm install -g appium@${APPIUM_VERSION} --unsafe-perm=true --allow-root && \
    exit 0 && \
    npm cache clean && \
    apt-get remove --purge -y npm && \
    apt-get autoremove --purge -y && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    apt-get clean

#==================================
# Fix Issue with timezone mismatch
#==================================
ENV TZ="US/Pacific"
RUN echo "${TZ}" > /etc/timezone

#===============
# Expose Ports
#---------------
# 4723
#   Appium port
#===============
EXPOSE 4723

#====================================================
# Scripts to run appium and connect to Selenium Grid
#====================================================
COPY entry_point.sh \
     generate_config.sh \
     wireless_connect.sh \
     wireless_autoconnect.sh \
     /root/

RUN chmod +x /root/entry_point.sh && \
    chmod +x /root/generate_config.sh && \
    chmod +x /root/wireless_connect.sh && \
    chmod +x /root/wireless_autoconnect.sh

#========================================
# Run xvfb and appium server
#========================================
CMD /root/wireless_autoconnect.sh && /root/entry_point.sh
