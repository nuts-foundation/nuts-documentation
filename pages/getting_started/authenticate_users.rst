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

For demo purposes we start with an empty generated express node.js application.
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

The code so far can be found at `this github branch <https://github.com/nuts-foundation/auth-tutorial/tree/empty-app>`_

Create login page
^^^^^^^^^^^^^^^^^

Ok, let's begin with making a place to handle session logic, a page to show the
above IRMA UI, some logic to validate and store the token in a session and some
logic to for the user to logout.
We can use the already generated ``/users`` route to show information about the
current logged in user so lets create a new route for session handling:

.. code-block:: javascript
  :name: /routes/session.js
  :caption: /routes/session.js
  :linenos:

  var express = require('express');
  var router = express.Router();

  router.get('/login', function (req, res, next) {
    res.render('login', {})
  });

  module.exports = router;


.. code-block:: html
  :name: /views/login.ejs
  :caption: /views/login.ejs
  :linenos:

  <h1>Login with IRMA</h1>

Now register the session router in `app.js`:

.. code-block:: diff
  :name: app.js

   var indexRouter = require('./routes/index');
   var usersRouter = require('./routes/users');
  +var sessionRouter = require('./routes/session');

   var app = express();
   app.use('/', indexRouter);
   app.use('/users', usersRouter);
  +app.use('/session', sessionRouter);

Change the landing page to display a login link:

.. code-block:: diff

    <p>Welcome to <%= title %></p>
  + <a href="session/login">Click here to login</a


Restart the app and navigate to `<http://localhost:3000/session/login>`_ to enjoy the result of your hard labour.

The code for progress so far can be found `at github <https://github.com/nuts-foundation/auth-tutorial/tree/empty-login-page>`_

Add login html and javascript
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To make it convenient to integrate Nuts with your existing application, we provided
some helpful tools.
We created a simple styleguide and a javascript IRMA state machine which we will
now embed on our freshly created login page.

Install the two npm packages:

.. code-block:: console

  $ npm add irma-web-frontend @nuts-foundation/auth

First start by pasting the following html which contains the instructions and
Now, embed the following html on the login.ejs page. The html is taken from the
`Nuts irma styleguide <https://nuts-foundation.github.io/irma-web-frontend/section-examples.html>`_.

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
        <canvas id="qrcode"></canvas>
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


On line #16 you see ``<canvas id="qrcode"></canvas>``. This is the element were the
qrcode will be shown.


In order to load the css and javascript from both libraries we need to configure
express to serve static content

.. code-block:: diff
  :caption: app.js

   app.use(express.static(path.join(__dirname, 'public')));
  +app.use('/scripts/nuts-auth', express.static(__dirname + '/node_modules/@nuts-foundation/auth/dist'));
  +app.use('/style/irma-web-frontend', express.static(__dirname + '/node_modules/irma-web-frontend/dist'));

Include them on top of the login.ejs page

.. code-block:: diff
  :caption: views/login.ejs

  +<link rel="stylesheet" href="/style/irma-web-frontend/irma-web-frontend.min.css" />
  +<script src="/scripts/nuts-auth/browser.js"></script>

   <section class="nuts-login-form irma-web-form">


To bring our login screen to life call the ``nutsAuth`` service as provided.
There are a few configuration options:

* **nutsAuthUrl** The external address of the nuts-node
* **qrEl** the canvas element to render the qr-code in
* **nutsAuthUrl** the address of our bundy nuts node
* **postTokenPath** the backend path to POST the acquired token to after successful
  login. We will create a route for this later on.

.. code-block:: html

  <script>
    nutsLogin = nutsAuth.init({
      nutsAuthUrl: "http://localhost:11323",
      qrEl: 'qrcode',
      logLevel: 'debug',
      postTokenPath: '/session/login',
      afterSuccessPath: '/users'
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
and navigate to `<localhost:3000/session/login>`_. If everything is alright you will
see a qr-code just like in the animation on top of this tutorial page. If not,
check the console log of your browser.

After scanning the qr-code you'll see a contract text on your phone which gives
the Demo EHR website permission to use the nuts network for 1 hour. After signing
the contract with the demo agb code, our demo EHR will redirect to the /user
endpoint. This is the subject of the next session of this tutorial.

The example code so far can be found on `the github repo <https://github.com/nuts-foundation/auth-tutorial/tree/working-login-page>`_

Handling a login token
^^^^^^^^^^^^^^^^^^^^^^

During the IRMA disclosure/signing session, the frontend is polling the nuts
node for status changes. After the user successfully discloses its agb code an
IRMA signature gets returned in the status update. Now the frontend will post
this signature to the ``postTokenPath`` of our Demo EHR web application.

When our backend receives the token, it needs to check that it is valid.
The npm package we already installed, contains a simple API wrapper.
Now, lets create that API endpoint which accepts a POST containing shall we?

The body of the post consists of a hash with only one key: ```nuts_auth_token``.
The value is a `base64` encoded IRMA signature. The decoded contents is not really
that interesting for us, but it is for our nuts node! With this token our node can
determine the user who signed the contract, and if the contract is still valid.

.. code-block:: javascript
  :linenos:
  :caption: routes/session.js

  const { nutsAuthClient } = require('@nuts-foundation/auth');

  router.post('/login', async function (req, res, next) {
    const nutsToken = req.body.nuts_auth_token

    const client = nutsAuthClient('http://localhost:11323', 'Demo EHR');
    const validationResponse = await client.validateToken(nutsToken);

    const sessionIsValid = validationResponse.validation_result === "VALID";

    if (!sessionIsValid) {
      res.status(403).send("token invalid");
      return;
    }

    // Store token in session information for one hour
    // Note: the cookie should be signed in production environments
    res.cookie('nutstoken', nutsToken, {maxAge: 60 * 60 * 1000});
    res.cookie('agb', validationResponse.signer_attributes['irma-demo.nuts.agb.agbcode'], {maxAge: 60 * 60 * 1000});

    res.status(200).send(validationResponse);
  });

This will take the token from the post body and post it to the the nuts-auth
server.

When the UI gets back the 200 it forwards the user to the ``/users`` page.
Lets make that one too:

.. code-block:: javascript
  :caption: routes/users.js

  /* GET users listing. */
  router.all('/', function(req, res, next) {
    // check if the cookie is set
    if ('nutstoken' in req.cookies) {
      next();
    } else {
      res.redirect('/session/login');
    }
  });

  // render the user page with the agb code
  router.get('/', function (req, res, next) {
    const agb = req.cookies.agb;
    res.render('user', {agb});
  });

.. code-block:: html
  :caption: views/user.ejs

  <h1>Welcome user</h1>
  <p>Your agb is: <%= agb %></p>

  <a href="/session/logout">Click here to logout</a>

We are on a roll! Lets create the logout route as well:

.. code-block:: javascript
  :caption: routes/session.js

  router.get('/logout', function (req, res, next) {
    res.clearCookie('nutstoken')
    res.clearCookie('agb')
    res.redirect('/')
  });

You did it! You created a fresh node.js express application. Included the
nuts npm packages. Added all the views and routes to show an IRMA UI. Contacted
the server to validate the token and stored it in the users session.
This token can be used to make request to the nuts network. A subject for a
future tutorial.
For now, play around with the code and try to embed this screen in your own
application.

The full demo source code is available on `this github branch <https://github.com/nuts-foundation/auth-tutorial/tree/validate-token>`_.
