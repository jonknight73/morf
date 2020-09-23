#HyperSQL Support

How to add support for a new DB.

``HyperSQL`` is closest to `H2` from the current implementations. This suggests a simple plan:

1. Copy the whole `morf-h2` branch to `morf-hsqldb` 
1. Fix all issues. 

## The Icky Details

+ First the low hanging fruit. We use the IDE to refactor the Class names -- changing `H2` to `HSQLDB` in classes.
+ Also in the IDE raname `h2` packages to `hsqldb`. Rename the packages in the Java Classes to match.
+ Next up delete `morf-hsqldb/morf-h2.iml`.
+ Open `morf-hsqldb/pom.xml`. Update `name` and `artifactId`.

```xml
  <name>Morf - HSqlDB</name>
  <description>Morf is a library for cross-platform evolutionary relational database mechanics, database access and database imaging/cloning.</description>
  <url>https://github.com/alfasoftware/morf</url>

  <artifactId>morf-hsqldb</artifactId>
```

+ Open the `morf-parent` `pom.xml` and add an entry for `morf-hsqldb`.

```xml
    <module>morf-h2</module>
    <module>morf-hsqldb</module>
    <module>morf-mysql</module>

...

      <dependency>
        <groupId>org.alfasoftware</groupId>
        <artifactId>morf-hsqldb</artifactId>
        <version>${project.version}</version>
      </dependency>
```


Change 
```xml
    <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
    </dependency>
```
to
```xml
    <dependency>
        <groupId>org.hsqldb</groupId>
        <artifactId>hsqldb</artifactId>
        <version>2.5.1</version>
    </dependency>
```

In `HSqlDB.java` set the Identifier:
```java
  public static final String IDENTIFIER = "HSqlDB";
```

Edit `morf-hsqldb/src/main/resources/META-INF/services/org.alfasoftware.morf.jdbc.DatabaseType` and set its contents to:
```text
org.alfasoftware.morf.jdbc.hsqldb.HSqlDB
```

+ Now back to the JAVA Classes and search and replace. 
  + Replace `H2Dialect` with `HSqlDBDialect`
  + Replace `H2.IDENTIFIER` with `HSqlDB.IDENTIFIER`
  + Replace `H2MetaDataProvider` with `HSqlDBMetaDataProvider`  
  
  
## JDBC  

The JDBC driver is at `org.hsqldb.jdbc.JDBCDriver` so set it in the constructor:
```java
  /**
   * Constructor.
   */
  public HSqlDB() {
    super("org.hsqldb.jdbc.JDBCDriver", IDENTIFIER);
  }
```

It feels like a good time to look up the JDBC parameter for HSQLDB. http://www.hsqldb.org/doc/2.0/guide/dbproperties-chapt.html

The first pass will just support in memory execution.  Which means the JDBC string will start `jdbc:hsqldb:mem:`. The H2 `tcp` maps to `hsql`. 


At this point we remember that we love TDD and embark on some unit tests. We start with `TestHSqlDBDatabaseType`. They all pass. Woo-hoo we have the major wiring in place. Then onto the trickier `TestHSqlDBDialect`.

And is an unexpected twist all 240 tests passed. 