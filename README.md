OrientEngine
============

A Python Object-Graph-Document-Mapper for working with OrientDB 


> This project is a META-Project using the RDD (Readme Driven Development) process.


- Inspired by [MongoEngine](http://github.com/mongoengine) 
- Based in [PyOrient](https://github.com/rochacbruno/pyorient)


The idea 
========

There is already awesome projects to work with Python and OrientDB, BulbFlow + Rexter is the most used tools, but they use HTTP/REST interface which is great but sometimes is slow and unstable (depending on your stack). 

Using PyOrient Driver is really faster!

OrientEngine will provide **Vertex, Edge, Class and Cluster** classes to be extended in order to create Graph and Document database representation.

```python

#### edges.py
from orientengine import Edge, InEdgeInterface, OutEdgeInterface IntProperty

class Eat(Edge):
    amount = IntProperty
    
def Eater(OutEdgeInterface):
    edge = Eat
    validators = []
    
    def link(self,):
        # optionally you can override the out link method
    
def Eatable(InEdgeInterface):
    edge = Eat
    validators = []
    
    def link(self,):
        # optionally you can override the in link method
    
#### vertices.py

from orientengine import Vertex, StringProperty
from .edges import Eatable, Eater

class Animal(Vertex, Eater): # Can eat Out
    specie = StringProperty()
    name = StringProperty()

class Food(Vertex, Eatable): # Can be eated In
    color = StringProperty()
    name = StringProperty()

#### SHELL

>>> from vertices import Animal, Food
>>> rat = Animal(specie='rodent', name='rat')
>>> man = Animal(specie='human', name='man')
>>> pea = Food(color='green', name='pea')


# define some links (edges) 
# on each vertex
>>> rat.eats(pea)
>>> man.eats(pea, amount=30)

# or extending another vertex "in" connection

>>> pea.eaters.extend(rat, man)


# Who eats peas?
# Starting by the pea Vertex
>>> pea.eaters
[<OrientRecord(rat)>, <OrientRecord(man)>, ...]

# or starting by Animal class
>>> Animal.objects(eats=pea)
[<OrientRecord(rat)>, <OrientRecord(man)>, ...]

```

