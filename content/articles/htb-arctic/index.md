---
title: "HTB Arctic"
description: "Windows Exploit Suggester"
date: "2025-11-18"
banner:
  src: "../../images/windows.jpg"
  alt: "Microsoft Windows Icon"
  caption: 'Photo by <a href="https://unsplash.com/@stadsa">Tadas Sar</a> onUnsplash'
categories:
  - "AD"
  - "Tool"
keywords:
  - "AD"
  - "Tool"
---

# Que aprendi de la maquina Arctic


## Info gatherig con Netcat

> Esto lo hice para matar el tiempo mientras Nmap mostraba resultados. No es algo que se vea en la maquina en si.

Puedo usar netcat para interactuar con un puerto abierto, pero solo obtendre información si el servicio que está detrás sabe qué responder.

1. Si el puerto está abierto y nc se conecta, 
   1. veras que se queda esperando.
   2. Si no responde banners, solo responde ante un protocolo válido. Si el servicio no tiene mensaje de bienvenida, no verás nada.
   3. Esto es típico en servicios binarios, propietarios, o protocolos no verbosos. No entienden texto, esperan un paquete binario (con cabeceras, flags, tamaños, checksums), comandos muy específicos, o no envían ningún mensaje inicial.
   4. Puedes mandar entradas genéricas para ver si el servicio lanza un error o banner.

