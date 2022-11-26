| **Command** | **Description** |
| --------------|-------------------|
| `pip install hashid` | Install the `hashid` tool |
| `hashid <hash>` OR `hashid <hashes.txt>` | Identify a hash with the `hashid` tool |
| `hashcat --example-hashes` | View a list of `Hashcat` hash modes and example hashes |
| `hashcat -b -m <hash mode>` | Perform a `Hashcat` benchmark test of a specific hash mode |
| `hashcat -b` | Perform a benchmark of all hash modes |
| `hashcat -O` | Optimization: Increase speed but limit potential password length |
| `hashcat -w 3` | Optimization: Use when Hashcat is the only thing running, use 1 if running hashcat on your desktop.  Default is 2 |
| `hashcat -a 0 -m <hash type> <hash file> <wordlist>` | Dictionary attack |
| `hashcat -a 1 -m <hash type> <hash file> <wordlist1> <wordlist2>` | Combination attack |
| `hashcat -a 3 -m 0 <hash file> -1 01 'ILFREIGHT?l?l?l?l?l20?1?d'` | Sample Mask attack |
| `hashcat -a 7 -m 0 <hash file> -1=01 '20?1?d' rockyou.txt` | Sample Hybrid attack |
| `crunch <minimum length> <maximum length> <charset> -t <pattern> -o <output file>` | Make a wordlist with `Crunch` |
| `python3 cupp.py -i` | Use `CUPP` interactive mode |
| `kwp -s 1 basechars/full.base keymaps/en-us.keymap  routes/2-to-10-max-3-direction-changes.route` | `Kwprocessor` example |
| `cewl -d <depth to spider> -m <minimum word length> -w <output wordlist> <url of website>` | Sample `CeWL` command |
| `hashcat -a 0 -m 100 hash rockyou.txt -r rule.txt` | Sample `Hashcat` rule syntax |
| `./cap2hccapx.bin input.cap output.hccapx` | `cap2hccapx` syntax |
| `hcxpcaptool -z pmkidhash_corp cracking_pmkid.cap ` | `hcxpcaptool`syntax |

## Hashing vs. Encryption

```shell
# Para crear un hash a partir de texto:
echo -n "texto" / 'texto' | tipo hash

# Para crear cipher texto (usando xor):
$ python3
$ from pwn import xor
$ xor("password","key")
```

## Identifying Hashes

```shell
# Si por ejemplo tenemos el siguiente hash: $a$bbb$ccccc, a (numero) seria es el id, b es el salt y c el hash en sí
# $1$  : MD5
# $2a$ : Blowfish
# $2y$ : Blowfish, with correct handling of 8 bit characters
# $5$  : SHA256
# $6$  : SHA512
# $apr1$ : Apache
# $P$ : Wordpress
```

## Dictionary Attack

```shell
# Esto es para usar un diccionario a nuestra elección para crackear un hash.
hashcat -a 0 -m <hash type> <hash file> <wordlist>
```

## Hashcat combination attack:

```shell
# Para combinar diccionarias a la hora de creackear un hash.
hashcat -a 1 -m <hash type> <hash file> <wordlist1> <wordlist2>
```

## Mask Attack

```shell
# Los mask attack se usan para generar palabras que coincidan con un patron especifico. Esto se hace cuando se tiene X informacion sobre la password, como su longitud, tipo de caracteres, etc.

# Para esto, hay que usar el modo de hasheo -a 3. 

# Ejemplo. Sabemos que la password tiene este formato: 
# VIKSANT<userid><año>, donde userid tiene 5 caracteres y el año es posterior a 2000. Podriamos usar la mask siguiente: VIKSANT?l?l?l?l?l20[0-1]?d, por lo que el comando quedaria así:
hashcat -a 3 -m 0 hash.txt -1 01 'VIKSANT?l?l?l?l?l20?1?d'

# Donde -1 es una opcion para espeficicar que 0 y 1 son placeholders
```

## Hybrid Mode

```shell
# Con el modo hybrido podemos usar varios modelos de ataque a la vez. Se especifica con el parametro -a 6. 

hashcat -a 6 -m 100 hash.txt /usr/share/wordlists/rockyou.txt '?d?s'

# Tambien podemos usar el modo -a 7 para añadir caracteres a nuestra wordlist en funcion de la MASK ATTACK que tengamos. Por ejemplo:

hashcat -a 7 -m 0 hash.txt -1 01 'mask' /usr/share/wordlists/rockyou.txt

```

## Creating Custom Wordlists

```shell
# Herramientas que podemos usar para crear Custom Wordlists:

# --Crunch--
## Create wordlist
crunch <minimum length> <maximum length> <charset> -t <pattern> -o <output file>

## Using Pattern
crunch "min.length" "max.length" -t "ILFREIGHT201%@@@@" -o wordlist

## Specified Repetition
crunch "min.length" "max.length" -t "texto-repetido"@@@@ -d 1 -o wordlist

# --Cupp-- 
## simplemente ponemos cupp -i y metemos los datos k se nos pidan.

# --CeWL--
cewl -d <depth to spider> -m <minimum word length> -w <output wordlist> <url of website>

## Ejemplo:
cewl -d 5 -m 8 -e http://inlanefreight.com/blog -w wordlist.txt
```

## Working with Rules
![[Pasted image 20221121000042.png]]
```shell
# Ejemplo: 
c so0 si1 se3 ss5 sa@ $2 $0 $1 $9
# cada valor que acompañe a la "s" será modificado por el valor proximo. so0 = o será cabmiado por 0 y asi. 
echo 'password_ilfreight' > test.txt
hashcat -r rule.txt test.txt --stdout
# Obtenemos:P@55w0rd_1lfr31ght2019
```

## Generate Random Rules

```shell
# Podemos usar nuestras propias reglas a la hora de romper un hash:
hashcat -a 0 -m 100 -g 1000 hash "wordlist"

# Tambien podemos añadirle 
```

![[Pasted image 20221121005307.png]]

## Cracking Miscellaneous Files & Hashes

```shell
# --Cracking Password Protected Microsoft Office Documents--
# Extraer hash
python office2john.py hashcat_Word_example.docx 

# Tambien podemos creackear archivos de office con hashcat:
-m 9400 -> MS OFFICE 2007
-m 9500 -> MS OFFICE 2010
-m 9600 -> MS OFFICE 2013

# --Cracking Password Protected Zip Files--
# hascat modes:
11600 ->	7-Zip
13600 ->	WinZip
17200 ->	PKZIP (Compressed)
17210 ->	PKZIP (Uncompressed)
17220 ->	PKZIP (Compressed Multi-File)
17225 ->	PKZIP (Mixed Multi-File)
17230 ->	PKZIP (Compressed Multi-File Checksum-Only)
23001 ->	SecureZIP AES-128
23002 ->	SecureZIP AES-192
23003 ->	SecureZIP AES-256
13400 ->	KeePass 1 AES / without keyfile
13400 ->	KeePass 2 AES / without keyfile
13400 ->	KeePass 1 Twofish / with keyfile
13400 ->	Keepass 2 AES / with keyfile
10400 ->	PDF 1.1 - 1.3 (Acrobat 2 - 4)
10410 ->	PDF 1.1 - 1.3 (Acrobat 2 - 4), collider #1
10420 ->	PDF 1.1 - 1.3 (Acrobat 2 - 4), collider #2
10500 ->	PDF 1.4 - 1.6 (Acrobat 5 - 8)
10600 ->	PDF 1.7 Level 3 (Acrobat 9)
10700 ->	PDF 1.7 Level 8 (Acrobat 10 - 11)
```

## Cracking Wireless (WPA/WPA2) Handshakes with Hashcat

```shell
# Pasamos el archivo .cap a hash mode usando el comando:
hcxpcapngtool archivo_entrada.cap -o archivo_salida.hccapx (MIC)
hcxpcapngtool archivo_entrada.cap -o archivo_salida.cap (PKMID)
```



