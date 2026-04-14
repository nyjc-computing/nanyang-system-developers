
# CSV & JSON

## Overview

Binary data (raw bits) is often **serialized** to a standard text format when it is to be passed to another application. The other application then **deserializes** the text-formatted back to binary.

[Comma-separated values](https://en.wikipedia.org/wiki/Comma-separated_values) (CSV) and [JavaScript Object Notation](https://www.json.org/json-en.html) (JSON) are two of the most common text formats for serialized data.

## Modules for importing and exporting data

The process of converting an object to text data is known as **serialisation**.
The process of converting text data to an object is known as **deserialisation**.

In python, we can manually serialise or deserialise data using [IOStream objects](https://docs.python.org/3/library/io.html). The simplest way to get an IOStream object is to use the built-in `open()` function. With an IOStream object, we write data to a file using the `f.write()` method, and read data from a file using the `f.read()` method (or other related methods).

Python also provides importable modules to ease the serialisation of data to and deserialisation of data from some commonly used formats. Two of the most commonly used modules are `csv` and `json`. These are text-based formats that allow object data to be written out in readable text-based form.

### CSV format

CSV stands for Comma-Separated Values. This format is commonly used for table exports, data science, etc. 

- All data is exported as text only; type conversions are implicit (i.e. guessed)
- Strings may be quoted if they contain restricted characters (e.g. commas)
- Does not support arrays

Format example:

    student_name,enrollment_year,class
    Student1,2020,2004
    Student2,2020,2005
    Student3,2020,2005
    ...

### JSON format

JSON stands for JavaScript Object Notation. This format is commonly used for exporting more complex data, and in web programming.
- Basic data types, including arrays and objects (documents) are supported
- JSON data tends to occupy more space

Format example:

    [{"student_name": "Student1",
      "enrollment_year": 2020,
      "class": "2004"},
     {"student_name": "Student2",
      "enrollment_year": 2020,
      "class": "2005"},
    ...
    ]

## Python `csv` module
### Using `csv`
The `csv` module provides `reader()` and `writer()` **functions**.

These functions return an **iterable** object. An iterable is an object that can be looped over (e.g. with a `for` or `while` loop). A Python list is an iterable, but an iterable need not be a list.

`open()` should be called with the optional argument `newline=''` (See [here](https://stackoverflow.com/a/3191811) for a detailed explanation)

[Official documentation](https://docs.python.org/3/library/csv.html) for `csv` module

### Reading into a list of lists with `csv`

Reading with `csv` module:

    import csv
    with open('data csv', 'r', newline='') as f:
        csv_iterable = csv.reader(f)
        for row in csv_iterable:
            # each row is a list of elements
            ...

### Using `csv` to get a list of dicts
The `csv` module also provides a `DictReader` class that makes it easy to obtain each row as a dictionary, with column headers as row keys.

Reading to list of dicts with `csv` module:

    import csv
    with open('data. csv', 'r', newline='') as f:
        # The csv file is assumed to have a header row
        csv_iterable = csv.DictReader(f)
        for row_dict in csv_iterable:
            # each row is a dict with col headers as keys
            ...

### Using `csv` to write a list of dicts to csv

Writing a list of dicts with `csv` module:

    import csv
    with open('data.csv', 'w', newline='') as f:
        # DictWriter requires a list of header names
        # these can be generated from the .keys() method of the row dict, and converted to a list
        fieldnames = ['key1', 'key2']
        dict_writer = csv.DictWriter(f, fieldnames=fieldnames)
        dict_writer.writeheader()
        for row_dict in list_of_dicts:
            # each row element is a dict
            dict_writer.writerow(row_dict)

## Python `json` module
### Using `json`

The `json` module provides `load()` and `dump()` **functions**.
`load()` deserialises data from a json file to a Python dict, while `dump()` serialises a Python object to a json file.

The JSON format must be represented in Unicode, with UTF-8 being the recommended default.

### Using `json` to load from a file

Deserialising with `json` module:

    import json
    with open('data. json', 'r', encoding='utf-8') as f:
        # data from data.json is deserialised into data_dict
        data_dict = json.load(f)

### Using `json` to write to a file

Serialising with json module:

    import json
    with open 'data. json', 'w', encoding='utf-8') as f:
        # data from obj is serialised into data.json
        json.dump(obj, f)

For a more readable file:

    import json
    with open('data. json', 'w', encoding='utf-8') as f:
        # data from obj is serialised into data.json
        json.dump(obj, f, indent=4)


The json module also provides `loads()` and `dumps()` functions. `loads()` deserialises data from a **json-encoded string** to a Python dict, while `dumps()` serialises a **json-encoded string** to a json file.

The JSON format must be represented in Unicode, with UTF-8 being the recommended default.

### Using `json` to write to a string

Serialising with `json` module:

    import json
    # data from obj is serialised into data_str
    data_str = json.dumps(obj)

### Using `json` to load from a string

Deserialising with `json` module:

    import json
    # data from data_str is deserialised into data_dict
    data_dict = json.loads(data_str)