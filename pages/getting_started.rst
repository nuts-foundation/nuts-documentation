Getting Started With Nuts
=========================

Welcome to the Nuts Getting Started guide. Nuts can help you with mainly two
common challenges in data exchange between Care Providers:

* Getting data from another party with consent from the patient
* Providing data to another party with consent from the patient

Some parties need to support only one use-ase, others need both.

Whatever your situation is, you always need to setup your Nuts node first.

This tutorial will help you with:

* :ref:`Setting up a local Nuts network <Setup a local Nuts network>`
* :ref:`Letting a user identify itself <Let users authenticate themselves to the Nuts network>`
* :ref:`Record a patients consent <Let patients record their consent>`
* :ref:`Validate incoming api request <Validate incoming api requests>`

Setup a local Nuts network
--------------------------
Since Nuts is a distributed network, every party in the network runs its own node. A node consist of 3 parts:

* The `nuts-go <https://github.com/nuts-foundation/nuts-go>`_ application which behaves as the main access point for vendors
* A `Corda application <https://github.com/nuts-foundation/nuts-consent-cordapp>`_ which contains all the logic for signing and distributing patient consents
* A `bridge application <https://github.com/nuts-foundation/nuts-consent-bridge>`_ between the Corda application and the nuts-go application

All three applications are containerized and can be found on `Docker hub <https://hub.docker.com/u/nutsfoundation>`_.

The easiest way for starting up a local development network is by using the docker-compose configuration.

This requires docker and Java installed

+------------------------------------------------------+----------------------------------+
| Useful links                                         | Description                      |
+======================================================+==================================+
| `Docker <https://docs.docker.com/get-started/>`_     | Introdcution into docker         |
+------------------------------------------------------+----------------------------------+
| `Docker compose <https://docs.docker.com/compose/>`_ | Introdcution into docker-compose |
+------------------------------------------------------+----------------------------------+

Checkout the nuts network local repository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: console

   $ git clone https://github.com/nuts-foundation/nuts-network-local.git
   Cloning into 'nuts-network-local'...
   remote: Enumerating objects: 129, done.
   remote: Counting objects: 100% (129/129), done.
   remote: Compressing objects: 100% (86/86), done.
   remote: Total 129 (delta 62), reused 98 (delta 33), pack-reused 0
   Receiving objects: 100% (129/129), 286.29 KiB | 1.04 MiB/s, done.
   Resolving deltas: 100% (62/62), done.

Inspect the package
^^^^^^^^^^^^^^^^^^^

This project is mainly a ``docker-compose.yml`` file, a cordapp and node configuration files.

The first node is called bundy, the second dahmer (Any resemblance to serial killers is purely coincidental).

When you inspect the `docker-compose.yml <https://github.com/nuts-foundation/nuts-network-local/blob/master/docker-compose.yml>`_ you will see two nodes with its 3 applications and a notary.

Each node has its own configuration directory in the config/node_name folder e.g. `dahmers config <https://github.com/nuts-foundation/nuts-network-local/tree/master/config/dahmer>`_.
It contains a private key, the bridge config (`application.properties <https://github.com/nuts-foundation/nuts-network-local/blob/master/config/dahmer/application.properties>`_) and the nuts-go config (`nuts.yaml <https://github.com/nuts-foundation/nuts-network-local/blob/master/config/dahmer/nuts.yaml>`_) file.
Documentation about all these configuration parameters can be found in the :ref:`Configuration section<Configuration>`

The registry config is shared between nodes and contains the addresses and public keys of both nodes.
You are encouraged to inspect these files.

Generate the corda nodes
^^^^^^^^^^^^^^^^^^^^^^^^

Get the Corda network bootstrapper tool from https://repo1.maven.org/maven2/net/corda/corda-tools-network-bootstrapper/4.1/
and generate the 2 network nodes + notary:

.. code-block:: console

  $ cd nuts-network-local
  $ cd nodes
  $ curl -O https://repo1.maven.org/maven2/net/corda/corda-tools-network-bootstrapper/4.1/corda-tools-network-bootstrapper-4.1.jar
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                Dload  Upload   Total   Spent    Left  Speed
  100  108M  100  108M    0     0  27.7M      0  0:00:03  0:00:03 --:--:-- 27.7M

  $ java -jar corda-tools-network-bootstrapper-4.1.jar --dir .
  Bootstrapping local test network in /Users/steven.vandervegt/projects/nuts/playground/nuts-network-local/nodes
  Generating node directory for dahmer
  Generating node directory for bundy
  Generating node directory for notary
  Nodes found in the following sub-directories: [notary, dahmer, bundy]
  Found the following CorDapps: [contract-0.6.1.jar, flows-0.6.1.jar]
  Copying CorDapp JARs into node directories
  Waiting for all nodes to generate their node-info files...
  ... still waiting. If this is taking longer than usual, check the node logs.
  Distributing all node-info files to all nodes
  Loading existing network parameters... none found
  Gathering notary identities
  Generating contract implementations whitelist
  QUASAR WARNING: Quasar Java Agent isn't running. If you're using another instrumentation method you can ignore this message; otherwise, please refer to the Getting Started section in the Quasar documentation.
  New NetworkParameters {
        minimumPlatformVersion=4
        notaries=[NotaryInfo(identity=CN=nuts_corda_development_notary, O=Nuts, L=Groenlo, C=NL, validating=false)]
        maxMessageSize=10485760
        maxTransactionSize=524288000
        whitelistedContractImplementations {
          nl.nuts.consent.contract.ConsentContract=[C689369163A4B1A2960AC90193ADD6C8470D6E48D57199B5DCC7B359C9953AC6]
        }
        eventHorizon=PT720H
        packageOwnership {

        }
        modifiedTime=2019-08-08T13:40:37.387Z
        epoch=1
    }
  Bootstrapping complete!

Starting up the nodes
^^^^^^^^^^^^^^^^^^^^^

.. note::

  if you get the following error:
  ``msg="Could not initialize IRMA library:" error="Error parsing scheme manager irma-demo: Could not read scheme manager timestamp: <nil>"``
  This happens sometimes during first boot since the irma schema is shared between nodes and one of the nodes is downloading it, the other cant access it.

.. note::

  It may take some time (up to ~4 minutes) for all nodes to be booted up. If you see errors about ``Cannot connect to server(s)`` just wait a little longer

.. code-block:: console

  $ docker-compose up -V


.. raw:: html

  <script id="asciicast-GXiWcMMk8nAgPwKHdfswtkGC8" src="https://asciinema.org/a/GXiWcMMk8nAgPwKHdfswtkGC8.js" data-speed="1.7" data-idle-time-limit="0.15" async></script>


Congratulations!! You just booted a full Nuts network on you local machine :)

Record your first consent
^^^^^^^^^^^^^^^^^^^^^^^^^

To see if everything is set up correctly, we will create a consent request by posting to the consent-logic api.
More details about the api and its endpoints can be found :ref:`here<Nuts consent logic API>`

.. code-block:: console

  $ curl -X POST \
  http://localhost:11323/api/consent \
  -H 'Content-Type: application/json' \
  -d '{
    "subject": "urn:oid:2.16.840.1.113883.2.4.6.3:999999990",
    "custodian": "urn:oid:2.16.840.1.113883.2.4.6.1:00000000",
    "actors": [
        "urn:oid:2.16.840.1.113883.2.4.6.1:00000001"
    ],
    "performer": "urn:oid:2.16.840.1.113883.2.4.6.1:00000007",
    "period": {
        "start": "2019-07-01T12:00:00+02:00",
        "end": "2020-07-01T12:00:00+02:00"
    }
  }'
  {"Actors":["urn:oid:2.16.840.1.113883.2.4.6.1:00000001"],"ConsentProof":null,"Custodian":"urn:oid:2.16.840.1.113883.2.4.6.1:00000000","Performer":"urn:oid:2.16.840.1.113883.2.4.6.1:00000007","Period":{"End":"2020-07-01T12:00:00+02:00","Start":"2019-07-01T12:00:00+02:00"},"Subject":"urn:oid:2.16.840.1.113883.2.4.6.3:999999990"}

You can check the status the process by querying the events endpoint of the event-store:

.. code-block:: console

  $ curl -X GET  http://localhost:11323/events
  {"events":[{"consentId":"7b8cf879-6d9f-4f62-b31e-165848ec5677","externalId":"13bf4c28d334d712c7297613cffc97d935c9972897d6fe678fc9f3ad8e354ada","initiatorLegalEntity":"","name":"completed","payload":"eyJjb25zZW50SWQiOnsiZXh0ZXJuYWxJZCI6IjEzYmY0YzI4ZDMzNGQ3MTJjNzI5NzYxM2NmZmM5N2Q5MzVjOTk3Mjg5N2Q2ZmU2NzhmYzlmM2FkOGUzNTRhZGEiLCJVVUlEIjoiN2I4Y2Y4NzktNmQ5Zi00ZjYyLWIzMWUtMTY1ODQ4ZWM1Njc3In0sIm1ldGFkYXRhIjp7ImRvbWFpbiI6WyJtZWRpY2FsIl0sInNlY3VyZUtleSI6eyJhbGciOiJBRVNfR0NNIiwiaXYiOiJpeWlJZDBmTzRyU3RUVHY4In0sIm9yZ2FuaXNhdGlvblNlY3VyZUtleXMiOlt7ImxlZ2FsRW50aXR5IjoidXJuOm9pZDoyLjE2Ljg0MC4xLjExMzg4My4yLjQuNi4xOjAwMDAwMDAxIiwiYWxnIjoiUlNBLU9BRVAiLCJjaXBoZXJUZXh0IjoiRnc2OFBjV0MwTkt4Ri9GSEFOUzRRNjFwMlJGbjQ3TWp2bUU5VGMzM2tJemhVWjhGRWF3UzJhWHZXdHNmeC9FSXBaU2dOeTVwVWpkUjhtMTdrSEJPYnZlUWduVlJhejIxcUllYWdWc2FUREhzNXVOZ1lIT3VUUThIY1BGdXNVNzcwWjJadzF1WHB1aHBxUXFsUnBtUXpyZCsvOGxxYUs1VTJkaEhFdVhKRXVBOVpkMmluTXhsL0FmUllPN0ZLUGEySUd6TnNPL0U3Y0djc3FBdlQ4RitXNUxRVjVtNTFza2xHejhYd3dwdnE4NjBRSnpzQk5Qa2NpbjRWamFLMkt0bERaQVNwVWFmWk1jNFE1emkyeDlNdlNVUzh6bjVGTEg1Q1I0OGVQazZ6TS9EQ0pxbGR6K2JvRm9LMmxpcEpEaEdaM2ZZK3VxUlE4NExsRE5nUzMveHJ3PT0ifSx7ImxlZ2FsRW50aXR5IjoidXJuOm9pZDoyLjE2Ljg0MC4xLjExMzg4My4yLjQuNi4xOjAwMDAwMDAwIiwiYWxnIjoiUlNBLU9BRVAiLCJjaXBoZXJUZXh0IjoiSnNBYmRPcWtzVE5iZ2Q0RzNZVHlSZmpRZXQ2bm1MMEdiSW1QQXBZU3hUZVZ2eWhmY0RueDdVdmYxQlkzR2FrU0VSUWFyTUFkemw3VWlidjFMcFh3YVRhb0o2WEZTd09iaUkvbWtLclVVaEI5ODJkS3dYVEh0cjd3RDdBTk5Bdysxamd1dllqRk50MVRmeVFsUUY2S01wRWI0dmRpVHJkWW4wc1RDY0o3MGV0a2szZWo5Qm0xR3VHMkswQmx5MmlDeTVBRHljZEU5aU0zY2taN1JLN2pSN1NKMnUzWnhrL1pXc3N4aWVYU0g1eldhbnpkUmVjb0FBejVYSDlRaUowZGxWbWN4TzEweXd0S040VHg0RVpVTnNEaVN1eDNOc0pWcGVBa0RYWS9RakJUa1pHNm1qUWhzTXRwRjl2dDhMZjB6ZHBjUWxTRnRlMnY3U0RMU2dKR0xBPT0ifV0sInBlcmlvZCI6eyJ2YWxpZEZyb20iOiIyMDE5LTA3LTAxVDAwOjAwOjAwWiIsInZhbGlkVG8iOiIyMDIwLTA3LTAxVDAwOjAwOjAwWiJ9fSwiY2lwaGVyVGV4dCI6Im1hcjltcjVLTDhWWFl2V1ZyVHJHY25td1ZuUXUxNUpORER0Z2NKMVZ3TUdBQlFJaUhwditiUUZrUzdYRE92MDhVaS9kNkdhcytJaHhPSDhOU043K2xmNjRiNUQzQzhYNkZPSi90cnN4eGs5c2FpY1JDb1VKTGZlaEI0M2RiaTFva0NES1k2TTBIWXZaRVRva0p4Q0pBU2plVXNDVVFxYnErSUxZRWtTY1Zka05LZno1azRkR3d4M0x5N1pNcFdNbDJxUDE4Zm1MRldGa28yMGRYcHVZQ0hINCtyUktmRytaNGM1czNmYUUvd3BNZER2Y2ptbWxqMTFXeGVLeC9nSmtFTFFPN3VQRFNsL1JIaGtMd2RqbEtzTXRod3FTUTZlbDY0a1g0UWpyeVFaUElPczZvSjFuUVRrM2loazJEZzY3NnJybzJicVQrRU9XYjRoYUp3UnhXSDF2eDF2czFYeVhwSitzVlhyTEw4S1V5aHFpbTNqZWJiL1Y3N3dsVDFDL2FsQzE3TThaN3NQVW13c1hBY3BYUDNFWEdWMFd2V21oa2RseUJhM2Nyb1h2TTJldWZKVVdzTHVBUWVkdnNMNW44czlad2F1VjBtR1pCOU5nQVRWVHhQNWJWWUZuKzZuM3JTV3lmUW91SnJKQW1nSEM1SHpLcEpCRU05VjM4Yk4vcWVpcTlhU0xqRUVHUjF5NXhRUUxUUW9pWkd3RksyZGo3MGRoOFlBcUJWL0JCQWF4cHlxSklPdXF0NTlzT2Z2YnhUclBaRENQWE12TU1ROU81TDhRUk9jRVNMRzltOTdzTHAwSVJyaU9aT0xxTEkwVVhpSUQ1dDAzV0ZnMmI0VGJybzJJWnlTQWE0bzZ2dlVhM09jakJvNVhqelhxL0V0SDFDUEN0N2pHK3QwWHlEUjg0WDNyVXdycVNZcC96U25LNjhvVWpoMjY4eExlNzRBZ3hBK0t5VlJnWVJSaC84bWt1S2Y2SFVwNXBIMG9xb05mMnZla2RNcTBRVHAvUXMvcXo4NExCR2dlSkNJWVYrbWZqZG9uSlJ0amJqU1BKbEhPQ0F0MXYySUhJWmxuUDhWRm9aZkNCRWlUdk1ibnBBRi9jbENzSFl3VnNCM2hnd0h0U1NCTXBtQ3FLUkYzY1hjeFN2T1hpZDR3QlV6UmFBVzA2TTZQYURTWXgvY29mRFlZQ3BXRDByTGgybGpRbE5hcWl0VE43Zjd4UVJucTJLbi95Y2ozRDVlMG9jb1BNWHR3L2ZuS0FQRVdXVCtwTG53VE9sVy9IU3VrWHN5VTM0NWErUXNjUEc2THR5cGY1bFV1U3p5UzBNWUxad3JnL1R3MmJ3cFprQWJZNEk5YUFPcUlIODYyOCtWQjlvdlFFc2Fjb0RrNlhxYXdKaGJYWGhUejdOSmF4c1ZGNkRNZWhvUWJiY0dhRHJpTlArbERPUXBxYjNVM1RodzRNQzYvWFVDRFh0NktqdTdJUXZWS2JJMHhzdlh4SkRWbDlxc0FFajM0aFlJYllUSmNSOUErZk96aVZ4bzJCVmdCWWFKZ2p6UW53T3dBb0xLOGlYS0hVZzBaNFJWajFYdC9UZ2VWbkU2K0l5TURhRFJrajh4RTJ2cVAvdEVCN25tYWFoK3JFTEg5b3JsVy9sa3dPR1VDQjRSYmFpNUx6Q0dUMkh5aFY0bE55US84ZnhXcUJJOFVUdXJ2b0FKQXZsQnhmckhYbnlnUWt0eHhQdWVCaVVOUUd0YUcxUmN3SEt4L3NpTGpCZ015V1BhNm10eXBYakdkVmFRUWRDOENFc3ZERnZrNHJERnpuLy9yZHY0bzBUZThtY0s5SUtMejdaYUo4YXZ5MGpUSTlHUUpienpmcDd0UDgrZ3FlYVFnTzBNQnIwUEU2ODlScldCK29XSGM0WGRTRnJ3SDQvYUxJTjdiSDg4Qjh6eFhLZHIvR1dGc3VWdDdWOFlWWHZDRlVTVUd2UFc5R24vNmxRZGpqMDNvdkt5eld5TWpIeTFDa0VVRDA4M0NxeTZjN3R4L2NzS2ptVHJnV2ZjUVFqZm4zRTBBMjF5cHEyZW1mclFJN3FpaTBzZFYyeWRLdE1lMmZmWW45ZW0xOTRPUlB0UWhmclpkYmVhMVo2dUI1K2JGV2trd3lxenZXcEZjSEh2N1dXSkdjZWZSWGF6MkNURWJSWEFxUTRIR3ZPd2QrSTRjbVpVSDNhOE5vdFNocG1PQ2xDUmoxYkg3dUJDQzVROXh1MENDWHZZOWk5QWROTVJSSjc2ZTRydEdGVGxhZmY2ckNoVUlUWEZHWDhDdXFOM2w2UHNDZDAxYWdhZlV1WlR3T3loZzJHM0pKRWhKK0gzcUhBM3pqRlJIZnp5eTV6WHVKZUFCMWtpODdKd1FXdVQwNnhtZDV1SkZsWGV4SzJVc0FITEVmZ0RoK2YxTldiYm5kUGJ1U2xSTm5ISmZ3MVQyMW1ZOHMxQ3ZWOFVHbXJ5NjgyWjN2YTM1WXVvWU5XakN2ZVZya205d2RqeWswOGk2UlZONDBrdnhpT2QxN3M2TXErc01UYTc5cUtWcXhobGVYNG1zUlUwK21zdmJkaHM0Rm5PRjJOQXE0VURPVVN1Y1NGa3hGRno0bjJyNHVwUlp6M0FFaVpoRHdvVm1XcHFZeUhMSVhIdjYwU0ZNSXAyMHQrVFBPMEF2aGVMT2JhU1VOWXpHZ3VJT0VjQ25nM0tFQTBxQlFLdTNhMjFLSTB5dG14bCtDMGZKY1FEK2hkY1JaR09NZHVja0R3RXFsV1FreUhTZTdLM2VXbEp5SFc4dGFYU0k2VlRRa1h0SzJaYWlEUTJiTkNJVUZpODJZQSthY1RLMWR0MitISm5mejdxeTVVNEczS21aQ3E1amlSV21XelJSa2FNeXVOOHM4dkVLd3hBSEQ0cFM5eWtYd1JuL01yaGNqY0g0dWtDR3lZTDVoNmc5ckdFSVRQWGErK05FT1NNSFpaS0RlbUNOMG84NFRZTUxKNlA0dVlCYXp5T2RibnRDeklPd3VITlJ3Q1JVeFpwS01FaEZRNWlYUFY3aXlKSEh5aDI4TjZ6Y2xvbzkrRzFYTlZQaVFaNHFkZzFnMDB1MldLalR2eUhNendlU2prZExKWEg1c2hTTGVTTFpDMHl6UWhEY0E4dlNwYTZVbUUzMVVZRnFXOFRSTnZoVVpncmNwSXUxQVVEdUdIQnRtZURsSDU0aVVVcWlWVG9ndkZPeGQ2bjBuTEV0VHY4b2ZtZVk0b3UraW5jbG81b09BclRETkl2UGQrWFRobUszVHg3RXNsSDFPdWZwVnk2YnVUZ1dmUkNOaHBrMEhwY002MXlvVng3T3p1MVN5OFpjMnBxL1hwUXRlUjkxUkJqY2JkeGdROHNVeVFQSWVhS1hETWpXeC9IcHR3UmFhanJIMlAvdG1JOFFjK0tTdEYrWnloV0xkbzdqdEJTNXNqMmg1aWJlYmVXeVQwemt0Um9ZdlF3SU43Z3ozTnFFdm5oeVIvdkErTVVOUXcwbUdqTVoya3J6UjZCT0RSSWNCQU9aMyszb1JKeDNoMU9Jc0tVQmlkS1hhUk9kUXNVR0szbEhjMHhvMXpjejJCcExvSURjaHpCSEhhdzMwaU45QVJlV2JoOVp4K1hQOWUzdGlRTDBoZmY2U20wN0hma3lOK3pDbEJ5R0VjbS9CS3VaRHhHZENJM2tsV3MzaG1Jb1NhTWpGUTlvZWhZNnNKYUZZZmpocnFWWVRqT1VDYkdTeDF2OC9XOGwvNU80RlBQYzRTKzI3cDNjK2pNPSJ9","retryCount":0,"uuid":"8d5a8857-064e-4cbd-b749-d17298335fcf"}]}

The ``"name":"completed"`` tells you the consent got successfully distributed over both nodes.
If you don't believe it, checkout the event on dahmer by performing the same request on address ``http://localhost:21323/events``.

There are quite a few steps to perform by the two nuts nodes before the consent has been recorded. Check out the :ref:`state machine with all the events here<Service space event specification>`.

These events get broadcasted to all parties, including your own application. Thats how you get notified about new patient consents.
Try to inspect the payload to see what you can expect. Hint: its base64 encoded.

That's it. You have just booted a open source distributed health infrasturcture on your computer and recorded your first consent.
If anything did not work out as described above, don't hesitate to :ref:`contact us<Contact>`.


Let users authenticate themselves to the Nuts network
-----------------------------------------------------

Stay tuned!

Let patients record their consent
---------------------------------

Bear with us!

Validate incoming api requests
------------------------------

Don't touch that dial!
