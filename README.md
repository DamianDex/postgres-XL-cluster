# Postgres-XL
## High-Availability (HA)

## Wstęp

Postgres-XL HA dostarcza zasoby dla takich agentów jak GTM (master), koordynator i data nody. Zasoby te mogą być zarządzane z poziomu Pacemakera.
Natywnie Postgres-XL nie wspiera automatycznego failoveru. W tym celu należy użyć Pacemakera, aby zautomatyzować ten proces.

This documentation explains how to install and configure Postgres-XL and Postgres-XL HA on an example cluster. The cluster has 4 nodes; 1 node (*t1*) for a GTM, 1 node (*t2*) for a coordinator, and 2 nodes (*t3*, *t4*) for datanodes.

W przedstawionym projekcie zainstalowano i skonfigurowano proste klastry Postgres-XL i Postgres-XL HA.
Klaster posiada 4 węzły:
- master t1,
- koordynator t2,
- data nody t3, t4.

## Pobranie zależności

Zależności dla Postgres-XL:

```sh
$ yum install wget gcc readline-devel zlib-devel flex bison
```

Zależności dla Clustra:

```sh
$ yum install pacemaker pcs resource-agents fence-agents-all
```

Na każdym z nodów należy uruchomić nastepujacy proces:

```sh
$ systemctl start pcsd.service
$ systemctl enable pcsd.service
```

Dla każdej VMki należy dodać do /etc/hosts:

```
192.168.56.11 t1
192.168.56.12 t2
192.168.56.13 t3
192.168.56.14 t4
```

### Budowanie i instalacja Postgres-XL

Użyty zostanie użytkownik 'farad':

```sh
$ useradd farad
$ passwd farad
```

Po stworzeniu uzytkownika 'farad' zaloguj się na maszyne t1 i zainstaluj Postgres-XL:

```sh
$ mkdir -p $HOME/pgxl/src
$ cd $HOME/pgxl/src
$ wget http://files.postgres-xl.org/postgres-xl-9.5r1.5.tar.gz
$ tar -xzf postgres-xl-9.5r1.5.tar.gz
$ cd postgres-xl-9.5r1.5
$ ./configure --prefix=$HOME/pgxl
$ make install-world
```

### Po instalacji Postgres-XL

Dodaj następujące linie do /home/farad/.bashrc. Za każdym razem, gdy zalogujesz się jako 'farad' będzie to dodawane automatycznie.

```sh
PATH=$HOME/pgxl/bin:$PATH
export PATH

LD_LIBRARY_PATH=$HOME/pgxl/lib:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH
```

### Deploying Postgres-XL na pozostałe nody t2, t3, t4

Aby zainstalować Postgres-XL na pozostałych maszynach użyjemy pgxc_ctl.
Wcześniej należy skonfigurować passwordless ssh

```sh
On host1, generate the authentication key file,
ssh-keygen -t rsa (Just press ENTER for all input values)
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

On host1, upload file authorized_keys to host2, host3 and host3, as following,
scp ~/.ssh/authorized_keys postgres@192.168.187.131:~/.ssh/
scp ~/.ssh/authorized_keys postgres@192.168.187.132:~/.ssh/
scp ~/.ssh/authorized_keys postgres@192.168.187.133:~/.ssh/

On every host, run following commands,
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

On host1, try to connect host2, host3 and host4, make sure no password is needed,
ssh postgres@192.168.187.131
ssh postgres@192.168.187.132
ssh postgres@192.168.187.133
```

```sh
[postgres@t1 ~]$ pgxc_ctl
/bin/bash
Installing pgxc_ctl_bash script as /home/postgres/pgxc_ctl/pgxc_ctl_bash.
ERROR: File "/home/postgres/pgxc_ctl/pgxc_ctl.conf" not found or not a regular file. No such file or directory
Installing pgxc_ctl_bash script as /home/postgres/pgxc_ctl/pgxc_ctl_bash.
Reading configuration using /home/postgres/pgxc_ctl/pgxc_ctl_bash --home /home/postgres/pgxc_ctl --configuration /home/postgres/pgxc_ctl/pgxc_ctl.conf
Finished reading configuration.
   ******** PGXC_CTL START ***************

Current directory: /home/postgres/pgxc_ctl
PGXC prepare config empty
PGXC q
```

Zmień ścieżkę w pliku `$HOME/pgxc_ctl/pgxc_ctl.conf`

```
pgxcInstallDir=$HOME/pgxl
```

Uruchom deployment:

```sh
[postgres@t1 ~]$ pgxc_ctl
/bin/bash
Installing pgxc_ctl_bash script as /home/postgres/pgxc_ctl/pgxc_ctl_bash.
Installing pgxc_ctl_bash script as /home/postgres/pgxc_ctl/pgxc_ctl_bash.
Reading configuration using /home/postgres/pgxc_ctl/pgxc_ctl_bash --home /home/postgres/pgxc_ctl --configuration /home/postgres/pgxc_ctl/pgxc_ctl.conf
Finished reading configuration.
   ******** PGXC_CTL START ***************

Current directory: /home/postgres/pgxc_ctl
PGXC deploy t2 t3 t4
Deploying Postgres-XL components.
Prepare tarball to deploy ...
Deploying to the server t2.
Deploying to the server t3.
Deploying to the server t4.
Deployment done.
```

Wykonaj `Po instalacji` na wszystkich VMkach.


## Creating a Postgres-XL cluster

Dodaj następujące linie do /home/farad/.bashrc. Za każdym razem, gdy zalogujesz się jako 'farad' będzie to dodawane automatycznie.

```sh
dataDirRoot=$HOME/DATA/pgxl/nodes
export dataDirRoot
```

### Adding a GTM

```sh
[postgres@t1 ~]$ pgxc_ctl
PGXC add gtm master gtm t1 6666 $dataDirRoot/gtm
```

### Adding a Coordinator

Stwórz `$HOME/DATA/pgxl/extra_pg_hba.conf`

```
host all all t1 trust
host all all t2 trust
host all all t3 trust
host all all t4 trust
```

Dodanie koordynatora:

```sh
[postgres@t1 ~]$ pgxc_ctl
PGXC add coordinator master coord t2 5432 6668 $dataDirRoot/coord none /home/postgres/DATA/pgxl/extra_pg_hba.conf
```

### Dodanie datanodów

Masters:

```sh
[postgres@t1 ~]$ pgxc_ctl
PGXC add datanode master dn1 t3 5433 6669 $dataDirRoot/dn1 none none /home/postgres/DATA/pgxl/extra_pg_hba.conf
PGXC add datanode master dn2 t4 5434 6670 $dataDirRoot/dn2 none none /home/postgres/DATA/pgxl/extra_pg_hba.conf
```

Slaves:

```sh
[postgres@t1 ~]$ pgxc_ctl
PGXC add datanode slave dn1 t4 5433 6669 $dataDirRoot/dn1 none $dataDirRoot/dn1a
PGXC add datanode slave dn2 t3 5434 6670 $dataDirRoot/dn2 none $dataDirRoot/dn2a
```

Sprawdź status:

```sh
[postgres@t1 ~]$ pgxc_ctl
PGXC monitor all
Running: gtm master
Running: coordinator master coord
Running: datanode master dn1
Running: datanode slave dn1
Running: datanode master dn2
Running: datanode slave dn2
```

## Postgres-XL HA

### Pacemaker Cluster

Stworzenia hasła dla użytownika 'hacluster'

```sh
$ echo PASSWORD | passwd --stdin hacluster
```

```sh
$ pcs cluster auth t1 t2 t3 t4 -u hacluster -p PASSWORD
```

```sh
$ pcs cluster setup --name postgres-xl t1 t2 t3 t4
```

### Startowanie Pacemaker Cluster

```sh
$ pcs cluster start --all
t1: Starting Cluster...
t2: Starting Cluster...
t4: Starting Cluster...
t3: Starting Cluster...
$ pcs status
Cluster name: postgres-xl
Stack: corosync
Current DC: t2 (version 1.1.15-11.el7_3.4-e174ec8) - partition with quorum
Last updated: Mon May 22 22:41:21 2017		Last change: Mon May 22 22:40:26 2017 by hacluster via crmd on t2

4 nodes and 0 resources configured

Online: [ t1 t2 t3 t4 ]

No resources


Daemon Status:
  corosync: active/disabled
  pacemaker: active/disabled
  pcsd: active/enabled
```

### Ustawianie Cluster Options

```sh
$ pcs property set stonith-enabled="false"
$ pcs property set symmetric-cluster="false"

$ pcs resource defaults migration-threshold=1
```

### Tworzenie GTM 

```sh
$ pcs cluster cib gtm.xml

$ pcs -f gtm.xml resource create gtm ocf:bitnine:postgres-xl-gtm \
        bindir=/home/postgres/pgxl/bin \
        datadir=/home/postgres/DATA/pgxl/nodes/gtm

$ pcs -f gtm.xml constraint location gtm prefers t1
$ pcs -f gtm.xml constraint location gtm avoids t2 t3 t4

$ pcs cluster cib-push gtm.xml
```

### Tworzenie koordynatora

```sh
$ pcs cluster cib coord.xml

$ pcs -f coord.xml resource create coord ocf:bitnine:postgres-xl-coord \
        bindir=/home/postgres/pgxl/bin \
        datadir=/home/postgres/DATA/pgxl/nodes/coord

$ pcs -f coord.xml constraint location coord prefers t2
$ pcs -f coord.xml constraint location coord avoids t1 t3 t4

$ pcs cluster cib-push coord.xml
```

### Tworzenie Data nodów

```sh
$ pcs cluster cib dn1.xml

$ pcs -f dn1.xml resource create dn1 ocf:bitnine:postgres-xl-data \
        bindir=/home/postgres/pgxl/bin \
        datadir=/home/postgres/DATA/pgxl/nodes/dn1 \
        port=5433 \
        nodename=dn1 \
        op start timeout=60s \
        op stop timeout=60s \
        op promote timeout=30s \
        op demote timeout=120s \
        op monitor interval=15s timeout=10s role="Master" \
        op monitor interval=16s timeout=10s role="Slave"

$ pcs -f dn1.xml resource master dn1-ha dn1 \
        master-max=1 master-node-max=1 \
        clone-max=2 clone-node-max=1

$ pcs -f dn1.xml constraint location dn1-ha prefers t3=1000 t4=0
$ pcs -f dn1.xml constraint location dn1-ha avoids t1 t2

$ pcs cluster cib-push dn1.xml
```

```sh
$ pcs cluster cib dn2.xml

$ pcs -f dn2.xml resource create dn2 ocf:bitnine:postgres-xl-data \
        bindir=/home/postgres/pgxl/bin \
        datadir=/home/postgres/DATA/pgxl/nodes/dn2 \
        port=5434 \
        nodename=dn2 \
        op start timeout=60s \
        op stop timeout=60s \
        op promote timeout=30s \
        op demote timeout=120s \
        op monitor interval=15s timeout=10s role="Master" \
        op monitor interval=16s timeout=10s role="Slave"

$ pcs -f dn2.xml resource master dn2-ha dn2 \
        master-max=1 master-node-max=1 \
        clone-max=2 clone-node-max=1

$ pcs -f dn2.xml constraint location dn2-ha prefers t3=0 t4=1000
$ pcs -f dn2.xml constraint location dn2-ha avoids t1 t2

$ pcs cluster cib-push dn2.xml
```

## Testy Failover

```sh
[postgres@t1 ~]$ psql -h t2
postgres=# CREATE TABLE disttab(col1 int, col2 int, col3 text) DISTRIBUTE BY HASH(col1);
CREATE TABLE
postgres=# INSERT INTO disttab SELECT generate_series(1,100), generate_series(101, 200), 'foo';
INSERT 0 100
postgres=# SELECT xc_node_id, count(*) FROM disttab GROUP BY xc_node_id;
 xc_node_id | count
------------+-------
 -560021589 |    42
  352366662 |    58
(2 rows)
```

```sh
[postgres@t1 ~]$ pgxc_ctl
PGXC stop -m immediate datanode master dn1
```

```sh
[postgres@t1 ~]$ psql -h t2
postgres=# SELECT xc_node_id, count(*) FROM disttab GROUP BY xc_node_id;
ERROR:  Failed to get pooled connections
HINT:  This may happen because one or more nodes are currently unreachable, either because of node or network failure.
 Its also possible that the target node may have hit the connection limit or the pooler is configured with low connections.
 Please check if all nodes are running fine and also review max_connections and max_pool_size configuration parameters
```

```sh
[postgres@t1 ~]$ psql -h t2
postgres=# SELECT xc_node_id, count(*) FROM disttab GROUP BY xc_node_id;
 xc_node_id | count
------------+-------
 -560021589 |    42
  352366662 |    58
(2 rows)
```
