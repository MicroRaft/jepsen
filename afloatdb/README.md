## How to run Jepsen tests for AfloatDB

### Install MicroRaft and AfloatDB projects locally
  
First of all, we need to install MicroRaft and AfloatDB projects locally 
because they are not available on the central Maven repository yet. 

```
git clone https://github.com/MicroRaft/MicroRaft.git && cd MicroRaft && mvn clean install -DskipTests=true

git clone https://github.com/MicroRaft/AfloatDB.git && cd AfloatDB && mvn clean install -DskipTests=true
```

The commands given above just clone MicroRaft and AfloatDB repositories, and
installs them to the local Maven repository. You can skip this step if you
already have them on your local.



### Install AfloatDB-Jepsen project locally

AfloatDB-Jepsen is a small utility project for testing AfloatDB with Jepsen 
easily. Basically, it is used for configuring AfloatDB server and clients for
the Docker environment used by Jepsen. There are no hacks or cheats for Jepsen
testing.

The following command installs it to the local Maven repository.

```
git clone https://github.com/MicroRaft/AfloatDB-Jepsen.git && cd AfloatDB-Jepsen && mvn clean install -DskipTests=true
```



### Install Docker and Docker Compose

Docker simplifies running Jepsen tests on both your local machine and a CI 
environment. Make sure you have docker and docker-compose on your environment.



### Set `$JEPSEN_ROOT`

Since we use the local JARs in our tests, we will mount the local Maven 
directory to Jepsen containers. Because of that, we need to set `$JEPSEN_ROOT` 
environment variable in order to make the Jepsen tool work correctly.

If we clone this repository to `/home/basri/repos/jepsen`, we need to set
`$JEPSEN_ROOT` as follows:

```
export JEPSEN_ROOT=/home/basri/repos/jepsen
```

Make sure `$JEPSEN_ROOT` is persisted.



### Start Jepsen 

`cd` into the `${JEPSEN_ROOT}/docker` directory of this repository and run `./up.sh --dev`. 
This command will pick `docker-compose.dev.yml` to start our containers. If you
check this file, you can see that we mount `$JEPSEN_ROOT` and `${HOME}/.m2` to
the `jepsen-control` container.



### Run the first test

Now we have our `jepsen-control` container up and running. We will use this 
container to run our tests. First, let's connect to it. Please run the 
following command in a new terminal without stopping the `up.sh` script.

```
docker exec -it jepsen-control bash
``` 

Now we are in the `jepsen-control` container. Let's `cd` into `afloatdb` here. 
If we run `ls -l` here, we see something like the following:

```
total 36
drwxr-xr-x  5 1000 1000 4096 May 31 13:49 .
drwxr-xr-x 35 1000 1000 4096 May 24 21:25 ..
-rw-r--r--  1 1000 1000   99 Apr 18 21:32 .gitignore
-rw-rw-r--  1 1000 1000 2850 May 31 13:49 README.md
-rw-r--r--  1 1000 1000  399 May 23 19:56 project.clj
drwxr-xr-x  4 1000 1000 4096 May 31 13:49 server
drwxr-xr-x  3 1000 1000 4096 Apr 18 21:32 src
drwxr-xr-x  3 1000 1000 4096 Apr 18 21:32 test
-rw-rw-r--  1 1000 1000  230 May 24 16:11 test.sh
```

Jepsen uses `lein` to manage and run test projects. We have `test.sh` to run
our tests without memorizing `lein` commands. Let's hit `bash test.sh 1 60` to
run the default test for 60 seconds. This command starts 5 containers from `n1`
to `n5`, each running a single AfloatDB server node. Then, it starts 5 
AfloatDB clients on the current container (`jepsen-control`), and starts the
actual test logic. If everything runs fine, we will see 
`Everything looks good! ヽ(‘ー`)ノ` at the end of the test run. 

All test files, such as AfloatDB server logs, client operation history, are put 
into the `store` directory inside `afloatdb`. We can use these files for 
analysis of test results.


### Make changes in MicroRaft, AfloatDB, AfloatDB-Jepsen projects

If we make any changes in one of MicroRaft, AfloatDB, AfloatDB-Jepsen 
projects, we need to make sure that their new versions are available in the 
local `.m2` directory. `mvn install` would to the work. 



### Make changes in the Jepsen tests

`${JEPSEN_ROOT}/afloatdb` contains the Jepsen testing setup for our AfloatDB
project. It is based on `lein`. It contains the Clojure source code to build 
and start AftloatDB servers and clients, and run the test scenarios. 
`${JEPSEN_ROOT}/afloatdb/server` directory contains the lein project for 
configuring AfloatDB server nodes. When we make any change in these projects, 
we can just delete `${JEPSEN_ROOT}/afloatdb/target` and 
`${JEPSEN_ROOT}/afloatdb/server/target` directories before hitting 
`${JEPSEN_ROOT}/docker/up.sh`.

