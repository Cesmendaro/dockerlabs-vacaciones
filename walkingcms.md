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

![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/5d0e16a5-4bb7-41e3-a357-818c34756168)

![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/bba021cc-d93e-4508-9295-c960a05356b6)

Es muy importante lo que nos ha encontrado el escaneo. No solo contamos con el panel de login predeterminado de WordPress activo, sino que también está activo el protocolo XML-RPC. Esto nos confirma que podremos realizar un ataque de fuerza bruta.

## Wpscan.

Bien, procedemos con el ataque de fuerza bruta utilizando el diccionario de contraseñas "rockyou.txt", especificando el nombre de usuario encontrado anteriormente.

```
sudo wpscan --url http://172.17.0.2/wordpress/ -U mario -P /usr/share/wordlists/rockyou.txt
```

![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/456d83db-2529-4606-85b4-3a4b4b6ceb7a)

![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/94c4ef3c-5dc7-4ba9-8925-28bab604fa22)

Ahora que disponemos del nombre de usuario y la contraseña, nos logueamos en dicho panel de autenticación.

![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/9a41173f-b897-4dfa-8ea9-a8a4ed013b64)

## Cargar código PHP y conseguir una reverse shell.

Para lograr nuestro objetivo, necesitamos un script en PHP que nos permita establecer una conexión de shell inversa. Para ello, primero accedemos al sitio web https://www.revshells.com/ y copiaremos el codigo que nos genera luego de haber puesto la dirección IP de nuestra máquina atacante y el puerto al que estaremos escuchando, para este ejemplo yo usare la opcion "PHP PentestMonkey".

![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/aa99d5f1-3d04-4f18-9c92-983a8db862c8)

Ahora, necesitamos iniciar la escucha en el puerto 443 utilizando Netcat.

```
sudo nc -nvlp 443
```

![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/0092e71e-4424-4b95-af1e-5f50ad89729d)

Después de iniciar la escucha en el puerto 443 con Netcat, nos dirigimos al panel de administración de WordPress. Una vez allí, cargamos el código en el editor del tema de la página. Es fundamental recordar que el script que hemos generado está en PHP, por lo que debemos elegir un archivo que sea compatible con este lenguaje para su correcta interpretación. En este caso, hemos seleccionado el archivo "functions.php", y podemos acceder a él a través de la ruta indicada por el título del editor, que sería "twentytwentytwo/functions.php".

![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/63d97991-daac-4976-a7c4-069e72d763ef)

Después de cargar el código en el archivo "functions.php" y guardarlo en el editor del tema de WordPress, accedemos a la URL correspondiente para que el script se ejecute y podamos recibir la shell inversa.

![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/9d9bf9c9-88f5-40e5-975f-961b86d0ae22)


![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/f926f014-81bc-40fb-a7c5-339f2885fbf5)


## Tratamiento de la TTY.

```
script /dev/null -c bash
```
```
stty raw -echo;fg
```
```
reset xterm
```
```
export TERM=xterm
```
```
export SHELL=bash
```

## Escalada de privilegios.

Como "sudo -l" nos da error, procederemos a buscar archivos en el sistema que puedan ejecutarse con privilegios de root, es decir con el SUID activo, y por supuesto que los errores los mande al dev/null.

```
find / -perm -4000  -type f 2>/dev/null
```

![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/889c22f9-d42b-497a-97d8-5eb1caca3540)

Como podemos ver, disponemos de una lista de binarios que pueden ser ejecutados con privilegios de root. La idea es probar cada uno de ellos para determinar cuál nos resulta útil. Sin embargo, el ya muy conocido binario "env" está disponible, por lo que procederemos directamente a https://gtfobins.github.io/

![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/eec695f8-f6c4-41ec-bf83-c80a51ad1dc4)

Copiamos el codigo que nos dice en la seccion de SUID y copiamos dicho codigo pero estableciendo la ruta absluta del binario.

![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/f082b4a9-731c-4617-94b5-6137aaaee3f0)

![image](https://github.com/Cesmendaro/dockerlabs-vacaciones/assets/153618246/aeabde0f-4792-452f-823a-eaf698de0de1)

Y como podemos ver, ya somos root.
Maquina 100% hackeada.




