# vue-todo
Simple todo app demonstrating Vue.js/Vuex working with a REST API.

## Requirements
Docker >= 17.05

Docker Compose

On desktop systems like Docker for Mac and Windows, Docker Compose is included as part of those desktop installs.

On Linux systems, first install the Docker for your OS, then install Docker Compose by using curl or pip or whatever the way your heart desires.

## General Information

The containerization of this app was done with the idea of portability and agility in mind. The Alpine Linux was chosen for the OS because of its simplicity, security and efficiency. The containerization was completed as if the resulting stack of services would be eventually deployed on live test/staging and production environments.

It is made up of three Docker containers:
  - A front-end container that serves the client built with Vue.js using nginx.
  - A back-end container that has a REST API that has nginx as a reverse proxy sitting in front of gunicorn, with supervisord being used as the process controller for both nginx and gunicorn.
  - A MySQL database container with data persistence using bind mounts. The ./database folder in the repository is mounted to /var/lib/mysql. Destroying and recreating the database container will not affect the data unless you delete the database folder.

The multi-stage build feature was demonstrated with the Vue.js to show how to optimize the image size.
The build process for the API is a bit more complex with many moving parts, therefore does not use multi-stage builds for now.

This containerization can also cater to developers by relatively simple modifications to the build processes. The dev version of the containers would still run on their own web servers that is more optimized for development such as flask for api and yarn server for the frontend. The folders that has the source code would be mounted inside the containers.


## Stack Summary

### Server / API

Built with:

* Flask (web server)
* Marshmallow (object serialization/deserialization)
* SQLAlchemy (ORM)
* Alembic (migrations)

### Requirements

* python >= 3.6.0
* pipenv >= 11.9.0
* mysql >= 5.7.21

Runs on python-alpine container.

Served by nginx + gunicorn + supervisord.

**Total size:** ~200 MB

## Client

Built with:

* Vue.js (UI framework)
* Vuex (state management)
* vue-router (routing)
* vue-cli (tooling)
* axios (HTTP client)

### Requirements

* node >= 8.6.0
* yarn >= 1.5.1

Runs on nginx-alpine container.

Served by nginx.

**Total size:** ~20 MB

### Database

MySQL 5.7.22 runs on the official mysql image.

**Total size:** ~370 MB

### Deployment / Setup

In the root folder of the project, run:
```
docker build -t vue-todo-api -f Dockerfile.api .

docker build -t vue-todo-frontend -f Dockerfile.frontend .

docker-compose -f docker-compose.yaml up -d

# run migrations (this could be automated too)
docker exec -it vuetodo_vue-todo-api_1 flask db upgrade
docker exec -it vuetodo_vue-todo-api_1 python seed.py

```
That's it! You can hit http://localhost:8081/ on your browser.

#### Some pain points
- When the JavaScript runs in a browser (outside of Docker) you can not use app because that is only available inside the Docker network (via the embedded DNS server). So it had to be hardcoded for the moment. Possible solution is a front-end proxy.

```
const ax = axios.create({
  baseURL: 'http://localhost:8081/',
});
```

- While building the API, there is a point that you need mysql_config executable in order to install mysql related python packages which requires a full-blown installation of the mariadb package which is quite large for container standards. The response to that is that the build process installs all the necessary packages to set up the framework then purges all of them to optimize the size of the final image.

- The use of virtualenv inside a docker container seemed unnecessary and its behavior added another layer of complexity to the automation and build process. As a result, although pipenv was still used as a package manager, it was installed with --system, meaning system python is used, and the migrations and seeds are run without pipenv as well.
