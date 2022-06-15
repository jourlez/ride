##########
User guide
##########

*****************
User Requirements
*****************

===============
Install Ansible
===============

Ansible will take care of setting your development envireonment, but for that you need to install Ansible first.

------
Fedora
------

.. code-block:: none
    
    sudo dnf install ansible
    
----
RHEL
----

.. code-block:: none
    
    sudo yum install ansible
    
------
CentOS
------

.. code-block:: none
    
    sudo yum install epel-release
    sudo yum install ansible
    
------
Ubuntu
------

.. code-block:: none

    sudo apt install ansible

---
Pip
---

.. code-block:: none
    
    sudo pip3 install ansible


=====================
Install Ansible Roles
=====================

.. code-block:: none
    
    ansible-galaxy install -r ./ansible/requirements.yml

====================
Run Ansible Playbook
====================

.. code-block:: none
    
    ansible-playbook ./ansible/install.yml --ask-become-pass

At this point Ansible will take a few minutes to set everything up so please be patient.

================================
Verify Private Node and Explorer
================================

You can verify that the node is running through REST API at `http://localhost:6869 <http://localhost:6869>`_.

On top of that you can also see an empty chain of you local node rapidly being built using your instance of the Explorer at `http://localhost:3000 <http://localhost:3000>`_.

********************
How to Run the Tests
********************

1. The command to run tests is the following:

.. code-block:: none

    surfboard test

2. If you have several scenarios (for instance, you need a separate deployment script), you can run a particular scenario by running

.. code-block:: none

    surfboard test my-scenario.js

Surfboard will pick up all the tests files in ./test/ folder and run the scenario over a node through the endpoint configured in surfboard.config.json . After several seconds, you will see the test results.

3. There are several parameters you can use while testing:

    * -v, --verbose          logs all transactions and node responses
    * --env=env              which environment should be used for test
    * --variables=variables  env variables can be set for usage in tests via env.{variable_name}. E.g.: MY_SEED="seed phraze",DAPP_ADDRESS=xyz, AMOUNT=1000

********
Examples
********

====================
Deposit and Withdraw
====================

This example will guide you through all the necessary steps for developing a simple decentralised application(dApp) in which anyone can deposit as many tokens to the dApp as they want, but they can only withdraw their own waves.

Navigate to the ./ride/examples/01/ folder which is where your Ride Project will be located in. Then run the following command:

.. code-block:: none
    
    surfboard init

If you get a warning saying the directory is not empty continue anyways by selecting yes, this part is very important. The above will produce an output similar to this one:

.. code-block:: none
    
    ~/ride/examples/01$ surfboard init
    ❗️Generated new local config
    Testnet seed="chapter position silver stage later verify account organ ride bronze emotion scissors"
    Testnet address="3MptwsrUdoZD5pnLTVmy7BuLE14G8sRp83V"❗
    Project initialization... done

The seed and address on the previous message are there because while creating you new Ride project, Surfboard also created a new wallet for you on the Testnet node of the Blockchain. Make sure to save them both since you will require them for the next steps.

-----------
Source Code
-----------

.. code-block:: none
    
    .    
    ├── surfboard.config.json 
    ├── ride  
    │    └── wallet.ride
    ├── scripts  
    │    └── wallet.deploy.js 
    └── test  
        └── wallet.ride-test.js 

* surfboard.config.json is the configuration file for your Ride project.
* ride/ contains all of your Ride Scripts(Smart Contracts/dApps).
* test/ has all your test files, in this case powered by javascript’s `Mocha <https://mochajs.org/>`_ framework. 
* scripts/ holds scripts used for deploying your smart contracts or dApps.

^^^^^^^^^^^^^^^^^^^^^
surfboard.config.json
^^^^^^^^^^^^^^^^^^^^^

.. literalinclude:: ../examples/01/surfboard.config.json
   :language: json

The envs (environment) entries which are: custom and testnet are important since these are used to set the default deployment environment for your dApps. Each environment is configured by:

* API_BASE: The REST API endpoint of the node that will be used for running a dApp as well as the CHAIN_ID of the network.
* SEED: The seed phrase for account with tokens, which will be the source of all the tokens in your test. As you noticed, the SEED for the testnet entry matches the seed of the account Surfboard automatically created for you.

^^^^^^^^^^^
wallet.ride
^^^^^^^^^^^

.. literalinclude:: ../examples/01/ride/wallet.ride
   :language: ride

There are two @Callable functions available for invocation via :ref:`InvokeScriptTransaction <docs:03_advanced:InvokeScriptTransaction>`.

* deposit(), which requires attached payment.
* withdraw(amount: Int), which transfers tokens back to the caller upon successful execution.

Throughout the dApp lifecycle, there will be a mapping(address → amount) maintained:

.. csv-table:: Mapping(address → amount)
  :file: _static/01_ride/tables/01_Mapping.csv 
  :header-rows: 1 
  :class: longtable
  :widths: 1 3

^^^^^^^^^^^^^^^^
wallet.deploy.js
^^^^^^^^^^^^^^^^

.. literalinclude:: ../examples/01/scripts/wallet.deploy.js
   :language: js

^^^^^^^^^^^^^^^^^^^
wallet.ride-test.js
^^^^^^^^^^^^^^^^^^^

.. literalinclude:: ../examples/01/test/wallet.ride-test.js
   :language: js

There's a before function and three tests. Let's review them:

* The before function funds a few accounts via :ref:`MassTransferTransaction <docs:03_advanced:MassTransferTransaction>` using `setupAccounts <https://wavesplatform.github.io/js-test-env/globals.html#setupaccounts>`_ (foo, bar and wallet),  `compiles <https://wavesplatform.github.io/js-test-env/globals.html#compile>`_ the script and creates a `set-script <https://wavesplatform.github.io/js-test-env/globals.html#setscript>`_ transaction of the compiled script which assigns it to the account under the name of wallet and `broadcasted <https://wavesplatform.github.io/js-test-env/globals.html#broadcast>`_ to the blockchain and then mined into a block via `waitForTx <https://wavesplatform.github.io/js-test-env/globals.html#waitfortx>`_.
* The "Can deposit" test verifies correct deposits are being processed successfully; signs and broadcasts an :ref:`InvokeScriptTransaction <docs:03_advanced:InvokeScriptTransaction>` invoking deposit() function with an `invokescript <https://wavesplatform.github.io/js-test-env/globals.html#invokescript>`_, for two accounts that were setup in the before function which are foo and bar. Check out the `invokescript parameters <https://wavesplatform.github.io/js-test-env/interfaces/iinvokescriptparams.html>`_
* The "Can't withdraw more than was deposited" test verifies that no-one can “steal” another account’s tokens
* The "Can withdraw" test verifies that correct withdrawals are being processed successfully.

-----
Tests
-----

When you run the tests Surfboard will pick up all the test files inside the ./test/ folder and run them over a node through the endpoint configured in surfboard.config.json. After several seconds, you will see an output like this one:

.. code-block:: none
    
    ~/ride/examples/01$ surfboard test
    Starting test with "custom" environment
    Root address: 3M4qwDomRabJKLZxuXhwfqLApQkU592nWxF

     wallet test suite
    Generating accounts with nonce: d1ef4b3
    Account generated: foofoofoofoofoofoofoofoofoofoofoo#d1ef4b3 - 3M9AKTNRKmFkwGQmXiTUZz9cE8haU3mNdVP
    Account generated: barbarbarbarbarbarbarbarbarbar#d1ef4b3 - 3M28pYYfjiUXFWvmHwdQnbgLYP2jkDDF3iL
    Account generated: wallet#d1ef4b3 - 3MBAK4oVXxWStk6JQJwcLK9riXhwFdujksw
    Accounts successfully funded

    Script has been set
        ✓ Can deposit (2592ms)
        ✓ Cannot withdraw more than was deposited (38ms)
        ✓ Can withdraw (67ms)

    3 passing (7s)

Now you can look more closely at what happened using your `Explorer <http://localhost:3000>`_ instance by just browsing blocks or pasting one of the addresses above to the search box. Remember to check the Blockchain Network to make sure you have selected the correct one(Custom in this case).

To test against a different environment like testnet, just use the following command:

.. code-block:: none
    
    surfboard test --env=testnet

If you want to explore what information is sent to and from the nodes during the calls of broadcast(...) in a test, run a test with -v (stands for “verbose”) to make Surfboard produce extensive logs:

.. code-block:: none
    
    surfboard test -v
    
----------
Deployment
----------

You can also deploy your dApp to the wallet created for you by the surfboard init command. To do that, run the command below:

.. code-block:: none
    
    surfboard run ./scripts/wallet.deploy.js --env=testnet --variables dappSeed="dApp seed"

When you run the above command, you will get an error(which is expected) because as you already know in order to deploy a dApp you would need to fund the wallet that will be scripted. You can request funding for your account using the `faucet <https://testnet.wavesexplorer.com/faucet>`_ and typing the address of the account Surfboard generated for you.

Now that your account has funds run the command once more and it will output the transaction id associated with the deployment carried out by the :ref:`wallet.ride-test.js <01_ride:wallet.ride-test.js>` file which you can look up using the `Explorer <http://localhost:3000>`_ instance. The output will look something like this:

.. code-block:: none
    
    ~/ride/examples/01$ surfboard run ./scripts/wallet.deploy.js --env=testnet --variables dappSeed="dApp seed"
    6TBUs1ZSTDB8LZeDE8jGTwhGMLCvPKe8pBM1FWnHzYe7

------------------
Keeper Integration
------------------

.. 
    ----------------
    Cubensis Connect
    ----------------
    If you want to go the extra mile this is your chance: you can integrate the project you have been working on with Cubensis Connect. Clone the GitHub repository on your computer using the command:

    .. code-block:: none
        
        git clone git@github.com:Decentral-America/CubensisConnect.git

    Go to your Chromium based browser of preference and load the unpacked extension you just cloned like this:

    * Open the Extension Management page by navigating to `chrome://extensions <chrome://extensions>`_.
        * Alternatively, open this page by clicking on the Extensions menu button and selecting Manage Extensions at the bottom of the menu.
        * Alternatively, open this page by clicking on the Chrome menu, hovering over More Tools then selecting Extensions
    * Enable Developer Mode by clicking the toggle switch next to Developer mode.
    * Click the Load unpacked button and select the extension directory.

    Open Cubensis Connect and click Get Started, choose a password and then Continue. On the bottom left instead of Mainnet select Custom as the Blockchain Network and set the Node Address to the one below which as you will notice is the address you assigned to your private node, then Save and Apply.

If you want to go the extra mile this is your chance: you can integrate the project you have been working on with Keeper. Go to your browser of preference and open the `official website <https://keeper-wallet.app/>`_ for the tool and follow the installation instructions based on the type of browser you're using.

Open Keeper and click Get Started, choose a password and then Continue. On the bottom left instead of Mainnet select Custom as the Blockchain Network and set the Node Address to the one below which as you will notice is the address you assigned to your private node, then Save and Apply.

.. code-block:: none
    
    http://localhost:6869

Import a seed with tokens for the Blockchain Network. For simplicity, just use the staking seed of your node: 

.. code-block:: none
    
    waves private node seed with waves tokens

Choose a name for the account, any name will do. Then start an instance of a `1-page serverless web appliction <https://raw.githubusercontent.com/jourlez/ride/master/docs/_static/01_ride/files/02_index.html>`_ using npm with the following command:

.. code-block:: none
    
    http-server-node --port 5050 --entryPoint ../../docs/_static/01_ride/files/02_index.html

Check the instance you just started at:

.. code-block:: none
    
    http://localhost:5050

Enter the account address of the account generated for you by the command surfboard test under the name of wallet during testing into the script address blank text field.

Type an amount and select Deposit which will cause Keeper to request your permission to sign an :ref:`InvokeScriptTransaction <docs:03_advanced:InvokeScriptTransaction>` with an attached payment of the amount you typed. 

Approve the transaction so it will be created and broadcasted to the network. You can use the transaction "View transaction" button at the bottom to get the transaction id and that way you can look up all the details using the `Explorer <http://localhost:3000>`_ instance. Remember to check the Blockchain Network to make sure you have selected the correct one(Custom in this case).

.. note:: You could also use a `service <https://waves-dapp.com/>`_ that allows you to automatically generate an interface for a dApp using the names and data of the arguments of your callable functions. Just sign in using Keeper and again enter the account address of the account generated for you by the command surfboard test under the name of wallet during testing into the smart contract blank text field.

Now you have a good foundation to start developing using Ride, feel free to modify this example as you like and try to play a little bit more with it so you can learn more on your own.

############################
Developer Guide/Contributing
############################

**********************
Developer Requirements
**********************

Follow the same process as specified in :ref:`user requirements <01_ride:User Requirements>`.

*****************
Project Structure
*****************

.. code-block:: none
    
    .    
    ├── ansible/ 
    │    ├── install.yml
    │    ├── requirements.yml
    │    ├── group_vars/
    │    │   └── all.yml
    │    └── roles/
    │        ├── deploy-node-explorer/tasks/
    │        │   ├── deploy-node-explorer
    │        │   └── main.yml
    │        └── update-system/tasks/
    │            ├── main.yml
    │            └── update-system.yml
    ├── docs/
    │    ├── buildDocs.sh
    │    ├── conf.py
    │    ├── index.rst
    │    ├── make.bat
    │    ├── Makefile
    │    ├── _build/
    │    ├── _static/
    │    ├── _templates/
    │    └── locales/
    └── examples/
        └── 01

ansible folder:

* install.yml is the main playbook in charge of running all the roles used for project setup.
* requirements.yml contains all the required roles for the main playbook to work properly.
* group_vars/all.yml has all the variables used in the main playbook.
* roles/ is the folder containing each and every one of the roles that will be used by the main playbook.

docs folder:

* buildDocs.sh is your build script which will be executed in a docker container on GitHub's cloud.
* conf.py is a Python script holding the configuration of the Sphinx project. It contains the project name and release you specified to sphinx-quickstart, as well as some extra configuration keys.
* index.rst is the root document of the project, which serves as welcome page and contains the root of the “table of contents tree” (or toctree).
* make.bat and Makefile are convenience scripts to simplify some common Sphinx operations, such as rendering the content.
* _static/ will contain custom stylesheets and other static files like images.
* _templates/ will contain custom html template files overriding jinja templates.
* locales/ will contain all the translated versions of the documentation.

*******************
Document Generation
*******************

====
HTML
====

.. code-block:: none
    
    sphinx-build -b html docs/ docs/_build/html

===
PDF
===

.. code-block:: none
    
    sphinx-build -b rinoh docs/ docs/_build/rinoh -D language="en"

====
EPUB
====

.. code-block:: none
    
    sphinx-build -b epub docs/ docs/_build/epub -D language="en"

***************************
Internationalization (i18n)
***************************

Run the command below from the 'docs/' directory. This tells sphinx to parse your reST files and automatically find a bunch of strings-to-be-translated and give them a unique msgid that will be used during translation.

.. code-block:: none
    
    make gettext

After this, .pot files will be generated for you. You can verify it using the command below.

.. code-block:: none
    
    ls _build/gettext/

Next, let's tell sphinx to prepare some Spanish destination-language '.po' files from your above-generated source-lananguage '.pot' files. Execute the following command to prepare your Spanish-specific translation files.

.. code-block:: none
    
    sphinx-intl update -p _build/gettext -l es

The above execution create '.po' files: one for each of your '.pot' source-language files, which correlate directly to each of your two '.rst' files. For example, for the new Spanish-specific 'docs/locales/es/LC_MESSAGES/index.po' file, as you see it has the same contents as the source '.pot' file.

These language-specific '.po' files are where you actually do the translating. If you're a large project, then you'd probably want to use a special program or service to translate these files.

After translating everything you can build your html static content in the language you want, for example Spanish:

.. code-block:: none
    
    sphinx-build -b html . _build/html/en -D language='es'