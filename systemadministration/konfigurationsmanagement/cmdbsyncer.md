# [cmdbsyncer](https://github.com/kuhn-ruess/cmdbsyncer)

```sh
cd /var/www
git clone https://github.com/kuhn-ruess/cmdbsyncer
cd cmdbsyncer
```

Always Make sure you are in /var/www/cmdbsyncer

`python3.11 -m venv ENV`

This Environment needs to be loaded from now on, every time something is done with the syncer, also for every Cronjob which you will run.

`source ENV/bin/activate`

To this Environment, you install the Python Libraries. This is done with just one command:

`pip install -r requirements.txt`

In Case, you plan to use Ansible, also import the Ansible requirements:

`pip install -r requirements-ansible.txt`

Extra Database stuff you find in requirements-extras.txt

## Install Mongodb Server

The Syncer needs the Mongodb. All you need to do is to install it, with your Packet Manager. Then you are ready to go.

## The Web Interface

To take a brief look, you can start the development Server:

`flask run --host 0.0.0.0 --port 8080`

But then you should Setup UWSGI. There is an Example with [UWSGI and Apache](https://cmdbsyncer.readthedocs.io/en/latest/basics/uwsgi_apache/), but it's even easier with NGINX. Or go with this simpler one, using [mod_wsgi and Apache](https://cmdbsyncer.readthedocs.io/en/latest/basics/install_wsgi/)

https://cmdbsyncer.readthedocs.io