## Description

Dockerizing [This Go Web framework Twitter-like API Recipe](https://echo.labstack.com/cookbook/twitter/)

The api uses [MongoDB](https://github.com/mongodb/mongo) for persistence and JWT for authentication.

## Install and Run

The easiest way to run this is by creating the necessary Docker infrastructure (containers, volumes, network) using the provided docker-compose.yml file:

```
$ docker-compose up -d --build
```

Once you're done booting everything up, use cURL or postman etc. to try out the api.

After you're done, shut everything done using:
```
$ docker-compose down
```

## Docker compose

The `docker-compose` file creates the following services:

### mongo
Uses the `mongodb` docker volume for its data (so that we don't lose the data every time we start/restart the container)

### api
Starts the api container.  This is a Go application that uses the Echo framework to expose a Twitter-like api that allows users to signup, login, follow other users, post messages, get their feed.

The `api` container used [Fresh](https://github.com/gravityblast/fresh) to automatically recompile/restart the server every time code changes in the container.  This, in combination with mounting the dev environment's `api` folder onto the containers `WORKDIR` will make development easier (even though Go is a compiled language)
```
volumes:
      - './api:/app'
```

### nginx
Fronts the web server.

The `docker-compose` file also creates a `mongodb` volume for persisting the mongodb data independent of the container lifecycle.

## Go App

The Go app uses the following dependencies:

```
$ go get github.com/labstack/echo/v4
$ go get gopkg.in/mgo.v2
$ go get github.com/golang-jwt/jwt
```

For authentication, it returns a JWT token which is subsequently provided to the other endpoints as part of the `Authorization` header value.

## Sample cURL requests

### Signup
```
curl \
  -X POST \
  http://localhost:2323/signup \
  -H "Content-Type: application/json" \
  -d '{"email":"ara@juljulian.com","password":"shhh!"}'
```

### Login
```
curl \
  -X POST \
  http://localhost:2323/login \
  -H "Content-Type: application/json" \
  -d '{"email":"ara@juljulian.com","password":"shhh!"}'
```

### Follow
```
curl \
  -X POST \
  http://localhost:2323/follow/58465b4ea6fe886d3215c6df \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE0ODEyNjUxMjgsImlkIjoiNTg0NjViNGVhNmZlODg2ZDMyMTVjNmRmIn0.1IsGGxko1qMCsKkJDQ1NfmrZ945XVC9uZpcvDnKwpL0"
```

### Post
```
curl \
  -X POST \
  http://localhost:2323/posts \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE0ODEyNjUxMjgsImlkIjoiNTg0NjViNGVhNmZlODg2ZDMyMTVjNmRmIn0.1IsGGxko1qMCsKkJDQ1NfmrZ945XVC9uZpcvDnKwpL0" \
  -H "Content-Type: application/json" \
  -d '{"to":"61c3d30416e72f00dee3ac1d","message":"hello"}'
```

### Feed
```
curl \
  -X GET \
  http://localhost:2323/feed \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2NDA0NjYxODcsImlkIjoiNjFjMmJjMWYyYmVlYzkwNGI2NGNjOWRiIn0.8-lPqWVIbYgD9GGEit0KbaOicJBO1bh3F7zcs0PCpG0"
```

## Mongo

This is the [MongoDB Go Library](https://pkg.go.dev/gopkg.in/mgo.v2?utm_source=godoc) I'm using.  It's been deprecated unfortunately.


### Logging in to the mongo shell from the running container

```
  $ docker container ls
  ...grab the <mongo container id>
$ docker exec -it <mongo container id> /bin/sh
# mongo
```

### Mongo shell commands
```
> show dbs
> use <db name>
> show collections
> db.users.find()
> db.users.find().pretty()
> db.users.find().limit(1)
> db.users.find().pretty().limit(1)

> db.posts.find().pretty()
```

## Docker

  ```
  $ docker volume ls
  ```