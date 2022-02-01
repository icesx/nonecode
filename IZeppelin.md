Zeppelin
======

# Install



- Requirements
  - [Downloading Binary Package](https://zeppelin.apache.org/docs/latest/quickstart/install.html#downloading-binary-package)
  - [Building Zeppelin from source](https://zeppelin.apache.org/docs/latest/quickstart/install.html#building-zeppelin-from-source)
- [Starting Apache Zeppelin](https://zeppelin.apache.org/docs/latest/quickstart/install.html#starting-apache-zeppelin)
- [Using the official docker image](https://zeppelin.apache.org/docs/latest/quickstart/install.html#using-the-official-docker-image)
- [Start Apache Zeppelin with a service manager](https://zeppelin.apache.org/docs/latest/quickstart/install.html#start-apache-zeppelin-with-a-service-manager)
- [Next Steps](https://zeppelin.apache.org/docs/latest/quickstart/install.html#next-steps)

Welcome to Apache Zeppelin! On this page are instructions to help you get started.

## Requirements



Apache Zeppelin officially supports and is tested on the following environments:

|         Name          |               Value               |
| :-------------------: | :-------------------------------: |
| OpenJDK or Oracle JDK |   1.8 (151+) (set `JAVA_HOME`)    |
|          OS           | Mac OSX Ubuntu 18.04 Ubuntu 20.04 |

### Downloading Binary Package

Two binary packages are available on the [download page](http://zeppelin.apache.org/download.html). Only difference between these two binaries is whether all the interpreters are included in the package file.

- **all interpreter package**: unpack it in a directory of your choice and you're ready to go.
- **net-install interpreter package**: only spark, python, markdown and shell interpreter included. Unpack and follow [install additional interpreters](https://zeppelin.apache.org/docs/latest/usage/interpreter/installation.html) to install other interpreters. If you're unsure, just run `./bin/install-interpreter.sh --all` and install all interpreters.

### Building Zeppelin from source

Follow the instructions [How to Build](https://zeppelin.apache.org/docs/latest/setup/basics/how_to_build.html), If you want to build from source instead of using binary package.

## Starting Apache Zeppelin



#### Starting Apache Zeppelin from the Command Line

On all unix like platforms:

```text
bin/zeppelin-daemon.sh start
```

After Zeppelin has started successfully, go to [http://localhost:8080](http://localhost:8080/) with your web browser.

By default Zeppelin is listening at `127.0.0.1:8080`, so you can't access it when it is deployed in another remote machine. To access a remote Zeppelin, you need to change `zeppelin.server.addr` to `0.0.0.0` in `conf/zeppelin-site.xml`.

#### Stopping Zeppelin

```text
bin/zeppelin-daemon.sh stop
```

## Using the official docker image



Make sure that [docker](https://www.docker.com/community-edition) is installed in your local machine.

Use this command to launch Apache Zeppelin in a container.

```bash
docker run -p 8080:8080 --rm --name zeppelin apache/zeppelin:0.9.0
```

To persist `logs` and `notebook` directories, use the [volume](https://docs.docker.com/engine/reference/commandline/run/#mount-volume--v-read-only) option for docker container.

```bash
docker run -p 8080:8080 --rm -v $PWD/logs:/logs -v $PWD/notebook:/notebook \
           -e ZEPPELIN_LOG_DIR='/logs' -e ZEPPELIN_NOTEBOOK_DIR='/notebook' \
           --name zeppelin apache/zeppelin:0.9.0
```

If you have trouble accessing `localhost:8080` in the browser, Please clear browser cache.

## Start Apache Zeppelin with a service manager



> **Note :** The below description was written based on Ubuntu.

Apache Zeppelin can be auto-started as a service with an init script, using a service manager like **upstart**.

This is an example upstart script saved as `/etc/init/zeppelin.conf` This allows the service to be managed with commands such as

```text
sudo service zeppelin start  
sudo service zeppelin stop  
sudo service zeppelin restart
```

Other service managers could use a similar approach with the `upstart` argument passed to the `zeppelin-daemon.sh` script.

```text
bin/zeppelin-daemon.sh upstart
```

**zeppelin.conf**

```aconf
description "zeppelin"

start on (local-filesystems and net-device-up IFACE!=lo)
stop on shutdown

# Respawn the process on unexpected termination
respawn

# respawn the job up to 7 times within a 5 second period.
# If the job exceeds these values, it will be stopped and marked as failed.
respawn limit 7 5

# zeppelin was installed in /usr/share/zeppelin in this example
chdir /usr/share/zeppelin
exec bin/zeppelin-daemon.sh upstart
```

## Next Steps



Congratulations, you have successfully installed Apache Zeppelin! Here are a few steps you might find useful:

#### New to Apache Zeppelin...

- For an in-depth overview, head to [Explore Zeppelin UI](https://zeppelin.apache.org/docs/latest/quickstart/explore_ui.html).
- And then, try run [Tutorial Notebook](http://localhost:8080/#/notebook/2A94M5J1Z) in your Zeppelin.
- And see how to change [configurations](https://zeppelin.apache.org/docs/latest/setup/operation/configuration.html) like port number, etc.

#### Spark, Python, SQL, and more

- [Spark support in Zeppelin](https://zeppelin.apache.org/docs/latest/quickstart/spark_with_zeppelin.html), to know more about deep integration with [Apache Spark](http://spark.apache.org/).
- [SQL support in Zeppelin](https://zeppelin.apache.org/docs/latest/quickstart/sql_with_zeppelin.html) for SQL support
- [Python support in Zeppelin](https://zeppelin.apache.org/docs/latest/quickstart/python_with_zeppelin.html), for Matplotlib, Pandas, Conda/Docker integration.
- [All Available Interpreters](https://zeppelin.apache.org/docs/latest/#available-interpreters)

#### Multi-user support ...

- Check [Multi-user support](https://zeppelin.apache.org/docs/latest/setup/basics/multi_user_support.html)
