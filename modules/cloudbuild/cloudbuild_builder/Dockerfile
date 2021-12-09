# Copyright 2021 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM gcr.io/cloud-builders/gcloud-slim AS build-env

# Use ARG so that values can be overriden by user/cloudbuild
ARG TERRAFORM_VERSION=1.0.0
ARG TERRAFORM_VERSION_SHA256SUM=8be33cc3be8089019d95eb8f546f35d41926e7c1e5deff15792e969dde573eb5
ARG TERRAFORM_VALIDATOR_RELEASE=0.11.0

ENV ENV_TERRAFORM_VERSION=$TERRAFORM_VERSION
ENV ENV_TERRAFORM_VERSION_SHA256SUM=$TERRAFORM_VERSION_SHA256SUM
ENV ENV_TERRAFORM_VALIDATOR_RELEASE=$TERRAFORM_VALIDATOR_RELEASE

COPY entrypoint.bash /builder/entrypoint.bash

RUN apt-get update && \
   /builder/google-cloud-sdk/bin/gcloud -q components install alpha beta && \
    apt-get -y install curl jq unzip git ca-certificates bash-static autoconf build-essential gettext libcurl4-openssl-dev && \
    curl https://releases.hashicorp.com/terraform/${ENV_TERRAFORM_VERSION}/terraform_${ENV_TERRAFORM_VERSION}_linux_amd64.zip \
      > terraform_linux_amd64.zip && \
    echo "${ENV_TERRAFORM_VERSION_SHA256SUM} terraform_linux_amd64.zip" > terraform_SHA256SUMS && \
    sha256sum -c terraform_SHA256SUMS --status && \
    unzip terraform_linux_amd64.zip -d /builder/terraform && \
    rm -f terraform_linux_amd64.zip && \
    gsutil cp gs://terraform-validator/releases/v${ENV_TERRAFORM_VALIDATOR_RELEASE}/terraform-validator_linux_amd64-${ENV_TERRAFORM_VALIDATOR_RELEASE}.tar.gz /terraform-validator-linux-amd64-${ENV_TERRAFORM_VALIDATOR_RELEASE}.tar.gz && \
    tar -xzf terraform-validator-linux-amd64-${ENV_TERRAFORM_VALIDATOR_RELEASE}.tar.gz -C /builder/terraform/ && \
    chmod +x /builder/terraform/terraform-validator && \
    git clone https://github.com/git/git.git && \
    mkdir /git-static && \
    cd git && \
    make configure && \
    ./configure prefix=/git-static/ \
    --without-tcltk \
    #--with-openssl \
    #--with-curl \
    NO_GETTEXT=YesPlease \
    NO_SVN_TESTS=YesPlease \
    NO_REGEX=YesPlease \
    USE_LIBPCRE2=NoThanks \
    NO_SYS_POLL_H=1 \
    CFLAGS="${CFLAGS} `pkg-config -static –libs libcurl`" && \
    make && \
    make install && \
    apt-get --purge -y autoremove && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

FROM gcr.io/distroless/python3-debian11:debug
WORKDIR /builder
COPY --from=build-env /builder ./
COPY --from=build-env /bin/bash-static /bin/bash
COPY --from=build-env /git-static/bin/git /bin/git
COPY --from=build-env /git-static /git-static
COPY --from=build-env /usr/lib/x86_64-linux-gnu/ /usr/lib/x86_64-linux-gnu/
ENV PATH=/bin:/builder/terraform:/builder/google-cloud-sdk/bin:/bin:$PATH
ENTRYPOINT ["/builder/entrypoint.bash"]
