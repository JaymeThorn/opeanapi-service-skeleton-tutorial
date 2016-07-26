# swagger-service-skeleton-tutorial

This tutorial is on using the excellent [swagger-service-skeleton](https://github.com/steve-gray/swagger-service-skeleton) from the incredible and generous @steve-gray to create a working application. 

## Getting started

There is a [companion repository](https://github.com/fastbean-au/swagger-service-skeleton-starter) for this tutorial which I recommend using; while it is not necessary, this tutorial will assume that you are using it.

### Step #1

Have an idea of what you are trying to achieve. This tutorial is not going to do _Hello World_, and neither is it going to use a contrived example that you wil need to sort through to determine what is necessary, and what is not.  So, maybe this should be called a _how to_ instead of a tutorial Whatever.

Now, the rest of the steps do not need to be done in the prescribed order, although some need to be done before others. I'll _try_ to note where this happens - bt, if in doubt, just do it in this order.

### Step #2

#### Using the companion repo

Fork the companion repo, and then clone your fork.

[Instructions](https://help.github.com/articles/fork-a-repo/)

[Companion repo](https://github.com/fastbean-au/swagger-service-skeleton-starter)

##### What's in the companion repo?

Not much, but what is there is pretty important.

You'll end up with a directory tree that looks like this:

```
|   .eslintrc.json
|   .gitignore
|   Gruntfile.js
|   index.js
|   jsconfig.json
|   LICENSE
|   package.json
|   README.md
|
+---config
|       default.json
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

`index.js` loads and executes `./src/server.js`. LiTFA - that goes for both of them. Really.  Don't touch them.

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
    x-swagger-router-controller: repository
```

Uses the controller `./src/controllers/repository.js`.

```javascript
'use strict';

class RepositoryControllerImpl {

}

module.exports = RepositoryControllerImpl;
```

### Step #6 Controller methods

For each method for a path there needs to be a corresponding `operationId`.  The value for the _operationId_ maps to a method within the controller class defined in the step above.

E.g.:

swagger:
```yaml
paths:
  /repository:
    x-swagger-router-controller: repository
    get:
      operationId: getRepository
```

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

### Starting

```bash
npm start
```

### Viewing the API documentation

Point your browser at localhost:10010/docs
