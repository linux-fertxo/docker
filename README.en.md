<h1>
    <p align="center" width="100%">
        <img width="60%" src=".recursos/img/linuxfertxo.png">
        </br>
        My Docker services for Homelab
    </p> 
</h1>

[![Static Badge](https://img.shields.io/badge/lang-%F0%9F%87%AA%F0%9F%87%B8_es-blue?style=plastic)](README.md)

### A commented⁽¹⁾ selection of containers that I hope will inspire your projects
###### (1) Note: All comments inside each file are in Spanish at the moment.

#### Repository structure:

* Each service has its own `README.md`. Check out each one to better understand how they work.
* They can be run from CLI or be imported to Portainer, Dockge...
* Although they can be deployed at any time, it's interesting to deploy the following containers in this order because there are dependencies on each other:

      1. Socket Proxy
      3. Traefik 
      4. Authelia
      5. Crowdsec
      6. Any other container with Traefik labels (Nextcloud, etc)

#### Creators who have served me as inspiration and material in which I've relied

Today almost everything is invented. Each case is unique and you've to adapt all the info on Internet to your needs. I wouldn't have begun if it weren't for the countless gentle experts who we're lucky to have nowadays on Internet.Therefore,

### Infinite thanks to:

* **Timothy Stewart (Techno Tim)**
    * __[Website](https://technotim.live)__
    * __[YouTube](https://www.youtube.com/@technotim)__
* **Jay LaCroix (Learn Linux TV)**
    * __[Website](https://www.learnlinux.tv)__
    * __[YouTube](https://www.youtube.com/@LearnLinuxTV)__
* **Carlos (Computadoras y Sensores)**
    * __[YouTube](https://www.youtube.com/@ComputadorasySensores)__
    * __[Facebook](https://www.facebook.com/ComputadorasySensores)__
* **Anand (Smart Home Beginner)**
    * __[Website](https://www.smarthomebeginner.com)__
    * __[YouTube](https://www.youtube.com/@AnandsLab)__
* **Christopher Barnatt (Explaining Computers)**
    * __[Website](https://explainingcomputers.com)__
    * __[YouTube](https://www.youtube.com/@christopherbarnatt)__
* **The Linux de facto Bible**
    * __[Arch Linux Wiki (btw)](https://wiki.archlinux.org/title/Main_page)__ 😎
* **Online docs**
    * **[Docker](https://docs.docker.com)**
    * **[Traefik](https://doc.traefik.io/traefik/)**
    * **[Authelia](https://www.authelia.com/overview/prologue/introduction/)**

**To each and every developer whose containers I've used in my projects. There's a reference to their work within each `README.md`. Thank you all for your effort and willingness to share.**