# DynaTest

## Description
A simple Tango class which can be used to test the Tango dynamic attributes features.  

It was created to experiment with Tango dynamic attributes features and to help 
identifying the current limitations.

## Build
`make`

If this does not work for your Tango installation, you can use Pogo to regenerate a Makefile or a CMakefile.txt that fits your install settings.

## How to use?

This class provides some device properties which can be used to define the names of the DoubleRO, DoubleRW, LongRO and LongRW scalar dynamic we want to be created.
These properties are:
- DynDoubleROAttrNames
- DynDoubleRWAttrNames
- DynLongROAttrNames
- DynLongRWAttrNames

These properties are of type array of string.
This means that if you edit these properties from jive, you must put 1 attribute name per line.

## cppTango#814 investigations

As described in https://github.com/tango-controls/cpptango/issues/814, it is currently impossible to get a server like DynaTest exporting several devices in the same process which are creating dynamic attributes with the same name but a different types or Read/Write type.
If the devices are created in different processes (different device server instances), it is possible.

To show this limitation, you can load the DynaTest-\*.txt files into jive (File -> load property file).

These property files will create 3 different DynaTest device server instances:
- DynaTest/test: which is exporting 2 devices (test/dyna/1 and test/dyna/2) which are creating each 4 dynamic attributes of the same names but with different types or read/write types. If you start this device server, you will see that some dynamic attributes are not created.
- DynaTest/test1: exporting test/dyna/test1 which has exactly the same properties as test/dyna/1 - All dynamic attributes created successfully
- DynaTest/test2: exporting test/dyna/test2 which has exactly the same properties as test/dyna/2 - All dynamic attributes created successfully



podman start cpptango
podman exec -it cpptango /bin/bash

LD_LIBRARY_PATH=/usr/local/lib TANGO_HOST=localhost:10000 ./DynaTest dummy -v5

LD_LIBRARY_PATH=/usr/local/lib TANGO_HOST=localhost:10000 ipython

```python
# Add devices
from tango import Database, DbDevInfo
db = Database()
new_device_name1 = "test/dyna/1"
new_device_name2 = "test/dyna/2"
new_device_info = DbDevInfo()
new_device_info._class = "DynaTest"
new_device_info.server = "DynaTest/dummy"
new_device_info.name = new_device_name1
db.add_device(new_device_info)
new_device_info.name = new_device_name2
db.add_device(new_device_info)


# Set properties
import tango
from tango import DeviceProxy
axis1 = DeviceProxy("test/dyna/1")
property_names = ["DynDoubleROAttrNames", "DynDoubleRWAttrNames"]
axis_properties = axis1.get_property(property_names)
axis_properties["DynDoubleROAttrNames"]=["dbl1", "dbl2", "dbl3"]
axis1.put_property(axis_properties)
axis2 = DeviceProxy("test/dyna/2")
axis_properties = axis2.get_property(property_names)
axis_properties["DynDoubleROAttrNames"]=["dbl1", "dbl3"]
axis_properties["DynDoubleRWAttrNames"]=["dbl2"]
axis2.put_property(axis_properties)

#Read write attrs
import tango
from tango import DeviceProxy
axis1 = DeviceProxy("test/dyna/1")
axis1.get_attribute_list()
axis1.read_attribute('dbl1')
axis1.read_attribute('MyDynLongROAttribute')

```
