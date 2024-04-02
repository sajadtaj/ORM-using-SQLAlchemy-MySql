**Introduction**

This guide provides a step-by-step introduction to Object-Relational Mapping (ORM) using SQLAlchemy, a powerful Python library that simplifies interaction with relational databases. You'll learn how to:

* Connect to a database
* Define models to represent database tables
* Perform Create, Read, Update, and Delete (CRUD) operations
* Filter data using queries
* `Send Pandas dataframe to Table`

**Prerequisites**

* Python (version 3.6 or later recommended)
* A relational database (e.g., MySQL, PostgreSQL, SQLite)
* Basic understanding of SQL concepts

**Installation**

1. Install SQLAlchemy using pip:
   Bash

   ```
   pip install sqlalchemy
   ```
2. Install a database driver specific to your database system (e.g., `mysql-connector-python` for MySQL). Refer to the documentation for your database for installation instructions.
3. CAll Packeges

```python
#Python
from sqlalchemy import create_engine,Column,Integer,String,DateTime,ForeignKey
from typing import List,Optional
from datetime import datetime
from sqlalchemy.orm import DeclarativeBase
from sqlalchemy.orm import Mapped
from sqlalchemy.orm import mapped_column
from sqlalchemy.orm import relationship
from sqlalchemy.orm import Session
from sqlalchemy.orm import sessionmaker
```

**Step 1: Connect to the Database**

```python
#Python
# Replace with your database connection details
engine = create_engine('mysql+pymysql://root:your_password@localhost:3306/your_database_name')
```

**Step 2: Define Models**

* Use SQLAlchemy's declarative base classes to define models that represent your database tables.
* Each model class maps to a table, and each column in the table is represented by an attribute in the model class.

```python
#Python
Base = declarative_base()

class User(Base):  # Adjust the class name and column definitions to match your table
    __tablename__ = 'users'  # Adjust the table name if needed

    id = Column(Integer, primary_key=True)
    name = Column(String(80))
    email = Column(String(120), unique=True)
    created_at = Column(DateTime, default=datetime.utcnow)

    def __repr__(self):
        return f"<User(id={self.id}, name='{self.name}', email='{self.email}')>"
```

* create Table: (`Optional`)
  If table dosnt exist, first create the Table

```python
#Python
Base.metadata.create_all(engine)
```

**Step 3: Create a Session**

* A session object is used to interact with the database and perform CRUD operations.

```python
#Python
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
```

**Step 4: Create (CRUD)**

```python
#Python
with SessionLocal() as db:
    new_user = User(name="Alice", email="alice@example.com")
    db.add(new_user)
    db.commit()  # Save changes to the database
```

**Step 5: Read (CRUD)**

```python
#Python
with SessionLocal() as db:
    user = db.query(User).filter(User.id == 1).first()  # Get user by ID
    if user:
        print(f"User found: {user}")
    else:
        print("User not found.")
```

**Step 6: Update (CRUD)**

```python
#Python
with SessionLocal() as db:
    user = db.query(User).filter(User.id == 1).first()
    if user:
        user.email = "alice.updated@example.com"
        db.commit()
        print(f"User updated: {user}")
    else:
        print("User not found.")
```

**Step 7: Delete (CRUD)**

```python
#Python
with SessionLocal() as db:
    user = db.query(User).filter(User.id == 1).first()
    if user:
        db.delete(user)
        db.commit()
        print(f"User deleted (ID: {user.id})")
    else:
        print("User not found.")
```

**Step 8: Filter Data**

* Use SQLAlchemy's query API to filter data based on specific conditions.

```python
#Python
with SessionLocal() as db:
    users_named_alice = db.query(User).filter(User.name == "Alice").all()
    print("Users named Alice:")
    for user in users_named_alice:
        print(user)

    users_with_example_email = db.query(User).filter(User.email.like("%example.com%")).all()
    print("\nUsers with email containing example.com")
    for user in users_with_example_email:
        print(user)
```

## `Send Pandas dataframe to Table`

**Step 1: define the Table**

```python
#Python
class Base(DeclarativeBase):
        pass

# Create Models : Table
class BTC_UCD(Base):
    __tablename__ = 'BTC_UCD'
    id = Column(Integer, primary_key=True)
    Open     = Column(Float)
    High     = Column(Float)
    Low      = Column(Float)
    Close    = Column(Float)
    AdjClose = Column(Float)
    Volume   = Column(BIGINT)
    date     = Column(DateTime)

    def __repr__(self):
        return f"< BTC_UCD(date='{self.date}', Close='{self.Close}')>"

```

**Step 2: Create Engine or (optional Create the Table if dosn exist)**

```python
#python
engine = create_engine('mysql+pymysql://root:your_password@localhost:3306/your_database_name')
Base.metadata.create_all(engine) #(optional Create the Table if dosn exist)
```

**Step 3: Crawl The Data**

```python
#python

BTC-USD = yf.download(tickers='BTC-USD', start="2020-01-01", end="2024-03-29",ignore_tz=True)
```

| Date       | Open       | High       | Low        | Close      | Adj Close  | Volume     |
| ---------- | ---------- | ---------- | ---------- | ---------- | ---------- | ---------- |
| 2015-01-01 | 320.434998 | 320.434998 | 314.002991 | 314.248993 | 314.248993 | 3414248993 |
| 2015-01-02 | 314.079010 | 315.838989 | 313.565002 | 315.032013 | 315.032013 | 5530320913 |
| 2015-01-03 | 314.846008 | 315.149994 | 281.082001 | 281.082001 | 281.082001 | 5681082001 |
| 2015-01-04 | 281.145996 | 287.230011 | 257.612000 | 264.195007 | 264.195007 | 4344195007 |

**Step 4:Iterate and Create Objects**
Alternatively, you can iterate through the DataFrame and create individual model objects based on each row:

```python
#python
data_objects = []
for index, row in BTC_USD_df.iterrows():
    user_object = BTC_UCD(
                    Open     = row['Open'],
                    High     = row['High'],
                    Low      = row['Low'],
                    Close    = row['Close'],
                    AdjClose = row['Adj Close'],
                    Volume   = row['Volume'],
                    date     = index
        )  # Create object with row values
    data_objects.append(user_object)
```

**Step 4:Insert Data into Database**

```python
#python
with SessionLocal() as db:
    # Using list of model objects
    db.add_all(data_objects)

    db.commit()  # Commit changes to the database
```
