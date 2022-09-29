# Maven


## Maven profiles
 Profiles are specified using a subset of the elements available in the POM itself (plus one extra section), and are triggered in any of a variety of ways. They modify the POM at build time, and are meant to be used in complementary sets to give equivalent-but-different parameters for a set of target environments (providing, for example, the path of the appserver root in the development, testing, and production environments). As such, profiles can easily lead to differing build results from different members of your team. However, used properly, profiles can be used while still preserving project portability.

There are four types of maven profile
- Per Project: defined in the POM itself (`pom.xml`).
- Per User: defined in the Maven-settings (`%USER_HOME%/.m2/settings.xml`).
- Global: defined in the global Maven-settings (`${maven.home}/conf/settings.xml`).
- Profile descriptor: a descriptor located in project basedir (`profiles.xml`) (no longer supported in Maven 3.0 and above; see Maven 3 compatibility notes.)

## Profile activation
A profile can be activated in several ways:
### From the command line
Profiles can be explicitly specified using the `-P` command line flag.

This flag is followed by a comma-delimited list of profile IDs to use. The profile(s) specified in the option are activated in addition to any profiles which are activated by their activation configuration in the pom, or the <activeProfiles> section in the `settings.xml`. Thus we may have several profiles activated at once!

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
Profiles listed in the <activeProfiles> tag would be activated by default every time a project use it.

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
This profile will automatically be active for all builds unless another profile in the same POM is activated via `-P` flag in the command line, or through some activation configuration as we'll see below. In fact, all profiles that are active by default are automatically deactivated when a profile in the POM is activated on the command line with `-P` or through its activation config.

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
As of Maven 3.0, profiles in the POM can also be activated based on properties from active profiles from the `settings.xml`.

Environment variables like `FOO` are available as properties of the form `env.FOO`. Furthermore  environment variable names are normalized to all upper-case on Windows.

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
As of Maven 2.0.9, the tags `<exists>` and `<missing>` could be interpolated. Supported variables are system properties like `${user.home}` and environment variables like `${env.HOME}`. However, properties and values defined in the POM itself are not available for interpolation here, e.g. the above example activator cannot use `${project.build.directory}` but needs to hard-code the path target.

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
<properties> (not actually available in the main POM, but used behind the scenes)
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

