
---
title: "Client Server Architecture"
aliases:
  - "Client-Server Model"
  - "Database Client and Server Architecture"
  - "Three-Tier Application Architecture"
type: learning-note
domain: technology
category: database-architecture
subject: client-server-architecture
level: intermediate
difficulty: intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12
tags:
  - database-architecture
  - client-server
  - application-architecture
  - distributed-systems
  - networking
  - backend
  - frontend
  - postgresql
concepts:
  - client
  - server
  - client-server-model
  - database-server
  - application-server
  - database-client
  - network-protocol
  - request-response
  - two-tier-architecture
  - three-tier-architecture
  - n-tier-architecture
  - connection-pooling
  - api
  - separation-of-concerns
prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[05 - Database Architecture Fundamentals]]"
  - "[[01 - Database Management Systems]]"
  - "[[02 - How a Query is Executed]]"
  - "[[06 - Concurrency and Locks]]"
related:
  - "[[02 - Database Scalability]]"
  - "[[03 - Replication]]"
  - "[[06 - High Availability]]"
  - "[[01 - Data Architecture Overview]]"
  - "[[01 - PostgreSQL Architecture Overview]]"
tools:
  - PostgreSQL
  - pgAdmin
  - psql
  - SQL
  - Python
  - HTTP
  - TCP/IP
learning_objectives:
  - Explain the client-server architectural model.
  - Distinguish clients, application servers, and database servers.
  - Understand how applications communicate with database systems.
  - Compare two-tier, three-tier, and n-tier architectures.
  - Explain the role of APIs and database drivers.
  - Understand connection pooling and network communication.
  - Identify architectural trade-offs involving security, performance, and scalability.
  - Apply client-server concepts to the SIAPV application.
review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# 01 — Client Server Architecture

## 1. Introduction to Client-Server Architecture

Most modern applications do not perform all their operations in a single place.

Instead, responsibilities are distributed among different components that communicate with one another.

A user may interact with a website through a browser, the browser may communicate with an application server, and that application server may retrieve information from a database server.

This organizational model is known as **client-server architecture**.

It is one of the fundamental architectural models used in information systems.

### 1.1 What is client-server architecture?

Client-server architecture is a computing model in which one component requests services or resources and another component provides them.

The two primary roles are:

- **Client:** Initiates requests for information or services.
- **Server:** Receives requests, processes them, and provides responses.

A simplified representation:

```text
+------------------+         +------------------+
|                  | Request |                  |
|      CLIENT      | ------> |      SERVER      |
|                  |         |                  |
|                  | <------ |                  |
+------------------+ Response+------------------+
```

The client and server may run:

- On the same computer.
- On different computers within a local network.
- On different machines across the internet.
- In separate cloud environments.
- Inside containers or virtual machines.

The defining characteristic is not physical distance.

It is the separation of responsibilities and the communication between components.

### 1.2 Example: A database application

Imagine an employee opening a vehicle management application.

The employee searches for a vehicle using its inventory number.

The application sends a request to retrieve the corresponding information.

The database processes the query and returns the matching record.

```text
Employee
   |
   v
Application Interface
   |
   v
Database Request
   |
   v
Database Server
   |
   v
Query Execution
   |
   v
Database Response
   |
   v
Application Displays Vehicle
```

The employee interacts with the application, not directly with the physical database files.

The database server manages the data and provides access through an established communication mechanism.

## 2. Understanding the Client

A client is a component that requests services from another component.

In database applications, clients can take several forms.

Examples include:

- A web browser.
- A desktop application.
- A mobile application.
- A Python script.
- A database administration tool.
- Another backend service.

### 2.1 Web browser as a client

A browser such as Chrome or Firefox can act as a client.

It sends HTTP requests to an application server.

For example:

```http
GET /api/vehicles/101 HTTP/1.1
Host: example.com
```

The request asks the application server for information about vehicle 101.

The browser does not necessarily communicate directly with the database.

Instead, it usually communicates with an API exposed by the application backend.

### 2.2 Database administration tools as clients

Tools such as pgAdmin and psql can connect directly to a PostgreSQL server.

For example:

```text
pgAdmin
   |
   | PostgreSQL connection
   v
PostgreSQL Server
```

The user writes SQL statements through the client.

The client sends those statements to PostgreSQL, which executes them and returns the results.

This is different from a browser-based application architecture because the administration tool communicates directly with the database server.

### 2.3 Client responsibilities

Depending on the application, a client may be responsible for:

- Displaying information.
- Collecting user input.
- Validating basic input formats.
- Sending requests.
- Receiving responses.
- Presenting errors.
- Managing the user interface.

A client should not automatically be trusted to enforce every business rule.

Critical validation must also be implemented in trusted application logic or enforced by the database.

## 3. Understanding the Server

A server is a component that provides services to clients.

In a database environment, the word server can refer to different things.

It is important to distinguish them.

### 3.1 Application server

An application server executes application logic.

Its responsibilities may include:

- Processing user requests.
- Authenticating users.
- Applying business rules.
- Validating data.
- Communicating with databases.
- Calling external services.
- Preparing responses.

For example, when an employee registers vehicle maintenance, the application server may verify that:

1. The vehicle exists.
2. The user has permission.
3. The maintenance information is valid.
4. The requested operation satisfies business rules.

It then communicates with the database to store the information.

### 3.2 Database server

A database server provides database services.

In PostgreSQL, the server manages:

- Database connections.
- SQL statement processing.
- Query execution.
- Transaction management.
- Locking and concurrency.
- Storage access.
- Logging and recovery.
- Security and permissions.

The database server is responsible for maintaining the database's internal consistency.

### 3.3 Physical server vs. server software

These terms should not be confused.

| Concept | Meaning |
|---|---|
| Physical server | A computer or hardware system that provides computing resources. |
| Virtual server | A virtualized computing environment. |
| Server software | A program that provides services to clients. |
| Database server | Software and its execution environment responsible for database services. |
| Application server | Software responsible for application execution and business logic. |

A single physical machine can host multiple server processes.

For example:

```text
Physical Machine
       |
       +-- Operating System
       |
       +-- Web Server
       |
       +-- Application Server
       |
       +-- PostgreSQL Server
```

However, larger architectures often distribute these responsibilities across different machines.

## 4. How a Client Communicates with a Database Server

Communication requires more than simply knowing the database name.

The client and server must establish a connection and exchange information using an appropriate protocol.

### 4.1 Connection parameters

A typical PostgreSQL connection requires information such as:

| Parameter | Purpose |
|---|---|
| Host | Address or hostname of the database server. |
| Port | Network port where PostgreSQL accepts connections. |
| Database | Database to connect to. |
| Username | Database account. |
| Password | Authentication credential. |
| SSL configuration | Settings for encrypted communication. |

PostgreSQL commonly uses port:

```text
5432
```

Example connection string:

```text
postgresql://app_user:password@db.example.com:5432/fleet_db
```

This is an illustrative connection string.

Real applications should not expose credentials in source code, public repositories, or logs.

### 4.2 Network communication

Client-server communication often relies on the TCP/IP networking model.

For example:

```text
Application
    |
    v
Database Driver
    |
    v
TCP Connection
    |
    v
Network
    |
    v
PostgreSQL Server
```

TCP provides a reliable, ordered communication channel between endpoints.

PostgreSQL uses its own application-level protocol on top of the network connection.

This allows clients and servers to exchange messages such as:

- Connection requests.
- Authentication messages.
- SQL statements.
- Query results.
- Errors.
- Transaction-related commands.

### 4.3 What happens during a connection?

A simplified connection process:

1. The client identifies the database server.
2. The client establishes a network connection.
3. The client and server exchange PostgreSQL protocol messages.
4. The server performs authentication.
5. The server checks connection permissions.
6. A database session is established.
7. The client can execute SQL statements.
8. The connection remains available until it is closed or terminated.

The exact protocol sequence depends on the connection configuration and authentication method.

## 5. The Role of Database Drivers

A database driver is software that allows an application to communicate with a database system.

Applications generally do not need to implement the database communication protocol themselves.

They use drivers or libraries that handle the communication details.

### 5.1 Why are drivers necessary?

Consider a Python application that needs to access PostgreSQL.

Python does not automatically understand every detail of PostgreSQL's wire protocol.

A database driver provides an interface between the application and the database.

```text
Python Application
       |
       v
Database Driver
       |
       v
PostgreSQL Protocol
       |
       v
PostgreSQL Server
```

### 5.2 Examples of PostgreSQL drivers

| Language | Driver or library |
|---|---|
| Python | psycopg |
| Python | asyncpg |
| Java | PostgreSQL JDBC Driver |
| Node.js | node-postgres (pg) |
| .NET | Npgsql |
| Go | pgx |

These libraries allow developers to execute queries and retrieve results through language-specific interfaces.

### 5.3 Python example using psycopg

```python
import psycopg

connection = psycopg.connect(
    host="localhost",
    port=5432,
    dbname="fleet_db",
    user="app_user",
    password="your_password"
)

with connection.cursor() as cursor:
    cursor.execute(
        """
        SELECT vehicle_id, mileage
        FROM vehicles
        WHERE vehicle_id = %s;
        """,
        (101,)
    )

    result = cursor.fetchone()
    print(result)

connection.close()
```

The driver handles communication with PostgreSQL.

The application uses Python objects and methods rather than manually constructing protocol messages.

**Important:** The `%s` placeholder is a parameter placeholder used by psycopg. It is not Python string interpolation.

Using parameterized queries helps protect against SQL injection and avoids unsafe construction of SQL statements.

For production applications, credentials should be supplied through secure configuration mechanisms rather than embedded in source code.

## 6. Two-Tier Architecture

A two-tier architecture separates an application into two primary layers:

1. Client.
2. Database server.

The client communicates directly with the database server.

### 6.1 Architecture diagram

```text
+---------------------------+
|                           |
|          CLIENT           |
|                           |
|  User Interface           |
|  Application Logic        |
|  Database Driver          |
|                           |
+-------------+-------------+
              |
              | Database Protocol
              |
              v
+---------------------------+
|                           |
|      DATABASE SERVER      |
|                           |
|  PostgreSQL               |
|  Query Processing         |
|  Transactions             |
|  Storage Management       |
|                           |
+---------------------------+
```

### 6.2 How it works

The client contains application functionality and database communication logic.

It sends SQL statements directly to the database.

The database executes them and returns the results.

Examples include:

- Desktop applications connected directly to PostgreSQL.
- Database administration tools.
- Internal applications running in controlled environments.

### 6.3 Advantages

- Relatively simple architecture.
- Fewer components.
- Direct database communication.
- Easier to understand and implement.
- Suitable for small or controlled environments.

### 6.4 Disadvantages

- Database credentials may be distributed across clients.
- Business logic can become duplicated.
- Database access must be managed for every client.
- Scaling may become difficult.
- Exposing the database directly increases the security surface.
- Updating application logic may require updating multiple clients.

### 6.5 When is two-tier appropriate?

Two-tier architecture can be appropriate for:

- Small internal applications.
- Desktop tools.
- Administrative utilities.
- Controlled environments with a limited number of trusted clients.

It is not automatically unsuitable for larger systems, but security, connection management, and application logic become increasingly important as the number of clients grows.

## 7. Three-Tier Architecture

Three-tier architecture separates the system into three logical layers:

1. Presentation layer.
2. Application layer.
3. Data layer.

This is one of the most common structures used in modern business applications.

### 7.1 Architecture diagram

```text
+---------------------------+
|                           |
|     PRESENTATION LAYER    |
|                           |
|  Web Browser              |
|  Mobile Application       |
|  User Interface           |
|                           |
+-------------+-------------+
              |
              | HTTP / HTTPS
              |
              v
+---------------------------+
|                           |
|      APPLICATION LAYER    |
|                           |
|  Backend                  |
|  Business Logic           |
|  Authentication           |
|  API                      |
|                           |
+-------------+-------------+
              |
              | Database Protocol
              |
              v
+---------------------------+
|                           |
|         DATA LAYER        |
|                           |
|  PostgreSQL               |
|  Tables                   |
|  Indexes                  |
|  Transactions             |
|  Persistent Storage       |
|                           |
+---------------------------+
```

### 7.2 Presentation layer

The presentation layer is responsible for user interaction.

It displays information and collects user input.

Examples:

- HTML.
- CSS.
- JavaScript.
- React.
- Angular.
- Vue.
- Mobile application interfaces.

The presentation layer should focus on how information is presented and how users interact with the system.

It should not be responsible for directly controlling database storage.

### 7.3 Application layer

The application layer contains the business logic.

It processes requests and coordinates operations.

Examples:

- Python with FastAPI or Django.
- Java with Spring Boot.
- Node.js with Express or NestJS.
- C# with ASP.NET Core.

Responsibilities include:

- Authentication.
- Authorization.
- Business rules.
- Input validation.
- Request processing.
- Database communication.
- Transaction coordination.
- Error handling.

This layer acts as an intermediary between the presentation and data layers.

### 7.4 Data layer

The data layer manages persistent information.

It may include:

- PostgreSQL.
- MySQL.
- SQL Server.
- Oracle Database.
- IBM Db2.

Its responsibilities include:

- Storing data.
- Executing SQL queries.
- Enforcing constraints.
- Managing transactions.
- Controlling concurrent access.
- Providing database recovery mechanisms.

The application layer communicates with the database through a driver or data-access component.

### 7.5 Advantages of three-tier architecture

| Advantage | Explanation |
|---|---|
| Separation of concerns | Each layer has a defined responsibility. |
| Security | Clients do not need direct database credentials. |
| Maintainability | Changes can be made within individual layers. |
| Scalability | Application servers can be scaled independently. |
| Reusability | Multiple clients can use the same backend services. |
| Centralized business logic | Rules are implemented in a controlled location. |
| Flexibility | Components can evolve independently when interfaces remain compatible. |

### 7.6 Disadvantages

- More components to deploy and maintain.
- Additional network communication.
- More complex monitoring.
- Application-layer failures can affect all connected clients.
- Requires proper API and connection management.

The architecture introduces additional complexity in exchange for better separation and control.

## 8. N-Tier Architecture

N-tier architecture extends the idea of separating application responsibilities into multiple layers or services.

The letter N represents a variable number of tiers.

A system may contain:

- Presentation layer.
- API gateway.
- Authentication service.
- Application services.
- Business logic services.
- Database layer.
- Cache layer.
- Message broker.
- Analytics platform.

### 8.1 Example architecture

```text
+-------------------------+
|       Web Client        |
+------------+------------+
             |
             v
+-------------------------+
|       API Gateway       |
+------------+------------+
             |
       +-----+-----+
       |           |
       v           v
+-----------+ +-----------+
| User      | | Vehicle   |
| Service   | | Service   |
+-----+-----+ +-----+-----+
      |             |
      v             v
+-----------+ +-----------+
| User DB   | | Fleet DB  |
+-----------+ +-----------+
```

This example illustrates a distributed application in which different services manage different responsibilities.

It is not necessarily required to implement every component as a separate physical server.

Some components may run in the same deployment environment.

### 8.2 Why use N-tier architecture?

Larger systems may require:

- Independent scaling.
- Service isolation.
- Multiple teams working on separate components.
- Different data stores for different workloads.
- Integration with external systems.
- More advanced security boundaries.
- Independent deployment cycles.

However, adding layers does not automatically make a system better.

Each component introduces operational responsibilities.

### 8.3 Three-tier vs. N-tier

| Characteristic | Three-tier | N-tier |
|---|---|---|
| Structure | Three main logical layers. | Multiple logical layers or services. |
| Complexity | Moderate. | Potentially high. |
| Deployment | Often relatively straightforward. | May involve distributed deployments. |
| Scalability | Can scale individual layers. | Can scale individual components more independently. |
| Management | Fewer components. | More monitoring and coordination required. |
| Typical use | Business applications. | Large or distributed systems. |

**Important distinction:** A tier is an architectural separation of responsibility or deployment. A layer is a logical separation of concerns. The terms are sometimes used interchangeably, but they are not always identical.

## 9. Understanding APIs in Client-Server Architecture

An API (Application Programming Interface) defines how software components communicate.

In a three-tier web application, the presentation layer commonly communicates with the backend through an HTTP API.

The API provides defined operations and data formats.

### 9.1 Example of an API request

A frontend application may request:

```http
GET /api/vehicles/101
```

The backend processes the request and queries PostgreSQL.

It may return a JSON response:

```json
{
  "vehicle_id": 101,
  "inventory_number": "PK-00101",
  "mileage": 25500,
  "status": "AVAILABLE"
}
```

The frontend uses this information to display the vehicle.

### 9.2 Request-response cycle

```text
Browser
   |
   | GET /api/vehicles/101
   v
Backend API
   |
   | Validate request
   | Apply business rules
   |
   v
Database Driver
   |
   | SQL query
   v
PostgreSQL
   |
   | Query result
   v
Backend API
   |
   | JSON response
   v
Browser
```

Each component has a different responsibility.

The browser handles presentation.

The backend handles application logic.

The database handles persistent data and transactional operations.

### 9.3 REST as an API architectural style

REST (Representational State Transfer) is an architectural style commonly used for web APIs.

REST APIs often organize resources around URLs and use HTTP methods.

| HTTP method | Common purpose |
|---|---|
| GET | Retrieve a resource. |
| POST | Create a resource or initiate an operation. |
| PUT | Replace a resource. |
| PATCH | Partially modify a resource. |
| DELETE | Delete a resource. |

Example endpoints:

```text
GET     /api/vehicles
GET     /api/vehicles/101
POST    /api/vehicles
PATCH   /api/vehicles/101
DELETE  /api/vehicles/101
```

These are illustrative API conventions.

A real API should define its authorization rules, validation behavior, error responses, and resource semantics.

## 10. Connection Pooling

A database connection consumes server resources.

Creating and closing a new database connection for every application request can be inefficient.

Connection pooling addresses this problem.

### 10.1 What is connection pooling?

Connection pooling is a technique that maintains a reusable collection of database connections.

Instead of creating a new connection for every request, the application borrows an available connection from the pool.

After the operation finishes, the connection is returned to the pool.

### 10.2 Without connection pooling

```text
Request 1 → Create Connection → Query → Close
Request 2 → Create Connection → Query → Close
Request 3 → Create Connection → Query → Close
Request 4 → Create Connection → Query → Close
```

Repeated connection creation can introduce unnecessary overhead.

### 10.3 With connection pooling

```text
             Connection Pool
           +-------------------+
           | Connection 1      |
           | Connection 2      |
           | Connection 3      |
           | Connection 4      |
           +-------------------+
              ^     ^     ^
              |     |     |
           Requests from application
```

Connections are reused across multiple requests.

### 10.4 Benefits

- Reduces repeated connection-establishment overhead.
- Limits the number of simultaneous database connections.
- Improves resource utilization.
- Helps manage database load.
- Can improve application responsiveness.

### 10.5 Potential problems

An incorrectly configured pool can create problems.

For example:

- Too many connections can overwhelm the database.
- Too few connections can create request queues.
- Connections may remain occupied by long-running queries.
- Transactions may accidentally remain open.
- Poor timeout configuration can cause application delays.

### 10.6 PostgreSQL connection pooling tools

Examples include:

- PgBouncer.
- Pgpool-II.
- Application-level connection pools.
- SQLAlchemy connection pooling in Python applications.

Connection pooling is particularly important when many users access a backend application simultaneously.

## 11. Security in Client-Server Architecture

Separating application and database responsibilities also creates important security boundaries.

A browser should generally not have direct access to a production database.

Instead, it communicates with an application backend.

### 11.1 Why should clients not connect directly to production databases?

Consider the following design:

```text
Browser
   |
   v
PostgreSQL
```

This design can expose:

- Database credentials.
- Database network ports.
- Internal database structure.
- Excessive database privileges.
- Direct access to sensitive information.

A safer common design is:

```text
Browser
   |
   | HTTPS
   v
Application Server
   |
   | Restricted database connection
   v
PostgreSQL
```

The backend controls which operations are available.

### 11.2 Security responsibilities by layer

| Layer | Security responsibilities |
|---|---|
| Presentation | Secure session handling and safe rendering of information. |
| Application | Authentication, authorization, validation, and request controls. |
| Database | Roles, permissions, constraints, and controlled access. |
| Network | Encryption, firewall rules, and restricted connectivity. |
| Infrastructure | Secrets management, operating-system security, and monitoring. |

### 11.3 Principle of least privilege

The principle of least privilege means that each component or user should receive only the permissions necessary to perform its responsibilities.

For example, a backend account that only needs to retrieve vehicle information should not automatically receive permission to drop tables or modify database roles.

Example:

```sql
CREATE ROLE vehicle_reader LOGIN PASSWORD 'replace_with_secure_secret';

GRANT CONNECT ON DATABASE fleet_db TO vehicle_reader;

GRANT USAGE ON SCHEMA public TO vehicle_reader;

GRANT SELECT ON vehicles TO vehicle_reader;
```

This account receives read access to the specified table rather than unrestricted database privileges.

In production, credentials should be stored securely and rotated according to organizational policies.

## 12. Deployment Architecture: Logical vs. Physical Separation

An important architectural distinction is that logical layers do not necessarily require separate physical machines.

### 12.1 Logical separation

Logical separation means that responsibilities are divided into components or modules.

For example:

```text
Application
  |
  +-- Presentation Module
  |
  +-- Business Logic Module
  |
  +-- Data Access Module
```

These components may run within the same application process.

### 12.2 Physical separation

Physical separation means that components run in different execution environments.

For example:

```text
Machine 1:
Web Application

Machine 2:
Application Backend

Machine 3:
PostgreSQL Database
```

This can improve isolation, security, and independent resource allocation.

However, it also introduces network dependencies.

### 12.3 Deployment comparison

| Deployment model | Description |
|---|---|
| Single-machine deployment | Multiple components run on one physical or virtual machine. |
| Multi-server deployment | Components run on separate machines. |
| Containerized deployment | Components run inside containers. |
| Cloud deployment | Components use cloud-hosted infrastructure and services. |
| Hybrid deployment | Components are distributed across on-premises and cloud environments. |

An architecture diagram should distinguish logical responsibilities from actual deployment locations whenever that distinction matters.

## 13. Latency and Network Communication

When components communicate over a network, the distance and quality of the network connection can affect application performance.

Latency is the time required for information to travel between endpoints.

### 13.1 Why latency matters

Consider this request:

```text
Browser → Backend → Database → Backend → Browser
```

The request involves multiple communication steps.

Even if the SQL query executes quickly, the total response time can be affected by:

- Network latency.
- Connection establishment.
- Application processing.
- Database execution.
- Serialization and deserialization.
- Response transmission.

### 13.2 Response time decomposition

A simplified model:

```text
Total Response Time =
    Network Time
  + Application Processing Time
  + Database Processing Time
  + Response Transmission Time
```

This is a conceptual decomposition, not a universal measurement formula.

Some stages may overlap, and real distributed systems introduce additional components.

### 13.3 Architectural implications

To reduce unnecessary communication:

- Avoid excessive small database requests.
- Retrieve only the information required.
- Use appropriate indexes.
- Reuse database connections.
- Avoid transferring unnecessary data.
- Design efficient API endpoints.
- Consider caching when appropriate.

A system should be optimized based on measurements rather than assumptions.

## 14. Applying Client-Server Architecture to SIAPV

Consider the SIAPV application developed for managing vehicles and machinery.

Its functionality includes:

- Inventory management.
- Vehicle and machinery details.
- Search.
- Reports.
- Record creation.
- Record updates.
- Record deletion.
- Database interaction through PostgreSQL.

Understanding its architecture helps identify how the application communicates with its data store.

### 14.1 Identify the components

A conceptual representation:

```text
+--------------------------------+
|          USER                  |
|                                |
|  Employee or Administrator     |
+---------------+----------------+
                |
                v
+--------------------------------+
|       PRESENTATION             |
|                                |
|  HTML                          |
|  CSS                           |
|  JavaScript / Interface        |
+---------------+----------------+
                |
                v
+--------------------------------+
|      APPLICATION LOGIC         |
|                                |
|  Request Handling               |
|  Validation                    |
|  Business Rules                 |
|  Database Operations           |
+---------------+----------------+
                |
                v
+--------------------------------+
|       DATABASE ACCESS          |
|                                |
|  SQL                           |
|  Database Connection           |
+---------------+----------------+
                |
                v
+--------------------------------+
|       DATA LAYER               |
|                                |
|  PostgreSQL                    |
|  Tables                        |
|  Constraints                   |
|  Indexes                       |
|  Persistent Records            |
+--------------------------------+
```

This is a conceptual model. The actual physical deployment and execution boundaries depend on how SIAPV is configured.

### 14.2 Questions for an architecture review

To understand the system completely, investigate:

1. Does the browser communicate directly with PostgreSQL?
2. Is there a backend service between the interface and the database?
3. Where are database credentials stored?
4. Where is validation performed?
5. Which component enforces business rules?
6. How are database connections established?
7. Are connections reused?
8. How are errors handled?
9. How are user permissions enforced?
10. Can the application interface be changed without redesigning the database?

These questions reveal the actual architectural boundaries.

### 14.3 Potential improvements

Depending on the current implementation, possible improvements could include:

- Introducing a dedicated backend API.
- Centralizing business logic.
- Implementing role-based authorization.
- Using parameterized SQL statements.
- Introducing connection pooling.
- Separating development and production environments.
- Adding structured application logs.
- Implementing automated tests.
- Deploying the application through containers.

The appropriate improvements should be based on actual requirements and existing implementation details.

An architecture should solve real problems rather than introduce unnecessary components.

## 15. Practical Laboratory — Building a Three-Tier Model

The objective is to understand the communication between a client, an application backend, and PostgreSQL.

We will use:

- Python.
- FastAPI.
- PostgreSQL.
- psycopg.
- HTTP requests.

### 15.1 Install the required packages

Inside a Python virtual environment:

```bash
pip install fastapi uvicorn psycopg[binary]
```

### 15.2 Create a database table

Execute this in PostgreSQL:

```sql
CREATE TABLE IF NOT EXISTS architecture_lab (
    vehicle_id INTEGER PRIMARY KEY,
    inventory_number VARCHAR(30) NOT NULL,
    mileage INTEGER NOT NULL,
    status VARCHAR(30) NOT NULL
);

INSERT INTO architecture_lab (
    vehicle_id,
    inventory_number,
    mileage,
    status
)
VALUES
    (101, 'PK-00101', 25000, 'AVAILABLE'),
    (102, 'PK-00102', 40000, 'IN_MAINTENANCE')
ON CONFLICT (vehicle_id) DO NOTHING;
```

### 15.3 Create the backend application

Create a file named:

```text
main.py
```

Example:

```python
from fastapi import FastAPI, HTTPException
import psycopg

app = FastAPI(
    title="Database Architecture Lab",
    description="A simple three-tier architecture example"
)

def get_connection():
    return psycopg.connect(
        host="localhost",
        port=5432,
        dbname="fleet_db",
        user="app_user",
        password="your_password"
    )

@app.get("/vehicles/{vehicle_id}")
def get_vehicle(vehicle_id: int):

    try:
        with get_connection() as connection:

            with connection.cursor() as cursor:

                cursor.execute(
                    """
                    SELECT
                        vehicle_id,
                        inventory_number,
                        mileage,
                        status
                    FROM architecture_lab
                    WHERE vehicle_id = %s;
                    """,
                    (vehicle_id,)
                )

                result = cursor.fetchone()

                if result is None:
                    raise HTTPException(
                        status_code=404,
                        detail="Vehicle not found"
                    )

                return {
                    "vehicle_id": result[0],
                    "inventory_number": result[1],
                    "mileage": result[2],
                    "status": result[3]
                }

    except HTTPException:
        raise

    except psycopg.Error:
        raise HTTPException(
            status_code=500,
            detail="Database operation failed"
        )
```

This is a learning example. A production implementation should use environment-based configuration, connection pooling, structured logging, and carefully designed exception handling.

### 15.4 Start the application

```bash
uvicorn main:app --reload
```

The backend will normally be available at:

```text
http://127.0.0.1:8000
```

### 15.5 Request vehicle information

Open:

```text
http://127.0.0.1:8000/vehicles/101
```

The application should return a JSON response similar to:

```json
{
  "vehicle_id": 101,
  "inventory_number": "PK-00101",
  "mileage": 25000,
  "status": "AVAILABLE"
}
```

### 15.6 Understand the complete request flow

```text
1. User opens the endpoint
          |
          v
2. Browser sends HTTP request
          |
          v
3. FastAPI receives the request
          |
          v
4. Backend validates the identifier
          |
          v
5. psycopg establishes database communication
          |
          v
6. PostgreSQL executes SQL
          |
          v
7. Database returns the record
          |
          v
8. Backend converts the result into JSON
          |
          v
9. Browser receives the response
```

### 15.7 What did we build?

We created a simple three-tier application:

| Tier | Component |
|---|---|
| Presentation | Browser or HTTP client. |
| Application | FastAPI backend. |
| Data | PostgreSQL database. |

The exercise demonstrates how a backend separates user-facing requests from direct database operations.

## 16. Architecture Design Principles

When designing client-server systems, consider the following principles.

### 16.1 Separation of concerns

Each component should have a clear responsibility.

For example:

- Frontend: presentation.
- Backend: business logic.
- Database: persistent data and integrity.

### 16.2 Loose coupling

Components should depend on well-defined interfaces rather than unnecessary implementation details.

For example, a frontend should communicate with an API contract instead of depending on the internal structure of PostgreSQL tables.

### 16.3 High cohesion

A component should group responsibilities that logically belong together.

For example, vehicle-related business operations should be organized coherently rather than scattered across unrelated modules.

### 16.4 Least privilege

Each component should receive only the access necessary to perform its function.

### 16.5 Independent scalability

Where requirements justify it, application and database resources should be scalable independently.

### 16.6 Fault isolation

A failure in one component should not unnecessarily compromise every other component.

For example, an API should handle database connection failures gracefully rather than exposing internal error details to users.

### 16.7 Observability

A system should provide sufficient information to understand its behavior.

Useful mechanisms include:

- Application logs.
- Database logs.
- Metrics.
- Error tracking.
- Connection monitoring.
- Request tracing.

## 17. Knowledge Check

Answer these questions without consulting the previous sections.

1. What is client-server architecture?
2. What distinguishes a client from a server?
3. What is the difference between an application server and a database server?
4. Can a physical machine host multiple servers?
5. What role does a database driver perform?
6. What is the purpose of TCP/IP in client-server communication?
7. What are the main characteristics of two-tier architecture?
8. What are the three layers of a three-tier architecture?
9. Why is business logic commonly placed in the application layer?
10. What is N-tier architecture?
11. What is the difference between a logical layer and a physical deployment tier?
12. What is an API?
13. What is REST?
14. What is connection pooling?
15. Why should a browser generally not connect directly to a production database?
16. What is the principle of least privilege?
17. How can network latency affect application performance?
18. Why should application and database responsibilities be separated?
19. How does FastAPI communicate with PostgreSQL in the laboratory?
20. Which architectural improvements could be useful for SIAPV?

## 18. Practical Exercises

### Exercise 1 — Identify the architecture

Classify each example as two-tier, three-tier, or N-tier.

A. A desktop application connects directly to PostgreSQL.

B. A browser communicates with a backend API, which accesses PostgreSQL.

C. A browser communicates with an API gateway, several independent services, a cache, and multiple databases.

Explain your classification.

### Exercise 2 — Design a vehicle management application

Design a three-tier architecture for a system that manages:

- Vehicles.
- Machinery.
- Maintenance.
- Mileage records.
- Work crew assignments.
- Reports.

Identify:

1. The responsibilities of the presentation layer.
2. The responsibilities of the application layer.
3. The responsibilities of the data layer.
4. How the layers communicate.
5. Where authentication and authorization should be implemented.

### Exercise 3 — Evaluate a design decision

An application currently allows every browser to connect directly to PostgreSQL using the same database credentials.

Explain:

- The security risks.
- The maintainability problems.
- The scalability limitations.
- How a backend API could improve the design.

### Exercise 4 — Analyze connection pooling

A backend receives 300 simultaneous requests.

Each request creates a new PostgreSQL connection and closes it after retrieving data.

Explain:

- Why this could become inefficient.
- How a connection pool changes the process.
- Why pool size matters.
- What happens if every connection is occupied.

### Exercise 5 — Architecture diagram

Create an architecture diagram for SIAPV using Mermaid.

Include:

- User.
- Browser interface.
- Application logic.
- Database driver.
- PostgreSQL.
- Persistent storage.

Then describe the direction of requests and responses.

## 19. Technical Vocabulary

| English term | Spanish meaning |
|---|---|
| Client | Cliente |
| Server | Servidor |
| Client-server architecture | Arquitectura cliente-servidor |
| Application server | Servidor de aplicaciones |
| Database server | Servidor de bases de datos |
| Database client | Cliente de base de datos |
| Database driver | Controlador de base de datos |
| Network protocol | Protocolo de red |
| Connection | Conexión |
| Request | Solicitud |
| Response | Respuesta |
| Presentation layer | Capa de presentación |
| Application layer | Capa de aplicación |
| Data layer | Capa de datos |
| Two-tier architecture | Arquitectura de dos capas |
| Three-tier architecture | Arquitectura de tres capas |
| N-tier architecture | Arquitectura de múltiples capas |
| API | Interfaz de programación de aplicaciones |
| Endpoint | Punto de acceso |
| Backend | Parte lógica del servidor |
| Frontend | Parte visual de la aplicación |
| Connection pooling | Agrupación y reutilización de conexiones |
| Latency | Latencia |
| Scalability | Escalabilidad |
| Separation of concerns | Separación de responsabilidades |
| Loose coupling | Bajo acoplamiento |
| High cohesion | Alta cohesión |
| Least privilege | Mínimo privilegio |
| Deployment | Despliegue |
| Fault isolation | Aislamiento de fallos |
| Observability | Observabilidad |

## 20. Final Mental Model

Client-server architecture organizes an application into components that communicate through defined interfaces.

The client requests information or services.

The application server processes requests and applies business rules.

The database server manages persistent information and executes database operations.

```text
             USER
              |
              v
       CLIENT APPLICATION
              |
              | HTTP / HTTPS
              |
              v
       APPLICATION SERVER
              |
              | Database Driver
              |
              v
       DATABASE SERVER
              |
              v
       DATABASE STORAGE
```

Two-tier architecture provides a direct client-to-database model.

Three-tier architecture introduces a dedicated application layer.

N-tier architecture distributes responsibilities across additional services or components.

The appropriate design depends on requirements involving security, scalability, maintainability, performance, and operational complexity.

The most important architectural lesson is this:

**A well-designed application does not simply connect components together. It defines clear responsibilities, controlled communication, and appropriate boundaries between them.**

---

## 21. Related Notes

### Previous concepts

- [[05 - Database Architecture Fundamentals]]
- [[01 - Database Management Systems]]
- [[02 - How a Query is Executed]]
- [[06 - Concurrency and Locks]]

### Next topics

- [[02 - Database Scalability]]
- [[03 - Replication]]
- [[04 - Partitioning]]
- [[05 - Sharding]]
- [[06 - High Availability]]
- [[07 - Backup and Disaster Recovery]]

### Practical technologies

- PostgreSQL
- Python
- FastAPI
- psycopg
- HTTP and HTTPS
- TCP/IP
- PgBouncer
- Docker

---

## 22. Completion Criteria

- [ ] Explain the client-server model.
- [ ] Distinguish application servers from database servers.
- [ ] Explain the role of database drivers.
- [ ] Describe how clients communicate with PostgreSQL.
- [ ] Compare two-tier, three-tier, and N-tier architectures.
- [ ] Explain the responsibilities of each application layer.
- [ ] Understand APIs and REST endpoints.
- [ ] Explain connection pooling.
- [ ] Identify common security boundaries.
- [ ] Distinguish logical architecture from physical deployment.
- [ ] Explain the impact of network latency.
- [ ] Design a basic three-tier architecture for a database application.
