Let users authenticate themselves to the Nuts network
-----------------------------------------------------

.. toctree::
    :maxdepth: 2
    :caption: Contents:

The Nuts network is based upon encryption instead of trust. This means that instead
of trusting all parties in the network to provide correct data, we ensure all data
is cryptographically verifiable. This includes the identity of people requesting
data of patients.

Therefor in order to make a request for data by another party, a user should provide
identity information that everyone in the network can verify. For that we use the
`IRMA <https://irma.app/docs/>`_ framework developed by the `privacy by design
foundation <https://privacybydesign.foundation/>`_ (a spin off of the Radboud university).

In this tutorial we will show you how to include a user interface inside your
application were users can authenticate themselves using IRMA.
The end result will look like this:

.. figure:: ../../_static/images/irma_flow.gif
    :width: 400px
    :align: center
    :alt: Nuts IRMA login flow
    :figclass: align-center

    Nuts IRMA login flow

+-------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Useful links                                                                        | Description                                                       |
+=====================================================================================+===================================================================+
| `IRMA web frontend <https://github.com/nuts-foundation/irma-web-frontend>`_         | Repository of the Nuts irma styleguide                            |
+-------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| `IRMA styleguide <https://nuts-foundation.github.io/irma-web-frontend/index.html>`_ | Styleguide on how to embed the IRMA screens in your application   |
+-------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| `Nuts auth-js <https://www.npmjs.com/package/@nuts-foundation/auth>`_               | Package for easy nuts-auth irma flow integration                  |
+-------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| `Tutorial Code <https://github.com/nuts-foundation/auth-tutorial.git>`_             | All code from this tutorial can be found in this github tutorial. |
+-------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| `ngrok <https://github.com/nuts-foundation/auth-tutorial.git>`_                     | Ngrok allows your phone to connect to the local nuts service      |
+-------------------------------------------------------------------------------------+-------------------------------------------------------------------+

Creating an empty Express JS application
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For demo purposes we start with an empty generated express js application.
If you have an existing app were you would like to implement this, just use that.


Install the `express-generator <https://www.npmjs.com/package/express-generator>`_ first

.. code-block:: console

  $ npm install -g express-generator
  /usr/local/bin/express -> /usr/local/lib/node_modules/express-generator/bin/express-cli.js
  + express-generator@4.16.1
  added 10 packages from 13 contributors in 0.762s

Make a new project directory and create an empty express project using `ejs <https://ejs.co>`_ for templating and `sass <https://sass-lang.com>`_ for CSS rendering:

.. code-block:: console

  $ md nuts-auth-tutorial
  $ cd nuts-auth-tutorial
  $ express --view=ejs --css=sass .

   create : public/
   create : public/javascripts/
   create : public/images/
   create : public/stylesheets/
   create : public/stylesheets/style.sass
   create : routes/
   create : routes/index.js
   create : routes/users.js
   create : views/
   create : views/error.ejs
   create : views/index.ejs
   create : app.js
   create : package.json
   create : bin/
   create : bin/www

Run the npm installer to install dependencies

.. code-block:: console

  $ npm install

Start up the empty app

.. code-block:: console

  $ npm start

The empty app starts at `<http://localhost:3000>`_

The code so far can be found at `this github branch <https://github.com/nuts-foundation/auth-tutorial/pull/new/empty-app>`_

Create login page
^^^^^^^^^^^^^^^^^

Ok, lets make an empty login page.

We need to add 2 files: A router/controller which handles the request and creates the viewmodel and a template.

.. code-block:: html
  :name: /views/login.ejs
  :caption: /views/login.ejs
  :linenos:

  <h1>Login with IRMA</h1>

.. code-block:: javascript
  :name: /routes/login.js
  :caption: /routes/login.js
  :linenos:

  var express = require('express');
  var router = express.Router();

  router.get('/', function (req, res, next) {
    res.render('login', {})
  });

  module.exports = router;

Now register the login router in `app.js`:


.. code-block:: diff
  :name: app.js

  diff --git a/app.js b/app.js
  index 3ccf8e4..2da894b 100644
  --- a/app.js
  +++ b/app.js
  @@ -7,6 +7,7 @@ var sassMiddleware = require('node-sass-middleware');

   var indexRouter = require('./routes/index');
   var usersRouter = require('./routes/users');
  +var loginRouter = require('./routes/login');

   var app = express();

  @@ -28,6 +29,7 @@ app.use(express.static(path.join(__dirname, 'public')));

   app.use('/', indexRouter);
   app.use('/users', usersRouter);
  +app.use('/login', loginRouter);

   // catch 404 and forward to error handler
   app.use(function(req, res, next) {


Restart the app and navigate to `<http://localhost:3000/login>`_ to enjoy the result of your hard labour.

The code for progress so far can be found `at github <https://github.com/nuts-foundation/auth-tutorial/tree/empty-login-page>`_

Add login html and javascript
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To make it easy to integrate Nuts with your existing application, we provided
tools to make your life easier.
We created a simple styleguide and a javascript IRMA state machine which we will
now embed on our freshly created login page.

First start by entering the following the html which contains the instructions and states to
inform the user. The html is taken from the `Nuts irma styleguide <https://nuts-foundation.github.io/irma-web-frontend/section-examples.html>`_.

.. code-block:: html
  :linenos:
  :caption: views/login.ejs

  <section class="nuts-login-form irma-web-form">
    <header class="header">
        <p>Login with <i class="irma-web-logo">IRMA</i></p>
        <section class="helper">
            <p>Don't know what to do here? Take a look at the <a href="https://privacybydesign.foundation/irma-begin/">de
                    website of IRMA</a>.</p>
        </section>
    </header>
    <section class="content">
        <section class="centered loading">
            <div class="irma-web-loading-animation"><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i>
            </div>
            <p>One moment please...</p>
        </section>
        <section class="centered initialized">
            <div id="qrcode"></div>
        </section>
        <section class="centered waiting-for-user">
            <div class="irma-web-waiting-for-user-animation"></div>
            <p>Follow the instructions on your phone</p>
        </section>
        <section class="centered success">
            <div class="irma-web-checkmark-animation"></div>
            <p>Success!</p>
        </section>
        <section class="centered expired">
            <p>The transaction took long</p>
            <p><a href="#" onclick="nutsLogin.start()">Try again</a></p>
        </section>
        <section class="centered cancelled">
            <p>The transaction got cancelled</p>
            <p><a href="#" onclick="nutsLogin.start()">Try again</a></p>
        </section>
        <section class="centered errored">
            <p>Something went wrong</p>
            <p><a href="#" onclick="nutsLogin.start()">Try again</a></p>
        </section>
    </section>
  </section>

On line #16 you see ``<div id="qrcode"></div>``. This is the element were the
qrcode will be shown.
For this we use a external library `qrcode.js <https://davidshimjs.github.io/qrcodejs/>`_.

Both the qr-code library and the ``@nuts-foundation/auth`` library can be included using this snippet.
Insert it at the top of your ``views/login.ejs``

.. code-block:: html

  <!--The nuts auth styleguide -->
  <link rel="stylesheet" href="//nuts-foundation.github.io/irma-web-frontend/application.css" />
  <!--A lib to render qr-codes -->
  <script src="https://cdn.jsdelivr.net/gh/davidshimjs/qrcodejs@gh-pages/qrcode.min.js"></script>
  <!--The nuts auth js lib -->
  <script src="https://cdn.jsdelivr.net/npm/@nuts-foundation/auth@0.1.0/index.min.js"></script>

.. note::

  On production environments it is recommended to pin the version of included libraries.

To bring our login screen to live call the ``NutsLogin`` service as provided.
There are a few configuration options:

* **nutsAuthUrl** The external address of the nuts-node
* **qrEl** the element to load the qr-code in
* **nutsAuthUrl** the address of our bundy nuts node
* **postTokenPath** the backend path to POST the acquired token to after successful
  login. We will create a route for this later on.

.. code-block:: html

  <script>
    nutsLogin = NutsLogin.init({
      nutsAuthUrl: "http://localhost:11323",
      qrEl: 'qrcode',
      logLevel: 'debug',
      postTokenPath: '/users'
    })
    nutsLogin.start();
  </script>


Setup your nuts-node
^^^^^^^^^^^^^^^^^^^^

Since the user has to connect its phone with your Nuts server which is probably
running behind a NAT or firewall, it is ofter a good idea to use a service like
`ngrok <https://ngrok.com/>`_. The nuts node Bundy will be listening at ``11323``.

.. code-block:: console

  $ ngrok http 11323
  ...
  Session Status                online
  Region                        United States (us)
  Forwarding                    https://8bc613e1.ngrok.io -> http://localhost:11323

The `Forwarding` address is the external location your local nuts node can be found at.
In order for IRMA to work, you need to set the ``auth.publicUrl`` to this address:
copy the https url and paste it into the config file of `bundy`.

.. code-block:: yaml
  :linenos:
  :caption: nuts-network-local/config/bundy/nuts.yaml

  verbosity: debug
  address: :1323
  auth:
    actingPartyCn: Demo EHR
    publicUrl: https://8bc613e1.ngrok.io
    irmaConfigPath: /opt/nuts/irma
    enableCORS: true
  crypto:
    fspath: /opt/nuts/keys
  registry:
    datadir: /opt/nuts/registry
  events:
    zmqAddress: tcp://bundy-bridge:5563
  cbridge:
    address: http://bundy-bridge:8080
  cstore:
    connectionstring: /opt/nuts/sqlite/sqlite.db

.. note::

  Since we are letting the browser directly connect to the nuts-node, we need to enable CORS.


Now your ready to start up the nuts-node. We use the docker-compose setup as
shown by the :ref:`previous tutorial <Setup a local Nuts network>`.

.. note::

  To save memory and startup time, for this tutorial, you can disable every
  service in the `docker-compose.yml` except for the `bundy-nuts-service-space`
  and the network.

.. code-block:: console

  $ docker-compose up -V
  Creating network "nuts" with the default driver
  Creating nuts-network-local_bundy-nuts-service-space_1 ... done
  Attaching to nuts-network-local_bundy-nuts-service-space_1
  bundy-nuts-service-space_1  | time="2019-08-12T13:26:59Z" level=info msg="***************************************************************"
  bundy-nuts-service-space_1  | time="2019-08-12T13:26:59Z" level=info msg="*************************** Config ****************************"
  bundy-nuts-service-space_1  | time="2019-08-12T13:26:59Z" level=info msg="address                              :1323"
  bundy-nuts-service-space_1  | time="2019-08-12T13:26:59Z" level=info msg="configfile                           /opt/nuts/nuts.yaml"
  bundy-nuts-service-space_1  | time="2019-08-12T13:26:59Z" level=info msg="auth.actingPartyCn                   Demo EHR"
  bundy-nuts-service-space_1  | time="2019-08-12T13:26:59Z" level=info msg="auth.address                         localhost:1323"
  bundy-nuts-service-space_1  | time="2019-08-12T13:26:59Z" level=info msg="auth.enableCORS                      true"
  bundy-nuts-service-space_1  | time="2019-08-12T13:26:59Z" level=info msg="auth.publicUrl                       https://8bc613e1.ngrok.io"
  ...
  bundy-nuts-service-space_1  | time="2019-08-12T13:26:59Z" level=info msg="***************************************************************"
  bundy-nuts-service-space_1  | time="2019-08-12T13:26:59Z" level=info msg="irma baseurl: https://8bc613e1.ngrok.io/auth/irmaclient"
  bundy-nuts-service-space_1  | time="2019-08-12T13:27:02Z" level=debug msg="enabling CORS"
  ...
  bundy-nuts-service-space_1  | ⇨ http server started on [::]:1323

Try a first login
^^^^^^^^^^^^^^^^^

In order to login with IRMA, you need the app on your phone and make sure to
collect a demo agb code from the `IRMA attribute index <https://privacybydesign.foundation/attribute-index/en/irma-demo.nuts.agb.html>`_
Choose a random agb code with 8 digits. You can leave the `role` field empty.

Now it's time to test if everything works together. Restart your webserver
and navigate to `<localhost:3000/login>`_. If everything is alright you will
see a qr-code just like in the animation on top of this tutorial page. If not,
check the console log of your browser.

After scanning the qr-code you'll see a contract text on your phone which gives
the Demo EHR website permission to use the nuts network for 1 hour. After signing
the contract with the demo agb code, our demo EHR will redirect to the /user
endpoint. This is the subject of the next session of this tutorial.

The example code so far can be found on `the github repo <https://github.com/nuts-foundation/auth-tutorial/tree/working-login-page>`_

Handling a login token
^^^^^^^^^^^^^^^^^^^^^^

Please wait while our automatic tutorial generator enjoys his coffee/weekend. We will expand this section soon.
