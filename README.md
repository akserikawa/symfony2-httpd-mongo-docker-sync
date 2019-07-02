# Docker environment for Manhattan development 
### With Apache, PHP 5, MongoDB and Composer.  


(02/07/2019) Work in progress... expect more updates and improvements!


## Docker setup

This repository provides a Docker environment with Apache, PHP5, MongoDB and Composer. Each service runs on its own container thanks to Docker Compose.

#### Requirements

* Install **Docker**.
* Install **Docker Compose**. (On desktop systems like Docker Desktop for Mac and Windows, Docker Compose is included as part of those desktop installs.)
* Install **docker-sync**. Developing with Docker under OSX/Windows is a huge pain, since sharing your code into containers will slow down the code-execution about 60 times (specially when working with frameworks like Symfony with lots of files). This tool lets you share/sync your code between your host machine and your Docker containers.
    * http://docker-sync.io/
    * https://docker-sync.readthedocs.io/en/latest/getting-started/installation.html

> Note: This repository includes docker-sync configuration for OSX. If you're running Windows, read the official docs and make the necessary changes.

#### Contents

* **`.docker-sync/`**  
    docker-sync daemon files and logs are stored here.
* **`bin/`**  
    Bash scripts and misc.
* **`build/`**  
    Dockerfiles for Apache and PHP5 services and configuration. 
* **`data/`**  
    Data files for MongoDB service
* **`web/`**  
    Our code lives here.
* **`docker-compose-dev.yaml`** An exact copy of `docker-compose.yaml`, but changing the volume of our code. We will use this file for development.
* **`docker-compose.yaml`** It defines which services we want to run and manages how our containers will function.
* **`docker-sync.yml`** It specifies a volume to sync with our host.
* **`Makefile`** Contains Make commands to automate our workflow (start/stop our containers, show logs, clear unused data,...).

#### Setup

*Note: make sure the ports 8000 and 9000 are available in your host machine.*
  
Follow the installation requirements above and then:

* Clone this repository.
* Run `make start_dev` and wait until you see a **success** message (it will take a while). This command does the following (in order):

      
      - **1.** Create/pull the images needed for our services (This happens only the first time, once the images are installed it jumps to the next step).
      - **2.** Create the Docker volume specified in `docker-sync.yml`.
      - **3.** Run Docker Compose to setup the containers.
      - **4.** Start docker-sync to sync our code. _Expected file sync times range from ~2 up to ~5 minutes (using native_osx sync strategy on a Macbook Pro)_.
      

* Run `docker ps` to list the containers and check they're up and running.
* Open **localhost:8000** in your browser :)
* Run `make stop_dev` to stop the containers, `make logs` to show Docker logs and `make clean` to delete unused data from Docker volumes and images, containers and networks.

## Manhattan setup

Manhattan is made of two applications:

- Manhattan Manager: a backoffice admin application to generate coupons/vouchers, create campaigns, products, articles, offers, and a wide variety of resources. Internally, this application makes requests to the **Manhattan API** and acts as a front-end for project managers to work with.

- Manhattan API: This is where the core business logic lives. It's an API (unavailable to the public) where every task from coupon generation/redemption up to the creation of administration reports are done. It provides integration with the APIs of our business partners (Amazon, Decathlon, Rakuten, Spotify, to name a few). 

Both applications are developed in Symfony2. Data is stored in MongoDB databases.

**Note: This environment comes without virtual hosts for Apache.** 

As mentioned above, our code runs in the **`web/`** directory.

* `cd web/`

* Run `docker ps` to list the containers. This command will output:  

| CONTAINER ID | IMAGE | COMMAND | CREATED | STATUS | PORTS | NAMES | 
|---	       |---	   |---	     |---	   |---	    |---    |---    |
| b46fbf87da19 | mnh-httpd-php5-mongo_php | "docker-php-entrypoi…" | 2 hours ago | Up 2 hours | 9000/tcp | php5
| ...          | ...   | ...     | ...     | ...    | ...   | ...   |

* Since the php5 image comes with git and composer out of the box, copy the container ID and clone the repositories:
    - Run `docker exec -it <php5_container_id> git clone <manhattan-api-repo-url> symfony/`
    - Run `docker exec -it <php5_container_id> git clone <manhattan-manager-repo-url> manager/`

* Once they get cloned, navigate to each directory and run:
    - `cd symfony`
    - `docker exec -it <php5_container_id> composer install` to install all the dependencies.
    - `docker exec -it <php5_container_id> php app/console assets:install --symlink web`

* The following commands are not strictly required, but recommended:
    - `docker exec -it <php5_container_id> chmod -R 777 app/cache`
    - `docker exec -it <php5_container_id> chmod -R 777 app/logs`

Now that dependencies are installed, some extra configuration is needed:

#### MongoDB connection 

**Manhattan API:**  

- Set "mongo" as your database host in the server url (after the **@**). This host points to your mongodb service declared in `docker-compose.yml`.


#### `app/config/config_dev.yml`




```yaml


doctrine_mongodb:
    ...
    connections:
        default:
            server: mongodb://mongo:27017
            options:
                ...
        billing:
            server: mongodb://mongo:27017
```

### **Important note**: some passwords/keys are hardcoded in the config parameters. Be sure to run Symfony in the "dev" environment (using the **app_dev.php** front controller) and replace these values with yours.


**Manhattan Manager:**

- Again, set "mongo" as your database host in the server url for doctrine_mongodb. 

#### `app/config/config_dev.yml`

  
```
    
doctrine_mongodb:
    ...
    connections:
        default:
            server: mongodb://%db_user%:%db_password%@mongo:27017/manager
```

- Run `docker inspect <php5_container_id> | grep IPAddress` to get your current Docker network IP address. Set this value in the `api_url` parameter.

#### `app/config/parameters.yml`


```yaml

parameters:
    ...
    api_url: http://%your_docker_network_ip_address%:8000/symfony/web/app_dev.php
```
    
Now the Manager will be able to make requests to the Manhattan API. It's important to know this IP address changes everytime the network gets deleted (when running `make clean` for example).

## Help notes and summary

If anything starts to fail or doesn't work as expected, it probably has to do with the Symfony cache and logs. Run:

- `docker exec -it <php5_container_id> php app/console cache:clear --no-warmup`
- `docker exec -it <php5_container_id> chmod -R 777 app/cache`
- `docker exec -it <php5_container_id> chmod -R 777 app/logs`

If this commands don't fix the situation,run `make stop_dev` and `make start_dev` and check the containers are up and running with `docker ps`.

As a last resort, try running `make clear` and start the Docker workflow explained above. Don't worry, the code won't be affected or deleted.

If you've gotten this far, congrats! Hopefully this guide has given you some knowledge. Btw, contributions are welcome! Feel free to add improvements and submit them via pull request.

Ask me anything!  
aserikawa@chequemotiva.com
    