# swagger-service-skeleton-tutorial

The purpose of this tutorial is to use the excellent [swagger-service-skeleton](https://github.com/steve-gray/swagger-service-skeleton) from the incredible and generous [@steve-gray](https://github.com/steve-gray) to create a working application API, or, at least to help show you how to do that. 

## Getting started

There is a [companion repository](https://github.com/fastbean-au/swagger-service-skeleton-starter) for this tutorial which I recommend using; while it is not necessary, this tutorial will assume that you are using it.

[![Dependency Status](https://dependencyci.com/github/fastbean-au/swagger-service-skeleton-starter/badge)](https://dependencyci.com/github/fastbean-au/swagger-service-skeleton-starter)

### Conventions

I will be using _my-_ to prefix names and values that you should substitute with appropriate values - these will be given from the context of the step in which they are found. So, please do not blindly apply these in other contexts without thinking. E.g. `my-path` in once context may be the same as `my-controller` in another.

### Step #1

Have an idea of what you are trying to achieve. This tutorial is not going to do _Hello World_, and neither is it going to use a contrived example that you wil need to sort through to determine what is necessary and what is not. So, maybe this should be called a _how to_ instead of a tutorial Whatever.

Now, the rest of the steps do not _need_ to be done in the prescribed order, although some need to be done before others. I'll _try_ to note where this happens - but, if in doubt, just do it in this order.

### Step #2

#### Using the companion repo

Fork the companion repo, rename it (__NB:__ this will become _my-app_), and then clone it.

* [Instructions on forking](https://help.github.com/articles/fork-a-repo/)
* [Instructions on renaming](https://help.github.com/articles/renaming-a-repository/)
* [Companion repo](https://github.com/fastbean-au/swagger-service-skeleton-starter)

##### What's in the companion repo?

Not much, but what is there is pretty important.

You'll end up with a directory tree that looks like this:

```
|   .eslintrc.json        <--- This helps to show where there are potential problems with your code
|   .gitignore
|   Gruntfile.js          <--- Runs ESLint over your code
|   index.js              <--- LiTFA!  Loads and executes ./src/server.js
|   LICENSE
|   package.json          <--- You need to edit this file and make it your own!
|   README.md             <--- Put your content in here
|
+---config
|       default.json      <--- Set the port here, and add any other configuration options you want
|
\---src
    |   server.js         <--- LiTFA!
    |
    +---contracts
    |       swagger.yaml  <--- Empty. Placeholder. Replace with your swagger.yaml file.
    |
    +---controllers       <--- This is where *your* Controllers are placed.
    |       .gitignore    <--- Empty. Used to allow the directory to exist in GitHub.
    |
    \---services          <--- This is where *your* Services are placed.
            .gitignore    <--- Empty. Used to allow the directory to exist in GitHub.
```

###### server.js

The paths used in `server.js` can be a little confusing.  Below, the contents of the file have been annotated .

```javascript
'use strict';

const config = require('config').get('server');
const skeleton = require('swagger-service-skeleton');

const instanceGenerator = () => skeleton({
  ioc: {
    /*
       This path is relative to *this* file, which is in the ./src directory, thus it is:
       ./src/services/*.js
    */
    autoRegister: { pattern: './services/*.js',
                    rootDirectory: __dirname },
  },
  codegen: {
    /*
       This is a generated directory - this is where all of the glue created by the skeleton lives.
       It is relative to the application (repository) root directory.
    */
    temporaryDirectory: './dist/codegen',
    /*
       This path is relative from the directory defined by temporaryDirectory, which resolves to:
       ./src/controllers
    */
    templateSettings: {
      implementationPath: '../../../src/controllers',
    },
  },
  service: {
    /*
       This is the path to your YAML file. It is relative to the application (repository) root 
       directory.
    */
    swagger: './src/contracts/swagger.yaml',
    listenPort: config.listenPort,
  },
});

module.exports = instanceGenerator;
```

#### Going it yourself 

Yeah - basically you need what's in the compaion repo - see above.

### Step #3 - Make it yours

Modify the `package.json` file.

You'll want to change the fields for `name`, `description`, `author`, `repository`, `issues`, `bugs`, `homepage`.  If you change the `license`, then be sure to replace the contents of the `LICENSE` file.

### Step #4 - Install the libraries

We will want to do this pretty early on as it ensures that we have the linting in place, and that will help ensure that we have correctly constructed code ..... I speak from experience here - the experience of not using it soon enough.

From the application (repo) root directory run the following:

```bash
npm install
```

This will add and populate the `node_modules` directory in the root of the application/repo. You shouldn't need to touch anything in here. You'll now have a directory tree that looks like this:

```
|   .eslintrc.json
|   .gitignore
|   Gruntfile.js
|   index.js
|   LICENSE
|   package.json
|   README.md
|
+---config
|       default.json
|
+---node_modules          <--- This has been added, with a whole bunch of sub-directories
|
\---src
    |   server.js
    |
    +---contracts
    |       swagger.yaml
    |
    +---controllers
    |       .gitignore
    |
    \---services
            .gitignore
```

## Construction

### Step #1 - Defining the API

Point your brower at [swagger.io's editor](editor.swagger.io) and define your API.

### Okay, I have my swagger, what now?

Not so fast. There are some specific things that need to go into the swagger configuration.

### Step #2 - Controllers

For every `path` in your swagger contract, you'll need a corresponding entry `x-swagger-router-controller: <path-name>`.  Replace the `<path-name>` with the name of the path (it doesn't need to be, you can call it anything, just so long as there will bea corresponding controler file of the same name).

You need a corresponding controller. Create a file `./src/controllers/<path-name>.js` if it doesn't already exist.

i.e.:

```yaml
paths:
  /<my-path>:
    x-swagger-router-controller: <my-path>
```

Uses the controller `./src/controllers/<my-path>.js`.

```javascript
'use strict';

class <My-path>ControllerImpl {

}

module.exports = <My-path>ControllerImpl;
```

### Step #3 - Response outcomes

Each response defined in your swagger contract needs to have a corresponding entry `x-gulp-swagger-codegen-outcome: <name>`. It is this _name_ that is used by the controller methods call on the responder object. The outcome names can be re-used in different path verbs, however, they must be unique within any given verb's outcomes.

#### Example:

##### swagger contract:

```yaml
      responses:
        204:
          x-gulp-swagger-codegen-outcome: success
          description: My response object
          schema:
            $ref: '#/definitions/<my-object>'
        400:
          x-gulp-swagger-codegen-outcome: badRequest
          description: bad request
          schema:
            $ref: '#/definitions/ErrorResponse'
        500:
          x-gulp-swagger-codegen-outcome: error
          description: server error
          schema:
            $ref: '#/definitions/ErrorResponse'
```

The example above would require an `ErrorResponse` object definition, and for the controller method implementing the path verb to supply the defined object.

### Step #4 Controller methods

For each verb for a path there needs to be a corresponding `operationId`.  The value for the _operationId_ maps to a method within the controller class defined in the step above.

It is the controller methods that call the responder outcomes.

#### Example

##### Swagger contract

```yaml
paths:
  /<my-path>:
    x-swagger-router-controller: <my-path>
    get:
      operationId: <my-method>
      responses:
        200:
          x-gulp-swagger-codegen-outcome: <my-outcome>
          schema:
            type: string
```

##### Controller
`./src/controller/<my-path>.js`:
```javascript
'use strict';

class <My-path>ControllerImpl {

  /**
  * 
  * @param {object} responder      - Automatically generated responder object
  */
  <my-method>(responder) {
    responder.<my-outcome>('Yippee!');
  }
}

module.exports = <My-path>ControllerImpl;
```

A note on operationId: think of these as scoped to the swagger file, not to the path as one might hope. Thus, in the example above `the repository GET method maps not to the repository controller's _get_ method, but to _getRepository_.

### Step #5 - Services

Optional, and, forthcoming.......

Services are used by the controllers. They have no direct correspondence with the Swagger contract definition file.

### Step #6 - Adding the Swagger

Now that the swagger contract has been updated with the requirements of the _swagger-service-skeleton_ framework, it can be placed into the application (delaying this up until this point helps to ensure that the swagger.io parser is able to identify any issues that might otherwise be very difficult to pick up through the application itself). You can either copy the contents onto your clipboard and paste into the file `./src/contracts/swagger.yaml` or you can download the YAML file from swagger.io and overwrite the existing `./src/contracts/swagger.yaml` file.

### Step #7 - Committing

You will really want to cmmit your changes now, before you run. 

## Running

### Installing for production

```bash
git clone https://github.com/<user name>/<my-app>.git
cd <my-app>
npm install --production
```

### Starting

```bash
npm start
```

Starting the application will result in a code generation step to be executed - this is where the glue between your swagger contract, the server, and your controllers and services is produced. This will be placed in the directory `./dist/codegen` in the root of your application/repo. You'll end up with a tree that looks like this:

```
|   .eslintrc.json
|   .gitignore
|   Gruntfile.js
|   index.js
|   LICENSE
|   package.json
|   README.md
|
+---dist                  <--- This has been added
|   \ codegen             <--- This contains the generated code. LiTFA!
|
+---config
|       default.json
|
+---node_modules
|
\---src
    |   server.js
    |
    +---contracts
    |       swagger.yaml
    |
    +---controllers
    |       my-route.js   <--- That's a made-up file name
    |
    \---services
            my-service.js <--- That's a made-up file name
```


### Viewing the API documentation

Point your browser at localhost:&lt;listenPort&gt;/docs, where `listenPort` is the port set in the config file, e.g.: http://localhost:10010/docs

## Other recommendations

### Debugging

Use the [debug package](https://www.npmjs.com/package/debug) in each of your controller and service files. It's already used by a very large portion of the stack that your application will be built on.

### Swagger

Don't fight swagger. It's generally only difficult to use when you are trying to do something in a less than optimal or non-standard way.......
