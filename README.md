# Unattended install of ossec-hids in Amazon Linux 2023

HIDS = **H**ost-based **I**ntrusion **D**etection **S**ystem


```bash
docker run -it amazonlinux:latest
```

```bash
yum update -y
yum install -y wget gzip hostname which vim
```

<!-- https://www.ossec.net/docs/docs/manual/installation/installation-package.html#rpm-installation

```bash
export NON_INT=1
wget -q -O - https://updates.atomicorp.com/installers/atomic | sh
unset $NON_INT
``` -->

## Install ossec-hids Dependencies
https://www.ossec.net/docs/docs/manual/installation/installation-requirements.html#redhat-centos-fedora-amazon-linux

```bash
yum install -y \
  zlib-devel \
  pcre2-devel \
  make \
  gcc \
  sqlite-devel \
  openssl-devel \
  libevent-devel \
  systemd-devel
```

### Download, Compile and Install GeoIP

```bash
export GEOIP_VERSION="1.6.12"

yum install -y 
    make \
    automake \
    gcc \
    gcc-c++ \
    libtool \
    zlib-devel \
    wget

wget https://github.com/maxmind/geoip-api-c/releases/download/v$GEOIP_VERSION/GeoIP-$GEOIP_VERSION.tar.gz
tar -zxvf GeoIP-$GEOIP_VERSION.tar.gz
cd GeoIP-$GEOIP_VERSION
./configure
make
make install
cd -
rm -rf GeoIP-$GEOIP_VERSION.tar.gz
```

### Compile ossec-hids from source

```bash
export OSSEC_VERSION="3.7.0"
wget -O ossec-hids.tar.gz "https://github.com/ossec/ossec-hids/archive/refs/tags/${OSSEC_VERSION}.tar.gz"
tar -xvzf ossec-hids.tar.gz
cd ossec-hids-$OSSEC_VERSION
```

Create a config file in `ossec-hids-<version>/etc/preloaded-vars.conf` for unattended installation

server, agent, local, hybrid
https://www.ossec.net/docs/docs/manual/installation/install-source-unattended.html


```bash
cat << EOF > etc/preloaded-vars.conf
USER_LANGUAGE="en"
USER_NO_STOP="y"
USER_INSTALL_TYPE="server"
USER_DELETE_DIR="y"
USER_DIR="/var/ossec"
USER_ENABLE_ACTIVE_RESPONSE="y"
USER_ENABLE_SYSCHECK="y"
USER_ENABLE_ROOTCHECK="y"
USER_ENABLE_OPENSCAP="y"
USER_ENABLE_SYSLOG="y"
USER_UPDATE="y"
USER_UPDATE_RULES="y"
USER_CLEANINSTALL="y"
USER_ENABLE_EMAIL="n"
USER_ENABLE_FIREWALL_RESPONSE="n"
USER_WHITE_LIST="127.0.0.1"
EOF
```

```bash
# https://www.ossec.net/docs/docs/manual/installation/install-source-unattended.html
USER_ENABLE_EMAIL="y"
USER_EMAIL_ADDRESS="<>"
USER_EMAIL_SMTP="<>"

```

```bash
./install.sh
```

```bash
mkdir -p /var/ossec/etc/
cat << EOF > /var/ossec/etc/ossec.conf
<ossec_config>
  <global>
    <logall>yes</logall>
  </global>

  <syscheck>

    <!-- THIS FILE IS A PLACEHOLDER TO ALLOW OSSEC INSTALL IN BAKING AND IS OVERWRITTEN ON THE INSTANCES -->

    <!-- This time is changed to a random time in userdata -->
    <scan_time>9:00</scan_time>

    <!-- Directories to check  (perform all possible verifications) -->
    <directories realtime="no" check_all="yes" report_changes="yes">/etc</directories>
    <directories realtime="no" check_all="yes" report_changes="yes">/var/ossec/etc</directories>

    <!-- Files/directories to ignore from alerting -->
    <ignore>/etc/mtab</ignore>
    <ignore>/etc/mnttab</ignore>
    <ignore>/etc/hosts.deny</ignore>
    <ignore>/etc/mail/statistics</ignore>
    <ignore>/etc/random-seed</ignore>
    <ignore>/etc/adjtime</ignore>
    <ignore>/etc/httpd/logs</ignore>
    <ignore>/etc/utmpx</ignore>
    <ignore>/etc/wtmpx</ignore>
    <ignore>/etc/cups/certs</ignore>
    <ignore>/etc/dumpdates</ignore>
    <ignore>/etc/svc/volatile</ignore>
    <ignore type="sregex">.log$</ignore>
    <ignore type="sregex">.tmp$</ignore>
  </syscheck>

  <rootcheck>
    <rootkit_files>/var/ossec/etc/shared/rootkit_files.txt</rootkit_files>
    <rootkit_trojans>/var/ossec/etc/shared/rootkit_trojans.txt</rootkit_trojans>
    <system_audit>/var/ossec/etc/shared/system_audit_rcl.txt</system_audit>
    <system_audit>/var/ossec/etc/shared/cis_rhel_linux_rcl.txt</system_audit>
  </rootcheck>

  <active-response>
    <disabled>yes</disabled>
  </active-response>

  <alerts>
    <log_alert_level>1</log_alert_level>
  </alerts>

</ossec_config>

<ossec_config> <!-- rules global entry -->
  <rules>
    <include>rules_config.xml</include>
    <include>squid_rules.xml</include>
    <include>vmware_rules.xml</include>
    <include>sshd_rules.xml</include>
    <include>ossec_rules.xml</include>
    <include>clam_av_rules.xml</include>
    <include>nginx_rules.xml</include>
    <include>web_rules.xml</include>
  </rules>
</ossec_config>
EOF
```