# OWASP API Top Ten

En este articulo vamos asumir que tenemos un conocimiento general de lo que son REST API y vamos a implementar un laboratorio para probar las vulnerabilidades mas conocidas utilizando el OWASP para la demostración. Vamos a utilizar el repositorio  [Tiredful API](https://github.com/payatu/Tiredful-API) el cual es una API diseñada para hacer pruebas de seguridad y enseñar a los desarrolladores o profesionales de seguridad sobre las falles presentes en las API.

En el 2019 OWASP inició un esfuerzo por crear una versión de su Top 10 dedicada específicamente a la seguridad de API. La primera lista OWASP API Security Top 10 se publicó el 31 de diciembre de 2019, por lo que recomendamos leerse el repositorio del proyecto de [OWASP API Security Project](https://owasp.org/www-project-api-security/) 

Procederemos a mencionarlas pero no vamos a entrar en detalles las mismas son las siguientes:

- API1:2019 Broken Object Level Authorization
- API2:2019 Broken User Authentication
- API3:2019 Excessive Data Exposure
- API4:2019 Lack of Resources & Rate Limiting
- API5:2019 Broken Function Level Authorization
- API6:2019 Mass Assignment
- API7:2019 Security Misconfiguration
- API8:2019 Injection
- API9:2019 Improper Assets Management
- API10:2019 Insufficient Logging & Monitoring

### Requisitos

* [VPS de Preferencia Ubuntu Server 20.04](https://www.vultr.com/?ref=8403796-6G)
* Instalar Docker y Docker Compose
* Un subdominio para realizar las pruebas, no es necesario pero para una mejor estética utilizaremos http://apitest.toadsec.xyz

### Instalación API Tiredful

Vamos a proceder a clonar el repositorio

```shell
git clone https://github.com/payatu/Tiredful-API.git
```

![](img/1.png)

Usaremos docker para creación de la imagen y lo vamos a mapear el puerto 80 del host

```
docker build -t tiredful .
docker images
docker run -p 80:8000 --name tiredful -dit tiredful
docker exec -it tiredful sh
```

![](img/2.png)

Verificamos que el contenedor este en ejecución

![](img/3.png)

Y accedemos al Navegador para verificar que este arriba el API http://apitest.toadsec.xyz

![](img/4.png)

Si navegamos al botón de **Let's Start** podremos ver las indicaciones iniciales y unas pistas para armar la consulta

![](img/5.png)



### Pruebas OWASP Top TEN API

No abarcaremos toda las pruebas pero haremos la demostración de un par de las categorías que observamos

1. **API7:2019 Security Misconfiguration**

En esta categoría vamos a probar information disclosure, el reto nos muestra lo siguiente

![](img/6.png)

Vamos a probar la consulta y verificar el funcionamiento normal utilizando el comando **curl**

```shell
curl -X GET http://apitest.toadsec.xyz/api/v1/books/978-93-80658-74-2/
```

![](img/7.png)

Obtuvimos la respuesta información sobre el libro consultado, ahora vamos a probar provocando un error intencional, en lugar de usar el **ISBN 0-06-250217-4**, vamos a probar modificando con uno inexistente

![8](img/8.png)

Si podemos observar la imagen anterior provocamos un error donde nos esta revelando información que no debería ser conocida porque revela nombres de vistas, modelos etc.

2. API2:2019 Broken User Authentication

Vamos a realizar consultas directas a objetos

![](img/9.png)

Se  nos indica que debemos usar el método GET y el usuario que vamos a utilizar es el usuario llamado batman y trateremos de solicitar la información de otro usuario, primero vamos a verificar los ID que se muestran están en base64, procedemos a decodificarlos.

![](img/10.png)



Luego accederemos a la sección de usuarios y crearemos un token para el usuario batman con la información solicitada

![](img/11.png)

Con la información del token vamos a proceder a generar una consulta con autenticación básica

```
access_token:C88cEIM1a4kO8Zc1MSRnKOIX1bDZSV
token_type:Bearer
expires_in:36000
refresh_token:YXFO6IjegyiyE1APXEUKSZ0Gso865A
scope:read write groups
```

Vamos a construir la petición, ya en la imagen previa nos indican el tipo de autenticación en este caso **Authotization: Bearer <token>**

```shell
curl -v -X  GET -H "Authorization: Bearer C88cEIM1a4kO8Zc1MSRnKOIX1bDZSV" -H "Accept: application/json" -H "Content-Type: application/json" http://apitest.toadsec.xyz/api/v1/exams/MQ==/
```

Confirmamos que la petición esta correcta ahora

![](img/12.png)

Siguiente realizaremos la consulta con el mismo token pero consultando el usuario superman, recordemos que el id anterior eran los números 1 y 2 encodeados en base64, vamos a probar con el 56 que sería **NTY=** en base64

![](img/13.png)

3. API5:2019 Broken Function Level Authorization (Acmes Control)

El último que vamos a demostrar, el cual nos permitirá realizar funciones como un usuario admin

![](img/14.png)

Como en las otras ocasiones vamos a probar con una consulta realizada con un usuario en este caso batman y utilizando el método GET

```shell
curl -v -X  GET -H "Authorization: Bearer C88cEIM1a4kO8Zc1MSRnKOIX1bDZSV" -H "Accept: application/json" -H "Content-Type: application/json" http://apitest.toadsec.xyz/api/v1/articles/2/
```

![](img/15.png)

Podemos observar los métodos HTTP permitidos son DELETE, OPTIONS, GET y PATH, probaremos ahora tratando de aprobar un articulo 

```shell
curl -v -X  GET -H "Authorization: Bearer C88cEIM1a4kO8Zc1MSRnKOIX1bDZSV" -H "Accept: application/json" -H "Content-Type: application/json" http://apitest.toadsec.xyz/api/v1/approve-article/9/
```

![](img/16.png)

Arrojo un error indicando que no tenemos permisos, para continuar la prueba usaremos la herramienta POSTMAN para ver el error, solamente a manera de entender que las peticiones siguen siendo la misma solamente con una herramienta diferente.

![](img/17.png)

Debemos agregarle de igual manera el token y los headers utilizados.



![](img/18.png)

Hicimos una petición de borrado en vista que los métodos salía como permitido y observamos  nos indica que falta un header

![](img/19.png)

Vamos a probar borrando el articulo 

![](img/20.png)



Como observamos, logramos realizar el borrado del mismo y podemos consultar el articulo con el método GET el mismo nos debe arrojar un error o en este caso no encontrar la información

![](img/21.png)



En conclusión las apis en la mayoría de los casos su propia documentación puede ser una falla y más si esta accesible a todo el público, muchos desarrolladores piensan que es una especie de magia arcana o el uso del API de por si les trae seguridad, algunas APIS implementan seguridad básica pero no por eso no van a ser susceptibles de fallas de seguridad.

