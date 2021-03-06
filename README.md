# Building Restful Webservices

Banuprakash C

Full Stack Architect,

Co-founder Lucida Technologies Pvt Ltd.,

Corporate Trainer,

Email: banuprakashc@yahoo.co.in

https://www.linkedin.com/in/banu-prakash-50416019/


https://github.com/BanuPrakash/RESTFUL

===================================

Softwares Required:
1) Java 8+
	https://www.oracle.com/in/java/technologies/javase/javase-jdk8-downloads.html

2) Eclipse for JEE  
	https://www.eclipse.org/downloads/packages/release/2020-03/m1/eclipse-ide-enterprise-java-developers

3) MySQL  [ Prefer on Docker]

Install Docker Desktop

Docker steps:

a) docker pull mysql

b) docker run --name local-mysql –p 3306:3306 -e MYSQL_ROOT_PASSWORD=Welcome123 -d mysql

container name given here is "local-mysql"

For Mac:
docker run -p 3306:3306 -d --name local-mysql -e MYSQL_ROOT_PASSWORD=Welcome123 mysql


c) CONNECT TO A MYSQL RUNNING CONTAINER:

$ docker exec -t -i <container_name> /bin/bash

d) Run MySQL client:

bash terminal> mysql -u "root" -p

mysql> exit

=======================================

SOLID Design Principles

D ==> Dependency Injection ==> Inversion Of Control

Container ==> Layer on top of JVM with service providers

Web Container / Servlet engine ==> DI only for web related ==> request and response, servletcontext
EJB Container ==> SessionBean, EntityBean, MessageBean

Spring Container ==> Lifecycle of bean and wiring
	lightweight container for building enterprise application ==> provides DI as core module
	Any object managed by spring container is termed as "bean"

Guice
Play Framework

============

	Spring Container:
	Metadata ==> XML

	interface EmployeeDao {
		void addEmployee(Employee e);
	}

	public class EmployeeDaoJdbcImpl implements EmployeeDao {
			public void addEmployee(Employee e) {//}
	}

	public class SampleService {
		private EmployeeDao empDao;

		public void setEmployeeDao(EmployeeDao ed) { this.empDao = ed; }

		public void insert(Employee e) {
			empDao.addEmployee(e);
		}
	}

	beans.xml

	<beans>
    <bean id="jdbcimpl" class ="pkg.EmployeeDaoJdbcImpl" />
    <bean id="service" class ="pkg.SampleService">
      <property name="employeeDao" ref="jdbcimpl" />
   	</bean>
 	</beans>

 	new ClassPathXmlApplicationContext("beans.xml");
 	OR
	new FilePathXmlApplicationContext("/users/data/a.xml")

===================================================

Spring Annotations at class level:

1) @Component
==> Utility classes / helpers 
2) @Repository
==> DAO
https://github.com/spring-projects/spring-framework/blob/main/spring-jdbc/src/main/resources/org/springframework/jdbc/support/sql-error-codes.xml
3) @Service
4) @Controller
5) @RestController
6) @Configuration

============


	interface EmployeeDao {
		void addEmployee(Employee e);
	}

	@Repository
	public class EmployeeDaoJdbcImpl implements EmployeeDao {
			public void addEmployee(Employee e) {//}
	}

	@Service
	public class SampleService {
		@Autowired
		private EmployeeDao empDao;
 
 		public void insert(Employee e) {
			empDao.addEmployee(e);
		}
	}

Spring containers creates
*  "employeeDaoJdbcImpl" instance of "EmployeeDaoJdbcImpl"
* "sampleService" instance of "SampleService"

@Service("myservice")

========

@Autowired
private EmployeeDao empDao;
OR
@Inject
private EmployeeDao empDao;

@Autowired uses internally libraries like [ CGLib / JavaAssist / ByteBuddy] for bytecode instrumentation
EmployeeDao empDao = serviceProvider.getInstance(EmployeeDao.class);
OR
EmployeeDao empDao = new EmployeeDaoJdbcImpl();



try {
	...
} catch(SQLException ex) {
	ex.getErrorCode()
}

=============

Eclipse ==> Help ==> Eclipse Market Place ==> Search for "STS" ==> Install Spring tools 4 

=========================================


Spring Boot?
Spring framework with some pre-configuration done
==> highly Opiniated Framework
* for Database related ==> configures HikariCP as database connection pool [ C3p0/ DriverManagerDataSource]
* Tomcat as embedded Servlet Container for web application
* Hibernate as ORM framework

==> makes application container ready [ Docker]
https://spring.io/guides/gs/spring-boot-docker/

Dockerfile

FROM openjdk:8-jdk-alpine
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]


Without spring boot:

FROM openjdk:8-jdk-alpine
FROM tomcat:tomcat-9
FROM maven:..

mvn clean intall
mvn tomcat:run

=====================

This line creates spring container:
SpringApplication.run(DemoApplication.class, args);

in Spring framework

ApplicationContext ctx  = new ClassPathXmlApplicationContext("beans.xml");
OR
ApplicationContext ctx  = new AnnotationConfigApplicationContext();

* SpringApplication.run(DemoApplication.class, args); uses @SpringBootApplication

@SpringBootApplication is 3 in one:
1) @ComponentScan
	scans for spring components having any of the above "6" mentioned annotations from "com.adobe.demo" and
	sub-packages [com.adobe.demo.service] and creates bean

	==> @ComponentScan(basePackages="{....}")
2) @EnableAutoConfiguration
	==> spring boot is highly opiniated and creates many things out of box [ Connection Pool / Tomcat/ ORM / 3rd party
	Jackson] if these are present in the for "jar" libraries 

	@EnableAutoConfiguration(exclude={HibernateJPAVendor.class})
	@EnableAutoConfiguration(excludeName={"nameOfBean"})

3) @Configuration class

=============================

BeanFactory and ApplicationContext are interfaces to access Spring Factory

================
* To resolve multiple beans implementing an interface
1) use @Primary on one of the bean

@Repository
@Primary
public class EmployeeDaoJpaImpl implements EmployeeDao {

	@Override
	public void addEmployee() {
		System.out.println("added using JPA!!!");
	}

}

package com.adobe.demo.dao;

import org.springframework.context.annotation.Primary;
import org.springframework.stereotype.Repository;

@Repository
public class EmployeeDaoMongoImpl implements EmployeeDao {

	@Override
	public void addEmployee() {
		System.out.println("mongo store!!!");
	}

}

----------

2) using @Qualifier [ Remove @Primary]

@Service
public class SampleService {
	@Autowired
	@Qualifier("employeeDaoMongoImpl")
	private EmployeeDao empDao;
	
	public void insertEmployee() {
		empDao.addEmployee();
	}
}


@Service
public class OtherService {
	@Autowired
	@Qualifier("employeeDaoJdbcImpl")
	private EmployeeDao empDao;
	
	public void insertEmployee() {
		empDao.addEmployee();
	}
}

===============
3) using @Profile
@Repository
@Profile("dev")
public class EmployeeDaoMongoImpl implements EmployeeDao {

	@Override
	public void addEmployee() {
		System.out.println("mongo store!!!");
	}

}


@Repository
@Profile("prod")
public class EmployeeDaoJpaImpl implements EmployeeDao {

	@Override
	public void addEmployee() {
		System.out.println("added using JPA!!!");
	}
}


3.1) 
Program Arguments:
--spring.profiles.active=dev

3.2) 
application.properties
spring.profiles.active=prod

Command Line arguments [program arguments] ==> system.properties => application.properties


3.2)

application.properties
dao=mongo

@Repository
//@Profile("dev")
@ConditionalOnProperty(name = "dao", havingValue = "mongo")
public class EmployeeDaoMongoImpl implements EmployeeDao {

	@Override
	public void addEmployee() {
		System.out.println("mongo store!!!");
	}

}

=================

@ConditionalOnMissingBean("employeeDaoMongoImpl")

===========================================================

* Spring creates instances using default constructor

Fails to create the instance of below class:
@Component
public class Example {
	public Example(String name) {

	}
}


* We need Spring container to manage bean of classes which are programatically created using "new" keyword
==> 3rd party apis which doesn't have @component, @Repository, ..

@Configuration
public class MyConfig {

@Bean(name="c3p0")
public DataSource getDataSource() {
	ComboPooledDataSource cpds = new ComboPooledDataSource();
	cpds.setDriverClass( "org.postgresql.Driver" ); //loads the jdbc driver            
	cpds.setJdbcUrl( "jdbc:postgresql://localhost/testdb" );
	cpds.setUser("swaldman");                                  
	cpds.setPassword("test-password");                                  
	cpds.setMinPoolSize(5);                                     
	cpds.setAcquireIncrement(5);
	cpds.setMaxPoolSize(20);
	return cpds;
}

}

@Service
class SampleService {
	@Autowired
	DataSource ds;
}

=============


Day 2

JPA using Spring Data JPA

JPA ==> Java Persistence API => not a part of Spring ==> It's a specification to use ORM over RDBMS.

ORM ==> Object Relational Mapping ==> Framework over JDBC to interact with RDBMS

JDBC ==> integration API [ java ] ==> Impedence mismatch
	Java ==> Employee and Address 
*	single table
	employee_id | first_name | street | city | zip ...

* Share PK
	employee table
	employee_id | first_name 

	address table
	employee_id | street | city | zip

* FK
	employee table
	employee_id | first_name 

	address table
	address_id | street | city | zip | employee_id

=====================

@Configuration
public class MyConfiguration {

	@Bean
	public DataSource getDataSource() {
		ComboPooledDataSource cpds = new ComboPooledDataSource();
		cpds.setDriverClass( "org.postgresql.Driver" ); //loads the jdbc driver            
		cpds.setJdbcUrl( "jdbc:postgresql://localhost/testdb" );
		cpds.setUser("swaldman");                                  
		cpds.setPassword("test-password");                                  
	
		cpds.setMinPoolSize(5);                                     
		cpds.setAcquireIncrement(5);
		cpds.setMaxPoolSize(20);
		return cpds;
	}

	@Bean
	public EntityManagerFactory getEmf(DataSource ds) {
		LocalContainerEntityMaangerFactoryBean emf = new LocalContainerEntityMaangerFactoryBean();
			emf.setJPAVendor(new HibernateJPAVendor());
			<!-- emf.setDataSource(getDataSource()); -->
			emf.setDataSource(ds);
			emf.setPackagesToScan("com.adobe.prj.entity");
			...
		return emf;
	}
}


public interface ProductDao {
	void addProduct(Product p);
	Product getProduct(int id);
}

@Repository
public class ProductDaoJPAImpl implements ProductDao {
	@PersitenceContext
	private EntityManager em;

	public void addProduct(Product p) {
		em.save(p);
	}

	public Product getProduct(int id) {
		return em.findById(id);
	}
}

=========================

Spring Data JPA simplifies using JPA

* No need to write @Repository classes
* just create interface extends JPARepository

public interface ProductDao extends JPARepository<Product, Integer> {
}

* you get methods for save(), findById, findAll(), pagination , sorting, limit, delete
* can add additional methods 
* Spring uses Byte Code Instrumentation libraries to generate classes for these interface


* JPARepository ==> PageAndSortingRepository ==> CrudRepository

================

Build New Spring boot application with "MySQL", "jpa" and "web" dependencies

=============

@Embeddable
public class UserPK implements Serializable {
	private String firstName;
	private String lastName;
	private String middleName;
	//
}


@Table(name="users")
@Entity
public class User {
	@EmbeddedId
	private UserPK userPK;
	..
}

==============

* spring.jpa.hibernate.ddl-auto=update

	==> DDL ==> Data Definition Language ==> CReaTE, ALTER and DROP
	update ==> map to existing table if table exist; else create table; if required alter table to match mapping
	TOP to BottOM approach
	create ==> drop and create tables for every application run ==> good for testing
	verify ==> map to existing tables; no alter on tables; if mapping fails ==> application fails
		Bottom to TOP approach

spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

* em.save() or em.findById(4); ==> check if appropriate SQL is generated


spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

* Generate SQL for MySQL8


==


docker exec -it local-mysql bash

 # mysql -u "root" -p
Enter password:Welcome123

use ADOBE_JPA

 select * from products;

insert into products  (id, name, price, category, quantity) values (0, 'Samsung Z fold', 130000.00 , 'mobile', 100);
insert into products  (id, name, price, category, quantity) values (0, 'Sony Bravia', 95000.00 , 'tv',100);
insert into products  (id, name, price, category, quantity) values (0, 'Onida Thunder', 3500.00 , 'tv', 100);
insert into products  (id, name, price, category, quantity) values (0, 'Logitech Mouse', 600.00 , 'computer', 100);
insert into products  (id, name, price, category, quantity) values (0, 'Lamp', 900.00 , 'electrical', 100);


===============

@Query(value= "select * from products where price >= :low and price <= :high", nativeQuery=true)

=======================================

REST ==> Representational State Transfer
* Resource on server [ database / noSQL/ files / printer]
* state of the resource is served to clients in various representation [ XML / JSON / CSV / ATOM ]

Characterstics of REST
1) client - server logic seperation
2) Stateless [ Server should not have client session details ==> No Cookies / HttpSession concepts]
3) Cacheable [ client side [ Cache-Control, Expires, ETag] or middle tier [Redis]]
4) Uniform Resource

Resource URI should be plural nouns and actions are done using HTTP verbs

Resource: http://server/api/v1/products

* 1)
GET:
http://server/api/v1/products

get all products representation based on "accept" HTTP header

2) 

GET:
http://server/api/v1/products/5

extra path param is to identify based on PK or sub-resources

--> get product represetntation of product whose id is 5

http://server/api/v1/customers/banu@gmail.com/orders

3)

http://server/api/v1/products?page=3&size=10
http://server/api/v1/products?lower=1000&higher=10000

Query param for filtered response

============

4) 
POST
http://server/api/v1/products

"content-type" header is used to specify what type of representation is sent to server

===

5) 
DELETE
http://server/api/v1/products/5

6)
PUT
http://server/api/v1/products/5

"content-type" header is used to specify what type of representation is sent to server which needs to be used
to modify product whose id is "5"

7) PATCH
http://server/api/v1/products/5

for partial update of product

8) JSON-PATCH+PATCH
"content-type" : "application/json-patch+patch"
http://server/api/v1/products/5

payload:

{
	"op" : "replace",
	"path" "/price",
	"value" : 5555.22
}

==

 op can be "add", "replace", "remove"


=========================

GET and DELETE can't have payload ==> IDEMPOTENT

POST, PUT, PATCH can have payload ==> NOT IDEMPOTENT

===============================================


@RestController
@RequestMapping("/api/products")
public class ProductController {

	@GetMapping()
	m1() {

	}

	@PostMapping()
	m2() {

	}
}


==

@RestController
@RequestMapping("/api/customers)
public class ProductController {

	@GetMapping()
	m1() {

	}

	@PostMapping()
	m2() {

	}
}


@PostMapping()
@ResponseStatus(HttpStatus.CREATED)
	public  @ResponseBody Product addProduct(@RequestBody Product p) {
		Product prd = service.saveProduct(p);
		return prd;
	}
		
=========

Browser:
GET
http://localhost:8080/api/products
GET using path param @PathVariable
http://localhost:8080/api/products/4
GET by Query param @RequestParam
http://localhost:8080/api/products?low=1000&high=100000



=========
Download POSTMAN

POSTMAN for POST, PUT and DELETE testing

================================================


