.. _nuts-setup-local-network:

Setup a local Nuts network
--------------------------
Since Nuts is a distributed network, every party in the network runs its own node. A node consist of 3 parts:

* The `nuts-go <https://github.com/nuts-foundation/nuts-go>`_ application which behaves as the main access point for vendors
* A `Corda application <https://github.com/nuts-foundation/nuts-consent-cordapp>`_ which contains all the logic for signing and distributing patient consents
* A `bridge application <https://github.com/nuts-foundation/nuts-consent-bridge>`_ between the Corda application and the nuts-go application

All three applications are containerized and can be found on `Docker Hub <https://hub.docker.com/u/nutsfoundation>`_.

The easiest way for starting up a local development network is by using the docker-compose configuration.

This requires Docker to be installed.

+------------------------------------------------------+----------------------------------+
| Useful links                                         | Description                      |
+======================================================+==================================+
| `Docker <https://docs.docker.com/get-started/>`_     | Introduction into docker         |
+------------------------------------------------------+----------------------------------+
| `Docker compose <https://docs.docker.com/compose/>`_ | Introduction into docker-compose |
+------------------------------------------------------+----------------------------------+

Checkout the nuts network local repository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

At the time of writing v0.13.0 is the latest version. Make sure to use a stable version corresponding to this version of the documentation.

.. code-block:: console

   $ git clone https://github.com/nuts-foundation/nuts-network-local.git
   Cloning into 'nuts-network-local'...
   remote: Enumerating objects: 129, done.
   remote: Counting objects: 100% (129/129), done.
   remote: Compressing objects: 100% (86/86), done.
   remote: Total 129 (delta 62), reused 98 (delta 33), pack-reused 0
   Receiving objects: 100% (129/129), 286.29 KiB | 1.04 MiB/s, done.
   Resolving deltas: 100% (62/62), done.
   cd nuts-network-local
   $ git checkout tags/0.13.0

Inspect the package
^^^^^^^^^^^^^^^^^^^

This project is mainly a ``docker-compose.yml`` file, a cordapp and node configuration files.

The first node is called bundy, the second dahmer (any resemblance to serial killers is purely coincidental).
A little trick to keep the nodes apart: they are in alphabetical ordering and so are the port numbers etc.

When you inspect the `docker-compose.yml <https://github.com/nuts-foundation/nuts-network-local/blob/master/docker-compose.yml>`_
you will see two nodes with its 3 applications and a notary.

Each node has its own configuration directory in the config/node_name folder e.g. `dahmers config <https://github.com/nuts-foundation/nuts-network-local/tree/master/config/dahmer>`_.
It contains a private key, the bridge config (`application.properties <https://github.com/nuts-foundation/nuts-network-local/blob/master/config/dahmer/application.properties>`_)
and the nuts-go config (`nuts.yaml <https://github.com/nuts-foundation/nuts-network-local/blob/master/config/dahmer/nuts.yaml>`_) file.
Documentation about all these configuration parameters can be found in the :ref:`Configuration section<Configuration>`

The registry config is shared between nodes and contains the addresses and public keys of both nodes.
You are encouraged to inspect these files.

Generate the corda nodes
^^^^^^^^^^^^^^^^^^^^^^^^

To create a network of trust, Corda uses a bootstrapper tool. This tool must be downloaded
because it's to large to keep in version control. For more information about the bootstrapping process see the corda docs: https://docs.corda.net/docs/corda-os/4.4/network-bootstrapper.html
We provided a script to download and run the tool.

.. code-block:: console

  $ ./bootstrap-corda.sh
  Nuts network bootstrapper with Corda apps v0.13.0
  WARNING: This script removes all existing corda nodes and generate new ones.
  Do you want to continue? [Yes/No]Yes
  removing all nodes (if any)
  download new cordapps of version 0.13.0
  downloading corda network boostrapper
  running bootstrapper (this may take a while)
  Bootstrapping local test network in /opt/app
  Generating node directory for dahmer
  Generating node directory for bundy
  Generating node directory for notary
  Nodes found in the following sub-directories: [notary, dahmer, bundy]
  Found the following CorDapps: [flows-0.13.0.jar, contract-0.13.0.jar]
  Copying CorDapp JARs into node directories
  Waiting for all nodes to generate their node-info files...
  ... still waiting. If this is taking longer than usual, check the node logs.
  Distributing all node-info files to all nodes
  Loading existing network parameters... none found
  Gathering notary identities
  Generating contract implementations whitelist
  New NetworkParameters {
        minimumPlatformVersion=6
        notaries=[NotaryInfo(identity=CN=nuts_corda_development_notary, O=Nuts, L=Groenlo, C=NL, validating=false)]
        maxMessageSize=10485760
        maxTransactionSize=524288000
        whitelistedContractImplementations {

        }
        eventHorizon=PT720H
        packageOwnership {

        }
        modifiedTime=2020-04-01T15:08:48.560Z
        epoch=1
    }
  Bootstrapping complete!
  done

Populate Nuts registry
^^^^^^^^^^^^^^^^^^^^^^

The Nuts registry contains identities of vendors and care organizations and endpoints of resource servers.
In order to make use of the nodes the registry must be populated. For this a script is provided:

.. code-block:: console

  $ ./setup-network-registry.sh

You are asked to enter the names of two software vendors. This makes it easier to implement a use-case.
The script generated a lot of registry events. Take a look in config/registry/events.

Starting up the nodes
^^^^^^^^^^^^^^^^^^^^^

The Nuts node needs to accept some incoming connections from the IRMA app. To make this easier, we
provided a script that boots up ngrok and puts the endpoints in the nuts config.
The script will ask for a ngrok token during first boot. For this you will need a free ngrok account. Get your token at https://dashboard.ngrok.com/auth.

.. note::

  It may take some time (up to ~4 minutes) for all nodes to be booted up. If you see errors about ``Cannot connect to server(s)`` just wait a little longer

.. code-block:: console

  $ ./start-network.sh

Congratulations!! You just booted a full Nuts network on your local machine :)

Demo EHR
^^^^^^^^

To make interacting with Nuts a bit more fun we added a Demo EHR. There are 3 care organizations available on the addresses:

+---------------------------------+------------------------------+--------------------------+
| Care provider name              | Address                      | Corda Node               |
+=================================+==============================+==========================+
| Verpleeghuis de Nootjes         | http://localhost:8000        | Bundy                    |
+---------------------------------+------------------------------+--------------------------+
| Huisartsenpraktijk Nootenboom   | http://localhost:8001        | Dahmer                   |
+---------------------------------+------------------------------+--------------------------+
| Medisch Centrum Noot aan de Man | http://localhost:8002        | Dahmer                   |
+---------------------------------+------------------------------+--------------------------+

To register a consent, choose a patient who is know by both organizations. Luuk Meijer is such a patient who is known to Verpleeghuis de Nootjes and known to Huisartsenpraktijk Nootenboom.
Go to the patient, click in the network tab and Add a consent. This should take less than a minute. You can see the progress by going to the patients list and scroll down tot the Transactions section.

Record your first consent using the API
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To see if everything is set up correctly, we will create a consent request by posting to the consent-logic api.
More details about the api and its endpoints can be found :ref:`here<Nuts consent logic API>`

.. code-block:: console

  $ curl -X POST \
  http://localhost:11323/api/consent \
  -H 'Content-Type: application/json' \
  -d '{
    "subject": "urn:oid:2.16.840.1.113883.2.4.6.3:99999990",
    "custodian": "urn:oid:2.16.840.1.113883.2.4.6.1:12345678",
    "actor": "urn:oid:2.16.840.1.113883.2.4.6.1:87654321",
    "performer": "urn:oid:2.16.840.1.113883.2.4.6.1:00000007",
    "records": [{
      "consentProof": {
        "ID": "11112222-2222-3333-4444-555566667777",
        "title": "Toestemming inzage huisarts.pdf",
        "URL": "https://some.url/path/to/reference.pdf",
        "contentType": "application/pdf",
        "hash": "string"
      },
      "period": {
        "start": "2019-05-20T17:02:33+10:00",
        "end": "2019-11-20T17:02:33+10:00"
      },
      "dataClass": [
        "urn:oid:1.3.6.1.4.1.54851.1:MEDICAL"
      ]
    }]
  }'

You can check the status the process by querying the events endpoint of the event-store:

.. code-block:: console

  $ curl -X GET  http://localhost:11323/events
  {"events":[{"consentId":"7b8cf879-6d9f-4f62-b31e-165848ec5677","externalId":"13bf4c28d334d712c7297613cffc97d935c9972897d6fe678fc9f3ad8e354ada","initiatorLegalEntity":"","name":"completed","payload":"eyJjb25zZW50SWQiOnsiZXh0ZXJuYWxJZCI6IjEzYmY0YzI4ZDMzNGQ3MTJjNzI5NzYxM2NmZmM5N2Q5MzVjOTk3Mjg5N2Q2ZmU2NzhmYzlmM2FkOGUzNTRhZGEiLCJVVUlEIjoiN2I4Y2Y4NzktNmQ5Zi00ZjYyLWIzMWUtMTY1ODQ4ZWM1Njc3In0sIm1ldGFkYXRhIjp7ImRvbWFpbiI6WyJtZWRpY2FsIl0sInNlY3VyZUtleSI6eyJhbGciOiJBRVNfR0NNIiwiaXYiOiJpeWlJZDBmTzRyU3RUVHY4In0sIm9yZ2FuaXNhdGlvblNlY3VyZUtleXMiOlt7ImxlZ2FsRW50aXR5IjoidXJuOm9pZDoyLjE2Ljg0MC4xLjExMzg4My4yLjQuNi4xOjAwMDAwMDAxIiwiYWxnIjoiUlNBLU9BRVAiLCJjaXBoZXJUZXh0IjoiRnc2OFBjV0MwTkt4Ri9GSEFOUzRRNjFwMlJGbjQ3TWp2bUU5VGMzM2tJemhVWjhGRWF3UzJhWHZXdHNmeC9FSXBaU2dOeTVwVWpkUjhtMTdrSEJPYnZlUWduVlJhejIxcUllYWdWc2FUREhzNXVOZ1lIT3VUUThIY1BGdXNVNzcwWjJadzF1WHB1aHBxUXFsUnBtUXpyZCsvOGxxYUs1VTJkaEhFdVhKRXVBOVpkMmluTXhsL0FmUllPN0ZLUGEySUd6TnNPL0U3Y0djc3FBdlQ4RitXNUxRVjVtNTFza2xHejhYd3dwdnE4NjBRSnpzQk5Qa2NpbjRWamFLMkt0bERaQVNwVWFmWk1jNFE1emkyeDlNdlNVUzh6bjVGTEg1Q1I0OGVQazZ6TS9EQ0pxbGR6K2JvRm9LMmxpcEpEaEdaM2ZZK3VxUlE4NExsRE5nUzMveHJ3PT0ifSx7ImxlZ2FsRW50aXR5IjoidXJuOm9pZDoyLjE2Ljg0MC4xLjExMzg4My4yLjQuNi4xOjAwMDAwMDAwIiwiYWxnIjoiUlNBLU9BRVAiLCJjaXBoZXJUZXh0IjoiSnNBYmRPcWtzVE5iZ2Q0RzNZVHlSZmpRZXQ2bm1MMEdiSW1QQXBZU3hUZVZ2eWhmY0RueDdVdmYxQlkzR2FrU0VSUWFyTUFkemw3VWlidjFMcFh3YVRhb0o2WEZTd09iaUkvbWtLclVVaEI5ODJkS3dYVEh0cjd3RDdBTk5Bdysxamd1dllqRk50MVRmeVFsUUY2S01wRWI0dmRpVHJkWW4wc1RDY0o3MGV0a2szZWo5Qm0xR3VHMkswQmx5MmlDeTVBRHljZEU5aU0zY2taN1JLN2pSN1NKMnUzWnhrL1pXc3N4aWVYU0g1eldhbnpkUmVjb0FBejVYSDlRaUowZGxWbWN4TzEweXd0S040VHg0RVpVTnNEaVN1eDNOc0pWcGVBa0RYWS9RakJUa1pHNm1qUWhzTXRwRjl2dDhMZjB6ZHBjUWxTRnRlMnY3U0RMU2dKR0xBPT0ifV0sInBlcmlvZCI6eyJ2YWxpZEZyb20iOiIyMDE5LTA3LTAxVDAwOjAwOjAwWiIsInZhbGlkVG8iOiIyMDIwLTA3LTAxVDAwOjAwOjAwWiJ9fSwiY2lwaGVyVGV4dCI6Im1hcjltcjVLTDhWWFl2V1ZyVHJHY25td1ZuUXUxNUpORER0Z2NKMVZ3TUdBQlFJaUhwditiUUZrUzdYRE92MDhVaS9kNkdhcytJaHhPSDhOU043K2xmNjRiNUQzQzhYNkZPSi90cnN4eGs5c2FpY1JDb1VKTGZlaEI0M2RiaTFva0NES1k2TTBIWXZaRVRva0p4Q0pBU2plVXNDVVFxYnErSUxZRWtTY1Zka05LZno1azRkR3d4M0x5N1pNcFdNbDJxUDE4Zm1MRldGa28yMGRYcHVZQ0hINCtyUktmRytaNGM1czNmYUUvd3BNZER2Y2ptbWxqMTFXeGVLeC9nSmtFTFFPN3VQRFNsL1JIaGtMd2RqbEtzTXRod3FTUTZlbDY0a1g0UWpyeVFaUElPczZvSjFuUVRrM2loazJEZzY3NnJybzJicVQrRU9XYjRoYUp3UnhXSDF2eDF2czFYeVhwSitzVlhyTEw4S1V5aHFpbTNqZWJiL1Y3N3dsVDFDL2FsQzE3TThaN3NQVW13c1hBY3BYUDNFWEdWMFd2V21oa2RseUJhM2Nyb1h2TTJldWZKVVdzTHVBUWVkdnNMNW44czlad2F1VjBtR1pCOU5nQVRWVHhQNWJWWUZuKzZuM3JTV3lmUW91SnJKQW1nSEM1SHpLcEpCRU05VjM4Yk4vcWVpcTlhU0xqRUVHUjF5NXhRUUxUUW9pWkd3RksyZGo3MGRoOFlBcUJWL0JCQWF4cHlxSklPdXF0NTlzT2Z2YnhUclBaRENQWE12TU1ROU81TDhRUk9jRVNMRzltOTdzTHAwSVJyaU9aT0xxTEkwVVhpSUQ1dDAzV0ZnMmI0VGJybzJJWnlTQWE0bzZ2dlVhM09jakJvNVhqelhxL0V0SDFDUEN0N2pHK3QwWHlEUjg0WDNyVXdycVNZcC96U25LNjhvVWpoMjY4eExlNzRBZ3hBK0t5VlJnWVJSaC84bWt1S2Y2SFVwNXBIMG9xb05mMnZla2RNcTBRVHAvUXMvcXo4NExCR2dlSkNJWVYrbWZqZG9uSlJ0amJqU1BKbEhPQ0F0MXYySUhJWmxuUDhWRm9aZkNCRWlUdk1ibnBBRi9jbENzSFl3VnNCM2hnd0h0U1NCTXBtQ3FLUkYzY1hjeFN2T1hpZDR3QlV6UmFBVzA2TTZQYURTWXgvY29mRFlZQ3BXRDByTGgybGpRbE5hcWl0VE43Zjd4UVJucTJLbi95Y2ozRDVlMG9jb1BNWHR3L2ZuS0FQRVdXVCtwTG53VE9sVy9IU3VrWHN5VTM0NWErUXNjUEc2THR5cGY1bFV1U3p5UzBNWUxad3JnL1R3MmJ3cFprQWJZNEk5YUFPcUlIODYyOCtWQjlvdlFFc2Fjb0RrNlhxYXdKaGJYWGhUejdOSmF4c1ZGNkRNZWhvUWJiY0dhRHJpTlArbERPUXBxYjNVM1RodzRNQzYvWFVDRFh0NktqdTdJUXZWS2JJMHhzdlh4SkRWbDlxc0FFajM0aFlJYllUSmNSOUErZk96aVZ4bzJCVmdCWWFKZ2p6UW53T3dBb0xLOGlYS0hVZzBaNFJWajFYdC9UZ2VWbkU2K0l5TURhRFJrajh4RTJ2cVAvdEVCN25tYWFoK3JFTEg5b3JsVy9sa3dPR1VDQjRSYmFpNUx6Q0dUMkh5aFY0bE55US84ZnhXcUJJOFVUdXJ2b0FKQXZsQnhmckhYbnlnUWt0eHhQdWVCaVVOUUd0YUcxUmN3SEt4L3NpTGpCZ015V1BhNm10eXBYakdkVmFRUWRDOENFc3ZERnZrNHJERnpuLy9yZHY0bzBUZThtY0s5SUtMejdaYUo4YXZ5MGpUSTlHUUpienpmcDd0UDgrZ3FlYVFnTzBNQnIwUEU2ODlScldCK29XSGM0WGRTRnJ3SDQvYUxJTjdiSDg4Qjh6eFhLZHIvR1dGc3VWdDdWOFlWWHZDRlVTVUd2UFc5R24vNmxRZGpqMDNvdkt5eld5TWpIeTFDa0VVRDA4M0NxeTZjN3R4L2NzS2ptVHJnV2ZjUVFqZm4zRTBBMjF5cHEyZW1mclFJN3FpaTBzZFYyeWRLdE1lMmZmWW45ZW0xOTRPUlB0UWhmclpkYmVhMVo2dUI1K2JGV2trd3lxenZXcEZjSEh2N1dXSkdjZWZSWGF6MkNURWJSWEFxUTRIR3ZPd2QrSTRjbVpVSDNhOE5vdFNocG1PQ2xDUmoxYkg3dUJDQzVROXh1MENDWHZZOWk5QWROTVJSSjc2ZTRydEdGVGxhZmY2ckNoVUlUWEZHWDhDdXFOM2w2UHNDZDAxYWdhZlV1WlR3T3loZzJHM0pKRWhKK0gzcUhBM3pqRlJIZnp5eTV6WHVKZUFCMWtpODdKd1FXdVQwNnhtZDV1SkZsWGV4SzJVc0FITEVmZ0RoK2YxTldiYm5kUGJ1U2xSTm5ISmZ3MVQyMW1ZOHMxQ3ZWOFVHbXJ5NjgyWjN2YTM1WXVvWU5XakN2ZVZya205d2RqeWswOGk2UlZONDBrdnhpT2QxN3M2TXErc01UYTc5cUtWcXhobGVYNG1zUlUwK21zdmJkaHM0Rm5PRjJOQXE0VURPVVN1Y1NGa3hGRno0bjJyNHVwUlp6M0FFaVpoRHdvVm1XcHFZeUhMSVhIdjYwU0ZNSXAyMHQrVFBPMEF2aGVMT2JhU1VOWXpHZ3VJT0VjQ25nM0tFQTBxQlFLdTNhMjFLSTB5dG14bCtDMGZKY1FEK2hkY1JaR09NZHVja0R3RXFsV1FreUhTZTdLM2VXbEp5SFc4dGFYU0k2VlRRa1h0SzJaYWlEUTJiTkNJVUZpODJZQSthY1RLMWR0MitISm5mejdxeTVVNEczS21aQ3E1amlSV21XelJSa2FNeXVOOHM4dkVLd3hBSEQ0cFM5eWtYd1JuL01yaGNqY0g0dWtDR3lZTDVoNmc5ckdFSVRQWGErK05FT1NNSFpaS0RlbUNOMG84NFRZTUxKNlA0dVlCYXp5T2RibnRDeklPd3VITlJ3Q1JVeFpwS01FaEZRNWlYUFY3aXlKSEh5aDI4TjZ6Y2xvbzkrRzFYTlZQaVFaNHFkZzFnMDB1MldLalR2eUhNendlU2prZExKWEg1c2hTTGVTTFpDMHl6UWhEY0E4dlNwYTZVbUUzMVVZRnFXOFRSTnZoVVpncmNwSXUxQVVEdUdIQnRtZURsSDU0aVVVcWlWVG9ndkZPeGQ2bjBuTEV0VHY4b2ZtZVk0b3UraW5jbG81b09BclRETkl2UGQrWFRobUszVHg3RXNsSDFPdWZwVnk2YnVUZ1dmUkNOaHBrMEhwY002MXlvVng3T3p1MVN5OFpjMnBxL1hwUXRlUjkxUkJqY2JkeGdROHNVeVFQSWVhS1hETWpXeC9IcHR3UmFhanJIMlAvdG1JOFFjK0tTdEYrWnloV0xkbzdqdEJTNXNqMmg1aWJlYmVXeVQwemt0Um9ZdlF3SU43Z3ozTnFFdm5oeVIvdkErTVVOUXcwbUdqTVoya3J6UjZCT0RSSWNCQU9aMyszb1JKeDNoMU9Jc0tVQmlkS1hhUk9kUXNVR0szbEhjMHhvMXpjejJCcExvSURjaHpCSEhhdzMwaU45QVJlV2JoOVp4K1hQOWUzdGlRTDBoZmY2U20wN0hma3lOK3pDbEJ5R0VjbS9CS3VaRHhHZENJM2tsV3MzaG1Jb1NhTWpGUTlvZWhZNnNKYUZZZmpocnFWWVRqT1VDYkdTeDF2OC9XOGwvNU80RlBQYzRTKzI3cDNjK2pNPSJ9","retryCount":0,"uuid":"8d5a8857-064e-4cbd-b749-d17298335fcf"}]}

The ``"name":"completed"`` tells you the consent got successfully distributed over both nodes.
If you don't believe it, checkout the event on dahmer by performing the same request on address ``http://localhost:21323/events``.

To check consents for the combination of actor and subjec you can perform the following query to the consent-store:

.. code-block:: console

  $ curl -X POST \
  http://localhost:11323/consent/query \
  -H 'Content-Type: application/json' \
  -d '{
  "actor": "urn:oid:2.16.840.1.113883.2.4.6.1:87654321",
  "subject": "urn:oid:2.16.840.1.113883.2.4.6.3:99999990"
  }'

There are quite a few steps to perform by the two nuts nodes before the consent is recorded. Check out the :ref:`state machine with all the events here<Service space event specification>`.

These events get broadcast to all parties, including your own application. That's how you get notified about new patient consents.
Try to inspect the payload to see what you can expect. Hint: it's base64 encoded.

That's it. You have just booted an open source distributed health infrastructure on your computer and recorded your first consent.
If anything did not work out as described above, don't hesitate to :ref:`contact us<Contact>`.
