# Instalar o Oracle no Ubuntu 

1. Download Oracle Database Express Edition.

2. Instructions before install Oracle

* Copy the downloaded file and paste it in home directory.
* Unzip using the command:
  ```unzip oracle-xe-11.2.0-1.0.x86_64.rpm.zip```
* Install required packages using the command:
  ```sudo apt-get install alien libaio1 unixodbc```
* Enter into the Disk1 folder using command:
  ```cd Disk1/```
* Convert RPM package format to DEB package format (that is used by Ubuntu) using the command:
  ```sudo alien --scripts -d oracle-xe-11.2.0-1.0.x86_64.rpm```
* Create the required chkconfig script using the command:
  ```sudo vi /sbin/chkconfig```
  Now copy and paste the following into the file and save:
  ```#!/bin/bash
     # Oracle 11gR2 XE installer chkconfig hack for Ubuntu
     file=/etc/init.d/oracle-xe
     if [[ ! `tail -n1 $file | grep INIT` ]]; then
         echo >> $file
         echo '### BEGIN INIT INFO' >> $file
         echo '# Provides: OracleXE' >> $file
         echo '# Required-Start: $remote_fs $syslog' >> $file
         echo '# Required-Stop: $remote_fs $syslog' >> $file
         echo '# Default-Start: 2 3 4 5' >> $file
         echo '# Default-Stop: 0 1 6' >> $file
         echo '# Short-Description: Oracle 11g Express Edition' >> $file
         echo '### END INIT INFO' >> $file
     fi
     update-rc.d oracle-xe defaults 80 01
     ```
* Change the permission of the chkconfig file using the command:
  ```sudo chmod 755 /sbin/chkconfig  ```
* Set kernel parameters. Oracle 11gR2 XE requires additional kernel parameters which you need to set using the command:
  ```sudo vi /etc/sysctl.d/60-oracle.conf```
* Copy the following into the file and save:
  ```
  # Oracle 11g XE kernel parameters 
  fs.file-max=6815744  
  net.ipv4.ip_local_port_range=9000 65000  
  kernel.sem=250 32000 100 128 
  kernel.shmmax=536870912 
  ```
* Verify the change using the command:
  ```sudo cat /etc/sysctl.d/60-oracle.conf ```
* You should see what you entered earlier. Now load the kernel parameters:
  ```sudo service procps start```
* Verify the new parameters are loaded using:
  ```sudo sysctl -q fs.file-max```
* You should see the file-max value that you entered earlier.
```Set up /dev/shm mount point for Oracle. Create the following file using the command:```
```sudo vi /etc/rc2.d/S01shm_load```
* Copy the following into the file and save.
```
  #!/bin/sh
  case "$1" in
  start)
    mkdir /var/lock/subsys 2>/dev/null
    touch /var/lock/subsys/listener
    rm /dev/shm 2>/dev/null
    mkdir /dev/shm 2>/dev/null
  *)
    echo error
    exit 1
    ;;
  esac 
  ```
* Change the permissions of the file using the command:
  ```sudo chmod 755 /etc/rc2.d/S01shm_load```
* Now execute the following commands:

  ```sudo ln -s /usr/bin/awk /bin/awk ```
```sudo mkdir /var/lock/subsys ```
```sudo touch /var/lock/subsys/listener```
* Now, Reboot Your System
* Step 3: Install Oracle

* Install the oracle DBMS using the command:
```sudo dpkg --install oracle-xe_11.2.0-2_amd64.deb```
* Configure Oracle using the command:
```sudo /etc/init.d/oracle-xe configure ```
* Setup environment variables by editting your .bashrc file:
```vi ~/.bashrc```
* Add the following lines to the end of the file:
```
export ORACLE_HOME=/u01/app/oracle/product/11.2.0/xe
export ORACLE_SID=XE
export NLS_LANG=`$ORACLE_HOME/bin/nls_lang.sh`
export ORACLE_BASE=/u01/app/oracle
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH
export PATH=$ORACLE_HOME/bin:$PATH
```
* Load the changes by executing your profile:
```. ~/.profile```
* Start the Oracle 11gR2 XE:
```sudo service oracle-xe start```
* Add user YOURUSERNAME to group dba using the command:
```sudo usermod -a -G dba YOURUSERNAME```
* Step 4: Using the Oracle XE Command Shell

* Start the Oracle XE 11gR2 server using the command:
```sudo service oracle-xe start```
* Start command line shell as the system admin using the command:
```sqlplus sys as sysdba```
* Enter the password that you gave while configuring Oracle earlier. You will now be placed in a SQL environment that only understands SQL commands.
* Create a regular user account in Oracle using the SQL command:
```
create user USERNAME identified by PASSWORD;
```
* Replace USERNAME and PASSWORD with the username and password of your choice. Please remember this username and password. If you had error executing the above with a message about resetlogs, then execute the following SQL command and try again:
```alter database open resetlogs;```
* Grant privileges to the user account using the SQL command:

```grant connect, resource to USERNAME;```
* Replace USERNAME and PASSWORD with the username and password of your choice. Please remember this username and password.
* Exit the sys admin shell using the SQL command:
```exit;```

* Start the commandline shell as a regular user using the command:
```sqlplus```
Now, you can run sql commands...
