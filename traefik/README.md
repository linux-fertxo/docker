<h1>
  <p align="center" width="100%">
    <img width="22%" src="../.recursos/img/traefik.png">
    </br></br>
    Træfik v3
  </p> 
</h1>

<h4> 
  <p align="center" width="100%">
    Un proxy inverso con certificados SSL para redirigir las peticiones entrantes a sus respectivos servicios mediante HTTPS
  </p>
  </br>
</h4>

[![Static Badge](https://img.shields.io/badge/lang-%F0%9F%87%AC%F0%9F%87%A7_en-blue?style=plastic)](README.en.md)

##### Basado en la imagen de [Traefik](https://traefik.io): [traefik](https://github.com/traefik/traefik)

- [Estructura](#estructura)
- [Explicación](#explicación)
  - [*Espera...¿Qué es un middleware?*](#esperaqué-es-un-middleware)
  - [*¿Y una chain?*](#y-una-chain)
  - [*Otras observaciones*](#otras-observaciones)
  - [*Variables de entorno*](#variables-de-entorno)
  - [*Antes de empezar*](#antes-de-empezar)
- [Arranque del contenedor](#arranque-del-contenedor)

#### Estructura

    traefik/
      ├─ docker-compose.yml               → archivo docker
      ├─ .env                             → variables de entorno
      ├─ traefik.yml                      → configuración estática
      ├─ acme/
      │    └─ acme.json                   → certificado SSL
      ├─ logs/
      │    ├─ access.log                  → registro de acceso
      │    └─ traefik.log                 → registro de apliación
      └─ rules/                           → configuración dinámica
           ├─ external.yml                → redirección a servicios no-docker
           ├─ middlewares.yml             → middlewares (ver más adelante)
           ├─ middlewares-chains.yml      → middlewares encadenados (ver más adelante)
           └─ tls-opts.yml                → opciones de TLS

#### Explicación

Los archivos `docker-compose.yml` y `.env` no necesitan presentación, son los archivos que contienen todas las instrucciones y variables para crear el contenedor de Traefik.

El archivo `traefik.yml` recoge la configuración de Traefik **_"estàtica"_**, esto es, la configuración que no cambia con el tiempo (o que cambia muy raramente).

La carpeta `acme/` que contiene el archivo `acme.json` es donde se almacenan los certificados SSL generados por Let's Encrypt. Este archivo tiene un tratamiento especial, ya que deberemos crearlo personalmente y asignarle permisos de lectura y escritura **sólo** al usuario que ejecuta el contenedor de Traefik, mediante `chmod 600 acme.json`

La carpeta `logs/` contiene los archivos de registro de acceso `access.log` y de la aplicación de Traefik `traefik.log`. Se puede funcionar sin ellos, pero es recomendable tenerlos a mano para depurar problemas o utilizarlos para que otros contenedores como por ejemplo [**crowdsec**](../crowdsec), busquen patrones de comportamiento sospechosos. Estos archivos también tendremos que crearlos personalmente, ya que Docker sólo es capaz de crear directorios si es necesario, pero no archivos.

Por último, la carpeta `rules/` contiene la configuración **_"dinámica"_** de Traefik. Dinámica en el sentido de que cada archivo que soltemos dentro será leído por Traefik y se aplicará a la configuración que en ellos se indica. Y dinámica en el sentido de que podemos crear nuevos servicios (o eliminarlos) con cierta asiduidad, como es muy probable que ocurra en el entorno de un Homelab. Hablemos primero de `external.yml` y `tls-opts.yml` que son los más sencillos de explicar.

  * `external.yml`: Contiene las reglas para redirigir a servicios que no son contenedores de Docker. Por ejemplo, si tenemos un servicio web en un servidor remoto, podemos indicarle a Traefik mediante éstas reglas a dónde tiene que ir cada petición. Este archivo en realidad es un ejemplo de cómo hacerlo, ya que podemos crear un archivo para cada servicio, uno para todos los servicios o una combinación de ambos.
  * `tls-opts.yml`: Contiene un conjunto de opciones de uso común de TLS.

De entre todos los archivos que hay dentro de `rules/` hay dos que son de especial importancia: `middlewares` y `middlewares-chains`. 

##### *Espera...¿Qué es un middleware?*

Los middlewares son trozos de código que se intercalan entre la petición y el servicio. Son una forma de "interceptar" las peticiones entrantes y realizar cambios en ellas, normalmente necesarios para que lo que le llegue al servicio sea entendible por éste y por tanto que funcione.

##### *¿Y una chain?*

Habitualmente los servicios hacen uso de los mismos middlewares una y otra vez. Una chain, o cadena en inglés, es una forma de agruparlos bajo un mismo nombre y así aplicarlos de manera conjunta a cada servicio. Dentro de las etiquetas de cada servicio podremos especificar la chain que queramos que se aplique.

##### *Otras observaciones*

Como hemos visto, el uso de la carpeta `rules/` permite soltar archivos dentro de ella y que Traefik los lea y aplique sobre la marcha. Esto no es así para cualquier subcarpeta que haya dentro, por lo que no es recursivo. Si nos gusta tener todo ordenado agrupado por carpetas en éste caso no funcionará.

El contenedor hace uso de [socket-proxy](../socket-proxy/) para mayor seguridad, pero contiene las líneas necesarias (comentadas) para funcionar sin él.

Utiliza [Cloudflare](cloudflare.com) como resolvedor DNS, pero es posible utilizar cualquier otro.

##### *Variables de entorno*

* `PUID` y `PGID` son los identificadores de usuario y grupo en formato numérico (ejecutar `id` para conocerlos)
* `TZ` es la zona horaria en formato `Continente/Ciudad`. [Listado de zonas](https://www.joda.org/joda-time/timezones.html)
* `DOCKERDIR` es el directorio que contiene todos los servicios de Docker.
* `DOMAINNAME` es el nombre de nuestro dominio.
* `CLOUDFLARE_EMAIL` es el correo con el que tengamos el dominio registrado en Cloudflare.

##### *Antes de empezar*

Crear la estructura arriba indicada, con la especial atención de los permisos de `acme.json`. **Si no son 600 (rw- --- ---) Traefik no arrancará.**

La red `proxy` debe estar presente antes de arrancar el contenedor de Traefik. Se puede comentar la línea `external: true` en `docker-compose.yml` y Traefik la creará automáticamente, pero esto tiene el inconveniente de que si el contenedor falla o no está en servicio, el resto de servicios que la tienen definida también caerán. Por eso es mejor crearla manualmente:

```bash
docker network create proxy
```

Si queremos usar el panel de Traefik desde el exterior, tendremos que crear un registro CNAME en nuestro DNS que apunte hacia él. En `docker-compose.yml` se indica que el panel estará disponible en `https://traefik.$DOMAINNAME`.

#### Arranque del contenedor

```bash
docker compose up -d     → arrancamos Traefik en segundo plano

docker logs traefik -f   → examinamos los registros para ver si hay algún problema (CTRL+c para salir)
```
</br>

¡Listo! Ya podemos ir a nuestros servicios por su nombre y con certificados SSL.