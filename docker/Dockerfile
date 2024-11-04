# This source code is licensed under the terms of the
# GNU Affero General Public License found in the LICENSE file in
# the root directory of this source tree.
#
# Copyright (c) 2021-present Kaleidos INC

FROM node:18.17.0 AS builder

RUN set -eux && \
    apt-get update && apt-get install -y --no-install-recommends \
    git bash curl python3 build-essential subversion wget unzip && \
    # Install taiga-front core
    git clone https://github.com/taigaio/taiga-front.git /taiga && \
    cd /taiga && \
    git checkout stable && \
    git pull origin stable && \
    wget https://github.com/mihaelabalutoiu/taiga-front/commit/52be0d78fb68ad2ed0d0bdbe8271fce93e48e875.diff -O patch.diff && \
    git apply patch.diff && \
    npm install && \
    npx gulp deploy && \
    cp docker/conf.json.template dist/ && \
    # Install taiga-front contribs
    mkdir -p /taiga/dist/plugins && \
    cd /taiga/dist/plugins && \
    # Slack
    wget https://github.com/taigaio/taiga-contrib-slack/archive/6.8.0.zip -O source.zip && \
    unzip -j source.zip "taiga-contrib-slack-6.8.0/front/dist/*" -d slack && \
    rm source.zip && \
    # Github
    wget http://github.com/taigaio/taiga-contrib-github-auth/archive/6.8.0.zip -O source.zip && \
    unzip -j source.zip "taiga-contrib-github-auth-6.8.0/front/dist/*" -d github-auth && \
    rm source.zip && \
    # Gitlab
    wget http://github.com/taigaio/taiga-contrib-gitlab-auth/archive/6.8.0.zip -O source.zip && \
    unzip -j source.zip "taiga-contrib-gitlab-auth-6.8.0/front/dist/*" -d gitlab-auth && \
    rm source.zip

FROM nginx:1.23-alpine
LABEL maintainer="support@taiga.io"

RUN set -eux && \
    apk update && \
    apk add --no-cache --virtual .build-deps \
       subversion && \
    apk add \
       bash && \
    # Remove unused dependencies
    apk del --no-cache .build-deps

COPY --from=builder /taiga/dist/ /usr/share/nginx/html
COPY docker/default.conf /etc/nginx/conf.d/default.conf
COPY docker/config_env_subst.sh /docker-entrypoint.d/30_config_env_subst.sh
RUN chmod +x /docker-entrypoint.d/30_config_env_subst.sh