## Scanning Options

| **Nmap Option** | **Description** |
|---|----|
| `10.10.10.0/24` | Target network range. |
| `-sn` | Disables port scanning. |
| `-Pn` | Disables ICMP Echo Requests |
| `-n` | Disables DNS Resolution. |
| `-PE` | Performs the ping scan by using ICMP Echo Requests against the target. |
| `--packet-trace` | Shows all packets sent and received. |
| `--reason` | Displays the reason for a specific result. |
| `--disable-arp-ping` | Disables ARP Ping Requests. |
| `--top-ports=<num>` | Scans the specified top ports that have been defined as most frequent.  |
| `-p-` | Scan all ports. |
| `-p22-110` | Scan all ports between 22 and 110. |
| `-p22,25` | Scans only the specified ports 22 and 25. |
| `-F` | Scans top 100 ports. |
| `-sS` | Performs an TCP SYN-Scan. |
| `-sA` | Performs an TCP ACK-Scan. |
| `-sU` | Performs an UDP Scan. |
| `-sV` | Scans the discovered services for their versions. |
| `-sC` | Perform a Script Scan with scripts that are categorized as "default". |
| `--script <script>` | Performs a Script Scan by using the specified scripts. |
| `-O` | Performs an OS Detection Scan to determine the OS of the target. |
| `-A` | Performs OS Detection, Service Detection, and traceroute scans. |
| `-D RND:5` | Sets the number of random Decoys that will be used to scan the target. |
| `-e` | Specifies the network interface that is used for the scan. |
| `-S 10.10.10.200` | Specifies the source IP address for the scan. |
| `-g` | Specifies the source port for the scan. |
| `--dns-server <ns>` | DNS resolution is performed by using a specified name server. |

## Output Options

| **Nmap Option** | **Description** |
|---|----|
| `-oA filename` | Stores the results in all available formats starting with the name of "filename". |
| `-oN filename` | Stores the results in normal format with the name "filename". |
| `-oG filename` | Stores the results in "grepable" format with the name of "filename". |
| `-oX filename` | Stores the results in XML format with the name of "filename". |

## Performance Options

| **Nmap Option** | **Description** |
|---|----|
| `--max-retries <num>` | Sets the number of retries for scans of specific ports. |
| `--stats-every=5s` | Displays scan's status every 5 seconds. |
| `-v/-vv` | Displays verbose output during the scan. |
| `--initial-rtt-timeout 50ms` | Sets the specified time value as initial RTT timeout. |
| `--max-rtt-timeout 100ms` | Sets the specified time value as maximum RTT timeout. |
| `--min-rate 300` | Sets the number of packets that will be sent simultaneously. |
| `-T <0-5>` | Specifies the specific timing template. |


## Host Discovery

```shell
sudo nmap -sn -oA tnet -iL hosts.lst | grep for | cut -d" " -f5

# Con esto conseguimos la respuesta de aquellas IPS que NO ignoran las ICMP echo requests debido a la configuracion del Firewall

sudo nmap -sn -oA tnet 10.129.2.18 10.129.2.19 10.129.2.20| grep for | cut -d" " -f5

# Escanear varias IPs. Tambien se puede poner rango: 10.129.2.18-20
```

```shell
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace 

# Si no usamos el parametro -sn (port scanning) usaremos ICMP echo requests scans por defecto (-PE), por que nos comunicaremos mediante pings y respuestas ARP-ARP reply, por lo que hemos de usar el parametro --packet-trace para confirmar la comunicacion con los puertos. 

# Si no deseamos ver las comunicaciones ARP podemos quitarlas usando el paratmetro
--disable-arp-ping
```

## Host and Port Scanning

Al hacer un escaneo, podemos encontrarnos ante 6 tipos de puertos:

```shell
open 
# La conexión mediante el escaneo NMAP ha sido exitoso.
closed 
# Al hacer el escaneo nmap recibimos un paquete que nos dice que el puerto esta cerrado.
filtered 
# Nmap no sabe si dicho puerto esta abierto o cerrado, pues la respuesta que recibimos es "confusa".
unfiltered
# Ocurre durante el handshake TCP-ACK y significa que el puerto está accesible pero no sabemos si está abierto o cerrado.
open | filtered 
# Si no recibimos una respuesta al escanear este tipo de puerto, le será asignado este estado por defecto. Puede ser que el firewall esté protegiendo dicho puerto
closed | filtered 
# Mediante el escaneo TCP idle es imposible determinar si el puerto está abierta o cerrado.
```

### Escanear puertos por  TCP

```shell
# Por defecto los escaneos NMAP SIENDO ROOT se realizan usando SYN scan (-sS), sino se usará TCP scan (-sT). 
# Maneras de definir los puertos a escanear:
-Individualmente: (-p p1,p2,p3..etc)
-Rango (-p puerto_inicio-puerto_fin -> -p 22-445)
-Top ports (--top-ports=x), siendo X el numero de puertos deseados. 
```

Ejemplo: 

```RUBY
VIKSANT@USER[/USER]$ sudo nmap 10.129.2.28 --top-ports=10 
PORT     STATE    SERVICE
21/tcp   closed   ftp
22/tcp   open     ssh
23/tcp   closed   telnet
25/tcp   open     smtp
80/tcp   open     http
110/tcp  open     pop3
139/tcp  filtered netbios-ssn
443/tcp  closed   https
445/tcp  filtered microsoft-ds
3389/tcp closed   ms-wbt-server
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

# Si queremos analizar detenidamente la respuesta que recibimos de un puerto, debemos quitarle las requests ICMP (-Pn), DNS Resolution (-n) y el arp ping scan (--disable-arp-ping), por lo que nos reportará algo así:

VIKSANT@USER[/USER]$ sudo nmap 10.129.2.28 -p 21 --packet-trace -Pn -n --disable-arp-ping

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:39 CEST
SENT (0.0429s) TCP 10.10.14.2:63090 > 10.129.2.28:21 S ttl=56 id=57322 iplen=44  seq=1699105818 win=1024 <mss 1460>
RCVD (0.0573s) TCP 10.129.2.28:21 > 10.10.14.2:63090 RA ttl=64 id=0 iplen=40  seq=0 win=0
Nmap scan report for 10.11.1.28
Host is up (0.014s latency).

PORT   STATE  SERVICE
21/tcp closed ftp
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.07 seconds
# De esta manera, podemos sacar más informacion, como por ejemplo el ttl de la maquina (ttl = 64) y saber que estamos ante una maquina linux (windows tiene 128 de ttl)

# Podemos hacer lo mismo con los puertos filtrados. Por ejemplo 

VIKSANT@USER[/USER]$ sudo nmap 10.129.2.28 -p 139 --packet-trace -n --disable-arp-ping -Pn

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:45 CEST
SENT (0.0381s) TCP 10.10.14.2:60277 > 10.129.2.28:139 S ttl=47 id=14523 iplen=44  seq=4175236769 win=1024 <mss 1460>
SENT (1.0411s) TCP 10.10.14.2:60278 > 10.129.2.28:139 S ttl=45 id=7372 iplen=44  seq=4175171232 win=1024 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up.

PORT    STATE    SERVICE
139/tcp filtered netbios-ssn
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 2.06 seconds

# Como se puede apreciar,  hemos enviado 2 paquetes (Porque tenemos 2 SENT) y no tenemos ningun RCVD (como en el caso anterior) entoces significa que el puerto o bien está cerrado o filtrado. Además, viendo el tiempo de escaneo (2.06s vs 0.07s) tambien tenemos una pista de que el puerto no está abierto. (Avanzado: Hemos enviado dos paquetes SYN y no hemos recibido ningun paquete SYN-ACK que es el que indica que hemos conectado satisfactoriamente con el puerto). 

# No obstante, tambien existe el caso de que recibamos una respuesta del puerto pero este siga estando cerrado/filtrado. Esto ocurre cuando el firewall nos lo rechaza. Como es en este caso:

VIKSANT@USER[/USER]$ sudo nmap 10.129.2.28 -p 445 --packet-trace -n --disable-arp-ping -Pn

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:55 CEST
SENT (0.0388s) TCP 10.129.2.28:52472 > 10.129.2.28:445 S ttl=49 id=21763 iplen=44  seq=1418633433 win=1024 <mss 1460>
RCVD (0.0487s) ICMP [10.129.2.28 > 10.129.2.28 Port 445 unreachable (type=3/code=3) ] IP [ttl=64 id=20998 iplen=72 ]
Nmap scan report for 10.129.2.28
Host is up (0.0099s latency).

PORT    STATE    SERVICE
445/tcp filtered microsoft-ds
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.05 seconds
```

### Escanear puertos por  UDP

```ruby
# La pregunta que probablemente te haces es: Para que se escanea por UDP? Pues bien. Te voy a dar la respuesta corta: UDP no es tan estricto como TCP y puede mostrarte puertos abiertos mientras que ese mismo puerto TCP te lo puede mostrar cerrrado. Además, este protocolo se usa para pasar sobre todo archivos media (fotos, videos, etc). 

# Para hacer el escaneo por UDP simplemente tenemos que pasarle el parametro -sU: sudo nmap 10.129.2.28 -F -sU

VIKSANT@USER[/USER]$ sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 137 --reason

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 16:01 CEST
Nmap scan report for 10.129.2.28
Host is up (0.059s latency).
Not shown: 95 closed ports
PORT     STATE         SERVICE
68/udp   open|filtered dhcpc
137/udp  open          netbios-ns
138/udp  open|filtered netbios-dgm
631/udp  open|filtered ipp
5353/udp open          zeroconf
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 98.07 seconds

# Para no entrar en detalles, ten en cuenta que el escaneo por UDP tardo mucho más que el TCP porque es un protocolo sin estado, por lo que no requiere tantas "confirmaciones" como el TCP. 
```

### Consejos/Tips

```ruby
# Podemos usar los paramentros -sV y -sC (-sCV) una vez tenemos los puertos enumerados mediante nmap para ver más informacion sofisticada del target. Por ejemplo:

❯ sudo nmap -p- -sS --min-rate 5000 10.129.123.74 -n -vvv -Pn
[sudo] password for viksant: 
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.93 ( https://nmap.org ) at 2022-11-18 22:48 CET
Initiating SYN Stealth Scan at 22:48
Scanning 10.129.123.74 [65535 ports]
Discovered open port 80/tcp on 10.129.123.74
Discovered open port 22/tcp on 10.129.123.74
Discovered open port 110/tcp on 10.129.123.74
Discovered open port 139/tcp on 10.129.123.74
Discovered open port 445/tcp on 10.129.123.74
Discovered open port 143/tcp on 10.129.123.74
Discovered open port 31337/tcp on 10.129.123.74
Completed SYN Stealth Scan at 22:48, 11.99s elapsed (65535 total ports)
Nmap scan report for 10.129.123.74
Host is up, received user-set (0.049s latency).
Scanned at 2022-11-18 22:48:07 CET for 12s
Not shown: 65528 closed tcp ports (reset)
PORT      STATE SERVICE      REASON
22/tcp    open  ssh          syn-ack ttl 63
80/tcp    open  http         syn-ack ttl 63
110/tcp   open  pop3         syn-ack ttl 63
139/tcp   open  netbios-ssn  syn-ack ttl 63
143/tcp   open  imap         syn-ack ttl 63
445/tcp   open  microsoft-ds syn-ack ttl 63
31337/tcp open  Elite        syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 12.08 seconds
   Raw packets sent: 65538 (2.884MB) | Rcvd: 65535 (2.62)


❯ sudo nmap -p22,80,110,139,143,445,31337 -sCV 10.129.123.74
Starting Nmap 7.93 ( https://nmap.org ) at 2022-11-18 22:49 CET
Nmap scan report for 10.129.123.74
Host is up (0.046s latency).

PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 71c189907ffd4f60e054f385e6356c2b (RSA)
|   256 e18e531842af2adec0121e2e54064f70 (ECDSA)
|_  256 1accacd4945cd61d71e739de14273c3c (ED25519)
80/tcp    open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
110/tcp   open  pop3        Dovecot pop3d
|_pop3-capabilities: RESP-CODES CAPA UIDL SASL TOP AUTH-RESP-CODE PIPELINING
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp   open  imap        Dovecot imapd
|_imap-capabilities: LOGINDISABLEDA0001 ENABLE OK IDLE more post-login capabilities LITERAL+ listed Pre-login have IMAP4rev1 SASL-IR LOGIN-REFERRALS ID
445/tcp   open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
31337/tcp open  Elite?
Service Info: Host: NIX-NMAP-DEFAULT; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-time: 
|   date: 2022-11-18T21:54:12
|_  start_date: N/A
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
|_clock-skew: mean: -18m02s, deviation: 34m38s, median: 1m57s
|_nbstat: NetBIOS name: NIX-NMAP-DEFAUL, NetBIOS user: <unknown>, NetBIOS MAC: 000000000000 (Xerox)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: nix-nmap-default
|   NetBIOS computer name: NIX-NMAP-DEFAULT\x00
|   Domain name: \x00
|   FQDN: nix-nmap-default
|_  System time: 2022-11-18T22:54:12+01:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 186.45 seconds
```