## **DBMS (Database Management System)**

DBMS is the umbrella term for all database systems. Software that allows you to store, retrieve, and manage data.

**DBMS types:**

* **RDBMS Relational DBMS** (SQL) → Uses SQL for structured data in tables.

    * Examples: PostgreSQL, MySQL, Oracle.


* **Non-Relational DBMS** (NoSQL) → Uses flexible models like documents, key-value, graphs.

  * Examples: MongoDB, Cassandra, Redis.



**Quick Summary Table**

| Feature      | SQL (Relational)      | NoSQL (Non-Relational)     |
| ------------ | --------------------- | -------------------------- |
| Data Model   | Tables (rows/columns) | Document, Key-Value, Graph |
| Schema       | Fixed                 | Flexible                   |
| Scalability  | Vertical              | Horizontal                 |
| Transactions | ACID                  | BASE                       |
| Best For     | Structured data       | Big data, real-time apps   |


***

### **Data Model**

* SQL:
    * Structured data stored in tables (rows & columns).
    * Schema is fixed and predefined.
        * Before inserting any data, you must define the structure of the table (columns, data types, constraints).
        * If you need to add, remove, or modify columns, you must explicitly change the schema.

* NoSQL:
    * Flexible data models: document, key-value, column-family, graph.
    * Schema is dynamic or schema-less.

***

### **Query Language**

* SQL:

  * Uses Structured Query Language (SQL) for queries.
  * Standardized and widely adopted.
    `SELECT * FROM employee;`
  * SQL: Works on tables → rows & columns.

* NoSQL:

    * No standard query language; queries depend on database type (e.g., MongoDB uses JSON-like queries).
        `db.employee.find({});`
    * NoSQL: Works on collections → documents (key-value pairs).


***

### **Scalability?**

Scalability means **how well a system can handle increased load** (more users, more data, more requests) by adding resources.
    
* Vertical Scaling (RDBMS)
* Horizontal Scaling (Non RDBMS)

**1. Vertical Scaling (SQL)**

*   **Definition:** Add more power (CPU, RAM, SSD) to a **single server**.
    *   **Example:**
        *   You have a PostgreSQL database running on a server with 8 cores and 16GB RAM.
        *   If traffic grows, you upgrade to 32 cores and 128GB RAM.
    *   **Pros:**
        *   Simple to implement.
        *   No changes in application logic.
    *   **Cons:**
        *   **Hardware limit**: You can’t upgrade forever.
        *   Expensive.
        *   Single point of failure.


**2. Horizontal Scaling (NoSQL)**

*   **Definition:** Add **more servers (nodes)** to share the load.
    *   **Example:**
        *   You have a MongoDB cluster with 3 nodes.
        *   When traffic grows, you add 5 more nodes.
        *   Data is **sharded** (split) across nodes, and queries are distributed.
    *   **Pros:**
        *   Scales almost infinitely.
        *   High availability (replication).
    *   **Cons:**
        *   Complex setup.
        *   Requires distributed system design.



**Why SQL struggles with horizontal scaling?**

*   SQL databases maintain **strong ACID consistency** and complex joins.
*   Distributing relational data across multiple servers is hard because:
    *   Joins across servers are expensive.
    *   Transactions across nodes are complex.


**Why NoSQL is designed for horizontal scaling?**

*   NoSQL databases often use **denormalized data** (store related info together).
*   They follow **BASE** principles (eventual consistency), making distribution easier.
*   Example: MongoDB, Cassandra, DynamoDB use **sharding** and **replication**.

**Example**

*   **Vertical scaling:** Upgrade your car engine → faster car, but still one car.
*   **Horizontal scaling:** Add more cars → deliver more packages in parallel.

***

### Transactions

* SQL:

    * Strong ACID compliance (Atomicity, Consistency, Isolation, Durability).

* NoSQL:

    * Often BASE (Basically Available, Soft state, Eventually consistent).
    * Some NoSQL DBs support ACID for specific operations.

**ACID (Used in SQL Databases)**

**ACID stands for:**

1.  **Atomicity**
    *   All parts of a transaction succeed or none do.
    *   Example: Transferring money → debit and credit happen together or not at all.

2.  **Consistency**
    *   Data remains valid according to rules and constraints.
    *   Example: If a column must be unique, the DB enforces it.

3.  **Isolation**
    *   Transactions don’t interfere with each other.
    *   Example: Two users updating the same record won’t corrupt data.

4.  **Durability**
    *   Once committed, data is saved even if the system crashes.
    *   Example: After a successful transfer, the record stays even after a power failure.

**SQL databases (PostgreSQL, MySQL)** follow ACID strictly for reliability.


**BASE (Used in NoSQL Databases)**

**BASE stands for:**

1.  **Basically Available**
    *   System guarantees availability even during failures.

2.  **Soft State**
    *   Data may change over time (not immediately consistent).

3.  **Eventually Consistent**
    *   Data will become consistent across nodes eventually, not instantly.

**NoSQL databases (MongoDB, Cassandra)** use BASE for scalability and performance.

**Key Difference**

*   **ACID:** Strong consistency, perfect for banking, ERP, financial apps.
*   **BASE:** High availability and scalability, perfect for social media, big data apps.

***

###  Use Cases

* SQL:
    * Complex queries, structured data, financial systems, ERP, CRM.


* NoSQL:
    * Big data, real-time analytics, IoT, social media, content management.

*** 

### Performance

* SQL:
  * Great for structured data and joins.
  * Can be slower for huge unstructured datasets.

* NoSQL:
  * Optimized for large-scale, high-speed operations.
  * No joins; data often denormalized.

***

### Database Acronyms & Mnemonics 

* **ACID**
* **BASE**
* **CAP**
* **CRUD**
* **DDL**
* **DML**

***


1. **ACID** (Atomicity, Consistency, Isolation, Durability)
Meaning:
- Ensures reliable transactions in SQL databases.
- Atomicity: All or nothing.
- Consistency: Data integrity maintained.
- Isolation: Transactions don't interfere.
- Durability: Changes persist after commit.
Example:
- Banking system: Transfer money · debit and credit happen together.
*** 
2. **BASE** (Basically Available, Soft state, Eventually consistent)
Meaning:
- Used in NoSQL databases for scalability.
- Basically Available: System remains available.
- Soft state: Data may change over time.
- Eventually consistent: Data syncs eventually.
Example:
- Social media likes: May take time to reflect across servers.
*** 
3. **CAP** (Consistency, Availability, Partition tolerance)
Meaning:
- CAP theorem for distributed systems.
- You can only guarantee two of the three.
Example:
- MongoDB prioritizes Availability and Partition tolerance over strict Consistency.
*** 
4. **CRUD** (Create, Read, Update, Delete)
Meaning:
- Basic operations for persistent storage.
Example:
- SQL: INSERT, SELECT, UPDATE, DELETE.
- MongoDB: insertOne(), find(), updateOne(), deleteOne().

***

5. **DDL** (Data Definition Language)

*   **Purpose:** Defines and manages the **structure** of the database (schema).
*   **Used for:** Creating, altering, and deleting tables or database objects.
*   **Common Commands:**
    *   `CREATE TABLE` → Create a new table.
    *   `ALTER TABLE` → Modify an existing table.
    *   `DROP TABLE` → Delete a table.
*   **Example:**

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50),
    department VARCHAR(50)
);
```

***

6. **DML** (Data Manipulation Language)

*   **Purpose:** Deals with **data inside tables** (insert, update, delete, retrieve).
*   **Used for:** Manipulating the actual records.
*   **Common Commands:**
    *   `SELECT` → Retrieve data.
    *   `INSERT` → Add new data.
    *   `UPDATE` → Modify existing data.
    *   `DELETE` → Remove data.
*   **Example:**

```sql
INSERT INTO employees (name, department) VALUES ('Alice', 'IT');
SELECT * FROM employees WHERE department = 'IT';
```

***

**Key Difference**

*   **DDL** = Structure (schema-level changes).
*   **DML** = Data (row-level changes).

***

#### Here’s a **real-world comparison** of where **SQL** and **NoSQL** 


### **Where SQL Outperforms (Joins & Complex Relationships)**

**Scenario:**  
You are building a **banking system** where:

*   Customers have multiple accounts.
*   Accounts have multiple transactions.
*   You need to generate a report:  
    *Customer Name → Account Details → Last 10 Transactions.*

**Why SQL is better here:**

*   SQL databases (PostgreSQL, MySQL) handle **complex joins** efficiently.
*   Example query:

```sql
SELECT c.name, a.account_number, t.amount, t.date
FROM customers c
JOIN accounts a ON c.id = a.customer_id
JOIN transactions t ON a.id = t.account_id
WHERE c.id = 101
ORDER BY t.date DESC
LIMIT 10;
```

*   ACID guarantees ensure **data consistency** (critical for financial apps).

***

###  **Where NoSQL Outperforms (High Scalability & Flexible Schema)**

**Scenario:**  
You are building a **social media app** where:

*   Each user has posts, comments, likes.
*   Data grows rapidly (millions of users).
*   Schema changes frequently (adding new features like reactions, stories).

**Why NoSQL is better here:**

*   NoSQL (MongoDB, Cassandra) stores **denormalized documents**:

```json
{
  "user_id": 101,
  "name": "Alice",
  "posts": [
    {
      "post_id": 5001,
      "content": "Hello World!",
      "likes": 120,
      "comments": [
        {"user": "Bob", "text": "Nice post!"}
      ]
    }
  ]
}
```

*   No joins needed → everything is in one document.
*   Horizontal scaling → add more servers easily.
*   BASE principles allow **eventual consistency**, which is fine for likes/comments.

***

### **Key Takeaways**

| Feature       | SQL (Relational)           | NoSQL (Non-Relational)       |
| ------------- | -------------------------- | ---------------------------- |
| Best For      | Banking, ERP, Inventory    | Social media, IoT, Big Data  |
| Strength      | Joins, ACID transactions   | Scalability, Flexible schema |
| Example Query | Complex JOIN across tables | Single document fetch        |

***