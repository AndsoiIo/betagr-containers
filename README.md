##How to use

--
Add Dockerfile to your application main folder (_'server/'_).
If you use base Dockerfile, just put it into your application main folder, ``requirements.txt`` should be in the same folder.
In docker-compose file use standard syntax with ``command`` directive. 

If you need to apply some DB migrations - use liquibase Dockerfile.
Changelog file is required.

Structure:
```
server/
 ├─ requirements.txt
 ├─ Dockerfile
 ├─ app.py
 ├─ *changelog.xml
 └─ *migrations/
     ├─ 001_table_create.xml
     └─ ...

* - if liquibase Dockerfile use
```

##### Docker-compose file for liquibase Dockerfile:

Use ``args`` in ``build`` directive to configurate liquibase docker.
To be able to execute multiple commands in ``command`` directive, use multiline syntax (since the entrypoint of container is set to the '/bin/sh', you should indicate directly ``-c`` flag for mulriline commands.)

```docker-compose
version: '3'
services:
    server:
        build:
            context: path/to/server
            dockerfile: ./Dockerfile
            args:
                POSTGRESQL_DRIVER_VERSION: postgresql-42.2.9
                POSTGRES_HOST: database
                POSTGRES_PORT: ${POSTGRES_PORT}
                POSTGRES_DB: ${POSTGRES_DB}
                POSTGRES_USER: ${POSTGRES_USER}
                POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
                CHANGELOG_FILE: changelog.xml
        depends_on:
            - ...
        ports:
            - ...
        env_file:
            - .env
        command:
            - -c
            - |
                liquibase update
                python app.py

```

##### Required arguments for building a container

arg | means
----|-------
POSTGRESQL_DRIVER_VERSION | postgresql-42.2.9 (driver for liquibase, download from official site)
POSTGRES_HOST| database service name 
POSTGRES_PORT| 5432
POSTGRES_DB| database name
POSTGRES_USER| database user
POSTGRES_PASSWORD| password
CHANGELOG_FILE| changelog.xml (see example below)


##### changeog.xml file example

```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">

  <includeAll path="migrations"/> 
</databaseChangeLog>
```