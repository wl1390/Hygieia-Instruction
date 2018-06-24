# Hygieia Dashboard Instruction

* Dependencies 

* Database - MongoDB

* Build The Project

* Connect API

* Congifure Collectors

## Dependencies

maven, mongodb 3.0 or above, java 1.8 recommended

### Note for hygieia on Raspberry PI

Hygieia requires MongoDB 3.0 or above. Starting from version 3.2 MongoDB only supports 64-bit binary which the current Raspberry PI does not support. So it's essential to download the 32 bit MongoDB 3.0 version for Hygieia. The official MongoDB 3.0 32-bit binary for Linux could not be executed by Raspberry PI. So we have to compile binaries ourselves.

Luckily we can find ready-to-use binaries here:

https://andyfelong.com/2017/08/mongodb-3-0-14-for-raspbian-stretch/

https://andyfelong.com/2016/01/mongodb-3-0-9-binaries-for-raspberry-pi-2-jessie/

Download both the core and tool binaries and copy them into /usr/bin by `sudo cp [your binary directory]/* /usr/bin/`.

For maven, download maven from its website and follow the tutorial here to add the path:

http://agilerule.blogspot.com/2016/06/how-to-install-maven-on-raspberry-pi.html


## Database - MongoDB

Start a mongoDB session by running `mongod` if there isn't one already running. 

If `mongod` is not working, try `sudo mongod` or `mongod --repair && mongod` depending on the error.  

Run `mongo` to start a mongo shell.

Create a new database with custom name, for example "database", by running `use database`

Create a new database user with custom name and password by running 
```
db.createUser( { user: "user", pwd: "password", roles: [ {role: "readWrite", db: "database"} ] } )
```

You can view all the databases and users of each by running `show dbs` and `db.getUsers()` respectively.

## Build The Project 
Go to https://github.com/capitalone/Hygieia and get master either by git clone or download.

Cd in into this folder and run `mvn clean install package`

After successfully building the project, go into /UI and run `gulp serve`, you should be able to see the ui opening in a browser.

### Note for hygieia on Raspberry PI
`mvn clean install package` may fail in the UI step on the `npm install --ignore scripts` command, we'll have to do everything after this manually. 

When it fails on the UI step, go to /UI and delete /node and /node_modules

Follow the following command to install new node
```
pi@raspberrypi:~ $  sudo su -
root@raspberrypi:~ # apt-get remove nodered -y
root@raspberrypi:~ # apt-get remove nodejs nodejs-legacy -y
root@raspberrypi:~ # apt-get remove npm  -y
root@raspberrypi:~ # curl -sL https://deb.nodesource.com/setup_8.x | sudo bash -
root@raspberrypi:~ # apt-get install nodejs -y
    
```

check the node and npm version by running `node -v` and `npm -v`

run `npm install --ignore scripts`
After success, run `npm install bower -g` and `npm install gulp`
After success, run `bower install` and `gulp build`

`gulp build` might show error, but as long as `gulp serve` brings the dashboard browser up we are fine. 


## Connect API
Go to /api and create a dashboard.properties with content
```
dbname=database
dbusername=user
dbpassword=password
dbhost=localhost
dbport=27017
```
The dbname, username, password need to match your custom database information in mongoDB.

Run this command in root directory.
```
java -jar api/target/api.jar --spring.config.location=api/dashboard.properties -Djasypt.encryptor.password=hygieiasecret
```

The port should be available. If shows address already in use error, kill the application that is occuping the port.

Run `gulp serve` in /UI and you should see the UI with api connected. Sign up for an account and log in to configure your dashboard.

If in UI API shows connected but no edit text is there to log in, clear the browser cache and cookie and try again.

## Configure Collectors
Go to http://capitalone.github.io/Hygieia/collectors.html#tool-collectors and find the collector you want to use.

Follow Setup Instructions Step 1 and 2 (which is cd into the collector fold and run `mvn install`)

Configure the `application.properties` file according to the sample properties provided there.

Run the following command in the current collector's folder
```
java -jar target/[collectorname.jar] --spring.config.name=git --spring.config.location=../../../api/src/main/resources/application.properties
```

Refresh your dashboard and now you can configure the collector. (The collector might take several minutes to be enabled, but it shouldn't be longer than that.)

(Take into account that you might need to configure the api daily fetch limit of the source from which you are collecting, i.e. github repo or bitbucket repo.)



## Notes

### api connected but ui does not
If this happen, check the port and see if any application occupies the 8080 port.

### jenkins
Jenkins default port is 8000. Should not change it.

### github
The url you put in the ui should be the git clone path.
There is a problem with api commit timezone, it might not work properly. 
https://bitbucket.org/rest/api/1.0/projects/wl1390/repos/test/commits?until=master&limit=25

### bitbucket
For the .properties set up, write `git.api=/2.0/repositories/` instead of the documentation. The url you put in the UI should be `https://api.bitbucket.org/{username}/{reponame}`

### jira
There might be bugs in the collector codes. Collector links to jira successfully on the API layer, but UI is not displaying the content properly.
