<h1>
    <p align="center" width="100%">
        <img width="50%" src=".recursos/img/linuxfertxo.png">
        </br>
        Mis servicios Docker para Homelab
    </p> 
</h1>

[![Static Badge](https://img.shields.io/badge/lang-%F0%9F%87%AC%F0%9F%87%A7_en-blue?style=plastic)](README.en.md)

### Una selección comentada de contenedores que espero sirva de inspiración para vuestros proyectos

#### Estructura del repositorio:

* Cada servicio tiene dentro su propio `README.md`, con ejemplos y archivos auxiliares así como comentarios donde he pensado que era relevante. Echa un vistazo a cada uno para entender mejor cómo funcionan.
* Se pueden hacer funcionar tanto desde CLI como importándolos en Portainer, Dockge...
* Aunque se pueden desplegar en cualquier momento, los siguientes contenedores es interesante hacerlo en este orden por tener dependencias unos de otros:

      1. Socket Proxy
      2. Traefik 
      3. Authelia
      4. Crowdsec
      5. El resto de contenedores con etiquetas para Traefik (Nextcloud, etc)

#### Creadores que me han servido de inspiración y material en el que me he apoyado

Hoy en día casi todo está inventado. Cada caso es particular y hay que adaptar toda la información que hay en la red a nuestras necesidades. Todo ésto no habría empezado si no fuera por los numerosos expertos que tenemos la suerte de tener hoy en día en la red. Por lo tanto,

### Gracias infinitas a:

* **Timothy Stewart (Techno Tim)**
    * __[Página web](https://technotim.live)__
    * __[YouTube](https://www.youtube.com/@technotim)__
* **Jay LaCroix (Learn Linux TV)**
    * __[Página web](https://www.learnlinux.tv)__
    * __[YouTube](https://www.youtube.com/@LearnLinuxTV)__
* **Carlos (Computadoras y Sensores)**
    * __[YouTube](https://www.youtube.com/@ComputadorasySensores)__
    * __[Facebook](https://www.facebook.com/ComputadorasySensores)__
* **Anand (Smart Home Beginner)**
    * __[Página web](https://www.smarthomebeginner.com)__
    * __[YouTube](https://www.youtube.com/@AnandsLab)__
* **Christopher Barnatt (Explaining Computers)**
    * __[Página web](https://explainingcomputers.com)__
    * __[YouTube](https://www.youtube.com/@christopherbarnatt)__
* **La Biblia de Linux**
    * __[Arch Linux Wiki (btw)](https://wiki.archlinux.org/title/Main_page)__ 😎
* **Documentación online**
    * **[Docker](https://docs.docker.com)**
    * **[Traefik](https://doc.traefik.io/traefik/)**
    * **[Authelia](https://www.authelia.com/overview/prologue/introduction/)**

**A todos y cada uno de los programadores cuyos contenedores he utilizado en mis proyectos. Hay una referencia a su trabajo dentro de cada `README.md`. Gracias por vuestro esfuerzo y a vuestra voluntad de compartir.**