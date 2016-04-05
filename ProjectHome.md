# maven plugins #
  * [maven-db-plugin](http://code.google.com/p/maven-db-plugin/) (sql)
  * [maven-mongodb-plugin](http://code.google.com/p/maven-mongodb-plugin/) (MongoDB)

# repository configuration: #

```
    <repository>
        <id>maven-db-plugin-repo</id>
        <name>maven db plugin repository</name>
        <url>http://maven-db-plugin.googlecode.com/svn/maven/repo</url>
        <layout>default</layout>
    </repository>
```

# goals #
  * db:create - execute the create database statements
  * db:drop - execute the drop database statements
  * db:schema - execute all of the scripts in the schema scripts directory
  * db:data - execute all of the data in the schema scripts directory
  * db:update - execute all of the update in the schema scripts directory

# common usage #
to get started:
```
mvn db:create db:schema db:data
```

to re-initialize database:
```
mvn db:drop db:create db:schema db:data
```

to update database:
```
mvn db:update
```

# sample development database configuration #

this sample assumes the following:
  * database is mysql and located at localhost - root password is empty
  * database name is someAppDb, user is someAppDbUser and password is someAppDbPassword
  * schema scripts are in src/main/sql/schema
  * data scripts are in src/main/sql/data
  * update scripts are in src/main/sql/update

```
<profiles>

	<!-- 
	 | Developer profile
	 +-->
	<profile>
		<id>dev</id>
		<activation><activeByDefault>true</activeByDefault></activation>
		<build>
			<plugins>
				<plugin>
					<groupId>com.googlecode</groupId>
					<artifactId>maven-db-plugin</artifactId>
					<version>1.4</version>
					<configuration>
						<adminDbConnectionSettings>
							<jdbcDriver>com.mysql.jdbc.Driver</jdbcDriver>
							<jdbcUrl>jdbc:mysql://localhost:3306/</jdbcUrl>
							<userName>root</userName>
							<password></password>
						</adminDbConnectionSettings>
						<appDbConnectionSettings>
							<jdbcDriver>com.mysql.jdbc.Driver</jdbcDriver>
							<jdbcUrl>jdbc:mysql://localhost:3306/someAppDb</jdbcUrl>
							<userName>someAppDbUser</userName>
							<password>someAppDbPassword</password>
						</appDbConnectionSettings>
						<sqlDelimiter> #--;</sqlDelimiter>
						<dbDataScriptsDirectory><param>src/main/sql/data</param></dbDataScriptsDirectory>
						<dbSchemaScriptsDirectory><param>src/main/sql/schema</param></dbSchemaScriptsDirectory>
						<dbUpdateScriptsDirectory><param>src/main/sql/update</param></dbUpdateScriptsDirectory>
						<dbCreateStatements>
							create database someAppDb; #--;
							grant all on someAppDb.* tosomeAppDbUser@localhost identified by 'someAppDbPassword'; #--;
							flush privileges; #--;
						</dbCreateStatements>
						<dbDropStatements>drop database someAppDb; #--;</dbDropStatements>
						<!-- optional <scriptEncoding>UTF-8</scriptEncoding> -->
					</configuration>
					<dependencies>
						<dependency>
							<groupId>mysql</groupId>
							<artifactId>mysql-connector-java</artifactId>
							<version>5.1.9</version>
						</dependency>
					</dependencies>
				</plugin>
			</plugins>
		</build>
	</profile>
</profiles>
```

the `adminDbConnectionSettings` is for administrative access to the database (ability to create and drop databases - etc.).  `appDbConnectionSettings` is for  non-admin access to the database.  This is the database that you would be working on in your application for example.  Both of these also support the `<serverId>` attribute in case you'd rather store the host/usr/pass in your maven `settings.xml` instead.

Script files can have any extension but should contain sql code.  Any script file ending in the .gz extension is assumed to be a compressed script and will be decompressed in memory before executing it on the database server.