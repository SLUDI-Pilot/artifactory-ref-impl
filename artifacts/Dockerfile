# Use Debian-based base image instead of non-existent nginx:1.25.3-bullseye
FROM debian:bullseye-slim

# can be passed during Docker build as build time environment for github branch to pickup configuration from.
ARG container_user=mosip

# can be passed during Docker build as build time environment for github branch to pickup configuration from.
ARG container_user_group=mosip

# can be passed during Docker build as build time environment for github branch to pickup configuration from.
ARG container_user_uid=1001

# can be passed during Docker build as build time environment for github branch to pickup configuration from.
ARG container_user_gid=1001

# Basic environment early (used by later steps)
ENV MAVEN_VERSION=3.8.8
ENV MAVEN_HOME=/usr/local/maven
ENV PATH=$MAVEN_HOME/bin:$PATH
ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
ENV work_dir=/home/${container_user}
ENV base_path=/usr/share/nginx/html/artifactory

# Install packages (split & defensive). We install curl for reliable downloads.
RUN set -eux; \
    apt-get update --fix-missing -y; \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        ca-certificates \
        gnupg \
        curl \
        wget \
        unzip \
        zip \
        openjdk-11-jre-headless \
        nginx \
        adduser; \
    apt-get clean; \
    rm -rf /var/lib/apt/lists/*

# Create group (if missing) and user (if missing) — do not fail if already present.
RUN set -eux; \
    if ! getent group "${container_user_group}" >/dev/null 2>&1 ; then \
        groupadd -g "${container_user_gid}" "${container_user_group}"; \
    else \
        echo "group ${container_user_group} already exists"; \
    fi; \
    if ! id -u "${container_user}" >/dev/null 2>&1 ; then \
        useradd -u "${container_user_uid}" -g "${container_user_group}" -s /bin/sh -m "${container_user}"; \
    else \
        echo "user ${container_user} already exists"; \
    fi

# Download & install Maven using curl with retries (writes to /usr/local/maven)
RUN set -eux; \
    MAVEN_TGZ_URL="https://dlcdn.apache.org/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz"; \
    TMP_TGZ="/tmp/apache-maven-${MAVEN_VERSION}.tar.gz"; \
    # retry loop for transient network failures
    i=0; \
    until [ $i -ge 5 ]; do \
        curl -fSL --retry 3 --retry-delay 5 -o "${TMP_TGZ}" "${MAVEN_TGZ_URL}" && break; \
        i=$((i+1)); sleep 2; \
    done; \
    [ -f "${TMP_TGZ}" ]; \
    tar -xzf "${TMP_TGZ}" -C /usr/local; \
    mv /usr/local/apache-maven-${MAVEN_VERSION} "${MAVEN_HOME}"; \
    rm -f "${TMP_TGZ}"

# Prepare directories used by nginx and application, set ownership by numeric UID:GID
RUN set -eux; \
    mkdir -p /var/run/nginx /var/tmp/nginx /var/log/nginx ${base_path}; \
    # create logs dir and redirect to stdout/stderr (prevents permission denied at runtime)
    mkdir -p /var/log/nginx; \
    rm -f /var/log/nginx/access.log /var/log/nginx/error.log || true; \
    ln -sf /dev/stdout /var/log/nginx/access.log; \
    ln -sf /dev/stderr /var/log/nginx/error.log; \
    # ensure webroot exists
    mkdir -p ${base_path}; \
    # chown using numeric ids to avoid name mismatch problems
    chown -R "${container_user_uid}:${container_user_gid}" /usr/share/nginx /var/run/nginx /var/tmp/nginx /var/log/nginx ${work_dir} ${base_path} || true

# copy nginx config (if present in build context)
COPY ./nginx.conf /etc/nginx/nginx.conf

# Keep your original environment variables (shortened here to show pattern)
ENV biosdk_client_zip_path=${base_path}/libs-release-local/biosdk/mock/0.9

# environment variable for Demosdk Zip path
ENV demosdk_zip_path=${base_path}/libs-release-local/demosdk/default/

# environment variable for Biosdk lib Zip path
ENV biosdk_lib_zip_path=${base_path}/libs-release-local/biosdk/

# environment variable for NeuroTech-Biosdk lib Zip path
ENV neurotech_biosdk_lib_zip_path=${base_path}/libs-release-local/neurotech/

# environment variable for Biosdk lib Zip path
ENV jpegsdk_lib_zip_path=${base_path}/libs-release-local/jpeg-sdk/

# environment variable for Cache Provider Jar path
ENV cache_path=${base_path}/libs-release-local/cache

# environment variable for ida shell scripts path
ENV ida_sh_path=${base_path}/libs-release-local/deployment/docker/id-authentication

# environment variable for hsm client path
ENV hsm_client_path=${base_path}/libs-release-local/hsm

# environment variable for mosip-plugins zip path
ENV mosip_plugins_zip_path=${base_path}/libs-release-local/mosip-plugins

# environment valiable for sdk jar
ENV sdk_path=${base_path}/libs-release-local/mock-sdk/1.1.5/

# environment valiable for sdk jar
ENV jpeg_sdk_path=${base_path}/libs-release-local/jpeg-mock-sdk/1.1.5/

# environment valiable for image-compressor jar
ENV image_compressor_path=${base_path}/libs-release-local/compressor/

# environment variable for Kernel Jars
ENV kernel_jar_path=${base_path}/libs-release-local/io/mosip/kernel

# environment variable for regproc reprocessor ceylon cache jar path
ENV test_jar_path=${base_path}/libs-release-local/io/mosip/testing/

# environment variable for ref-idobjectvalidator jar
ENV idobject_jar_path=${base_path}/libs-release-local/io/mosip/kernel/kernel-ref-idobjectvalidator

# environment variable for regproc auth jar
ENV regproc_jar_path=${base_path}/libs-release-local/io/mosip/kernel/kernel-auth-adapter

# environment variable for prereg transliteration jar path
ENV icu4j_jar_path=${base_path}/libs-release-local/icu4j

# environment variable for i18n zip path
ENV i18n_zip_path=${base_path}/libs-release-local/i18n

# environment variable for theme zip path
ENV theme_zip_path=${base_path}/libs-release-local/theme

# environment variable for idp auth wrapper zip path
ENV idp_auth_wrapper_lib_zip_path=${base_path}/libs-release-local/idp/idp-auth-wrapper

# environment variable for esignet wrappers zip path
ENV esignet_wrapper_lib_zip_path=${base_path}/libs-release-local/esignet

# environment variable for Clamav dependency
ENV clamav_path=${base_path}/libs-release-local/clamav

ENV regclient_jar_path=${base_path}/libs-release-local

ENV master_template_path=${base_path}/libs-release-local/master-templates

# environment variable for child auth filter jar path
ENV child_auth_filter_jar_path=${base_path}/libs-release-local/io/mosip/authentication/authentication-ref-impl

# environment variable for script path
ENV scripts_path=/home/mosip/scripts

# environment variable for jar version to be downloaed, this needed to be passed buildtime
ENV version=1.2.1-SNAPSHOT
ENV idp_auth_wrapper_version=0.0.1-SNAPSHOT

# Create directories required by your app (keeps clearer error output if one fails)
RUN set -eux; \
    mkdir -p ${biosdk_client_zip_path}/biosdk-client ${demosdk_zip_path}/demosdk \
    ${biosdk_lib_zip_path}/biosdk-lib ${cache_path} ${ida_sh_path} ${hsm_client_path} \
    ${mosip_plugins_zip_path} ${kernel_jar_path} ${test_jar_path} ${idobject_jar_path} \
    ${regproc_jar_path} ${sdk_path} ${image_compressor_path}/image-compressor \
    ${icu4j_jar_path} ${clamav_path} ${i18n_zip_path} ${theme_zip_path} \
    ${child_auth_filter_jar_path} ${base_path}/libs-release-local/reg-client \
    ${scripts_path} ${master_template_path} ${jpeg_sdk_path} ${jpegsdk_lib_zip_path}/jpeg-sdk-lib/ \
    ${idp_auth_wrapper_lib_zip_path} ${esignet_wrapper_lib_zip_path}/esignet-wrapper \
    ${neurotech_biosdk_lib_zip_path}/windows-sdk; \
    chown -R "${container_user_uid}:${container_user_gid}" ${base_path} ${work_dir} || true

# COPY resources (kept exactly as you had them — these will fail early if paths are missing)
COPY /src/sdk/biosdk/install.sh ${biosdk_client_zip_path}/biosdk-client/

COPY src/sdk/neuroTech-bioSdk/sdk-dependency-neurotech.zip ${neurotech_biosdk_lib_zip_path}/windows-sdk/

COPY /src/sdk/demosdk/install.sh ${demosdk_zip_path}/demosdk/

COPY /src/deployment/docker/id-authentication/* ${ida_sh_path}/

COPY /src/hsm/client.zip ${hsm_client_path}/

COPY /src/mosip-plugins/sign-in-with-esignet/sign-in-with-esignet.zip ${mosip_plugins_zip_path}/

COPY /src/testing/regproc-reprocessor-ceylon-cache-repo.zip ${test_jar_path}/

COPY /src/testing/glowroot.zip ${test_jar_path}/

COPY /src/i18n/regclient-i18n-bundle/*  ${work_dir}/resources/

COPY /src/i18n/admin-entity-spec-bundle/*  ${work_dir}/admin-entity-spec-bundle/

COPY /src/i18n/admin-i18n-bundle/*  ${work_dir}/admin-i18n-bundle/

COPY /src/i18n/pmp-entity-spec-bundle/*  ${work_dir}/pmp-entity-spec-bundle/

COPY /src/i18n/pmp-i18n-bundle/*  ${work_dir}/pmp-i18n-bundle/

COPY /src/i18n/pre-registration-i18n-bundle/*  ${work_dir}/pre-registration-i18n-bundle/

COPY /src/master-templates/* ${work_dir}/master-templates/

COPY /src/i18n/idp-i18n-bundle/*  ${work_dir}/idp-i18n-bundle/

COPY /src/i18n/oidc-demo-i18n-bundle/*  ${work_dir}/oidc-demo-i18n-bundle/

COPY /src/i18n/mock-relying-party-i18n-bundle/*  ${work_dir}/mock-relying-party-i18n-bundle/

COPY /src/i18n/esignet-i18n-bundle/*  ${work_dir}/esignet-i18n-bundle/

COPY /src/theme/esignet-theme/* ${work_dir}/esignet-theme/

COPY /src/jre/zulu11.41.23-ca-fx-jre11.0.8-win_x64.zip ${regclient_jar_path}/zulu11.41.23-ca-fx-jre11.0.8-win_x64.zip

COPY /src/icu4j/* ${icu4j_jar_path}/

COPY /src/auth/* ${regproc_jar_path}/

COPY ./configure.sh ${work_dir}/

COPY pom.xml ${work_dir}/

# Run maven build (runs as root during build so maven can write to work_dir)
RUN set -eux; \
    cd ${work_dir}; \
    # ensure mvn exists and print version for debugging
    mvn -v || true; \
    mvn dependency:resolve && mvn clean install

# Run your configure script
RUN set -eux; \
    bash ${work_dir}/configure.sh

# update nginx default listen port to 8080 (if default config present)
RUN set -eux; \
    if [ -f /etc/nginx/sites-enabled/default ]; then \
        sed -i -e '/listen/!b' -e '/80;/!b' -e 's/80;/8080;/' /etc/nginx/sites-enabled/default; \
    fi

# switch to the application user
USER ${container_user_uid}:${container_user_gid}

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
