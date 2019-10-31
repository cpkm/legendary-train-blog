---
layout: single
title:  "Flask deployment on Heroku"
date:   2019-05-31 13:27:01 -0400
categories: code
---

You've developed a Flask app locally. It runs, and you are at the point that you'd like to publish to the web. Or get the ball rolling, at least. It is time for deployment. Unlike static a webpage, a Flask app requires significant computing power in order to run: Python, and all the app's package bependencies, must be available on the host server, and, rather importantly, dedicating processing resources (CPU) and hard disk storage may be required. If your app utilizes a database, this must be stored on the host server and be readily accessible at all times.

If you have the required experience, you may opt to run your own Linux server to host your app. While not particulary difficult, this option does require a decent amount of background knowledge and manual maintenance. Fortunately, there is a better way. Several third-party cloud hosting companies now offer app hosting services, typically branded as Platform as a Service (PaaS). These services offer dedicated storage and computing resources specifically for app deployment. Many also offer manageed database solutions and a variety of other cloud-based services. Popular PaaS clients include Amazon Web Services (AWS), Google Cloud Services (GCS), and Heroku. The offerings and pricing scheme from each provider varies slight, but most offer a free (or nearly-free) option for low-use or hobby-level web applications. 

Here I will run through the process required to deploy your app to a Heroku server. This will include configuring the Heroku app, migrating your database to the Heroku platform, and deploying your app to the web.

## Heroku
[Heroku](https://heroku.com) is a very popular PaaS. It is a flexible platform with capabilities for the deployment of a variety of app languages including: Ruby, Java, Node.js, and importantly Python. Heroku is a tiered service, allowing you to customize the computing power and storage dedicated to your app. For low-demand apps there is a hobby-level tier, which is free and sufficient for most individual needs.

Heroku allows for deployment through Git. This is extremely convienient as updating your deployed app can be as simple as pushing your git commit to the Heroku server. Heroku also offers continuous deployment via GitHub. If your app is stored in a GitHub repository then the Heroku app is updated whenever a commit is pushed to the monitored branch. Easy. I'll assume your app is stored in a git repository.


### Heroku account and CLI

I you don't have on already, begin by signing up for a [Heroku account](https://signup.heroku.com/). For now you can select all of the free options. You may need to enter a credit card for some services or verification, but unless you are scaling up your app you should be able to use the free tier for everything shown here.

Most (if not all) of the setup can be performed on the Heroku web interface, however Heroku also offers a command-line-interface (CLI) which can be utilized, for example, in the Mac OS terminal. Installation commands can be found [here](https://devcenter.heroku.com/articles/heroku-cli#download-and-install). To install the Heroku CLI on a Mac OS system run the terminal command (requires Homebrew)
```
brew tap heroku/brew && brew install heroku
```

Once you've created a Heroku account and the CLI is installed you can login using
```
heroku login
```
This will open a browser window for you to enter you Heroku account details. You should only need to do this once, after which your authenticated status is rememebred.

### Create a Heroku app

Again, this can be performed (rather intuitively) using the web interface. Alternatively, using the CLI you can create a Heroku app using
```
heroku apps:create <your_app_name>
```
The name of your app must be unique globally as it serves as a unique identifier of your app in the Heroku system (it is also the default domain of you app once deployed: https://<your_app_name>.herokuapp.com)

You may also wish to create a Heroku Pipeline and add you app to it. A Pipeline is a sort of workflow, useful for dealing with larger projects with multiple apps. You may find it useful, but it isn't necessary.

### Web server

Use gunicorn
```
pip install gunicorn
```



### Install Postgres
Install Postgres

Also need ```psycopg2```
```
pip install psycopg2
```
Will need to install Postgres *before* installing psycopg2, otherwise the install will fail due to missing dependencies.

### Requirements and Procfile

Heroku needs a requirements.txt file in order to install the needed Python packages for your app. While in your app main directory
```
pip freeze > requirements.txt
```
Heroku also needs a custom *Procfile*, which give a list of commands to execute when the app is deployed.

```
web: flask db upgrade; gunicorn <your-app>:app
```

This upgrades the database and starts the gunicorn server for your app. ```<your-app>``` is the name of your main .py file and ```app``` is the assignmnt of ```create_app()``` in this file.
    
### Transfer database data

Heroku utillizes the PostgreSQL database structure. If you have been using this in development then transfering your data to the Heroku database is simple (proceed directly to ```pg:push```). However, you may have been using MySQL, or SQLite during development, in which case migrating your database is slightly more complicated.

There are a few migration tools which can accomplish this task, such as [pgloader](https://pgloader.io/), or the Ruby gem [sequel](https://sequel.jeremyevans.net/). I've had issues using pgloader to migrate from SQLite where primary key squences are not initialized properly (the result is an attempted *null* insertion as a primary key when making a new database entry, which is not allowed in Postgres), so I'd recommend using sequel instead for this task.

Provided you have ruby, rubygems, and postgres installed, as well as the relavent postgres-dev and sqlite3-dev libraries, you can install sequel by running
```
gem install sequel pg sqlite3
```
Next, I'll create a new, empty postgres database and fill it with data from my SQLite database used in development.
```
createdb newdb
sequel -C sqlite://app.db postgres://localhost/newdb
```
Here I've assumed we are in the local directory containing the development SQLite dabase named ```app.db```. In general this portion of the command can be written using the relative path to your .db file: ```sqlite://<path-to-sqlite-db>```.

Once this transfer is complete you can use Heroku's ```pg:push``` command to push your newly-populated local posgres database to your Heroku postgrea database. In order to *push* data to a Heroku database, the database must be empty. If your Heroku database is not empty you must reset it
```
heroku pg:reset -a <your-app-name>
```
This is a destructive process that deletes all of the data in your Heroku database. Take necessary precautions! Now you can perform the push
```
heroku pg:push newdb DATABASE_URL <your-app-name>
```
That's it - you're set. Note that ```DATABASE_URL``` is the environment variable pointing to your Heroku postgres database. Heroku will interpret this variable properly, so you don't need to copy/paste an actual url.

<p>{{ jekyll.environment }} {{ site.comments.provider }} {{ page.comments }}</p>
{% include comments.html %}







