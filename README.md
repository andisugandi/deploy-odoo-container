# Deploying Containerized Odoo Web Apps

Let's deploy [Odoo](https://github.com/odoo/odoo) web apps by using [Docker Engine](https://docs.docker.com/engine/) ([Container](https://www.docker.com/resources/what-container/)), or any other tools you choose (like [Podman](https://podman.io/)).

Disclaimer: This tutorial is for development purposes, not for the production environment.

## Installing Container Tool (Docker)

I believe the tutorial described [here](https://docs.docker.com/engine/install/) is sufficient to follow. Just make sure that you are installing the latest version of Docker Engine and can run it [without the need to use root privilege](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user).

See if your Docker Engine and Docker Compose are up and running:

~~~bash
docker --version
~~~

The output: `Docker version 24.0.7-ce, build 311b9ff0aa93`.

~~~bash
docker-compose --version
~~~

The output: `Docker Compose version 2.23.3`

The version number may vary depending on the version you install and the operating system you are currently using.

## Preparing

Let's create directories where Odoo will reside.

~~~bash
export ODOO_PATH=/mnt/odoo
~~~

Please choose your own preferred `ODOO_PATH` directory.

~~~bash
sudo mkdir $ODOO_PATH/{odoo-web-data,odoo-extra-addons,postgresql}
sudo chown 101:101 -R $ODOO_PATH/{odoo-web-data,odoo-extra-addons}
sudo chown 999:root -R $ODOO_PATH/postgresql
~~~

## Run The Odoo (Container) Image

1. Let's run the chosen Odoo container image (I am using `odoo:16` in this tutorial, you will probably choose the latest version: `odoo:17` as of [November 2023](https://www.odoo.com/odoo-17-release-notes)).

    ~~~bash
    cd $ODOO_PATH
    ~~~

2. Create `docker-compose.yml` file (It using `vim`, please choose your preferred text editor):

    ~~~bash
    sudo vim docker-compose.yml
    ~~~

3. Then fill in the following script to the file.

    ~~~yaml
    version: '3.9'

    services:
      db:
        image: 'docker.io/library/postgres:16'
        restart: always
        environment:
          - POSTGRES_DB=postgres
          - POSTGRES_USER=odoo
          - POSTGRES_PASSWORD=please_choose_your_own_super_secret_password
          - POSTGRESQL_ADMIN_PASSWORD=please_choose_your_own_super_secret_password
          - PGDATA=/var/lib/postgresql/data/pgdata
        networks:
          - odoo16network
        volumes:
          - '/mnt/odoo/postgresql:/var/lib/postgresql/data'

      odoo16:
        depends_on:
          - db
        image: 'docker.io/library/odoo:16'
        environment:
          -  HOST=db
          -  PORT=5432
          -  USER=odoo
          -  PASSWORD=please_choose_your_own_super_secret_password
        restart: always
        volumes:
          - '/mnt/odoo/odoo-extra-addons:/mnt/extra-addons'
          - '/mnt/odoo/odoo-web-data:/var/lib/odoo'
        ports:
          - '8069:8069'
        networks:
          - odoo16network
        
    networks:
      odoo16network:
    ~~~

    Save it and quit `vim`.

4. Execute the file by using Docker Compose:

    ~~~bash
    docker-compose up -d
    ~~~

5. See if the services are up and running:

    ~~~bash
    docker-compose ps --format "table {{.Name}}\t{{.Image}}\t{{.Service}}\t{{.Status}}\t{{.Ports}}"
    ~~~

    The output will looks like:

    ~~~bash
    NAME            IMAGE                           SERVICE   STATUS          PORTS
    odoo-db-1       docker.io/library/postgres:16   db        Up 13 minutes   5432/tcp
    odoo-odoo16-1   docker.io/library/odoo:16       odoo16    Up 13 minutes   0.0.0.0:8069->8069/tcp, :::8069->8069/tcp, 8071-8072/tcp
    ~~~

6. Follow the final installation step by accesing the web ui at: `http://YOU_IP_ADDRESS:8069`.

## Login with The Credentials

Finally, login and enjoy the Odoo running via these simple steps.
