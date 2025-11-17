---
title: "HTB Active"
description: "Active Directory"
date: "2025-11-16"
banner:
  src: "../../images/windows.jpg"
  alt: "Microsoft Windows Icon"
  caption: 'Photo by <a href="https://unsplash.com/@stadsa">Tadas Sar</a> onUnsplash'
categories:
  - "AD"
  - "Theory"
  - "Basic"
keywords:
  - "AD"
  - "Theory"
  - "Basic"
---

## Que aprendi de la maquina Active

* Service Principal Name or SPNs

Kerberos tiene dos puntos que podemos atacar
* AS_REP roasting, cuentas con flag "DONT_REQUIERE_PREAUTH" activada
* Kerberosting, SPNs para decifrar hash NTLM

> En AD las cuentas vinculadas a servicios (SPNs) son objetivos de alto interes porque los servicios suelen correr con privilegios.

### Cuentas ejecutando servicios

En Windows, un servicio utiliza cuentas para correr servicios porque es como correr Apache o Nginx en Linux con una cuenta www-data.

Algunas cuentas para correr servicios pueden ser:

- Con LocalSystem
- Con NetworkService
- Con una cuenta dedicada
  - Cualquier cuenta de usuario, incluso Administrator. 

> krbtgt lleva SPN, pero no es kerberoasteable. Es una cuenta interna del dominio usada por Kerberos. No es “un servicio normal”, es parte del propio protocolo Kerberos. 

### Relacionando Objetos, Cuentas y Servicios

El campo sAMAccountName muestra la cuenta real, "dn" muestra el objeto que guarda la cuenta, detras de la barra el servicio y su puerto asociado (active/CIFS:445), y lo que relaciona el sAMAccountName con el servicePrincipalName (SPN) es el objeto ("dn: CN=Administrator,CN=Users,DC=active,DC=htb").