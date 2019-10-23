# PySmores

The hook: your database operations have been simplified to .get(), .save(), and .delete(), no serialization or other boilerplate necessary. It's Python objects all the way down. Intrigued? Read on. 

Note: PySmores is not production-ready and is being actively improved. When it reaches a highly stable state, I'll create a PyPi package for easy import. Until then, use at your own risk.  

PySmores is a layer on top of [Marshmallow](https://github.com/marshmallow-code/marshmallow), an ORM (Object-Relational Mapping) library that interfaces between Python and MongoDB. PySmores simplifies the process, minimizes the amount of boilerplate required, and allows you to focus on writing code rather than dealing with database interactions. 

Similar to Marshmallow, PySmores requires that you define a Schema for each class in addition to the class itself. The only major differences are that in the class definition, you 
- Inherit from PySmores
- Define which collection you want to save the objects to 
- Instantiate a schema, preferably using instantiateSchemaWithJit

```
from PySmores import PySmores
class Car(PySmores): # Inherit from PySmores
  collection = "Cars" # Name of the collection where we will save off Car objects 
  schema = PySmores.instantiateSchemaWithJit(CarSchema) # Assumes your CarSchema is in the same file, which avoids cyclical imports 
```

This allows you to save new objects with Car.save(myCar) as well as updating (WARNING: this will always overwrite) objects that have already been saved to the database using the same syntax, Car.save(myCar) -  serialization is taken care of for you. 

Get ready-to-go Python objects by id or criteria via Car.get(id = carId) and Car.get(criteria = {"field" : "value"}}; deserialization also taken care of courtesy of PySmores. 

Car.getAll({"genericField" : "genericValue"}) will get you a list of objects representing all the documents that match the criteria.

Car.delete(myCar) does exactly what the label says, assuming the id on myCar exists in the collection - kiss that sucker goodbye. 

**For a more in-depth example that shows off the minimalism of PySmores, check out Example.py** 

---

I'm planning to create a video tutorial soon to walk through how to set up PySmores and use it effectively in your applications, stay tuned! 

For most questions regarding setting up your models and schemas, you'll want to check out the [Marshmallow documentation](https://marshmallow.readthedocs.io/en/stable/index.html). If the question has to do with saving/loading/deleting, it's probably something I need to fix, so please leave an issue that I can investigate! 
