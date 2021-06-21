# spring-boot-h2-crud

This project explains CRUD (**C**reate, **R**ead, **U**pdate, **D**elete) operations using spring boot and H2 in-memory database.
In this app we are using Spring Data JPA for built-in methods to do CRUD operations.     
`@EnableJpaRepositories` annotation is used on main class to Enable H2 DB related configuration, which will read properties from `application.properties` file.


## Prerequisites 
- Java
- [Spring Boot](https://spring.io/projects/spring-boot)
- [Maven](https://maven.apache.org/guides/index.html)
- [H2 Database](https://www.h2database.com/html/main.html)
- [Lombok](https://objectcomputing.com/resources/publications/sett/january-2010-reducing-boilerplate-code-with-project-lombok)


## Tools
- Eclipse or IntelliJ IDEA (or any preferred IDE) with embedded Maven
- Maven (version >= 3.6.0)
- Postman (or any RESTful API testing tool)


<br/>


###  Build and Run application
_GOTO >_ **~/absolute-path-to-directory/spring-boot-h2-crud**  
and try below command in terminal
> **```mvn spring-boot:run```** it will run application as spring boot application

or
> **```mvn clean install```** it will build application and create **jar** file under target directory 

Run jar file from below path with given command
> **```java -jar ~/path-to-spring-boot-h2-crud/target/spring-boot-h2-crud-0.0.1-SNAPSHOT.jar```**

Or
> run main method from `SpringBootH2CRUDApplication.java` as spring boot application.  


||
|  ---------    |
| **_Note_** : In `SpringBootH2CRUDApplication.java` class we have autowired Contact repositories. <br/>If there is no record present in DB for any one of that module class - Contact , static data is getting inserted in DB from `HelperUtil.java` class when we are starting the app for the first time.| 



### Code Snippets
1. #### Maven Dependencies
    Need to add below dependencies to enable H2 DB related config in **pom.xml**. Lombok's dependency is to get rid of boiler-plate code.   
    ```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
   
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
   
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    ```
    
   
2. #### Properties file
    Reading H2 DB related properties from **application.properties** file and configuring JPA connection factory for H2 database.  

    **src/main/resources/application.properties**
     ```
     server.port=8088
    
     spring.datasource.url=jdbc:h2:mem:sampledb
     spring.datasource.driverClassName=org.h2.Driver
     spring.datasource.username=sa
     spring.datasource.password=password
     spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
    
     spring.h2.console.enabled=true
    
     #spring.data.rest.base-path=/phone
     spring.data.rest.base-default-page-size=10
     spring.data.rest.base-max-page-size=20
    
     springdoc.version=1.0.0
     springdoc.swagger-ui.path=/swagger-ui-custom.html 
     ```
   
   
3. #### Model class
    Below are the model classes which we will store in H2 DB and perform CRUD operations.  
        
    **Contact.java**  
    
    ```
    @Getter
	@Setter
	@NoArgsConstructor
	@AllArgsConstructor
	@Builder
	@Entity
	@Table
	@Component
	public class Contact implements Serializable {
	
	    @Id
	    @GeneratedValue
	    @JsonIgnore
	    private int id;
	    
	    @JsonManagedReference
	    //@JsonBackReference
	    @OneToOne(cascade=CascadeType.ALL)
	    @JoinColumn(name="name")
	    private Name name;
	
	    @JsonManagedReference
	    //@JsonBackReference
	    @OneToOne(cascade=CascadeType.ALL)
	    @JoinColumn(name="address")
	    private Address address;
	
	
	    @JsonManagedReference
	    //@JsonBackReference
	    @OneToMany(fetch = FetchType.LAZY, mappedBy = "contact", cascade = { CascadeType.ALL},orphanRemoval = true)
	    private List<PhoneNumber> phoneNumbers;
	
	    private String email;
    }
    ```
    
    **Name.java**
    
    ```
   
	@Getter
	@Setter
	@NoArgsConstructor
	@AllArgsConstructor
	@Builder
	@Entity
	@Table
	@Component
	public class Name {
	
		@Id
	    @GeneratedValue
	    @JsonIgnore
	    private int id;
	
		@Column(name = "first_name")
	    private String firstName;
		
		@Column(name = "middle_name")
	    private String middleName;
		
	    @Column(name = "last_name")
	    private String lastName;
	    
	    @JsonBackReference
	    @OneToOne(mappedBy="name", cascade=CascadeType.ALL,orphanRemoval = true)
	    private Contact contact;
	}
    ```
   
    **Address.java**
    
    ```
   
	@Getter
	@Setter
	@NoArgsConstructor
	@AllArgsConstructor
	@Builder
	@Entity
	@Table
	@Component
	public class Address implements Serializable {
	
	    @Id
	    @GeneratedValue
	    @JsonIgnore
	    private int id;
	
	    @Column(name = "street_address")
	    private String street;
	    private String city;
	    private String state;
	    private String zip;
	  
	    @JsonBackReference
	    @OneToOne(mappedBy="address", cascade=CascadeType.ALL,orphanRemoval = true)
	    private Contact contact;
	}
    ```
   
    **PhoneNumber.java**
    
    ```
    @Getter
	@Setter
	@NoArgsConstructor
	@AllArgsConstructor
	@Builder
	@Entity
	@Table
	@Component
	public class PhoneNumber implements Serializable {
	
	    @Id
	    @GeneratedValue
	    @JsonIgnore
	    private int id;
	    private String type;
	    private String number;
	
	
	    @JsonBackReference
	    @ManyToOne(cascade= { CascadeType.ALL})
	    @JoinColumn(name="contact_id")
	    private Contact contact;
	
	}
    ```
   
   
   
4. #### CRUD operation for Contacts

   In **ContactController.java** class, 
   we have exposed 6 endpoints for basic CRUD operations and an special operation
   - GET All Contact
   - GET by ID
   - POST to store Contact in DB
   - PUT to update Contact
   - DELETE by ID
   - GET Contact based call-list
    
   ```
   @RestController
   @RequestMapping("/contact")
   public class ContactController {
        
    @GetMapping
    public ResponseEntity<List<?>> findAll() 

    @GetMapping("/{id}")
    public ResponseEntity<?> findById(@PathVariable int id) 

    @PostMapping
    public ResponseEntity<?> save(@RequestBody Contact contact)

    @PutMapping("/{id}")
    public ResponseEntity<?> update(@PathVariable int id, @RequestBody Contact contact) 

    @DeleteMapping("/{id}")
    public ResponseEntity<?> delete(@PathVariable int id) 
    
    @GetMapping("/call-list")
    public ResponseEntity<?> findAllCallList() 
    
   ```
   
   In **ContactRepository.java**, we are extending `JpaRepository<Class, ID>` interface which enables CRUD related methods.  
    
   ```
   public interface ContactRepository extends JpaRepository<Contact, Integer> {

	}
   ```
   
   
    
### API Endpoints

- #### Contact CRUD Operations
    > **GET Mapping** http://localhost:8088/contact - Get all Contact
    
    > **GET Mapping** http://localhost:8088/contact/{id} - Get Contact by ID
       
    > **POST Mapping** http://localhost:8088/contact - Add new Contact in DB  
    
     Request Body  
     ```
        {
		    "name": {
		        "firstName": "Robert",
		        "middleName": "Sampson",
		        "lastName": "Rooney"
		    },
		    "address": {
		        "street": "7874 Walker St",
		        "city": "Columbia",
		        "state": "South Carolina",
		        "zip": "19797"
		    },
		    "phoneNumbers": [
		        {
		            "type": "Mobile",
		            "number": "981-443-9148"
		        },
		        {
		            "type": "Home",
		            "number": "981-792-9427"
		        },
		        {
		            "type": "Mobile",
		            "number": "981-443-9148"
		        },
		        {
		            "type": "Home",
		            "number": "981-792-9427"
		        }
		    ],
		    "email": "robert.rooney@yahoo.com"
		}
     ```
    
    > **PUT Mapping** http://localhost:8088/contact/{id} - Update existing Contact for given ID 
                                                       
     Request Body  
     ```
        {
		    "name": {
		        "firstName": "Robert",
		        "middleName": "Sampson",
		        "lastName": "Rooney"
		    },
		    "address": {
		        "street": "7874 Walker St",
		        "city": "Columbia",
		        "state": "South Carolina",
		        "zip": "19797"
		    },
		    "phoneNumbers": [
		        {
		            "type": "Mobile",
		            "number": "981-443-9148"
		        },
		        {
		            "type": "Home",
		            "number": "981-792-9427"
		        },
		        {
		            "type": "Mobile",
		            "number": "981-443-9148"
		        },
		        {
		            "type": "Home",
		            "number": "981-792-9427"
		        }
		    ],
		    "email": "robert123.rooney@yahoo.com"
		}
     ```
    
    > **DELETE Mapping** http://localhost:8088/contact/{id} - Delete Contact by ID
    
    > **GET Mapping** http://localhost:8088/contact/call-list- Get all Contact for whom Home Number is present in contact

