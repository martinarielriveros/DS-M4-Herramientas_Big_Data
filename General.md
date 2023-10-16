## 1) HDFS

Crea toda la aquitectura (copias de imagenes, contenedores, network, volumnes, puertos, etc.) y arranca los contenedores

```
sudo docker-compose up
``` 
Lista la network creada con docker-compose up

```
sudo docker network ls
```
Inspecciona la network creada con los contenedores asociados (`dsm4herramientasbigdata_default`).

```
sudo docker network inspect
```

El namenode esta vacio. Se deben copiar los arhivos del Dataset del repositorio al directorio que se crea dentro del namenode

Ingresa al contenedor denominado `namenode` (creado por docker-compose)
en una sesion interactiva de bash
```
sudo docker exec -it namenode bash
``` 
crea el directorio de destino (en la guia esta mal, ya que se deben crear las carpetas correspondientes primero)

```
cd home
mkdir Datasets/DIRECTORIOS COINCIDENTES CON EL ARCHIVO Paso00.sh
```
sale de la sesion de linux del contenedor
```
exit
```
Ahora se deben copiar los archivos desde el respositorio clonado al destino recien (creado

Existe un archivo `.sh` sin permisos de ejecucion que se puede utlizar para hacerlo todo junto:

agrega permiso de ejecucion al archivo Paso00.sh

```
chmod +x Paso00.sh
```

Una vez copiados los archivos en el fileSystem de linux de namenode, se deben guardar en HDFS

Se crea un directorio llamado data
hdfs dfs -put /home/Datasets/* /data    --> se mueven los archivos desde el filesystem de linux del contenedor al HDFS
```
hdfs dfs -mkdir -p /data 
```
verificar efectivamente que se guardaron los 12 archivos.
```
hdfs dfs -ls /data
```
Ingresando en la ruta `http://192.168.0.101:9870/conf` (IP de la maquina virtual donde corren los contenedores) se pueden ver las siguientes caracteristicas (tamano de bloque y factor de replica)
```
<property>
<name>dfs.blocksize</name>
<value>134217728</value>          Tamano de bloque: 128 Mb
<final>false</final>
<source>hdfs-default.xml</source>
</property>
```
```
<property>
<name>dfs.replication</name>
<value>3</value>                  Factor de replica: 3 veces por archivo
<final>false</final>
<source>hdfs-default.xml</source>
</property>
```
##  2) HIVE

En este caso hay dos contenedores corriendo la misma imagen, exponiendo dos puertos diferentes, practica habitual para mantenimiento, escalabilidad y claridad.

```bde2020/hive:2.3.2-postgresql-metastore  0.0.0.0:10000->10000/tcp, :::10000->10000/tcp, 10002/tcp         hive-server```

```bde2020/hive:2.3.2-postgresql-metastore  10000/tcp, 0.0.0.0:9083->9083/tcp, :::9083->9083/tcp, 10002/tcp  hive-metastore```

copia el archivo de creacion de tablas hive Paso02.sql dentro del contenedeor de hive-server

```
sudo docker cp Paso02.hql hive-server:/
```

Se ingresa a la sesion interactiva dentro del contenedor "hive-server" para crear las tablas desde HIVE a partir de los csv


```
sudo docker exec -it hive-server bash
``` 

ejecuta el script de creacion (-f permite ejecutar en cadena dentro del hql)

```
hive -f Paso02.hql
```

Tengo un error en la ejecucion

```hdfs://namenode:9000/data/compra is not a directory or unable to create one```

Esto se debe a que en el Paso 1 cree los archivos sueltos dentro de data y no dentro de cada subcarpeta.
Para subsanar esto tengo dos caminos: cambio el hql o creo las carpetas. Voy por la opcion de crearlas porque puede ser mas engorroso despues.

Debo crear cada subcarpeta y mover el archivo creado dentro para todos los archivos creados.

Luego de hacer esto, se crearon las tablas correctamente.

se puede verificar la creacion correcta de las tablas del Paso02.hql mediante (se abre una linea de comandos de Hive dentro del contenedor):


```
hive
use integrador;
show tables;
```

##  3) Formatos de Almacenamiento

Copia el archivo de creacion de los formatos de almacenamiento Paso03.sql dentro del contenedor de `hive-server`

```
sudo docker cp Paso03.hql hive-server:/
```

ejecuta el script de creacion

```
hive -f Paso03.hql
``` 

##  4) Crear Indice sobre una tabla

Dentro del contenedor `hive-server` en una sescion CLI de Hive
```
use integrador2;

CREATE  INDEX index_venta_sucursal
        ON TABLE venta(IdSucursal) AS 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler'
        WITH DEFERRED REBUILD;
```






