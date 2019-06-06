# flask-tutorial-oreilly
Building the tutorial app from Miguel Grinberg's "Flask Web Development," 2nd ed., published by O'Reilly

## Development

In order to continue working on this project, you will need to set up a virtual environment with the proper dependencies and versions. The following Linux commands will suffice:

```sh
# Clone the project repo
# In project folder, create a new virtualenv called env
# Naming it env will ensure it is ignored by git
$ python -m venv env

# Activate the virtualenv
$ source env/bin/activate

# Install required dependencies with pip
$ pip install -r requirements.txt

# If dependencies or versions need to be changed, update requirements.txt
$ pip freeze > requirements.txt

# All development work must be done while the virtualenv is activated. Running the app will fail if attempted without virtualenv.

# Deactivate the virtualenv if needed
$ deactivate
```

## Deployment

Flask's built-in web server is sufficient for local testing and debugging but should never be used in a public-facing production environment. For my Raspberry Pi (Raspbian) deployment, I have preferred using uWSGI to run the Flask app behind an nginx web server. A daemon can be created to ensure the web app will automatically restart in the event of a server failure or power cycle. Below are steps to replicate my deployment, but various components are swappable for your favorite Unix distro, web server, etc.

#### Download everything
Clone the repo, create/activate the virtual environment, and install the required dependencies with pip (as shown in "Development" instructions above)

Install the necessary system packages

```sh
$ sudo apt-get install systemd nginx uwsgi
```

#### Configure .ini file
The appserver.ini file contains the options used when spinning up the uWSGI server. A few items in the file may need to be altered to fit your system:

* `chdir` should be the path to the project's top level folder
* `env` likewise should be adjusted to the correct project folder location
* `socket` may be altered to any custom port, but ensure that the port listed in `socket` and in the `app.run()` arguments of hello.py and wsgi.py all match
* `stats` may likewise be set to any custom port

#### Configure nginx
Since nginx can work as a reverse proxy and interface multiple cotenant app deployments with the internet, it needs to be configured to point a given web request to the correct application.

```sh
# Create a new configuration file (called flasktut here,
# but any name will do)
$ sudo nano /etc/nginx/sites-available/flasktut

# Save the following to that file (note that this does not yet
# support HTTPS and so port 443 is not included):
server {
  listen 80;
  server_name YOUR_DOMAIN_NAME_OR_IP_HERE;

  location / {
    include uwsgi_params;
    # IMPORTANT: if you changed the socket port in appserver.ini # you will need to change it here as well
    uwsgi_pass 127.0.0.1:5001;
  }
}

# --------
# Symlink this config file from the sites-available director to
# the sites-enabled directory
$ sudo ln -s /etc/nginx/sites-available/flasktut /etc/nginx/sites-enabled

# Restart the nginx service
$ sudo service nginx restart

# The nginx service can be stopped or started (for maintenance,
# etc.) with the following
$ sudo service nginx stop
$ sudo service nginx start
```

#### Set up the daemon
Creating a systemd daemon ensures that the app server restarts itself after failures and power on

```sh
# Create the configuration file (named flasktut.service, but
# ANY_NAME.service will do)
$ sudo nano /etc/systemd/system/flasktut.service

# Save the following in that file, replacing username and paths
# to project directory to match your deployment:
[Unit]
Description=uWSGI instance to serve Flask tutorial app

[Service]
User=kevin
Group=www-data
WorkingDirectory=/home/kevin/projects/flask-tutorial-oreilly/
Environment="PATH=/home/kevin/projects/flask-tutorial-oreilly/env/bin"
ExecStart=/home/kevin/projects/flask-tutorial-oreilly/env/bin/uwsgi --ini /home/kevin/projects/flask-tutorial-oreilly/appserver.ini
Restart=on-failure

[Install]
WantedBy=multi-user.target

# ----------
# Start the daemon
$ sudo systemctl start flasktut.service

# Enable the daemon to start on boot
$ sudo systemctl enable flasktut.service

# It will be necessary to restart the daemon if changes are made
# to the app which would initialize during the app's start up
# process -- such as swapping Bootstrap for another framework
$ sudo systemctl restart flasktut.service
```
