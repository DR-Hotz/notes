## Maven Usage

### Build Lifecycle

	Lifecycles:
		default
		clean
		site

	Build phases:
		validate : validate the project is correct and all necessary information is available

		compile : compile the source code of the project

		test : test the compiled source code using a suitable unit testing framework. These tests should not require the code be packaged or deployed

		package : take the compiled code and package it in its distributable format, such as a JAR

		verify : run any checks on results of integration tests to ensure quality criteria are met

		install : install the package into the local repository, for use as a dependency in other projects locally

		deploy : done in the build environment, copies the final package to the remote repository for sharing with other developers and projects.

### Dependency

	scopes:
		compile: default scope, dependencies are propagated to dependent projects

		provided: much like compile, but indicates you expect the JDK or a container to provide the dependency at runtime. This scope is only available on the compilation and test classpath, and is not transitive

		runtime: This scope indicates that the dependency is not required for compilation, but is for execution. It is in the runtime and test classpaths, but not the compile classpath.

		test: indicates that the dependency is not required for normal use of the application, and is only available for the test compilation and execution phases. This scope is not transitive.

		system: similar to provided except that you have to provide the JAR which contains it explicitly. The artifact is always available and is not looked up in a repository.

		import:

#### Transitive Dependencies



### Pom configuration

```shell

<project>
[...]
	<repositories>
		<repository>
        	<releases />
			<id>offline-repository</id>
			<name>local offline build repo</name>
			<url>file:///c:/repo/offline-repo-4/</url>
		</repository>
	</repositories>
	<pluginRepositories>
    	<pluginRepository>
        	<id>blaRepo</id>
        	<url>file:///c:/repo/offline-repo-4</url>
        	<layout>default</layout>
      	</pluginRepository>
    </pluginRepositories>
[...]
</project>
```

```shell
<settings>
	<profile>
		<id>lc</id>
		<pluginRepositories>
  			<pluginRepository>
  				<id>central</id>
          		<url>file:///data/local/localRepo</url>
          		<releases>
            		<enabled>true</enabled>
          		</releases>
          		<snapshots>
            		<enabled>true</enabled>
          		</snapshots>
        	</pluginRepository>
      	</pluginRepositories>
      	<repositories>
        	<repository>
          	<id>cre</id>
          	<url>file:///data/local/localRepo</url>
          	<releases>
            	<enabled>true</enabled>
          	</releases>
        </repository>
      </repositories>
    </profile>
</settings>


```

### Plugins


#### maven-clean-plugin

	Goals:
		clean:clean

```shell
<plugin>
	<artifactId>maven-clean-plugin</artifactId>
	<version>3.0.0</version>
	<executions>
		<execution>
			<id>auto-clean</id>
            <phase>initialize</phase>
            <goals>
				<goal>clean</goal>
	            </goals>
	 	</execution>
	</executions>
</plugin>

```


#### maven-source-plugin

	Goals:
		source:aggregate aggregrates sources for all modules in an aggregator project.
		source:jar is used to bundle the main sources of the project into a jar archive.
		source:test-jar on the other hand, is used to bundle the test sources of the project into a jar archive.
		source:jar-no-fork is similar to jar but does not fork the build lifecycle.
		source:test-jar-no-fork

```shell
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-source-plugin</artifactId>
	<version>3.0.1</version>
	<executions>
    	<execution>
        	<id>attach-sources</id>
            <phase>verify</phase>
            <goals>
            	<goal>jar-no-fork</goal>
            </goals>
      	</execution>
  	</executions>
</plugin>

```


#### maven-javadoc-plugin

	Goals:
		javadoc:javadoc
		javadoc:jar
		javadoc:aggregate
		javadoc:aggregate-jar
		javadoc:test-javadoc
		javadoc:test-jar
		javadoc:test-aggregate
		javadoc:test-aggregate-jar

```shell
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-javadoc-plugin</artifactId>
	<version>2.10.4</version>
	<configuration>
          ...
	</configuration>
</plugin>

```

#### maven-dependency-plugin

	Goals : 
		dependency:go-offline
		dependency:copy-dependencies
		dependency:unpack-dependencies

```shell
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>3.0.0</version>
    <executions>
    	<execution>
        	<id>copy-dependencies</id>
        	<phase>package</phase>
          	<goals>
          		<goal>copy-dependencies</goal>
         	</goals>
         	<configuration>
            	<outputDirectory>${project.build.directory}/alternateLocation</outputDirectory>
            	<overWriteReleases>false</overWriteReleases>
            	<overWriteSnapshots>false</overWriteSnapshots>
            	<overWriteIfNewer>true</overWriteIfNewer>
          	</configuration>
      	</execution>
   	</executions>
</plugin>

```



#### maven-compiler-plugin

	Goals:
		compile

```shell
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.6.1</version>
    <configuration>
    	<source>1.7</source>
    	<target>1.7</target>
    </configuration>
</plugin>

```


#### maven-install-plugin

	Actions:
		install:install
		install:install-file


#### maven-deploy-plugin


	Goal:
		deploy:deploy
		deploy:deploy-file

#### maven-jar-plugin

	Goals:
		jar:jar
		jar:test-jar

```shell
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-jar-plugin</artifactId>
	<version>3.0.2</version>
	<configuration>
		<archive>
	  		<manifestFile>${project.build.outputDirectory}/META-INF/MANIFEST.MF</manifestFile>
         </archive>
 	</configuration>
        ..
</plugin>
```


#### Archiver


```shell
<archive>
  <addMavenDescriptor/>
  <compress/>
  <forced/>
  <index/>
  <manifest>
    <addClasspath/>
    <addDefaultImplementationEntries/>
    <addDefaultSpecificationEntries/>
    <addExtensions/>
    <classpathLayoutType/>
    <classpathMavenRepositoryLayout/>
    <classpathPrefix/>
    <customClasspathLayout/>
    <mainClass/>
    <packageName/>
    <useUniqueVersions/>
  </manifest>
  <manifestEntries>
    <key>value</key>
  </manifestEntries>
  <manifestFile/>
  <manifestSections>
    <manifestSection>
      <name/>
      <manifestEntries>
        <key>value</key>
      </manifestEntries>
    <manifestSection/>
  </manifestSections>
  <pomPropertiesFile/>
</archive>

```


### Nexus