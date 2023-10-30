# Gestión Avanzada de Bases de Datos Geoespaciales para Análisis Urbanos
## Clase de introducción al uso de consultas espaciales en PosrgreSQL-PostGIS

El objetivo de esta práctica es entender la forma en que se deben realizar las consultas espaciales y no espaciales haciendo uso del lenguaje SQL (Structured Query Language). Se comenzará a explorar los datos proporcionados del municipio de Umán.

En esta práctica vamos a explorar datos relacionados con los incidentes viales, el uso de suelo y datos de población del municipio de Umán.   

Para comenzar, creamos una base de datos que se llame practica lum y un esquema que se llame practica01: 

``` sql
create database lum
;
create extension postgis;
create schema practica01;
```
¿Porqué usar esquemas? 

Permiten organizar los objetos de la base de datos, por ejemplo, las tablas en grupos lógicos para hacerlos más manejables.
Hacen posible que varios usuarios utilicen una base de datos sin interferir entre sí.

# PREPARANDO LOS DATOS

Para poder comenzar a trabajar con los datos, es necesario explorarlos e identificar en qué condiciones se encuentran para poder procesarlos y extraer información de ellos. Cuando se trabaja con datos tanto espaciales como no espaciales es necesario que los datos cumplan con tres condiciones importantes: contar como llaves primarias y secundarias, tener indices espaciales y definir la proyección espacial idonea para trabajar con ellos.

Es conveniente que las tablas y las columnas no tengan espcios en sus nombres o caracteres especiales.
Puede ser necesario cambiar el nombre tamién con fines de estandarizar los datos.

Se visualizan los diez primeros datos de las tablas para revisar el contenido

``` sql
--Polígonos de uso de suelo 
select * from practica01."USOS DE SUELO_final" limit 10;

--Polígono
select * from practica01."DENSIDAD Umán_Final_ITRF2008" limit 10;

--Calles
select * from practica01."Ejes viales_topologia" limit 10;

--Puntos de incidentes viales
select * from practica01."Incidentes viales AXA Umán" limit 10;

```

Se realiza un cambio de nombre de las tablas 

``` sql

ALTER TABLE practica01."USOS DE SUELO_final" RENAME TO USOS_DE_SUELO;

ALTER TABLE practica01."DENSIDAD Umán_Final_ITRF2008" RENAME TO DENSIDAD;

ALTER TABLE practica01."Ejes viales_topologia" RENAME TO Ejes_viales_topologia;

ALTER TABLE practica01."Incidentes viales AXA Umán" RENAME TO Incidentes_viales

```
Una vez cambiado el nombre, se acutualizan los nombres de las tablas

``` sql
--Polígonos de uso de suelo 
select * from practica01.USOS_DE_SUELO limit 10;

--Polígono
select * from practica01.DENSIDAD limit 10;

--Calles
select * from practica01.Ejes_viales_topologia limit 10;

--Puntos de incidentes viales
select * from practica01.Incidentes_viales_AXA_Uman limit 10;

```
Se pueden verificar las tablas que hay en las bases de datos


``` sql
--Verificar que la tabla existe en tu base de datos
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'practica01'; -- Reemplaza 'public' por el esquema en el que se encuentra la tabla si es diferente.

```
Para concatenar datos en una nueva columna

``` sql
SELECT
  CONCAT(cve_edo, cve_mun, cve_mun, cve_zona, cve_mnzn) AS cvegeo
FROM practica01.usos_de_suelo;

```

Cada tabla debe tener una llave primaria y de ser el caso un índice espacial 

-- TAREA: investiga qué son y para qué sirven los índices espaciales y de qué tipos hay

Primero crearemos los índices espaciales de todas las tablas con geometría en la base de datos. Toma de referencia la siguiente sentencia: 

``` sql
create index nombre_indice_gix on esquema.tabla1 using GIST(columna_geometría);
```

``` sql
create index usos_de_suelo_gix on practica01.usos_de_suelo using GIST(geom);
```
Creamos los constraints necesarios para agregar una llave primaria PK.

``` sql
ALTER TABLE practica01.mi_tabla ALTER COLUMN "mi_columna" SET NOT NULL;
ALTER TABLE practica01.mi_tabla ADD UNIQUE ("mi_columna");
ALTER TABLE practica01.mi_tabla ADD PRIMARY KEY ("mi_columna");

NOTA: Para que una columna sea PK, debe tener relación con otras tablas y no aceptar valores nulos ni repetidos. Si son tablas que no se unirán con otras no es necesario tener un PK, hasta contar con una relación en la BD. En caso de tener valores repetidos puedes recurrir a las siguientes consultas:  


 ```sql
select count(*), columna_identificador  from esquema.tabla  group by columna_identificador order by count(*) desc;
```
Para eliminar los duplicados corremos la siguiente consulta y creamos otra tabla

``` sql
create table practica01.usos_de_suelo_sd as
select DISTINCT ON (geom) * from practica01.usos_de_suelo
```

Para contar el número de celdas podemos utilizar:
``` sql
select count(*) from practica01.usos_de_suelo

select count(*) from practica01.usos_de_suelo_sd
```

# PODEMOS COMENZAR A HACER UNIONES

Las funciones JOIN unen las tablas ya sea a través de una clave unica o con ayuda de funciones espaciales. Estas uniones nos permiten seleecionar los datos que utilizaremos de cada tabla a través de la siguiente sintaxis general. 

``` sql
select tabla1.columna, tabla2.* 
from tabla1 a
join tabla2 b
on a.id_llave = b.id_llave 
``` 

NOTA: Cuanto usamos el ".*" despues del nombre de la tabla, le estamos diciendo que queremos todas las columnas. Las letras a y b despues de los tablas de union 
son apodos que nos permitirán no tener que escribir los nombres completos de las tablas. 

Con la siguiente consulta, traeremos de la tabla de USOS_DE_SUELO la columna de geometría y de la tabla de Ejes_viales_topologia todas las columnas. 

``` sql
SELECT *
FROM practica01.USOS_DE_SUELO AS U
INNER JOIN practica01.Ejes_viales_topologia AS E
ON ST_Intersects(U.geom, E.geom);
``` 



SELECT AddGeometryColumn ('practica01','tabla','geom',4326,'POINT',2);
UPDATE practica01.tabla SET geom = ST_SetSRID(ST_MakePoint(longitud::float, latitud::float), 4326);


# TIPOS DE DATOS

Cada columna de una tabla de la base de datos debe tener un nombre y un tipo de dato. Se debe decidir qué tipo de datos se almacenará dentro de cada columna basado en la información que se espere extraer. Vamos a revisar las columnas y su tipo de datos para determinar cuales necesitamos cambiar para contestar   preguntas de investigación como:

1) ¿En qué colonia hay mayor número de choques?
2) ¿Qué colonia tiene mayor número de áreas verdes? 



