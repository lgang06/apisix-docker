#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

FROM centos:7


ARG APISIX_VERSION=2.13.3
ARG APISIX_WORKER_DIR=/usr/local/apisix
ARG YUM_COMMON_DEPS="git gcc make"
ARG APISIX_SOURCE_PATH=/tmp/apache-apisix-${APISIX_VERSION}
LABEL apisix_version="${APISIX_VERSION}"

COPY ./apisix-base-2.13.3-0.el7.x86_64.rpm /tmp/apisix-base-2.13.3-0.el7.x86_64.rpm
COPY ./linux-install-luarocks.sh /tmp/linux-install-luarocks.sh
RUN yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo && \
    yum -y install openresty-openssl111 openresty-openssl111-devel openresty-pcre openresty-pcre-devel openresty-zlib unzip wget && yum install -y /tmp/apisix-base-2.13.3-0.el7.x86_64.rpm \
        && yum -y install ${YUM_COMMON_DEPS} openldap-devel which \
	&& yum clean all \
	&& sed -i 's/PASS_MAX_DAYS\t99999/PASS_MAX_DAYS\t60/g' /etc/login.defs \
        && bash /tmp/linux-install-luarocks.sh \
        && curl -k "https://downloads.apache.org/apisix/${APISIX_VERSION}/apache-apisix-${APISIX_VERSION}-src.tgz" -o ${APISIX_SOURCE_PATH}-src.tgz \
        && mkdir -p /tmp/apache-apisix-${APISIX_VERSION} && tar -zxvf /tmp/apache-apisix-${APISIX_VERSION}-src.tgz -C ${APISIX_SOURCE_PATH} \
        && cd ${APISIX_SOURCE_PATH} \ && luarocks config variables.PCRE_DIR /usr/local/openresty/pcre && luarocks config variables.PCRE_INCDIR /usr/local/openresty/pcre/include \
        && sed -i 's/nginx-lua-prometheus = 0.20220127/nginx-lua-prometheus = 0.20220527/' rockspec/apisix-master-0.rockspec > /dev/null \
        && LUAROCKS_SERVER=https://luarocks.cn make deps && ENV_INST_LUADIR=${APISIX_WORKER_DIR} make install \
        && cp -r ${APISIX_SOURCE_PATH}/deps ${APISIX_WORKER_DIR}/deps \
        && yum install -y perl-App-cpanminus.noarch && `cpanm --notest Test::Nginx IPC::Run > build.log 2>&1 || (cat build.log && exit 1)` && echo "export TEST_NGINX_BINARY=/usr/local/openresty/bin/openresty >> /root/.bash_profile" \
        && ln -s /usr/local/openresty/nginx/sbin/nginx /usr/local/sbin/nginx \
        && source /root/.bash_profile \
        && rm -rf /tmp/* && yum -y remove ${YUM_COMMON_DEPS} \
        && yum clean all
WORKDIR $APISIX_WORKER_DIR


# forward request and error logs to docker log collector
RUN ln -sf /dev/stdout /usr/local/apisix/logs/access.log \
    && ln -sf /dev/stderr /usr/local/apisix/logs/error.log

EXPOSE 9080 9443

COPY ./docker-entrypoint.sh /docker-entrypoint.sh

ENTRYPOINT ["/docker-entrypoint.sh"]

CMD ["docker-start"]

STOPSIGNAL SIGQUIT
