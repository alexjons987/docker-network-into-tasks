# Docker Network - Introduction tasks
## Task 1: Docker Network – BusyBox communication
### Background
You will now explore Docker Networks by starting two containers that will be able to communicate with each other.
We will use BusyBox, which is a small container with built-in tools such as httpd and wget.

The goal is to understand:
* How user-defined bridge networks work
* How Docker DNS allows containers to reach each other by name
* How to test container-to-container communication

### Objectives
1. Create your own Docker network
2. Start two BusyBox containers in the network
3. Start an HTTP server in the server container
4. Communicate from the client container with the server container via DNS
5. Show the difference to the default bridge network

Hint: The container to be started is called busybox

1. Create a user-defined network: `docker network create mynet`
2. Start a server container: `docker run -d --name server --network mynet busybox sh -c "while true; do sleep 1000; done"`
3. Start a client container: `docker run -d --name client --network mynet busybox sh -c "while true; do sleep 1000; done"`
4. Start an HTTP server in the server container, see example below:
* `docker exec server sh -c "echo 'Hello from the BusyBox server!' > index.html"`
* `docker exec server sh -c "httpd -f -p 8080 &"`

5. Test communication from the client with Docker network hint (wget): `docker exec client wget -qO- http://server:8080`
6. Bonus: show that ping does not work in default bridge:
* `network disconnect mynet client` Disconnect `client` from `mynet`
* `docker exec client ping -c 3 server` Try to ping `server` from `client` (it will fail as we disconnected)
* `network connect mynet client` Reconnect `client` to `mynet`
* `docker exec client ping -c 3 server` Try to ping `server` from `client` (it will now work as we are connected)

## Task 2: Docker Network – MySQL and connect from localhost
### Background
You will now start a MySQL database in Docker that is accessible from your own computer (localhost).
The aim is to understand:
* How to expose a container via port mapping
* How user-defined networks work
* How to connect with credentials and networks

### Objectives
1. Create your own Docker network:
2. Start a MySQL container with user, password and database
3. Expose the MySQL port to the host computer
4. Connect to the database from localhost
5. Optional: connect from another container via the network

### Steps
1. Create a user-defined network: `docker network create mynet`
2. Start the MySQL container: `docker run -d --name mysql-db --network mynet -e MYSQL_ROOT_PASSWORD=rootpass -e MYSQL_DATABASE=demo_db -e MYSQL_USER=demo_user -e MYSQL_PASSWORD=demo_pass -p 3307:3306 mysql:8.0`
3. Check that the database is started: `docker ps` and `docker logs -f mysql-db` to follow logs
4. Connect from your computer with any MySQL client or spring-boot: I chose **MySQL Workbench 8.0**
* Hostname: `127.0.0.1`
* Port: `3307`
* Username: `demo_user`
* Password: `demo_pass`
5. Test SQL commands:
```sql
USE demo_db;
CREATE TABLE message (
  id INT PRIMARY KEY AUTO_INCREMENT,
  text VARCHAR(255)
);
INSERT INTO message(text) VALUES ("Hello from MySQL in Docker!");
SELECT * FROM message;
```
6. Bonus: connect from another container in the same network.
