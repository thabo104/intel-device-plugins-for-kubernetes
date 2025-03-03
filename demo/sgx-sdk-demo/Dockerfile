FROM ubuntu:22.04 AS builder

WORKDIR /root

RUN apt-get update && \
    env DEBIAN_FRONTEND=noninteractive apt-get install -y \
    wget \
    unzip \
    protobuf-compiler \
    libprotobuf-dev \
    build-essential \
    cmake \
    pkg-config \
    gdb \
    vim \
    python3 \
    git \
    gnupg \
 && apt-get -y -q upgrade \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# SGX SDK is installed in /opt/intel directory.
WORKDIR /opt/intel

ARG DCAP_VERSION=DCAP_1.15

RUN echo "deb [arch=amd64 signed-by=/usr/share/keyrings/intel-sgx.gpg] https://download.01.org/intel-sgx/sgx_repo/ubuntu jammy main" | \
    tee -a /etc/apt/sources.list.d/intel-sgx.list \
 && wget -qO - https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key | \
    gpg --dearmor --output /usr/share/keyrings/intel-sgx.gpg \
 && apt-get update \
 && env DEBIAN_FRONTEND=noninteractive apt-get install -y \
    libsgx-dcap-ql-dev \
    libsgx-dcap-default-qpl-dev \
    libsgx-quote-ex-dev

# Install SGX SDK
ARG SGX_SDK_URL=https://download.01.org/intel-sgx/sgx-linux/2.18/distro/ubuntu22.04-server/sgx_linux_x64_sdk_2.18.100.3.bin
RUN wget ${SGX_SDK_URL} \
 && export SGX_SDK_INSTALLER=$(basename $SGX_SDK_URL) \
 && chmod +x $SGX_SDK_INSTALLER \
 && echo "yes" | ./$SGX_SDK_INSTALLER \
 && rm $SGX_SDK_INSTALLER

RUN git clone -b $DCAP_VERSION https://github.com/intel/SGXDataCenterAttestationPrimitives.git

RUN cd sgxsdk/SampleCode/SampleEnclave \
    && . /opt/intel/sgxsdk/environment \
    && make \
    && cd -

RUN cd SGXDataCenterAttestationPrimitives/SampleCode/QuoteGenerationSample \
    && . /opt/intel/sgxsdk/environment \
    && make \
    && cd -

FROM ubuntu:22.04

RUN apt-get update && \
    apt-get install -y \
    wget \
    gnupg-agent

# Add 01.org to apt for SGX packages and install SGX runtime components
RUN echo "deb [arch=amd64 signed-by=/usr/share/keyrings/intel-sgx.gpg] https://download.01.org/intel-sgx/sgx_repo/ubuntu jammy main" | \
    tee -a /etc/apt/sources.list.d/intel-sgx.list \
 && wget -qO - https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key | \
    gpg --dearmor --output /usr/share/keyrings/intel-sgx.gpg \
 && apt-get update \
 && env DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    libsgx-enclave-common \
    libsgx-urts \
    libsgx-quote-ex \
    libsgx-dcap-ql \
    libsgx-dcap-default-qpl \
 && mkdir -p /opt/intel/sgx-sample-app/ \
 && mkdir -p /opt/intel/sgx-quote-generation/

COPY --from=builder /opt/intel/sgxsdk/SampleCode/SampleEnclave/app /opt/intel/sgx-sample-app/sgx-sample-app
COPY --from=builder /opt/intel/sgxsdk/SampleCode/SampleEnclave/enclave.signed.so /opt/intel/sgx-sample-app/enclave.signed.so

COPY --from=builder /opt/intel/SGXDataCenterAttestationPrimitives/SampleCode/QuoteGenerationSample/app /opt/intel/sgx-quote-generation/sgx-quote-generation
COPY --from=builder /opt/intel/SGXDataCenterAttestationPrimitives/SampleCode/QuoteGenerationSample/enclave.signed.so /opt/intel/sgx-quote-generation/enclave.signed.so

ENTRYPOINT /opt/intel/sgx-sample-app/sgx-sample-app
