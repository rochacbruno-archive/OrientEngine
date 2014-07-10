OrientEngine
============

A Python Object-Graph-Document-Mapper for working with OrientDB 


> This project is a META-Project in pre-alpha version, do not use it yet, contributions are very welcome!


- Inspired by [MongoEngine](http://github.com/mongoengine) 
- Based in a fork of [PyOrient](https://github.com/rochacbruno/pyorient)


The idea 
========

There is already awesome projects to work with Python and OrientDB, BulbFlow + Rexter is the most used tools, but they use HTTP/REST interface which is great but sometimes is slow and error prone. 

Using the binary access via [liborient](https://github.com/dam2k/liborient) OrientEngine will provide the way to model orient classes as Python classes.

OrientEngine will provide **Vertex, Edge, Class and Cluster** classes to be extended in order to create Graph and Document database representation.

## Some syntax examples, not defined which one will be implemented.

Example 1 
============================


# Modeling 

### vertices.py
```python

from orientengine import Vertex
from orientengine.properties import StringProperty, IntegerProperty, BooleanProperty


class Animal(Vertex):
    specie = StringProperty()
    endangered = BooleanProperty()
    

class Food(Vertex):
    color = StringProperty()
    
    
class Environment(Vertex):
    size = IntegerProperty()
    
```

### edges.py
```python
from orientengine import Edge
from orientengine.properties import IntegerProperty, DateTimeProperty


class Eat(Edge):
    amount = IntegerProperty()
    
    
class Live(Edge):
    since = DateTimeProperty()

```


# Creating instances

### controller.py
```python
from datetime import datetime
from .vertices import Animal, Food, Environment
from .edges import Eat, Live


# create some instances (vertices)

rabbit = Animal(specie='lagomorph', endangered=False)
dog = Animal(specie='canis lupus familiaris', endangered=False)
quokka = Animal(specie='marsupial', endangered=True)


carrot = Food(color='yellow')
lettuce = Food(color='lightgreen')
apple = Food(color='red')

forest = Environment(size=10000)
city = Environment(size=30000)
beach = Environment(size=5500000)


# define some links (edges) 

rabbit.out(Eat(amount=70), carrot)
rabbit.out(Eat(amount=30), lettuce)
rabbit.out(Live(since=datetime.now), forest)

dog.out(Eat(amount=10), carrot)
dog.out(Live(since=datetime.now), city)

quokka.out(Eat(amount=100), apple)
quokka.out(Live(since=datetime.now), beach)


```


IN the example above, each EDGE are being created by the **Vertex.out** method, but there is a way to create **EdgeManagers** in each vertex.


### managers.py
```python

from orientengine import EdgeManager, Vertex
from .edges import Eat, Live

class Eatable(EdgeManager, Vertex):
    def eats(self, vertex, **kwargs):
        return self.out(Eat(**kwargs), vertex)
        

class Livable(EdgeManager, Vertex):
    def lives_in(self, vertex, **kwargs):
        return self.out(Live(**kwargs), vertex)
        
        
```

Now use that managers as mixins on your vertices

### vertices.py
```python
from orientengine import Vertex
from orientengine.properties import StringProperty, IntegerProperty, BooleanProperty
from .managers import Eatable, Livable


class Animal(Eatable, Livable):
    specie = StringProperty()
    endangered = BooleanProperty()
    

class Food(Vertex):
    color = StringProperty()
    
    
class Environment(Vertex):
    size = IntegerProperty()
    
```

Now **Animal** extends Eatable and Livable and to connect the edges will easily to connect the edges.

### controller.py
```python
from datetime import datetime
from .vertices import Animal, Food, Environment
from .edges import Eat, Live


# create some instances (vertices)

rabbit = Animal(specie='lagomorph', endangered=False)

carrot = Food(color='yellow')
lettuce = Food(color='lightgreen')

forest = Environment(size=10000)


# define some links (edges) 

rabbit.eats(carrot, amount=70)
rabbit.eats(lettuce, amount=30)
rabbit.lives_in(forest, since=datetime.now)


```

# Browsing

OrientDB supports SQL like language and it can be used through orientengine in two forms:

### Raw Queries

```python

from orientengine import Query

query = Query("SELECT expand( out() ) from Animal where name = 'Rabbit'")
result = query.run(connection)

```

### orientengine objects lookup

```python

from vertex import Animal, Food

carrot = Food.objects.get(name='carrot')
carrot_eaters = Animal.objects.filter(Eat_out=carrot)

```


Example 2
==========

This example takes the same syntax to create the vertices and edges, but uses a bulbflow inspired way to connect edges and query. The problem with this approach is that we lose the track of edge creation (we could do it by overriding **out** method or creating **EdgeManager**, but now we need to use **Blinker** to manage this with signals only.


# Modeling 

### vertices.py
```python

from orientengine import Vertex
from orientengine.properties import StringProperty, IntegerProperty, BooleanProperty


class Animal(Vertex):
    specie = StringProperty()
    endangered = BooleanProperty()
    

class Food(Vertex):
    color = StringProperty()
    
    
class Environment(Vertex):
    size = IntegerProperty()
    
```

### edges.py
```python
from orientengine import Edge
from orientengine.properties import IntegerProperty, DateTimeProperty


class Eat(Edge):
    amount = IntegerProperty()
    
    
class Live(Edge):
    since = DateTimeProperty()

```


# Creating instances

### controller.py
```python
from datetime import datetime
from orientengine import Orient
from .vertices import Animal, Food, Environment
from .edges import Eat, Live


graph = Orient('remote:localhost/animals_database', user='admin', password='admin')
graph.connect()

# create some instances (vertices)

rabbit = Animal(specie='lagomorph', endangered=False)
dog = Animal(specie='canis lupus familiaris', endangered=False)
quokka = Animal(specie='marsupial', endangered=True)


carrot = Food(color='yellow')
lettuce = Food(color='lightgreen')
apple = Food(color='red')

forest = Environment(size=10000)
city = Environment(size=30000)
beach = Environment(size=5500000)


# define some links (edges) 

graph.connect(rabbit, Eat(), carrot)
graph.connect(rabbit, Eat(), lettuce)
graph.connect(rabbit, Live(), forest)
...

```

That simplifies a bit the edge creation, but now we do not have any method to track it, so need to use a signal handler.


Suggestions
============

Have a suggestion or more ideas to the sintax? please comment on an issue.





