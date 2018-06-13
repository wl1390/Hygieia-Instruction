# Hygieia Dashboard Instruction

* Dependencies 

* Database- MongoDB

* Build The Project

* Connect API

* Congifure Collectors

## Dependencies

maven, mongodb, java

## Database - MongoDB

Start a mongoDB session by running `mongod` if there isn't one already running. 

Run `mongo` to start a mongo shell.

Create a new database with custom name, for example "database", by running `use database`

Create a new database user with custom name and password by running `db.createUser( { user: "user", pwd: "password", roles: [ {role: "readWrite", db: "database"} ] } )`

You can view all the databases and users of each by running `show dbs` and `db.getUsers()` respectively.

## Build The Project 
Go to https://github.com/capitalone/Hygieia and get master either by git clone or download.

Cd in into this folder and run `mvn clean install package`

Go into /UI and run `gulp serve`, you should be able to see the ui opening in a browser.

## Connect API
Go to /api and create a dashboard.properties with content
```
dbname=database
dbusername=user
dbpassword=password
dbhost=localhost
dbport=27017
```
The dbname, username, password need to match your custom database information.

Run this command in root directory.
```
java -jar api/target/api.jar --spring.config.location=api/dashboard.properties -Djasypt.encryptor.password=hygieiasecret
```

The port should be available. If shows address already in use error, kill the application that is occuping the port.

Run `gulp serve` in /UI and you should see the UI with api connected. Sign up for an account and log in to configure your dashboard.

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
if this happen, check the port and see if any application occupies the 8080 port.

### jenkins
jenkins default port is 8000. Should not change it.

### github
The url you put in the ui should be the git clone path.
There is a problem with api commit timezone, it might not work properly. 
https://bitbucket.org/rest/api/1.0/projects/wl1390/repos/test/commits?until=master&limit=25

### bitbucket
for the .properties set up, write `git.api=/2.0/repositories/` instead of the documentation. The url you put in the UI should be `https://api.bitbucket.org/{username}/{reponame}`
