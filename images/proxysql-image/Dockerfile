FROM centos:7
MAINTAINER Percona Development <info@percona.com>

# the numeric UID is needed for OpenShift
RUN groupadd -g 1001 proxysql
RUN useradd -u 1001 -r -g 1001 -s /sbin/nologin \
            -c "Default Application User" proxysql

# check repository package signature in secure way
RUN export GNUPGHOME="$(mktemp -d)" \
    && gpg --keyserver ha.pool.sks-keyservers.net --recv-keys 430BDF5C56E7C94E848EE60C1C4CBDCDCD2EFD2A \
    && gpg --export --armor 430BDF5C56E7C94E848EE60C1C4CBDCDCD2EFD2A > ${GNUPGHOME}/RPM-GPG-KEY-Percona \
    && rpmkeys --import ${GNUPGHOME}/RPM-GPG-KEY-Percona /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7 \
    && curl -L -o /tmp/percona-release.rpm https://repo.percona.com/centos/7/RPMS/noarch/percona-release-0.1-8.noarch.rpm \
    && rpmkeys --checksig /tmp/percona-release.rpm \
    && yum install -y /tmp/percona-release.rpm \
    && rm -rf "$GNUPGHOME" /tmp/percona-release.rpm \
    && rpm --import /etc/pki/rpm-gpg/PERCONA-PACKAGING-KEY \
    && percona-release disable all \
    && percona-release enable percona release

# we need licenses from docs
RUN sed -i '/nodocs/d' /etc/yum.conf

RUN yum install -y Percona-Server-client-57 vim curl sysvinit-tools
RUN percona-release enable percona experimental \
    && yum install -y proxysql-2.0.3

COPY LICENSE /licenses/LICENSE.Dockerfile
RUN cp /usr/share/doc/proxysql-2.0.3/LICENSE /licenses/LICENSE.proxysql

COPY proxysql.cnf /etc/proxysql/proxysql.cnf
COPY proxysql-admin.cnf /etc/proxysql-admin.cnf
RUN chmod 664 /etc/proxysql/proxysql.cnf /etc/proxysql-admin.cnf \
    && chown 1001:0 /etc/proxysql/proxysql.cnf /etc/proxysql-admin.cnf

COPY proxysql-entry.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

COPY add_pxc_nodes.sh /usr/bin/add_pxc_nodes.sh
RUN chmod a+x /usr/bin/add_pxc_nodes.sh

COPY add_proxysql_nodes.sh /usr/bin/add_proxysql_nodes.sh
RUN chmod a+x /usr/bin/add_proxysql_nodes.sh

COPY proxysql-admin /usr/bin/proxysql-admin
RUN chmod a+x /usr/bin/proxysql-admin

COPY peer-list /usr/bin/peer-list

RUN install -o 1001 -g 0 -m 775 -d /etc/proxysql /var/lib/proxysql
VOLUME /var/lib/proxysql

EXPOSE 3306 6032
ONBUILD RUN yum update -y

ENTRYPOINT ["/entrypoint.sh"]
USER 1001

CMD [ "/usr/bin/proxysql", "-f", "-c", "/etc/proxysql/proxysql.cnf" ]
