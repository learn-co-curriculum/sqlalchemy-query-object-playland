
# SQLAlchemy Playland


```python
import pandas as pd
from sqlalchemy import *
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from sqlalchemy.sql import func
```


```python
Base = declarative_base()
```


```python
class Project(Base):
    __tablename__ = "projects"
    id = Column(Integer, primary_key=True)
    name = Column(String)
    category = Column(String)
    main_category = Column(String)
    currency = Column(String)
    deadline = Column(Date)
    goal = Column(Float)
    launched = Column(Date)
    pledged = Column(Float)
    state = Column(String)
    backers = Column(Integer)
    country = Column(String)
    
engine = create_engine('sqlite:///kickstarter.db')
```


```python
engine = create_engine('sqlite:///kickstarter.db')
chunks = pd.read_csv('./ks-projects-201801.csv')
chunks.to_sql(name='projects', if_exists='append', con=engine)

Base.metadata.create_all(engine)
```


```python
Session = sessionmaker(bind=engine)
session = Session()
```

## Query Challenges

1. query the database with sqlalchemy to find the project named 'Beach Wrestling Documentary'
    - set the variable `beach_wresting_documentary` equal to the result of this query
    - return `beach_wresting_documentary.goal`
        * 20000.0
    - return `beach_wresting_documentary.backers`
        * 36
    - return `beach_wresting_documentary.state`
        * successful
2. collect a list of projects whose states are equal to 'successful'
    - return the length of this list
3. collect a list of projects whose pledged amounts are greater than or equal to their goal amounts
    - check the length of this list of projects --> does the length of this list match the length of the list from (2)?
4. collect a list of `main_category`s and each of their respective average `pledge`s
    - **BONUS**: rewrite the query to order the returned list by highest average pledge to lowest


```python
# 1. beach_wresting_documentary
beach_wresting_documentary = session.query(Project).filter(Project.name=='Beach Wrestling Documentary').first()
```


```python
beach_wresting_documentary.goal
```




    20000.0




```python
beach_wresting_documentary.backers
```




    36




```python
beach_wresting_documentary.state
```




    'successful'




```python
# 2. length of projects with successful state
len(session.query(Project).filter(Project.state=='successful').all())
```




    133956




```python
# 3. num of projects with pledge >= goal
num_of_projs = session.query(Project).filter(Project.pledged >= Project.goal).all()
len(num_of_projs)
```




    137042




```python
# 4. average pledged amt for each main_category
from sqlalchemy.sql import func
average = session.query(Project.main_category, (func.avg(Project.pledged).label('average'))).group_by(Project.main_category).order_by(desc('average')).all()
```


```python
average
```




    [('Design', 27119.751279015596),
     ('Technology', 22586.16481377989),
     ('Games', 21865.173174193107),
     ('Comics', 6899.31118864957),
     ('Fashion', 6549.0318136395545),
     ('Film & Video', 6362.733852638248),
     ('Food', 5340.16326762052),
     ('Theater', 4097.224678823399),
     ('Music', 3992.7356015255427),
     ('Dance', 3690.7986836517994),
     ('Photography', 3664.6465766768615),
     ('Publishing', 3638.7163743291567),
     ('Art', 3606.970043689866),
     ('Journalism', 3218.085215562564),
     ('Crafts', 2016.1539470995554)]


