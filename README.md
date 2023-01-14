
# Music Library v5

Finally it's time for the backend to be dockerized as well.

## New project structure

This project has a different structure from the previous ones: the root folder contains:
- the `.git` repository folder (meanings this is the root of the repo),
- `.gitignore`,
- two folders, one for the backend and one for the database,
- as well as this `README` file.

You are going to create one more file in the root folder: `docker-compose.yml`.

In it, you're going to define two containers: one for the backend and one for the database.

The backend will no longer be served from your host computer. Instead, it will run inside the `backend` container.

It will still use the database from the other container, called `database`, but for it to do that, you need to update the url of its datasource in `application.yml`.

Before, it was `jdbc:postgresql://localhost:5432/music-library`. However, now it cannot reach it through `localhost`, because the other database container is isolated from the backend container. The two containers are in a virtual network, and they can reach each other using their names as the host name. So the `backend` container can reach the `database` container through the host name `database`. More specifically, it can reach the `database` container's Postgres database through `database:5432`. So make sure to update the datasource url to the appropriate value.

## The backend source

The root of the backend Maven project is inside the `backend` folder. When you want to open the project in IntelliJ, you should open the `.pom` file inside that folder.

All the source files, including resources (like `application.yml`) and tests, are inside the `backend` folder.

<br/>

---

<br/>

## The docker-compose file

Define two services named `backend` and `database`.

### **backend**
- start from image `maven`
- map host port 8091 to container port 8091
- mount two volumes:
  - host folder `./backend` to container path `/app` (this is where the backend app's source will live)
  - host folder `./backend/.mvn_repo` to container path `/root/.m2/repository` (this is where the Maven dependencies will live)
- set the `working_dir` key to value `/app`
- set the command to run when the container starts to `"mvn spring-boot:run"` (including the quotes) (this will actually build the app and start the Tomcat server)
- set the restart policy to `unless-stopped`

### **database**
- start from image postgres
- no need to map a container port to a host port, because the `backend` container can reach the `database` container directly through port 5432
- mount two volumes:
  - host file `./database/init.sql` to container path `/docker-entrypoint-initdb.d/init.sql` (this will initialize the database with some Artists and Tracks)
  - host folder `./database/data` to container path `/data/music-library` (this is where the database's data will actually live)
- set the environment variables to the following (*just declare a key* `environment:` *and add the following four lines as its children; you can use the `docker-compose` from the Project Assignment for reference*):
  - `POSTGRES_DB: music-library`
  - `POSTGRES_USER: jensenyh`
  - `POSTGRES_PASSWORD: losenord`
  - `PGDATA: /data/music-library`
  - set the restart policy to `unless-stopped`