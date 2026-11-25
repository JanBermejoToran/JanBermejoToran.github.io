+++
date = '2026-11-17T22:03:21+01:00'
draft = false
title = 'Classic Passwd THM Writeup'
summary = 'Writeup de Classic Passwd de TryHackMe'
tags = ["THM", "Blue-Team", "Reversing", "Hardcoded-Credentials", "Writeup"]
+++

![logo de Classic Passwd](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Classic_Passwd_logo.jpeg)

# Introducció

- **Plataforma:** TryHackMe
- **Sala:** Classic Passwd
- **Dificultat:** Medium
- **Categoria:** Blue Team (Reversing)

## Enunciat

He oblidat la contrasenya, em pots donar accés al programa?

## Eines

- `file`: per saber quin tipus d'arxiu és
- `strings`: per treure les cadenes de text que porta el binari
- `objdump`: per desensamblar l'executable i llegir-ne el codi

## Anàlisi inicial

El primer que faré serà descarregar l'arxiu que ens dona la màquina, clicant el botó blau de l'enunciat:

![Botó per descarregar aquest arxiu](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Boton_Descarga_Classic_Passwd-THM.png)

I veiem com es descarrega l'arxiu:

![Arxiu ja descarregat](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Archivo_Descargado-Classic-Psswd-THM.png)

Ara que ja el tinc, miraré quin tipus d'arxiu és:

```bash
file Challenge_1609966715991.Challenge
```

![Resultat de file](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Tipo_Archivo-Classic-Passwd-THM.png)

Veiem que és un executable ELF de 64 bits, així que li done permisos d'execució i l'executo:

```bash
chmod +x Challenge_1609966715991.Challenge
./Challenge_1609966715991.Challenge
```

![Li done els permisos i l'executo per primera vegada](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Permisos_ejecucion_+_Ejecutado-Classic-Psswd-THM.png)

Bé, veiem com arrenca el programa i ens demana un username. Provo els típics (`admin`, `root`, `guest`...) però tots em retornen `Authentication Error`, així que per aquí no arribarem enlloc.

![Intents de username](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Intentos_username-Classic-Passwd-THM.png)

## Extracció de cadenes

Com que endevinar l'usuari no funciona, miraré què porta el programa a dins. Per a això faig servir `strings`, que em treu tot el text llegible del binari:

```bash
strings Challenge_1609966715991.Challenge
```

![Sortida de strings](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/string-Classic-Passwd-THM.png)

Entre tot el que surt hi ha diverses coses que em criden l'atenció:

- `Made by 4non`
- Una URL de GitHub
- `AGB6js5d`
- `9dkGf`
- `THM{%d%d}`

El més interessant és l'última: `THM{%d%d}` té pinta de ser el format de la flag, i els dos `%d` ens diuen que es construeix amb dos números. A més, la flag no està escrita tal qual, el programa la genera. Això està bé saber-ho.

## Desensamblat

Amb `strings` no veig el username complet enlloc (`AGB6js5d` i `9dkGf` surten separats, i aquesta `f` del final és un byte de la instrucció següent que `strings` confon amb text), així que toca mirar el codi. Per a això faig servir `objdump`:

```bash
objdump -d -M intel Challenge_1609966715991.Challenge
```

![Resultat del desensamblat](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/objdump-Classic-Psswd-THM.png)

Resumidament, el que fa és el següent:

- `-d`: desensambla el binari i m'ensenya les instruccions
- `-M intel`: me les mostra amb sintaxi Intel, que és més fàcil de llegir

Dins del resultat busque la funció `vuln`, i allí veig vàries instruccions `mov` que van guardant valors hexadecimals a la memòria. Aquests valors són caràcters ASCII, però estan guardats en little endian, és a dir, que cal llegir cada bloc al revés.

Per exemple, el valor `0x6e6f6e34` es guarda com els bytes `34 6e 6f 6e`, que en ASCII són `4non`. Es pot comprovar per terminal:

```bash
echo 34 6e 6f 6e | xxd -r -p
```

![Traducció i inversió dels valors](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/traducción_valores_invertidos-Classic-Passwd-THM.png)

En convertir-los i ajuntar-los tots em surt el username, que és just el que el programa després compara amb `strcmp()` per veure si el que hem escrit és correcte:

Username: `AGB6js5d9dkG7`

Així que ara que tenim l'usuari, tornarem a executar el programa i hi posarem aquest usuari:

![Inici de sessió amb l'usuari](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Logramos_entrar_+_Flag-Classic-Passwd-THM.png)

Perfecte, en iniciar sessió s'executa el bloc de codi de `gfl` i ens imprimeix la flag a la terminal.

## Vulnerabilitat

La vulnerabilitat d'aquest programa són les **credencials hardcodejades** (CWE-798): el username correcte està escrit dins del propi executable. Que estiga trossejat en diversos `mov` en lloc d'aparéixer com una cadena sencera no és una protecció real, només una ofuscació mínima: qualsevol amb `objdump` pot reconstruir-lo, com acabem de fer.

## Segona manera de resoldre-ho

Fins ara hem obtingut la flag executant el programa amb el username correcte. Però també es pot deduir sense executar-lo, llegint directament el codi de la funció `gfl`. Les dues maneres porten a la mateixa flag.

### Anàlisi de la funció gfl

Una vegada posem bé el username, el programa executa una altra funció anomenada `gfl`.

![Funció gfl](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/función_gfl-Classic-Passwd-THM.png)

Aquí hi ha dos bucles amb condicions de comparació, i mirant les instruccions `cmp`, `add` i els salts condicionals puc veure quins valors fan que s'arribe fins a la crida a `printf`.

El programa no calcula res: cada bucle va sumant 1 al seu comptador i, quan el primer arriba a `0x638a78` i el segon a `0x2130`, executa `printf` amb aquests dos valors. Per això la flag són directament aquests dos números.

- 0x638a78 en hexadecimal = 6.523.512 en decimal
- 0x2130 en hexadecimal = 8.496 en decimal

![Identificació dels valors del bucle](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/identificacion_bucle-Classic-Passwd-THM.png)

I com vam veure abans amb `strings`, el format és `THM{%d%d}`, així que la flag surt de dos números enters en decimal:

- Primer número: *6523512*
- Segon número: *8496*

I amb això ja podem formar la flag:

**THM{valor1+valor2}**


Com veiem, el resultat coincideix amb la flag que ens va retornar el programa en iniciar sessió.

## Obtenció de la flag

Amb aquests dos números ja tinc la flag completa i la poso a la sala:

![Flag censurada](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Flag_en-THM.png)

## Conclusió

Aquest repte mostra per què no s'han de guardar credencials dins del binari: amb `strings` i `objdump` n'hi ha prou per recuperar-les. A més, hem vist que la flag es pot obtenir de dues maneres: executant el programa amb l'usuari correcte o llegint el codi de `gfl`.

## Gràcies per llegir