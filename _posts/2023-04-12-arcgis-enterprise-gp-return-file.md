---
layout: default
title: Making an ArcGIS Enterprise Geoprocessing that return a file.
tags: arcgis geoprocessing python
author: Marc_Alx
---

_published first as an answer on [comunity.esri.com](https://community.esri.com/t5/python-questions/geoprocessing-service-to-return-a-file/m-p/1277808/highlight/true#M67402)_

# Making an ArcGIS Enterprise Geoprocessing that return a file.

Here's a tutorial on how to make an ArcGIS Enterprise python Geoprocessing that return a file.

## Important notes

- Your parameter file must have a "direction" of "Output" and a "parameterType" of "Derived"

- The file type for geoprocessing parameter is "DEFile", for some reason "GPDataFile" results in no file being accessible to end user. Once published it appears as "GPDatafile"

- Result must be set via arcpy.setParameterAsText whereas setting "value" of a saved parameter does nothing.

- In order to be accessible you file must be wrote/saved to "arcpy.env.scratchFolder". Arcpy rewrite this path once published to an accessible URL.

- If you wish to backup your file outside of "arcpy.env.scratchFolder" you have to register the save location, from ArcGIS Pro: add any folder you need via: Share→Manage→Data stores, here add a new `folder` with `Publish folder path` set to the path you want to expose. Keep in mind that any hard coded path will be consolidated (to a sandbox path) on publish by ArcGIS.

## Useful links

- [Authoring web tools with Python scripts](https://pro.arcgis.com/en/pro-app/latest/help/analysis/geoprocessing/share-analysis/authoring-web-tools-with-python-scripts.htm) _(to understand path consolidation)_

- [Publish web tool](https://pro.arcgis.com/en/pro-app/latest/help/analysis/geoprocessing/share-analysis/publishing-web-tools-in-arcgis-pro.htm)

## The code

```
# -*- coding: utf-8 -*-

import arcpy
from os.path import join

class Toolbox(object):
    def __init__(self):
        """Sample file toolbox"""
        self.label = "Sample file toolbox"
        self.alias = "File toolbox"

        # List of tool classes associated with this toolbox
        self.tools = [GetFileTool]

class GetFileTool(object):

    def __init__(self):
        """Define the tool (tool name is the name of the class)."""
        self.label = "Get file tool"
        self.description = "A tool to get a file"
        self.canRunInBackground = False

    def getParameterInfo(self):
        """Define parameter definitions"""
        return [arcpy.Parameter(displayName="Output_File",
                                name='Output_File',
                                datatype=['DEFile'],
                                parameterType='Derived',
                                direction='Output')]

    def isLicensed(self):
        """Set whether tool is licensed to execute."""
        return True

    def updateParameters(self, parameters):
        """Modify the values and properties of parameters before internal
        validation is performed.  This method is called whenever a parameter
        has been changed."""
        return

    def updateMessages(self, parameters):
        """Modify the messages created by internal validation for each tool
        parameter.  This method is called after internal validation."""
        return
    
    def exitWithError(self,msg):
        """
            exit geoprocessing with a given error msg
            under the hood it raise an exception, msg is logged as error
        """
        raise ValueError(msg)

    def execute(self, parameters, messages):
        """The source code of the tool."""
        p = join(arcpy.env.scratchFolder, "test.txt")
        with open(p, 'w') as f:
            f.write('hello')
        #set result with SetParameterAsText, setting result via 'value' parameter of objects inside parameters does nothing
        arcpy.SetParameterAsText(0,p)
```