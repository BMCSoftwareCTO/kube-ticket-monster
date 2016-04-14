# What is this?

This is the TicketMonster distribution, a showcase application for [JBoss Developer Framework](http://jboss.org/jdf).

## What can you find here?

The content of the underlying directories is as follows:

* `demo` - the sources of TicketMonster application - you can build and run it! Follow the [instructions](demo/README.md). Or you can see it at work [here](http://ticketmonster-jdf.rhcloud.com). 
* `cordova` - the sources of the TicketMonster Hybrid Mobile (Cordova) application. Follow the [instructions](cordova/README.md) to build and run it.

# Build

* Create alternate global settings.xml for Maven
	* settings_ticketmonster.xml in <Maven HOME>/conf
* Add the below <profile>
```xml
  <profiles>
    <profile>
        <id>redhat-techpreview-all-repository</id>
        <repositories>
            <repository>
                <id>redhat-techpreview-all-repository</id>
                <name>Red Hat Tech Preview repository (all)</name>
                <url>http://maven.repository.redhat.com/techpreview/all/</url>
            </repository>
        </repositories>
    </profile>
```

* Build with PostGRESql as External Datasource rather than use internal H DB
	* `cd ticket-monster/demo`
	* `mvn -gs c:\galapagos\apache-maven-3.3.3\conf\settings_ticketmonster.xml -Pdefault,postgresql clean package`

* Build Docker Image and Push to Docker Registry
	* `cd ticket-monster/demo`
	* `mvn -gs c:\galapagos\apache-maven-3.3.3\conf\settings_ticketmonster.xml -Pdefault,postgresql docker:build -DpushImage -DdockerRegistry=localhost:5000/dslm/`

# Deploy

You can deploy multiple parallel Ticket Monster stacks so as long as you ensure the following for the below commands:
* The Docker Host port in a `-p` mapping is unique for PostGreSQL. Recall that the host port is the number before the colon character
* The Docker Container ports in the `-p` mapping for PostGreSQL are constant at 5432. Recall that the container port is the number after the colon character.
* The Docker Container names are unique
* When starting the Wildfly container with Ticket Monster WAR
	* That the container names passed in the `--link` flag for PostGreSQL are the unique container names you used to start them. NOTE: the container names are the first part of the `--link` argument, before the colon character.

* Start postgresql container
  * `docker run --name db -d -p 5432:5432 -e POSTGRES_USER=ticketmonster -e POSTGRES_PASSWORD=ticketmonster-docker postgres`
		
* Start Wildfly app server with Ticket Monster WAR Listening on Port 8888
  * `docker run -d --name server1 -e "DB_HOST=<some hostname or IP>" -p 8888:8080 localhost:5000/dslm/ticket-monster`

* Start Wildfly app server with Ticket Monster WAR Listening on Port 8888 with Docker Container Linking
  * `docker run -d --name server1 --link db:db -e "DB_HOST=db" -p 8888:8080 localhost:5000/dslm/ticket-monster`
  
* Check postgresql data
  * Use pgadmin3 1.2 and connect on port 5432
  * There should be a database called ticketmonster and 12 tables
		
* Scale out app server with Ticket Monster WAR Listening on a Unique Port
  * `docker run -d --name server<X> --link db:db -e "DB_HOST=db" -p <unique port>:8080 localhost:5000/dslm/ticket-monster`

Environment variables available for WildFly + Ticket Monster Container
* TM_DB_HOST=`<hostname or IP address or Docker container link hostname via --link; no default>`
* TM_DB_PORT=`<port; default: 5432>`
* TM_DB_NAME=`<name of database; default: ticketmonster>`
* TM_DB_USER=`<user name to access database; default: ticketmonster>`
* TM_DB_PASSWORD=`<user password to access database; default: ticketmonster-docker>`

* Access Ticket Monster app
  * http://localhost:<unique port>/ticket-monster
