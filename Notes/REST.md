## REST vs RESTful

Both terms are related to designing web APIs, but they refer to slightly different things.

### 1. What is REST?

**REST** stands for **Representational State Transfer**.

It is an **architectural style** for designing network-based applications, especially web APIs.

REST defines principles for how a client and server should communicate using resources, HTTP methods, and standard HTTP behavior.

For example, in an employee management system:

* **Employee** is a resource.
* `/employees` identifies the collection of employees.
* `/employees/101` identifies one employee.
* HTTP methods define what action to perform.

### 2. What is RESTful?

**RESTful** means **an API that follows REST principles**.

So, REST is the **architectural style**, while RESTful describes an **implementation that follows that style**.

> **REST = Design principles**
>
> **RESTful API = API designed according to those principles**

### Example

| Operation         | HTTP Method | Endpoint         | Meaning                      |
| ----------------- | ----------- | ---------------- | ---------------------------- |
| Get all employees | GET         | `/employees`     | Retrieve employee collection |
| Get one employee  | GET         | `/employees/101` | Retrieve employee 101        |
| Create employee   | POST        | `/employees`     | Create a new employee        |
| Update employee   | PUT         | `/employees/101` | Replace employee 101         |
| Partially update  | PATCH       | `/employees/101` | Modify selected fields       |
| Delete employee   | DELETE      | `/employees/101` | Delete employee 101          |

An API using these conventions is generally called **RESTful**.

### REST vs RESTful — Simple Comparison

| REST                                    | RESTful                                         |
| --------------------------------------- | ----------------------------------------------- |
| Architectural style                     | API or service following REST                   |
| Defines principles and constraints      | Implements those principles                     |
| Not a programming language or framework | Can be built using FastAPI, Flask, Django, etc. |
| Describes the design approach           | Describes the resulting API                     |

### Important Interview Point

**REST is not a protocol.** It is an architectural style. RESTful APIs commonly use HTTP, but REST principles are broader than simply using HTTP methods.

Also, **not every API that uses HTTP is fully RESTful**. For example:

```http
POST /getEmployee
POST /deleteEmployee
```

These are HTTP APIs, but they do not follow typical REST resource-oriented conventions as closely as:

```http
GET    /employees/101
DELETE /employees/101
```

### Interview Answer

> "REST stands for Representational State Transfer. It is an architectural style that defines principles for designing web services around resources, HTTP methods, stateless communication, and standard HTTP behavior. RESTful refers to an API that follows these REST principles. For example, using GET to retrieve an employee, POST to create one, PUT or PATCH to update one, and DELETE to remove one through resource-based URLs."


---

**Representational State Transfer (REST)** is the full form of REST. Let's break down each word in simple terms.

### 1. Representational

A **resource** is something your application manages, such as an employee, product, or order.

The client does not directly receive the database object. Instead, the server sends a **representation** of that resource, usually in JSON.

For example, an employee resource:

```json
{
  "id": 101,
  "name": "Kishore",
  "department": "IT"
}
```

This JSON is a **representation of the employee**, not the actual database record.

### 2. State

Here, **state** means the current data or condition of a resource.

For example, an employee's state might be:

```json
{
  "id": 101,
  "name": "Kishore",
  "status": "Active"
}
```

If the employee's status changes to `"Inactive"`, the resource's state has changed.

**Important:** In REST, "state" in the name does not mean the server must maintain a user's session. REST APIs are generally **stateless**, meaning each request contains the information needed to process it.

### 3. Transfer

**Transfer** means moving the representation of a resource between the client and server.

For example:

```text
Client                         Server
  |                              |
  | GET /employees/101           |
  |----------------------------->|
  |                              |
  | 200 OK + Employee JSON       |
  |<-----------------------------|
```

The server transfers a representation of the employee to the client.

### Putting It Together

**Representational State Transfer** means:

> **Transferring a representation of a resource's current state between a client and a server.**

### Real-World Example

Imagine an employee record in a database:

```text
Database:
Employee ID: 101
Name: Kishore
Status: Active
```

When your frontend requests:

```http
GET /employees/101
```

The server sends:

```json
{
  "id": 101,
  "name": "Kishore",
  "status": "Active"
}
```

* **Representation** → JSON response
* **State** → Employee's current data
* **Transfer** → Server sends that representation to the client

**Interview one-liner:**

> "Representational State Transfer means transferring a representation of a resource's state between a client and server, typically using HTTP and formats such as JSON."


---

## What is Stateless?

**Stateless** means the server **does not store client-specific session information between requests**. Each request must contain all the information the server needs to process it.

### Simple Example

Imagine you are using an employee API.

**Request 1: Login**

```http
POST /login
```

```json
{
  "username": "kishore",
  "password": "12345"
}
```

The server validates the credentials and returns a token:

```json
{
  "access_token": "abc123"
}
```

**Request 2: Get employee details**

```http
GET /employees/101
Authorization: Bearer abc123
```

The server uses the token to identify and authorize the request. It does not need to remember that you logged in through a previous request's session.

### Stateful vs Stateless

| Stateful                                         | Stateless                                                      |
| ------------------------------------------------ | -------------------------------------------------------------- |
| Server remembers client session information      | Server does not maintain client session state between requests |
| Later requests may depend on stored session data | Each request contains the required context                     |
| Often uses server-side sessions                  | Commonly uses tokens such as JWT                               |
| Session storage may create scaling complexity    | Easier to distribute requests across multiple servers          |

---

REST APIs can use **both HTTP and HTTPS**.

### HTTP vs HTTPS in REST

| Protocol  | Meaning                            | REST API usage                                         |
| --------- | ---------------------------------- | ------------------------------------------------------ |
| **HTTP**  | HyperText Transfer Protocol        | Can be used for REST APIs                              |
| **HTTPS** | HyperText Transfer Protocol Secure | Can be used for REST APIs with encrypted communication |

### Example

**Using HTTP:**

```http
http://example.com/employees
```

**Using HTTPS:**

```http
https://example.com/employees
```

Both can follow REST principles and use methods such as:

```http
GET     /employees
POST    /employees
PUT     /employees/101
DELETE  /employees/101
```

### Which one is used in real applications?

**HTTPS is preferred for production APIs** because it encrypts data exchanged between the client and server, including authentication tokens and sensitive information.

### Interview Answer

> "REST is an architectural style, not a communication protocol. REST APIs commonly use HTTP or HTTPS. HTTPS is preferred in production because it provides encryption and secure communication."


---

## What are OSI and TCP/IP Models?

Both **OSI** and **TCP/IP** are **networking models**. They explain how data travels from one device to another over a network, such as when your browser communicates with a FastAPI server.

They divide network communication into **layers**, where each layer has a specific responsibility.

---

## 1. OSI Model

**OSI** stands for **Open Systems Interconnection**.

It is a **7-layer conceptual reference model** created by the International Organization for Standardization (ISO) to explain how network communication works.

### OSI 7 Layers


```text
┌──────────────────────────────┐
│ 7. Application               │ ← HTTP, DNS, SMTP
├──────────────────────────────┤
│ 6. Presentation              │ ← Data format, encryption concepts
├──────────────────────────────┤
│ 5. Session                   │ ← Session management
├──────────────────────────────┤
│ 4. Transport                 │ ← TCP, UDP
├──────────────────────────────┤
│ 3. Network                   │ ← IP, routing
├──────────────────────────────┤
│ 2. Data Link                 │ ← Ethernet, Wi-Fi frames
├──────────────────────────────┤
│ 1. Physical                  │ ← Cables, radio signals
└──────────────────────────────┘
```

### Each Layer Explained

| Layer | Name         | Main Responsibility                                   | Examples             |
| ----- | ------------ | ----------------------------------------------------- | -------------------- |
| **7** | Application  | Provides network services to applications             | HTTP, DNS, SMTP      |
| **6** | Presentation | Data representation, translation, encryption concepts | Encoding, encryption |
| **5** | Session      | Establishes and manages communication sessions        | Session management   |
| **4** | Transport    | End-to-end delivery, reliability, ports               | TCP, UDP             |
| **3** | Network      | Logical addressing and routing                        | IP, routers          |
| **2** | Data Link    | Local network delivery using frames and MAC addresses | Ethernet, Wi-Fi      |
| **1** | Physical     | Transmits raw bits over the medium                    | Cables, fiber, radio |

> **OSI is mainly used as a reference model to understand and troubleshoot networking.**

---

## 2. TCP/IP Model

The **TCP/IP model** is the practical networking model based on the **Internet protocol suite**. It describes how real-world internet communication works.

It is commonly represented using **4 layers**.

### TCP/IP 4 Layers


```text
┌──────────────────────────────┐
│ Application Layer            │ ← HTTP, DNS, FTP
├──────────────────────────────┤
│ Transport Layer              │ ← TCP, UDP
├──────────────────────────────┤
│ Internet Layer               │ ← IP, ICMP
├──────────────────────────────┤
│ Network Access Layer         │ ← Ethernet, Wi-Fi
└──────────────────────────────┘
```

### Each Layer Explained

| Layer              | Main Responsibility                                      | Examples              |
| ------------------ | -------------------------------------------------------- | --------------------- |
| **Application**    | Network services and application communication           | HTTP, HTTPS, DNS, FTP |
| **Transport**      | End-to-end communication, reliability, ports             | TCP, UDP              |
| **Internet**       | Addressing and routing packets                           | IP, ICMP              |
| **Network Access** | Communication over the local network and physical medium | Ethernet, Wi-Fi       |

---

## OSI vs TCP/IP

| Feature                | OSI Model                         | TCP/IP Model                                      |
| ---------------------- | --------------------------------- | ------------------------------------------------- |
| Full form              | Open Systems Interconnection      | Transmission Control Protocol / Internet Protocol |
| Number of layers       | **7**                             | **4** commonly                                    |
| Purpose                | Reference / conceptual model      | Practical internet networking model               |
| Developed by           | ISO                               | DARPA / Internet research community               |
| Session & Presentation | Separate layers                   | Included in Application                           |
| Physical & Data Link   | Separate layers                   | Combined as Network Access                        |
| Usage                  | Learning, design, troubleshooting | Real-world networking                             |

### Layer Mapping

| OSI Model       | TCP/IP Model       |
| --------------- | ------------------ |
| 7. Application  | **Application**    |
| 6. Presentation | **Application**    |
| 5. Session      | **Application**    |
| 4. Transport    | **Transport**      |
| 3. Network      | **Internet**       |
| 2. Data Link    | **Network Access** |
| 1. Physical     | **Network Access** |

---

## Example: Opening an HTTPS API

Suppose your frontend calls:

```http
GET https://api.example.com/employees/101
```

The data travels through the networking stack:

```text
Application Layer
HTTP request
       ↓
TLS security (HTTPS)
       ↓
Transport Layer
TCP
       ↓
Internet Layer
IP
       ↓
Network Access Layer
Wi-Fi / Ethernet
       ↓
Physical transmission
```

At the server, the process is reversed, and the response travels back.

### Easy Way to Remember

**OSI = 7 layers for understanding networking**

**TCP/IP = 4 layers used to explain practical internet communication**

### Interview Answer

> "The OSI model is a seven-layer reference model that explains network communication, from the physical transmission of bits to application-level protocols. The TCP/IP model is a practical four-layer model used by the Internet, consisting of Application, Transport, Internet, and Network Access layers. The TCP/IP model combines the OSI Session and Presentation layers into the Application layer and combines the Data Link and Physical layers into the Network Access layer."
