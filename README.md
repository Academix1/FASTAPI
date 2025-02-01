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

```python
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
### Day 3

#### Database
```python
# database.py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

# SQLite is built into Python, no need to install any package.
DATABASE_URL = "sqlite:///./test.db"  # The database file will be created in the current directory

# Set up the database engine
engine = create_engine(DATABASE_URL, connect_args={"check_same_thread": False})

# Create a SessionLocal instance to interact with the DB
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Create a base class for our models
Base = declarative_base()
```
#### Models

```python
# models.py
from sqlalchemy import Column, Integer, String, Float
from database import Base

class Movie(Base):
    __tablename__ = "movies"  # Table name in the database

    id = Column(Integer, primary_key=True, index=True)
    title = Column(String, index=True)
    director = Column(String)
    release_year = Column(Integer)
    poster_url = Column(String)  # URL to the movie poster
    price = Column(Float)

```

#### Schemas

```python
# schemas.py
from pydantic import BaseModel

# Base schema for common movie fields
class MovieBase(BaseModel):
    title: str
    director: str
    release_year: int
    poster_url: str
    price: float

# Schema for creating a movie (POST request)
class MovieCreate(MovieBase):
    pass

# Schema for the movie response (GET request)
class Movie(MovieBase):
    id: int  # ID generated by the database

    class Config:
        orm_mode = True  # Tells Pydantic to read the ORM data correctly
```

#### main.py

```python
from fastapi import FastAPI, Depends, HTTPException, status
from sqlalchemy.orm import Session
import models, schemas
from database import SessionLocal, engine

# Create the database tables if they don't already exist
models.Base.metadata.create_all(bind=engine)

app = FastAPI()

# Dependency to get the database session
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# Route to create a new movie (POST request)
@app.post("/movies/", response_model=schemas.Movie, status_code=status.HTTP_201_CREATED)
def create_movie(movie: schemas.MovieCreate, db: Session = Depends(get_db)):
    db_movie = db.query(models.Movie).filter(models.Movie.title == movie.title).first()
    if db_movie:
        raise HTTPException(status_code=status.HTTP_400_BAD_REQUEST, detail="Movie already exists")
    
    # Create a new movie instance
    db_movie = models.Movie(
        title=movie.title,
        director=movie.director,
        release_year=movie.release_year,
        poster_url=movie.poster_url or "",  # Allow poster_url to be empty
        price=movie.price
    )
    
    # Add to database session and commit
    db.add(db_movie)
    db.commit()
    db.refresh(db_movie)  # Refresh to get the generated ID

    # Return the newly created movie
    return db_movie


```

