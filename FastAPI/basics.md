### Basics of FastAPI

```
from fastapi import FastAPI
app = FastAPI()
```

The first step in creating a FastAPI app is to declare the application object of FastAPI class.

----

The uvicorn server uses this object to listen to clients request.
```
@app.get("/")
async def root():
   return {"message": "Hello World"}
```

- `@app.get("/")`
   - This is called a decorator in Python.
     - It tells FastAPI:
     -  “Whenever a client sends a GET request or request any other method (Post, Patch, Put, Delete) to the / (root URL), run the function below.”

**Key parts:**

- app → your FastAPI application instance
(Usually created earlier like: app = FastAPI())
- .get(...) → specifies HTTP method (GET request)
- "/" → the route/path, meaning the homepage

---

`async def root():`

# Async & Await in FastAPI — Complete Notes

## 1. Is `async` mandatory?
No.
You can write FastAPI routes using either:

```python
def root():
    return {"message": "Hello"}
```

or

```python
async def root():
    return {"message": "Hello"}
```

Both work.

***

## 2. What is a normal `def`?

```python
def root():
    return {"message": "Hello"}
```

Synchronous (Sync)

* Executes line by line.
* Waits until current work finishes.
* Blocking in nature.

**Answer:** Yes, `def` is synchronous by default.

***

## 3. What is `async def`?

```python
async def root():
    return {"message": "Hello"}
```
Asynchronous function.

Used when the function may need to wait for:

* Database calls
* API calls
* Network operations
* File operations

***

## 4. Why use `async`?

### Banking Example

Customer A transfers money.

Backend needs to:

```python
balance = await check_balance()
```

Database takes 2 seconds.

Instead of waiting idle:

* Server pauses Customer A's request.
* Serves Customer B, C, and D.
* Returns to Customer A when the database responds.

This improves scalability.

***

## 5. What is `await`?

```python
balance = await check_balance()
```

Meaning:

> "Wait for the result, but don't block the entire server while waiting."

The current function pauses.

Other requests continue processing.

***

## 6. Can `await` be used without `async`?
 
No.

Invalid:

```python
def get_balance():
    balance = await db_call()
```

Error:

```text
SyntaxError: 'await' outside async function
```

***

## 7. Can `async` be used without `await`?

Yes.
```python
async def root():
    return {"message": "Hello"}
```

This is valid.

But in most cases there is little benefit if no async operation exists.

***

## 8. Relationship between Async and Await

```text
async def  ---> can exist without await 

await      ---> cannot exist without async def 
```

Memory trick:

```text
async = permission to use await

await = actual waiting statement
```

***

## 9. Banking Use Case

### Transfer Money API

```python
async def transfer_money():
    balance = await check_balance()
    beneficiary = await verify_beneficiary()
    await save_transaction()
    await send_sms()
```

### Why Await?

While:

```python
await check_balance()
```

is waiting for database response,

the server can serve:

* Customer B → Balance Enquiry
* Customer C → Mini Statement
* Customer D → Transaction History

***

## 10. Does Async Handle Multiple Tasks for Same User?

Partially yes, but the main goal is:

- Handle many requests efficiently.

Example:

```text
Customer A → Waiting for DB
Customer B → Checking balance
Customer C → Viewing statement
```

When Customer A's request reaches:

```python
await check_balance()
```

it pauses.

FastAPI works on B and C.

Once the database responds, A's request resumes.

***

## 11. When Should You Use `async`?

- Database operations

```python
await db.fetch()
```

- External API calls

```python
await api_call()
```

- File operations

```python
await read_file()
```

- Network operations

```python
await send_data()
```

***

## 12. When Should You Use Normal `def`?

For CPU-intensive work:

```python
def calculate_interest():
    ...
```

```python
def generate_report():
    ...
```

```python
def perform_calculations():
    ...
```

`async` does not make calculations faster.

It helps only when waiting for I/O operations.

***

# Interview-Friendly Summary

* `def` = Synchronous function.
* `async def` = Asynchronous function.
* `await` can only be used inside `async def`.
* `async def` can exist without `await`.
* `await` pauses the current function without blocking the server.
* Use `async` for database/API/network/file operations.
* Use normal `def` for CPU-heavy calculations.

------

