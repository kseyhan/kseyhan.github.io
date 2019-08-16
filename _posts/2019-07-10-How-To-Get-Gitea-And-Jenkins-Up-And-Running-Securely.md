---
date: 2019-08-16
layout: post
title: "How to create a secured development environment using Gitea, Jenkins and Docker"
description: How to create a development environment using Gitea, Jenkins and Docker
categories: [jenkins, gitea]
tags: [jenkins, gitea, docker-compose, docker]
---

Today a little walkthru on how to setup yourself a somewhat secured development environment using Gitea as a Git Server and Jenkins as a CI/CD Solution on top.

We are starting with following ***docker-compose.yml***.

{% highlight yaml %}
version: "2"

networks:
  gitea:
    external: false

services:
  gitea:
    image: gitea/gitea:latest
    environment:
      - ROOT_URL=http://localhost:3000/
      - SSH_DOMAIN=localhost
      - RUN_MODE=prod
      - DISABLE_REGISTRATION=true
      - REQUIRE_SIGNIN_VIEW=true
      - DB_TYPE=postgres
      - DB_HOST=db:5432
      - DB_NAME=gitea
      - DB_USER=gitea
      - DB_PASSWD=asecret
    restart: always
    networks:
      - gitea
    volumes:
      - gitea:/data
    ports:
      - "127.0.0.1:3000:3000"
      - "127.0.0.1:11111:22"
    depends_on:
      - db
    cpuset: "1"
    mem_limit: 1024m

  db:
    image: postgres:11
    restart: always
    environment:
      - POSTGRES_USER=gitea
      - POSTGRES_PASSWORD=asecret
      - POSTGRES_DB=gitea
    networks:
      - gitea
    volumes:
      - postgres:/var/lib/postgresql/data
    cpuset: "1"
    mem_limit: 1024m    
  
  jenkins:
    image: "jenkins/jenkins:lts"
    ports:
      - "127.0.0.1:8080:8080"
      # - "50000:50000"
    volumes:
      - "jenkins:/var/jenkins_home"
    networks:
      - gitea
    cpuset: "2,3"
    mem_limit: 4096m
    restart: always
volumes:
  postgres:
  gitea:
  jenkins:

{% endhighlight %}


Description of some uncommented relevant parts:

* `ROOT_URL`: This is useful if the internal and the external URL don’t match (e.g. in Docker).
* `SSH_DOMAIN`: localhost: Domain name of this server, used for the displayed clone URL in Gitea’s UI.
* `RUN_MODE`: For performance and other purposes, change this to prod when deployed to a production environment.
* `DISABLE_REGISTRATION`: Disable registration, after which only admin can create accounts for users.
* `REQUIRE_SIGNIN_VIEW`: Enable this to force users to log in to view any page.

* `cpuset`:  limit the process to a specific cpu of the host
* `mem_limit`: limit the maximum ammount of memory this container can use

Most of the previously mentioned settings can happily be ignored and commented out if youre planing to run your environment on a save network or on localhost only but can come in pretty handy if youre trying to run this setup on a server thats facing the internet.

As we see ive got a special setup which requires the users to be signed in to view the code and where only administrators are able to create user accounts. This setup needs special threatment in the way we're going to setup a account for jenkins on the gitea server like we're going to see later.


Lets run `docker-compose up` to get this up and running.

Somwhere in the startup log in the console youre going to find the password for jenkins.

![screenshot jenkins password in console](/assets/img/posts/2019-08-16/jenkins_passwd.png)

Note it down, we're going to need this later.

Open `http://localhost:3000` to setup gitea and click on the login button.

![gitea main window](/assets/img/posts/2019-08-16/gitea_main.png)

Change the ssh port to the one you exposed on the hostsystem and setup your admin account.

![gitea setup window](/assets/img/posts/2019-08-16/gitea_setup.png)

create a new organisation.

![gitea new organisation 1 window](/assets/img/posts/2019-08-16/gitea_new_organisation_1.png)

![gitea new organisation 2 window](/assets/img/posts/2019-08-16/gitea_new_organisation_2.png)

Now it starts to get interesting and we're getting closer to the point why i decided to create this tutorial.

We're going to create a user for Jenkins.

![gitea website administration](/assets/img/posts/2019-08-16/gitea_website_admin_1.png)

![gitea website administration](/assets/img/posts/2019-08-16/gitea_website_admin_2.png)

![gitea new user](/assets/img/posts/2019-08-16/gitea_new_user.png)

After creating the new user youre going to get redirected to the user settings overview.

Update the settings for the user as follows:

![gitea user settings](/assets/img/posts/2019-08-16/gitea_usersettings.png)

Now add the user to the organisations you want your jenkins to have access to.

* open up the overview of your newly created organisation.

![gitea organisation settings 1](/assets/img/posts/2019-08-16/gitea_organisation_settings_1.png)

And here we are tada the point why i decided to create this lenghtly tutorial/report.. As i had to find out the hard way due to some bugs with the gitea jenkins plugin and the integration with the required sign in view i wanted gitea to have you have to add your jenkinsuser as a owner of the organisation to get it properly running or give your jenkinsuser global admin rights. If you decide to make your gitea public readable you can create a new group and give your jenkinsuser readonly rights instead of following the next few gitea steps here.

![gitea organisation overview](/assets/img/posts/2019-08-16/gitea_organisations.png)

![gitea organisation add user](/assets/img/posts/2019-08-16/gitea_organisations_add_user.png)

So far so good.. Your Gitea should be properly configured now.

Lets dive into getting Jenkins up and running.

Open up `http://localhost:8080` and enter the password from the console you noted down earlier.

![Jenkins first login](/assets/img/posts/2019-08-16/Jenkins_first_login.png)

Install the suggested plugins and wait for the installation routine to finish.
![Jenkins first login](/assets/img/posts/2019-08-16/Jenkins_suggested_plugins.png)


Create your admin user account
![Jenkins first login](/assets/img/posts/2019-08-16/Jenkins_create_admin_account.png)


Configure your the jenkins instance url as follows and click on finish!
![Jenkins first login](/assets/img/posts/2019-08-16/Jenkins_instance_url.png)


Now install the Gitea Plugin for Jenkins by clicking the configure Jenkins button.
![Jenkins first login](/assets/img/posts/2019-08-16/Jenkins_overview_1.png)


![Jenkins plugins link](/assets/img/posts/2019-08-16/Jenkins_Plugin_Link.png)

refresh the list of plugins

![Jenkins plugins refresh button](/assets/img/posts/2019-08-16/Jenkins_plugins_refresh.png)

and install the Gitea plugin
![Jenkins plugins install gitea plugin](/assets/img/posts/2019-08-16/Jenkins_Gitea_plugin_install.png)

For the sake of it lets restart jenkins and start to configure the gitea plugin

![Jenkins Gite Plugin successfully installed](/assets/img/posts/2019-08-16/Jenkins_Gitea_plugin_sucessfully_installed.png)

After logging back in change to the cofigure Jenkins view again.
![Jenkins first login](/assets/img/posts/2019-08-16/Jenkins_overview_1.png)

Click on configure system
![Jenkins configure system](/assets/img/posts/2019-08-16/Jenkins_cofiguration_button.png)

Click on add new Gitea Server
![Add new Gitea Server Button](/assets/img/posts/2019-08-16/Add_Gitea_Server_button.png)

Add a Name enter the url to your instance
Allow Jenkins to manage the hooks and add the credentials for your jenkinsuser.
![Gitea Server Configuration](/assets/img/posts/2019-08-16/gitea_server_configuration.png)

The error message that gives back a 403/Forbidden can be happily ignored since thats another bug in the gitea plugin that doesnt handle the sign in to view option for gitea properly.

Add the credentials of the user you configured earlier in Gitea as Jenkinsuser
![Gitea Server Configuration](/assets/img/posts/2019-08-16/Jenkins_gitea_server_add_credentials.png)

and save your configuration.
![Gitea Save Configuration Button](/assets/img/posts/2019-08-16/save_jenkins_configuration.png)

Now we're going to create a Gitea organisation in Jenkins.
Click on create new element in the main view.
![Gitea Save Configuration Button](/assets/img/posts/2019-08-16/Jenkins_create_new_element.png)


Give it the name of the organisation you chose before in gitea
Click on the Gitea Organisation button
and acknowledge with a click on the ok button.
![Gitea Save Configuration Button](/assets/img/posts/2019-08-16/Jenkins_new_gitea_organisation.png)

In the next window select the credentials for your gitea user fromt the dropdown.
The rest can stay as is save your configuration.
![Gitea Save Configuration Button](/assets/img/posts/2019-08-16/Jenkins_Gitea_Organisation_configuration.png)

At this point you should be up and running.
To test if thats realy the case create a testrepository for your organisation in gitea and add a Jenkinsfile to it.
![Gitea Testrepo](/assets/img/posts/2019-08-16/Gitea_testrepo.png)

Back to Jenkins scan your organisation for new repositories.
![Jenkins scan organisation](/assets/img/posts/2019-08-16/Jenkins_scan_organisation.png)

In the log you should find an entry that states that a repository was found.
![Jenkins Gitea Organization Log](/assets/img/posts/2019-08-16/Jenkins_organisation_scan_log.png)

Now back to Gite again go into the settings of the repository and make sure that Jenkins added a Webhook for your project.
![Jenkins Gitea Organization Log](/assets/img/posts/2019-08-16/gitea_testrepo_settings.png)

Afer 34 Screenshots we're finally done.

Happy coding, integrating and delivering!