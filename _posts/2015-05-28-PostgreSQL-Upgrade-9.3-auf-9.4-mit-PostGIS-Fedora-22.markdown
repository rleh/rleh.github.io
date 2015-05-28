---
layout: post
title:  "PostgreSQL Upgrade 9.3 auf 9.4 mit PostGIS Fedora 22"
date:   2015-05-28 23:30:00
categories: linux postgresql ibis_server
---

Da hat man gerade mit zwei Installationen gute Erfahrungen beim *Fedora Linux* Upgrade von Version 21 auf 22  mittels [FedUp](https://fedoraproject.org/wiki/FedUp) gemacht, wird man in dritten Fall leider enttäuscht.
Normalerweise bedarf es keiner einzigen manuellen Intervention.

```
sudo yum update -y
sudo yum install fedup
sudo fedup --network 22
shutdown -r 0
```

Sofern *PostgreSQL* und *PostGIS* installiert ist, gestaltet sich der Vorgang jedoch nicht so leicht.
Da *Fedora 22* mit den Standardpaketquellen einen großen Versionssprung von *PostgreSQL 9.3* auf *9.4* durchführt müssen die vorhandenen Datenbanken migriert werden.

Da steht zwar nach den Reboot im Error-Log (```journalctl -xe```) wenn man sich wundert (```systemctl status postgresql.service```) warum der *PostgreSQL* Service nach dem Upgrade und Reboot nicht mehr läuft, wird aber leider nicht von selbst erledigt.

Zum Glück hilft das Tool ```postgresql-setup``` hier seit dem Update weiter.
```postgresql-setup``` kann einfach als normaler Admin-Nutzer mit ```sudo``` oder als root ausgeführt werden und sollte alle nötigen Schritte selbst erledigen.

```
postgresql-setup --upgrade
```

Leider beendete sich das Tool in meinem Fall sehr schnell selbst.

```
 * Upgrading database.
ERROR: failed
 * See /var/lib/pgsql/upgrade_postgresql.log for details.
```

Da steht dann:

```
Performing Consistency Checks
-----------------------------
Checking cluster versions                                   ok
Checking database user is a superuser                       ok
Checking database connection settings                       ok
Checking for prepared transactions                          ok
Checking for reg* system OID user data types                ok
Checking for contrib/isn with bigint-passing mismatch       ok
Checking for invalid "line" user columns                    ok
Creating dump of global objects                             ok
Creating dump of database schemas
  postgres
  routing1

*failure*

Consult the last few lines of "pg_upgrade_dump_7863130.log" for
the probable cause of the failure.
Failure, exiting
```

Und weiter:

```
command: "/usr/bin/pg_dump" --host "/home/pgsql" --port 5432 --username "postgres" --schema-only --quote-all-identifiers --binary-upgrade --format=custom  --file="pg_upgrade_dump_7863130.custom" "mydb" >> "pg_upgrade_dump_7863130.log" 2>&1
pg_dump: [Archivierer (DB)] Anfrage fehlgeschlagen: ERROR:  could not access file "$libdir/postgis-2.1": Datei oder Verzeichnis nicht gefunden
pg_dump: [Archivierer (DB)] Anfrage war: SELECT a.attnum, a.attname, a.atttypmod, a.attstattarget, a.attstorage, t.typstorage, a.attnotnull, a.atthasdef, a.attisdropped, a.attlen, a.attalign, a.attislocal, pg_catalog.format_type(t.oid,a.atttypmod) AS atttypname, array_to_string(a.attoptions, ', ') AS attoptions, CASE WHEN a.attcollation <> t.typcollation THEN a.attcollation ELSE 0 END AS attcollation, pg_catalog.array_to_string(ARRAY(SELECT pg_catalog.quote_ident(option_name) || ' ' || pg_catalog.quote_literal(option_value) FROM pg_catalog.pg_options_to_table(attfdwoptions) ORDER BY option_name), E',
    ') AS attfdwoptions FROM pg_catalog.pg_attribute a LEFT JOIN pg_catalog.pg_type t ON a.atttypid = t.oid WHERE a.attrelid = '7864479'::pg_catalog.oid AND a.attnum > 0::pg_catalog.int2 ORDER BY a.attrelid, a.attnum
```
Es fehlt anscheinend die *PostGIS* Erweiterung in den *PostgreSQL 9.3* Binaries die unter ```/usr/lib64/pgsql/postgresql-9.3/``` liegen.
Die installierte ```postgis-2.1.so``` mittels ```ln -s /usr/lib64/pgsql/postgis-2.1.so /usr/lib64/pgsql/postgresql-9.3/lib/``` dort zu verlinken führt leider zur Fehlermeldung ```ERROR:  incompatible library "/usr/lib64/pgsql/postgresql-9.3/lib/postgis-2.1.so": version mismatch  DETAIL:  Server is version 9.3, library is version 9.4.```.

Es muss folglich passende eine ```postgis-2.1.so``` für *PostgreSQL 9.3* her und nach ```/usr/lib64/pgsql/postgresql-9.3/lib/``` kopiert werden, das geht leider nicht über die Paketverwaltung (neuerdings ```dnf```).

```postgis-2.1.so``` kann beispielsweise von einer noch nicht aktualisierten *Fedora 21* Installation mit installiertem *PostGIS* kommen oder aus [postgis-2.1.3-5.fc21.x86_64.rpm](http://mirror2.hs-esslingen.de/fedora/linux/releases/21/Everything/x86_64/os/Packages/p/postgis-2.1.3-5.fc21.x86_64.rpm) extrahiert werden.

So gelingt ```postgresql-setup --upgrade```:

```                                             
 * Upgrading database.
 * Upgraded OK.
WARNING: The configuration files were replaced by default configuration.
WARNING: The previous configuration and data are stored in folder
WARNING: /var/lib/pgsql/data-old.
 * See /var/lib/pgsql/upgrade_postgresql.log for details.
```

Zum Abschluss sollten noch die Anweisungen aus ```upgrade_postgresql.log``` befolgt werden

```
Upgrade Complete
----------------
Optimizer statistics are not transferred by pg_upgrade so,
once you start the new server, consider running:
    analyze_new_cluster.sh

Running this script will delete the old cluster's data files:
    delete_old_cluster.sh
```

und der *PostgreSQL* Service neu gestartet werden.

Tipp: Die beiden Skripte liegen im Verzeichnis ```/var/lib/pgsql/```.
