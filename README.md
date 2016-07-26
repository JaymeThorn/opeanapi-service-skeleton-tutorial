# swagger-service-skeleton-tutorial

The purpose of this tutorial is to use the excellent [swagger-service-skeleton](https://github.com/steve-gray/swagger-service-skeleton) from the incredible and generous [@steve-gray](https://github.com/steve-gray) to create a working application, or, at least to help show you how to do that. 

## Getting started

There is a [companion repository](https://github.com/fastbean-au/swagger-service-skeleton-starter) for this tutorial which I recommend using; while it is not necessary, this tutorial will assume that you are using it.

### Conventions

I will be using _my-_ to prefix names and values that you should substitute with appropriate values - these will be given from the context of the step in which they are found. So, please do not blindly apply these in other contexts without thinking. E.g. `my-path` in once context may be the same as `my-controller` in another.

### Step #1

Have an idea of what you are trying to achieve. This tutorial is not going to do _Hello World_, and neither is it going to use a contrived example that you wil need to sort through to determine what is necessary, and what is not.  So, maybe this should be called a _how to_ instead of a tutorial Whatever.

Now, the rest of the steps do not need to be done in the prescribed order, although some need to be done before others. I'll _try_ to note where this happens - but, if in doubt, just do it in this order.

### Step #2

#### Using the companion repo

Fork the companion repo, rename it (__NB:__ this will become _my-app_), and then clone it.

[Instructions on forking](https://help.github.com/articles/fork-a-repo/)
[Instructions on renaming](https://help.github.com/articles/renaming-a-repository/)
[Companion repo](https://github.com/fastbean-au/swagger-service-skeleton-starter)

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

You'll want to change the fields for `name`, `description`, `author`, `repository`, `issues`, `bugs`, `homepage`.

### Step #4 - Defining the API

Point your brower at [swagger.io's editor](editor.swagger.io) and define your API.

### Okay, I have my swagger, what now?

Not so fast. There are some specific things that need to go into the swagger configuration.

### Step #5 - Controllers

For every `path` in your swagger, you'll need a corresponding entry `x-swagger-router-controller: <path-name>`.  Replace the `<path-name>` with the name of the path (it doesn't need to be, you can call it anything, just so long as there will bea corresponding controler file of the same name).

You need a corresponding controller.  Create a file `./src/controllers/<path-name>.js` if it doesn't already exist.

E.g.:

```yaml
paths:
  /repository:
    x-swagger-router-controller: <my-path>
```

Uses the controller `./src/controllers/<my-path>.js`.

```javascript
'use strict';

class <My-path>ControllerImpl {

}

module.exports = <My-path>ControllerImpl;
```

### Step #6 Controller methods

For each method for a path there needs to be a corresponding `operationId`.  The value for the _operationId_ maps to a method within the controller class defined in the step above.

#### Example

##### Swagger:
```yaml
paths:
  /repository:
    x-swagger-router-controller: repository
    get:
      operationId: getRepository
```

##### Controller
`./src/controller/repository.js`:
```javascript
'use strict';

class RepositoryControllerImpl {

  /**
  * Get the list of known waffle types
  * @param {object} responder      - Automatically generated responder object
  */
  getRepository(responder) {
    responder.success('Yippee!');
  }
}

module.exports = RepositoryControllerImpl;
```

A note on operationId: these are scoped to the swagger file, not to the path as one might hope.  Thus, in the example above `the repository GET method maps not to the repository controller's _get_ method, but to _getRepository_.

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

### Viewing the API documentation

Point your browser at localhost:<listenPort>/docs

where `listenPort` is the port set in the config file.
