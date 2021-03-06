---
title: Upgrading from Previous Versions
aliases:
    - /enterprise/v1.3/administration/upgrading/
menu:
  enterprise_influxdb_1_3:
    weight: 0
    parent: Administration
---

Content for `/administration/upgrading/`. The steps below assume the new version number is 1.3.1-c1.3.1. 

## Upgrading from version 1.2.5 to 1.3.1

Version 1.3.1 is a drop-in replacement for version 1.2.5 with no data migration required.

Version 1.3.1 introduces changes to the data node configuration file.
Please update that configuration file to avoid any unnecessary downtime.
The steps below outline the upgrade process and include a list of the required configuration changes.

### Step 1: Download the 1.3.1 packages

#### Meta node package download
**Ubuntu & Debian (64-bit)**
```
wget https://dl.influxdata.com/enterprise/releases/influxdb-meta_1.3.1-c1.3.1_amd64.deb
```

**RedHat & CentOS (64-bit)**
```
wget https://dl.influxdata.com/enterprise/releases/influxdb-meta-1.3.1_c1.3.1.x86_64.rpm
```

#### Data node package download
**Ubuntu & Debian (64-bit)**
```
wget https://dl.influxdata.com/enterprise/releases/influxdb-data_1.3.1-c1.3.1_amd64.deb
```

**RedHat & CentOS (64-bit)**
```
wget https://dl.influxdata.com/enterprise/releases/influxdb-data-1.3.1_c1.3.1.x86_64.rpm
```

### Step 2: Install the 1.3.1 packages

#### Meta node package Install

**Ubuntu & Debian (64-bit)**
```
sudo dpkg -i influxdb-meta_1.3.1-c1.3.1_amd64.deb
```

**RedHat & CentOS (64-bit)**
```
sudo yum localinstall influxdb-meta-1.3.1_c1.3.1.x86_64.rpm
```

#### Data node package install

When you run the install command, your terminal asks if you'd like to keep your current configuration file or overwrite your current configuration file with the file for version 1.3.1.
Please keep your current configuration file by entering `N` or `O`;
we update the configuration file with the necessary changes for version 1.3.1 in step 3.

**Ubuntu & Debian (64-bit)**
```
sudo dpkg -i influxdb-data_1.3.1-c1.3.1_amd64.deb
```

**RedHat & CentOS (64-bit)**
```
sudo yum localinstall influxdb-data-1.3.1_c1.3.1.x86_64.rpm
```

### Step 3: Update the data node configuration file

Add:

* [index-version = "inmem"](/enterprise_influxdb/v1.3/administration/configuration/#index-version-inmem) to the `[data]` section
* [wal-fsync-delay = "0s"](/enterprise_influxdb/v1.3/administration/configuration/#wal-fsync-delay-0s) to the `[data]` section
* [max-concurrent-compactions = 0](/enterprise_influxdb/v1.3/administration/configuration/#max-concurrent-compactions-0) to the `[data]` section
* [pool-max-idle-streams = 100](/enterprise_influxdb/v1.3/administration/configuration/#pool-max-idle-streams-100) to the `[cluster]` section
* [pool-max-idle-time = "1m0s"](/enterprise_influxdb/v1.3/administration/configuration/#pool-max-idle-time-1m0s) to the `[cluster]` section
* the [[anti-entropy]](/enterprise_influxdb/v1.3/administration/configuration/#anti-entropy) section:
```
[anti-entropy]
  enabled = true
  check-interval = "30s"
  max-fetch = 10
```

Remove:

* `max-remote-write-connections` from the `[cluster]` section
* the [[admin]](/enterprise_influxdb/v1.3/administration/configuration/#admin) section

Update:

* [cache-max-memory-size](/enterprise_influxdb/v1.3/administration/configuration/#cache-max-memory-size-1073741824) to `1073741824` in the `[data]` section

The new configuration options are set to their default settings.
Follow the links for more information about those options.

### Step 4: Restart the processes

#### Meta node restart
**sysvinit systems**
```
service influxdb-meta restart
```
**systemd systems**
```
sudo systemctl restart influxdb-meta
```

#### Data node Restart
**sysvinit systems**
```
service influxdb restart
```
**systemd systems**
```
sudo systemctl restart influxdb
```

### Step 5: Confirm the upgrade

Check your nodes' version numbers using the `influxd-ctl show` command.
The [`influxd-ctl` tool](/enterprise_influxdb/v1.3/features/cluster-commands/) is available on all meta nodes.

```
~# influxd-ctl show

Data Nodes
==========
ID	TCP Address		Version
4	rk-upgrading-01:8088	1.3.1_c1.3.1   # 1.3.1_c1.3.1 = 👍
5	rk-upgrading-02:8088	1.3.1_c1.3.1
6	rk-upgrading-03:8088	1.3.1_c1.3.1

Meta Nodes
==========
TCP Address		Version
rk-upgrading-01:8091	1.3.1_c1.3.1
rk-upgrading-02:8091	1.3.1_c1.3.1
rk-upgrading-03:8091	1.3.1_c1.3.1
```

If you have any issues upgrading your cluster, please do not hesitate to contact support at the email 
provided to you when you received your InfluxEnterprise license.
