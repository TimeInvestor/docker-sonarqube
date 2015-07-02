## SonarQube(Docker) Operational Manual ##

### Introduction
This is a setup guide for SonarQube in production.

 - Ubuntu
 - [MySQL in Docker](https://registry.hub.docker.com/_/mysql/)
 - [SonarQube in Docker](https://registry.hub.docker.com/_/sonarqube/)
 
### Prerequisite
**Operating System**: Ubuntu
**Software/Tools**:

 - Docker ([official installation guide](https://docs.docker.com/installation/))
	 - install by running command: `wget -qO- https://get.docker.com/ | sh`
 - MySQL client
	 - install by running command: `sudo apt-get install mysql-client`
 
### Initial Setup
 1. On hosting server
	 1. Start MySQL: <br/>`docker run --name sonardb -p 3306:3306 -e MYSQL_ROOT_PASSWORD=sonar -e MYSQL_DATABASE=sonar -e MYSQL_USER=sonar -e MYSQL_PASSWORD=sonar -d mysql:5.6`
	 2. Check status:<br/>`docker logs -f sonardb`<br/>Wait until you see "MySQL init process done. Ready for start up." in the log.
	 3. Start SonarQube linked to MySQL:<br/>`docker run --name sonarqube --link sonardb:mysql -p 9000:9000 -p 3306:3306 -e SONARQUBE_JDBC_USERNAME=sonar -e SONARQUBE_JDBC_PASSWORD=sonar -e SONARQUBE_JDBC_URL="jdbc:mysql://mysql:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance" -v opt/sonarqube/extensions/plugins:/opt/sonarqube/extensions/plugins -d sonarqube:5.1.1`
	 4.  Check status:<br/>`docker logs -f sonarqube`<br/>Wait until you see "Process[web] is up" in the log.
 2. Go to http://hostname:9000/ to access SonarQube dashboard (default Admin user name: *admin*, password: *admin*)

### Host server restarted -OR-  Need to restart SonarQube
1. `docker restart sonardb`
2. `docker logs -f sonardb`<br/>Wait unitl see "mysqld: ready for connections." in the log.
2. `docker restart sonarqube`
3. `docker logs -f sonarqube`<br/>Wait unitl see "app[o.s.p.m.Monitor] Process[web] is up" in the log.

### Backup and Restore
#### Back Up DB Data
On host server, first run below command

    mysqldump --host=localhost --port=3306 --protocol TCP -u sonar -p sonar --single-transaction --databases sonar > sonardb_backup-<date>.sql
    (password: sonar)

Then, ***store the backup sql file to a secure place***.

#### Back Up SonarQube Plugins
Back up folder/directory:

 - /opt/sonarqube/extensions/plugins

#### Restore
#### Restore DB Data
On host server, navigate to folder/directory where DB backup file is stored.
Then, run below command

    mysql --host=localhost --port=3306 --protocol TCP -u sonar -p sonar < backup_file.sql
    (password: sonar)
   
#### Restore others
Put back backup folders/directories.