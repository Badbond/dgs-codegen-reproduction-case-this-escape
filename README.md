# DGS Codegen issue violating this-escape JVM linter

A reproduction case for an issue found with running Netflix DGS Codegen with
JDK21, violating the `this-escape` JVM linter.

Using JDK21, run `mvn clean compile` and observe the following warning (raised
to an error by our Maven config):

```
[INFO] --- compiler:3.11.0:compile (default-compile) @ dgs-codegen-reproduction-case-this-escape ---
[INFO] Changes detected - recompiling the module! :source
[INFO] Compiling 4 source files with javac [debug release 17] to target/classes
[INFO] -------------------------------------------------------------
[WARNING] COMPILATION WARNING :
[INFO] -------------------------------------------------------------
[WARNING] /home/badbond/src/github.com/Badbond/dgs-codegen-reproduction-case-this-escape/target/generated-sources/me/soels/api/client/CreateThingGraphQLQuery.java:[16,17] possible 'this' escape before subclass is fully initialized
```

The potential violation can be seen in `me.soels.api.client.
CreateThingGraphQLQuery`'s constructor invoking `getInput()`:

```java
public class CreateThingGraphQLQuery extends GraphQLQuery {
    public CreateThingGraphQLQuery(String id, String queryName, Set<String> fieldsSet) {
        super("mutation", queryName);
        if (id != null || fieldsSet.contains("id")) {
            getInput().put("id", id);
        }
    }
    // ...
}
```

See also [JDK-8299995](https://bugs.openjdk.org/browse/JDK-8299995),
