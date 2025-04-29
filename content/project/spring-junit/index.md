---
title: Effective Automated Testing in Spring
summary: Testing a SpringBoot REST API
tags:
- Spring Boot
- SpringCloudContract

date: "2016-04-27T00:00:00Z"
toc: true

weight: 50

# Optional external URL for project (replaces project detail page).
external_link: ""

image:
  caption:
  focal_point: Smart

---

### Overview

In this project we would be demonstrating automated testing principles using a hotel reservation application built following a microservices architecture using SpringBoot ecosystem. A common industry best practice is to have microservices communicating via JSON over RESTful APIs.

We would be using JUnit5 for writing the tests.

```xml
  <dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
  </dependency>
```
For further application setup support, take a look at the different [JUnit 5 project setup examples](https://github.com/junit-team/junit5-samples) on GitHub.

### RoomService

RoomService is a pretty standard SpringBoot CRUD Microservice.

#### Dependencies

We start with [Spring Initializer](https://start.spring.io/):
- pick spring-boot-starter-parent as v2.4.10,
- choose WAR packaging,
- and add the following dependencies:
  - spring-boot-starter-data-jpa
  - spring-boot-starter-web
  - h2
  - spring-boot-starter-test
  - spring-restdocs-mockmvc

<details>
  <summary>pom.xml</summary>

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
  	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  	<modelVersion>4.0.0</modelVersion>
  	<parent>
  		<groupId>org.springframework.boot</groupId>
  		<artifactId>spring-boot-starter-parent</artifactId>
  		<version>2.4.10</version>
  		<relativePath /> <!-- lookup parent from repository -->
  	</parent>
  	<groupId>host.honeycomb.room</groupId>
  	<artifactId>room-service</artifactId>
  	<version>0.0.1-SNAPSHOT</version>
  	<packaging>war</packaging>
  	<name>room-service</name>
  	<description>Demo project for Spring Boot</description>
  	<properties>
  		<java.version>1.8</java.version>
  	</properties>

  	<distributionManagement>
  		<repository>
  			<id>release</id>
  			<name>releases</name>
  			<url>http://192.168.56.30:8082/artifactory/springboot-junit-libs-release-local</url>
  		</repository>
  		<snapshotRepository>
  			<id>snapshot</id>
  			<name>snapshots</name>
  			<url>http://192.168.56.30:8082/artifactory/springboot-junit-libs-snapshot-local</url>
  		</snapshotRepository>
  	</distributionManagement>

  	<scm>
  		<connection>scm:git@github.com:avijitliberty/springboot-junit.git</connection>
  		<developerConnection>scm:git@github.com:avijitliberty/springboot-junit.git</developerConnection>
  		<url>git@github.com:avijitliberty/springboot-junit.git</url>
  		<tag>HEAD</tag>
  	</scm>

  	<dependencies>
  		<dependency>
  			<groupId>org.springframework.boot</groupId>
  			<artifactId>spring-boot-starter-data-jpa</artifactId>
  		</dependency>
  		<dependency>
  			<groupId>org.springframework.boot</groupId>
  			<artifactId>spring-boot-starter-web</artifactId>
  		</dependency>

  		<dependency>
  			<groupId>com.h2database</groupId>
  			<artifactId>h2</artifactId>
  <!-- 			<scope>runtime</scope> -->
  		</dependency>
  		<dependency>
  			<groupId>org.springframework.boot</groupId>
  			<artifactId>spring-boot-starter-tomcat</artifactId>
  			<scope>provided</scope>
  		</dependency>
  		<dependency>
  			<groupId>org.springframework.boot</groupId>
  			<artifactId>spring-boot-starter-test</artifactId>
  			<scope>test</scope>
  		</dependency>
  		<dependency>
  			<groupId>org.junit.jupiter</groupId>
  			<artifactId>junit-jupiter</artifactId>
  			<scope>test</scope>
  		</dependency>
  		<dependency>
  			<groupId>org.springframework.restdocs</groupId>
  			<artifactId>spring-restdocs-mockmvc</artifactId>
  			<scope>test</scope>
  		</dependency>
  	</dependencies>

  	<build>
  		<plugins>
  			<plugin>
  				<groupId>org.asciidoctor</groupId>
  				<artifactId>asciidoctor-maven-plugin</artifactId>
  				<version>1.5.8</version>
  				<executions>
  					<execution>
  						<id>generate-docs</id>
  						<phase>prepare-package</phase>
  						<goals>
  							<goal>process-asciidoc</goal>
  						</goals>
  						<configuration>
  							<sourceDocumentName>index.adoc</sourceDocumentName>
  							<backend>html</backend>
  							<attributes>
  								<snippets>${project.build.directory}/snippets</snippets>
  							</attributes>
  						</configuration>
  					</execution>
  				</executions>
  				<dependencies>
  					<dependency>
  						<groupId>org.springframework.restdocs</groupId>
  						<artifactId>spring-restdocs-asciidoctor</artifactId>
  						<version>${spring-restdocs.version}</version>
  					</dependency>
  				</dependencies>
  			</plugin>
  			<plugin>
  				<groupId>org.springframework.boot</groupId>
  				<artifactId>spring-boot-maven-plugin</artifactId>
  			</plugin>
  			<plugin>
  				<groupId>org.apache.maven.plugins</groupId>
  				<artifactId>maven-release-plugin</artifactId>
  				<version>2.5.1</version>
  				<configuration>
  					<tagNameFormat>v@{project.version}</tagNameFormat>
  					<autoVersionSubmodules>true</autoVersionSubmodules>
  				</configuration>
  			</plugin>
  		</plugins>
  	</build>

  </project>
  ```

</details>

#### Application Code

We have the domain entity Room.java which has the basic attributes describing a hotel room and we have annotated the class and attributes with JPA annotations to persist the entity in a database.

<details>
  <summary>Room.java</summary>

```java

@Entity
@Table(name="rooms")
public class Room {
	@Id
	@GeneratedValue
	private long id;
	@Column(name="room_number")
	private String roomNumber;
	@Column(name="weekday_price")
	private double weekdayPrice;
	@Column(name="weekend_price")
	private double weekendPrice;
	@Column(name="room_type")
	private String roomType;
	@Column(name="floor")
	private String floor;

	// Getters and Setters
}

```
</details>

In the repository we are using the springframework.data.repository.CrudRepository for handling the database persistence with standard CRUD operations.

<details>
  <summary>RoomRepo.java</summary>

```java
public interface RoomRepo extends CrudRepository<Room, Long> {
	Room findByRoomNumber(String anyString);
}
```
</details>

We have the RoomController which exposes a standard RESTful API:

<details>
  <summary>RoomController.java</summary>

```java
@RestController
@RequestMapping("/rooms")
public class RoomController {

	private RoomService service;

	public RoomController(RoomService service) {
		this.service = service;
	}

	@GetMapping
	public ResponseEntity<Iterable<Room>> getAllRooms(){
		return ResponseEntity.ok(service.getAllRooms());
	}

	@GetMapping("/{id}")
	public ResponseEntity<Optional<Room>> findRoomById(long id){
		return ResponseEntity.ok(service.findRoom(id));
	}

	@PostMapping
	public ResponseEntity<?> addRoom(Room room){
		return ResponseEntity.ok().build();
	}

	@PutMapping("/{id}")
	public ResponseEntity<?> updateRoom(Room room){
		return ResponseEntity.ok().build();
	}

	@DeleteMapping("/{id}")
	public ResponseEntity<?> deleteRoom(long id){
		return ResponseEntity.ok().build();
	}
}

```
</details>

As is pretty standard we have the Service class which basically passes on the CRUD actions from the Controller to the RoomRepo:

<details>
  <summary>RoomServiceImpl.java</summary>

```java
@Service
public class RoomServiceImpl implements RoomService {
	private RoomRepo repo;


	public RoomServiceImpl(RoomRepo repo) {
		this.repo = repo;
	}

	@Override
	public Iterable<Room> getAllRooms() {
		return repo.findAll();
	}

	@Override
	public Optional<Room> findRoom(long roomId) {
		return repo.findById(roomId);
	}

	@Override
	public void updateRoom(Room room) {
		repo.save(room);
	}

	@Override
	public void addRoom(Room room) {
		repo.save(room);
	}

	@Override
	public Room findByRoomNumber(String roomNumber) {
		if (!StringUtils.isNullOrEmpty(roomNumber) && StringUtils.isNumber(roomNumber)){
			Room room = repo.findByRoomNumber(roomNumber);
			if (room == null) {
				throw new RoomServiceClientException("Room number: " + roomNumber + ", does not exist.");
			}
			return room;
		}
		else {
			throw new RoomServiceClientException("Room number: " + roomNumber + ", is an invalid room number format.");
		}
	}
}
```
</details>

#### Unit Testing

To unit test the RoomServiceImpl we would create the TestRoomServiceImpl and mock the RoomRepo using the Mockito library. We would simulate the behavior of the RoomRepo.findByRoomNumber() method call for the different test cases and asserting expected behavior.

<details>
  <summary>TestRoomServiceImpl.java</summary>

```java
public class TestRoomServiceImpl {

	/*
	 * Test the Happy Path i.e RoomServiceImpl.findByRoomNumber() works as
	 * expected. We mock the RoomRepo.findByRoomNumber() method to return an empty
	 * room given any string as input and assert that the room is NotNull.
	 */

	@Test
	public void lookupExistingRoom() {
		RoomRepo mockRepo = mock(RoomRepo.class);
		when(mockRepo.findByRoomNumber(anyString())).thenReturn(new Room());
		RoomService service = new RoomServiceImpl(mockRepo);

		Room room = service.findByRoomNumber("100");

		assertNotNull(room);
	}

	/*
	 * Here we test exception is thrown if there was no room found for a given
	 * roomNumber. We mock the RoomRepo.findByRoomNumber() method to return null
	 * given any string as input and assert the expected Exception is thrown.
	 */
	@Test
	public void throwExceptionForNonExistingRoom() {
		RoomRepo mockRepo = mock(RoomRepo.class);
		when(mockRepo.findByRoomNumber(anyString())).thenReturn(null);
		RoomService service = new RoomServiceImpl(mockRepo);
		try {
			service.findByRoomNumber("100");
			fail("Exception should had been thrown");
		} catch (Exception e) {
			assertEquals("Room number: 100, does not exist.", e.getMessage());
		}
	}

	/*
	 * Here we test exception is thrown given a malformed roomNumber. Given any
	 * malformed string as input we assert that the expected Exception is thrown.
	 */
	@Test
	public void throwExceptionInvalidRoomNumberFormat() {
		RoomRepo mockRepo = mock(RoomRepo.class);
		RoomService service = new RoomServiceImpl(mockRepo);
		try {
			service.findByRoomNumber("BAD ROOM NUMBER!");
			fail("Exception should had been thrown");
		} catch (Exception e) {
			assertEquals("Room number: BAD ROOM NUMBER!, is an invalid room number format.", e.getMessage());
		}
	}

	/*
	 * Here we test exception is thrown given a null roomNumber. Given null input we
	 * assert that the expected Exception is thrown.
	 */
	@Test
	public void throwExceptionInvalidRoomNumberNull() {
		RoomRepo mockRepo = mock(RoomRepo.class);
		RoomService service = new RoomServiceImpl(mockRepo);
		try {
			service.findByRoomNumber(null);
			fail("Exception should had been thrown");
		} catch (Exception e) {
			assertEquals("Room number: null, is an invalid room number format.", e.getMessage());
		}
	}

	/*
	 * Here we test exception is thrown given a null roomNumber. Given -ve number input we
	 * assert that the expected Exception is thrown.
	 */
	@Test
	public void throwExceptionInvalidRoomNumberNegative() {
		RoomRepo mockRepo = mock(RoomRepo.class);
		RoomService service = new RoomServiceImpl(mockRepo);
		try {
			service.findByRoomNumber("-100");
			fail("Exception should had been thrown");
		} catch (Exception e) {
			assertEquals("Room number: -100, is an invalid room number format.", e.getMessage());
		}
	}

}
```
</details>

#### Component Integration Testing

##### Testing the Web Layer

A unit test has limited scope and tests your code separately from other collaborators. Unit tests should not involve any external dependencies directly. Examples of external dependencies are databases, message brokers, and web services.

Since well-written unit tests run in isolation, we require a mechanism for emulating collaborators. This can be achieved by using mock objects.

A mock object implements the interface of the real object but provides only enough code to simulate its behavior. This is acceptable in unit tests since we are not testing the collaborator, only that our code is calling its methods correctly and receiving the expected response.

However, some objects depend on the infrastructure to function. This is especially true of web MVC applications that require a Tomcat or other application server. This can be expensive for unit testing because of the overhead associated with starting and instantiating the various tiers of the infrastructure. For Spring applications, the Spring Test Framework provides us with options to help you write unit tests in these cases.

MockMvc is one such option. MockMvc is a utility class that gives you the ability to send mock HTTP servlet requests in a simulated MVC environment. This gives us the ability to test MVC applications without incurring the cost of instantiating an application server, In this example, we will demonstrate how to write unit tests for a Spring Boot MVC application using MockMVC.

##### Testing the Data Layer

In the **Customers** class we have added some JPA annotations to retrieve them from a persistent storage.

##### Security Testing

##### Testing JSON
