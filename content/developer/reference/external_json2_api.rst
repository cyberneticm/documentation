===================
External JSON-2 API
===================

.. versionadded:: 19.0

Odoo is usually extended internally via modules, but many of its features and all of its data are
also available from the outside for external analysis or integration with various other softwares.
Part of the :ref:`reference/orm/model` API is easily available over HTTP via the ``/json/2``
endpoint. The actual models, their fields and methods are specific to every database and can be
consulted at the ``/doc`` page.


Request
=======

POST a json object at the ``/json/2/<model>/<method>`` URL. 


.. tabs::

   .. code-tab:: http

      POST /json/2/res.partner/search_read HTTP/1.1
      Host: mycompany.odoo.com
      X-Odoo-Database: mycompany
      Authorization: Bearer 6578616d706c65206a736f6e20617069206b6579
      Content-Type: application/json; charset=utf-8
      User-Agent: mysoftware python-requests/2.25.1

      {
         "ids": [],
         "context": {
            "lang": "en_US"
         }
         "domain": [
            ["name", "ilike", "%deco%"],
            ["is_company", "=", true]
         ],
         "fields": ["name"],
      }

   .. code-tab:: python

      import requests

      HOST = "https://mycompany.odoo.com"
      DATABASE = "mycompany"
      API_KEY = ...  # get it from a secure location

      response = requests.post(
          HOST + "/json/2/res.partner/search_read",
          headers={
              "Authorization": f"Bearer {API_KEY}",
              "X-Odoo-Database": DATABASE,
          },
          json={
              "ids": [],
              "context": {
                  "lang": "en_US",
              }
              "domain": [
                  ("name", "ilike", "%deco%"),
                  ("is_company", "=", True),
              ],
              "fields": ["name"],
          },
      )
      response.raise_for_status()
      data = response.json()

   .. code-tab:: javascript

      (async () => {
          const HOST = "https://mycompany.odoo.com";
          const DATABASE = "mycompany";
          const API_KEY = ;  // get it from a secure location

          const request = {
              method: "POST",
              headers: {
                  "Content-Type": "application/json",
                  "Authorization": "Bearer " + API_KEY,
                  "X-Odoo-Database": DATABASE,
              },
              body: {
                  domain: [
                      ["name", "ilike", "%deco%"],
                      ["is_company", "=", true],
                  ],
                  fields: ["name"],
              },
          };
    
          const response = await fetch(HOST + "/json/2/res.partner/search_read", request);
          if (response.ok) {
              const data = await response.json();
              console.log(data)
          } else {
              // Handle errors
          }
      })();

The body must be a json-object containing the arguments for the model's method. Both ``ids`` and
``context`` are special arguments: they are used to craft the environment and recordset on which the
method is executed.

The headers ``Host``, ``Authorization`` with an API key and ``Content-Type`` are required. The 
``X-Odoo-Database`` header is only necessary when multiple databases are hosted behind a same
``Host``. A ``User-Agent`` with the name of the software where the request comes from is
recommended.

The available models and methods depend on the list of modules that are installed in the database.
The exact list of what's available is accessible on the ``/doc`` page of every database.


Response
========

Success
-------

A **200 OK** status with the method's return value serialized as json in the body.

.. code:: http

   HTTP/1.1 200 OK
   Content-Type: application/json; charset=utf-8

   [
      {"id": 25, "name": "Deco Addict"}
   ]

Error
-----

A **4xx**/**5xx** status with the error message serialized as a json string in the body.

.. code:: http

   HTTP/1.1 401 Unauthorized
   Date: Fri, 18 Jul 2025 08:33:35 GMT
   Content-Type: application/json; charset=utf-8

   "Invalid apikey"

The complete traceback is available in the server log, at the same date as the error response.


API Key
=======

An API key must be set in the ``Authorization`` request header, as a bearer token.

Create a new API key for a user via :guilabel:`Preferences`, :guilabel:`Account Security`, and
:guilabel:`New API Key`.

.. have the three images appear next to each other
.. list-table::

   * - .. image:: external_api/preferences2.png
          :align: center

     - .. image:: external_api/account-security2.png
          :align: center

     - .. image:: external_api/new-api-key.png
          :align: center

A description and a duration are needed to create a new api key. The description makes it possible
to identify the key, and to determine later whether the key is still in use or should be removed.
The duration determines the lifetime of the key after which the the key becomes invalid. It is
recommended to set a short duration (typically 1 day) for interactive usage. It is not possible to
create keys that last for more than 3 months, it means that long lasting keys must be rotated at
least once every 3 months.

The :guilabel:`Generate Key` creates a 160 bits strong random key. Its value appears on screen, this
is the only time and place the key is visible on screen. It must be copied, kept secret and stored
somewhere secure. If it ever gets compromized or lost, then it must be removed.

Please refer to OWASP's `Secrets Management Cheat Sheet`_ for further guidance on the management of
API keys.

.. _Secrets Management Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html#secrets-management-cheat-sheet


Access Rights
=============

The JSON-2 API uses the standard :ref:`security <reference/security>` model of Odoo. All operations
are validated against the access rights, record rules and field accesses of the user.

For **interfactive usage**, such as discovering the API or running one-time scripts, it is fine to
use a **personal account**.

For **extended automated usage**, such as an integration with another software, it is recommended to 
create and use **dedicated bot users**.

Using dedicated bot users has several benefits:

* The minimum required permissions can be granted to the bot, limiting the impact may the API key
  gets compromised;
* The password can be set empty to disable login/password authentication, limiting the likelihood
  the account gets compromized;
* The :ref:`reference/fields/automatic/log_access` use the bot account. No user gets impersonalized.


Database
========

Depending on the deploiement, the ``Host`` and/or ``X-Odoo-Database`` request headers might be
required. The ``Host`` header is required on servers where Odoo is installed next to other web
applications, so a web-server/reverse-proxy is able to route the request to the Odoo server. The
``X-Odoo-Database`` header is required when a single Odoo server hosts multiple databases, and that
:ref:`dbfilter` wasn't configured to use the ``Host`` header.

Most HTTP client libraries automatically set the ``Host`` header using the connection url.


Transaction
===========


