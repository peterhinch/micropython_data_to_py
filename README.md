# data_to_py.py

A means of storing read-only data in Flash, conserving RAM.

The utility runs on a PC and converts an arbitrary binary file into a Python
source file. The generated Python file may be frozen as bytecode. Read-only
data may thus be stored in Flash and accessed with little RAM use.

## Arguments

The utility requires two arguments, the first being the path to a file for
reading and the second being the path to a file for writing. The latter must
have a ``.py`` extension. If the output file exists it will be overwritten.

## The Python output file

This instantiates a ``bytes`` object containing the data, which may be accessed
via the file's ``data()`` function: this returns a ``memoryview`` of the data.

The use of a ``memoryview`` ensures that slices of the data may be extracted
without using RAM:

```python
import mydata  # file mydata.py (typically frozen)
d = mydata.data()  # d contains a memoryview
s = d[1000:3000]  # s contains 1000 bytes but consumes no RAM
```

A practical example is rendering a JPEG file to an LCD160CR display. The Python
file ``img.py`` is generated using

```
$ ./data_to_py.py my_image.jpg img.py
```

The file ``img.py`` may then be frozen, although for test purposes it can just
be copied to the target. The following code displays the image:

```python
import lcd160cr
import img
lcd = lcd160cr.LCD160CR('Y')
lcd.set_orient(lcd160cr.PORTRAIT)
lcd.set_pen(lcd.rgb(0, 0, 0), lcd.rgb(0, 0, 0))
lcd.erase()
lcd.set_pos(0, 0)
jpeg_data = img.data()
lcd.jpeg(jpeg_data)  # Access the data and display it
```

## Validation

The utility can accept any file type as input, so no validation is performed.
If required, this must be done by the application at runtime. For example in
the above code, to be suitable for the device jpeg_data[:2] should be
``0xff, 0xd8`` and jpeg_data[-2:] should be ``0xff, 0xd9``.
