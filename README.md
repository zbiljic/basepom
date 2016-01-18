# Base POM

## Usage

Add BasePom as the parent to a project:

```xml
<parent>
  <groupId>com.zbiljic</groupId>
  <artifactId>basepom</artifactId>
  <version> ... current pom release version ...</version>
</parent>
```


## Project pom structure

The following elements should be present in a pom using BasePom as parent:

* `groupId`, `artifactId`, `version`, `packaging`, `name`, `description` and `inceptionYear`

  Define the new project. These elements should always be present. If any of those fields are missing from the project, the values from BasePom are picked up instead.

* `scm`

  Defines the SCM location and URL for the project. This is required to use the `release:prepare` and `release:perform` targets
  to deploy artifacts to Maven Central.

* `distributionManagement`, `organization`, `developers`

  Empty elements override the values inherited from BasePom.

This is a sample skeleton pom using BasePom:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <parent>
    <groupId>com.zbiljic</groupId>
    <artifactId>basepom</artifactId>
    <version> ... current version ... </version>
  </parent>

  <groupId> ... group id of the new project ... </groupId>
  <artifactId> ... artifact id of the new project ... </artifactId>
  <version> ... version of the new project ... </version>
  <packaging> ... jar|pom ... </packaging>

  <name>${project.artifactId}</name>
  <description> ... description of the new project ...  </description>

  <inceptionYear>2016</inceptionYear>

  <licenses/>

  <scm>
    <connection> ... scm read only connection ... </connection>
    <developerConnection>... scm read write connection ... </developerConnection>
    <url> ... project url ... </url>
  </scm>

  <distributionManagement/>

  <organization/>
  <developers/>
  ...
</project>
```


## Project POM conventions

In large maven projects, especially with multi-module builds, the pom files can become quite large. In many places, properties defined in the `<properties>` section of the pom are used.

To avoid confusion with properties, the following conventions are used in BasePom:

* Properties defined in the POM that influence the build configuration are prefixed with `basepom`.
* Properties that factor out plugin versions (because the plugin is used in multiple places in the POM and the versions should be uniform) start with `dep.plugin` and end with `version`.
* Properties that factor out dependency versions (to enforce uniform dependency versions across multiple, related dependencies) start with `dep` and end with `version`.

Examples:

```xml
<properties>
  <!-- Sets the minimum maven version to build (influences build) -->
  <basepom.maven.version>3.0.4</basepom.maven.version>

  <!-- The surefire plugin version -->
  <dep.plugin.surefire.version>2.13</dep.plugin.surefire.version>

  <!-- The version for all guice related dependencies -->
  <dep.guice.version>4.0</dep.guice.version>
<properties>
```


## Project Build and Checkers

BasePom hooks various checkers into the build lifecycle and executes them on each build.

Generally speaking, running a set of checks at each build is a good way to catch problems early and any problem reported by a checker should be treated as something that needs to be fixed before releasing.

Checkers are organized in two groups, basic and extended.

### Basic checkers
* Maven Enforcer                  (http://maven.apache.org/enforcer/maven-enforcer-plugin/)
* Maven Dependencies              (http://maven.apache.org/plugins/maven-dependency-plugin/)
* Maven Dependency version check  (https://github.com/ning/maven-dependency-versions-check-plugin)
* Maven Duplicate finder          (https://github.com/basepom/duplicate-finder-maven-plugin)

### Extended checkers
* Findbugs                        (http://mojo.codehaus.org/findbugs-maven-plugin/)
* Code coverage                   (http://www.eclemma.org/jacoco/trunk/doc/maven.html)
* PMD                             (http://maven.apache.org/plugins/maven-pmd-plugin/)
* Checkstyle                      (http://maven.apache.org/plugins/maven-checkstyle-plugin/)

All checkers are enabled by default, but the checkers will *NOT* fail the build if a problem is encountered.

Each checker has a switch to turn it on or off and also whether a problem will be a warning or fatal to the build.

```xml
...
<properties>
  <!-- Do not run the duplicate finder --->
  <basepom.check.skip-duplicate-finder>true</basepom.check.skip-duplicate-finder>
</properties>
...
```

The following switches exist:

<table>
  <tr>
    <th>Group</th>
    <th>Check</th>
    <th>Skip check (Setting to <tt>true</tt> skips the check)</th>
    <th>Fail build (Setting to <tt>false</tt> only reports a warning)</th>
  </tr>
  <tr>
    <td>Basic</td>
    <td>Maven Enforcer</td>
    <td><tt>basepom.check.skip-enforcer</tt></td>
    <td><tt>basepom.check.fail-enforcer</tt></td>
  </tr>
  <tr>
    <td>Basic</td>
    <td>Maven Dependencies</td>
    <td><tt>basepom.check.skip-dependency</tt></td>
    <td><tt>basepom.check.fail-dependency</tt></td>
  </tr>
  <tr>
    <td>Basic</td>
    <td>Maven Dependency version check</td>
    <td><tt>basepom.check.skip-dependency-version-check</tt></td>
    <td><tt>basepom.check.fail-dependency-version-check</tt></td>
  </tr>
  <tr>
    <td>Basic</td>
    <td>Maven Duplicate finder</td>
    <td><tt>basepom.check.skip-duplicate-finder</tt></td>
    <td><tt>basepom.check.fail-duplicate-finder</tt></td>
  </tr>
  <tr>
    <td>Extended</td>
    <td>Findbugs</td>
    <td><tt>basepom.check.skip-findbugs</tt></td>
    <td><tt>basepom.check.fail-findbugs</tt></td>
  </tr>
  <tr>
    <td>Extended</td>
    <td>Code coverage</td>
    <td><tt>basepom.check.skip-coverage</tt></td>
    <td><tt></tt></td>
  </tr>
  <tr>
    <td>Extended</td>
    <td>PMD</td>
    <td><tt>basepom.check.skip-pmd</tt></td>
    <td><tt>basepom.check.fail-pmd</tt></td>
  </tr>
  <tr>
    <td>Extended</td>
    <td>Checkstyle</td>
    <td><tt>basepom.check.skip-checkstyle</tt></td>
    <td><tt>basepom.check.fail-checkstyle</tt></td>
  </tr>
</table>

Checks can be turned on and off in groups:

<table>
  <tr>
    <th>Group</th>
    <th>Skip checks</th>
    <th>Fail build</th>
  </tr>
  <tr>
    <td>All Checks</td>
    <td><tt>basepom.check.skip-all</tt></td>
    <td><tt>basepom.check.fail-all</tt></td>
  </tr>
  <tr>
    <td>All Basic checks</td>
    <td><tt>basepom.check.skip-basic</tt></td>
    <td><tt>basepom.check.fail-basic</tt></td>
  </tr>
  <tr>
    <td>All Extended Checks</td>
    <td><tt>basepom.check.skip-extended</tt></td>
    <td><tt>basepom.check.fail-extended</tt></td>
  </tr>
</table>

A more specific setting (checker specific, then group, then all) overrides a more general setting:

```xml
...
<properties>
  <basepom.check.skip-all>true</basepom.check.skip-all>
  <basepom.check.skip-duplicate-finder>false</basepom.check.skip-duplicate-finder>
</properties>
...
```
will skip *all* checks except the duplicate finder.


### Checker notes

#### Enforcer checker

The Enforcer plugin outlaws a number of dependencies that project might use for various reasons:

<table>
  <tr>
    <th>Outlawed dependency</th>
    <th>Rationale</th>
    <th>What to use</th>
  </tr>
  <tr>
    <td><tt>junit:junit</tt></td>
    <td>Has all its dependencies packed inside, therefore leads to duplicate classes.</td>
    <td><tt>junit:junit-dep</tt></td>
  </tr>
  <tr>
    <td><tt>com.google.collections:google-collections</tt></td>
    <td>Superseded by Guava, duplicate classes with Guava.</td>
    <td><tt>com.google.guava:guava</tt></td>
  </tr>
  <tr>
    <td><tt>com.google.code.findbugs:jsr305</tt></td>
    <td>Subset of the full findbugs annotations, contains only the JSR-305 annotations.</td>
    <td><tt>com.google.code.findbugs:annotations</tt></td>
  </tr>
  <tr>
    <td><tt>org.eclipse.jetty.orbit:javax.servlet</tt></td>
    <td>Jetty variant of the 3.x servlet API jar.</td>
    <td><tt>javax.servlet:javax.servlet-api</tt></td>
  </tr>
</table>


## Well known dependencies

BasePom provides a number of dependencies to projects. These dependencies are considered "well known and stable". When a project wants to use any of these dependencies, it can declare them in the project `<dependencies>` section without a version and automatically pick it up from BasePom.

BasePom provides versions for the following well-known dependencies:

<table>
  <tr><th>Dependency name</th><th>Group/Artifact Ids</th></tr>
  <tr>
    <td>Findbugs Annotations</td>
    <td><tt>com.google.code.findbugs:annotations</tt></td>
  </tr>
  <tr>
   <td>Google Guava</td>
   <td><tt>com.google.guava:guava</tt></td>
  </tr>
  <tr>
   <td>Google Guice</td>
   <td><tt>com.google.inject:guice</tt><p/><tt>com.google.inject.extensions:guice-servlet</tt><p/><tt>com.google.inject.extensions:guice-assistedinject</tt><p/><tt>com.google.inject.extensions:guice-multibindings</tt><p/><tt>com.google.inject.extensions:guice-throwingproviders</tt></td>
  </tr>
  <tr>
    <td>slf4j (Simple Logging Facade for Java)</td>
    <td><tt>org.slf4j:slf4j-api</tt><p/><tt>org.slf4j:slf4j-nop</tt><p/><tt>org.slf4j:slf4j-simple</tt><p/><tt>org.slf4j:slf4j-ext</tt><p/><tt>org.slf4j:jcl-over-slf4j</tt><p/><tt>org.slf4j:jul-to-slf4j</tt><p/><tt>org.slf4j:log4j-over-slf4j</tt><p/><tt>org.slf4j:slf4j-jdk14</tt></td>
  </tr>
  <tr>
    <td>Java Inject API</td>
    <td><tt>javax.inject:javax.inject</tt></td>
  </tr>
  <tr>
    <td>Java Servlet API</td>
    <td><tt>javax.servlet:javax.servlet-api</tt></td>
  </tr>
  <tr>
    <td>Java Validation API</td>
    <td><tt>javax.validation:validation-api</tt></td>
  </tr>
  <tr>
    <td>Bean Validation Framework</td>
    <td><tt>org.apache.bval:bval-jsr303</tt></td>
  </tr>
  <tr>
    <td>JmxUtils</td>
    <td><tt>org.weakref:jmxutils</tt></td>
  </tr>
  <tr>
    <td>JMH</td>
    <td><tt>org.openjdk.jmh:jmh-core</tt><p/><tt>org.openjdk.jmh:jmh-generator-annprocess</tt></td>
  </tr>
  <tr>
    <td>TestNG testing</td>
    <td><tt>org.testng:testng</tt></td>
  </tr>
  <tr>
    <td>Mockito</td>
    <td><tt>org.mockito:mockito-core</tt></td>
  </tr>
  <tr>
    <td>Hamcrest matchers</td>
    <td><tt>org.hamcrest:hamcrest-core</tt><p/><tt>org.hamcrest:hamcrest-library</tt></td>
  </tr>
  <tr>
    <td>Objenesis</td>
    <td><tt>org.objenesis:objenesis</tt></td>
  </tr>
</table>


## Other properties

These are default properties that affect some aspects of the build. All of them can be overridden in the `<properties>` section of the project pom.

### project.build.targetJdk

By default, BasePom enforces JDK 1.8. To use another version, add

```xml
<properties>
  <project.build.targetJdk>1.6</project.build.targetJdk>
  ...
</properties>
```


### basepom.build.jvmsize

Sets the default heap size for the compiler, javadoc generation and other plugins. Default is 1024M.

### basepom.release.push-changes

When a project creates a release using the maven-release-plugin and `mvn release:prepare`, this switch controls whether the generated tags, modified POM files etc. are pushed automatically to the upstream repository or not. Default is `false` (do not push the changes).

### basepom.maven.version

The minimum version of Maven to build a project. Default is "3.x".

### basepom.main.basedir

The 'root' directory of a project. For a simple project, this is identical to `project.basedir`. In a multi-module build, it should point at the root project.

For a multi-module project, all other sub-modules must have this explicitly set to the root directory and therefore the following code

```xml
<properties>
  <basepom.main.basedir>${project.parent.basedir}</basepom.main.basedir>
</properties>
```


must be added to each pom. This is a limitation of the Maven multi-module build process (see http://stackoverflow.com/questions/1012402/maven2-property-that-indicates-the-parent-directory for details).

### basepom.test.fork-count

Defines the number of VMs to fork in parallel in order to execute the tests. When terminated with "C", the number part is multiplied with the number of CPU cores. Floating point value are only accepted together with "C". If set to "0", no VM is forked and all tests are executed within the main process.
Default is "1.0C".


