# swagger-service-skeleton-tutorial

The purpose of this tutorial is to use the excellent [swagger-service-skeleton](https://github.com/steve-gray/swagger-service-skeleton) from the brilliant and generous [@steve-gray](https://github.com/steve-gray) to create a working application API, or, at least to help show you how to do that.

The Swagger service skeleton allows you to produce a web service without having to worry about most of the plumbing. It's use, though, does require some care and attention to detail to get it right. Helping you to get that right is what this hopes to achieve.

You might want to use the Swagger service skeleton as part of a microservice, a complete nanoservice, a service in a service-oriented-architecture (SOA), or just to provide a RESTful (or stateful) API for a system. It's not limited to those things, but that's the target for it. It's not going to serve up your UI for you - it's not user facing.

## What this is not

This is not a tutorial or how-to for NodeJS/Javascript, Swagger, API design (RESTful or otherwise), or GitHub.  This assumes that you have enough knowledge of those to operate in a somewhat competent manner. Which is not to say that some pointers or references will not be give, it's just that this it is not the purpose of this tutorial/how-to.

## Getting started

There is a [companion repository](https://github.com/fastbean-au/swagger-service-skeleton-starter) for this tutorial which I recommend using; while it is not necessary, this tutorial will assume that you are using it.

[![Dependency Status](https://dependencyci.com/github/fastbean-au/swagger-service-skeleton-starter/badge)](https://dependencyci.com/github/fastbean-au/swagger-service-skeleton-starter)

### Conventions

I will be using _my-_ to prefix names and values that you should substitute with appropriate values - these will be given from the context of the step in which they are found. So, please do not blindly apply these in other contexts without thinking. E.g. `my-path` in one context may be the same as `my-controller` in another.

### Step #1

Have an idea of what you are trying to achieve. This tutorial is not going to do _Hello World_, and neither is it going to use a contrived example that you will need to sort through to determine what is necessary and what is not. So, maybe this should be called a _how to_ instead of a tutorial Whatever.

Now, the rest of the steps do not _need_ to be done in the prescribed order, although some need to be done before others. I'll _try_ to note where this happens - but, if in doubt, just do it in this order.

### Step #2 - Using the companion repository

Fork the companion repository, rename it (__NB:__ this will become what will be referred to as _my-app_), and then clone it.

* [Instructions on forking](https://help.github.com/articles/fork-a-repo/)
* [Instructions on renaming](https://help.github.com/articles/renaming-a-repository/)
* [Companion repository](https://github.com/fastbean-au/swagger-service-skeleton-starter)

You don't really need to fork the companion repository. Clone it and copy the contents, download it, it doesn't really matter all that much. But, if you fork it, you can more easily see if the companion repository has been updated (of course, you could watch it too). 

#### What's in the companion repository

Not much really, but what is there is pretty important. Once you have forked it you'll end up with a directory tree that looks like this:

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

##### server.js

The paths used in `server.js` can be a little confusing. Below, the contents of the file have been annotated. This is important to know only if you want to change the structure - but don't do that now, not for this.

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

Yeah - basically you need what's in the companion repository - see above.

### Step #3 - Make it yours

Modify the `package.json` file.

You'll want to change the fields for `name`, `description`, `author`, `repository`, `issues`, `bugs`, `homepage`. If you change the `license`, then be sure to replace the contents of the `LICENSE` file.

### Step #4 - Install the libraries

We will want to do this pretty early on as it ensures that we have the linting in place, and that will help ensure that we have correctly constructed code ..... I speak from experience here, the experience of not using it soon enough.

From the application (repository) root directory run the following:

```bash
npm install
```

This will add and populate the `node_modules` directory in the root of the application/repository. You shouldn't need to touch anything in here. You'll now have a directory tree that looks like this:

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

Point your browser at [swagger.io's editor](editor.swagger.io) and define your API.

__Important note:__ Before you get carried away, you need to be aware of an issue in the stack that prevents the HTTP/S verb _PATCH_ from working, instead of using _PATCH_ you will probably want to use _PUT_.

### Okay, I have my swagger, what now

Not so fast. There are some specific things that need to go into the Swagger configuration.

### Step #2 - Controllers

For every `path` in your Swagger configuration, you'll need a corresponding entry `x-swagger-router-controller: <path-name>`. Replace the `<path-name>` with the name of the path (it doesn't need to be, you can call it anything, just so long as there will be a corresponding controller file of the same name).

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

__Q:__ Is it necessary to have the controller name the same as the contract path? 

__A:__ No, absolutely not. Except that it's easier and more intuitive to do it that way.

### Step #3 - Response outcomes

Each response defined in your Swagger configuration needs to have a corresponding entry `x-gulp-swagger-codegen-outcome: <name>`. It is this _name_ that is used by the controller methods call on the responder object. The outcome names can be re-used in different path verbs, however, they must be unique within any given verb's outcomes. I am going to suggest that you include a 501 - Not implemented response outcome. Each response outcome defined in the Swagger configurationb for a given path's verb should be used when the method is fully implemented.

#### Example

##### Swagger configuration

```yaml
      responses:
        204:
          x-gulp-swagger-codegen-outcome: success
          description: My response object
          schema:
            $ref: '#/definitions/<my-object>'
        400:
          x-gulp-swagger-codegen-outcome: badRequest
          description: Bad request
          schema:
            $ref: '#/definitions/ErrorResponse'
        500:
          x-gulp-swagger-codegen-outcome: error
          description: Server error
          schema:
            $ref: '#/definitions/ErrorResponse'
        501:
          x-gulp-swagger-codegen-outcome: notImplemented
          description: Not implemented
          schema:
            $ref: '#/definitions/ErrorResponse'
```

The example above would require an `ErrorResponse` object definition, and for the controller method implementing the path verb to supply the defined object.

### Step #4 Controller methods

For each verb for a path there needs to be a corresponding `operationId`. The value for the _operationId_ maps to a method within the controller class defined in the step above.

It is the controller methods that call the responder outcomes.

I would _suggest_ just creating the controller methods as stubs for the time being. Maybe include a debug statement, and return a successful response, or a 501 - Not implemented response (this would need to be included in your Swagger configuration). Go back later and actually write the functional aspects of the methods.

#### Example

##### Swagger configuration

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

The controller, at this stage of what we are doing, will probably look something like:
```javascript
'use strict';

class <My-path>ControllerImpl {

  /**
  * 
  * @param {object} responder      - Automatically generated responder object
  */
  <my-method>(responder) {
    responder.notImplemented({ message: 'This method has not been implemented' });
  }
}

module.exports = <My-path>ControllerImpl;
```

A note on operationId: think of these as scoped to the Swagger configuration file, not to the path as one might hope. Thus, in the example above, the repository GET method maps not to the repository controller's _get_ method, but to _getRepository_.

### Step #4A - Controller method parameters

Controller method parameters are determined by the parameters as defined in the Swagger configuration, in lexicographical order, with the special _responder_ object as an added final parameter.

If you want to keep it straightforward, you can use JSON objects as the sole parameter for the configuration path's verb. If you do this, then you will have two parameters always in the same predictable order. Yeah, I know, helpful (maybe), but not really informative.

### Step #5 - Services

Let's start with a simple statement. Services are _optional_, They are good practice for a separation of concerns and simplifying your code, but they remain _optional_.

In our context, services are used by the controllers. They have no direct correspondence with the Swagger configuration definition file, that is, they are services that support the controllers, and not the services of which _swagger-service-skeleton_ speaks. Services might include things like security, persistence (database), etcetera.

The services are injected, and that means that a bit of magic happens by the framework that allows this to work, and that makes it easier ...

The services do not _need_ to be written now, and I would suggest deferring the writing of them until once the application structure has been completed (step #7)

#### Name resolution

Name resolution is vital to understand how to allow the magic that is injection work it's, well, magic.

To keep this fairly simple, the name of the service as referenced in the controller class' constructor parameters should be a camel case of the filename without extension (the basename), using the dash/hyphen as a word separator.

Thus, the camel case for camel-case would be camelCase.

Working back from that, name your service file based on the logical name for your service class. A service class called _MyDatabaseBackup_ should be in a file named `my-database-backup.js`.

#### Example

In this example, unlike any other uses elsewhere, the `my-` in `my-services` is being used as a literal example rather than as simply a placeholder. This is because of the importance of the name construction to the injection magic.

##### Service

`./services/my-service.js`

```javascript
'use strict';

class MyService {
  constructor() {
  }
}

module.exports = MyService;
```

##### Controller

In your contract code you would make this service available similar to this:

`./src/controllers/<my-path>.js`

```javascript
'use strict';

class <My-path>ControllerImpl {
  constructor(myService) {
    this.myService = myService;
  }
```

### Step #6 - Adding the Swagger

Now that the swagger contract has been updated with the requirements of the _swagger-service-skeleton_ framework, it can be placed into the application (delaying this up until this point helps to ensure that the swagger.io parser is able to identify any issues that might otherwise be very difficult to pick up through the application itself). You can either copy the contents onto your clipboard and paste into the file `./src/contracts/swagger.yaml` or you can download the YAML file from swagger.io and overwrite the existing `./src/contracts/swagger.yaml` file.

### Step #7 - Committing

You will really want to commit your changes now, before you go any further. This is the point at which you can run the application and check your API "works".

### Step #8 - Writing the code

Go through and write your functional code in the methods for the controllers and services. Remove the 501 - Not implemented response from the method, and the corresponding response outcomes from the Swagger configuration (again, I would suggest doing this in the [swagger editor](http://editor.swagger.io)). Commit as you go.

## An Example

The example shown in the image below (using Visual Studio Code), is a highly simplified contrivance, which I am hoping helps to show the relationships between various pieces.

![alt text](https://github.com/fastbean-au/swagger-service-skeleton-tutorial/blob/master/images/contract-controllers.png "Contract and controllers")

![alt text](https://github.com/fastbean-au/swagger-service-skeleton-tutorial/blob/master/images/contract-controllers-annotated.png "Contract and controllers - annotated")

## Running

```bash
npm start
```

Starting the application will result in a code generation step to be executed - this is where the glue between your Swagger configuration, the server, and your controllers and services is produced. This will be placed in the directory `./dist/codegen` in the root of your application/repo. You'll end up with a tree that looks like this:

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

Point your browser at localhost:&lt;listenPort&gt;/docs, where `listenPort` is the port set in the configuration file, e.g.: [http://localhost:10010/docs](http://localhost:10010/docs)

## Other recommendations

### Debugging

Use the [debug package](https://www.npmjs.com/package/debug) in each of your controller and service files. It's already used by a very large portion of the stack that your application will be built on.

### Configuration

The [config package](https://www.npmjs.com/package/config) is already used by the server, and there is already a default configuration file which can be extended. If you need to have configuration settings, I would suggest using this package.

Opinionated advice:

* Use consistent return data structures, for errors and data. If you are returning JSON data, return JSON error objects, too. It just makes it easier, IMO, for the consumer. 

### Deployment

Opinionated advice:

* Use containers (e.g. [Docker](http://docker.io/)) - this application structure is eminitely suitable for use as a container (based on the [node container](https://hub.docker.com/_/node/)).
* Do not package as an NPM package - this is an application, not a library - it needs to be instantiated, not installed.

#### Installing for production

```bash
git clone https://github.com/<user name>/<my-app>.git
cd <my-app>
npm install --production
npm start
```

### Swagger

Don't fight swagger. It's generally only difficult to use when you are trying to do something in a less than optimal or non-standard way.......
