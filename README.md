# Immutant feature demo

An app showing trivial examples of all the Immutant libraries. Most
log something to stdout, but the `demo.web` examples are available at
`http://localhost:8080`.

The minimum supported Leiningen version is 2.4.0

You can run the app in several ways:

### run

The value of `:main` in `project.clj` is `demo.core`, which runs
`-main` for all of the demo namespaces.

    $ lein run

You can use the -m option to run specific namespaces, e.g.

    $ lein run -m demo.web

### repl

You can fire up a repl and invoke each namespace directly

    $ lein repl

Once at a prompt, try `(demo.web/-main)`

### jar

Create an uberjar and run it

    $ lein uberjar
    $ java -jar target/demo-0.2.0-SNAPSHOT-standalone.jar 

### wildfly

WildFly is installed by downloading and unpacking an archive. We'll
just stick it in the project dir for now.

    $ wget http://download.jboss.org/wildfly/8.1.0.Final/wildfly-8.1.0.Final.zip
    $ unzip wildfly-8.1.0.Final.zip
    $ lein immutant war -o wildfly-8.1.0.Final
    $ wildfly-8.1.0.Final/bin/standalone.sh -c standalone-full.xml

Note the web examples will be deployed with a context path of `/demo`
on WildFly so go to `http://localhost:8080/demo` to see the web
examples. You can override this by renaming the war file beneath
`wildfly-8.1.0.Final/standalone/deployments/` to `ROOT.war`.

#### clustering

We'll simulate a cluster by "installing" another WildFly instance:

    $ cp -R wildfly-8.1.0.Final wildfly-too
    $ rm -rf wildfly-too/standalone/data/

Because we already deployed the war file, it gets copied over, too.
And to avoid spurious errors, we remove the `standalone/data`
directory where WildFly keeps some runtime state.

Now we'll start our first instance:

    $ wildfly-8.1.0.Final/bin/standalone.sh -c standalone-full-ha.xml -Djboss.node.name=one -Djboss.messaging.cluster.password=demo

Note the following:

* We use the `standalone-full-ha.xml` file in which clustering is
  configured
* Since both of our peers will be on the same host, we need to
  give each node a unique name
* The cluster requires a password

In another shell, we fire up the second instance with similar options
plus a system property to avoid port conflicts, since we're on the
same host.

    $ wildfly-too/bin/standalone.sh -c standalone-full-ha.xml -Djboss.node.name=two -Djboss.messaging.cluster.password=demo -Djboss.socket.binding.port-offset=100

You can correlate the output from both peers to the code beneath
`src/demo` to observe HA singleton jobs, load-balanced messaging, and
distributed caching. And you can observe session replication by
reloading the following pages in your browser:

* <http://localhost:8080/demo/counter>
* <http://localhost:8180/demo/counter>

