# ----------------------------------------------------------------------------------------------------------------------
# Schema definition
# ----------------------------------------------------------------------------------------------------------------------

# Import Marshmallow modules
from marshmallow import Schema, fields, post_load

# Import nested object schemas from elsewhere in your project
from models.Owner import OwnerSchema

# Every first-level schema should inherit from Schema, though if you have schemas that inherit from CarSchema, they should inherit
# CarSchema instead of schema
class CarSchema(Schema):
    id = fields.String() # Required by PySmores (for now, working on using the BSON object _id instead)
    currentOwner = fields.Nested(Driver) # Like all fields, Nested comes from Marshmallow and allows us to nest objects under each other
    # For dictionaries, you can optionally enforce key and value types, including Nested. 
    previousOwners = fields.Dict(keys = fields.Date(allow_none = False), values = fields.Nested(OwnerSchema, allow_none= False))
    year = fields.Integer()  # See the Marshmallow documentation (called out in the README) for details on other fields 

    # All Marshmallow objects need to be able to create themselves. passing data as a set of key arguments using **data will always suffice. 
    # The name of this function can be whatever you want, though I suggest using something obvious and universal like create_<objectName>
    @post_load
    def create_car(self, data):
        return Car(**data)


# ----------------------------------------------------------------------------------------------------------------------
# Class definition
# ----------------------------------------------------------------------------------------------------------------------

# Always import PySmores :) 
from PySmores import PySmores 

# Every first-level object should inherit from PySmores, though if you have objects that inherit from Car, they should inherit
# Car instead of PySmores 
class Car(PySmores):
    collection = "Cars" # Name of the collection where we will save off Car objects 
    schema = PySmores.instantiateSchemaWithJit(CarSchema) # Makes your database operations ~fast~. If you don't use instantiateSchemaWithJit,
    # you still have to set schema equal to a valid schema instance, i.e. CarSchema(). I highly advise using the instantiateSchemaWithJit 
    # call though, as it is faster, future-proof, and there's really no reason to not. 

    # Constructor should include a parameter for each attribute you're saving/loading. Feel free to leave null-esque default values
    # if you plan on creating these without having all the data at the time of instantiation. 
    def __init__(self, id = '', previousOwners = none, year = ''):
        # Assign the values passed in to attributes to create your object 
        self.id = id
        self.previousOwners = previousOwners or {} # If we're passed None, make this an empty dict so we don't have to check for None at any other time 
        self.year = year
        
        
# ----------------------------------------------------------------------------------------------------------------------
# Pretend this is the main body of your code and not in a Model 
# ----------------------------------------------------------------------------------------------------------------------
from models.Example import Car 

# Make a new car. Note that we DON'T use an id - PySmores will get one and fill in the field automatically when we save the object to the db.
firstCar = Car(year = 1998)

# Good enough for now. Ready for the magic? Let's save this to the database. It'll automatically be saved to the "Cars" collection 
# because we specified collection = "Cars" in the model class. 
firstCarId = Car.save(firstCar) # Yep, that's it! The id is returned from save() if we want to use it later. 
# Note: PySmores checks for a value in id when we save(). If it's blank, it creates a new document.
# Technically, it'll also create a new document if you pass an id that doesn't match any document, but that implies you used 
# a custom id, which is liable to break on future use of PySmores. Let MongoDB assign an id. 
# If the id matches that of another document, we will overwrite that document (see below). 

# Let's get that car back from the database and save it off to a different variable for the sake of example:
myCar = Car.get(id = firstCarId) # gets you a ready-to-go Python object from the data for the document with id = carId in your database

# You add some previousOwners and want to save it back. Same syntax as before! 
Car.save(myCar) # Since the id will match that of firstCar, we'll overwrite firstCar with the data in myCar. 

# Nevermind, hate that piece of trash. Delete it. 
Car.delete(myCar) # PySmores will grab the id from the object and delete it from the collection.

# Maybe year is unique field (ignoring the unrealisticness of that example), and we want to get the car that was made in 1998.
# This is basically regular use of MongoDB via PyMongo: 
oldCarCriteria = {"year" : 1998}
oldCar = Car.get(criteria = oldCarCriteria)
# We're just passing a dictionary, so you can define the dict within the parameter if you prefer
oldCar = Car.get({"year" : 1998}) 

# Or maybe we want to get all cars that were made in 1998:
oldCars = Car.getAll({"year" : 1998})


