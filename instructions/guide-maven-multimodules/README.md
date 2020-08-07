
# Creating a multi-module application
## What you'll learn

Demo test 7/8 11:26

A Java Platform, Enterprise Edition (Java EE) application consists of modules that work together as one entity. An enterprise archive (EAR) is a wrapper for a Java EE application, which consists of web archive (WAR) and Java archive (JAR) files. Package modules and resources into an EAR file to deploy or distribute the Java EE application to new environments.

You will learn how to establish a dependency between a web module and a Java library module. Next, use Maven to package the WAR file and the JAR file into an EAR file so that you can run and test the application on Open Liberty.

You will build a unit converter application that converts heights from centimeters into feet and inches. Enter heights in centimeters from a web page, and the application processes the input with functions in the JAR file to return the corresponding height in Imperial units.




# Getting Started

If a terminal window does not open navigate:

> Terminal -> New Terminal

Check you are in the **home/project** folder:

```
pwd
```
{: codeblock}

The fastest way to work through this guide is to clone the Git repository and use the projects that are provided inside:

```
git clone https://github.com/openliberty/guide-maven-multimodules.git
cd guide-maven-multimodules
```
{: codeblock}

The **start** directory contains the starting project that you will build upon.


Access partial implementation of the application from the **start** folder. This folder includes a web module in the **war** folder, a Java library in the **jar** folder, and template files in the **ear** folder. However, the Java library and the web module are independent projects, and you will need to complete the following steps to implement the application:

1. Add a dependency relationship between two modules.

2. Assemble the entire application into an EAR file.

3. Aggregate the entire build.

4. Test the multi-module application.

### Try what you'll build

The **finish** directory in the root of this guide contains the finished application. Give it a try before you proceed.

To try out the application, first go to the **finish** directory and run the following
Maven goal to build the application:

mvn install

To deploy your EAR application on an Open Liberty server, run the Maven **liberty:run** goal from the **ear** directory:

cd ear
mvn liberty:run

Once the server is running, you can find the application at the following URL: 
```
curl http://localhost:9080/converter/
```
{: codeblock}



After you are finished checking out the application, stop the Open Liberty server by pressing **CTRL+C** in the command-line session where you ran the server. Alternatively, you can run the **liberty:stop** goal from the **finish\ear** directory in another command-line session:

mvn liberty:stop


# Adding dependencies between WAR and JAR modules

To use the Java library in your web module, add a dependency relationship between them.

As you might notice, each module has its own **pom.xml** file because each module is treated as an independent project. You can rebuild, reuse, and reassemble every module on its own.

Navigate to the **start** directory to begin.

```
cd start
```
{: codeblock}



Replace the war/POM file.

> [File -> Open]guide-maven-multimodules/start/war/pom.xml



```
<?xml version='1.0' encoding='utf-8'?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
    http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>io.openliberty.guides</groupId>
    <artifactId>guide-maven-multimodules-war</artifactId>
    <packaging>war</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>guide-maven-multimodules-war</name>
    <url>http://maven.apache.org</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>

        <!-- Provided dependencies -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>jakarta.platform</groupId>
            <artifactId>jakarta.jakartaee-api</artifactId>
            <version>8.0.0</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.eclipse.microprofile</groupId>
            <artifactId>microprofile</artifactId>
            <version>3.3</version>
            <type>pom</type>
            <scope>provided</scope>
        </dependency>

        <!-- tag::dependency[] -->
        <dependency>
            <groupId>io.openliberty.guides</groupId>
            <artifactId>guide-maven-multimodules-jar</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!-- end::dependency[] -->

    </dependencies>

</project>
```
{: codeblock}



The **<dependency/>** element is the Java library module that implements the functions that you need for the unit converter.

With this dependency, you can use any functions included in the library in the **HeightsBean.java** file of the web module.

Replace the `HeightsBean` class.

> [File -> Open]guide-maven-multimodules/start/war/src/main/java/io/openliberty/guides/multimodules/web/HeightsBean.java



```
package io.openliberty.guides.multimodules.web;

public class HeightsBean implements java.io.Serializable {
    private String heightCm = null;
    private String heightFeet = null;
    private String heightInches = null;
    private int cm = 0;
    private int feet = 0;
    private int inches = 0;

    public HeightsBean() {
    }

    public String getHeightCm() {
        return heightCm;
    }

    public String getHeightFeet() {
        return heightFeet;
    }

    public String getHeightInches() {
        return heightInches;
    }

    public void setHeightCm(String heightcm) {
        this.heightCm = heightcm;
    }

    public void setHeightFeet(String heightfeet) {
        this.cm = Integer.valueOf(heightCm);
        this.feet = io.openliberty.guides.multimodules.lib.Converter.getFeet(cm);
        String result = String.valueOf(feet);
        this.heightFeet = result;
    }

    public void setHeightInches(String heightinches) {
        this.cm = Integer.valueOf(heightCm);
        this.inches = io.openliberty.guides.multimodules.lib.Converter.getInches(cm);
        String result = String.valueOf(inches);
        this.heightInches = result;
    }

}
```
{: codeblock}




The **getFeet(cm)** invocation was added to the **setHeightFeet** method to convert a measurement into feet.

The **getInches(cm)** invocation was added to the **setHeightInches** method to convert a measurement into inches.



# Assembling multiple modules into an EAR file

To deploy the entire application on the Open Liberty server, first package the application. Use the EAR project to assemble multiple modules into an EAR file.

Navigate to the **ear** folder and find a template **pom.xml** file.
Replace the ear/POM file.

> [File -> Open]guide-maven-multimodules/start/ear/pom.xml



```
<?xml version='1.0' encoding='utf-8'?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
    http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <!-- tag::packaging[] -->
    <groupId>io.openliberty.guides</groupId>
    <artifactId>guide-maven-multimodules-ear</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!-- tag::packagingType[] -->
    <packaging>ear</packaging>
    <!-- end::packagingType[] -->
    <!-- end::packaging[] -->

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <!-- Liberty configuration -->
        <liberty.var.default.http.port>9080</liberty.var.default.http.port>
        <liberty.var.default.https.port>9443</liberty.var.default.https.port>
    </properties>

    <dependencies>
        <!-- web and jar modules as dependencies -->
        <!-- tag::dependencies[] -->
        <!-- tag::dependency-jar[] -->
        <dependency>
            <groupId>io.openliberty.guides</groupId>
            <artifactId>guide-maven-multimodules-jar</artifactId>
            <version>1.0-SNAPSHOT</version>
            <type>jar</type>
        </dependency>
        <!-- end::dependency-jar[] -->
        <!-- tag::dependency-war[] -->
        <dependency>
            <groupId>io.openliberty.guides</groupId>
            <artifactId>guide-maven-multimodules-war</artifactId>
            <version>1.0-SNAPSHOT</version>
            <!-- tag::warType[] -->
            <type>war</type>
            <!-- end::warType[] -->
        </dependency>
        <!-- end::dependency-war[] -->
        <!-- end::dependencies[] -->

        <!-- For tests -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.6.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <finalName>${project.artifactId}</finalName>
        <plugins>
            <!-- tag::maven-ear-plugin[] -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-ear-plugin</artifactId>
                <version>3.0.2</version>
                <configuration>
                    <modules>
                        <!-- tag::jarModule[] -->
                        <jarModule>
                            <groupId>io.openliberty.guides</groupId>
                            <artifactId>guide-maven-multimodules-jar</artifactId>
                            <uri>/guide-maven-multimodules-jar-1.0-SNAPSHOT.jar</uri>
                        </jarModule>
                        <!-- end::jarModule[] -->
                        <!-- tag::webModule[] -->
                        <webModule>
                            <groupId>io.openliberty.guides</groupId>
                            <artifactId>guide-maven-multimodules-war</artifactId>
                            <uri>/guide-maven-multimodules-war-1.0-SNAPSHOT.war</uri>
                            <!-- Set custom context root -->
                            <!-- tag::contextRoot[] -->
                            <contextRoot>/converter</contextRoot>
                            <!-- end::contextRoot[] -->
                        </webModule>
                        <!-- end::webModule[] -->
                    </modules>
                </configuration>
            </plugin>
            <!-- end::maven-ear-plugin[] -->

            <!-- Enable liberty-maven plugin -->
            <!-- tag::liberty-maven-plugin[] -->
            <plugin>
                <groupId>io.openliberty.tools</groupId>
                <artifactId>liberty-maven-plugin</artifactId>
                <version>3.2</version>
            </plugin>
            <!-- end::liberty-maven-plugin[] -->

            <!-- Since the package type is ear,
            need to run testCompile to compile the tests -->
            <!-- tag::maven-compiler-plugin[] -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <executions>
                    <execution>
                        <phase>test-compile</phase>
                        <!-- tag::testCompile[] -->
                        <goals>
                            <goal>testCompile</goal>
                        </goals>
                        <!-- end::testCompile[] -->
                    </execution>
                </executions>
            </plugin>
            <!-- end::maven-compiler-plugin[] -->

            <!-- Plugin to run integration tests -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-failsafe-plugin</artifactId>
                <version>2.22.2</version>
                <configuration>
                    <systemPropertyVariables>
                        <default.http.port>
                            ${liberty.var.default.http.port}
                        </default.http.port>
                        <default.https.port>
                            ${liberty.var.default.https.port}
                        </default.https.port>
                        <cf.context.root>/converter</cf.context.root>
                    </systemPropertyVariables>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```
{: codeblock}




Set the **basic configuration** for the project and set the **<packaging/>** element to **ear**.

The **Java library module** and the **web module** were added as dependencies. Specify **<type>war</type>** for the web module. If you donâ€™t specify the web module, Maven looks for a JAR file.

The definition and configuration of the **maven-ear-plugin** plug-in were added to create an EAR file. Define the **<jarModule/>** and **<webModule/>** modules to be packaged into the EAR file.
To customize the context root of the application, set the **<contextRoot>/converter</contextRoot>** element in the **<webModule/>**. Otherwise, Maven automatically uses the WAR file **artifactId** ID as the context root for the application while generating the **application.xml** file.

To download and start an Open Liberty server, use the **liberty-maven-plugin** plug-in for Maven. This configuration is provided, and the executions of the plug-in follow the typical phases of a Maven life cycle.

To deploy the EAR application, you need to configure a server.

Create the server configuration file.
```
touch ear/src/main/liberty/config/server.xml
```
{: codeblock}


> [File -> Open]guide-maven-multimodules/start/ear/src/main/liberty/config/server.xml
```

<server description="Sample Liberty server">

    <featureManager>
        <feature>jsp-2.3</feature>
    </featureManager>

    <variable name="default.http.port" defaultValue="9080" />
    <variable name="default.https.port" defaultValue="9443" />

    <!-- tag::server[] -->
    <httpEndpoint host="*" httpPort="${default.http.port}"
        httpsPort="${default.https.port}" id="defaultHttpEndpoint" />

    <enterpriseApplication id="guide-maven-multimodules-ear"
        location="guide-maven-multimodules-ear.ear"
        name="guide-maven-multimodules-ear" />
    <!-- end::server[] -->
</server>
```
{: codeblock}





# Aggregating the entire build

Because you have multiple modules, aggregate the Maven projects to simplify the build process.

Create a parent **pom.xml** file under the **start** directory to link all of the child modules together. A template is provided for you.

Replace the start/POM file.

> [File -> Open]guide-maven-multimodules/start/pom.xml



```
<?xml version='1.0' encoding='utf-8'?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
    http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <!-- tag::packaging[] -->
    <!-- tag::groupId[] -->
    <groupId>io.openliberty.guides</groupId>
    <!-- end::groupId[] -->
    <artifactId>guide-maven-multimodules</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!-- tag::packagingType[] -->
    <packaging>pom</packaging>
    <!-- end::packagingType[] -->
    <!-- end::packaging[] -->

    <!-- tag::modules[] -->
    <modules>
        <module>jar</module>
        <module>war</module>
        <module>ear</module>
    </modules>
    <!-- end::modules[] -->
</project>
```
{: codeblock}



Set the **basic configuration** for the project. Set **pom** as the **<packaging/>** element of the parent **pom.xml** file.

In the parent **pom.xml** file, list all of the **<modules/>** that you want to aggregate for the application.



# Building the modules

By aggregating the build in the previous section, you can run **mvn install** once from the **start** directory and it will automatically build all your modules. This command creates a JAR file in the **jar/target** directory, a WAR file in the **war/target** directory, and an EAR file in the **ear/target** directory, which contains the JAR and WAR files.

Use the following command to build the entire application from the **start** directory:

mvn install

Since the modules are independent, you can re-build them individually by running **mvn install** from the corresponding module directory.


# Starting the application

To deploy your EAR application on an Open Liberty server, run the Maven **liberty:run** goal from the **ear** directory:

cd ear
mvn liberty:run

Once the server is running, you can find the application at the following URL: 
```
curl http://localhost:9080/converter/
```
{: codeblock}



After you are finished checking out the application, stop the Open Liberty server by pressing **CTRL+C** in the command-line session where you ran the server. Alternatively, you can run the **liberty:stop** goal from the **start\ear** directory in another command-line session:

mvn liberty:stop



# Testing the multi-module application

To test the multi-module application, add integration tests to the EAR project.

Navigate to the **start\ear** directory.

Create the integration test class.
```
touch src/test/java/it/io/openliberty/guides/multimodules/IT.java
```
{: codeblock}


> [File -> Open]guide-maven-multimodules/start/src/test/java/it/io/openliberty/guides/multimodules/IT.java
```



The **testIndexPage** tests to check that you can access the landing page.

The **testHeightsPage** tests to check that the application can process the input value and calculate the result correctly.

For a Maven EAR project, the **testCompile** goal is
specified for the **maven-compiler-plugin**
plug-in in your **ear/pom.xml** file so that the test cases are
compiled and picked up for execution. Run the following command to compile the
test class:

mvn test-compile



Then, enter the following command to start the server, run the tests, then stop the server:

mvn liberty:start failsafe:integration-test liberty:stop

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running it.io.openliberty.guides.multimodules.IT
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.712 sec - in it.io.openliberty.guides.multimodules.IT

Results :

Tests run: 2, Failures: 0, Errors: 0, Skipped: 0



# Summary

## Clean up your environment

Delete the **guide-maven-multimodules** project by navigating to the **/home/project/** directory

```
cd ../..
rm -r -f guide-maven-multimodules
rmdir guide-maven-multimodules
```
{: codeblock}


# Great work! You're done!

You built and tested a multi-module unit converter application on the with Maven in Open Liberty.

