# gremlin-orm-neo4j


_A work in progress. Use at your own discretion._

An unofficial set of lightweight [Docker][docker], and [Docker Compose][docker-compose] files for working with [Apache TinkerPop™][tinkerpop] Gremlin Console, Gremlin Server, and [Neo4j][neo4j].

- `org.apache.tinkerpop neo4j-gremlin` is installed out-of-the-box for both the Gremlin Console, and the Server image (currently version `3.4.2`)
- `tinkerpop.neo4j`, and `tinkerpop.gephi` plugins are also enabled for the Gremlin Console image by default (i.e. no `:plugin use` needed)


## Disclaimer

**Binding to 0.0.0.0:**

By default, most Gremlin Server configuration files will set the host binding to `localhost`. The Gremlin Server image in this repository uses a custom configuration file ([`gremlin-server-custom.yml`](gremlin-server-neo4j/conf/gremlin-server-custom.yml)) in order to bind to `0.0.0.0` so that Docker containers may communicate with instantiations of it. i.e. any 'host' can connect to it.

**Embedded mode:**

Both Gremlin, and Neo4j run in embedded mode. That is, neither can be operating on the same database at the same time, and only one JVM process may access it at any given time.

To perform concurrent reads on the same database, Gremlin, and Neo4j must be configured for high availability support. The Docker images in this repository are not currently configured for the aforementioned purpose.

You can ignore this if both services are operating on different databases.

## Known Issues

- [ ] Currently databases accessed/created with Gremlin Server. Once opened with Neo4j browser these files become impossible to
      mount again with Gremlin Server. Current work around is to make a copy of the database before accessing with Neo4j.
      Then copy these files back to `./data/` (relative) after browsing. This is something we are looking at and hope to have fixed soon. 


## Install

You can skip this step if you are working with Docker Compose.

    docker pull gremlinorm/gremlin-console-neo4j
    docker pull gremlinorm/gremlin-server-neo4j


### Docker Compose

As mentioned in the disclaimer, you should not run multiple instances of Gremlin or Neo4j if they are operating on the same database. Otherwise, you will end up with errors related to _'lock' / 'locking'_.

The default [`docker-compose.yml`](docker-compose.yml) configuration for services below will mount the `./data/` (relative) directory in this repository to `/data/` (absolute) directory in the instantiated Docker container. i.e. all the services can / will access, and persist data into the same directory.

**Gremlin Console only:**

    docker-compose run --rm gremlin-console

You can also launch the console in interactive mode with a custom Groovy script by setting the `command` configuration in `docker-compose.yml`:

    command: "/bin/bash /gremlin-console/bin/gremlin.sh -i init.groovy"

Ensure that `init.groovy` is available in the container. The easiest way to do so is to mount the file from your host machine to the container (look for `volumes` in `docker-compose.yml`).

The above is useful if you would like to bootstrap / prepare your console environment, and avoid typing the same _commands_ every time you instantiate the console.

See [Docker Compose docs][docker-compose-docs] for more information.

**Gremlin Console with Server:**

    docker-compose run --rm gremlin-console-with-server

**Gremlin Server only:**

    docker-compose run --rm --service-ports gremlin-server

**Neo4j only:** This makes a duplicate copy of your database. See `Known Issues`

    docker-compose run --rm --service-ports neo4j-server


## Useful Scripts For Starting Services

From the gremlin-orm-neo4j root directory.

```./bin/server``` Will start the Gremlin server running ready for connection with Gremlin-Orm

```./bin/neo4j``` Will start the Neo4j server running on localhost:7474. Before doing this it creates a copy of your database - It 
                  is this copy you are browsing (No changes will take effect). See 'Known Issues'

```./bin/testserver``` Will start the Gremlin server running on a test database ready for connection with Gremlin-Orm Test suite


## Development

`grapeConfig.xml` is needed by the Gremlin Console, and Server to install the `neo4j-gremlin` plugin.


## Useful Console Commands

**Connect to remote Docker Compose service:**

The following command assumes that the console container, and the server container are linked. It should not be a problem if launched with the `gremlin-console-with-server` service.

```groovy
:remote connect tinkerpop.server conf/remote-docker-compose.yml
:remote console
```

**Connect to host machine Gephi:**

The following command assumes that you have [Gephi][gephi] installed on your host machine along with the Graph Streaming plugin (see [TinkerPop™ Gephi docs][gephi-docs] for more information).

```groovy
:remote connect tinkerpop.gephi <workspace> <host> <port>
```

- Replace `<workspace>` with your current Gephi workspace (normally it is `workspace1`)
- Replace `<host>` with the internal IP address of your host machine (you can find this using `ifconfig` on Unix)
- Replace `<port>` with the port of the Graph Streaming plugin (this can be omitted as the default `8080` is normally appropriate)

**Load Neo4j graph:**

```groovy
graph = Neo4jGraph.open("/data/databases/graph.db/")
g = graph.traversal()
g.V().count()
```

- To persist the database, ensure that the `/data/` directory is mounted between the host, and the container (this is handled by Docker Compose through `volumes`)
- To use an existing Neo4j database, set the path in `Neo4jGraph.open(<path>)` to the directory containing `neostore` (e.g. `/data/databases/graph.db/`)
- To create, and use a new Neo4j database, set the path to any new / unused directory (e.g. `/data/gremlin-neo4j/`)


[docker-compose-docs]: https://docs.docker.com/compose/compose-file/#/command
[docker-compose]: https://docs.docker.com/compose/
[docker-hub]: https://hub.docker.com/
[docker]: https://www.docker.com/
[gephi-docs]: http://tinkerpop.apache.org/docs/current/reference/#gephi-plugin
[gephi]: https://gephi.org/
[neo4j]: https://neo4j.com/
[tinkerpop]: https://tinkerpop.apache.org/
[travis-ci]: https://travis-ci.org/
