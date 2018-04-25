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

<h3>Po instalacji</h3>
<ol>
  <li>PATH=$HOME/pgxl/bin:$PATH</li>
  <li>export PATH</li>
  <li>LD_LIBRARY_PATH=$HOME/pgxl/lib:$LD_LIBRARY_PATH</li>
  <li>export LD_LIBRARY_PATH</li>
  <li>service sshd start</li>
    <li>Passwordless SSH
    <ol>
      <li>ssh-keygen -t rsa</li>
      <li>Press enter for each line</li>
      <li>cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys</li>
      <li>chmod og-wx ~/.ssh/authorized_keys</li>
    </ol>
  </li>
  <li>dataDirRoot=$HOME/DATA/pgxl/nodes</li>
  <li>export dataDirRoot</li>
</ol>

<h3>Deploying Postgres-XL cluster</h3>
<ol>
  <li>pgxc_ctl<li>
  <li>PGXC prepare config empty</li>
</ol>

Dodanie GTM mastera do Clustra
1. pgxc_ctl
2. PGXC$ add gtm master gtm localhost 6667 $dataDirRoot/gtm

Dodanie Coordinatora (x2) do Clustra
pgxc_ctl
add coordinator master coord1 localhost 30001 30011 $dataDirRoot/coord_master.1 none none
add coordinator master coord1 localhost 30002 30012 $dataDirRoot/coord_master.2 none none

Dodanie DataNodów (x2) do Clustra
pgxc_ctl
add datanode master dn1 localhost 40001 40011 $dataDirRoot/dn_master.1 none none none
add datanode master dn2 localhost 40002 40012 $dataDirRoot/dn_master.2 none none none

<h3>Stworzenie bazy testowej</h3>
psql -p 30001 postgres
postgres=# CREATE DATABASE testdb;
CREATE DATABASE
postgres=# \q

<h3>bashrc</h3>
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi

PATH=$PATH:$HOME/pgxl/bin
export PATH

PATH=$HOME/pgxl/bin:$PATH
export PATH

LD_LIBRARY_PATH=$HOME/pgxl/lib:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH

dataDirRoot=$HOME/DATA/pgxl/nodes
export dataDirRoot

# User specific aliases and functions

