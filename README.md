# NodeJS with Express App Template


## What is this?

Just my way to structure Node JS applications, making this document just to remember how to do it next time.

## Create project

Go to the folder of your project and execute

```shell
npm init
```

## Folder structure

```
NodeJS-Project/
|-- app/
|   |-- controllers/
|   |   |-- authController.js
|   |   |-- usersController.js
|   
|   |-- middlewares/
|   |   |-- authMiddleware.js
|   |   |-- corsMiddleware.js
|
|   |-- models/
|   |   |-- userModel.js
|
|   |-- routes/
|   |   |-- authRoutes.js
|   |   |-- index.js
|   |   |-- usersRoutes.js
|
|-- assets/
|   |-- something.png
|
|-- config/
|   |-- db/
|   |   |-- creates.sql
|   
|   |-- db.js
|   
|-- docs/
|   |-- docs.md
|  
|-- tests/   
|   |-- integration/
|   |   |-- something.test.js
|
|   |-- unit/
|   |   |-- testControllers
|   |   |   |-- authController.test.js
|   |   |   |-- usersController.test.js
|
|-- .env
|-- .gitignore
|-- app.js
|-- README.md
|-- package.json
```

## IDEs
I use IntelliJ to develop so all the instructions will be based on IntelliJ

## Installation
1. On `app.js` the code should look like this:
```js
require('dotenv').config();

const express = require('express');
const app = express();

// Config imports
const { connectDB } = require('./config/db'); // Directly import the connectDB function

// Apply routes
const applyRoutes = require('./app/routes');

// Apply Middlewares
const corsMiddleware = require('./app/middlewares/corsMiddleware');

// Apply middlewares
app.use(express.json()); // Middleware to parse JSON bodies
app.use(corsMiddleware);

// Apply routes
applyRoutes(app);

// Establish database connection
connectDB();

// Start the server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

2. This is an example of an `authMiddleware.js` middleware
```js
/**
 * Middleware module containing functions for user authentication and password hashing.
 * These middlewares are used to enhance security and manage user authentication.
 *
 * @module authMiddleware
 */
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');

/**
 * Middleware function to hash the password in the request body before processing.
 *
 * @param {Object} req - The HTTP request object.
 * @param {Object} res - The HTTP response object.
 * @param {Function} next - The next middleware function.
 * @throws {Object} - An error response object if password hashing fails.
 */

const hashPassword = async (req, res, next) => {
    try {
        if (req.body.password) {
            req.body.password = await bcrypt.hash(req.body.password, 10);
        }
        next();
    } catch (error) {
        next(error);
    }
};

/**
 * Middleware function to authenticate and verify JWT tokens in the Authorization header.
 *
 * @param {Object} req - The HTTP request object.
 * @param {Object} res - The HTTP response object.
 * @param {Function} next - The next middleware function.
 * @throws {Object} - An error response object if authentication fails, the token is invalid, or the Authorization header is missing.
 */

const authMiddleware = (req, res, next) => {
    const authHeader = req.headers.authorization;

    if (authHeader) {
        const token = authHeader.split(' ')[1]; // Authorization: Bearer <token>

        jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
            if (err) {
                return res.status(403).json({ error: 'Token is not valid' });
            }

            req.user = user;
            next();
        });
    } else {
        res.status(401).send('Authorization header is missing');
    }
};

module.exports = {
    authMiddleware,
    hashPassword
};
```

3. This is an example of a `corsMiddleware.js` middleware
```js
/**
 * Middleware that enables Cross-Origin Resource Sharing (CORS) for HTTP requests.
 * By default, this middleware allows all origins and common HTTP methods.
 * Adjust the 'origin' property in 'corsOptions' to restrict allowed origins.
 *
 * @module corsMiddleware
 */
const cors = require('cors');

// Setup CORS options if needed
const corsOptions = {
    origin: '*', // This is a wildcard that allows all origins - adjust as needed
    methods: 'GET,HEAD,PUT,PATCH,POST,DELETE',
    allowedHeaders: ['Content-Type', 'Authorization'],
};

module.exports = cors(corsOptions);

```

4. This is an example of a router in `authRoutes.js`
```js
/**
 * Authentication routes module.
 * Defines the routes related to user authentication, such as logging in.
 *
 * @module authRoutes
 */
const express = require('express');
const router = express.Router();

const { login } = require('../controllers/authController');

/**
 * Route for user login.
 * Accepts a POST request to authenticate a user with their credentials.
 * It invokes the 'login' controller function to handle the login process.
 *
 * @name POST /login
 * @param {string} path - Express route path.
 * @param {function} controller - Controller function responsible for user login.
 */
router.post('/login', login);

module.exports = router;
```

5. This is an example of indexing multiple routers in `index.js`
```js
const authRoutes = require('./authRoutes');
const usersRoutes = require('./usersRoutes')
const {authMiddleware} = require("../middlewares/authMiddleware");

module.exports = (app) => {
    app.use('/api/v1', authRoutes);
    app.use('/api/v1/users', usersRoutes);
};
```

## Login Base
This Login is implemented using JWT and bycrypt (to hash passwords). We need the following dependencies

1. Install dependencies
    ```shell
    npm install bycryptjs
    npm install jsonwebtoken
    ```
2. In [this repository](https://github.com/NeutronBlast/Node-Simple-Login) I have a login implementation with OAuth and Discord. Check `userRoutes.js`, `authController.js` and `userController`


## Libraries
1. [Nodemon](https://www.npmjs.com/package/nodemon)
2. [Jest](https://www.npmjs.com/package/jest)
3. [Sequelize](https://sequelize.org)
4. [Express](https://expressjs.com/es/)
