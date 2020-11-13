# HOW TO RELEASE TO KYLIGENCE NEXUS MAVEN REPO

This guide introduces how to release apache curator kylin branch  to Kyligence nexus maven repository.

## I. Configure the setting in Maven

Firstly, add the nexus server information inside `<servers></servers>` tag into maven setting profile, which locates at `~/.m2/settings.xml` by default.

Inside the `<servers>` tag, you need to configure the `id`, `username` and `password` of the nexus server.

> Note: You can simply put the pre-configured `release/settings.xml` into your `~/.m2/`

## II. Configure the POM file 

In the pom file of the project. We need to configure the `<distributionManagement>` as below

```xml
    <distributionManagement>
        <snapshotRepository>
            <id>user-snapshots</id>
            <name>User Project SNAPSHOTS</name>
            <url>http://repository.kyligence.io:8081/repository/maven-snapshots/</url>
        </snapshotRepository>
        <repository>
            <id>user-releases</id>
            <name>User Project Release</name>
            <url>http://repository.kyligence.io:8081/repository/maven-releases/</url>
        </repository>
    </distributionManagement>
```

As apache curator uses `maven-release-plugin` to release the binary, it's also need to configure the `scm` in pom.xml file.

```xml
    <scm>
        <url>https://github.com/Kyligence/curator.git</url>
        <connection>scm:git:https://github.com/Kyligence/curator.git</connection>
        <developerConnection>scm:git:https://github.com/Kyligence/curator.git</developerConnection>
        <tag>apache-curator-2.12.0</tag>
    </scm>
```
Please replace the `url`, `connection` and `developerConnection` inside `<scm>` tag.
Note that the `<tag>` tag inside `<scm>` is handled by `maven-release-plugin`.

So far the pom.xml has been configured with Kyligence nexus server.

Then remember to run `mvn clean install -DskipTests` to compile the code

## III. Update project-version to Snapshot

This step is **OPTIONAL**.

When using the `maven-release-plugin` to release the binary, it requires the current version of project must be a SNAPSHOT version. So if the current version is not SNAPSHOT version, we need to reset the version to SNAPSHOT.

You can edit the pom.xml manually or use the maven command to set it automatically.

For example, if set the current project version to `2.12.0-kylin-r1-SNAPSHOT`
```shell
mvn versions:set -DnewVersion=2.12.0-kylin-r1-SNAPSHOT 
git clean -f -d
git commit -m "Commit message" 
``` 
Maven will change the project version and also the versions of all sub-modules automatically.

Note that the version must follow the pattern `\d+\.\d+\.\d+.*-SNAPSHOT`

## IV. Prepare the release and Perform the release

After finished the above steps, now it's ready to prepare and perform the release.

Use maven command as below to prepare the release:

```shell
mvn release:clean release:prepare
```
During the release prepare process, it will promotes to tag the release version, an example as below.

```
[INFO] Checking dependencies and plugins for snapshots ...
What is the release version for "Apache Curator"? (org.apache.curator:apache-curator) 2.12.0-kylin-r1: : 
What is SCM release tag or label for "Apache Curator"? (org.apache.curator:apache-curator) apache-curator-2.12.0-kylin-r1-SNAPSHOT: :                     
What is the new development version for "Apache Curator"? (org.apache.curator:apache-curator) 2.12.1-kylin-r1-SNAPSHOT: : 2.12.0-kylin-r2-SNAPSHOT
```

You can change the default value as you want and press enter, then the release prepare will be finished.

Now you will see two more commits which committed by `maven-release-plugin` in your local git commit history. Next step is to perform release.

Use maven command as below to perform release:

```shell
mvn release:perform
```

Then the release just prepared will be performed and uploaded to Kyligence nexus server.

## V. Check

Check the released binary in nexus server: [nexus](http://repository.kyligence.io:8081/#browse/browse/assets)

And also don't forget to commit your local git repository to the remote git repository