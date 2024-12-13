<h1>
  <p align="center" width="100%">
    <img width="33%" src="../.recursos/img/logos/crowdsec.png">
    </br>
    Crowdsec
  </p> 
</h1>

<h2> 
  <p align="center" width="100%">
Crowdsec is a system that detects and blocks unwanted access attempts at various levels by analyzing logs, comparing them with known threat patterns called scenarios and sharing the results among all users.</p>
</h2>

<h3> 
  <p align="left" width="100%">
    Based on the official <a href="https://crowdsec.net">Crowdsec container</a> and <a href="https://github.com/maxlerebourg">maxlerebourg's</a> bouncer.
</h3>

[![Static Badge](https://img.shields.io/badge/lang-%F0%9F%87%AA%F0%9F%87%B8_es-blue?style=plastic)](README.es.md)

<h3>
  Content:
</h3>

- [Structure](#structure)
- [Explanation](#explanation)
  - [*How does Crowdsec work?*](#how-does-crowdsec-work)
  - [*Files*](#files)
  - [*Traefik's modifications*](#traefiks-modifications)
  - [*Collections*](#collections)
  - [*Sorry... The command `what`?*](#sorry-the-command-what)
  - [*Environment variables*](#environment-variables)
  - [*Before starting*](#before-starting)
- [First run and initial configuration](#first-run-and-initial-configuration)
- [EXTRA - Post installation](#extra---post-installation)

## Structure

    crowdsec/
      ├─ docker-compose.yml               → docker file
      ├─ .env                             → environment variables
      ├─ cs-private-key                   → token for the bouncer (see later)
      └─ acquis.d/                        → log acquisition (see later)
           ├─ traefik.yaml                → Traefik service configuration
           └─ other services...

## Explanation

### *How does Crowdsec work?*
</br>
<figure>
  <p align="center" width="100%">
    <img width="75%" src="../.recursos/img/crowdsec/simplified_SE_overview.svg"/>
    <figcaption align="center"><i>Crowdsec's workflow (img © Crowdsec)</i></figcaption>
  </p>
</figure>

Crowdsec is divided into several components. Although the explanation of each one [is much more technical](https://docs.crowdsec.net/docs/intro), to simplify we will say that it is divided into:

* **Data Sources**: These are the log files of our services, or HTTP requests from our applications. Crowdsec needs to read them to find out if there is any suspicious behavior.
* **Security Engine**: This is the component in charge of orchestrating all the parts of Crowdsec. It is made up of:
  * **Local API (LAPI)**: This is an interface that sends alerts and decisions to a database. Once there, it allows the *bouncers* to use them.
  * **Parsers**: They are responsible for reading a specific log and "understanding" what is written in them. There's practically a parser for each type of service/application.
  * **Scenarios**: When the parser finds a known suspicious pattern, it's compared with the scenarios we have installed. If there's a match, an alert is sent to the local API.
  * **Decisions**: They are the consequence that is triggered when a certain number of alerts is received.
* **Remediation components**: Commonly known as *"bouncers"*, they are responsible for executing the decisions.
* **Central API (CAPI)**: The interface on Crowdsec's servers that centralizes data from "bad actors" and distributes it among its users. In addition we will have access to the...
* **Console**: From here we can manage our machines, block lists and alerts among many other features through the online platform.
  
What makes Crowdsec so powerful is that it **leverages the community to share data about "bad actors"** among its users. This way, we can **prevent an attack before it even happens** and bypass the entire process described above.

### *Files*

As usual, both `docker-compose.yml` and `.env` need no introduction. They are the files that contain all the instructions and variables to create the Crowdsec container.

The `cs-private-key` file contains the *token* for the bouncer. This random code is required to link the Traefik plugin to this service. It's empty for the moment because we need to generate it once we have started the container.

>**THIS METHOD IS NOW DEPRECATED, BUT I'LL KEEP THE EXPLANATION AS IT'S ALMOST THE SAME BUT CHANGING ONE CENTRALIZED FILE FOR SEVERAL FILES PLUS A FOLDER**
The file that defines where are the logs and of which type they are is `acquis.yaml` **(note the extension `.yaml` and not `.yml`)**. Here we tell Crowdsec which services' logs we want to be analyzed. They must be previously mounted as volume in `docker-compose.yml`, so they can be read from within the container.
The structure of `acquis.yaml` is divided in a `filenames` section with the logs to be analyzed (one or more files) and a `labels` section, where we specify what type of data is expected in them, so the Security Engine can send them to the appropriate parser.

Instead of `acquis.yaml`, the `acquis.d/` folder is now used. Inside we create a file for each service we want to parse. These files' structure is exactly the same as explained above, but for one single service.

>**Note**: **In most cases, passing only the Traefik logs is more than enough**. We can still pass the Nginx, Apache, Nextcloud, etc. logs to improve detection. But this has a computational cost, so we have to take into account the increase in CPU and memory usage. Some examples (commented out) of services seen here are already in `docker-compose.yml` and `acquis.d/`

**In brief:**

* We write the name of one or more files that we want to be analyzed by the Security Engine.

* Then we indicate to which service or application they belong in order to pass them to the appropriate *parser*.

### *Traefik's modifications*

Some modifications need to be made to Traefik so that it works in coordination with Crowdsec. In this repository you can find [a section dedicated to Traefik](../traefik/). In that example, both `docker-compose.yml`, `traefik.yml`, `middlewares.yml` and `middlewares-chains.yml` have all the necessary lines, but commented out. You just have to uncomment them and restart Traefik to take effect.

In other existing guides I've hit on Internet, it's common to find Crowdsec service together with [Traefik-Crowdsec-Bouncer](https://github.com/fbonalair/traefik-crowdsec-bouncer) by [Fabien Bonalair](https://github .com/fbonalair), as an additional service within `docker-compose.yml`. However, at the time of writing this guide, this container has not been updated for 2+ years and since then there are new features (such as support for AppSec), so in this case we will use the [Crowdsec-Bouncer-Traefik-Plugin](https://github.com/maxlerebourg/crowdsec-bouncer-traefik-plugin) plugin by [maxlerebourg](https://github.com/maxlerebourg). **From here, my most sincere thanks to both of them for their work.**

The most notable difference between the two *bouncers* is that the plugin is defined inside Traefik's static configuration file (`traefik.yml`) and not as another service in `docker-compose.yml`, so there's no need to spin up any additional containers alongside Crowdsec.

>If we are also users of **Cloudflare proxy** (the famous orange cloud), we also take the opportunity to introduce [Real IP from Cloudflare Proxy/Tunnel](https://plugins.traefik.io/plugins/62e97498e2bf06d4675b9443/real-ip-from-cloudflare-proxy-tunnel) *plugin* from [BetterCorp](https://github.com/BetterCorp) **(also thanks to BetterCorp from here)**, which allows us to pass the real IP from the requester.
>Actually this is **not necessary for Crowdsec**, since Traefik is able to write the real IP in its access log, but **other services that we use** later (and that do not have to access log files) **will indeed need it**. Also, it doesn't hurt anyone, so we leave it on. 😎👍

In `traefik.yml` uncomment the following block:

```yaml
experimental:
  plugins:
    cloudflarewarp:
      moduleName: 'github.com/BetterCorp/cloudflarewarp'
      version: 'v1.3.3'
# check the most updated version at https://plugins.traefik.io/plugins/62e97498e2bf06d4675b9443/real-ip-from-cloudflare-proxy-tunnel
    crowdsec-bouncer:
      moduleName: 'github.com/maxlerebourg/crowdsec-bouncer-traefik-plugin'
      version: 'v1.3.5'
# check the most updated version at https://plugins.traefik.io/plugins/6335346ca4caa9ddeffda116/crowdsec-bouncer-traefik-plugin
```
>

And in `middlewares.yml` uncomment the following:
```yaml
      middlewares-crowdsec-bouncer:
        plugin:
          crowdsec-bouncer:
            enabled: true
            logLevel: INFO
            updateIntervalSeconds: 60
            updateMaxFailure: 0
            defaultDecisionSeconds: 60
            httpTimeoutSeconds: 10
            crowdsecMode: live
            crowdsecLapiKeyFile: /etc/traefik/cs-private-key
            crowdsecLapiHost: crowdsec:8080
            crowdsecLapiScheme: http
            forwardedHeadersCustomName: X-Custom-Header
              clientTrustedIPs:
                - "192.168.0.0/24"
                - '172.16.0.0/10'
```
>

>We can adjust each parameter to our needs (there are many more, see the Crowdsec documentation). The last three lines allow trusting the two shown example local networks. They are completely optional.

In `middlewares-chains.yml` uncomment the line corresponding to Crowdsec for each block. They are the sigle lines commented out, you can't miss them.

Finally, **totally optional but highly recommended**, is to configure the Traefik container with a **fixed IP**. This is because every time we restart and Docker assigns Traefik a different IP than any previous one, Crowdsec will generate a new bouncer (with the same name and the @IP as a suffix) and then an unreal number of items will be shown in the web console. I recognize that this change is more aesthetic than practical, since it will still fulfill its purpose, but I like to have everything tidy. 😎

To do this, just uncomment the corresponding lines in `docker-compose.yml` (lines 7-10,26,27) **and adjust both the range of the networks block and the IP of the traefik service to the ones we have in our environment.**

### *Collections*

All this theory is very interesting, but we have so many terms that this give the impression of being a very complicated thing to set up. With parsers, scenarios, bouncers and so on, it's normal that everything seems very confusing.

**But here comes another concept to our rescue 🤣 : [collections](https://app.crowdsec.net/hub/collections)**. Fortunately these will make us forget all the other terms, at least in the practical part. Let me explain:

> ***A collection is the set of parsers, scenarios and the way in which they are all work coherently for a specific use case.***

This way, instead of having to worry about the details of each component, we can download a **"pack"**. Each pack takes care of everything necessary to analyze and make decisions against unwanted access attempts on the application it's aimed at: Wordpress, Nextcloud, Windows operating system... What's more, we can download as many collections as we want in the same Security Engine **always within a logic**. It makes no sense to install a collection to protect the admin panel of a Mikrotik router if we have a one with OPNsense.

In the current case, and to be brief, we'll download the basic collections for the Linux system, some common for http services and Traefik. We can do this either as an environment variable in `docker-compose.yml` (the minimal ones are already selected) or by running **the `cscli` command** once the container is up and running.

### *Sorry... The command `what`?*

The most significant command in Crowdsec is `cscli` (CrowdSec Command Line Interface). It is our way of interacting with it locally. It has a large number of options that we won't cover here, so we'll focus on the most common ones:

  * `cscli bouncers`   : To list, add or remove bouncers.
  * `cscli collections`: Same as above but for collections.
  * `cscli console`    : For enrolling web console, enable/disable or watch its status.
  * `cscli decisions`  : To see decissions taken or add/del them manually.
  * `cscli metrics`    : Quick glance at all Crowdsec data.

Since this command "lives" inside the container, we will have to see how to work with it. The most common methods are:

* **From outside the container, if we only want to run one single command**
```bash
docker exec <container> <command>
```
>

>Where `<container>` is the name of the container and `<command>` is what we want to be executed inside.

* **From inside the container, if we have to execute multiple tasks**
```bash
docker exec -it <container> /bin/sh
# (in some cases /bin/bash can also be used)
```
>

>This way we get "inside" the container (`-it` stands for ***interactive***) and then we can run all the commands we want. Think in this method as a "ssh" of sorts.

### *Environment variables*

Only one:

* `DOCKERDIR` as always, the root folder of all our Docker services.

### *Before starting*

* If we are going to use the other log files (Nextcloud, Authelia...), check that they are mounted in the `volumes:` section of `docker-compose.yml`, and that they have a dedicated file in `acquis.d/`
* We have uncommented all the necessary lines in `docker-compose.yml`, `traefik.yml`, `middlewares.yml` and `middlewares-chains.yml` of the [Traefik](../traefik/) service. **Restart the container once done.**

## First run and initial configuration

```bash
docker compose up -d       → run Crowdsec in dettached mode

docker logs crowdsec -f    → look at the logs to find any issues (CTRL+c para salir)
```

Allow a few moments to let the container start completely, and then run the following command to generate the Token:

**Important!**: the token will only be displayed **ONCE**, if we don't take it **we will have to delete the bouncer and start over!**

```bash
docker exec crowdsec cscli bouncers add traefik-bouncer
```
>
A random code will be displayed on the screen. Copy that code and paste it in `cs-private-key` file. Now restart both services:

```bash
docker restart traefik crowdsec
```
>

Again, allow a few minutes until all is properly restarted. In the `crowdsec` log we'll start to see interesting things:

    ..."loading acquisition file : /etc/crowdsec/acquis.d/traefik.yaml"
    ..."Adding file /var/log/traefik.log to datasources" type=file
>

We can check if it has recognized the bouncer (it's possible that not all the followinf information appear during the first few minutes):
```bash
docker exec crowdsec cscli bouncers list
----------------------------------------------------------------------------------------------------------------
 Name             IP Address   Valid  Last API pull         Type                             Version  Auth Type 
----------------------------------------------------------------------------------------------------------------
 traefik-bouncer  10.0.0.10    ✔      2024-12-12T19:29:24Z  Crowdsec-Bouncer-Traefik-Plugin  1.X.X    api-key   
----------------------------------------------------------------------------------------------------------------
```
>

Let's add an example collection for one of the services discussed throughout the document, ***Nextcloud***
```bash
docker exec crowdsec cscli collections install nextcloud
```
>


Finally, go to the Crowdsec console (https://app.crowdsec.net), click on the ***Sign up for free*** button and create an account.
In ***Security Engines → Engines*** section, click on ***Add Security Engine***.
Two blocks of info will be shown: the first one with links to configure a Security Engine according to the Operating System (we have already done this), and the second one where it says: ***Enroll your CrowdSec Security Engine:***

    sudo cscli console enroll -e context blahblahblahblahblahblah

Copy the command and paste it into the terminal. It will respond something like

```bash
INFO[2024-12-12T19:50:17Z] manual set to true
INFO[2024-12-12T19:50:17Z] Enabled manual : Forward manual decisions to the console
INFO[2024-12-12T19:50:17Z] Enabled tainted : Forward alerts from tainted scenarios to the console
INFO[2024-12-12T19:50:17Z] Watcher successfully enrolled. Visit https://app.crowdsec.net to accept it.
INFO[2024-12-12T19:50:17Z] Please restart crowdsec after accepting the enrollment.
```
>

We are asked to accept the request in [the Crowdsec console](https://app.crowdsec.net). Click on ***Accept enroll*** and finally restart Crowdsec one last time.

<h3>
Done! We now have an extra layer of security in our services supported by a great community.
</h3>

## EXTRA - Post installation

Crowdsec offers remarkable protection with the free account. To get the most out of it, you can subscribe to up to three additional lists from the ***Blocklists*** section: filter by ***Free***, choose the one you like best, and inside there is a ***Security Engines*** section. Here you can add your newly created Security Engine, being mandatory to choose what behavior we want the bouncer to have in the event of a positive entry from the blocklist: Ban, present a Captcha or custom (advanced).

We can perform this operation with up to two more blocklists. The three lists we choose are associated with the account and not the Engine, so if we register more Engines they will have to use the same three lists. Although they can be changed at any time, ideally, if we are going to make intensive use we should consider acquiring a subscription. It has many benefits other than the quantity and quality of the block lists.

***Furthermore, if the central server (CAPI) receives our data periodically, Crowdsec will assign us a fourth list, Crowdsec Community Blocklist.***

And that's it for today. See you at the next service!