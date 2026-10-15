+++
date = '2026-08-13T03:25:39+02:00'
draft = true
title = 'Bandit OverTheWire: Write-up'
summary = 'Write-up de cómo resolver el wargame Bandit de OverTheWire'
tags = ["OverTheWire", "Linux", "tutorial"]
+++

Bandit es un *wargame* de OverTheWire orientado al aprendizaje de los fundamentos de Unix y Linux mediante una serie de retos progresivos.


![OverTheWire Logo](/images/otw/OverTheWire_Logo.png)

Bandit es el punto de inicio de muchos en OverTheWire. Está pensado para aprender (o recordar algunas cosillas) sobre el manejo de Linux desde la terminal, así que los primeros niveles son bastante sencillos y poco a poco van introduciendo retos más interesantes y un poco más complicados.

La idea es ir resolviendo cada nivel para conseguir la contraseña del siguiente. Durante el camino aparecen un montón de comandos y herramientas que está bien conocer, así que estos apuntes son mi forma de dejar por escrito lo que voy aprendiendo y las soluciones que he ido encontrando.



>**Aviso sobre las normas**: este write-up contiene spoilers y explica cómo resolver los niveles de Bandit. Por respeto a las normas de OverTheWire, las credenciales que obtenga durante la resolución no las mostraré en ningún momento.


# Mecánica básica de Bandit

Antes de comenzar con los niveles, voy a recordar la mecánica básica de estos niveles, es importante entender la mecánica general del wargame. La dinámica de Bandit se basa en avanzar de un nivel al siguiente utilizando las credenciales obtenidas en la resolución de cada reto.

Para acceder a un nivel, nos conectamos al servidor mediante SSH siguiendo esta estructura:


```tmux
ssh -p 2220 bandit[número_del_nivel]@bandit.labs.overthewire.org 
```

Una vez dentro, nuestro objetivo será resolver el reto que nos toque y obtener las credenciales necesarias para acceder al siguiente nivel. Cuando las hayamos obtenido, cerramos la sesión con:


```tmux
exit
```


y establecemos una nueva conexión utilizando el usuario del siguiente nivel y las credenciales que acabamos de obtener.

Con esto ya aclarado podemos comenzar con la resolución de niveles:

# Nivel 0

### Objetivo:

El objetivo de este nivel es que nos podamos conectar a la máquina por SSH, con el usuario **bandit0** y contraseña **bandit0**, a través del puerto **2220**, la IP a la que nos tenemos que conectar es **bandit.labs.overthewire.org**

--- 

Con lo cual vamos a abrir la terminal y vamos a conectarnos utilizando la sintaxis que ya he explicado antes:

```tmux
ssh -p 2220 bandit0@bandit.labs.overthewire.org
```

Si hemos puesto bien el comando, podremos notarlo rápidamente al ver que ya nos sale el panel de OverTheWire:


```tmux
ssh -p 2220 bandit0@bandit.labs.overthewire.org
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       

                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

backend: gibson-1
bandit0@bandit.labs.overthewire.org's password: 
```


Ahora lo que nos piden es la contraseña de este nivel, que también era bandit0


Una vez hecho esto, si hemos puesto todo bien, ya estaremos dentro de la máquina a la que nos intentamos conectar, la cual primero nos soltará un texto bastante grande:

```tmux


      ,----..            ,----,          .---.
     /   /   \         ,/   .`|         /. ./|
    /   .     :      ,`   .'  :     .--'.  ' ;
   .   /   ;.  \   ;    ;     /    /__./ \ : |
  .   ;   /  ` ; .'___,/    ,' .--'.  '   \' .
  ;   |  ; \ ; | |    :     | /___/ \ |    ' '
  |   :  | ; | ' ;    |.';  ; ;   \  \;      :
  .   |  ' ' ' : `----'  |  |  \   ;  `      |
  '   ;  \; /  |     '   :  ;   .   \    .\  ;
   \   \  ',  /      |   |  '    \   \   ' \ |
    ;   :    /       '   :  |     :   '  |--"
     \   \ .'        ;   |.'       \   \ ;
  www. `---` ver     '---' he       '---" ire.org


Welcome to OverTheWire!

If you find any problems, please report them to the #wargames channel on
discord or IRC.

--[ Playing the games ]--

  This machine might hold several wargames.
  If you are playing "somegame", then:

    * USERNAMES are somegame0, somegame1, ...
    * Most LEVELS are stored in /somegame/.
    * PASSWORDS for each level are stored in /etc/somegame_pass/.

  Write-access to homedirectories is disabled. It is advised to create a
  working directory with a hard-to-guess name in /tmp/.  You can use the
  command "mktemp -d" in order to generate a random and hard to guess
  directory in /tmp/.  Read-access to both /tmp/ is disabled and to /proc
  restricted so that users cannot snoop on eachother. Files and directories
  with easily guessable or short names will be periodically deleted! The /tmp
  directory is regularly wiped.
  Please play nice:

    * don't leave orphan processes running
    * don't leave exploit-files laying around
    * don't annoy other players
    * don't post passwords or spoilers
    * again, DONT POST SPOILERS!
      This includes writeups of your solution on your blog or website!

--[ Tips ]--

  This machine has a 64bit processor and many security-features enabled
  by default, although ASLR has been switched off.  The following
  compiler flags might be interesting:

    -m32                    compile for 32bit
    -fno-stack-protector    disable ProPolice
    -Wl,-z,norelro          disable relro

  In addition, the execstack tool can be used to flag the stack as
  executable on ELF binaries.

  Finally, network-access is limited for most levels by a local
  firewall.

--[ Tools ]--

 For your convenience we have installed a few useful tools which you can find
 in the following locations:

    * gef (https://github.com/hugsy/gef) in /opt/gef/
    * pwndbg (https://github.com/pwndbg/pwndbg) in /opt/pwndbg/
    * gdbinit (https://github.com/gdbinit/Gdbinit) in /opt/gdbinit/
    * pwntools (https://github.com/Gallopsled/pwntools)
    * radare2 (http://www.radare.org/)

--[ More information ]--

  For more information regarding individual wargames, visit
  http://www.overthewire.org/wargames/

  For support, questions or comments, contact us on discord or IRC.

  Enjoy your stay!

```


A partir de aquí ya estaremos dentro, y como sabes que estamos buscando la clave para el siguiente nivel, lo que vamos a hacer es hacer un `ls` en busca de un archivo que contenga la clave:


```tmux
bandit0@bandit:~$ ls
readme
```

Vemos que fácilmente haciendo un `ls` encontramos este archivo de aquí, el cual vamos a leer haciendo un `cat`

```tmux
bandit0@bandit:~$ cat readme 
Congratulations on your first steps into the bandit game!!
Please make sure you have read the rules at https://overthewire.org/rules/
If you are following a course, workshop, walkthrough or other educational activity,
please inform the instructor about the rules as well and encourage them to
contribute to the OverTheWire community so we can keep these games free!

The password you are looking for is: [next_level_key]

bandit0@bandit:~$ 
```
Bien, como podemos ver en el texto que imprime al hacer cat a ese archivo, vemos que nos están dando como la enhorabuena por haber completado el primer nivel, entre otras cosas, y al final nos dan la [next_level_key] que básicamente es la clave al siguiente nivel, pero que mantendré censurada para respetar las normas de OTW.



# Nivel 1

### Objetivo: 

"La contraseña para el siguiente nivel se almacena en un archivo llamado - Ubicado en el directorio de inicio"

---

Como ya tenemos la clave para acceder al siguiente nivel, vamos a salirnos del que estamos ahora con exit y vamos a entrar en el nuevo cambiando el número al conectarnos por SSH:


```tmux
bandit0@bandit:~$ exit
logout
Connection to bandit.labs.overthewire.org closed.
[x@x ~]$ ssh -p 2220 bandit1@bandit.labs.overthewire.org
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       

                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

backend: gibson-1
bandit1@bandit.labs.overthewire.org's password: 
```


Ahora, vamos a poner la clave que nos han dado antes, en la contraseña que nos piden ahora, y veremos que nos deja entrar:

Cuando veamos esto, quiere decir que ya estamos dentro:

```tmux
bandit1@bandit:~$ 
```
Okey, ya estamos dentro, ahora, como nuestro objetivo indica, debemos buscar un archivo llamado "-", que está ubicado en el directorio de inicio.

Podemos comprobar en qué directorio estamos con:

```tmux
bandit1@bandit:~$ pwd
/home/bandit1
bandit1@bandit:~$ 
```
Ahora que ya vemos que estamos en el directorio de inicio, vamos a hacer un ls para ver si realmente está este archivo:

```tmux
bandit1@bandit:~$ ls
-
bandit1@bandit:~$ 
```

Como podemos ver tenemos este archivo, que se llama "-", ahora para leerlo deberíamos hacer un cat:

```tmux
bandit1@bandit:~$ cat -
^C
bandit1@bandit:~$ 
```

Pero si hacemos un `cat -` no sirve, pues está tomando el `-` como el inicio de una orden, como cuando hacemos `cat --help`

Con lo cual debemos especificar que nos referimos a ESE archivo.

Para ello vamos a utilizar ./
```tmux
bandit1@bandit:~$ cat ./-
[next_level_key]
bandit1@bandit:~$ 
```

Y con esto ya tenemos la clave del nivel dos, con lo cual, en resumen, solo teníamos que especificar que nos referíamos al archivo llamado "-" y no a las opciones de cat.

Salimos de la máquina:

```tmux
bandit1@bandit:~$ exit
```


# Nivel 2

### Objetivo:

La contraseña para el siguiente nivel se almacena en un archivo llamado `--spaces in this filename--` Ubicado en el directorio de inicio

---

Como en el anterior nivel, lo que vamos a hacer es conectarnos a la máquina

```tmux
[x@x ~]$ ssh -p 2220 bandit2@bandit.labs.overthewire.org
```

Ponemos la contraseña que hemos descubierto antes:

```tmux
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       

                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

backend: gibson-0
bandit2@bandit.labs.overthewire.org's password: 
```


Sabiendo ya cual es nuestro objetivo y estando ya dentro de la máquina, lo que vamos a hacer es un `ls` para ver dónde está ese archivo del que nos hablan

```tmux
bandit2@bandit:~$ ls
--spaces in this filename--
```

Okey, vemos el problema inicial, básicamente es un archivo que contiene "-" como en el nivel anterior, pero es que además este archivo tiene espacios en el nombre, esto quiere decir que si intentamos hacer un cat no nos va a dejar:

```tmux
bandit2@bandit:~$ cat ./--spaces in this filename-- 
cat: ./--spaces: No such file or directory
cat: in: No such file or directory
cat: this: No such file or directory
cat: filename--: No such file or directory
bandit2@bandit:~$ 
```

Vemos que nos está reconociendo cada palabra separada como un archivo, con lo cual, tenemos dos formas fáciles de hacerlo

1- Si tabulamos al poner el archivo:

```tmux
bandit2@bandit:~$ cat ./--spaces\ in\ this\ filename-- 
[next_level_key]
```
2- Si ponemos comillas ("")

```tmux
bandit2@bandit:~$ cat ./"--spaces in this filename--"
[next_level_key]
```


Estas son las formas más fáciles que podemos utilizar


# Nivel 3

### Objetivo:

"La contraseña para el siguiente nivel se almacena en un archivo oculto en la carpeta `inhere`."

---
Después de habernos logeado correctamente con la contraseña anterior en este nivel, podemos continuar.


Hacemos `ls` para ver la carpeta de la que nos están hablando:

```tmux
bandit3@bandit:~$ ls
inhere
```

Vemos que efectivamente tenemos la carpeta de la que nos hablaban está aquí, así que vamos a entrar con `cd` y volver a hacer un `ls` para buscar ese archivo, que al estar oculto, le haremos `ls -la` para ver archivos ocultos.

```tmux
bandit3@bandit:~/inhere$ ls -la
total 12
drwxr-xr-x 2 root    root    4096 Jun 24 14:59 .
drwxr-xr-x 3 root    root    4096 Jun 24 14:59 ..
-rw-r----- 1 bandit4 bandit3   33 Jun 24 14:59 ...Hiding-From-You
bandit3@bandit:~/inhere$ 
```

Ahora sí, vemos que tenemos este archivo llamado `...Hiding-From-You`

Sabiendo que el archivo está en esta carpeta, lo que vamos a hacer es leerlo como cualquier otro:

```tmux
bandit3@bandit:~/inhere$ cat ...Hiding-From-You 
[next_level_key]
```


Ya tenemos la clave para el nivel 4, nos salimos de la máquina

```tmux
bandit3@bandit:~/inhere$ exit
logout
Connection to bandit.labs.overthewire.org closed.
```


# Nivel 4

### Objetivo:

"La contraseña para el siguiente nivel está guardada en el único archivo legible por humanos dentro del directorio inhere. Consejo: si tu terminal se ha desconfigurado, prueba el comando `reset`."

---

Okey, vamos a conectarnos a esta máquina:

```tmux
[x@x ~]$ ssh -p 2220 bandit4@bandit.labs.overthewire.org
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       

                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

backend: gibson-0
bandit4@bandit.labs.overthewire.org's password: 
bandit4@bandit:~$ 
```
Ahora que ya estamos dentro, lo primero que vamos a hacer va a ser ver los directorios, para ver si está el directorio del que nos hablan:

```tmux

bandit4@bandit:~$ ls
inhere

```

okey, por el momento tenemos ubicada la carpeta de la que nos hablan, vamos a entrar y hacer un ls dentro para ver los archivos de los que nos hablan:

```tmux
bandit4@bandit:~$ cd inhere/
bandit4@bandit:~/inhere$ ls
-file00  -file01  -file02  -file03  -file04  -file05  -file06  -file07  -file08  -file09

```

Efectivamente, no va a ser tan fácil encontrar la key, en este caso tenemos 10 archivos, los cuales, no podemos leer de forma normal:

```tmux
bandit4@bandit:~/inhere$ cat ./-file00
�F1d�aIڗ�3n̊�����7��@Ԭ��ci^�bandit4@bandit:~/inhere$ 
```
Así que debemos encontrar el archivo que contenga la clave legible, y podríamos ir uno a uno, pero queremos es mejor buscar qué archivo es el que está en un formato normal para los humanos

Para ello vamos a utilizar el comando `file` y vamos a especificar que mire todos los archivos que estén en el directorio actual, para ello vamos a utilizar `./` para marcar que queremos el directorio actual, y `*` para marcar que mire todos los archivos, quedándonos un comando así: `file ./*`

```tmux
bandit4@bandit:~/inhere$ file ./*
./-file00: data
./-file01: data
./-file02: data
./-file03: data
./-file04: data
./-file05: data
./-file06: data
./-file07: ASCII text
./-file08: data
./-file09: data
bandit4@bandit:~/inhere$ 
```

Y con esto podemos ver que todos los archivos menos uno son de tipo "data" y uno de ellos es tipo ASCII text, que es un texto que podemos leer, con lo cual, ya sabemos que el archivo 07 es el que contiene la contraseña, así que vamos a leerlo con un simple cat:

```tmux
bandit4@bandit:~/inhere$ cat ./-file07
[next_level_key]
bandit4@bandit:~/inhere$ 
```
# Nivel 5

### Objetivo:

---

# Nivel 6

### Objetivo:

---


# Nivel 7

### Objetivo:

---


# Nivel 8

### Objetivo:

---


# Nivel 9

### Objetivo:

---


# Nivel 10

### Objetivo:

---


# Nivel 11

### Objetivo:

---


# Nivel 12

### Objetivo:

---


# Nivel 13

### Objetivo:

---

# Nivel 14

### Objetivo:

---

# Nivel 15

### Objetivo:

---


# Nivel 16

### Objetivo:

---


# Nivel 17

### Objetivo:

---


# Nivel 18

### Objetivo:

---


# Nivel 19

### Objetivo:

---


# Nivel 20

### Objetivo:

---



# Nivel 21

### Objetivo:

---


# Nivel 22

### Objetivo:

---

# Nivel 23

### Objetivo:

---


# Nivel 24

### Objetivo:

---


# Nivel 25

### Objetivo:

---


# Nivel 26

### Objetivo:

---



# Nivel 27

### Objetivo:

---


# Nivel 28

### Objetivo:

---


# Nivel 29

### Objetivo:

---


# Nivel 30

### Objetivo:

---

# Nivel 31

### Objetivo:

---

# Nivel 31

### Objetivo:

---



# Nivel 32

### Objetivo:

---


# Nivel 33

### Objetivo:

---


# Fin

Y con esto podemos dar por concluido el write-up de cómo resolver los minijuegos Bandit de OverTheWire
