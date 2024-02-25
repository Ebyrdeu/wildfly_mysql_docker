# Dockerized MySQL and WildFly Setup

## Overview
![image](https://github.com/Ebyrdeu/wildfly_mysql_docker/assets/42122071/99ecb922-fe70-4c7e-baa7-31c4886c9114)

Setting up a Docker environment that includes a MySQL database and a WildFly application server, supporting up to JDK 20. This configuration is suitable for development, testing, and production environments requiring a Java EE application server and a relational database.

## Requirements

- Docker and Docker Compose must be installed on your system.
- Basic familiarity with Docker and Docker Compose is assumed.
- For deploying applications on the WildFly server, an application WAR file is required.

## Configuration

The provided `docker-compose.yml` file orchestrates two main services:

1. `db`: A service running a MySQL database.
2. `wildfly`: A WildFly application server service that depends on the `db` service.

### MySQL Service Configuration

- **Image**: Uses `mysql:8.3.0`.
- **Environment Variables**: Sets the `MYSQL_ROOT_PASSWORD` using an environment variable `DB_PASSWORD`.
- **Ports**: Maps the container's port `3306` to the host's port `3306`.
- **Volumes**: Utilizes a volume `db-data` for persisting database data.
- **Restart Policy**: Configured to `unless-stopped`.

### WildFly Service Configuration

- **Build Context**: Defined as the current directory.
- **Dockerfile**: Specifies the build instructions for the WildFly service, ensuring it's built with JDK 20 support.
- **Args**: Specifies the Arugments of JDK for Wildfly image, up to version 64 support.
- **Ports**: Exposes port `8080` on the host, mapping it from the same port inside the container.
- **Dependencies**: Specifies a dependency on the `db` service, ensuring the database is ready before the application server starts.
- **Restart Policy**: Set to `unless-stopped`.

### Dockerfile Instructions for WildFly

1. **Base Image**: Utilizes `quay.io/wildfly/wildfly:latest-jdk20` to ensure JDK 20 compatibility.
2. **MySQL JDBC Connector**: Incorporates the MySQL JDBC connector jar into the WildFly server.
3. **Configuration Steps**: Includes adding a MySQL module, configuring a MySQL JDBC driver, and setting up a datasource for MySQL in the WildFly server.

### Environment Configuration

Required configurations for the WildFly server include:

- `DB_NAME`: Name of the database.
- `DB_USERNAME`: Username for database access.
- `DB_PASSWORD`: Password for the database user.
- `DB_CONNECTION_URL`: JDBC connection URL for the database.

### Additional Environment Variable Configuration

In addition to the previously mentioned environment variables, it's essential to understand how these variables interconnect, especially the `DB_CONNECTION_URL`. This variable is crucial for configuring the connection between the WildFly application server and the MySQL database. The format for this variable is as follows:
```env
DB_CONNECTION_URL=jdbc:mysql://db:3306/${DB_NAME}
```
Here, `db` refers to the service name of the MySQL database defined in `docker-compose.yml`. 
This naming is vital because Docker Compose uses the service names to facilitate network communication between containers. 
By using db as the hostname, you instruct the WildFly server to connect to the MySQL database using the internal network established by Docker Compose.

Furthermore, you might consider adding an `DB_NAME` for automatically creating a database with a specified name.

## Deployment Instructions

### Preparing for Deployment

1. **Environment Variables**:
    - Create a `.env` file in the same directory as your `docker-compose.yml`.
    - Specify `DB_PASSWORD`, `DB_CONNECTION_URL`, `DB_NAME` and `DB_USERNAME`.

2. **Application Packaging**:
    - If deploying a Java application, execute `mvn clean package` to build your WAR file.
    - For non-Java applications, ensure your WAR file is prepared and located in the target directory.

3. **Building and Starting Services**:
    - Execute `docker-compose up --build` to initiate the build process and start the services. The WildFly server will be accessible at `http://localhost:8080`.

## Accessing the Services

- **WildFly Server**: Available at `http://localhost:8080` for accessing the deployed applications.
- **MySQL Database**: Connect using the `DB_USERNAME` and `DB_PASSWORD` at `localhost:3306`.

## Stopping the Services

- To halt and remove the containers, run `docker-compose down`. Adding the `-v` option will also remove the volumes, clearing any persisted data. To also remove locally built images, add the option `-rmi local`.
