# Auth Tutorial

This is a tutorial for myself to go through the proccess of adding auth to my own projects using React and Redux.

## Contents
- [File layout](#file-layout)
- [Getting Started](#getting-started)
- [Issue Token](#a-issue-token)
- [Apply Token](#b-apply-token)
- [References](#references)


### File Layout


### Getting Started
Make sure you can install the following dev dependencies.  
- sodium (this takes a while)
- passport
- express-jwt
- jsonwebtoken
- dotenv
- jwt-decode
   
```shell
~ yarn add sodium passport express-jwt jsonwebtoken dotenv jwt-decode
or
~ npm i sodium passport express-jwt jsonwebtoken dotenv jwt-decode
```

#### A. Issue Token
1. Create an api auth route in `server/server.js` that will link to the file in the next step.

2. Create the route file in `server/routes/auth.js`.  
The route should have a POST `/register` route that accepts a JSON object with `username` and `password` properties (as well as any other information you need for a user to register). The `/api/v1/auth` part is defined in `server/server.js`.  
You can leave the route empty at this point. We need step A6 to complete it.

3. Create your register form component and client side route.
4. Create your database so we have a place to store the user info.
   - Create a `users` table in your database. 
   - Complete your migration file for `users`.
   - Create the `users` seed data.
   - Then make sure it's up and running in your database.

5. Create a `server/auth/` folder. We'll use this folder to hold some auth-related helper functions.  
We need a way of saving a password not in clear-text so we will use the `sodium` package to help create our secret password in a new `hash.js` file in our new folder. The hash module should export a `generate` function that takes the clear-text password as its only parameter, and use the `sodium api` to return a hash of that password.  
We will use `generate` in the next step.
 
6. Create a `server/db/users.js` file. We now need a way to save the new user to the database with the hash password and we should also make sure we can check that the username doesn't already exist. Don't forget to export these functions.
So call `generate` in the `createUser` function, and import hash from `server/auth/hash`.
   - createUser(newUser:{username, password})
   - userExists(username)
   
7. Let's return to our `/register` route in `server/routes/auth.js`.  
Require your two new functions created in the `server/db/users.js` file and use its functions to complete the register function for your register route.
   - If the username is already exists, send back a status 400 and this JSON object: {message: 'User exists'}.
   - If the username is available, create the user and respond with a status 201.
   - If there is an error, respond with a status 500 and this JSON object: {message: err.message}.

8. The last step in registering a new user is to create and issue a JSON Web Token (JWT) the client can use when making future requests to protected endpoints. To ensure a JWT is valid, it is signed with a _secret string_. We normally keep that string in an environment variable on the server.
Create a `.env` file in the root of your project. Use this _example_ until you deploy.  
`JWT_SECRET=a31sl86dfk862jsd54lfk123lksjhd92`.  
> When you deploy your project **CHANGE the `.env` file** to `.env.example` and create a new `.env` with a new 20 character code. Put .env into your `.gitignore`. You **DO NOT** want the secret to be publically displayed in your repo.

9. To enable the `dotenv` package so the environment variables are available, require its config function as early as possible in the server startup code `require('dotenv').config()` (e.g. at the top of server/index.js).

10. The JWT we issue should contain the user's ID and username, so we need an object that contains these properties. Export a `getUserByName` function from `server/db/users.js` that takes a username, and returns the user data.

11. Let's put the code for signing and issuing the token in a new `server/auth/token.js` file. This file should export an `issue` function. Because we're going to use it as Express middleware, it should have this signature:

issue(req:Request, res:Response, next:Function)

This function should use the username property on the request along with the new `getUserByName` function in `server/db/users.js` to retrieve the user from the database and create a JWT. The JWT secret is available from process.env.JWT_SECRET. The sign function from the jsonwebtoken package has this signature:

sign(user:Object, secret:string, options:Object)

The options parameter has an expiresIn property that is in zeit/ms format.

12. The last step to implement user registration is to add the `token.issue` middleware function to the register route in `server/routes/auth`. Specifically,
    - Import `server/auth/token`.
    - Add token.issue as the 3rd parameter to router.post('/register')
    - Add next as the 3rd parameter to the register function
    - After the user is created, call .then(() => next()) instead of the res.json call
13. Now use Postman to test registering new users. You should see the JWT in the response body.

#### B. Apply Token
We must be able to verify the authenticity of the token provided before we trust it. We can do this because it was signed with a secret (JWT_SECRET). Once we know we can trust it, we can decode it to extract the user's ID and username, which we will likely need to fulfil the request.
The token will come in on the Authorization header. The `express-jwt` package is capable of getting the token out of the header, verifying its authenticity and populating `req.user` with the contents of the decoded token. We just need to give it the secret we used to sign it.

1. To use `express-jwt` let's expose a `decode` function from our `server/auth/token.js` file. The decode function will be used as Express router middleware on all routes that need authentication. Here's how we'll use it:

```javascript
// server/routes/ROUTES-YOU-NEED-AUTH-TO-VISIT.js
const token = require('../auth/token')

router.get('/:ROUTE-YOU-NEED-AUTH-TO-VISIT', token.decode, (req, res) => {
  // now req.user will contain the contents of our token
})
```

2. That means the signature of the decode function should look like this:

decode(req:Request, res:Response, next:Function)

Show start of `decode` function code

3. The express-jwt package exports a function. Let's name it verifyJwt. We're going to call it inside of our decode function. The verifyJwt function accepts an object as a parameter where each property is an option. We only need to provide the secret option/property and its value should be a getSecret function.

Show more of `decode` function's code

4. The getSecret function is described in the express-jwt docs here. Basically, it takes 3 parameters, the 3rd of which is an error-first callback that accepts the secret (process.env.JWT_SECRET) as the 2nd parameter.

Show code for `getSecret` function

5. Lastly, the verifyJwt function returns a function with the same parameters as our decode function. So let's just pass the parameters through to that function call.

Show all new code in `server/auth/token`

6. To test this is working correctly, let's create an /api/v1/auth/username route that returns the username of the requester. The username is encoded into the token so it will be on the req.user object after our token.decode function is applied.

7. Start with creating a GET /username route in server/routes/auth.js that looks similar to the example at the beginning of step B2. In the route, respond with a JSON object that has a username property with a value of req.user.username.

Show code

8. Make sure you've registered a new user and captured the JWT token in the response body. Configure postman with the Authorization header and issue a GET request to `localhost:/3000/api/v1/auth/username`. If all is well, the response will be:

```
{
  "username": "your username"
}
```

Congratulations! You're verifying JWT tokens!




### References
[Tutorial from EDA Bootcamp](https://github.com/harakeke-2018/jwt-auth)
