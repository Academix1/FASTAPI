## Day 1

### Basic Installations
Here's a comprehensive **Day 1 and Day 2** guide to **FastAPI CRUD operations** from scratch. We'll set up the FastAPI application, create a simple in-memory movie list, and perform CRUD operations.

---

### **Day 1: Installing and Introducing Python (venv and FastAPI)**

#### Step 1: Set up a virtual environment

1. **Create a new virtual environment:**
   ```bash
   python3 -m venv env
   ```

2. **Activate the virtual environment:**
   - For macOS/Linux:
     ```bash
     source env/bin/activate
     ```
   - For Windows:
     ```bash
     .\env\Scripts\activate
     ```

#### Step 2: Install FastAPI and Uvicorn

1. **Install FastAPI**:
   ```bash
   pip install fastapi
   ```

2. **Install Uvicorn** (for running the FastAPI app):
   ```bash
   pip install uvicorn
   ```

#### Step 3: Create a Basic FastAPI Application

Create a new Python file, `main.py`, and add the following content:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello, FastAPI!"}
```

#### Step 4: Run the FastAPI app

Run the FastAPI app with Uvicorn:

```bash
uvicorn main:app --reload
```

Now, visit `http://127.0.0.1:8000` in your browser, and you should see:
```json
{"message": "Hello, FastAPI!"}
```

---

### Day 2
- Day2 We Will Deal With Basic CRUD Operation

```fastapi
from fastapi import FastAPI
from pydantic import BaseModel

app=FastAPI()

class Item(BaseModel):
    name: str
    price: float

class ResponseModel(BaseModel):
    name: str


@app.get("/")
def root():
    return {"message": "Hello World"}


@app.get("/items/{item_id}")
def read_item(item_id:int):
    return {"item_id": item_id}

@app.get("/items")
def read_items(a,b):
    return {"a": a, "b": b}

@app.post("/item/")
def create_item(item: Item):
    return {"name": item.name, "price": item.price}

@app.post("/response/",response_class=ResponseModel)
def create_item(item: Item):
    return item
```
