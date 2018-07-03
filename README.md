# Auth Tutorial

This is a tutorial for myself to go through the proccess of adding auth to my own projects using React and Redux.

## Contents
- [File layout](#file-layout)
- [Getting Started](#getting-started)
- [References](#references)


### File Layout


### Getting Started

1. Make sure you can install dev dependencies.
```shell
npm i sodium --save
npm i jsonwebtoken dotenv --save
```

2. Create an api auth route in `server/server.js`.
```shell
const path = require('path')
const express = require('express')
const bodyParser = require('body-parser')

const authRoutes = require('./routes/auth').default

const server = express()
server.use(express.static(path.join(__dirname, 'public')))
server.use(bodyParser.json())

// these are the routes we have created
server.use('/api/v1/auth', authRoutes)

// Default route for non-API requests
server.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, 'public/index.html'))
})

module.exports = server
```

3. Create the route file in `server/routes/auth`.

```shell
const express = require('express')

const router = express.Router()

router.post('/register', register, token.issue)

module.exports = router
```

Create a `users` table in your database. 

```shell
yarn knex migrate:make users
```

example migration file for `users`

```shell
exports.up = (knex, Promise) => {
  return knex.schema.createTableIfNotExists('users', table => {
    table.increments('id')
    table.string('username')
    table.string('hash')
  })
}

exports.down = (knex, Promise) => {
  return knex.schema.dropTableIfExists('users')
}
```

Create the `users` seed data.

```shell
yarn knex seed: make users
```

then 

```shell
yarn knex migrate:latest
yarn knex seed:run
```

### References
[Tutorial from EDA Bootcamp](https://github.com/harakeke-2018/jwt-auth)
