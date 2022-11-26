# Ffuf

| **Command**   | **Description**   |
| --------------|-------------------|
| `ffuf -h` | ffuf help |
| `ffuf -w wordlist.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ` | Directory Fuzzing |
| `ffuf -w wordlist.txt:FUZZ -u http://SERVER_IP:PORT/indexFUZZ` | Extension Fuzzing |
| `ffuf -w wordlist.txt:FUZZ -u http://SERVER_IP:PORT/blog/FUZZ.php` | Page Fuzzing |
| `ffuf -w wordlist.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ -recursion -recursion-depth 1 -e .php -v` | Recursive Fuzzing |
| `ffuf -w wordlist.txt:FUZZ -u https://FUZZ.hackthebox.eu/` | Sub-domain Fuzzing |
| `ffuf -w wordlist.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb' -fs xxx` | VHost Fuzzing |
| `ffuf -w wordlist.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key -fs xxx` | Parameter Fuzzing - GET |
| `ffuf -w wordlist.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx` | Parameter Fuzzing - POST |
| `ffuf -w ids.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx` | Value Fuzzing |  

# Wordlists

| **Command**   | **Description**   |
| --------------|-------------------|
| `/opt/useful/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt` | Directory/Page Wordlist |
| `/opt/useful/SecLists/Discovery/Web-Content/web-extensions.txt` | Extensions Wordlist |
| `/opt/useful/SecLists/Discovery/DNS/subdomains-top1million-5000.txt` | Domain Wordlist |
| `/opt/useful/SecLists/Discovery/Web-Content/burp-parameter-names.txt` | Parameters Wordlist |

# Misc

| **Command**   | **Description**   |
| --------------|-------------------|
| `sudo sh -c 'echo "SERVER_IP  academy.htb" >> /etc/hosts'` | Add DNS entry |
| `for i in $(seq 1 1000); do echo $i >> ids.txt; done` | Create Sequence Wordlist |
| `curl http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=key' -H 'Content-Type: application/x-www-form-urlencoded'` | curl w/ POST |

## Directory Fuzzing:

```shell
# Para fuzear directorios: 
ffuf -w "wordlist" -u http://SERVER_IP:PORT/FUZZ

# Tambien se puede fuzzear por tipos de archivos, pero primero hemos de ver cuales admite la pagina. Para ello:

ffuf -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://134.209.19.24:32508/blog/indexFUZZ

# Una vez que sabemos que es php, podemos fuzear archivos php:

ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://134.209.19.24:32508/blog/FUZZ.php
```

## Recursive Fuzzing:

```shell
# Para fuzzear recursivamente, basta con pasarle el paremtro -recursion. Además, podemos elegir cuando de hondo queremos ir con : -recursion-depth X, siendo X el numero de subdirectorios a fuzzear.

ffuf -w "wordlist":FUZZ -u http://"ip:puerto"/FUZZ -recursion -recursion-depth 3 -e .php -v
```

##  Sub-domain Fuzzing & Vhost fuzzing

```shell
# Para fuzzear subdominios: 
ffuf -w "wordlist":FUZZ -u https://FUZZ."dominio"/

# Para fuzzear vhosts:
ffuf -w "wordlist":FUZZ -u http://"dominio":PORT/ -H 'Host: FUZZ."dominio"'
```

## Filtrado de resultados

```shell
# Para ocultar ciertos codigos al fuzzear:
-fs ocultar size
-fc http status code
-fl filtrar por las lineas en la respuesta
-fr regexp
-fw por las palabras que obtenemos en la respuesta

Ejemplo: ffuf -w /opt/useful/SecLists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb' -fs 900
```

## Parameter Fuzzing - GET

```shell
# Para fuzear los parametros que acepta la pagina:
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://"dominio/IP":"puerto"/"directorio"?FUZZ=key -fs 798

# ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:31419/admin/admin.php?FUZZ=key -fs 798

```

## Parameter Fuzzing - POST

```shell
# Post: Las peticiones no se exponen en la URL. 
# GET: las peticiones si se exponen en la URL. 

ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx

# Para fuzear los parametros obtenidos:
ffuf -w ids.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx

# Finalmente, podemos hacerle un curl a la pagina para ver k nos devuelve
curl http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=key' -H 'Content-Type: application/x-www-form-urlencoded'
```