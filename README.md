## Poker Flat Ground Control Station

UAV ground station software emphasizing data collection and mission operations.

### Installation (for development)

#### Prerequisites

You need ```git```, ```node```, ```npm``` (distributed with node, usually), and ```grunt```.  On OSX, [install Node.js from an installer](http://nodejs.org/#download), then the rest with brew and npm:

```
brew install git
sudo npm install -g grunt_cli
sudo npm install -g nodemon
```

#### Running

After installing prerequisites, clone this project and install dependencies, then build runtime files with ```grunt```:

```
git clone git@github.com:poker-flat/ground-control-station.git
npm install
grunt
```

### Configuration

Work in progress, you need to specify the connection type in the ```config.json``` file at the root of the project (copy/paste the ```config.json.example``` file as a template), and update the relevant fields (host, port, serial device, etc).

### Running the project

Running the project has two parts: starting the node server process, and launching the grunt watcher process.  This kicks off a new build anytime any files that are used by the client are modified -- it takes a few moments, so when you save the file, it may be a few seconds before your changes are visibile in the restarted node process.

```bash
nodemon server.js &
grunt && grunt watch
```

### Project architecture and structure

```
├── app <<< We write the code. Shared code (mainly/exclusively Backbone + RequireJS) between server/client
│   ├── Templates
│   │   └── try.jade
│   ├── Templates.js <<< Auto-generated upon build so we can Require the templates.
│   ├── Models
│   ├── Collections
│   ├── Views
│   │   └── Try.js
│   ├── app.js <<< "Top level application namespace", so to speak (rename gcs?)
│   ├── config.js <<< Config point for require.js
│   ├── main.js <<< Like main() in C, this is the code entry execution point for the client GUI
│   └── router.js <<< Standard Backbone router, if we need multiple they should go in a Routers/ dir
├── assets <<< We may write the code.  Static/external/other code used in the build process.
│   ├── js
│   │   ├── libs <<< Code we maintain, non-Backbone or otherwise odd; we may need to wrap for AMD, etc.
│   │   └── vendor <<< Code we don't maintain at all
│   │       ├── backbone.js
│   │       ├── jade.js
│   │       ├── jquery.js
│   │       ├── jshint.js
│   │       ├── require.js
│   │       └── underscore.js
│   └── less <<< Stylesheets for our Backbone Views, all get compiled during build process into /public/stylesheets/
│       └── assets.less
├── build <<< Interim files created during build / continuous-integration processes
├── grunt.js <<< Build configuration script
├── package.json <<< Node dependencies
├── public <<< We don't write the code!  It's autogenerated.  Express web root.
│   ├── images
│   ├── javascripts
│   │   ├── require.js <<< Necessary to bootstrap the client.
│   │   └── required.js <<< Auto-generated during build process, contains ALL client app code
│   └── stylesheets
│       ├── assets.css <<< This file is autogenerated during build process (from assets/less)
│       ├── style.css <<< these are generated at runtime by Express
│       └── style.less <<< This is requested by the client, Express munges it
├── routes <<< Express routes
│   ├── index.js
│   └── user.js
├── server.js <<< Node.js server (rename gcs-server?)
├── test
│   └── jasmine <<< It's in a bad state with Grunt 0.3.x vs 0.4.x, no CI possible now
│       ├── index.html
│       ├── spec
│       │   └── TrySpec.js
│       └── vendor
│           ├── MIT.LICENSE
│           ├── jasmine-html.js
│           ├── jasmine-jquery.js
│           ├── jasmine.css
│           ├── jasmine.js
│           └── jasmine_favicon.png
└── views <<< Express views
    ├── index.jade
    └── layout.jade
```

[Grunt](http://gruntjs.com/) is used as the Javascript build system, and our build file (```grunt.js```) is a work in progress.  [Backbone](http://backbonejs.org/) handles the heavy lifting on the client side, and [express](http://expressjs.com/) is the routing architecture/middleware manager on the server.

### Development
(OSX) You'll need to install the [FTDI Arduino MegaPilot driver](http://www.ftdichip.com/Drivers/VCP.htm) before the system will recognize the system.

if connecting via the usb, be sure to set the baudrate in config.json:

```json
{
  ...
  "baudrate" : 115200
  ...
}
```

Many things are dependent upon the build cycle, so you usually want to have this running:

```bash
grunt && grunt watch
```

#### Using SITL on a remote host

Assuming you're able to get the [SITL guide on this page](http://dev.ardupilot.com/software-in-the-loop-sitl/) running, here's how to connect this ground station to that SITL/JBSim instance pair.  For this setup, we won't have MAVProxy in the loop.  On the machine running the simulation, you need to ensure that you can access some TCP port (by default, ArduPilot will use 5760) from wherever the ground station is running.

 1. Start the ArduPlane simulator: ```/tmp/ArduPlane.build/ArduPlane.elf &```
 2. Start JSBSim: ```python ./ardupilot/Tools/autotest/jsbsim/runsim.py --home=-35.362938,149.165085,584,270 &```
 3. Change the config file for your ground station to be ```tcp```, and to connect to the correct IP/port on  the machine running the ArduPlane and JSBSim.
 4. Start the ground station: ```grunt && nodemon server.js```
 5. Open a web page to ```localhost:3000```

### Testing

#### Configuration on Jenkins

The ```config.json.example``` file is used to configure Jenkins' runtime environment.  It's copied to ```config.json```, so that file needs to be updated as appropriate if new config options are added.  For testing, the important configuration at this time is the location of the virtual serial ports.

#### Running tests

Some additional prerequsites may be required:

```
npm install -g grunt
```

There's two different collections of test suites: ones that run on the server, and ones that run on the GUI/interface layer.  Mocha is used for the server-side tests, Jasmine for the GUI layer.

To run server-side (mocha) tests in the development environment:

```bash
mocha test
```

To run Mocha tests for xunit output (for CI/Jenkins):

```bash
mocha --reporter xunit test/mocha
```

Running client-side Jasmine tests through grunt:

```bash
grunt jasmine
```