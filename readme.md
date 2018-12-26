TLC-Brazil Website
===========================================

This repository is the base for the **IBM Technical Leadership Council Brazil (TLC-BR)** website. In order to facilitate the deployment,'ve used [Wagtail CMS](https://github.com/wagtail/wagtail) and IBM Cloud (CIO) to host the application.

Access to this website is currently restricted to IBM'ers only, through the following link:

-  https://tlcbr.w3ibm.mybluemix.net

Setup on your local machine
------------------------------------

You can setup a local development environment on your own workstation using Vagrant, Docker or Python Virtualenv, as described on the sections below:

### Setup with Vagrant

#### Dependencies
* [Vagrant](https://www.vagrantup.com/)
* [Virtualbox](https://www.virtualbox.org/)

#### Installation

Once you've installed the necessary dependencies run the following commands:

```bash
git clone git@github.com:fsilveir/tlc-brazil.git
cd tlc-brazil
vagrant up
vagrant ssh
# then, within the SSH session:
./manage.py runserver 0.0.0.0:8000
```

The site will now be accessible at [http://localhost:8000/](http://localhost:8000/) and the Wagtail admin
interface at [http://localhost:8000/admin/](http://localhost:8000/admin/).

Log into the admin with the credentials ``admin / changeme``.

Use `Ctrl+c` to stop the local server. To stop the Vagrant environment, run `exit` then `vagrant halt`.

### Setup with Docker

#### Dependencies
* [Docker](https://docs.docker.com/engine/installation/)
* [Docker Compose](https://docs.docker.com/compose/install/)

#### Installation
Run the following commands:

```bash
git clone git@github.com:fsilveir/tlc-brazil.git
cd tlc-brazil
docker-compose up --build -d
docker-compose run app /venv/bin/python manage.py load_initial_data
docker-compose up
```

The site will now be accessible at [http://localhost:8000/](http://localhost:8000/) and the Wagtail admin
interface at [http://localhost:8000/admin/](http://localhost:8000/admin/).

Log into the admin with the credentials ``admin / changeme``.

**Important:** This `docker-compose.yml` is configured for local testing only, and is _not_ intended for production use.

#### Debugging
To tail the logs from the Docker containers in realtime, run:

```bash
docker-compose logs -f
```

### Setup with Virtualenv

You can run the site locally without setting up Vagrant or Docker and simply use Virtualenv, which is the [recommended installation approach](https://docs.djangoproject.com/en/1.10/topics/install/#install-the-django-code) for Django itself.

#### Dependencies
* Python 3.4, 3.5 or 3.6
* [Virtualenv](https://virtualenv.pypa.io/en/stable/installation/)
* [VirtualenvWrapper](https://virtualenvwrapper.readthedocs.io/en/latest/install.html) (optional)

### Installation

With [PIP](https://github.com/pypa/pip) and [virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/)
installed, run:

    mkvirtualenv tlc-brazil
    python --version

Confirm that this is showing a compatible version of Python 3.x. If not, and you have multiple versions of Python installed on your system, you may need to specify the appropriate version when creating the virtualenv:

    deactivate
    rmvirtualenv tlc-brazil
    mkvirtualenv tlc-brazil --python=python3.6
    python --version

Now we're ready to set up the project itself:

    cd ~/dev [or your preferred dev directory]
    git clone git@github.com:fsilveir/tlc-brazil.git
    cd tlc-brazil
    pip install -r requirements/base.txt

Next, we'll set up our local environment variables. We use [django-dotenv](https://github.com/jpadilla/django-dotenv)
to help with this. It reads environment variables located in a file name `.env` in the top level directory of the project. The only variable we need to start is `DJANGO_SETTINGS_MODULE`:

    $ cp tlcbr/settings/local.py.example tlcbr/settings/local.py
    $ echo "DJANGO_SETTINGS_MODULE=tlcbr.settings.local" > .env

To set up your database and load initial data, run the following commands:

    ./manage.py migrate
    ./manage.py load_initial_data
    ./manage.py runserver

Log into the admin with the credentials ``admin / changeme``.

Deploy to IBM Cloud
---------------------

1. First you'll need an IBM Cloud account. Visit the following depending on your requirements:

- [IBM Cloud](https://console.bluemix.net/) **(publicly available)**
- [IBM Cloud (CIO)](https://console.w3ibm.bluemix.net/) **(restricted to IBMer's)**


2. After creating an account, you'll need to install the IBM Cloud CLI, instructions can be found at the following:

- https://console.bluemix.net/docs/cli/reference/ibmcloud/download_cli.html

3. After successfully installing the IBM Cloud CLI, you'll need to configure it and perform the login with your credentials, as instructed at the following link:

- https://console.bluemix.net/docs/cli/index.html#step3

4. After successfully performing login, you must customize your [manifest.yml](./manifest.yml) file in order to successfully deploy the application to IBM Cloud, you can use the following as reference:

```yaml
---
applications:
- name: tlcbr
  memory: 256M
  instances: 1
  host: tlcbr
  path: .
  buildpack: python_buildpack
  random-route: false
  env:
    DJANGO_DEBUG: off
    DJANGO_SETTINGS_MODULE: tlcbr.settings.production
    DJANGO_SECURE_SSL_REDIRECT: on
    DJANGO_SECRET_KEY: changeme
```

5. After successfully preparing your manifest, you can push your application to IBM Cloud by using `ibmcloud cf push`, if all steps were followed accordingly, you should see a similar output:

```bash

[fsilveir@fsilveir tlcbr]$ ibmcloud cf push
Invoking 'cf push'...

Pushing from manifest to org FELIPE_STUDY / space dev as fsilveir@br.ibm.com...
Using manifest file /home/fsilveir/git/tlcbr/manifest.yml

Creating app tlcbr in org FELIPE_STUDY / space dev as fsilveir@br.ibm.com...
OK

Using route tlcbr.w3ibm.mybluemix.net
Binding tlcbr.w3ibm.mybluemix.net to tlcbr...
OK

Uploading tlcbr...
Uploading app files from: /home/fsilveir/git/tlcbr
Uploading 10.1M, 378 files
Done uploading               
OK

Starting app tlcbr in org FELIPE_STUDY / space dev as fsilveir@br.ibm.com...
Downloading python_buildpack...
Downloaded python_buildpack
Creating container
Successfully created container
Downloading app package...
Downloaded app package (15.9M)
Staging...
-------> Buildpack version 1.5.15

[ ... ]

Staging complete
Uploading droplet, build artifacts cache...
Uploading build artifacts cache...
Uploading droplet...
Uploaded build artifacts cache (78.6M)
Uploaded droplet (108.9M)
Uploading complete
Destroying container
Successfully destroyed container

0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
1 of 1 instances running

App started


OK

App tlcbr was started using this command `uwsgi --http-socket=:$PORT --master --workers=2 --threads=8 --die-on-term --wsgi-file=tlcbr/wsgi_production.py  --static-map /media/=/app/tlcbr/media/ --offload-threads 1`

Showing health and status for app tlcbr in org FELIPE_STUDY / space dev as fsilveir@br.ibm.com...
OK

requested state: started
instances: 1/1
usage: 256M x 1 instances
urls: tlcbr.w3ibm.mybluemix.net
last uploaded: Wed Dec 26 13:26:23 UTC 2018
stack: cflinuxfs2
buildpack: python_buildpack

     state     since                    cpu    memory      disk      details
#0   running   2018-12-26 11:32:33 AM   0.0%   0 of 256M   0 of 1G

```

For more information on how to work with cloudfoundry manifests, check the following link:

- https://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html
