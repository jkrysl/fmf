
======================
    Features
======================

Let's demonstrate the features on a simple wget example with the
following directory structure::

    wget
    ├── download
    ├── protocols
    │   ├── ftp
    │   ├── http
    │   └── https
    ├── recursion
    └── smoke


Simple
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The most common use cases super simple to read & write. Test
metadata for a single test look like this::

    description: Check basic download options
    tester: Petr Šplíchal <psplicha@redhat.com>
    tags: [Tier2, TierSecurity]
    test: runtest.sh
    time: 3 min


Hierarchy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Hierarchy is defined by directory structure (see example above) and
explicit nesting using attributes starting with ``/``.  Defining
metadata for several tests in a single file is straightforward::

    /download:
        description: Check basic download options
        tester: Petr Šplíchal <psplicha@redhat.com>
        tags: [Tier2, TierSecurity]
        test: runtest.sh
        time: 3 min
    /recursion:
        description: Check recursive download options
        tester: Petr Šplíchal <psplicha@redhat.com>
        tags: [Tier2, TierSecurity]
        test: runtest.sh
        time: 20 min

Content above would be stored in ``wget/main.fmf`` file.


Inheritance
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Metadata is inherited from parent objects::

    tester: Petr Šplíchal <psplicha@redhat.com>
    tags: [Tier2, TierSecurity]
    test: runtest.sh
    
    /download:
        description: Check basic download options
        time: 3 min
    /recursion:
        description: Check recursive download options
        time: 20 min

This nicely prevents unnecessary duplication. Redefining an
attribute in a child object will by default overwrite value
inherited from the parent. It is also possible to use a "+"
appended to the attribute name to add given value instead::

    time: 1
    /download:
        time+: 3

This operation is possible only for attributes of the same type.
Exception ``MergeError`` is raised if types are different. When
the "+" operator is applied on dictionaries ``update()`` method is
used to merge content of given dictionary instead of replacing it.


Elasticity
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use a single file or scatter metadata across the hierarchy,
whatever is more desired for the project.

File ``wget/main.fmf``::

    tester: Petr Šplíchal <psplicha@redhat.com>
    tags: [Tier2, TierSecurity]
    test: runtest.sh

File ``wget/download/main.fmf``::

    description: Check basic download options
    time: 3 min

File: ``wget/recursion/main.fmf``::

    description: Check recursive download options
    time: 20 min

This allows reasonable structure for both small and large
projects.


Scatter
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Thanks to elasticity, metadata can be scattered across several
files. For example ``wget/download`` metadata can be defined in
the following three files:

File ``wget/main.fmf``::

    /download:
        description: Check basic download options
        test: runtest.sh

File ``wget/download.fmf``::

    description: Check basic download options
    test: runtest.sh

File ``wget/download/main.fmf``::

    description: Check basic download options
    test: runtest.sh

Parsing is done from top to bottom (in the order of examples
above). Later/lower defined attributes replace values defined
earlier/higher in the structure.


Leaves
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When searching, **key content** is used to define which leaves
from the metadata tree will be selected. For example, every test
case to be executed must have the ``test`` attribute defined,
every requirement to be considered for test coverage evaluation
must have the ``requirement`` attribute defined. Otherwise object
data is used for inheritance only::

    description: Check basic download options
    test: runtest.sh
    time: 3 min

The key content attributes are not supposed to be hard-coded in
the Flexible Metadata Format but freely configurable. Multiple key
content attributes (e.g. script & backend) could be used as well.


Virtual
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Using a single test code for testing multiple scenarios can be
easily implemented using leaves inheriting from the same parent::

    description: Check basic download options
    test: runtest.sh
    
    /fast:
        description: Check basic download options (quick smoke test)
        environment: MODE=fast
        tags: [Tier1]
        time: 1 min
    /full:
        description: Check basic download options (full test set)
        environment: MODE=full
        tags: [Tier2]
        time: 3 min

In this way we can efficiently create virtual test cases.


Format
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When investigating metadata using the ``fmf`` command line tool,
object identifiers and all associated attributes are printed by
default, each on a separate line. It is also possible to use the
``--format`` option together with ``--value`` options to generate
custom output. Python syntax for expansion using ``{}`` is used to
place values as desired. For example::

    fmf --format 'name: {0}, tester: {1}\n' \
        --value 'name' --value 'data["tester"]'

Individual attribute values can be accessed through the ``data``
dictionary, variable ``name`` contains the object identifier and
``root`` is assigned to directory where metadata tree is rooted.

Python modules ``os`` and ``os.path`` as well as other python
functions are available and can be used for processing attribute
values as desired::

    fmf --format '{}' --value 'os.dirname(data["path"])'
