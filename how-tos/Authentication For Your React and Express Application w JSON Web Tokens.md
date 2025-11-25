---
tags:
  - json
  - react
  - express
  - js
  - node
---
any web applications today wouldn’t be complete without authentication; allowing users have an identity on your website.

Authentication allows you to cater content specifically to your user and allow users to personalize their settings and experience on your application.

This guide will demonstrate an approach to implement authentication for a web application built using [React](https://reactjs.org/) and [react-router](https://reacttraining.com/react-router/) on the frontend and [node.js](https://nodejs.org/en/) using [Express](http://expressjs.com/) and [MongoDB](https://www.mongodb.com/) with [Mongoose](https://mongoosejs.com/) on the backend.

I kept this approach relatively simple so that it’s both easy to understand and won’t require many changes to incorporate into an existing application.

> To look at a fully functioning example, you can check out [this Github repository](https://github.com/faizanv/react-auth-example)
# **Table of Contents**

## [Boilerplate Application](https://faizanv.medium.com/authentication-for-your-react-and-express-application-w-json-web-tokens-923515826e0#61f1) (skip if you already have a working project)

## [Backend](https://faizanv.medium.com/authentication-for-your-react-and-express-application-w-json-web-tokens-923515826e0#ff1c)

- [User Model](https://faizanv.medium.com/authentication-for-your-react-and-express-application-w-json-web-tokens-923515826e0#18d5)
- [Secure Passwords](https://faizanv.medium.com/authentication-for-your-react-and-express-application-w-json-web-tokens-923515826e0#3c21)
- [Authentication](https://faizanv.medium.com/authentication-for-your-react-and-express-application-w-json-web-tokens-923515826e0#beb6)
- [Issuing Tokens](https://faizanv.medium.com/authentication-for-your-react-and-express-application-w-json-web-tokens-923515826e0#5f52)
- [Protecting Routes (express)](https://faizanv.medium.com/authentication-for-your-react-and-express-application-w-json-web-tokens-923515826e0#4010)
- [Verifying Tokens](https://faizanv.medium.com/authentication-for-your-react-and-express-application-w-json-web-tokens-923515826e0#f07a)
## [Frontend](https://faizanv.medium.com/authentication-for-your-react-and-express-application-w-json-web-tokens-923515826e0#3432)

- [Login Page](https://faizanv.medium.com/authentication-for-your-react-and-express-application-w-json-web-tokens-923515826e0#93c3)
- [Saving Token](https://faizanv.medium.com/authentication-for-your-react-and-express-application-w-json-web-tokens-923515826e0#6563)
- [Protecting Routes (react-router)](https://faizanv.medium.com/authentication-for-your-react-and-express-application-w-json-web-tokens-923515826e0#e320)
# Boilerplate Application

> Note: This section can be skipped if you already have a working application

To get the boilerplate application going I used [this guide](https://dev.to/loujaybee/using-create-react-app-with-express) which uses [create-react-app](https://github.com/facebook/create-react-app) with some tweaks to add an express server. Then I followed [this guide](https://reacttraining.com/react-router/web/example/basic) to get react-router running.

> _Keep in mind I am using npm for this tutorial but this can be interchanged with yarn._

I also made sure I had [Mongoose](https://mongoosejs.com/) installed and connected in my server code and MongoDB running on my computer:

```c
const mongoose = require('mongoose');const mongo_uri = 'mongodb://localhost/react-auth';mongoose.connect(mongo_uri, function(err) {  
  if (err) {  
    throw err;  
  } else {  
    console.log(`Successfully connected to ${mongo_uri}`);  
  }  
});
```

Then, using [Express](http://expressjs.com/), I created two routes that look like this:

```c
app.get('/api/home', function(req, res) {  
  res.send('Welcome!');  
});app.get('/api/secret', function(req, res) {  
  res.send('The password is potato');  
});
```

For the frontend I created two components with their own corresponding routes which will fetch those messages from the back-end:

```
import React, { Component } from 'react';  
import { Link, Route, Switch } from 'react-router-dom';  
import Home from './Home';  
import Secret from './Secret';  
export default class App extends Component {  
  render() {  
    return (  
      <div>  
        <ul>  
          <li><Link to="/">Home</Link></li>  
          <li><Link to="/secret">Secret</Link></li>  
        </ul>        <Switch>  
          <Route path="/" exact component={Home} />  
          <Route path="/secret" component={Secret} />  
        </Switch>  
      </div>  
    );  
  }  
}
```

And here is an example of my `Home` component (the `Secret` component is almost identical):

```
export default class Home extends Component {  
  constructor() {  
    super();  
    //Set default message  
    this.state = {  
      message: 'Loading...'  
    }  
  }  componentDidMount() {  
    //GET message from server using fetch api  
    fetch('/api/home')  
      .then(res => res.text())  
      .then(res => this.setState({message: res}));  
  }  render() {  
    return (  
      <div>  
        <h1>Home</h1>  
        <p>{this.state.message}</p>  
      </div>  
    );  
  }}
```

Which ends up looking like this:

![](https://miro.medium.com/v2/resize:fit:600/1*9i2r2Llt_MnQUw2eLqzwDw.gif)

Now with this working we can get to the fun part!

# Backend

## User Model

To get started we need to create a MongoDB/Mongoose model for a User object. Here is an example of one that I created:

```
// User.js  
const mongoose = require('mongoose');const UserSchema = new mongoose.Schema({  
  email: { type: String, required: true, unique: true },  
  password: { type: String, required: true }  
});module.exports = mongoose.model('User', UserSchema);
```

Pretty simple, this allows us to create User objects which have their own unique email and password fields which can then be saved and retrieved from MongoDB to authenticate users.

## Secure Passwords

Obviously we can’t just store passwords in plain text, that’s [how bad things happen.](https://www.neowin.net/news/t-mobile-austria-stores-passwords-in-plain-text-because-its-security-is-amazingly-good)

To secure our passwords we will use a nice little library called [bcrypt](https://github.com/kelektiv/node.bcrypt.js). This will allow us to hash our passwords (if you don’t know what that means [read this](https://auth0.com/blog/hashing-passwords-one-way-road-to-security/)).

`npm install --save bcrypt`

Once [bcrypt](https://github.com/kelektiv/node.bcrypt.js) is installed, we can [add a hook](https://mongoosejs.com/docs/api.html#schema_Schema-pre) to our `User` schema to hash passwords before we save them to our database:

```
// User.js  
const mongoose = require('mongoose');  
const bcrypt = require('bcrypt');const saltRounds = 10;const UserSchema = new mongoose.Schema({  
  email: { type: String, required: true, unique: true },  
  password: { type: String, required: true }  
});UserSchema.pre('save', function(next) {  
  // Check if document is new or a new password has been set  
  if (this.isNew || this.isModified('password')) {  
    // Saving reference to this because of changing scopes  
    const document = this;  
    bcrypt.hash(document.password, saltRounds,  
      function(err, hashedPassword) {  
      if (err) {  
        next(err);  
      }  
      else {  
        document.password = hashedPassword;  
        next();  
      }  
    });  
  } else {  
    next();  
  }  
});module.exports = mongoose.model('User', UserSchema);
```

If you’re curious about what `saltRounds` is used for, [read here](https://stackoverflow.com/questions/46693430/what-are-salt-rounds-and-how-are-salts-stored-in-bcrypt).

Now that we have our `User` object setup we can test it out and create some users. To do so, I’ll create another express route like so:

```
// Import our User schema  
const User = require('./models/User.js');...// POST route to register a user  
app.post('/api/register', function(req, res) {  
  const { email, password } = req.body;  
  const user = new User({ email, password });  
  user.save(function(err) {  
    if (err) {  
      res.status(500)  
        .send("Error registering new user please try again.");  
    } else {  
      res.status(200).send("Welcome to the club!");  
    }  
  });  
});
```

And then we can test it using [Postman](https://www.getpostman.com/) or in this case I’ll just do a simple cURL on the command line:

```sh
curl -X POST \  
  [http://localhost:3000/api/register](http://localhost:3000/api/register) \  
  -H 'Content-Type: application/json' \  
  -d '{  
 "email": "[me@example.com](mailto:me@example.com)",  
 "password": "mypassword"  
}'

```
With which we should get back a nice message saying `Welcome to the club!` and if we check our MongoDB records, we should see something like this:

```
{  
 "_id" : ObjectId("5b89a402ec9ad51db3d37c44"),  
 "email" : "[me@example.com](mailto:me@example.com)",  
 "password" : "$2b$10$j/e4G.D1HzW1HjlkC9NclOQDDiIsLCm09Euj9QGvTzJNTOLmI9Tpm",  
 "__v" : 0  
}

```
> Note: Don’t actually share production data hashed or not

## Authentication

Now that we have some users saved to the database, we need a way to authenticate them against our database.

To do this we will [add a method](https://mongoosejs.com/docs/guide.html#methods) to our `User` schema that will take in a password, as a string, and use [bcrypt](https://github.com/kelektiv/node.bcrypt.js) tell us if it's the correct password for that `User`

```
UserSchema.methods.isCorrectPassword = function(password, callback){  
  bcrypt.compare(password, this.password, function(err, same) {  
    if (err) {  
      callback(err);  
    } else {  
      callback(err, same);  
    }  
  });  
}
```

## Issuing Tokens

Now that we have all the tools in place, we can start issuing tokens to clients.

> Note: Here is a quick summary of [token-based authentication](https://stormpath.com/blog/token-authentication-scalable-user-mgmt)

First step is we need a `secret` string to use when signing the tokens. For the sake of this example I will simply define it at the top of my server file

`const secret = 'mysecretsshhh';`

> For any real application, you should keep your secret an actual secret [using environment variables](https://github.com/motdotla/dotenv) or [some other method](https://www.digitalocean.com/community/tutorials/an-introduction-to-managing-secrets-safely-with-version-control-systems) and make sure you [DO NOT commit it to version control](https://git-scm.com/docs/gitignore) if you happen to be using git.

Next we need to install the [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken) library which will allow us to issue and verify JSON web tokens:

```
npm install --save jsonwebtoken
```

Finally we can create a new express route which, given an email and password, will find a `User` with the given email and verify that the given password is correct. If the password is correct, we will issue a signed token to the requester:

```
const jwt = require('jsonwebtoken');app.post('/api/authenticate', function(req, res) {  
  const { email, password } = req.body;  
  User.findOne({ email }, function(err, user) {  
    if (err) {  
      console.error(err);  
      res.status(500)  
        .json({  
        error: 'Internal error please try again'  
      });  
    } else if (!user) {  
      res.status(401)  
        .json({  
          error: 'Incorrect email or password'  
        });  
    } else {  
      user.isCorrectPassword(password, function(err, same) {  
        if (err) {  
          res.status(500)  
            .json({  
              error: 'Internal error please try again'  
          });  
        } else if (!same) {  
          res.status(401)  
            .json({  
              error: 'Incorrect email or password'  
          });  
        } else {  
          // Issue token  
          const payload = { email };  
          const token = jwt.sign(payload, secret, {  
            expiresIn: '1h'  
          });  
          res.cookie('token', token, { httpOnly: true })  
            .sendStatus(200);  
        }  
      });  
    }  
  });  
});
```

> View full code on [Github](https://github.com/faizanv/react-auth-example)

This looks like a lot going but basically we are checking if we have a user registered with the given email and if we do, then we check if the given password is correct and issue a token to the client if it is.

One particular things to note here is that when we issued the token, we set it as a `cookie` and set the `httpOnly` flag to `true` . This method of issuing tokens is ideal for a browser environment because its sets an [httpOnly cookie](https://blog.codinghorror.com/protecting-your-cookies-httponly/) which helps secure the client from certain vulnerabilities such as [XSS](https://www.techrepublic.com/blog/it-security/what-is-cross-site-scripting/).

There are definitely other ways to issue tokens depending on the client but this particular method works well for browser authentication where we aren’t explicitly using any data stored within the token itself on our client and using it purely as a means of authentication.

## Protecting Routes (express)

Now that we have established a way to issue signed token to authorized users, we need to define what routes in our application are off limits to non-authenticated users. In this case, we want our `'/secret'` route to only be accessible if the requesting client has a valid token.

First, let’s make sure that we have [cookie-parser](https://github.com/expressjs/cookie-parser) installed so that can express can parse cookies passed by our browser:

```sh
npm install --save cookie-parser
```

and let’s add the middleware to our express setup:

```
// server.js  
const cookieParser = require('cookie-parser');
...
app.use(cookieParser());
```

Next, we will create our own custom [express middleware](https://expressjs.com/en/guide/using-middleware.html) which will sit in between a request and a protected route and verify if the request is authorized.

This middleware function will look for the token in the cookies from the request and then validate it.

> Note: I’ve hardcoded our `secret` here again which is bad practice and I’m only doing so to keep the example simple

```c
// middleware.jsconst jwt = require('jsonwebtoken');  
const secret = 'mysecretsshhh';const withAuth = function(req, res, next) {  
  const token = req.cookies.token;  if (!token) {  
    res.status(401).send('Unauthorized: No token provided');  
  } else {  
    jwt.verify(token, secret, function(err, decoded) {  
      if (err) {  
        res.status(401).send('Unauthorized: Invalid token');  
      } else {  
        req.email = decoded.email;  
        next();  
      }  
    });  
  }  
}module.exports = withAuth;
```

> View full code on [Github](https://github.com/faizanv/react-auth-example)

Finally we can use this middleware whenever we want to have a protected route by simply editing its route configuration to use the new middleware we just wrote:

```c
// server.js  
const withAuth = require('./middleware');...app.get('/api/secret', **withAuth**, function(req, res) {  
  res.send('The password is potato');  
});

```
We can test this change with some more cURL commands or we can just take a look at our React application which should now look like this:

![](https://miro.medium.com/v2/resize:fit:600/1*VnhuOCLpAll8nnaIsaQpIw.gif)

Our secret is now off limits

As you can see, our secret is now actually a secret that will require a valid signed JSON web token to view.

## Verifying Tokens

It will be a lot more apparent later why this is useful, but sometimes we need a way to simply ask our server if we have a valid token saved to our browser cookies.

For this, we are going to just create a simple route will return a `200` HTTP status if our requester has a valid token:

```c
// server.js
...
app.get('/checkToken', withAuth, function(req, res) {  
  res.sendStatus(200);  
});
```

# Frontend

Now that we’ve got a backend that can register and authenticate users, we can work on securing our React web application.

> Note: To keep things simple, we won’t create a registration page and instead use our previous cURL command if we want to create new users

## Login Page

To get this started we are going to create a simple React component with a form that will be used to authenticate the user:

```c
// Login.jsx  
import React, { Component } from 'react';export default class Login extends Component {  
  constructor(props) {  
    super(props)  
    this.state = {  
      email : '',  
      password: ''  
    };  
  }  handleInputChange = (event) => {  
    const { value, name } = event.target;  
    this.setState({  
      [name]: value  
    });  
  }  onSubmit = (event) => {  
    event.preventDefault();  
    alert('Authentication coming soon!');  
  }  render() {  
    return (  
      <form onSubmit={this.onSubmit}>  
        <h1>Login Below!</h1>  
        <input  
          type="email"  
          name="email"  
          placeholder="Enter email"  
          value={this.state.email}  
          onChange={this.handleInputChange}  
          required  
        />  
        <input  
          type="password"  
          name="password"  
          placeholder="Enter password"  
          value={this.state.password}  
          onChange={this.handleInputChange}  
          required  
        />  
       <input type="submit" value="Submit"/>  
      </form>  
    );  
  }  
}
```

> View full code on [Github](https://github.com/faizanv/react-auth-example)

This is just a simple form, very similar to the example [on the react website](https://reactjs.org/docs/forms.html).

## Saving Token

As you probably noticed, the `onSubmit` method of the `Login` component is incomplete. We want this method to make a request to authenticate with our backend and save the resulting token to a [browser cookie](https://us.norton.com/internetsecurity-how-to-what-are-cookies.html).

Our express application is setting an httpOnly cookie with a signed JSON Web Token for us so all we have to do is redirect the user properly if they receive a `200` HTTP response when calling `'/api/authenticate’` from our React application.

So we are going to update the `onSubmit` method of our `Login` component to look like this:

```c
// Login.jsx
...
onSubmit = (event) => {  
  event.preventDefault();  
  fetch('/api/authenticate', {  
    method: 'POST',  
    body: JSON.stringify(this.state),  
    headers: {  
      'Content-Type': 'application/json'  
    }  
  })  
  .then(res => {  
    if (res.status === 200) {  
      this.props.history.push('/');  
    } else {  
      const error = new Error(res.error);  
      throw error;  
    }  
  })  
  .catch(err => {  
    console.error(err);  
    alert('Error logging in please try again');  
  });  
}
```

All we’ve done here is use [fetch](https://github.com/github/fetch) to authenticate against our backend and retrieve a JSON Web Token which is saved to our browser cookies.

Now we can add our Login component to our route configuration:

```
import Login from './Login';
...
<Route path="/login" component={Login} />
```

And we’ve got ourselves a functioning login screen:

![](https://miro.medium.com/v2/resize:fit:700/1*_Fh_csfRU1Fb8782khD45A.png)

Functioning login screen

Now if you login successfully, the secret should no longer be a secret!

![](https://miro.medium.com/v2/resize:fit:600/1*icPUNQARNLWJeoT-QE1ukg.gif)

Accessing the secret using our signed JSON web token

## Protecting Routes (react-router)

So we have a working login process that will fetch us a signed token from our backend, save it to our cookies, and subsequently use that token to access protected routes on the server:

![](https://miro.medium.com/v2/resize:fit:628/1*gW4_f6Up3o9lXrTV0-GqxA.png)

Users shouldn’t see this and instead be redirected to login

To add the final touches, we need a way to specify routes to protect on our front end so that a user doesn’t have to see this screen when they aren’t logged in.

Instead we need to redirect them to the login page like a good web application should.

To accomplish this, we are going to be using a concept called [higher-order components](https://reactjs.org/docs/higher-order-components.html) to wrap react-router routes which we want protected.

A [higher-order component](https://reactjs.org/docs/higher-order-components.html) is nothing more than a function which takes in a component and returns a component. So we want to create a higher-order component, `withAuth`, that will take in a component we want to protect, like `<Secret />` , and slightly modify it so that users can’t access it unless they are logged in:

```c
// withAuth.jsx
import React, { Component } from 'react';  
import { Redirect } from 'react-router-dom';export default function withAuth(ComponentToProtect) {  return class extends Component {  
    constructor() {  
      super();  
      this.state = {  
        loading: true,  
        redirect: false,  
      };  
    }    componentDidMount() {  
      fetch('/checkToken')  
        .then(res => {  
          if (res.status === 200) {  
            this.setState({ loading: false });  
          } else {  
            const error = new Error(res.error);  
            throw error;  
          }  
        })  
        .catch(err => {  
          console.error(err);  
          this.setState({ loading: false, redirect: true });  
        });  
    }    render() {  
      const { loading, redirect } = this.state;  
      if (loading) {  
        return null;  
      }  
      if (redirect) {  
        return <Redirect to="/login" />;  
      }  
      return <ComponentToProtect {...this.props} />;  
    }  
  }}
```

> View full code on [Github](https://github.com/faizanv/react-auth-example)

As you can see our `'/checkToken'` route from earlier came in handy like I said it would.

So hopefully it’s clear from the example that `withAuth` is a high-order component which takes in a component to protect, `ComponentToProtect` , and renders it to our view if we have a valid token. Otherwise, we are redirecting the user, [using react-router <Redirect />](https://reacttraining.com/react-router/web/api/Redirect), to `'/login'` .

We can then incorporate this back into our route configuration like so:

```c
import withAuth from './withAuth';
...
<Route path="/secret" component={withAuth(Secret)} />
```

And now we should have a protected route!

![](https://miro.medium.com/v2/resize:fit:600/1*fJnVbSsykAjlixpTlY3SPw.gif)

Protected route using react-router and higher-order component

And that’s it!

We know have a working implementation of a React web app with authentication using JSON web tokens!

There’s definitely much more we can do to polish this up before making it production ready (password validation, cookie expiration checks and refreshes, more user-friendly authentication flow) but my hope is that serves as a good starting point towards a nice and secure application.

> You can find a fully functioning example of the above at [this Github repository](https://github.com/faizanv/react-auth-example).