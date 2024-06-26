#
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.
#

FROM ubuntu:22.04

# prepare the directory for pulsar related files
RUN mkdir /pulsar

RUN apt-get update && \
    apt-get install -y software-properties-common && \
    apt-get update && \
    apt-get install -y tig g++ cmake libssl-dev libcurl4-openssl-dev \
                liblog4cxx-dev google-mock libgtest-dev \
                libboost-dev libboost-program-options-dev libboost-system-dev libboost-python-dev \
                libxml2-utils wget apt-transport-https \
                curl doxygen clang-format \
                gnupg2 golang-go zip unzip libzstd-dev libsnappy-dev wireshark-dev

# Install Eclipse Temurin Package
RUN mkdir -p /etc/apt/keyrings \
     && wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc \
     && echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list \
     && apt-get update \
     && apt-get -y dist-upgrade \
     && apt-get -y install temurin-17-jdk

# Compile and install gtest & gmock
RUN cd /usr/src/googletest && \
    cmake . && \
    make && \
    make install

# Include gtest parallel to speed up unit tests
RUN git clone https://github.com/google/gtest-parallel.git

# Build protobuf 3.x.y from source since the default protobuf from Ubuntu's apt source is 2.x.y
RUN curl -O -L https://github.com/protocolbuffers/protobuf/releases/download/v3.17.3/protobuf-cpp-3.17.3.tar.gz && \
    tar xvfz protobuf-cpp-3.17.3.tar.gz && \
    cd protobuf-3.17.3/ && \
    CXXFLAGS=-fPIC ./configure && \
    make -j8 && make install && \
    cd .. && rm -rf protobuf-3.17.3/ protobuf-cpp-3.17.3.tar.gz
ENV LD_LIBRARY_PATH /usr/local/lib

## Website build dependencies

# Install Ruby-2.4.1
RUN (curl -sSL https://rvm.io/mpapis.asc | gpg --import -) && \
    (curl -sSL https://rvm.io/pkuczynski.asc | gpg --import -) && \
    (curl -sSL https://get.rvm.io | bash -s stable)
ENV PATH "$PATH:/usr/local/rvm/bin"
RUN rvm install 2.4.1

# Install nodejs and yarn
RUN curl -sL https://deb.nodesource.com/setup_12.x | bash -
RUN apt-get install -y nodejs
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt-get update && apt-get install yarn

# Install crowdin
RUN wget https://artifacts.crowdin.com/repo/deb/crowdin.deb -O crowdin.deb
RUN dpkg -i crowdin.deb

# Install PIP
RUN curl https://bootstrap.pypa.io/get-pip.py  | python3 -
RUN pip3 --no-cache-dir install pdoc
#
# Installation
ARG MAVEN_VERSION=3.6.3
ARG MAVEN_FILENAME="apache-maven-${MAVEN_VERSION}-bin.tar.gz"
ARG MAVEN_HOME=/opt/maven
ARG MAVEN_URL="http://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries/${MAVEN_FILENAME}"
ARG MAVEN_TMP="/tmp/${MAVEN_FILENAME}"
RUN wget --no-verbose -O ${MAVEN_TMP} ${MAVEN_URL}

# Cleanup
RUN tar xzf ${MAVEN_TMP}  -C /opt/ \
        && ln -s /opt/apache-maven-${MAVEN_VERSION} ${MAVEN_HOME} \
        && ln -s ${MAVEN_HOME}/bin/mvn /usr/local/bin

RUN unset MAVEN_VERSION
