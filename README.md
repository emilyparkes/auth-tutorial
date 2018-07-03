# Auth Tutorial

This is a tutorial for myself to go through the proccess of adding auth to my own projects using React and Redux.

## Contents
- [File layout](#file-layout)
- [Getting Started](#getting-started)
- [References](#references)


### File Layout


### Getting Started

1. Make sure you can install the following dev dependencies.  
2. Create an api auth route in `server/server.js` that will link to the file in the next step.
3. Create the route file in `server/routes/auth.js`.  
The route should have a POST `/api/v1/auth/register` route that accepts a JSON object with `username` and `password` properties. The `/api/v1/auth` part is defined in `server/server.js`.  
You can leave the route empty at this point. We need step XX to complete it.

4. Create your database.
   - Create a `users` table in your database. 
   - Complete your migration file for `users`.
   - Create the `users` seed data.
   - Then make sure it's up and running in your database.
5. Create a `server/db/users.js` file. We need a way to save the new user to the database and we should also make sure we can check that the username is available. Don't forget to export these functions. 
   - createUser(newUser:{username, password})
   - userExists(username)
6. Let's return to our `/register` route in `server/routes/auth.js`.  
Add the check for username availability and add the new user if it's available. Be sure to require the new server/db/users.js file and use its functions to complete the register function.
   - If the username is already taken, send back a status 400 and this JSON object: {message: 'User exists'}.
   - If the username is available, add the user and respond with a status 201.
   - If there is an error, respond with a status 500 and this JSON object: {message: err.message}.
7.
8.



### References
[Tutorial from EDA Bootcamp](https://github.com/harakeke-2018/jwt-auth)
