
# Création base

## Vérification espace

```bash
df -h
```

## Création base
### Analyse préalable
Type d'allocation mémoire, destination files, types de caractères, taille des redologs (journaux), environnement de conteneur.

```bash
dbca -silent -createDatabase \
> -databaseType MULTIPURPOSE \
> -memoryMgmtType auto_sga \
> -totalMemory 4096 \
> -templateName New_Database.dbt \
> -datafileDestination "/oracle/oradata" \
> -redoLogFileSize 50 \
> -emConfiguration NONE \
> -gdbname BASE -sid BASE -responseFile NO_VALUE \
> -characterSet WE8MSWIN1252 \
> -nationalCharacterSet AL16UTF16 \
> -sysPassword XXXX \
> -systemPassword XXXX \
> -createAsContainerDatabase false \
> -useOMF true \
> -storageType FS \
> -ignorePreReqs

[WARNING] [DBT-06208] The 'SYS' password entered does not conform to the Oracle recommended standards.
   CAUSE:
a. Oracle recommends that the password entered should be at least 8 characters in length, contain at least 1 uppercase character, 1 lower case character and 1 digit [0-9].
b.The password entered is a keyword that Oracle does not recommend to be used as password
   ACTION: Specify a strong password. If required refer Oracle documentation for guidelines.
[WARNING] [DBT-06208] The 'SYSTEM' password entered does not conform to the Oracle recommended standards.
   CAUSE:
a. Oracle recommends that the password entered should be at least 8 characters in length, contain at least 1 uppercase character, 1 lower case character and 1 digit [0-9].
b.The password entered is a keyword that Oracle does not recommend to be used as password
   ACTION: Specify a strong password. If required refer Oracle documentation for guidelines.
Prepare for db operation
5% complete
Creating and starting Oracle instance
6% complete
9% complete
Creating database files
10% complete
14% complete
Creating data dictionary views
15% complete
18% complete
19% complete
22% complete
23% complete
25% complete
27% complete
Oracle JVM
34% complete
41% complete
48% complete
50% complete
Oracle Text
51% complete
54% complete
55% complete
Oracle Multimedia
68% complete
Oracle OLAP
69% complete
70% complete
71% complete
72% complete
73% complete
Oracle Spatial
74% complete
82% complete
Completing Database Creation
85% complete
86% complete
Executing Post Configuration Actions
100% complete
Database creation complete. For details check the logfiles at:
 /oracle/cfgtoollogs/dbca/BASE.
Database Information:
Global Database Name:BASE
System Identifier(SID):BASE
Look at the log file "/oracle/cfgtoollogs/dbca/BASE/BASE.log" for further details.
```

## Modification password SYSTEM et SYS

```sql
SQL> alter user SYSTEM identified by "XXX";

User altered.

SQL> alter user SYS identified by "XXX";

User altered.
```

## Modification des paramètres d'initialisation
### Analyse préalable
* OPEN_CURSORS défaut à 50, correspond au nb max de curseurs qu'une session peut avoir en meme temps (nb = dépend de chaque appli) prévient l'ouverture d'un nb excessif
* PROCESSES correspond au nb max d'os user qui utilisent des process, attention tient compte des processus arrière plan, locks, file d'attente, exécution parallèle
    > en découlent les param SESSIONS et TRANSACTIONS, à contrôler
* SESSIONS résulte de PROCESSES, calcul : (1.5 * PROCESSES) + 22
* TRANSACTIONS résulte de SESSIONS, calcul : (1.1 * SESSIONS) mais semble être limité par la taille du tablespace d'annulation UNDO_MANAGEMENT
* DEFERRED_SEGMENT_CREATION à true = segments de tabl et objets dépendants ne sont créés qu'à l'insertion de la première ligne (optimise temps d'install)
* DISPATCHERS va de paire avec SHARED_SERVERS, si ce dernier est différent de 0 alors DISPATCHERS prend la valeur (PROTOCOL=tcp) et crée un dispatcher TCP/IP
* SHARED_SERVERS est à 0 par défaut, à confirmer mais si DISPATCHERS différent de 0, la valeur de SHARED_SERVER passe à 1
    > induit une architecture de srv partagés
* RECYCLEBIN défaut à on, si off alors les drop table ne vont plus dans la corbeille (cas de suppression massive sans trace)
* SGA_TARGET indique la taille des sous composants SGA, si différent de 0 alors prend ce paramètre, si param AUTO_SGA alors auto géré (au moment du createdatabase)
* PGA_AGGREGATE_TARGET défaut à 10MB ou 20% de la SGA (si pas auto), spécifie la cible de mémoire disponible pour tous les process serveur attaché à l'instance
* DB_RECOVERY_FILE_DEST_SIZE défaut à 0, définit la limite totale utilisée par database dans la fast recovery area
* CONTROL_FILE_RECORD_KEEP_TIME défaut à 7 (jours)

```sql
SQL> alter system set audit_trail='NONE' scope=spfile;
alter system set db_create_file_dest='/oracle/oradata' scope=spfile;
alter system set db_domain='' scope=spfile;
alter system set db_recovery_file_dest='/oracle/archives' scope=spfile;
alter system set db_recovery_file_dest_size=20G scope=both ;
alter system set diagnostic_dest='/oracle' scope=spfile;
alter system set log_archive_format='log_%r_%t_%s.arc' scope=spfile;
alter system set open_cursors=600 scope=spfile;
alter system set processes=800 scope=spfile;
alter system set sessions=1224 scope=spfile;
alter system set remote_login_passwordfile='EXCLUSIVE' scope=spfile;
alter system set service_names='srv_BASE' scope=spfile;
System altered.

SQL>
System altered.

SQL>
System altered.

SQL>
System altered.

SQL>
System altered.

SQL>
System altered.

SQL>
System altered.

SQL>
System altered.

SQL>
System altered.

SQL>
System altered.

SQL>
System altered.

SQL>
System altered.
```

## Redémarrage base pour prise en compte 
```sql
SQL> shutdown immediate;
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL>
SQL>
SQL> startup
ORACLE instance started.

Total System Global Area 3221222456 bytes
Fixed Size                  8901688 bytes
Variable Size             654311424 bytes
Database Buffers         2550136832 bytes
Redo Buffers                7872512 bytes
Database mounted.
Database opened.
```

## Déplacement des fichiers de données
### Nommage des fichiers pour simplification dans le temps
```sql
SQL> alter database move datafile '/oracle/oradata/BASE/datafile/o1_mf_sysaux_mhzbhc3l_.dbf' to '/oracle/oradata/BASE/datafile/sysaux01.dbf';

Database altered.

SQL> alter database move datafile '/oracle/oradata/BASE/datafile/o1_mf_undotbs1_mhzbhfq5_.dbf' to '/oracle/oradata/BASE/datafile/undotbs01.dbf';

Database altered.

SQL> alter database move datafile '/oracle/oradata/BASE/datafile/o1_mf_users_mhzbhp9z_.dbf' to '/oracle/oradata/BASE/datafile/users01.dbf';

Database altered.

SQL> alter database move datafile '/oracle/oradata/BASE/datafile/o1_mf_system_mhzbgtcx_.dbf' to '/oracle/oradata/BASE/datafile/system01.dbf';

Database altered.
```

## Gestion fichiers de controle
* Nommage simplifié
* 2 fichiers présents sur 2 filesystem différents
```sql
show parameter control
			/oracle/oradata/BASE/controlfile/o1_mf_mhzbgmkm_.ctl
```
```bash
mkdir -p /oracle/archives/BASE/controlfile
```
```sql
SQL> alter system set control_files='/oracle/oradata/BASE/controlfile/ctrl01.ctl', '/oracle/archives/BASE/controlfile/ctrl02.ctl' scope=spfile ;

System altered.

SQL> shutdown immediate;
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL>
SQL> exit
```
```bash
cp /oracle/oradata/BASE/controlfile/o1_mf_mhzbgmkm_.ctl /oracle/oradata/BASE/controlfile/ctrl01.ctl
cp /oracle/oradata/BASE/controlfile/ctrl01.ctl /oracle/archives/BASE/controlfile/ctrl02.ctl
```
```sql
SQL> startup
ORACLE instance started.

Total System Global Area 3221222456 bytes
Fixed Size                  8901688 bytes
Variable Size             654311424 bytes
Database Buffers         2550136832 bytes
Redo Buffers                7872512 bytes
Database mounted.
Database opened.

SQL> show parameter control_files;

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
control_files                        string      /oracle/oradata/BASE/controlfi
                                                 le/ctrl01.ctl, /oracle/archive
                                                 s/BASE/controlfile/ctrl02.ctl

```


## Gestion fichiers journaux
* Nommage simplifié
* Répartition sur 4 groupes / 2 membres par groupe sous-répartis sur 2 filesystem différents
* Taille à ajuster pour une une optimisation à 2/4 bascules par heure 
```bash
mkdir -p /oracle/archives/BASE/onlinelog
```

```sql
SQL> alter database drop logfile group 1 ;

Database altered.

SQL> alter database add logfile group 1 ('/oracle/oradata/BASE/onlinelog/redo01a.rdo', '/oracle/archives/BASE/onlinelog/redo01b.rdo') size 50M;

Database altered.

SQL> alter database drop logfile group 2 ;

Database altered.

SQL> alter database add logfile group 2 ('/oracle/oradata/BASE/onlinelog/redo02a.rdo', '/oracle/archives/BASE/onlinelog/redo02b.rdo') size 50M;

Database altered.

SQL> alter system switch logfile;

System altered.

SQL> alter system checkpoint;

System altered.

SQL> alter database drop logfile group 3;

Database altered.

SQL> alter database add logfile group 3 ('/oracle/oradata/BASE/onlinelog/redo03a.rdo', '/oracle/archives/BASE/onlinelog/redo03b.rdo') size 50M;

Database altered.

SQL> alter database add logfile group 4 ('/oracle/oradata/BASE/onlinelog/redo04a.rdo', '/oracle/archives/BASE/onlinelog/redo04b.rdo') size 50M;

Database altered.
```
```sql
SQL> shutdown immediate;
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> startup;
ORACLE instance started.

Total System Global Area 3221222456 bytes
Fixed Size                  8901688 bytes
Variable Size             654311424 bytes
Database Buffers         2550136832 bytes
Redo Buffers                7872512 bytes
Database mounted.
Database opened.
```

## Contrôle du tnsnames et listener
* Mode de connexion à la base pour les clients (alias)
* Paramètres d'écoute sur le serveur hôte
```bash
BASE =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = HOST.domain)(PORT = PORT))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = BASE)
    )
  )

LISTENER_BASE =
  (ADDRESS = (PROTOCOL = TCP)(HOST = HOST.domain)(PORT = PORT))
```

## Modification oratab
### Comportement au redémarrage du serveur hôte
```bash
vi /etc/oratab
    BASE:/oracle/product/version/dbhome_1:Y
```
