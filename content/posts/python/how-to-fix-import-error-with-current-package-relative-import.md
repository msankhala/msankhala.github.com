---
title: "How fix import error with current package relative import"
date: 2023-09-05T08:06:25+06:00
description: "How to fix ImportError attempted relative import with no known parent package"
author: Mahesh Sankhala
# menu:
#   sidebar:
#     name: How to restore tmux lost session
#     identifier: rich-content
#     parent: sub-category
#     weight: 10
hero: images/python-local-import-error.png
tags: ["python", "import"]
categories: ["Basic"]
---

### Problem

## ImportError: attempted relative import with no known parent package

If you are developing a python package and try to import a module from the current package using relative path, you may get the following error.

```python
from .docdb_json_importer  import DocDbJsonImporter
```

OR

```python
from . import utils
```

> ImportError: attempted relative import with no known parent package

For example in my case I was developing a package called `docdb-import-export` and I was trying to import a module from the current package using relative path like this:

**src/docdb_import_export/\_\_main\_\_.py**

```python
import sys
import os
# Import local packages
from . import utils
from .docdb_json_importer  import DocDbJsonImporter
```

And I was getting error like this:

```sh
Traceback (most recent call last):
  File "/Users/mutant/Sites/htdocs/work/docdb-import-export/src/docdb_import_export/__main__.py", line 26, in <module>
    from . import utils
ImportError: attempted relative import with no known parent package
(docdb-import-export)
```

### Solution

According to the [Module documentation](https://docs.python.org/3/tutorial/modules.html#intra-package-references), for __main__ modules, you have to use absolute imports.

When we do the relative import it is going to look for `sys.path` and it is not going to find the current package in the `sys.path` and that is why it is going to throw the error.

So the solution is that we have to add the current package path to the `sys.path` and then we can do the relative import. Then use the current package name.

```python
from <CURRENT_PACKAGE_NAME> import <FILENAME/MODULENAME>
```

**src/docdb_import_export/\_\_main\_\_.py**

```python
import sys
import os

# Add current package path to sys.path
currentdir = os.path.dirname(os.path.realpath(__file__))
parentdir = os.path.dirname(currentdir)
sys.path.append(parentdir)

# Now import local packages with current package name
from docdb_import_export import utils
from docdb_import_export.docdb_json_importer  import DocDbJsonImporter
```

### Example

**src/docdb_import_export/docdb_json_importer.py**

```python
import json
import os
from abc import ABC, abstractmethod
# Now import local packages with current package name
from docdb_import_export.docdb_client import DocDbClient
```
