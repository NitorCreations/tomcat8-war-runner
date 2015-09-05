# tomcat8-war-runner

## Introduction

Currently there exists no Tomcat 8 version of Tomcat Maven plugin. Exec-wars built using [Tomcat 7 Maven plugin](http://tomcat.apache.org/maven-plugin-trunk/tomcat7-maven-plugin/) work on Tomcat 8, except if you want to use a custom `server.xml`.

In addition to fixing the issue with `server.xml`, this runner sets the [exec-war parameters affecting Tomcat configuration to system properties](http://tomcat.apache.org/maven-plugin-2.0/executable-war-jar.html#Generated_executable_jarwar), when a custom `server.xml` is detected. This enables you to refer them in the custom `server.xml` and smoothly switch between having and not having a custom `server.xml`.

## Usage

This runner has not (yet) been published to Maven Central, so currently you need to create a local repository under your web project.

* Clone and build this repository (`mvn package`)
* Create directory `lib/com/nitorcreations/tomcat8-war-runner/1.0-SNAPSHOT` under your web project. Copy `target/tomcat8-war-runner-1.0-SNAPSHOT.jar` from the previous step to the new directory.
* Setup local repository pointing to the created directory in you web project `pom.xml`:
```
<repositories>
  <repository>
    <id>lib_id</id>
    <url>file://${project.basedir}/lib</url>
  </repository>
</repositories>
```
* Configure you tomcat7-maven-plugin to use this runner in your web project `pom.xml`:
```
<plugin>
  <groupId>org.apache.tomcat.maven</groupId>
  <artifactId>tomcat7-maven-plugin</artifactId>
  <executions>
    <execution>
      <id>build-exec-war</id>
      <configuration>
        <path>/app/v1</path>
        <mainClass>org.apache.tomcat.maven.runner.Tomcat8RunnerCli</mainClass>
        <extraDependencies>
          <extraDependency>
            <groupId>com.nitorcreations</groupId>
            <artifactId>tomcat8-war-runner</artifactId>
            <version>1.0-SNAPSHOT</version>
          </extraDependency>
        </extraDependencies>
      </configuration>
    </execution>
  </executions>
</plugin>
```
* Create your custom `server.xml` file to your web project directory `src/main/tomcatconf/server.xml`. For example:
```
<Server port="8010" shutdown="SHUTDOWN">
  <GlobalNamingResources></GlobalNamingResources>
  <Service name="Catalina">
    <Connector port="${httpPort}" maxThreads="${maxThreads}" protocol="HTTP/1.1"/>
    <Engine name="Catalina" defaultHost="localhost">
      <Valve className="org.apache.catalina.valves.AccessLogValve" 
             pattern="%h %l %u %t &quot;%r&quot; %s %D %b &quot;%{Referer}i&quot; &quot;%{User-Agent}i&quot;"/>
      <Valve className="org.apache.catalina.valves.RemoteIpValve"/> 
      <Host name="localhost" appBase="webapps">
        <Context path="/app/v1" docBase="app/v1.war"/>
      </Host>
    </Engine>
  </Service>
</Server>
```
* After rebuilding your web project, you can run and configure your `app-webapp-1.0-SNAPSHOT-war-exec.jar` by mixing default Tomcat Maven plugin parameters (httpPort) and/or system properties (maxThreads) depending on your needs. (System properties take precedence over default Tomcat Maven plugin parameters, if both are defined.)
```
java -DmaxThreads=300 -jar app-webapp-1.0-SNAPSHOT-war-exec.jar -httpPort 9999
```
