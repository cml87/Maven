# Maven


## Maven profiles
Profiles are specified using a subset of the elements available in the POM itself (plus one extra section), and are triggered in any of a variety of ways. They modify the POM at build time, and are meant to be used in complementary sets to give equivalent-but-different parameters for a set of target environments (providing, for example, the path of the appserver root in the development, testing, and production environments). As such, profiles can easily lead to differing build results from different members of your team. However, used properly, profiles can be used while still preserving project portability. 

Each profile will have an id which should hint to their intended use. One good way to do this is to use the common system property trigger as part of the name for the profile. This might result in names like `env-dev`, `env-test`, and `env-prod` for profiles that are triggered by the system property `env` (see below):
 ```shell
 $ mvn -Denv=test <phase>
 ```

There are four types of maven profile
- Per Project: defined in the POM itself (`pom.xml`).
- Per User: defined in the Maven-settings (`%USER_HOME%/.m2/settings.xml`).
- Global: defined in the global Maven-settings (`${maven.home}/conf/settings.xml`).
- Profile descriptor: a descriptor located in project basedir (`profiles.xml`) (no longer supported in Maven 3.0 and above; see Maven 3 compatibility notes.)

## Profile activation
A profile can be activated in several ways:
### From the command line
Profiles can be explicitly specified using the `-P` command line flag.

This flag is followed by a comma-delimited list of profile IDs to use. The profile(s) specified in the option are activated in addition to any profiles which are activated by their activation configuration in the pom or settings.xml (`<activation></activation>`), or the <activeProfiles> section in the `settings.xml`. Thus we may have several profiles activated at once!

From Maven 4 onward, Maven will refuse to activate or deactivate a profile that cannot be resolved. To prevent this, prefix the profile identifier with an ?, marking it as optional:
```shell
$ mvn groupId:artifactId:goal -P profile-1,profile-2,?profile-3
```
### Through Maven settings
Profiles can be activated in the Maven settings, settings.xml file, via the `<activeProfiles>` section. This section takes a list of `<activeProfile>` elements, each containing a profile-id inside.
```xml
<settings>
  ...
  <activeProfiles>
    <activeProfile>profile-1</activeProfile>
  </activeProfiles>
  ...
</settings>
```
Profiles listed in the <activeProfiles> tag would be activated by default every time a project uses it.

Profiles can also be active by default using a configuration like the following in the pom:
```xml
<profiles>
  <profile>
    <id>profile-1</id>
    <activation>
      <activeByDefault>true</activeByDefault>
    </activation>
    ...
  </profile>
</profiles>
```
This profile will automatically be active for all builds unless another profile in the same POM is activated via `-P` flag in the command line, or through some activation configuration as we'll see below. In fact, all profiles that are active by default in the pom (not in the settings.xml) are automatically deactivated when a profile in the POM is activated on the command line with `-P` or through its activation config. Active profiles by default in the settings.xml instead stay active in this case.

Thus, we have two options to activate one or more profiles purposely: either through the `-P` command line flag, or through some activation configuration in the profile itself, which may be in the pom or the settings.xml.

### Based on environment variables and activation setting
The profile below will be activated when the system property "debug" is specified with any value (via `-D` flag):
```xml
<profiles>
  <profile>
    <activation>
      <property>
        <name>debug</name>
      </property>
    </activation>
    ...
  </profile>
</profiles>
```
The following profile will be activated when the system property "debug" is not defined at all:
```xml
<profiles>
  <profile>
    <activation>
      <property>
        <name>!debug</name>
      </property>
    </activation>
    ...
  </profile>
</profiles>
```
The following profile will be activated when the system property "debug" is not defined, or is defined with a value which is not "true" (property not defined or false).
```xml
<profiles>
  <profile>
    <activation>
      <property>
        <name>debug</name>
        <value>!true</value>
      </property>
    </activation>
    ...
  </profile>
</profiles>
```
More precisely, the profile above will activate when we do in the command line:
```shell
$ mvn groupId:artifactId:goal
$ mvn groupId:artifactId:goal -Ddebug=false
```
The next example will trigger the profile when the system property "environment" is specified with the value "test":
```xml
<profiles>
  <profile>
    <activation>
      <property>
        <name>environment</name>
        <value>test</value>
      </property>
    </activation>
    ...
  </profile>
</profiles>
```
To activate this we would type this on the command line:
```shell
$ mvn groupId:artifactId:goal -Denvironment=test
```
As of Maven 3.0, profiles in the POM can also be activated based on properties from active profiles from the `settings.xml`. Moreover, environment variables like `FOO` are available as properties of the form `env.FOO`. On Windows, environment variable names are normalized to all upper-case.

### Present or missing files
This example will trigger the profile when the generated file `target/generated-sources/axistools/wsdl2java/org/apache/maven` is missing:
```xml
<profiles>
  <profile>
    <activation>
      <file>
        <missing>target/generated-sources/axistools/wsdl2java/org/apache/maven</missing>
      </file>
    </activation>
    ...
  </profile>
</profiles>
```
As of Maven 2.0.9, the tags `<exists>` and `<missing>` could be interpolated. Supported variables are system properties like `${user.home}` and environment variables like `${env.HOME}`. However, properties and values defined in the POM itself are not available for interpolation here, e.g. the above example activator cannot use `${project.build.directory}` but needs to hard-code the path target. In other words, we can define properties in a `<property></property>` block of a pom profile and use them for interpolation outside the profile, in the same pom. But properties defined outside the profile cannot be used for interpolation inside profiles, in a given pom. Inside the pom profile, however, environment variables or settings.xml properties can be interpolated, as we saw above, I think.


### OS settings and activation setting
Profiles can be automatically triggered based on the detected state of the build environment. These triggers are specified via an <activation> section in the profile itself. Currently, this detection is limited to prefix-matching of the JDK version, the presence of a system property or the value of a system property. Here are some examples:
The following configuration will trigger the profile when the JDK's version starts with "1.4" (eg. "1.4.0_08", "1.4.2_07", "1.4"):
```xml
<profiles>
  <profile>
    <activation>
      <jdk>1.4</jdk>
    </activation>
    ...
  </profile>
</profiles>
```
Ranges can also be used as of Maven 2.1 (refer to the Enforcer Version Range Syntax for more information). The following honours versions 1.3, 1.4 and 1.5:
```xml
<profiles>
  <profile>
    <activation>
      <jdk>[1.3,1.6)</jdk>
    </activation>
    ...
  </profile>
</profiles>
```
An upper bound such as `,1.5]` is likely not to include most releases of 1.5, since they will have an additional "patch" release such as _05 that is not taken into consideration in the above range.

This next one will activate based on OS settings. See the Maven Enforcer Plugin for more details about OS values.
```xml
<profiles>
  <profile>
    <activation>
      <os>
        <name>Windows XP</name>
        <family>Windows</family>
        <arch>x86</arch>
        <version>5.1.2600</version>
      </os>
    </activation>
    ...
  </profile>
</profiles>
```

## Deactivating a profile
Starting with Maven 2.0.10, one or more profiles can be deactivated using the command line by prefixing their identifier with either the character '!' or '-' as shown below:
```shell
$ mvn groupId:artifactId:goal -P !profile-1,!profile-2,!?profile-3
$ mvn groupId:artifactId:goal -P -profile-1,-profile-2,-?profile-3
```
This can be used to deactivate profiles marked as activeByDefault or profiles that would otherwise be activated through their activation config in the pom or settings.xml` I think.

## Pom configuration in profiles
Depending on where you choose to configure your profile, you will have access to varying POM configuration options.

### Profiles in external files
Profiles specified in external files (i.e in `settings.xml` or `profiles.xml`) are not portable in the strictest sense. Anything that seems to stand a high chance of changing the result of the build is restricted to the inline profiles in the POM. Thus in profiles in the mentioned files will only be able to modify the <repositories> and <pluginRepositories> sections, plus an extra <properties> section.

The <properties> section allows you to specify free-form key-value pairs which will be included in the interpolation process for the POM, as we say above. This allows us to specify a plugin configuration in the form of `${profile.provided.path}` (is this the value of the property in the active profile ?)

### Profiles in POMs
Profiles specified in the POM can modify the following POM elements:
```xml
<repositories>
<pluginRepositories>
<dependencies>
<plugins>
<properties> (not actually available in the main POM, but used behind the scenes ?)
<modules>
<reports>
<reporting>
<dependencyManagement>
<distributionManagement>
```
as well as a subset of the `<build>` element:
```xml
<defaultGoal>
<resources>
<testResources>
<directory>
<finalName>
<filters>
<pluginManagement>
<plugins>
```
Thus profiles specified inside the POM have many more options. The trade-off, of course, is that we'll be able to modify only that project and its sub-modules by editing the profile in the pom. Since these profiles are specified inline, and therefore have a better chance of preserving portability, it's reasonable to say we can add more information to them without the risk of that information being unavailable to other users.

Profiles can include a `<property>` block that apply one or other properties values in the pom where they are, depending on the active, or activated one:
```xml
<project>
  ...
  <build>
    <plugins>
      <plugin>
        <groupId>org.myco.plugins</groupId>
        <artifactId>spiffy-integrationTest-plugin</artifactId>
        <version>1.0</version>
        <configuration>
          <appserverHome>${appserver.home}</appserverHome>
        </configuration>
      </plugin>
      ...
    </plugins>
  </build>
  ...
</project>

<project>
  ...
  <profiles>
    <profile>
      <id>appserverConfig-dev</id>
      <activation>
        <property>
          <name>env</name>
          <value>dev</value>
        </property>
      </activation>
      <properties>
        <appserver.home>/path/to/dev/appserver</appserver.home>
      </properties>
    </profile>
 
    <profile>
      <id>appserverConfig-dev-2</id>
      <activation>
        <property>
          <name>env</name>
          <value>dev-2</value>
        </property>
      </activation>
      <properties>
        <appserver.home>/path/to/another/dev/appserver2</appserver.home>
      </properties>
    </profile>
  </profiles>
  ..
</project>
```
Here executing:
```shell
$ mvn -Denv=dev-2 integration-test
```
will result in a successful build, applying the properties given by profile named `appserverConfig-dev-2`. And when we execute:
```shell
$ mvn -Denv=dev integration-test
```
it will result in a successful build applying the properties given by the profile named `appserverConfig-dev`. However, executing:
```shell
$ mvn -Denv=production integration-test
```
will not do a successful build. The reason is no profile will get activated when variable `dev` has value `production` leaving the needed property to configure a plugin `appserver.home` undefined.


## POM elements outside `<profiles>`
Some POM elements cannot be modified outside profiles bla bla bla. 
External files such as `settings.xml` and `profiles.xml` also does not support elements outside the POM-profiles bla bla bla (see the documentation).

One of the goals in Maven 2 is to consolidate all the information needed to run a build into a single file, or file hierarchy, which is the POM. Thus it is recommended to define profiles in the POM instead of in settings.xml, which normally is not distributed or versioned.

## Profile Order
All profile elements in a POM from active profiles overwrite the global elements with the same name of the POM which are not inside any profile, or extend those in case of collections. In case multiple profiles are active in the same POM or external file, the ones which are <u>defined</u> later take precedence over the ones defined earlier (independent of their profile id and activation order).
The example below shows two profiles activated by default, that add more values to a collection property `<repository>` which also gets a value globally, outside any profile:
```xml
<project>
  ...
  <repositories>
    <repository>
      <id>global-repo</id>
      ...
    </repository>
  </repositories>
  ...
  <profiles>
    <profile>
      <id>profile-1</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <repositories>
        <repository>
          <id>profile-1-repo</id>
          ...
        </repository>
      </repositories>
    </profile>
    <profile>
      <id>profile-2</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <repositories>
        <repository>
          <id>profile-2-repo</id>
          ...
        </repository>
      </repositories>
    </profile>
    ...
  </profiles>
  ...
</project>
```
This leads to the repository list: `profile-2-repo, profile-1-repo, global-repo`.

## Project portability and profiles
Attention must be paid to properties defined in not pom, but settings.xml profiles. Such properties can be resolved in a project's pom, only if the user compiling that maven project is using the specific settings.xml file with that profile and property definition. Since often the settings.xml file is user specific and is not distributed, in contrast to the pom file, this approach is much likely to break project portability:
```xml
<!-- pom file -->
<project>
  ...
  <build>
    <plugins>
      <plugin>
        <groupId>org.myco.plugins</groupId>
        <artifactId>spiffy-integrationTest-plugin</artifactId>
        <version>1.0</version>
        <configuration>
          <appserverHome>${appserver.home}</appserverHome>
        </configuration>
      </plugin>
      ...
    </plugins>
  </build>
  ...
</project>
```
```xml
<!-- setting.xml file -->
<settings>
  ...
  <profiles>
    <profile>
      <id>appserverConfig</id>
      <properties>
        <appserver.home>/path/to/appserver</appserver.home>
      </properties>
    </profile>
  </profiles>
 
  <activeProfiles>
    <activeProfile>appserverConfig</activeProfile>
  </activeProfiles>
  ...
</settings>
```
To solve this issue we should inline the profile in the project's pom.
In this case, since the culprit property is used to define the version of a plugin, the issue could also be solved by using a `<pluginManagement>` section in a distributed parent pom.

Another common issue when defining profiles is not defining all the profiles that are "natural" for the project. For example, dev, test, prod and qa. See the example above where we did `mvn -Denv=production integration-test`.

## Determining the active profile
Determining active profiles helps the user to know what particular profiles has been executed during a build. We can use the Maven Help Plugin to tell what profiles are in effect during a build. For example if we have defined in our pom a profile that gets activated when system property `env` has value `dev` and we do
```shell
$ mvn help:active-profiles -Denv=dev
```
we'll get:
```shell
The following profiles are active:

 - appserverConfig-dev (source: pom)
```
If instead we have a profile active by default in our settings.xml and we run the previous command without passing any `-D` option, we'll get something like:
```shell
The following profiles are active:

 - appserverConfig (source: settings.xml)
```

If we have both things, ie. an active profile from the settings.xml and an activated one from the pom passing option `-P`, both will be activated simultaneously:
```shell
$ mvn help:active-profiles -P appserverConfig-dev
The following profiles are active:

 - appserverConfig-dev (source: pom)
 - appserverConfig (source: settings.xml)
```

Even though here we get listed the two active profiles, we are not sure which one of them has been applied. To see the effect on the build execute:
```shell
$  mvn help:effective-pom -P appserverConfig-dev
```
This will print the effective POM for this build configuration out to the console. Profiles in the settings.xml have higher priority than profiles in the POM. So the profile that has been applied here is appserverConfig not appserverConfig-dev (I think the author mean the profile whose properties values will override those of the other).

We can redirect the output from the `effective-pom` plugin to a file with flag `-Doutput=effective-pom.xml`.


## Profiles inheritance
http://www.dashbay.com/2011/03/maven-profile-inheritance/

Profiles aren't inherited. Profiles in parent poms can be triggered by the building of a child. When activated, they are applied directly to the parent pom, prior to that parent being used for inheritance into the child. So the effects of an active profile in a parent pom may be felt by the child.

Imagine we have a parent pom with a "prod" profile whose job is to insert some properties into the build. On a production system, Maven will activate that profile and the parent pom will contain the properties. Since properties are inherited from parents, your child pom will get them. The end effect is just like the profile was inherited, but in fact the profile was NOT inherited. Up to here its clear to me.

Things that are inherited in maven are http://www.dashbay.com/2011/03/maven-profile-inheritance/

?
On the other hand, say your parent pom has a "prod" profile that adds a module for publishing your app to a webserver. On your production system, Maven will activate that profile and the parent pom will contain the module.  However, that module will try to publish the parent to the webserver, not your child project.

If you simply insist on trying to “inherit” plugins via a profile, you’d have to:

Set up the profile in the parent pom, but instead of defining the module in a <plugin> section, define it in a <pluginManagement> section.  The pluginManagement is explicitly for declaring plugins that aren’t used in the parent but need to be propagated to the child.
Set up the profile again in the child pom (yes, copy and paste).  Within the profile, REdeclare the plugin  (in a <plugin> section not <pluginManagement>), but without all the configuration.  This will activate the plugin defined in the parent.
Yes, it sucks that you have to double-declare things.   I try to find other solutions if at all possible.