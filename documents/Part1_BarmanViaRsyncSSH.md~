# The DBAs Corner presents: PgBarman Tutorial Series

# Part 1: 

#### How to Backup and Restore PostgreSQL Server with Barman via Rsync/SSH - Archive Command

### Objetives

This guide attempts to show step by step how to setup backups and restores via log shipping.  
The log shipping is the old Postgres method for create a Stanby Server.  
We use this method for _full filesystem backup_, make _incremental backups_ and save WAL archives for _diferential backups_.  
Then we will use those backups for recover full backups and PITR too, using the 'get-wal' feature.

## Backup

### Let's Go!

* If you don't have Barman yet installed, follow this _how to_ [Install Barman on CentOS 7](github.com/sarasa)
* For the examples we use Postgres 9.5 versions. 
* You can see the full Barman features by versions in [Barman Docs](http://docs.pgbarman.org/release/2.0/index.html), under the *Feature matrix* section.

### Step 1

On the Primary|Master Postgres Server, set the next entries in _postgresql.conf_  
On CentOS/RedHat Linux, the default _postgresql.conf_ is in the datadir,  
e.g `/var/lib/pgsql/[version]/data/postgresql.conf`

```
wal_level = hot_standby # minimal, archive, hot_standby, or logical 
archive_mode = on # enables archiving; off, on, or always 
archive_command = 'rsync -a %p barman@[Barman_Server_IP_Address]:/var/lib/barman/[Primary_Server_Name]/incoming/%f' 
```

*Note:* In this way, we could have one Slave via Streaming and the Barman server via Rsync log shipping.

### Step 2

Agregamos un nuevo archivo de configuracion en el servidor de backup, en `/etc/barman.d/[servername].conf`.
Podemos optar por copiar el template de ejemplo que alli se encuentra (sample) y modificamos las siguientes lineas:

```
[servername]
description =  "Primary PostgreSQL Database Server (via SSH)"
ssh_command = ssh postgres@[Server_Postgres_Ip_Address]
conninfo = host=[Server_Postgres_Ip_Address] user=postgres dbname=postgres
backup_method = rsync
reuse_backup = link
archiver = on
retention_policy_mode = auto
retention_policy = RECOVERY WINDOW OF 7 days
wal_retention_policy = main
recovery_options = 'get-wal'
```

### Step 3

Ponemos las llaves RSA del usuario _barman_ en el servidor Postgres-Server y probamos el acceso ssh, desde la sesion del usuario _barman_

```
barman@[Barman-Server]:~$ ssh postgres@[Postgres]
Last login: Fri Nov 11 14:28:54 2016 from [Postgres].int.cmd.com.ar
################################################################
#
# [Postgres]     Descripcion:  rol: DB Master
# Creado por:SomeAdmin Fecha:YYYY/MM/DD Ticket:OP-NNN
#
################################################################
postgres@[Postgres-Server]:~$
```

### Step 4

Hacemos lo mismo en viceversa con la llave RSA.
Ponemos la llave publica del usuario _postgres_ en el servidor Barman-Server y probamos el acceso ssh desde la sesion del usuario _postgres_

```
postgres@[Postgres-Server]:~$ ssh postgres@[Barman-Server]
Last login: Wed Nov  9 15:59:35 2016
################################################################
#
# [Barman-Server]     Descripcion: DBA rol: PgBarman
# Creado por:SomeAdmin Fecha:DD-MM-YYYY Ticket:
#
################################################################
barman@[Barman-Server]:~$
```

### Step 5

Damos permisos en pg_hba.conf del servidor Postgres, para que el usuario postgres se pueda conectar desde Barman-Server-

```
host all all [ip address Barman-Server]/32 trust
```

### Step 6

Desde barman chequeamos si el setup es correcto

```
barman@[Barman-Server]:~$ barman check [Postgres-Server] 
Server [Postgres-Server]: 
			PostgreSQL: OK 
			superuser: OK 
			wal_level: OK 
			directories: OK 
			retention policy settings: OK 
			backup maximum age: FAILED (interval provided: 1 day, latest backup age: No available backups) 
			compression settings: OK 
			failed backups: OK (there are 0 failed backups) 
			minimum redundancy requirements: OK (have 0 backups, expected at least 0) 
			ssh: OK (PostgreSQL server) 
			not in recovery: OK 
			archive_mode: OK 
			archive_command: OK 
			continuous archiving: OK 
			archiver errors: OK
```

*Nota:* `Backup maximum age` va a arrojarnos una alerta si es la primera vez que lanzamos un backup contra el server, esto es normal.

### Step 7

Finalmente, hacemos el backup de nuestro Postgres-Server

```
barman@[Barman-Server]:~$ barman backup [Postgres-Server] 
Starting backup using rsync-exclusive method for server [Postgres-Server] in /var/lib/barman/[Postgres-Server]/base/20161109T162030 
Backup start at xlog location: 0/A000060 (00000001000000000000000A, 00000060) 
This is the first backup for server [Postgres-Server] 
WAL segments preceding the current backup have been found: 
		000000010000000000000008 from server [Postgres-Server] has been removed 
Copying files. 
Copy done. 
This is the first backup for server [Postgres-Server] 
Asking PostgreSQL server to finalize the backup. 
Backup size: 57.1 MiB. Actual size on disk: 57.1 MiB (-0.00% deduplication ratio). 
Backup end at xlog location: 0/A000130 (00000001000000000000000A, 00000130) 
Backup completed 
Processing xlog segments from file archival for [Postgres-Server] 
		000000010000000000000009 
		00000001000000000000000A 
		00000001000000000000000A.00000060.backup
```

*Nota:* Si hicimos la instalacion de Barman via package (yum/apt, no debería de ser de otra forma, pero lo aclaro igual) se habra instalado un cron de barman que chequeará cada minuto si hay nuevos WAL archives para copiar a Barman-Server desde Postgres-Server, mediante los cuales tenemos nuestros incrementales

### Step 8

Somos felices :D

## Restore

### Let's Go!

* You need have installed _barman-cli.noarch_, if you don't have this follow this _how to_ [Install Barman on CentOS 7](github.com/sarasa)
* You need know the datadir location for the target Postgres-Server.
* You need know the backup name to recover.
* The same steps are valid for a normal recover, not PITR.

### Step 1

```
barman@[Barman-Server]:~$ barman recover [Postgres-Server] 20161114T173219 /var/lib/pgsql/[version]/data --remote-ssh-command "ssh postgres@[Postgres-Server]"
Starting remote restore for server [Postgres-Server] using backup 20161114T173219
Destination directory: /var/lib/pgsql/[version]/data
Doing PITR. Recovery target 
Copying the base backup.
Generating recovery.conf
Identify dangerous settings in destination directory.

IMPORTANT
These settings have been modified to prevent data losses

postgresql.conf line 206: archive_command = false

WARNING: 'get-wal' is in the specified 'recovery_options'.
Before you start up the PostgreSQL server, please review the recovery.conf file
inside the target directory. Make sure that 'restore_command' can be executed by the PostgreSQL user.

Your PostgreSQL server has been successfully prepared for recovery!
barman@[Barman-Server]:~$ 
```

*Note:* As you can see, there is a warning messagge, that indicates the 'get-wal' setting up. Also there is a entry "Doing PITR"

### Step 2

In the last step, you can see that Barman took the atributte to change _archive_command_ value to _false_
This will do an alert in postgres log, so you can change this to default _'cd .'_
Additionally you can check port and tunning settings.

### Step 3

Check that recovery.conf was generating and it values are like next e.g

```
# The 'barman-wal-restore' command is provided in the 'barman-cli' package
restore_command = 'barman-wal-restore -U barman v925uprod.int.cmd.com.ar v1105uprod %f %p'
```

### Step 4

Now, is time to start postgres.
Simply hit `service postgresql-9.5 start`

### Step 5

Check if there is nothing wrong in logs

The _pgstartup.log_ must seem like

```
< 2016-11-15 09:56:17.968 ART >LOG:  redirecting log output to logging collector process
< 2016-11-15 09:56:17.968 ART >HINT:  Future log output will appear in directory "pg_log".
```

And postgresql.log like

```
< 2016-11-15 09:56:17.971 ART >LOG:  database system was interrupted; last known up at 2016-11-14 17:32:19 ART
< 2016-11-15 09:56:18.126 ART >LOG:  starting archive recovery
< 2016-11-15 09:56:19.070 ART >LOG:  restored log file "000000010000000000000089" from archive
< 2016-11-15 09:56:19.314 ART >LOG:  redo starts at 0/89000028
< 2016-11-15 09:56:19.337 ART >LOG:  consistent recovery state reached at 0/89000168
< 2016-11-15 09:56:19.338 ART >LOG:  database system is ready to accept read only connections
< 2016-11-15 09:56:20.381 ART >LOG:  restored log file "00000001000000000000008A" from archive
< 2016-11-15 09:56:21.826 ART >LOG:  restored log file "00000001000000000000008B" from archive
< 2016-11-15 09:56:23.299 ART >LOG:  restored log file "00000001000000000000008C" from archive
< 2016-11-15 09:56:24.777 ART >LOG:  restored log file "00000001000000000000008D" from archive
< 2016-11-15 09:56:26.257 ART >LOG:  restored log file "00000001000000000000008E" from archive
< 2016-11-15 09:56:27.729 ART >LOG:  restored log file "00000001000000000000008F" from archive
ERROR: WAL file '000000010000000000000090' not found in server 'v1105uprod'
ERROR: Remote 'barman get-wal' command has failed!
< 2016-11-15 09:56:28.690 ART >LOG:  redo done at 0/8FFFFF50
< 2016-11-15 09:56:28.690 ART >LOG:  last completed transaction was at log time 2016-11-15 09:32:15.102826-03
< 2016-11-15 09:56:29.677 ART >LOG:  restored log file "00000001000000000000008F" from archive
ERROR: WAL file '00000002.history' not found in server 'v1105uprod'
ERROR: Remote 'barman get-wal' command has failed!
< 2016-11-15 09:56:30.333 ART >LOG:  selected new timeline ID: 2
```

### Step 6

Lastest, check access to postgres with psql client and be happy.

Done!