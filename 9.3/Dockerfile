FROM centos:7
MAINTAINER Alejandro Endo <theHilikus@gmail.com>

ENV PG_MAJOR="9.3"
ENV PGDATA="/var/lib/postgresql/data" GOSU_VERSION="1.7" LANG="en_US.utf8" PATH="/usr/pgsql-$PG_MAJOR/bin:$PATH"

# grab gosu for easy step-down from root

RUN set -x \
    	&& yum -y update \
	&& yum -y install wget \
	&& wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-amd64" \
	&& wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-amd64.asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
	&& gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
	&& rm -r "$GNUPGHOME" /usr/local/bin/gosu.asc \
	&& chmod +x /usr/local/bin/gosu \
	&& gosu nobody true \
        && yum clean all


# Install postgres

RUN yum -y install https://download.postgresql.org/pub/repos/yum/9.3/redhat/rhel-6-x86_64/pgdg-centos93-9.3-2.noarch.rpm epel-release \
    && yum -y install postgresql93-server.x86_64 \
    && yum clean all

# make the "en_US.UTF-8" locale so postgres will be utf-8 enabled by default
RUN localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8

RUN mkdir /docker-entrypoint-initdb.d


# make the sample config easier to munge (and "correct by default")
RUN mkdir /usr/share/postgresql/ \
    && mv -v /usr/pgsql-$PG_MAJOR/share/postgresql.conf.sample /usr/share/postgresql/ \
    && ln -sv /usr/share/postgresql/postgresql.conf.sample /usr/pgsql-$PG_MAJOR/share/ \
    && sed -ri "s!^#?(listen_addresses)\s*=\s*\S+.*!\1 = '*'!" /usr/share/postgresql/postgresql.conf.sample

RUN mkdir -p /var/run/postgresql && chown -R postgres /var/run/postgresql

VOLUME /var/lib/postgresql/data

COPY docker-entrypoint.sh /

ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 5432
CMD ["postgres"]
