# Ericsson SmartHome Packaging (SmartHome on Concierge)

<img src="http://logok.org/wp-content/uploads/2014/05/Ericsson-logo-blue.png" alt="ericsson logo" width="200"/>

This repo contains a maven build to create the complete smarthome runtime for ericsson smartSTB.
As runtime concierge is used. More information about concierge: https://www.eclipse.org/concierge/index.php

1. Prerequisites - Install Maven
================
Please use the instructions on main project's readme to install maven: https://github.com/eclipse/smarthome#1-prerequisites
* Make sure **mvn** command is available on your path.
* Make sure you're using a **JDK 8**.
* Add credentials to Ericsson artifactory to your maven configuration (maven/conf/settings.xml) like this

```
	    <server>
            <id>ericsson-snapshots</id>
            <username>jenkins</username>
            <password>THE_PASSWORD</password>
        </server>

```

2. Checkout
================
Checkout the source code from GitHub, e.g. by running:

```
git clone git@github.com:Ericsson-Smart-Home/smarthome-packaging-sample.git
```

3. Define Eclipse SmartHome version
================
You can define which Eclipse SmartHome version you want to use for packaging. 
* define the version in [pom.xml](/pom.xml#L18)
* define the version in [smarthome.xargs](/distro/runtime/concierge/smarthome.xargs#L6)

See here for example the available versions on nexus: https://repo.eclipse.org/content/repositories/releases/org/eclipse/smarthome/core/org.eclipse.smarthome.core/


4. Build the distribution
================
Run
```
mvn clean install
```

The maven build will create an ZIP file with all required components like
* the concierge runtime
* Eclipse SmartHome bundles
* 3rd party bundles
* Ericsson bundles/bindings
* start scripts

You can find the created distribution under **/target/smarthome-packaging-sample-[version].zip**

### The directory structure of the distribution
* **addons**: Folder for hotdeployment of bundles
* **runtime**: Contains the runtime
 * **concierge**: Contains the conciege osgi runtime
    * **bundles**: Additional concierge bundles
    * **framework**: The Framework
    * **system**: Contains commons and 3rd party bundles
      * **org.eclipse.jetty**: All jetty bundles
      * **org.eclipse.smarthome**: All SmartHome bundles
      * **com.ericsson.smarthome**: All ericsson bundles/bindings.
 * **etc**: Quartz configuration, Jetty configuration, keystore
* **userdata**: This folder is created during the first startup and contains persistent userdata and the osgi storage.

5. Start runtime
================
Extract the distribution zip file and start the runtime:
```
unzip smarthome-packaging-sample-[version].zip
./start.sh
```

6. Using the UI
================
The distribution already includes the PaperUI. 
Goto: **http://your-host:8080/** you will be redirected to **/ui/index.html**

Also included is the Apache Felix Web Console. The Apache Felix Web Console is a simple tool to inspect and manage OSGi framework
Goto: **http://your-host:8080/system/console/**

Customize the distribution
================
## Concierge configuration with XARGS file
The .xargs file can contain both runtime properties for configuring the framework, as well as a set of framework commands. Properties are declared as `-Dkey=value`. The following commands are allowed:

* `-install <bundle URL> `: installs a bundle from a given bundle URL
* `-start <bundle URL> `: starts a bundle that was previously installed from the given bundle URL
* `-istart <bundle URL> `: install and starts a bundle from a given bundle URL
* `-all <file directory> `: install and start all .jar files in a given directory
* `-initlevel <level> `: sets the startlevel that will be used for all next bundles to be installed
* `-skip `: skips the remainder of the .xargs file (handy for debugging)

See for more details: https://www.eclipse.org/concierge/documentation.php#basic

## How to add further bundles?
1. Add your bundle as dependecy in the pom.xml. For example:
```
  <!-- Eclipse SmartHome dependencies - Bindings -->
        <dependency>
            <groupId>org.eclipse.smarthome.binding</groupId>
            <artifactId>org.eclipse.smarthome.binding.wemo</artifactId>
            <version>${esh.version}</version>
        </dependency>
```
2. Add the install and startup command to XARGS file. For example:
```
# Eclipse SmartHome Bindings. Start all bindings here
-istart ${esh.dir}/org.eclipse.smarthome.binding.wemo-${esh.version}*.jar
```

The bundle couldn't be started? Please check the following conditions:
 * Is the JAR file included in the assembled ZIP? Check the /src/assemble/concierge.xml which defines the included file sets of the assembly.
 * Maybe the required import-packages are not satisfied. Add another bundle which exports the required packages.

## Change the OSGi Shell
The sample does already include two versions of osgi shell implementations:
 * Concierge Shell (default)
 * Apache Felix GoGo
 
If you want using Apache Felix GoGo simple uncomment the GoGo bundles and remove startup of concierge shell 
like:
```
# -istart ${concierge.dir}/org.eclipse.concierge.shell-${concierge.version}*.jar
-istart ${system.dir}/org.apache.felix.gogo.runtime-0.*.jar
-istart ${system.dir}/org.apache.felix.gogo.command-0.*.jar
-istart ${system.dir}/org.apache.felix.gogo.shell-0.*.jar
```

At next startup you will see the **g!** prompt instead of **concierge>**

## Passing own options to JVM
You can use `JAVA_OPTS` for passing own parameter to JVM e.g. the Java Heap size etc.

Limitations
================
It's not possible to run XText under concierge due to dependencies on equinox runtime. 
For this reason all modeling packages have been removed from this distribution. 
There are no .items, .rule, .thing, etc. files. Try to use new generation rule engine to manage rules.
