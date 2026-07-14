# vim:set ft=dockerfile:
#
# Copyright The CloudNativePG Contributors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
# 
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
FROM postgis/postgis:17-3.5

# Do not split the description, otherwise we will see a blank space in the labels
LABEL name="PostgreSQL + TimescaleDB + PostGIS + PGVector + PGVectorScale Container Images" \
	version="${PG_VERSION}" \
	release="127" \
	summary="PostgreSQL + TimescaleDB + PostGIS + PGVector + PGVectorScale Container images." \
	description="This Docker image contains PostgreSQL, TimescaleDB, PostGIS, PGVector, PGVectorScale and Barman Cloud based on Postgres 17-3.5."

LABEL org.opencontainers.image.description="This Docker image contains PostgreSQL, PostGIS, PGVector, PGVectorScale, TimescaleDB and Barman Cloud based on Postgres 17-3.5."

COPY requirements.txt /

# Install timescaledb 2.x
RUN apt-get update \
	&& apt-get install -y lsb-release wget \
	&& echo "deb https://packagecloud.io/timescale/timescaledb/debian/ $(lsb_release -c -s) main" | tee /etc/apt/sources.list.d/timescaledb.list \
	&& wget --quiet -O - https://packagecloud.io/timescale/timescaledb/gpgkey | apt-key add - \
	&& apt-get update \
	&& apt-get install -y "timescaledb-2-postgresql-${PG_MAJOR}" "timescaledb-toolkit-postgresql-${PG_MAJOR}" \
	&& apt-get remove -y lsb-release wget \
	&& 	rm -fr /tmp/* \
	&& 	rm -rf /var/lib/apt/lists/*

# Install pgvectorscale from source using Rust (supports PostgreSQL 16 and 17)
RUN set -xe; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		build-essential \
		curl \
		git \
		libssl-dev \
		pkg-config \
		postgresql-server-dev-${PG_MAJOR} \
	; \
	curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y; \
	. $HOME/.cargo/env; \
	git clone https://github.com/timescale/pgvectorscale.git /tmp/pgvectorscale; \
	cd /tmp/pgvectorscale/pgvectorscale; \
	$HOME/.cargo/bin/cargo install --locked cargo-pgrx --version 0.12.9; \
	$HOME/.cargo/bin/cargo pgrx init --pg${PG_MAJOR} pg_config; \
	$HOME/.cargo/bin/cargo pgrx install --release; \
	cd /; \
	rm -rf /tmp/pgvectorscale; \
	apt-get remove -y build-essential curl git postgresql-server-dev-${PG_MAJOR}; \
	apt-get autoremove -y; \
	rm -rf /var/lib/apt/lists/*;

# Install pgaudit
RUN set -xe; \
	apt-get update; \
	apt-mark hold "postgresql-${PG_MAJOR}" "postgresql-client-${PG_MAJOR}" postgresql-client-common postgresql-common; \
	apt-get install -y --no-install-recommends \
	"postgresql-${PG_MAJOR}-pgaudit" \
	"postgresql-${PG_MAJOR}-pg-failover-slots" \
	"postgresql-${PG_MAJOR}-pgrouting" \
	; \
	rm -fr /tmp/* ; \
	rm -rf /var/lib/apt/lists/*;

# Install barman-cloud
RUN set -xe; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
	python3-pip \
	python3-psycopg2 \
	python3-setuptools \
	; \
	pip3 install --upgrade pip; \
	# TODO: Remove --no-deps once https://github.com/pypa/pip/issues/9644 is solved
	pip3 install --no-deps -r requirements.txt; \
	rm -rf /var/lib/apt/lists/*;

# Install pgvector
RUN set -xe; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
	postgresql-${PG_MAJOR}-pgvector \
	; \
	rm -rf /var/lib/apt/lists/*;

# Change the uid of postgres to 26
RUN usermod -u 26 postgres
USER 26
