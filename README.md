# Api Auth Nestjs template

This repository provides a base template to develop APIS in Nestjs with the authentication and authorization functions with user roles already implemented. Designed to be reusable in future projects, this template offers a robust structure that facilitates the rapid creation of safe and scalable applications.

## Getting started

### Requirements

Before starting, make sure you have at least those components on your workstation:

- [NodeJS 20.x](https://nodejs.org/en/download/package-manager)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [Nest CLI](https://docs.nestjs.com/cli/overview#installation) command line interface tool

```sh
# windows
yarn add -g @nestjs/cli
# Linux and Mac
sudo yarn add -g @nestjs/cli
```

> [!NOTE]
> For a correct installation of the Nest Cli in Windows it is necessary to open the terminal as administrator and in Linux and Mac use the sudo command `sudo npm install -g @nestjs/cli`

### Project setup

1. Start cloning this repository.

```sh
git clone https://github.com/ayunierto/api-auth-nestjs-base.git my-project
```

2. The next thing will be to install all the dependencies of the project.

```sh
cd ./my-project
npm install
# or
yarn
```

3. For this application to work correctly, it is necessary to configure the following environment variables, create a new `.env` file.

```sh
cp .env.example .env
```

- **`DB_PASSWORD`**: The password used to connect to the database. Be sure to use a safe password.  
  _Example:_ `MyPassword`

- **`DB_NAME`**: The name of the database that the application will use to store and recover information.  
  _Example:_ `dbname`

- **`DB_HOST`**: The server address where the database is housed. For local development environments, it can be `localhost`.  
  _Example:_ `localhost`

- **`DB_PORT`**: The port through which the application will connect to the database. The default value for postgresql is `5432`.  
  _Example:_ `5432`

- **`DB_USERNAME`**: The username of the database. Generally, this will be the main user to access the database.  
  _Example:_ `postgres`

- **`JWT_SECRET`**: The secret key used to sign and verify the tokens JWT (JSON Web tokens) in the authentication process. It is important that this key is complex and remains safe.  
  _Example:_ `thisismysecretpasswordforjwt`

### Create and start database container

```sh
docker-compose up -d
```

### Compile and run the project

```bash
# development
$ yarn run start

# development watch mode
$ yarn run start:dev

# production mode
$ yarn run start:prod
```
