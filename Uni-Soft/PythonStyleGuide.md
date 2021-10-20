# Uni-Soft Python Style Guide


## Introduction

This guide aims to summarise the best practices and conventions most commonly used when writing python code. The practices outlined here should provide the basis for writing readable and extensible code. However, they are by no means absolute, and discretion should always be used in context to find the most consistent and appropriate style.

When working with naming and styling conventions, consistency is always more important than the convention itself. So long as one remains consistent, any convention should provide easily readable code. However, certain conventions are arguably more favourable than others, and are elaborated here. 

# Naming conventions

## General

- In general, names should aim to be as __descriptive__ yet __succinct__ as possible, such that functionality/purpose can be easily inferred from the name. 
    - However, this is not always possible and further elaboration should then be deferred to a comment, rather than creating names longer than 3-5 words. 

- Where possible, abbreviations should be avoided unless absolutely necessary or commonnly understood (`str`, `var`, `int` for example).

- Module names should be as short as possible and all lowercase (e.g `script.py`), with underscores used if absolutely necessary. 



## CamelCase:

```CamelCase``` is the preferred method for composite names consisting of several words (as opposed to ```the_underscores_method```). 

As a general rule, classes should be written using `CamelCase` (capitalised first letter), whilst methods and variables be written with `mixedCase` (lowercase first letter). 

- ```CamelCase```: 
    - Classes - ```ClassName```

- ```mixedCase```:
    - Methods - ```someMethod```
    - Variables - ```someVariable```

```python
# Class name uses CamelCase
class ClassName():
    # Internal python methods use dunder __name__
    def __init__(self):
        # mixedCase for variables
        classVariable = ...
        # Comment each variable
        anotherClassVariable = ...

    
    # Class methods use mixedCase
    def classMethod(self):
        ...


# Module level methods also use mixedCase 
def descriptiveMethodName():
    ...
```


## Underscores

The 'dunder' (double underscores around an \_\_object\_\_ ) should be reserved for internal python commands only.  

Where appropriate, underscores may be used to group several items that derive from a more general property, for example:
```stat_netSize, stat_topSize, stat_average``` (note the continued usage of mixedCase where appropriate).

Leading underscores are used to indicate "weak internal use" - methods/variables that are only used internally to the relevant scope. These cannot be imported using the ```from module import ``` syntax  
```python
module.py:

# This method can only be used from within module.py
def _anInternalMethod():
    ...

# This method will be importable from other modules
def importableMethod(args):
    # The imported method can still use any internal methods within module.py
    _anInternalMethod(args)
    ...
```

## Comments
- Class and method definitions should be preceeded by a single line comment that summarises the class/method in one line. Detailed comments should be included in the [docstring](#Docstrings).

- Comments should be inserted before every line, unless a line can be directly inferred from context or a previous line. 

- For example, simple statements such as the following do not need comments, since their action is completely elaborated by the statement (comments are shown to indicate their verbosity). Whilst it does not harm to include these comments, they can clutter otherwise readable code and offer little extra information.

```python
# Check if the bool is true
if(boolean == True):

# Construct a range of integers from 1 to 5 
for(x in range(1, 5)):

# Add the variables a and b 
result = a + b
```

- When commenting code you have not written yourself, either when working with others or using external code, you must differentiate your own comments to those already present. For example, decorate the comment with initials or your name 
```python
# Original developer comment
def somebodyElsesMethod():
    #JS - My added comment 

    #JohnSmith - My added comment 
```

## Docstrings

A 'docstring' is a more descriptive comment that should appear as the __first statement__ within any method or class definition (specified as a string literal using triple quotes as ```"""docstring"""```). 

Docstrings are recognised by python (only if the first statement after a definition!) and accesible through the \_\_doc\_\_ attribute - useful when building documentation using external tools. 

The docstrings for classes and methods should be formatted as follows:

```python
class ClassName():
    """
    Class: ClassName
    
    Description:
    ------------ 
    Paragraph describing the class. 
    Elaborate variables, class methods and usage.
    
    Notes:    
    ------
    Any extra notes

    """
    def __init__(self, parameters):
        """
        Method: ClassName.__init__
        
        Description:
        ------------
        Paragraph describing the method. 
        Elaborate inputs, functionality and outputs.
        
        Inputs:
        -------
        Args: 
            argName:type 
                -- description
            anotherArg:type
                -- description
        Kwargs:
            kwargName:type = defaultValue 
                -- description 
        
        Outputs:
        --------
        Return:
            output:type 
                -- description 
        
        Notes:    
        ------
        Any extra notes

        """
```
Note: the ------- is markdown syntax to indicate the previous line should be a heading. This aids readability as well as docstring processing by external tools.

## Line breaks 

- Line breaks should be used where appropriate to separate related sections within a class or method. 

- A double line break should be inserted after the last line of a class or method (indicated here with ```<br>``` for clarity).
```python

# Method description
def someMethod():
    '''
    Method docstring
    '''
    # Define a variable
    var = ...
    # A related variable
    relatedVar = ...
    # A relevant method
    resultVar = relevantMethod(var, relatedVar)
    
    # A new section 
    sectionVar = ...
    # Related method for this section
    sectionResult = sectionMethod(newVar)

    # Another separate section
    output = finalMethod(resultVar, sectionResult)

    # Always insert a line break (and comment if appropriate) before return statement
    return output 
'<br>'
'<br>'
# Another method description
def anotherMethod():
    ...
```

## Comment headers

Comment headings should be used where appropriate to separate relevant sections within the code. 
```python
# ======================================================== #
# ==================== COMMENT HEADER ==================== #
# ======================================================== #
```
The VSCode extension [Comment Bars](https://marketplace.visualstudio.com/items?itemName=zfzackfrost.commentbars) is particularly useful for quickly creating correctly spaced comment bars using the appropriate comment style. It is highly customisable but the 'quick' option that fills with `=` characters as shown here is more than sufficient for most cases.


# General Good Practices

Here some general good practices are outlined, whilst these are applicable in most cases, some might not be appropriate in context.

## Type annotation

Since python is not a type-safe language, the following expression is completely general and can accept an input of an arbitrary type
```python
def someMethod(inputs):
    ...
```
whilst this allows for a great deal of flexibility, it is ambiguous as to what the inputs should be. If the input is required to be of a specific type, it is customary to specify the type as part of the argument as follows
```python
def someMethod(inputList:list, inputDict:dict, value:float):
    ...
```
note here the variable names can also be used to elaborate the variable type, but this will not always be the case and so direct specification is preferred. 