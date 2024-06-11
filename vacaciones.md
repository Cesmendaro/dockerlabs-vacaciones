---
Nombre de la máquina: Trust
Sistema Operativo: Linux
Dificultad: Muy Facil 🟢
Enlace de descarga: https://dockerlabs.es
---

## Ejecutamos la maquina

Para comenzar, es necesario ubicarnos en la ruta donde hemos descargado y descomprimido la máquina. Una vez allí, procedemos a ejecutarla utilizando el siguiente comando.

```
sudo bash auto_deploy.sh trust.tar
```

![image](https://github.com/Cesmendaro/Dockerlabs.es/assets/153618246/c16ba732-c73f-45e3-99f6-20418d084296)

## Nmap.

Después de haber lanzado el entorno vulnerable, procedemos a realizar un escaneo utilizando nmap.

```
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 172.17.0.2
```

![image](https://github.com/Cesmendaro/Dockerlabs.es/assets/153618246/f175244f-74df-438e-8f73-75fbe3678345)

El escaneo revela la existencia de dos puertos abiertos: el puerto 22, que corresponde al protocolo SSH, y el puerto 80, donde se está ejecutando un servidor Apache versión 2.4.57. Por consiguiente, procedemos a inspeccionar la aplicación web para determinar su contenido y funcionalidades.

![image](https://github.com/Cesmendaro/Dockerlabs.es/assets/153618246/7cbb281a-b793-42f5-aa2b-609e3639673b)


Hemos verificado que se trata de la plantilla predeterminada de Apache. Tras revisar su código fuente sin encontrar elementos destacables, es hora de proceder con el fuzzing.

## Fuzzing.

Lanzamos el escaneo de directorios utilizando el diccionario "directory-list-lowercase-2.3-medium.txt". Además, hemos agregado extensiones de archivos que nos interesan buscar adicionalmente utilizando el comando "-x".

```
sudo gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -u "http://172.17.0.2/" -x .php,.sh,.py,.txt
```

![image](https://github.com/Cesmendaro/Dockerlabs.es/assets/153618246/826a5a63-2e29-4b91-ab22-74cf415a8937)

Como podemos observar, se nos informa sobre la existencia del archivo "secret.php", Por consiguiente, procederemos a revisar su contenido para determinar su naturaleza y relevancia.

![image](https://github.com/Cesmendaro/Dockerlabs.es/assets/153618246/4c23b30c-e3c8-4b0e-9cb8-dc471e1e82a8)

Al parecer, el archivo contiene un mensaje simple, pero menciona la existencia de un usuario llamado "Mario". En consecuencia, vamos a realizar un ataque de fuerza bruta con Hydra al protocolo SSH. Dado que el escaneo de Nmap también ha revelado el puerto 22 abierto.

## Hydra.

Vamos a proceder con el ataque de fuerza bruta utilizando el diccionario de contraseñas "rockyou.txt", especificando el nombre de usuario encontrado anteriormente.

```
sudo hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

![image](https://github.com/Cesmendaro/Dockerlabs.es/assets/153618246/fe2496d4-32a5-43f1-b22b-48dae8526848)


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

