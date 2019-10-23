from bson.objectid import ObjectId
import toastedmarshmallow

class PySmores(object):
    # main.py populates this static class variable with the instance of __mongo we're using
    __mongo= None
    # Access point for establishing PySmores's persistent mongo connection. Only allow if we don't have one yet.
    @staticmethod
    def setMongo(mongoConnection):
        if PySmores.__mongo is None:
            PySmores.__mongo = mongoConnection
        else:
            print("We already have a mongo connection")
            return ValueError


    # collection must be defined in the child class.
    # Needs to be the string that is the name of the collection.
    collection= None
    # schema must be defined in the child class.
    # Needs to be an instantiated schema of the child class.
    # Call into PySmores.instantiateSchemaWithJit from the child class to use toastedMarshmallow
    schema= None

    # ----------------------------------------------------------------------------------------------------------------------
    # CRUD-esque operations
    # ----------------------------------------------------------------------------------------------------------------------


    # Criteria should be a dictionary of attribute name:value pairs
    # If an Id is passed, the criteria parameter will be ignored
    # * is not a real parameter, but forces anything after it to be passed by name. so we don't allow get(<id>), you have
    # to specify get(id= <id>).
    @classmethod
    def get(childClass, *, id= "", criteria= None):
        if id is not "":
            json = getattr(PySmores.__mongo, childClass.collection).find_one({"_id": ObjectId(id)})
        elif criteria is not None:
            json = getattr(PySmores.__mongo, childClass.collection).find_one(criteria)
        else:
            return None

        if json is None:
            return None

        object = childClass.schema.load(json)
        return object


    # Criteria should be a dictionary of attribute name:value pairs
    @classmethod
    def getAll(childClass, criteria = None):
        __mongo__Cursor = getattr(PySmores.__mongo, childClass.collection).find(criteria)

        objectList = []
        for jsonObject in __mongo__Cursor:
            objectList.append(childClass.schema.load(jsonObject))
        return objectList


    @classmethod
    def save(childClass, object):
        jsonData= childClass.schema.dump(object)

        # Id needs to be stored in the parent object in a standardized way. We use id.
        if "id" in jsonData and jsonData["id"] is not "":
            id = jsonData["id"]
            # Don't want to use an upsert because if we make a new document we have additional actions to perform afterwards
            getattr(PySmores.__mongo, childClass.collection).update_one({"_id": ObjectId(id)}, {"$set": jsonData})

        # Else if there's no id, we assume that this is a new document and insert it
        else:
            id = str(getattr(PySmores.__mongo, childClass.collection).insert_one(jsonData).inserted_id)
            getattr(PySmores.__mongo, childClass.collection).update_one({"_id": ObjectId(id)}, {"$set": {"id": str(id)}})
            # Add the id as a string to the object. Easier to deal with in many situations than converting an ObjectId.
            object.id = id

        return id

    @classmethod
    def delete(childClass, object= None):
        if object is not None:
            return getattr(PySmores.__mongo, childClass.collection).delete_one({"_id": ObjectId(object.id)})
        # If we don't get passed anything, return an error
        else:
            return ValueError

    # ----------------------------------------------------------------------------------------------------------------------
    # Setup methods
    # ----------------------------------------------------------------------------------------------------------------------

    @staticmethod
    def instantiateSchemaWithJit(schema):
        instance = schema()
        instance.jit = toastedmarshmallow.Jit
        return instance
