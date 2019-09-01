# Docker y Docker Compose
Empezamos repasando algunos componentes de docker, y de como los desarrolladores empezamos a usar docker como  una herramienta del día a día entre los componentes importantes basicos para un dev son:

- Dockerfile
- Imagenes
- Contenedores

Pero para entrar en producción, empezamos a ver los siguientes componenetes:

- volumenes
- network
- docker compose
- docker swarm.

## Caso de Estudio
El caso de estudio fue levantar una pagina con docker compose, tomando en cuenta:
- las ultimas versiones de las imagenes de wordpress, y mysql
- El acceso a la bd por parte del administrador de la pagina
- Poder cambiar las configuraciones de php del contenedor de wordpress
- Poder cargar plugins de wordpress sin la necesidad de ingresar en el contenedor

### Docker compose file

Para poder cumplir estas caracteristicas existieron algunos problemas

worpres tiene tipo de conexion incompatible con la version 8 de mysql, fue necesaria la siguiente configuracion

```
command: '--default-authentication-plugin=mysql_native_password'

```

y se configuro un volumen para cuidar el almacenamiento de la base de datos

```
volumes:
      - ./my-db:/var/lib/mysql
```

Para poder configurar las propiedades de php, como ser uso de memoria, tamano de upload necesario podemos
configurarlas desde el volumen
```
- ./uploads.ini:/usr/local/etc/php/conf.d/uploads.ini
```

y para el tema de plugins

```
- ./wp-content:/var/www/html/wp-content
```

Para administrar la base de datos usamos la imagen del phpmyadmin

el archivo completo se puede ver en [link](...)

### Ejecucion

Para ejecutar el docker compose file, necesitamos tener instalado
- docker
- docker-compose
- usuario actual en el grupo docker

Para la ejecucion del Docker compose file usamos el siguiente comando

```
docker-compose up
```
El cual lo ejecuta los contenedores de manera iterativa, ahi observamos que no se levantan en los mismos tiempos y que gracias a la configuracion restart always, logra levantar correctamente.

Al ser de manera iterativa, cuando interrumpimos el proceso mediante el ctrl-c, los contenedores se detienen pero siguen ahi y podemos verlos con un:

```
docker ps -a
```

Para que se ejecute de manera desatachada ejecutamos

```
docker-compose up -d
```
y podemos administrar cada contenedor de forma individual tambien.

Entre otras características de docker compose, estepermite contruir imagenes desde docker files, usar variables de entornos como variables.

### Despliegue
En este caso wordpress esta publicado en el puerto 8080 y para que pueda ser accedido usamos un proxy reverso
en este caso nginx. Para el php-myadmin usaremos el proxy reverso y un path, asi evitar usar subdominios
```
	 location / {
   		proxy_set_header Host $host;
    		proxy_set_header Accept-Encoding "";
    		proxy_pass http://localhost:8080;
  	}
   	location /bd/ {
   		proxy_set_header Host $host;
    		proxy_set_header Accept-Encoding "";
    		proxy_pass http://localhost:4444;
  	}
```
Si hay otros servicios, estos igual saldran mediante path's

### Recomendaciones
Luego de ver este ejemplo, hicimos lluvia de ideas acerca de como mejorarlo, entre las ideas estaban:

- Que reenvie las cabeceras en el proxy reverso
- Adicionar certificados al proxy
- Hacer health-check en los contenedores docker
- Que el proxy reverso tambien funcione dentro de un contenedor
- Como se esta corriendo el ejemplo en una VPS revisar que tenga un firewall y que solo tenga los puertos 22 y 80 abiertos
- Un ejemplo interesante seria usar docker swarm
- tener cuidado con el time zone de los contenedores
- en el caso de imagenes basadas en alpine tener cuidado con los archivos temporales
