# Move Rails Application onto Docker
Docker Basics
## Pre-requisites
1) Create new virtual machine or AWS EC2 server using ubuntu 16 [Any fresh image you prefer]
2) Now install the prerequisites needed for running rails server like rvm, ruby version, bundler and postgres etc 
    *Make sure to copy the commands executed
3) Pull or clone the latest codebase from git or any version control system on the server
4) Run the deployment commands like precompile, db:migrate and any other custom commands need to be executed while deploying the application.
    *Make sure to copy the commands executed
5) Once environment creation is completed, we need to make sure the rails service(rails s) process keeps running, so we will use built in ubuntu process management tool called supervisord.
6) Go to codebase home directory and create new file called 'supervisord.conf'
7) Copy below content into the file
######### Here is a sample supervisord.conf for Rails service process management ##########
```
[supervisord]
nodaemon=false
 
stdout_events_enabled=true
stderr_events_enabled=true
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
 
[program:codebase]
command=/bin/bash -l -c "rails s -b 0.0.0.0"     
user=root
autostart=true
directory=/codebase
 
stdout_events_enabled=true
stderr_events_enabled=true
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
```
8) Now update the database.yml with the environment variables,
```
development: 
  adapter: postgresql
  encoding: unicode
  host: <%= ENV['DB_HOST'] %>
  database: <%= ENV['DB_NAME'] %>
  user: <%= ENV['DB_USER'] %>
  password: <%= ENV['DB_PWD'] %>
  pool: 5
```

## Configuration
1) Install latest docker from https://www.docker.com
2) Run below command to confirm docker is working fine,
   sudo docker -v
   Result: Docker version 17.10.0-ce, build f4ffd25 [You might have newer version]
3) Now, go to root folder of your codebase you cloned earlier
4) Create new file with name as 'Dockerfile' [This is the file where you will copy all the commands you executed for configuring your application as per pre-reauisites steps]
######### Here is a sample Dockerfile for Rails application using postgres server ##########
```
   FROM ubuntu:16.04
   RUN apt-get update
   RUN apt-get install --yes --no-install-recommends software-properties-common
   RUN apt-get install --yes --no-install-recommends curl nano
   RUN apt-get install --yes --no-install-recommends libpq-dev
   RUN apt-get install --yes --no-install-recommends git
   RUN apt-get install --yes --no-install-recommends wget
   RUN apt-get install --yes --no-install-recommends default-jre
   RUN apt-get install --yes --no-install-recommends postgresql-9.4
   RUN gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3    7D2BAF1CF37B13E2069D6956105BD0E739499BDB
   RUN curl -sSL https://get.rvm.io | bash -s stable
   RUN /bin/bash -l -c "source /etc/profile.d/rvm.sh"
   RUN /bin/bash -l -c "rvm install ruby-2.0.0-p247"
   RUN /bin/bash -l -c "gem install bundler"
   ADD . /codebase
   WORKDIR /codebase
   * You can provide the RAILS_ENV name for below rails commands,
   RUN /bin/bash -l -c "bundle install --without development test"
   RUN /bin/bash -l -c "rake assets:precompile --trace"
   
   RUN apt-get install --yes --no-install-recommends supervisor
   RUN mkdir -p /var/log/supervisor
   ADD supervisord.conf /etc/supervisor/conf.d/supervisord.conf
   ENTRYPOINT ["/bin/bash", "-c", "/usr/bin/supervisord -c /etc/supervisor/conf.d/supervisord.conf -n"]
```



   
   

