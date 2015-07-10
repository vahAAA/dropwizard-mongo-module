# dropwizard-mongo-module
A guice module for connecting a Dropwizard app to a Mongo DB cluster

This JAR is loosely based off of https://github.com/eeb/dropwizard-mongo, but re-written to support guice integration and to use Mongo URI connection strings.

# Usage
This module assumes you are including it in a Dropwizard 0.8.1+ application with dropwizard-guice wiring.

Include as a maven dependency:

```xml
<dependency>
    <groupId>com.washingtonpost.dropwizard</groupId>
    <artifactId>dropwizard-mongo</artifactId>
    <version>${version.wp.dropwizard.mongo}</version>
</dependency>
```

In your application's main initialize() method, add a new MongoBundle to your guice bundle, for example:

```java
guiceBundle = GuiceBundle.<MyConfiguration>newBuilder()
                .addModule(new MongoModule())
                .enableAutoConfig("com.washingtonpost")
                .setConfigClass(MyConfiguration.class)
                .build();
```

Add configuration to your application's configuration class, like:

```java
public class MyConfiguration extends Configuration {

    private MongoFactory mongoFactory = new MongoFactory();

    @JsonProperty("mongoDB")
    public MongoFactory getMongoFactory() {
        return this.mongoFactory;
    }

    @JsonProperty("mongoDB")
    public void setMongoFactory(MongoFactory mongoFactory) {
        this.mongoFactory = mongoFactory;
    }
```

Add reasonable configuration options to your application YML file:

```yaml
mongoDB:
    user: ${MONGO_USER}
    pass: ${MONGO_PASS}
    hosts: ${MONGO_HOSTS}
    dbName: ${MONGO_DBNAME}
    options: ${MONGO_OPTIONS}
```

And finally make sure your Guice wiring "provides" the MongoFactory as a bean (it's needed by name "mongoFactory" in the MongoModule that you wired into your Guice bundle):

```java
import com.google.inject.AbstractModule;
import com.google.inject.Provides;
import com.washingtonpost.mongo.dropwizard.MongoFactory;
import javax.inject.Named;

public class MyModule extends AbstractModule {

    @Override
    protected void configure() {
    }

    @Provides
    @Named("mongoFactory")
    public MongoFactory provideMongoFactory(MyConfiguration configuration) {
        return configuration.getMongoFactory();
    }
```

# Configuration
The Mongo configuration options map directly to the different parts of an allowed com.mongo.MongoClientURI, per the scheme defined here [http://docs.mongodb.org/manual/reference/connection-string/]

e.g. mongodb://[user:pass@][mongo_hosts,like:123,this:456][/[dbName][?options]]


```
mongoDB:
    user: // the username to connect to any & all hosts defined in hosts, optional, but required if pass is provided
    pass: // the password associated with the username.  Optional, but required if user is provided
    hosts: // at least one, but possibly many comma-separated, host:port pairs hosting Mongo DBs.
    dbName: // optional name of the DB Collection to initially connect to when requesting MongoModule.provideDB
    options: // optional connection configuration parameters for the Mongo driver
```
