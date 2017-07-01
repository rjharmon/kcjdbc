# Kafka Connect JDBC Connector

Original kafka-connect-jdbc is a [Kafka Connector](http://kafka.apache.org/documentation.html#connect)
for loading data to and from any JDBC-compatible database.

Documentation for the original connector can be found [here](http://docs.confluent.io/current/connect/connect-jdbc/docs/index.html).

# About this Fork

This fork corrects problems being able to build out-of-the-box (`mvn clean install`).  

It also adds support for LIMIT and WHERE clause as part of the configuration for jdbc 
Source workers.  These options are available for incremental-queries (not Custom Query 
or Bulk mode).

Note that currently,
all tables imported by a single Connect worker instance do share LIMIT and WHERE clause 
configurations.  To use different settings on different tables, you can make one or more 
additional Kafka Connect configuration files and feed them separately to the 
kafka-connect startup command-line (or to Kafka Connect's REST configuration URL). Each
one needs a distinct 'name' property.

Note that the 'batch.max.rows' property in the original repo doesn't perform LIMIT queries.
Its original effect was to allow logical workers for multiple tables to each make progress
in batches, in a cooperative-multitasking way.  This fork changes that behavior 

Tested and working with Confluent platform 3.1.1, but probably works fine with later versions 
too.  May need to adjust the Kafka version dependency in pom.xml to match.

### Building

To build this fork, just build with Maven, then copy the resulting jar to your Confluent 
distribution's jarfile location:

```
$ mvn clean install && \
  cp target/kafka-connect-jdbc-3.5.0-SNAPSHOT.jar confluent/share/java/kafka-connect-jdbc
```

Here, confluent/ is a symlink to a binary distribution of the Confluent platform.  You may need
to remove a pre-existing kafka-connect-jdbc-*.jar file from the target directory.

### Usage:

#### With Distributed Kafka Connect:

First, use standalone mode (see below) to do initial setup and verification of the configuration(s), 
then submit your configuration to the Kafka Connect REST proxy to add the workers to your production 
Connect cluster.

You may also need to make the custom JAR file available to the Connect cluster.

#### Standalone

Once you've updated your Confluent distribution with the newly-built JAR (see above), you 
can run it in standalone mode like this:

```
$ confluent/bin/connect-standalone \
  confluent/etc/schema-registry/connect-avro-standalone.properties \
  config/source-with-where-clause.properties
```

The first properties file configures Kafka Connect itself.  Confluent also offers other
packaged configurations for distributed-deployment scenarios and other data-encoding 
formats.

The second properties file on this example command-line creates a specific Connect worker task
for sourcing Kafka events from JDBC, and configures it with the example settings.  

Additional properties files listed on this command-line will configure additional, separate 
Kafka Connect workers, for example to connect with different databases.  Additional properties 
files can also be used to apply different batch-size or WHERE clauses to different sets of 
tables.  Note that each one needs a distinct "name" property.

`config/source-with-where-clause.properties` is a simple setup example demonstrating a 
configuration with the patched features and specific whitelisted table names.


# Development

To build a development version you'll need a recent version of Kafka. You can build
kafka-connect-jdbc with Maven using the standard lifecycle phases.


# Contribute

- Source Code: https://github.com/confluentinc/kafka-connect-jdbc
- Issue Tracker: https://github.com/confluentinc/kafka-connect-jdbc/issues


# License

The project is licensed under the Apache 2 license.
