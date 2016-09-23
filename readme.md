
# Django Deployment V1.02

# Step 1
----
##  Build a project!

In your terminal:
```bash
> django-admin startproject projectName
> cd projectname
> touch .gitignore or nul> .gitignore (PCs)
```

In your `.gitignore` file, add `*.pyc` and `venv/`

Now back in your terminal:

```bash
> git init
> git add .
> git commit -m "my first commit"
```

Now build the rest of your project, committing to your local repo as necessary.

###Important!
>Note where the `git init` command occurred: At the *same level* as your Django project's `manage.py` file


# Step 2
----
1. Go to GitHub
2. Create a repository (don't initialize with a README.md file)

# Step 3
----

Time to connect our local repository with the GitHub repo. Return to your terminal and paste the code generated by GitHub to your terminal. Those ones might look something like this:

```bash
> git remote add origin https://github.com/MikeHannon/myRepoName.git
> git push -u origin master
```

# Step 4
----

When finished with your project (with your Django virtual environment active), let's create a text file that lists our dependencies. This will make it easier to install those dependencies later. In your terminal:

```bash
(djangoEnv)> pip freeze > requirements.txt
```

#### edit this requirements.txt file removing pygraphviz, pydot, mysql and other similar 'tricky' to install pieces.


Make sure to `add`, `commit` and `push` your final changes to GitHub. Once you login to AWS and set up a cloud server, you'll be pulling code from your GitHub (or Bitbucket) repository.

```bash
> git add .
> git commit -m "add python dependencies"
> git push origin master
```



# Step 5
----
### On to AWS!
>Note: You'll need an AWS account, which you can sign up for [here](http://aws.amazon.com). It's free for a year, so long as you don't have more than 1 (free-tier) instance up at a time!

1. Login to AWS Console
2. Launch a new instance from the EC2 Dashboard (top left of main console leads to a page with a blue button)
3. Select *Ubuntu Server 14.04* option
4. Select *t2.micro* option and click *Review and Launch*

###Time to set up security settings
+ Click the *Edit security groups* link in the left hand column, choose the security group, (e.g. launch-wizard-1) and edit/add the following *Inbound* rules
+ SSH type should be sourced to MyIP (this has to be updated everytime you change IP addresses. It adds a level of security, but can be annoying to maintain.)
+ HTTP and HTTPS types should be sourced to *Anywhere* 

##Click *Review and Launch*

You'll be asked to create a key file. This is what will let us connect and control the server from our local machine. Download the file to a place you'll remember (probably a good idea is to have a `keys` folder to house this types of files). Do *NOT* put this folder in your project or *ANYWHERE* that gets pushed to GitHub.

##Click *Launch*
You should see a launched instance in your dashboard. A blue button at the top saying connect will show up. Eventually we'll click that and select *Connect*. But first:

# Step 6
----

Back in your terminal, navigate to the folder that holds the key file you just downloaded.

```bash
> cd My/Keys/Folder
```

*Mac users:* Run a `chmod` command to alter the permissioning of your key file.

```bash
> chmod 400 my-key-pair.pem
```

Now we're ready to use that file to connect to the AWS instance! That means, in your AWS console, connect to your instance and use the supplied code in your terminal (PC users: use a bash terminal or putty to do this).

If all goes well, you should be in your Ubuntu cloud server! Yay you! Your terminal should show something like this in the far left of your prompt:

```bash
ubuntu@ip-my-ip:~$ #Commands you write appear here
```

# Step 7
----
Now we are going to set up our Linux box for deployment.

In the terminal:

```bash
ubuntu@ip-my-ip:~$ sudo apt-get update
ubuntu@ip-my-ip:~$ sudo apt-get install python-pip python-dev libpq-dev postgresql postgresql-contrib nginx git
```
You just installed `pip` and some other python libraries, a `postGRES` database, `ngnix` (we'll talk about that shortly) and `git`, which you already know and love.

```bash
ubuntu@ip-my-ip:~$ sudo apt-get update
ubuntu@ip-my-ip:~$ sudo pip install virtualenv
```

Time to clone your project from GitHub.

```
ubuntu@ip-my-ip:~$ git clone https://github.com/MikeHannon/myRepoName.git
```
At the moment your current folder directory should looks something like this.

```bash
- ubuntu
  - myRepoName
    - apps
    - projectName
    - ... # other files/folders
```

Navigate into this project and run `ls` in your terminal. If you don't see `manage.py` as one of the files, *STOP*. Review the setting up GitHub/Git pieces from earlier.

If everything looks good, let's make a virtual environment in our cloud server!

```bash
ubuntu@ip-my-ip:~/myRepoName$ virtualenv venv
ubuntu@ip-my-ip:~/myRepoName$ source venv/bin/activate
```
~~ubuntu@ip-my-ip:~/myRepoName$ pip install -r requirements.txt~~
```bash
(venv) ubuntu@ip-my-ip:~/myRepoName$ pip install django bcrypt django-extensions
(venv) ubuntu@ip-my-ip:~/myRepoName$ pip install gunicorn
(venv) ubuntu@ip-my-ip:~/myRepoName$ pip install psycopg2
```
# NOTE FOR STEP 8 and BELOW
### Anywhere you see {{myRepoName}} -- replace that whole thing INCLUDING the {{}} with your outer folder name.
### Anywhere you see {{projectName}} -- replace that whole thing INCLUDING the {{}} with the project folder name.
# Step 8
---
Navigate into your main project directory (where `settings.py` lives). We're going to use a built-in text editor in the terminal to update the code in `settings.py`. For example:

```bash
(venv) ubuntu@ip-my-ip:~/myRepoName$ cd {{projectName}}
(venv) ubuntu@ip-my-ip:~/myRepoName/projectName$ sudo nano settings.py
```

`settings.py` is now open for editing. Change the following:

``` python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'anyname', # This name must be lowercase but can be anything!
        'USER': 'mikehannon', # you can put whatever you want here
        'PASSWORD': 'passwordYO', # you can put whatever you want here
        'HOST': 'localhost',
        'PORT': '',
    }
}
```

And add the following (this will allow you to serve static content):

```python
#Inside settings.py
STATIC_ROOT = os.path.join(BASE_DIR, "static/")
```

Now just `ctrl-X` and save! We just set up our `settings.py` for a postgres database and added a place for all of our static files.

# Step 9
---

Let's actually set up the database.  Note that all of the commands in the Postgres shell must end with a semicolon.

```bash
(venv) ubuntu@ip-my-ip:~$ sudo su - postgres
(venv) ubuntu@ip-my-ip:~$ psql
postgres=# CREATE DATABASE anyname; #matches the anyname in the DATABASES and is all lower case!
postgres=# CREATE USER mikehannon WITH PASSWORD 'passwordYO'; #username is not in quotes, but password is, and both should match what you put into settings.py.
postgres=# GRANT ALL PRIVILEGES ON DATABASE anyname TO mikehannon;
postgres=# \q #quits this prompt
ubuntu@ip-my-ip:~$ exit
```

# Step 10
---
# Getting close!
Navigate back to the folder that holds `manage.py`. Make sure your virtual environment is on!

```bash
(venv) ubuntu@ip-my-ip:~myRepoName$ python manage.py collectstatic #say yes
(venv) ubuntu@ip-my-ip:~myRepoName$ python manage.py makemigrations
(venv) ubuntu@ip-my-ip:~myRepoName$ python manage.py migrate
```
Migrations made. Now let's run the following:

```bash
(venv) ubuntu@ip-my-ip:~myRepoName$ gunicorn --bind 0.0.0.0:8000 projectName.wsgi:application
```

Run `ctrl-c` and `deactivate` your virtual environment.

# Step 11
---

Next, we're going to tell this green unicorn (`gunicorn`) service to start a virtualenv, navigate to and start our project, *all behind the scenes*!

```bash
ubuntu@ip-my-ip:~$ sudo nano /etc/init/gunicorn.conf
```

Add the following to this empty file, updating the code that's in between curly braces {{}} (this assumes the name of your virtual environment is `venv`):

```
description "Gunicorn application server handling our project"
start on runlevel [2345]
stop on runlevel [!2345]
respawn
setuid ubuntu
setgid www-data
chdir /home/ubuntu/{{myRepoName}}
exec venv/bin/gunicorn --workers 3 --bind unix:/home/ubuntu/{{myRepoName}}/{{projectName}}.sock {{projectName}}.wsgi:application
```

*REMINDER: myRepoName is the name of the repo you cloned in, projectName is the name of the folder that was used when you ran the django-admin startproject command. This folder is sibling to your apps folder.*

Here's what's actually happening:
1. `runlevel`s are system configuration bytes (just use 2,3,4,5 as stated on the start, stop)
2. `respawn` -- if the project stops, restart it
3. `setuid` -- ubuntu can use this project
4. `setgid` -- establishes a group
5. `chdir` -- go to the /home/ubuntu/{yourProject} #This needs to be updated to have your project's name
6. `exec venv/bin/gunicorn`... -- execute `gunicorn` in your `virtualenv` where you pip installed it. Futhermore, bind some workers to it and activate the `wsgi` file in your main project folder. *Look at these names carefully!*

To turn on or off this process:

```bash
ubuntu@ip-my-ip:~$ sudo service gunicorn start
#OR
ubuntu@ip-my-ip:~$ sudo service gunicorn stop
```
##Turn it on!

# Step 12
---

One final file to edit. From your terminal:

```bash
ubuntu@ip-my-ip:~$ sudo nano /etc/nginx/sites-available/{{projectName}}
```

Add this to the following, editing what's inside curly brackets {{}}:

```
server {
    listen 80;
    server_name {{yourEC2.public.ip.here}}; # This should be just the digits from AWS public ip!
    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/ubuntu/{{myRepoName}};
    }
    location / {
        include proxy_params;
        proxy_pass http://unix:/home/ubuntu/{{myRepoName}}/{{projectName}}.sock;
    }
}
```
Remove the # This should be just the digits from AWS public ip! statement

Run `ctrl-x` and save.


Now in terminal, run the following (taking note of the space after {{projectName}}):

```bash
ubuntu@ip-my-ip:~$ sudo ln -s /etc/nginx/sites-available/{{projectName}} /etc/nginx/sites-enabled
ubuntu@ip-my-ip:~$ sudo nginx -t
```

Take a careful look at everything that's in that file. Compare these names to the `gunicorn` names, and what you actually are using!

##Finally:

```bash
ubuntu@ip-my-ip:~$ sudo service nginx restart
```
If you get an *OK*, hopefully you are rockin and rollin' and your app is deployed! Go to the public domain and bask in its brilliance!
