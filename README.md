<h1>postgres-XL-cluster</h1>

<h3>Instalacja niezbędnych zależności do zbudowania Postgres-XL</h3>
<ol>
  <li>yum install wget gcc readline-devel zlib-devel flex bison</li>
  <li>yum install pacemaker pcs resource-agents fence-agents-all</li>
  <li>systemctl start pcsd.service</li>
  <li>systemctl enable pcsd.service</li>
</ol>
  
<h3>Budowanie i instalacja Postgres-XL</h3>
<ol>
  <li>useradd postgres</li>
  <li>passwd postgres</li>
<li>su postgres</li>
<li>mkdir -p $HOME/pgxl/src</li>
<li>cd $HOME/pgxl/src</li>
<li>wget https://www.postgres-xl.org/downloads/postgres-xl-9.5r1.6.tar.bz2</li>
<li>tar jxf postgres-xl-9.5r1.6.tar.bz2</li>
<li>cd postgres-xl-9.5r1.6</li>
<li>./configure --prefix=$HOME/pgxl</li>
<li>make install-world</li>
</ol>

Po instalacji:
1. PATH=$HOME/pgxl/bin:$PATH
2. export PATH
3. LD_LIBRARY_PATH=$HOME/pgxl/lib:$LD_LIBRARY_PATH
4. export LD_LIBRARY_PATH

Deploying Postgres-XL:
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

dataDirRoot=$HOME/DATA/pgxl/nodes
export dataDirRoot

The next step is to add the GTM master to the setup:
1. $ pgxc_ctl
2. PGXC$ add gtm master gtm localhost 6667 $dataDirRoot/gtm


service sshd start

Passwordless SSH:
1. ssh-keygen -t rsa
Press enter for each line 
2. cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
3. chmod og-wx ~/.ssh/authorized_keys 
