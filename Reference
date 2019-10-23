# ----------------------------------------------------------------------------------------------------------------------
# Schema definition
# ----------------------------------------------------------------------------------------------------------------------
import importlib
import sys

# Import Marshmallow modules
from marshmallow import Schema, fields, post_load, pre_dump, post_dump

class ReferenceSchema(Schema):
    referenceId = fields.String(allow_none= False)
    referenceClass = fields.String(allow_none= False)
    referenceClassImportPath = fields.String(allow_none= False)
    alwaysRetrieve = fields.Boolean()

    @post_load
    def postLoad_Reference(self, data):
        '''
        # If we didn't translate the reference into another object, return a Reference object
        if data is Reference:
            return ReferenceSchema.referenceToReferenceObject(data)
        # Otherwise return the object we made
        else:
            return data
        '''
        # Convert the reference into the object we care about to be saved off
        # We will return the desired object
        retrieve = data['alwaysRetrieve']
        if retrieve:
            return ReferenceSchema.getObjectFromReferenceDict(data)

    @pre_dump
    def preDump_Reference(self, object):
        '''
        convert the object into a real reference
        make sure the underlying object gets saved
        save the reference itself
        '''
        object.id = object.save(object)
        return ReferenceSchema.getReferenceFromObject(object)


    @post_dump
    def postDump_Reference(self, data):
        # Return the Reference object we created
        #return ReferenceSchema.referenceToReferenceObject(data)
        return data


    # ----------------------------------------------------------------------------------------------------------------------
    # pre/post_load helpers
    # ----------------------------------------------------------------------------------------------------------------------

    @staticmethod
    def getObjectFromReferenceDict(referenceDict):
        importPath = referenceDict['referenceClassImportPath']
        className = referenceDict['referenceClass']
        id = referenceDict['referenceId']

        return ReferenceSchema.retrieveObject(className, importPath, id)

    @staticmethod
    def retrieveObject(className, importPath, id):
        # If we haven't already imported this class...
        if className not in sys.modules:
            # Import the class we need to instantiate the object
            objClassFile = importlib.import_module(importPath)
            objClass = getattr(objClassFile, className)
        # Otherwise just grab it
        else:
            objClass = sys.modules[className]
        # collection = class.collection
        object = objClass.get(id= id)

        return object

    @staticmethod
    def referenceToReferenceObject(data):
        return Reference(**data)

    @staticmethod
    def getReferenceFromObject(object, alwaysRetrieve= True):
        ref = Reference()
        ref.referenceClassImportPath = object.__module__
        ref.referenceClass = object.__class__.__name__
        ref.referenceId = object.id
        ref.alwaysRetrieve = alwaysRetrieve

        return ref

    @staticmethod
    def getObjectFromReferenceObject(ref):
        importPath = ref.referenceClassImportPath
        className = ref.referenceClassName
        id = ref.referenceId

        return ReferenceSchema.retrieveObject(className, importPath, id)

# ----------------------------------------------------------------------------------------------------------------------
# Class definition
# ----------------------------------------------------------------------------------------------------------------------

from Controllers.Experimenting.Models.MongoDataSuperClass import MongoDataSuperClass

class Reference(MongoDataSuperClass):
    schema = MongoDataSuperClass.instantiateSchemaWithJit(ReferenceSchema)

    def __init__(self, referenceId= "", alwaysRetrieve= True, referenceClass= None, referenceClassImportPath= None):
        self.referenceId = referenceId
        self.referenceClass = referenceClass
        self.referenceClassImportPath = referenceClassImportPath
        self.alwaysRetrieve = alwaysRetrieve
