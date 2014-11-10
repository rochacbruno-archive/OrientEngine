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
    amount = IntProperty()
    
    
class Eater(OutEdgeInterface):
    action = 'eat'
    action_plural = 'eats'  # there is also get_plural_action and get_singular_action if it needs to be dynamic
    edge = Eat
    validators = []
    
    def link(self,):
        # optionally you can override the out link method
    
class Eatable(InEdgeInterface):
    related_names = ('eaters', 'who_eats')  # multiple terms can be used
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

>>> import orientengine
>>> orientengine.connect(uri="localhost:2424", db="animals", user="admin", password="12345")
>>> from vertices import Animal, Food
>>> rat = Animal(specie='rodent', name='rat')
>>> man = Animal(specie='human', name='man')
>>> pea = Food(color='green', name='pea')


# define some links (edges) 
# on each vertex
>>> rat.eat(pea)
>>> man.eat(pea, amount=30)

# or extending another vertex "in" connection

>>> pea.eaters.extend(rat, man)

# or a vertex can eat any foods at the same time
>>> man.eats([pea, corn, rice, tofu])


# Who eats peas?
# Starting by the pea Vertex
>>> pea.eaters
[<OrientRecord(rat)>, <OrientRecord(man)>, ...]

# or starting by Animal class
>>> Animal.objects(eats=pea) or Animal.objects(who_eats=pea)
[<OrientRecord(rat)>, <OrientRecord(man)>, ...]

```

