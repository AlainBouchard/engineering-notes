+++
title = "Java Spring Boot 2"
LastModifierDisplayName = "Alain Bouchard"
LastModifierEmail = "alain.bouchard@appdirect.com"
disableToc = "false"
+++

- [Java Spring Boot 2](#java-spring-boot-2)
- [Terminology](#terminology)
- [Getting Started](#getting-started)
  - [Why Spring Boot?](#why-spring-boot)
    - [Key Aspects](#key-aspects)
    - [Auto Config](#auto-config)
  - [Spring Initializr](#spring-initializr)
  - [Inversion of Control](#inversion-of-control)
    - [Spring IoC](#spring-ioc)
  - [Proxies](#proxies)
- [Data Access in Spring](#data-access-in-spring)
  - [Spring Data](#spring-data)
    - [Benefits of Spring Data](#benefits-of-spring-data)
    - [Key Components](#key-components)
  - [Embeedded DB with Spring Boot](#embeedded-db-with-spring-boot)
  - [Repository with Spring Data](#repository-with-spring-data)
  - [Using remote database](#using-remote-database)
- [Service Tier](#service-tier)
  - [Utilizing IoC](#utilizing-ioc)
  - [Service abstraction](#service-abstraction)
  - [Spring Service Object](#spring-service-object)
- [Web pages with Spring](#web-pages-with-spring)
  - [Controller](#controller)

## Java Spring Boot 2

## Terminology

- POJO : Plain Old Java Object (may have more that setters/getters in Spring world)
- Java Beans : Simple objects with only setters/getters
- Spring Beans : POJOs confiugered in the application context
- DTO : Data Transfer Objects are Java Beans used to move state between layers
- IOC : Inversion Of Control
  - IoC provides mechanism of dependency injection
  - *Application Context* wraps the *Bean Factory* which serves the beans at the runtime of the application
  - Spring Boot provides auto-configuration of the *Application Context*

## Getting Started

### Why Spring Boot?

- Support rapid development
- Remove boilerplate of application setup
- Many uses
- *Cloud Native* support but also traditional

#### Key Aspects

- Embedded tomacat (or others)
- Auto-configuration of *Application Context*
- Automatic Servlet Mappings
- Database support and Hibermate/JPA dialect
- Automatic Controller Mappings

#### Auto Config

- Default opiniated configuration
- Very to override defaults
- Configuration on presence

### Spring Initializr

1. start.spring.io
1. Spring Boot: pick latest released version (ie. 2.5.6)
    1. Packaging: Jar
1. Add Dependencies:
    1. Pring Web
    1. TBD...
1. Generate

Now it can build and run as is:

> java -jar target/xyz-0.0.1-SNAPSHOT.jar

Use Chrome: `localhost:8080`

### Inversion of Control

- Container mainains your class dependencies
- Objects injected at runtime or startup time
- An object accepts the dependencies for construction instead of constructing them

#### Spring IoC

- Bean Factory
- Application Context
- References
- Analysis of construction order

### Proxies

- Beans in Bean Faactory are proxied
- Annitations drive proxies
- Annitations are easy extension points, for your own abstracts too
- Method calling order matters

## Data Access in Spring

### Spring Data

- Provides a common set of interfaces
- Provides a common naming convention
- Provides aspected behavior
- Provides Repository and Data Mapping convention

#### Benefits of Spring Data

- Remove boilerplate code
- Allows for swapping datasources easier
- Allows to focus on buisiness logic

#### Key Components

- Repository Interface
- Entity Object
- DataSource no accessed directly

### Embeedded DB with Spring Boot

Needed dependencies:

```properties
org.springframework.boot:spring-boot-starter-data-jpa
com.h2database:h2
```

Set application.properties:

- logging.level.org.springframework.jdbc.datasource.init.ScriptUtils=debug : by default is set to info
- spring.jpa.hibernate.ddl-auto=none : don't create schema, and just connect to DB

### Repository with Spring Data

- Java Persistence API (or [JPA])
  - Mapping Java objects to DB tables and vice versa is called Object-relational Mapping (ORM)
  - JPA permits the developer to works directly with objects rather tahn with SQL statements
  - Based on Annotations

- [Entity]
  - A class which should be persisted in a database it must be annotated with `javax.persistence.Entity`
  - JPA uses a database table for every entity
  - Persisted instances of the class will be represented as one row in the table
  - JPA allows to auto-generate the primary key in the database via the `@GeneratedValue` annotation
  - By default, the table name corresponds to the class name. You can change this with the addition to the annotation `@Table(name="NEWTABLENAME")`

<details>
    <summary>Code Example</summary>

```java
@Entity
@Table(name="ROOM")
public class Room {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name="ROOM_ID")
    private long id;
    @Column(name="NAME")
    private String name;
    @Column(name="ROOM_NUMBER")
    private String roomNumber;
    @Column(name="BED_INFO")
    private String bedInfo;

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getRoomNumber() {
        return roomNumber;
    }

    public void setRoomNumber(String roomNumber) {
        this.roomNumber = roomNumber;
    }

    public String getBedInfo() {
        return bedInfo;
    }

    public void setBedInfo(String bedInfo) {
        this.bedInfo = bedInfo;
    }

    @Override
    public String toString() {
        return "Room{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", roomNumber='" + roomNumber + '\'' +
                ", bedInfo='" + bedInfo + '\'' +
                '}';
    }
}
```

</details>

- Create a [CrudRepository] interface for the created [Entity]:

<details>
    <summary>Code Example</summary>

```java
@Repository
public interface RoomRepository extends CrudRepository<Room, Long> {
    // Room is the Entity class
    // Long is the ID type
}
```

</details>

- Create a [Component] Event:

  - A `@Component` Annotation is automatically picked by Spring

<details>
    <summary>Code Example</summary>

```java
@Component
public class AppStartupEvent implements ApplicationListener<ApplicationReadyEvent> {
    private final RoomRepository roomRepository;

    public AppStartupEvent(RoomRepository roomRepository) {
        this.roomRepository = roomRepository;
    }

    @Override
    public void onApplicationEvent(ApplicationReadyEvent event) {
        Iterable<Room> rooms = this.roomRepository.findAll();
        rooms.forEach(System.out::println);
    }
}
```
</details>

### Using remote database

Replace the H2 database for an other database, example a PostgreSQL:

```gradle
dependencies:
org.postgresql:postgresql
```

In `application.properties`:

```properties
spring.jpa.database=postgresql
spring.datasource.url=jdbc:postgresql://localhost:5432/dev
spring.datasource.username=postgres
spring.datasource.password=postgres
```

## Service Tier

### Utilizing IoC

Why use IoC?

- Allows you to focus on contracts
- Develoip business code only, leave constuction to the container
- Build intermediate abstractions
- Produce clean code

Srping and IoC:

- IoC container is configured by developer
- Spring maintains handles to objects constucted at startup
- Spring serves singletons to classes during construction
- Spring maintains lifecycle of beans
- Developer only accesses the application context

### Service abstraction

Why building Service Abstractions:

- Encapsulate layers?
- Abstract 3rd partys APIs
- Simplify implementations
- Swap out implementations as runtime (ie. factory pattern)

How to build one?

- Define our interface (or class)
- Create the API
- Inject the dependencies
- Annotate or configure (classes)
- Code the implemantation

### Spring Service Object

- We mark beans with [@Service] to indicate that they're holding the business logic. Besides being used in the service layer, there isn't any other special use for this annotation.
- Starting with Spring 2.5, the framework introduced annotations-driven Dependency Injection. The main annotation of this feature is [@Autowired]. It allows Spring to resolve and inject collaborating beans into our bean.
  - Using `@Autowired` or either properties or setters/getters isn't a good practice or easy to test;
  - Use final properties with constructors to have immutable object. If more than one constructor is defined then using `@Autowired` on the default one will make Spring to use it.

<details>
    <summary>Code Example</summary>

```java
@Service
public class ReservationService {
    private final RoomRepository roomRepository;
    private final GuestRepository guestRepository;
    private final ReservationRepository reservationRepository;

    @Autowired // optional if only one constructor
    public ReservationService(RoomRepository roomRepository, GuestRepository guestRepository, ReservationRepository reservationRepository) {
        this.roomRepository = roomRepository;
        this.guestRepository = guestRepository;
        this.reservationRepository = reservationRepository;
    }

    // Business logic here...
}
```
</details>

## Web pages with Spring

### Controller

Model View Controller (or MVC)

- Fundamental pattern for Web application development
- The `Model` is the data
- The `View` is the visual display that is populated
- The `Controller` wires the view with the model

Spring Controller

- Spring bean
- Annotated for the [servlet] mapping
- Responds to incoming web requests
- Output a view or raw data

Template Engines

- Spring supports several
- Thymeleaf most popular
- Provides a DSL for HTML leaving raw html documents
- Placeholders for dynamic data
- Rendiring engin allows for final products

<details>
    <summary>Code Example</summary>

```java
@Controller
@RequestMapping("/reservations")
public class RoomReservationController {

    private final DateUtils dateUtils;
    private final ReservationService reservationService;

    public RoomReservationController(DateUtils dateUtils, ReservationService reservationService) {
        this.dateUtils = dateUtils;
        this.reservationService = reservationService;
    }

    @RequestMapping(method = RequestMethod.GET)
    public String getReservations(@RequestParam(value="date", required=false) String dateString, Model model){
        Date date = this.dateUtils.createDateFromDateString(dateString);
        List<RoomReservation> roomReservations = this.reservationService.getRoomReservationsForDate(date);
        model.addAttribute("roomReservations", roomReservations);
        return "roomres";
    }
}
```

</details>

Can use [Thymeleaf] to create HTML pages.

- Add the web page to `src/main/resources/templates`





[@Autowired]: https://www.baeldung.com/spring-autowire
[Component]: https://www.baeldung.com/spring-component-annotation#component
[CrudRepository]: https://www.baeldung.com/spring-data-crud-repository-save
[Entity]: https://www.vogella.com/tutorials/JavaPersistenceAPI/article.html#jpaintro_entity
[JPA]: https://www.vogella.com/tutorials/JavaPersistenceAPI/article.html#jpaintro
[servlet]: https://www.baeldung.com/intro-to-servlets
[@Service]: https://www.baeldung.com/spring-component-repository-service#service
[Thymeleaf]: https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html