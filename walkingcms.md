---
Nombre de la máquina: WalkingCms
Sistema Operativo: Linux
Dificultad: Facil 🟢
Enlace de descarga: https://dockerlabs.es
---

## Ejecutamos la maquina

Para comenzar, es necesario ubicarnos en la ruta donde hemos descargado y descomprimido la máquina. Una vez allí, procedemos a ejecutarla utilizando el siguiente comando.

```
sudo bash auto_deploy.sh walkingcms.tar
```

![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/0f57257c-8dd6-4a89-acac-94073fb9fe82)


## Nmap.

Después de haber lanzado el entorno vulnerable, procedemos a realizar un escaneo utilizando nmap.

```
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 172.17.0.2
```

![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/4590fff3-e38a-455e-9f36-5398b72d261f)


El escaneo revela la existencia del puerto 80 abierto, donde se está ejecutando un servidor Apache versión 2.4.57. Por consiguiente, procedemos a inspeccionar la aplicación web para determinar su contenido y funcionalidades.

![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/5edfbc09-1ed0-41ce-8472-f04bb8b9887a)


Hemos verificado que se trata de la plantilla predeterminada de Apache. Tras revisar su código fuente sin encontrar elementos destacables, es hora de proceder con el fuzzing.

## Fuzzing.

Lanzamos el escaneo de directorios utilizando el diccionario "directory-list-lowercase-2.3-medium.txt". Además, hemos agregado extensiones de archivos que nos interesan buscar adicionalmente utilizando el comando "-x".

```
sudo gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -u "http://172.17.0.2/" -x .php,.sh,.py,.txt
```

![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/7e2b2e6a-0827-40ba-9883-a95d8b825185)


Como podemos observar, se nos informa sobre la existencia del directorio "wordpress" con codigo de estado 301, Por consiguiente, procederemos a revisar su contenido para determinar su naturaleza y relevancia.

Al parecer, se trata de un blog sin mucha información relevante. Únicamente vemos un texto "Hola Mundo" que es un enlace a una entrada. Al acceder a dicha entrada, podemos observar que tenemos un nombre de usuario, "mario".

![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/8a8f2a8c-6682-4b05-b248-af630282581b)

Bien, contamos únicamente con un nombre de usuario, pero ningún panel de login para intentar un ataque de fuerza bruta. Por lo tanto, volvemos a realizar fuzzing, pero esta vez lo hacemos a partir del directorio "wordpress".

Es muy importante lo que nos ha encontrado el escaneo. No solo contamos con el panel de login predeterminado de WordPress activo, sino que también está activo el protocolo XML-RPC. Esto nos confirma que podremos realizar un ataque de fuerza bruta.

![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/5d0e16a5-4bb7-41e3-a357-818c34756168)


![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/bba021cc-d93e-4508-9295-c960a05356b6)


## Wpscan.

Bien, procedemos con el ataque de fuerza bruta utilizando el diccionario de contraseñas "rockyou.txt", especificando el nombre de usuario encontrado anteriormente.

```
sudo wpscan --url http://172.17.0.2/wordpress/ -U mario -P /usr/share/wordlists/rockyou.txt
```

![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/456d83db-2529-4606-85b4-3a4b4b6ceb7a)

![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/94c4ef3c-5dc7-4ba9-8925-28bab604fa22)

Ahora que disponemos del nombre de usuario y la contraseña, procederemos a establecer una conexión SSH.


![image](https://github.com/Cesmendaro/Dockerlabs.es/assets/153618246/902181d4-5fdc-4246-8ee0-c0646d07d175)


## Escalada de privilegios.

Una vez dentro y bajo el usuario "mario", el siguiente paso es intentar escalar privilegios. Para ello, ejecutaremos el comando `sudo -l`para ver qué comandos puede ejecutar el usuario "mario" como root.


![image](https://github.com/Cesmendaro/Dockerlabs.es/assets/153618246/e5325cdb-915f-4cef-a2cd-efea05febd3f)


Perfecto, vamos a buscar en el sitio web https://gtfobins.github.io/ si encontramos algún comando que nos permita escalar privilegios utilizando el editor vim.

```
sudo vim -c ':!/bin/sh'
```

![image](https://github.com/Cesmendaro/Dockerlabs.es/assets/153618246/47631fb5-0fdc-4697-8d68-263c28b8f9e2)

Entendido, parece que podemos obtener una shell, pero para ejecutarlo correctamente necesitamos especificar la ruta absoluta del comando.

```
sudo /usr/bin/vim -c ':!/bin/sh'
```

![image](https://github.com/Cesmendaro/Dockerlabs.es/assets/153618246/79e850fd-ef92-4edc-8394-99115a1a9839)

## Tratamiento de la TTY

Una vez completado este paso, hemos obtenido privilegios de root. Ahora solo nos queda realizar el tratamiento de la TTY para tener una interacción más completa y funcional como root.

```
script /dev/null -c bash
```

![image](https://github.com/Cesmendaro/Dockerlabs.es/assets/153618246/538c7475-4c47-4486-a1ac-0390c6fb346d)

Maquina 100% hackeada.

