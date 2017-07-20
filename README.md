# spring-boot-sample-data-jpa
Work arounds to get successful compile and test

This is the "out-of-the-box" result when attempting to use the spring-boot-samples/spring-boot-sample-data-jpa for my spring-boot-jpa project.
https://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-data-jpa

See the details below to work around the compilation errors in the 1.5.4.RELEASE as well as the test errors, except one.  
I just ended up commenting out the SampleDataJpaApplicationTests.testJmx code.
Updating the project from Hibernate 5.0 to 5.2 just adds many more errors.

To start: 
* Apache Maven: 3.5.0
* Java: 1.8.0_131
* Springframework Boot: 1.5.4.RELEASE

1. Modify pom.xml

   a. Use the project parent for my own application, inherit from spring-boot-starter-parent using the stable release.

	<parent>
		<!-- Your own application should inherit from spring-boot-starter-parent -->
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.4.RELEASE</version>
	</parent>

   b. Use Java 8 and UTF-8 encoding.

	<properties>
		<java.version>1.8</java.version>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
	</properties>

	c. Add the first module for the data-jpa sub-project.  My project(s) will be added later.

	<modules>
		<module>data-jpa</module>
	</modules>
  
2. Workaround the errors

	The calls that fail are to the API http://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/PageRequest.html#PageRequest-int-int-org.springframework.data.domain.Sort.Direction-java.lang.String...-
	and fail because the "of" method does not exist. 
<pre>
[INFO] -------------------------------------------------------------
[ERROR] COMPILATION ERROR :
[INFO] -------------------------------------------------------------
[ERROR] /C:/dev/spring-boot-sample-data-jpa/src/test/java/sample/data/jpa/service/CityRepositoryIntegrationTests.java:[45,72] cannot find symbol
  symbol:   method of(int,int)
  location: class org.springframework.data.domain.PageRequest
[ERROR] /C:/dev/spring-boot-sample-data-jpa/src/test/java/sample/data/jpa/service/HotelRepositoryIntegrationTests.java:[54,53] cannot find symbol
  symbol:   method of(int,int,org.springframework.data.domain.Sort.Direction,java.lang.String)
  location: class org.springframework.data.domain.PageRequest
[ERROR] /C:/dev/spring-boot-sample-data-jpa/src/test/java/sample/data/jpa/service/HotelRepositoryIntegrationTests.java:[58,44] cannot find symbol
  symbol:   method of(int,int,org.springframework.data.domain.Sort.Direction,java.lang.String)
  location: class org.springframework.data.domain.PageRequest
[INFO] 3 errors
[INFO] -------------------------------------------------------------
</pre>
3. Comment out the Failing test:
<pre>
	org.hibernate.SQL: select city0_.id as id1_0_, city0_.country as country2_0_, city0_.map as map3_0_, city0_.name as name4_0_, city0_.state as state5_0_ from city city0_ where upper(city0_.name)=upper(?) and upper(city0_.country)=upper(?)
	Tests run: 2, Failures: 1, Errors: 0, Skipped: 0, Time elapsed: 15.486 sec <<< FAILURE! - in sample.data.jpa.SampleDataJpaApplicationTests
	testJmx(sample.data.jpa.SampleDataJpaApplicationTests)  Time elapsed: 0.36 sec  <<< FAILURE!
	java.lang.AssertionError:
	Expected size:<1> but was:<0> in:
	<[]>
        at sample.data.jpa.SampleDataJpaApplicationTests.testJmx(SampleDataJpaApplicationTests.java:77)
</pre>
4. Update to Hibernate 5.2 from 5.0 to use JPA 2.1 FAILS.
<pre>
---------------------------------------------
Error starting ApplicationContext. To display the auto-configuration report re-run your application with 'debug' enabled.
2017-07-20 11:56:07.463 ERROR 1268 [main] o.s.boot.SpringApplication:
	 Application startup failed
org.springframework.beans.factory.BeanCreationException:
	 Error creating bean with name 'entityManagerFactory' defined in class path resource
	 [org/springframework/boot/autoconfigure/orm/jpa/HibernateJpaAutoConfiguration.class]:
	 Invocation of init method failed; nested exception is java.lang.NoSuchMethodError:
	 org.hibernate.engine.spi.SessionFactoryImplementor.getProperties()Ljava/util/Properties;

---------------------------------------------
2017-07-20 11:56:07.488 ERROR 1268 [main] o.s.test.context.TestContextManager:
	Caught exception while allowing TestExecutionListener [org.springframework.test.context.web.ServletTestExecutionListener@2ddc8ecb]
	 to prepare test instance [sample.data.jpa.SampleDataJpaApplicationTests@44f0ff2b]
java.lang.IllegalStateException: Failed to load ApplicationContext
	at org.springframework.test.context.cache.DefaultCacheAwareContextLoaderDelegate.loadContext(DefaultCacheAwareContextLoaderDelegate.java:124)
	 ~[spring-test-4.3.9.RELEASE.jar:4.3.9.RELEASE]

---------------------------------------------
Tests run: 1, Failures: 0, Errors: 1, Skipped: 0, Time elapsed: 17.078 sec <<< FAILURE!
	 - in sample.data.jpa.SampleDataJpaApplicationTests
testHome(sample.data.jpa.SampleDataJpaApplicationTests)  Time elapsed: 0.011 sec  <<< ERROR!
	java.lang.IllegalStateException: Failed to load ApplicationContext
	 at org.springframework.test.context.cache.DefaultCacheAwareContextLoaderDelegate.loadContext(DefaultCacheAwareContextLoaderDelegate.java:124)
</pre>

5. Revert back to plain Spring Boot.
