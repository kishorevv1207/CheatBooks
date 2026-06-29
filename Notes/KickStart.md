### Windows:

```
python -m pip install --no-deps -r requirements.txt

pip freeze | % { pip uninstall -y $_.Split('==')[0] }

python -c "import sys; print(sys.executable)"

py -0

py -3.10 -m venv myenv

python -c "import sys; print(sys.executable)"

python -m pip install --upgrade

python -m pip check

python -m pip list

python -m pip show 

python --version

```
----

### **Python Versions Strucutre**

- `[MAJOR.MINOR.PATCH]`

***Examples***

 `Python 3.13.9`

- 3 → Major version
- 13 → Minor version
- 9 → Patch version

----

### Project Structure

*  One **Git account (like GitHub/GitLab)**
*  Inside it → **many repositories (repos)**
*  Each repo → **different Python version** (3.13, 3.14, etc.)
*  Each repo → **independently deployed**

* *This kind of System is built using **microservices or modular services**

***

### Big Picture Architecture

```
System = Multiple small services (repos)

Repo A (Python 3.13) → User service
Repo B (Python 3.14) → Payment service
Repo C (Python 3.12) → Notification service
```

Each repo:

* Works **independently**
* Has its **own Python version**
* Has its **own deployment pipeline**

***

### How All Repos Work Together

They don’t combine at code level — they connect via **APIs / communication**

### Example Flow

```
User → API Gateway → Repo A (User Service)
                          ↓
                      Calls Repo B (Payment Service)
                          ↓
                      Calls Repo C (Notification Service)
```

**Communication happens via:**

* REST APIs (HTTP)
* gRPC
* Message queues (Kafka, RabbitMQ)

 Python version differences don't matter  
 Because they comunicate through **network**, not direct code import

***

### Why Different Python Versions Work Fine

Because each repo runs in its own environment:

### Each repo has:

*  Virtual environment / Docker container
*  Own dependencies
*  Own Python version

Example:

```
Repo A:
  Python 3.13
  Flask

Repo B:
  Python 3.14
  FastAPI
```

- They are isolated → no conflict

***

##  Deployment Process (Simple)

Each repo has its own pipeline:

## Step 1: Developer makes change

* Push code to repo

## Step 2: CI/CD pipeline runs

* Install correct Python version
* Install dependencies
* Run tests

## Step 3: Build & deploy

* Docker image OR server deployment
* Deploy to environment (dev / staging / prod)

 This happens **independently for each repo**

***

## Example Deployment Flow

### Repo A updated:

```
Code change → CI/CD → Deploy User Service
```

### Repo B updated:

```
Code change → CI/CD → Deploy Payment Service
```

- No need to deploy everything together

***

##  How They Stay Connected After Deployment

Even after independent deployments:

* Each service has a **URL / endpoint**

Example:

```
User Service → calls → http://payment-service/api/pay
```

 As long as APIs don’t break, system works

***

## Important Rules in This Setup

### 1. API contract must be stable

* If Repo B changes API → Repo A may break

### 2. Versioning APIs

Example:

```
/api/v1/pay
/api/v2/pay
```

### 3. Use environment configs

* URLs
* credentials
* service endpoints

***
