# Description

Your Priorities is a web based platform that enables groups of people to share their ideas and together discover which are the most important ideas to implement by their communities. Users can add new ideas, add arguments for and against ideas, indicate if they support or oppose an idea, create a personal list of ideas and discuss all ideas. The end results are lists of top ideas in many categories as well as the best arguments for and against each idea. This service enables people to make up their minds about most issues in a short time.


Your Priorities is being used on the [YrPri.org](https://www.yrpri.org/) global eDemocracy service and the [Better Reykjavik](http://www.betrireykjavik.is) civic innovation website in Iceland amongst other places.

## 1: Docker on Ubuntu

Install [Docker](http://www.docker.io/) on your system: visit [http://docs.docker.io/en/latest/installation/#installation-list](http://docs.docker.io/en/latest/installation/#installation-list)


Clone Your Priorities locally
````bash
$ cd /yourpath
$ git clone https://github.com/rbjarnason/your-priorities.git
````
Copy database config template (no change needed):
````bash
$ cd your-priorities
$ cp config/database.yml.dist config/database.yml
````

Option 1 - Install Your Priorities images from Docker Index
sudo may not be necessary on your system (e.g. OSX)
````bash
$ [sudo] docker pull yrpri/base
$ [sudo] docker pull yrpri/postgresql
$ [sudo] docker pull yrpri/rails
````
(optional IRC support)
````bash
$ [sudo] docker pull yrpri/ngircd
$ [sudo] docker pull yrpri/kiwiirc
````

Option 2 - Build docker repositories from github dockerfiles
````bash
# Base docker repository
$ git clone https://github.com/rbjarnason/docker-base.git
$ cd docker-base
$ [sudo] docker build -t yrpri/base .
cd ..
````

And the same for:
https://github.com/rbjarnason/docker-postgresql.git and yrpri/postgresql
https://github.com/rbjarnason/docker-rails.git and yrpri/rails
(optional IRC support)
https://github.com/rbjarnason/docker-ngircd.git and yrpri/ngircd
https://github.com/rbjarnason/docker-kiwiirc.git and yrpri/kiwiirc

Start database
````bash
$ [sudo] docker run -i -t -d --name postgresql yrpri/postgresql
````

Optional IRC support
````bash
$ [sudo] docker -D run -d -p 6667:6667 yrpri/ngircd
$ [sudo] docker -D run -d -p 7778:7778 -v /root/certs:/etc/kiwiirc -e KIWI_IRC_SERVER_HOST=irc.yrpri.org -e KIWI_IRC_SERVER_PORT=6667 yrpri/kiwiirc
````

Start rails docker image pointing to your local Your Priorities installation
````bash
$ [sudo] docker -D run -d -link postgresql:db -p 3000:3000 -v /yourpath/your-priorities:/var/www/your-priorities -e APP_NAME=your-priorities yrpri/rails
````

See if it is running
````bash
$ [sudo] docker ps
````
Test it
````
The image will take a little while to start up, it will have to run bundle install each time its started.
Browse to http://localhost:3000/ or http://your.ip.addr.number:3000
````

Debug the Docker image
````bash
$ [sudo] docker ps -notrunc
$ [sudo] lxc-attach --name long_uid_from_docker_ps
<now you are in the image>
cd /var/log/supervisor
tail -f *
````

Default admin user and password
````
admin@admin.is
admin
````

## 2: Setup the project locally

Fork the project from GitHub
````
1. Create a GitHub account if you do not have one
2. Make sure you are logged in
3. Click the "Fork" button at the top of the page to create your own fork
````

Download / clone on your local Ubuntu installation
````bash
(replace YOURNAME with your github username)
$ git clone git@github.com:YOURNAME/your-priorities.git
````
Setup git to easily merge from the main branch. Add the following to the .git/config file:
````
[remote "robert"]
        url = git@github.com:rbjarnason/your-priorities.git
        fetch = +refs/heads/*:refs/remotes/robert/*
````

Merge the latest changes from the master branch
````bash
$ git fetch robert
$ git merge robert/master
````

## 2a: Development on Ubuntu


1. Install rvm the Ruby version manager
````bash
$ sudo apt-get install curl
$ curl -L https://get.rvm.io | bash -s stable
````

2. Go into the application and install all gems
````bash
sudo apt-get install build-essential
sudo apt-get install libxslt-dev libxml2-dev
sudo apt-get install libmysqlclient-dev
sudo apt-get install libpq-dev
sudo apt-get install libmagickwand-dev
bundle install
````

Install database dependencies
````bash
sudo apt-get install postgresql
sudo apt-get install mysql-server



````

4. Then start the psql shell
````bash
$ sudo su postgres
$ psql
````

5. When in psql create a user and the Your Priorities dev database
````bash
CREATE USER puser PASSWORD 'xxxxxxxx'
CREATE DATABASE yrpri_dev WITH ENCODING 'utf8';
GRANT ALL PRIVILEGES ON DATABASE yrpri_dev TO puser;
ALTER USER puser CREATEDB;
````

6. Then exit the postgres shell and copy and edit the config/database.yml.dist file
````bash
$ cd config
$ cp database.yml.dist database.yml
````

7. Then edit the database.yml file for your puser password

8. When ready create the database tables and seed the database:
````bash
$ rake db:schema:load
$ rake db:seed
````

9. Start the server
````bash
$ rails s
````

10. Navigate to http://localhost:3000/

## 2b: Development on OSX

1. Ensure prerequisites are installed
  * Ruby 2.0.0 (or later)
  * PostgreSQL ([Postgres.app](http://postgresapp.com/) seems to work OK)
  * Redis
    
    ````bash
        $ wget http://redis.googlecode.com/files/redis-2.6.14.tar.gz	
        $ tar xzf redis-2.6.14.tar.gz
        $ cd redis-2.6.14
        $ make
        $ cd src
        $ ./redis-server
    ````

2. From the application directory, install all gems
````bash
$ bundle install
````

3. Create a database user and the dev database. From psql shell:
````sql
create user puser password 'xxxxxxxx';
create database yrpri_dev with encoding 'utf8';
grant all privileges on database yrpri_dev to puser;
````

4. Copy the config/database.yml.dist file to config/database.yml, and edit the puser password.

5. Create the database tables and seed the database:
````bash
$ rake db:schema:load
$ rake db:seed
````

6. Start the server
````bash
$ rails s
````

10. Navigate to http://localhost:3000/

## Configuration

1. Log in as administrator
````
user email: admin@admin.is
password: admin
````

2. Change the administrator password, and configure the site.

## 3: Running the test


Currently Your Priorities only has one working test, more tests would be appricated. 
This one test is based on Selenium and tests most of the user facing features. To 
run the tests you need to open up two terminal windows.  You need to have Firefox 
installed.


In the first window you start the integration test before running the command in the other window
````bash
rake test:integration
````

In the second window when you see the test database being created from the output start the test server.
If you start it too early then the database cant be dropped for recreation and if you start the server too 
then Selenium won't have a server to test against.
````bash
rails s -e test
````

## 4: Production Deployment on Heroku

Your Priorities is now setup to be a Heroku app, using S3 and CloudFront for deployment.

Here are the parameters that are used in Heroku config.

Heroku parameters you need to setup yourself:

        AWS_ACCESS_KEY_ID:            ----------------------
        AWS_SECRET_ACCESS_KEY:        ----------------------
        CF_ASSET_HOST:                ----------------------.cloudfront.net
        DATABASE_URL:
		FACEBOOKER2_API_KEY:          ----------------------
        FACEBOOKER2_APP_ID:           ----------------------
        FOG_DIRECTORY:                ----------------------
        S3_BUCKET:                    ----------------------
        S3_KEY:                       ----------------------
        S3_SECRET:                    ----------------------
        YRPRI_ALL_DOMAIN:             1

The following Heroku addons need to be used and should be configured automatically:

* Adept Scale, from [$18/mo](https://addons.heroku.com/adept-scale)

        ADEPT_SCALE_URL:              ----------------------

* Flying Sphinx, from [$12/mo](https://addons.heroku.com/flying_sphinx)

        FLYING_SPHINX_API_KEY:        ----------------------

* Heroku PostgreSQL, from [free/$9/mo](https://addons.heroku.com/heroku-postgresql)

        HEROKU_POSTGRESQL_WHITE_URL:  ---------------------- 

* Memcachier, from [free/$15/mo](https://addons.heroku.com/memcachier)

        MEMCACHIER_USERNAME:          ----------------------

* PGBackups, [free](https://addons.heroku.com/pgbackups)

        PGBACKUPS_URL:                ---------------------- 

* RedisToGo, from [free/$9/mo](https://addons.heroku.com/redistogo)

        REDISTOGO_URL:                ---------------------- 

* Sendgrid, from [free/$10/mo](https://addons.heroku.com/sendgrid))

        SENDGRID_USERNAME:            ---------------------- 

# About

Your Priorities originated as a merge between [NationBuilder](https://github.com/jgilliam/nb-deprecated) by [Jim Gilliam](http://www.jimgilliam.com/) and [Open Direct Democracy](http://github.com/rbjarnason/open-direct-democracy) by [Róbert Viðar Bjarnason](https://github.com/rbjarnason) and [Gunnar Grimsson](https://github.com/gunnargrimsson) of the [Citizens Foundation](http://citizens.is). Earlier versions of this platform were called [Social Innovation](https://github.com/rbjarnason/social_innovation) and [Open Active Democracy](https://github.com/rbjarnason/open-active-democracy).

# License

Copyright (C) Róbert Viðar Bjarnason

This program is free software: you can redistribute it and/or modify it under the terms of the [GNU Affero General Public License](/agpl-3.0.txt) as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the [GNU Affero General Public License](/agpl-3.0.txt) for more details.



