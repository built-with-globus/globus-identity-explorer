# globus-id-explorer
This is a Web app that allows anyone to explore their Globus identity information. It is intended 
to be deployed on a public Web server so that one can connect to it using a Web browser, login, 
see the data that Globus's Auth API provides to the app, and logout. The purpose of the app is to 
show Web developers what the identity information returned from the Auth API looks like so that 
they can then write their own apps that use the Auth API.

This app is meant to be deployed as a [WSGI application](https://wsgi.readthedocs.io/en/latest/) 
using a standard Web server (e.g., Apache) as a host. The server is responsible for providing a 
secure HTTP (HTTPS) environment in which the app can run. The Web server administrator must 
enable the WSGI module and add a server configuration module that references the location where 
this app has been installed. Instructions are below, and this repository includes sample 
configuration files.

## Prerequisites
Before installing the app, you must have the following already available on your Web server system.

1. A Web server! The examples provided here are for the Apache Web server, available on Linux systems.
2. A WSGI module for your server. Check your server documentation.
3. A Python installation. Python 3 is preferred, and the examples below assume it.
4. The ``virtualenv`` and ``pip`` Python tools.

## Installation
The first installation step is to install the app files in a location where your web server can
access them. Assuming that your Web server uses the /var/www/html directory as its document 
root, you might want to create /var/www/apps as the root for your Web apps.  Create the directory
and set permissions so you can put things there.
```
% sudo su
[sudo] password for liming: 
# cd /var/www
# mkdir apps
# chown liming:liming apps
# exit
```
Now clone the git repository in the new directory to make a local copy of everything.
```
% cd /var/www/apps
% git clone http://github.com/lliming/globus-id-explorer.git
[git does its thing]
% cd globus-id-explorer
% ls
auth_example.conf  auth_example.py  auth_example.wsgi  flask_example.py  flask_example.wsgi  hello.wsgi  requirements.txt
% 
```
This will create a subdirectory called globus-id-explorer with the files in it.

Next, create a Python virtual environment and install the required Python packages in it.
```
% virtualenv -p python3 venv
[virtualenv does its thing]
% source venv/bin/activate
(venv) % pip install -r requirements.txt
[pip does its thing]
(venv) % deactivate
% 
```
Now, edit the ``globus-id-explorer.wsgi`` file and change the path in the sys.path.insert line 
so that it matches the path to your app directory. (If you installed your app in ``/var/www/apps/globus-id-explorer``
as shown above, the path is already set properly and you won't need to change it.)

Next, edit the ``auth.conf`` file. This is an Apache configuration snippet that tells the Apache
Web server how to find your app. The path to the app directory appears on three lines, and you'll
need to adjust each to match your installation (if it isn't /var/www/apps). On the first line
of the file, make sure the path is correct, up to and including the venv subdirectory that you
created above.  On the line beginning with ``WSGIScriptAlias``, make sure the path is correct,
up to and including the globus-id-explorer.wsgi file. Finally, inside the Directory directive,
make sure that the path is correct up to and including the apps directory *above* your installation
directory. (Don't include the globus-id-explorer directory name.)

After you've edited ``auth.conf``, you'll need to add it to your Web server configuration and
restart the Web server. On my system (Fedora with Apache installed), I can do it as follows.
```
% sudo cp auth.conf /etc/httpd/conf.d/
% sudo systemctl restart httpd
```
There's one more configuration piece that needs to be performed before you can use the app.
But in order to do it, you'll need to first register the app with Globus.

## Register with Globus
All OIDC/OAuth2 apps must be registered with their authentication service. To register your
app with Globus, open a Web browser and connect to https://developers.globus.org/. Click
"Register your app with Globus." 

If you haven't registered an app before, you'll first need to create a Project. This is 
to help you keep track of your apps, so you can name the project anything you like.

After you've created a Project, click the "Add..." button in the upper-right corner of the
Project panel and select "Add new app."

In order to be consistent with the app's prompts, you should name the app "Display Your
Auth Data." The scopes field should be 
``openid email profile urn:globus:auth:scope:auth.globus.org:view_identity_set``. (Note
that you only have to type the first few letters and the field will find what you're
looking for. The last scope is easiest to find by typing 'set'.) The most important
field is the "Redirects" field. It must be set to your Web server's HTTPS address, plus 
``/auth/login`` on the end. The ``/auth`` corresponds to the app path you specified in
the ``auth.conf`` file. The ``/login`` part is the login path defined in the app's code.
On my server, it looks like this:
```
https://home.leeandkristin.net/auth/login
```
You can leave the rest of the app registration form blank, or mess around with different
values if you feel adventurous.

When you click "Create App," you'll see your app's registration data. This is where you'll
get the data you need to complete configuring your app.

## Complete app configuration

Now that your app is registered with Globus, edit the ``globus-id-explorer.conf`` file.
First, change the ``SERVER_NAME=`` value to your Web server's HTTPS address, plus
``/auth`` on the end. E.g., ``https://home.leeandkristin.net/auth``. (If you don't do
this, your app won't respond to requests. It's important!) Then, copy and paste the
Client ID from your Web browser window (showing your app registration) into the line
beginning with ``APP_CLIENT_ID``.  Finally, in your Web browser, scroll to the bottom
of your app registration and click ``Generate New Client Secret``. Enter a label for
the client secret (it can be anything you like), and click ``Generate Secret``. Then
copy and paste the secret character string into the line beginning with
``APP_CLIENT_SECRET``.  Save the app's configuration file.

## Try it out
Now that your app has been installed and both the app and your Web server are
fully configured, you should be able to use the app.  Open a new Web browser window
and enter the app's address. E.g., ``https://home.leeandkristin.net/auth``.

Since you logged in to Globus when you registered the app, you probably won't have to
authenticate again. Instead, you'll jump straight to the "consent" page where you
tell Globus that it's ok for the app to access your identity information. If you
agree, then you'll return to the app and it will tell you you're logged in and
display a bunch of your identity data. If you click "Logout," you can return to the
app and it will ask you to login using Globus.
