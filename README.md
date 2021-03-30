Example Voting (Instavote) App
=========

Architecture
-----

![Architecture diagram](architecture.png)

* A Python webapp which lets you vote between two options
* A Redis queue which collects new votes
* A .NET worker which consumes votes and stores them in…
* A Postgres database backed by a Docker volume
* A Node.js webapp which shows the results of the voting in real time

Description
-----
![MicroServices](https://user-images.githubusercontent.com/6550812/158926965-b10d798b-7745-4a45-82d2-28d511c3c025.png)

Note
----

The voting application only accepts one vote per client. It does not register votes if a vote has already been submitted from a client.
