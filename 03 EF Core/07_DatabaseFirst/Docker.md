# Docker Images nutzen

Lade Docker für dein Betriebssystem von [docs.docker.com](https://docs.docker.com/get-docker/).

Die Installation von Docker Desktop und das Laden eines Oracle 21c Containers ist als Video
verfügbar: https://youtu.be/ekmGqHBVNTM

Nach der erfolgreichen Installation wird der Container für Oracle 21 XE mittels der folgenden
Befehle in der Windows Konsole geladen und ausgeführt. Der Container hat rund 3.5 GB.

Der *docker run* Befehl verwendet ein Verzeichnis (*C:/Temp/oracle-home*), um das Homeverzeichnis
zu mappen. Bei anderen Betriebssystemen (macOS, Linux) muss dieser Pfad angepasst werden, da es
dort keine Laufwerke gibt.

```text
docker pull gvenzl/oracle-xe:21-full
docker run -d -p 1521:1521 -e ORACLE_PASSWORD=oracle -v C:/Temp/oracle-home:/home --name oracle21c gvenzl/oracle-xe:21-full
```

Die Umgebungsvariable *ORACLE_PASSWORD* setzt das Systempasswort. Da es keine Produktionsdatenbank
ist, verwenden wir zur Vereinfachung *oracle*.

## Starten und Stoppen des Containers

Durch *docker run* wird unser Container bereits gestartet. Aber wie verhält es sich nach einem
Neustart von Windows? Docker Desktop startet automatisch mit
Microsoft Windows, der Container wird allerdings nicht automatisch gestartet.
Daher die zwei folgenden Befehle in der Konsole zum Starten bzw. manuellen Stoppen (wenn notwendig)
des Containers wichtig:

```text
docker start oracle21c
docker stop oracle21c
```

Natürlich kann mit Docker Desktop der Container ebenfalls gestartet und beendet werden.

> **Hinweis:** Gerade nach dem ersten Start des Containers vergeht etwas Zeit, bis die Datenbank 
> hochgefahren ist. Kontrolliere die Ausgaben in Docker Desktop, indem du auf den Containernamen
> klickst. Es muss die Meldung *DATABASE IS READY TO USE!* im Log Fenster erscheinen.

## Ausführen von Programmen im Container

Mit *docker exec -it oracle21c COMMAND* können Befehle direkt im Container ausgeführt werden.
Die Option *-i* bedeutet eine interaktive Ausführung. *-t* öffnet ein Terminal, sodass nicht CR+LF
von Windows gesendet wird (Linux verwendet nur CR).

### SQL*Plus 

SQL*Plus ist ein Kommandozeilentool, welches direkt SQL Befehle absetzen kann. Wollen wir als
System User direkt Befehle in der pluggable database absetzen, können wir
mittels *docker exec* das Dienstprogramm *sqlplus* starten. Das Passwort ist *oracle* und wurde
im *docker run* Befehl weiter oben als Umgebungsvariable *ORACLE_PASSWORD* gesetzt.

```text
docker exec -it oracle21c sqlplus system/oracle@//localhost/XEPDB1
```

Wollen wir *systemweite Änderungen* machen, gibt es noch den User *SYS*. Hier können Konfigurationen,
die das ganze System betreffen, gelesen und gesetzt werden. Beachte, dass *oracle* das Passwort
des Users sys ist.

```text
docker exec -it oracle21c sqlplus sys/oracle AS SYSDBA
```

Mit dem Befehl *quit* kann der SQL*Plus Prompt verlassen werden.

### Shell (bash) und Datenaustausch mit dem Host

Wir können auch eine Shell öffnen und Befehle in Linux absetzen:

```text
docker exec -it oracle21c /bin/bash
```

Mit *exit* kann die Shell verlassen und zu Windows zurückgekehrt werden. Du kannst auch in
Docker Desktop auf den Button *CLI*, der beim Container angeboten wird, klicken.

Beim Anlegen des Containers mit *docker run* haben wir mit dem Parameter
*-v C:/Temp/oracle-home:/home* einen Ordner angegeben, der auch im Container sichtbar ist.
Nun können wir z. B. in Windows in *C:/Temp/oracle-home* eine Textdatei anlegen. In der bash
ist sie im Homeverzeichnis sichtbar:

```text
bash-4.4$ cd /home/
bash-4.4$ ls
test.txt

bash-4.4$ cat test.txt
Das
ist
ein
Test!
```

## Andere Datenbanksysteme als Docker

### MariaDB (MySQL fork)

Über Docker kann auch MariaDb (ein Fork von MySQL) geladen werden. Die nachfolgende Anweisung
in der Konsole lädt die letzte Version von MariaDb in den Container mit dem Namen *mariadb*
und prüft die Version. Der root Benutzer hat kein Passwort.

```bash
docker run -d -p 3306:3306 --name mariadb -e MARIADB_ALLOW_EMPTY_ROOT_PASSWORD=true  mariadb:latest
docker exec -it mariadb mysql

MariaDB [(none)]> SELECT VERSION();
+-------------------------------------+
| VERSION()                           |
+-------------------------------------+
| 10.6.5-MariaDB-1:10.6.5+maria~focal |
+-------------------------------------+

MariaDB [(none)]> quit
```

### Microsoft SQL Server

Natürlich steht auch SQL Server zur Verfügung. Die nachfolgende Anweisung legt einen Container
mit dem Namen *sqlserver2019* an. Das Passwort wird in *SA_PASSWORD* gesetzt und ist in diesem
Beispiel *SqlServer2019*. Der Benutzer ist *sa*. Das Passwort muss folgender Richtlinie genügen:

> This password needs to include at least 8 characters of at least three of these four categories:
> uppercase letters, lowercase letters, numbers and non-alphanumeric symbols.

```text
docker run -d -p 1433:1433  --name sqlserver2019 -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=SqlServer2019" mcr.microsoft.com/mssql/server:2019-latest      
```