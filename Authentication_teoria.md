## Vulnerabilities in password-based login

#### Username enumeration
```shell
# Si la pagina muestra errores diferentes para el usuario y contraseña, podemos bruteforcearlas con BurpSuite Intruder. 
```

## Vulnerabilities in multi-factor authentication

#### Bypassing two-factor authentication
```shell
# A veces, la 2FA verification se hace en otra pagina la cual se puede bypassear. Para nuestro caso, como tenemos acceso al 2FA del usuario wiener:peter, podemos ver que una vez dentro nos redigire a POST /my-account HTTP/1.1 por lo que podemos copiar dicha cabezera POST y pegarla en la pagina de 2FA del usuario carlos:montoya
```

#### Flawed two-factor verification logic / 2FA borken logic
```shell
#A veces, una vez nos logeamos en la pagina web y tenemos que rellenar el apartado de 2FA, la pagina web no siempre revisa minuciosamente el usuario que se está identificando, sea el mismo que el de antes del 2FA, por lo que podemos interceptar la petición en la pestaña de /login2 (2FA), y cambiar el identificador de usuario de wiener al que nos interesa (Carlos en este caso).
```

#### Resetting passwords using a URL
```shell
# A veces, las paginas web a la hora de mandar el correo de cambio de contraseña, especifican un nombre de usuario en el URL, en lugar de un token seguro, por lo que cambiando el nombre de usuario cambiarimos la contraseña de dicho usuario.
```

#### Username enumeration via subtly different responses
```shell
# En ocasiones, las paginas no son uniformes en sus respuestas (excepto cuando introducimos el usuario y contraseña correcto), por lo que tenemos una via potencial de enumeración. En este caso, es de usuario, y aunque al hacer un simple login no sabemos que hemos introducido mal (ya que pone invalid username or password), ocurre algo en el backend de la pagina cuando introducimos el usuario correcto. Esto también occure en aquellas paginas que procesan antes el usuario metido en vez de hacerlo a la vez con usuario y contraseña.
```

#### Username enumeration via response timing
```shell
# Al igual que en el caso anterior, la pagina puede presentar diferentes tipos de respuesta de tiempo dependiendo de si le "gustan" o no los datos que le metemos
```

#### Flawed brute-force protection / Broken brute-force protection, IP block
```shell
# Ciertas paginas, cada X intentos fallidos de inicio de sesion te bloquean la IP cada Y segundos. No obstante, si conseguimos ingresar las credenciales de una cuenta ya existente (logearnos), este bloqueo se nos quita. Por lo tanto, hemos de ver cada cuantos intentos fallidos (Z) se nos bloquea la IP para asi meter las credenciales correctas en el momento Z-1.
```

#### Account locking
```shell
# A veces, los sitios web intentan evitar la fuerza bruta bloqueando la cuenta si se cumplen ciertos criterios sospechosos, normalmente un número determinado de intentos de inicio de sesión fallidos. Al igual que con los errores de inicio de sesión normales, las respuestas del servidor que indican que una cuenta está bloqueada también pueden ayudar a un atacante a enumerar los nombres de usuario.

# En el caso de este lab de portswigger, al introducirle un usuario correcto pero la contraseña incorrecta un X numero de veces, nos pondrá que la cuenta ha sido bloqueada durante 1minuto, por lo que sabemos que dicho usuario es valido. 
```