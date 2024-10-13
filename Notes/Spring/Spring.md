### API Contracts & JSON
Questions that must be asked before developing an API:
- How should API consumers interact with the API?
- What data do consumers need to send in various scenarios?
- What data should the API return to consumers, and when?
- What does the API communicate when it's used incorrectly (or something goes wrong)?
#### API Contracts
- The software industry has adopted several patterns for capturing agreed upon API behavior in documentation and code. These agreements are often called "contracts".
- Two examples include Consumer Driven Contracts and Provider Driven Contracts.
- We'll discuss a lightweight concept called API contracts. We define an API contract as a formal agreement between a software provider and a consumer that abstractly communicates how to interact with each other. This contract defines how API providers and consumers interact, what data exchanges looks like, and how to communicate success and failure cases. The provider and consumers do not have to share the same programming language, only the same API contracts.
- Example:
```
Request
	URI: /cashcards/{id}
	HTTP Verb: GET
	Body: None
Response:
	HTTP Status:
		200 OK if the user is authorized and the Cash Card was successfully retrieved
		401 UNAUTHORIZED if the user is unauthenticated or unauthorized
		404 NOT FOUND if the user is authenticated and authroized but the Cash Card cannot be found
		Response Body Type: JSON
		Example Response Body:
			{
				"id": 99,
				"amount": 123.45
			}
```
- API contracts are important because they communicate the behavior of a REST API. They provide specific details about the data being serialized (or deserialized) for each command and parameter being exchanged. The API contracts are written in such a way that can be easily translated into API provider and consumer functionality, and corresponding automated tests.
### Testing First
#### Test Driven Development
- We'll write tests _before_ implementing the application code. This is called test driven development (TDD).
- Why apply TDD? By asserting expected behavior before implementing the desired functionality, we’re designing the system based on what we **want** it to do, rather than what the system already does.
- Another benefit of “test-driving” the application code is that the tests guide you to write the minimum code needed to satisfy the implementation. When the tests pass, you have a working implementation (the application code), and a guard against introducing errors in the future (the tests).
#### The Testing Pyramid
- Different tests can be written at different levels of the system. At each level, there is a balance between the speed of execution, the “cost” to maintain the test, and the confidence it brings to system correctness. This hierarchy is often represented as a “testing pyramid”.
- ![[Pasted image 20241012194740.png]]
- **Unit Tests:** A Unit Test exercises a small “unit” of the system that's isolated from the rest of the system. They should be simple and speedy. You want a high ratio of Unit Tests in your testing pyramid, as they’re key to designing highly cohesive, loosely coupled software.
- **Integration Tests:** Integration Tests exercise a subset of the system and may exercise groups of units in one test. They are more complicated to write and maintain, and run slower than unit tests.
- **End-to-End Tests:** An End-to-End Test exercises the system using the same interface that a user would, such as a web browser. While extremely thorough, End-to-End Tests can be very slow and fragile because they use simulated user interactions in potentially complicated UIs. Implement the smallest number of these tests.
#### The Red, Green, Refactor Loop
- One of the only ways you can safely refactor is when you have a trustworthy test suite. Thus, the best time to refactor the code you're currently focusing on is during the TDD cycle. This is called the Red, Green, Refactor development loop:

1. **Red:** Write a failing test for the desired functionality.
2. **Green:** Implement the simplest thing that can work to make the test pass.
3. **Refactor:** Look for opportunities to simplify, reduce duplication, or otherwise improve the code without changing any behavior—to _refactor._
4. Repeat!
#### Serialization and Deserialization
- Deserialization is the reverse process of serialization. It transforms data from a file or byte stream back into an object for your application. This makes it possible for an object serialized on one platform to be deserialized on a different platform. For example, your client application can serialize an object on Windows while the backend would deserialize it on Linux.
- Serialization and deserialization work together to transform/recreate data objects to/from a portable format. The most popular data format for serializing data is JSON.
### Implementing GET
#### REST, CRUD, and HTTP
- Let’s start with a concise definition of **REST**: Representational State Transfer. In a RESTful system, data objects are called Resource Representations. The purpose of a RESTful API (Application Programming Interface) is to manage the state of these Resources.
- Said another way, you can think of “state” being “value” and “Resource Representation” being an “object” or "thing". Therefore, REST is just a way to manage the values of things. Those things might be accessed via an API, and are often stored in a persistent data store, such as a database.
- A frequently mentioned concept when speaking about REST is **CRUD**. CRUD stands for “Create, Read, Update, and Delete”. These are the four basic operations that can be performed on objects in a data store. We’ll learn that REST has specific guidelines for implementing each one.
- Another common concept associated with REST is the Hypertext Transfer Protocol. In **HTTP**, a caller sends a Request to a URI. A web server receives the request, and routes it to a request handler. The handler creates a Response, which is then sent back to the caller.
- The components of the Request and Response are:
	Request
	- Method (also called Verb)
	- URI (also called Endpoint)
	- Body
	Response
	- Status Code
	- Body
- The power of REST lies in the way it references a Resource, and what the Request and Response look like for each CRUD operation. Let’s take a look at what our API will look like when we're done with this course:
	- For **C**REATE: use HTTP method POST.
	- For **R**EAD: use HTTP method GET.
	- For **U**PDATE: use HTTP method PUT.
	- For **D**ELETE: use HTTP method DELETE.
- The chart below has more details about RESTful CRUD operations.

| Operation | API Endpoint                      | HTTP Method              | Response Status  |
| --------- | --------------------------------- | ------------------------ | ---------------- |
| Create    | ```json<br>/cashcards<br>```      | ```json<br>POST<br>```   | 201 (CREATED)    |
| Read      | ```json<br>/cashcards/{id}<br>``` | ```json<br>GET<br>```    | 200 (OK)         |
| Update    | ```json<br>/cashcards/{id}<br>``` | ```json<br>PUT<br>```    | 204 (NO CONTENT) |
| Delete    | ```json<br>/cashcards/{id}<br>``` | ```json<br>DELETE<br>``` | 204 (NO CONTENT) |
#### The Request Body
- When following REST conventions to create or update a resource, we need to submit data to the API. This is often referred to as the **request body**. The CREATE and  UPDATE operations require that a request body contain the data needed to properly create or update the resource. For example, a new Cash Card might have a beginning cash value amount, and an UPDATE operation might change that amount.
- Request and Response examples:
  ```
	  Request:
		  Method: GET
		  URL: http://cashcard.example.com/cashcards/123
		  Body: (empty)
	```
	```
	Response:
		Status Code: 200
		Body:
		{
			"id": 123,
			"amount": 25.00
		}
	```
### REST in Spring Boot
#### Spring Annotations and Component Scan
- One of the main things Spring does is to configure and instantiate objects. These objects are called Spring Beans, and are usually created by Spring (as opposed to using the Java new keyword). You can direct Spring to create Beans in several ways.
- In this lesson, you’ll annotate a class with a Spring Annotation, which directs Spring to create an instance of the class during Spring’s Component Scan phase. This happens at application startup. The Bean is stored in Spring’s IoC container. From here, the bean can be injected into any code that requests it.
#### Spring Web Controllers
- In Spring Web, Requests are handled by Controllers.
```java
	@RestController
	class CashCardController {
	}
```
- That’s all it takes to tell Spring: “create a REST Controller”. The Controller gets injected into Spring Web, which routes API requests (handled by the Controller) to the correct method.
- ![[Pasted image 20241012203514.png]]
- A Controller method can be designated a handler method, to be called when a request that the method knows how to handle (called a “matching request”) is received. Let’s write a Read request handler method! Here’s a start:
```java
	private CashCard findById(Long requestedId) {
	}
```
- Since REST says that Read endpoints should use the HTTP GET method, you need to tell Spring to route requests to the method only on GET requests. You can use @GetMapping annotation, which needs the URI Path:
```java
	@GetMapping("/cashcards/{requestedId}")
	private CashCard findById(Long requestedId) {
	}
```
- Spring needs to know how to get the value of the requestedId parameter. This is done using the @PathVariable annotation. The fact that the parameter name matches the {requestedId} text within the @GetMapping parameter allows Spring to assign (inject) the correct value to the requestedId variable:
```java
	@GetMapping("/cashcards/{requestedId}")
	private CashCard findById(@PathVariable Long requestedId) {
	}
```
- REST says that the Response needs to contain a Cash Card in its body, and a Response code of 200 (OK). Spring Web provides the ResponseEntity class for this purpose. It also provides several utility methods to produce Response Entities. Here, you can use ResponseEntity to create a ResponseEntity with code **200 (OK)**, and a body containing a CashCard. The final implementation looks like this:
```java
	@RestController
	class CashCardController {
	  @GetMapping("/cashcards/{requestedId}")
	  private ResponseEntity<CashCard> findById(@PathVariable Long requestedId) {
	     CashCard cashCard = /* Here would be the code to retrieve the CashCard */;
	     return ResponseEntity.ok(cashCard);
	  }
	}
```
### Repositories and Spring Data
#### Controller-Repository Architecture
- The **Separation of Concerns** principle states that well-designed software should be modular, with each module having distinct and separate concerns from any other module.
- Up until now, our codebase only returns a hard-coded response from the Controller. This setup violates the Separation of Concerns principle by mixing the concerns of a Controller, which is an abstraction of a web interface, with the concerns of reading and writing data to a data store, such as a database. In order to solve this, we’ll use a common software architecture pattern to enforce data management separation via the **Repository** pattern.
- A common architectural framework that divides these layers, typically by function or value, such as business, data, and presentation layers, is called **Layered Architecture**. In this regard, we can think of our Repository and Controller as two layers in a Layered Architecture. The Controller is in a layer near the Client (as it receives and responds to web requests) while the Repository is in a layer near the data store (as it reads from and writes to the data store). There may be intermediate layers as well, as dictated by business needs.
- The Repository is the interface between the application and the database, and provides a **common abstraction** for any database, making it easier to switch to a different database when needed.
- ![[Pasted image 20241013131350.png]]
- For our database selection, we’ll use an **embedded, in-memory** database. “Embedded” simply means that it’s a Java library, so it can be added to the project just like any other dependency. “In-memory” means that it stores data in memory only, as opposed to persisting data in permanent, durable storage. At the same time, our in-memory database is largely compatible with production-grade relational database management systems (RDBMS) like MySQL, SQL Server, and many others. Specifically, it uses JDBC (the standard Java library for database connectivity) and SQL (the standard database query language).
- ![[Pasted image 20241013131424.png]]
- There are tradeoffs to using an in-memory database instead of a persistent database. On one hand, in-memory allows you to develop without installing a separate RDBMS, and ensures that the database is in the same state (i.e., empty) on every test run. However, you do need a _persistent_ database for the live "production" application. This leads to a ** Dev-Prod Parity** mismatch: Your application might behave differently when running the in-memory database than when running in production.
- The specific in-memory database we’ll use is H2. Fortunately, H2 is highly compatible with other relational databases, so dev-prod parity won’t be a big issue. We’ll use H2 for **convenience for local development**, but we want to recognize the tradeoffs.
#### Auto Configuration
- In the lab, all we need for full database functionality is to add two dependencies. This wonderfully showcases one of the most powerful features of Spring Boot: **Auto Configuration**. Without Spring Boot, we’d have to configure Spring Data to speak to H2. However, because we’ve included the Spring Data dependency (and a specific data provider, H2), Spring Boot will automatically configure your application to communicate with H2.
#### Spring Data's CrudRepository
- For our Repository selection, we’ll use a specific type of Repository: Spring Data’s CrudRepository. At first glance, it’s slightly magical, but let’s unpack that magic. The following is a complete implementation of all CRUD operations by extending CrudRepository:
```json
	interface CashCardRepository extends CrudRepository<CashCard, Long> {
	}
```
- With just the above code, a caller can call any number of predefined CrudRepository methods, such as findById:
```json
	cashCard = cashCardRepository.findById(99);
```
- You might immediately wonder: Where is the implementation of the CashCardRepository.findById() method? CrudRepository and everything it inherits from is an Interface with no actual code! Well, based on the specific Spring Data framework used (which for us will be Spring Data JDBC) Spring Data takes care of this implementation for us during the IoC container startup time. The Spring runtime will then expose the repository as yet another bean that you can reference wherever needed in your application.
- As we’ve learned, there are typically trade-offs. For example the CrudRepository generates SQL statements to read and write your data, which is useful for many cases, but sometimes you need to write your own custom SQL statements for specific use cases.
- Spring Data has many implementations for a variety of relational and non-relational database technologies. Spring Data also has several abstractions on top of those technologies. These are commonly called an Object-Relational Mapping framework, or ORM.
- 