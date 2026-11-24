+++
date = '2026-11-17T22:03:21+01:00'
draft = false
title = 'Classic Passwd THM Writeup'
summary = 'Writeup de Classic Passwd de TryHackMe'
tags = ["THM", "Blue-Team", "Reversing", "Hardcoded-Credentials", "Writeup"]
+++

![logo de Classic Passwd](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Classic_Passwd_logo.jpeg)

# Introducción

- **Plataforma:** TryHackMe
- **Sala:** Classic Passwd
- **Dificultad:** Medium
- **Categoría:** Blue Team (Reversing)

## Enunciado

Olvidé mi contraseña, ¿puedes darme acceso al programa?

## Herramientas

- `file`: para saber qué tipo de archivo es
- `strings`: para sacar las cadenas de texto que lleva el binario
- `objdump`: para desensamblar el ejecutable y leer su código

## Análisis inicial

Lo primero que voy a hacer es descargar el archivo que nos da la máquina, dándole al botón azul del enunciado:

![Botón para descargar este archivo](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Boton_Descarga_Classic_Passwd-THM.png)

Y vemos cómo se nos descarga el archivo:

![Archivo ya descargado](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Archivo_Descargado-Classic-Psswd-THM.png)

Ahora que ya lo tengo, lo que voy a hacer es ver qué tipo de archivo es:

```tmux
file Challenge_1609966715991.Challenge
```

![Resultado de file](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Tipo_Archivo-Classic-Passwd-THM.png)

Vemos que es un ejecutable ELF de 64 bits, así que le doy permisos de ejecución y lo ejecuto:

```tmux
chmod +x Challenge_1609966715991.Challenge
./Challenge_1609966715991.Challenge
```

![Le doy los permisos y lo ejecuto por primera vez](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Permisos_ejecucion_+_Ejecutado-Classic-Psswd-THM.png)

Bien, vemos cómo arranca el programa y nos pide un username. Pruebo los típicos (`admin`, `root`, `guest`...) pero todos me devuelven `Authentication Error`, así que por aquí no vamos a llegar a ningún lado.

![Intentos de username](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Intentos_username-Classic-Passwd-THM.png)

## Extracción de cadenas

Como adivinar el usuario no funciona, voy a ver qué lleva el programa dentro. Para eso uso `strings`, que me saca todo el texto legible del binario:

```tmux
strings Challenge_1609966715991.Challenge
```

![Salida de strings](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/string-Classic-Passwd-THM.png)

Entre todo lo que sale hay varias cosas que me llaman la atención:

- `Made by 4non`
- Una URL de GitHub
- `AGB6js5d`
- `9dkGf`
- `THM{%d%d}`

Lo más interesante es la última: `THM{%d%d}` tiene pinta de ser el formato de la flag, y los dos `%d` nos dicen que se construye con dos números. Además, la flag no está escrita tal cual, el programa la genera. Eso está bien saberlo.

## Desensamblado

Con `strings` no veo el username completo por ningún lado (`AGB6js5d` y `9dkGf` salen separados, y esa `f` del final es un byte de la instrucción siguiente que `strings` confunde con texto), así que toca mirar el código. Para eso uso `objdump`:

```tmux
objdump -d -M intel Challenge_1609966715991.Challenge
```

![Resultado del desensamblado](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/objdump-Classic-Psswd-THM.png)

Resumidamente lo que hace es lo siguiente:

- `-d`: desensambla el binario y me enseña las instrucciones
- `-M intel`: me las muestra con sintaxis Intel, que es más fácil de leer

Dentro del resultado busco la función `vuln`, y ahí veo varias instrucciones `mov` que van guardando valores hexadecimales en memoria. Esos valores son caracteres ASCII, pero están guardados en little endian, o sea, que hay que leer cada bloque al revés.

Por ejemplo, el valor `0x6e6f6e34` se guarda como los bytes `34 6e 6f 6e`, que en ASCII son `4non`. Se puede comprobar por terminal:

```tmux
echo 34 6e 6f 6e | xxd -r -p
```

![Traducción e inversión de los valores](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/traducción_valores_invertidos-Classic-Passwd-THM.png)

Al convertirlos y juntarlos todos me sale el username, que es justo el que el programa luego compara con `strcmp()` para ver si lo que hemos escrito es correcto:

Username: `AGB6js5d9dkG7`

Así que ahora que tenemos el usuario, vamos a ejecutar otra vez el programa y a poner ese usuario:

![Inicio de sesión con el usuario](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Logramos_entrar_+_Flag-Classic-Passwd-THM.png)

Perfecto, al iniciar sesión se ejecuta el bloque de código de `gfl` y nos imprime la flag en la terminal.

## Vulnerabilidad

La vulnerabilidad de este programa son las **credenciales hardcodeadas** (CWE-798): el username correcto está escrito dentro del propio ejecutable. Que esté troceado en varios `mov` en vez de aparecer como una cadena entera no es una protección real, solo una ofuscación mínima: cualquiera con `objdump` puede reconstruirlo, como acabamos de hacer.

## Segunda forma de resolverlo

Hasta ahora hemos obtenido la flag ejecutando el programa con el username correcto. Pero también se puede deducir sin ejecutarlo, leyendo directamente el código de la función `gfl`. Las dos formas llevan a la misma flag.

### Análisis de la función gfl

Una vez metemos bien el username, el programa ejecuta otra función llamada `gfl`.

![Función gfl](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/función_gfl-Classic-Passwd-THM.png)

Aquí hay dos bucles con condiciones de comparación, y mirando las instrucciones `cmp`, `add` y los saltos condicionales puedo ver qué valores hacen que se llegue hasta la llamada a `printf`.

El programa no calcula nada: cada bucle va sumando 1 a su contador y, cuando el primero llega a `0x638a78` y el segundo a `0x2130`, ejecuta `printf` con esos dos valores. Por eso la flag son directamente esos dos números.

- 0x638a78 en hexadecimal = 6.523.512 en decimal
- 0x2130 en hexadecimal = 8.496 en decimal

![Identificación de los valores del bucle](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/identificacion_bucle-Classic-Passwd-THM.png)

Y como vimos antes con `strings`, el formato es `THM{%d%d}`, así que la flag sale de dos números enteros en decimal:

- Primer número: *6523512*
- Segundo número: *8496*

Y con esto ya podemos formar la flag:

**THM{valor1+valor2}**

Como vemos, el resultado coincide con la flag que nos devolvió el programa al iniciar sesión.

## Obtención de la flag

Con esos dos números ya tengo la flag completa y la meto en la sala:

![Flag censurada](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Flag_en-THM.png)

## Conclusión

Este reto muestra por qué no se deben guardar credenciales dentro del binario: con `strings` y `objdump` basta para recuperarlas. Además, hemos visto que la flag se puede obtener de dos formas: ejecutando el programa con el usuario correcto o leyendo el código de `gfl`.

## Gracias por leer