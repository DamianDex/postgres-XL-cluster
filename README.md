postgres-XL-cluster

Instalacja niezbędnych zależności:
1. yum install wget gcc readline-devel zlib-devel flex bison
2. yum install pacemaker pcs resource-agents fence-agents-all
3. systemctl start pcsd.service
4. systemctl enable pcsd.service

Budowanie i instalacja Postgres-XL:
1. useradd postgres
2. passwd postgres

  Ja dodałem u siebie: postgres/postgres

3. su postgres
4. mkdir -p $HOME/pgxl/src
5. cd $HOME/pgxl/src
6. wget https://www.postgres-xl.org/downloads/postgres-xl-9.5r1.6.tar.bz2
7. tar jxf postgres-xl-9.5r1.6.tar.bz2
8. cd postgres-xl-9.5r1.6
9. ./configure --prefix=$HOME/pgxl
10. make install-world

Po instalacji:
1. PATH=$HOME/pgxl/bin:$PATH
2. export PATH

3. LD_LIBRARY_PATH=$HOME/pgxl/lib:$LD_LIBRARY_PATH
4. export LD_LIBRARY_PATH

Deploying POstgres-XL:
1. pgxc_ctl

Logs:
/usr/bin/bash
Installing pgxc_ctl_bash script as /home/postgres/pgxc_ctl/pgxc_ctl_bash.
ERROR: File "/home/postgres/pgxc_ctl/pgxc_ctl.conf" not found or not a regular file. No such file or directory
Installing pgxc_ctl_bash script as /home/postgres/pgxc_ctl/pgxc_ctl_bash.
Reading configuration using /home/postgres/pgxc_ctl/pgxc_ctl_bash --home /home/postgres/pgxc_ctl --configuration /home/postgres/pgxc_ctl/pgxc_ctl.conf
Finished reading configuration.
   ******** PGXC_CTL START ***************

Current directory: /home/postgres/pgxc_ctl
PGXC prepare config empty
PGXC q

