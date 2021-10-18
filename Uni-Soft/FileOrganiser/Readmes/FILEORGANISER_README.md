# File Organiser Functionality

This readme is for the File Organiser application functionality. For the Django interface please see the respective [Django readme](DJANGO_README.md)  

Detailed API documentation for the FileOrganiser can be found by viewing [/documentation/FileOrganiser/index.html](../../documentation/FileOrganiser/index.html) from a browser.

# Summary:
The [djangoproject/src/fileorganiser](../../djangoproject/src/fileorganiser) folder contains the relevant scripts required for the file organiser application.

# General 
## Script: [general.py](../../djangoproject/src/general.py)

This script contains the main class definitions (File, Folder, Tree) and utility functions used throughout. 

## Pathlib 
The application utilises the pathlib Path object for specifying any file/folder path. This allows for cross compatibility between systems using different delimiting conventions (Windows:  root\directory vs UNIX: root/directory). Path instances are constructed using a path-like string representing the path, in any format using  \\ , \ \ \, or /  delimiters.

The Path class method ```path.as_posix()``` is used throughout when obtaining a string representation of the path (uses the / delimiter) 

## File and Folder objects (```fobjects```)
The main class objects used throughout are the File and Folder classes, defined in ```general.py```. Throughout the documentation, the File and Folder classes will collectively be referred to as `fobjects` when applicable. Each is constructed using a path-like string or `Path` object.

## The global PATH_DICT: 
To ensure each fobject instance is unique, a global dictionary ```PATH_DICT``` is used, which maps every path to a unique file/folder instance. This is instantiated in ```globals.py``` and imported into every submodule. File and folder instances should therefore only be created once; when constructing the entire tree. When referencing any fobject instance, use the pathDict to ensure uniqueness of the fobject instance. 

## Files: 

- The ```File``` class represents a single file and is constructed using a pathlike object. 
- Each `File` contains various strings representing parts of the path, filename, extension, etc and the file size
- The `File` also contains the fingerprint (once calculated) and any assigned `Tag` objects see [tagging](#tagging)
 

## Folders

- The ```Folder``` class represents a single folder and contains a list of all subfolders and files. 

- The folder constructor recursively constructs all subdirectories if the kwarg makeAllSubdirs=True

- The kwarg isTree specifies if this folder is the root of a tree (for jsTree)

## Tree

This class is used for convenience to differentiate the tree (starting from the master root) from any other Folder instance. ```Tree(root)``` constructs the folder `rootFolder` and all subdirectories. When constructing a tree, the pathDict is cleared to avoid discrepancies.

This class also contains two dataframes used for the search functionality. Each dataframe column consists of the class fields (obtained by iterating over the `__dict__` attribute of the respective fobject). The dataframe is indexed by the absolute path of the respective fobject. See [searching](#searching) for more info.  

## __Always use parsePath__ 

The method ```parsePath(object)``` returns the Path object when passed any of the following objects: Path, pathlike string or any fobject. Note - this does not validate the path when passed a string. This general parsePath method allows for generic methods such as validPath(object) to accept any path-like object or fobject. 

Whenever accessing the Path of any object it is therefore safest to use ```parsePath(object)``` rather than `object.path`. This avoids trying to access Path.path if the object happens to already be a ```Path``` instance and not a fobject (allowing generic methods to accept both).


# Treesize display
The functionality for visualising the tree is now largely contained within the django interface, using the jsTree plugin (see the corresponding [section](./DJANGO_README.md/#jsTree) in the django readme). This displays an interactive tree with nodes that can be expanded/collapsed.  

The size information is also displayed within this tree, showing the Net (all contents) and Top (no subdirs) sizes of all fobjects in the tree - file objects only display their net size (filesize). 

(Still to implement) The method 


# Duplicate Finder
## Script: [duplicates.py](../../djangoproject/src/duplicates.py)
The functionality for finding duplicates is located in the ```duplicates.py``` script. 

This script contains the definition of the DupeFpLog class, which is instantiated for each duplicate fingerprint and stores the list of duplicated files. 

## Summary

The application uses a 128-bit MD5 hash to fingerprint each file and then identifies duplicated files by their fingerprint. The result is a filesize sorted list showing any duplicated files.

The user can specify a list of paths to exclude from the search, as well as several search parameters. After one search run has completed, the user has the option to append (currently not implemented in Django) more folders to be excluded and repeat the search.


## Configuration + Log objects 

The duplicate search configuration and search results log are specified using the ```Config()``` and ```DupeLog()``` classes (defined in [general.py](../../djangoproject/src/general.py)).

### Config 
Previously the config was parsed from a config.ini file, is now constructed using the following kwargs set by a HTML form:

- Threshold (bytes) : Files below this size will be ignored (default: 0)
- Depth : Depth of penetration into subdirectories (default: -1 = all)
- Excluded : Paths to ignore (csv list, parse to list using ```general.parseCSVlist```)

### Log 
- Stores the results of an individual duplicate finding run, resets every run (when appending excluded folders). 
- Contains a list of all ```DupeFpLog``` for each duplicate fingerprint found 
### DupeFpLog
- Log for a single fingerprint that corresponds to duplicated files
- Contains reference to the duplicated files and the duplicated (excess) storage amount.



## The duplicate search (file fingerprints)

Stage 1:
- Before fingerprinting, files are clustered based on their size (within a threshold of 1000 bytes). This clustering removes any files that could not have a possible match of a similar size and are thus not duplicates.
- Within the size clusters, files are then further clustered by extension and singular clusters removed.
- These two steps will reduce the total number of files needing to be fingerprinted (clusters are not retained and simply used for removing files that do not need to be considered as potential duplicates).


Stage 2: 
- Fingerprint (128-MD5) all files within the specified folders, ignoring any excluded paths
- Store all fingerprints in the ```fingerprintFiles``` dict, mapping fingerprint -> file(s)
- This is a multi-value dict, with each fingerprint key corresponding to a list of files with that fingerprint 

Stage 3:
- From the list of _all_ fingerprints, identify any that appear more than once
- The resulting fingerprints therefore correspond to duplicated files, which are obtained from the fingerprintFiles dict and logged in a ```DupeFpLog```.
- If the duplicated files reside in different directories, those directories are checked for similarity based on the number of identical files.

Stage 4 (Not in Django interface):
- If desired, the user can then append new folders to exclude from the next search
- The appended folders are used to remove the appropriate fingerprints from the master list (rather than re scanning the entire tree)
- The application proceeds to repeat stage 2.

Notes:

When fingerprinting, if the file cannot be opened due to permission errors (or any other exception) the file is simply skipped - rather than halt the whole program. Skipped files are currently not logged.

## Duplicate Directories

Once the initial fingerprinting stage has completed, a second search is initiated that attempts to identify duplicated directories. 

If a file duplicate resides in multiple directories, the fingerprints of all files in the top level (not including subfolders) of those directories are compared (pairwise). The intersection/union of file fingerprints is used to determine the similarity. If a pair is found to have similarity above a preset threshold (90%), the parent directories are then checked recursively until the directories are found to be not identical. 



## Similarity measures

Since this method relies on file signatures to identify similarities, any minor changes (eg one pixel changed in a photo) within a directory will cause the method to conclude there is no similarity, since currently, the directories are only compared at the 100% similarity level. 

In future, a heuristic will be developed for determining the 'similarity' between folders that are not 100% identical (based on file signatures). Considering the intersection/union between pairs of folders does give a good measure of similarity between pairs, however, it cannot be easily extended beyond pairwise combinations. A possible method that eliminates pairs based on their average similarity has been proposed but requires further consideration before implementation.

# Searching
## Script: [search.py](../../djangoproject/src/tagging.py)

This script contains functionality for searching the tree (dataframes of files/folders) and using the results to 'prune' the tree to only include results matching the search terms. 


## Fuzzy search:

The search is implemented using the rapidfuzz library, using the `process.extractall` method. Given a dataframe column, this method searches the column and returns a list of scored results. Those above the set threshold (90%) are included in the final output.   

## Keyword expansion:

The method `FindSimilarSearchTerms` uses wordnet sysets to expand search terms using synonymns. The expanded search terms are then individually used for a dataframe search and aggregated to produce the results. 


# Tagging 
## Script: [tagging.py](../../djangoproject/src/tagging.py)

This section relates to the tagging of files. Files should be tagged based on all available information (metadata), and further using any information learned from context (surrounding fobjects etc). Currently, only tags based on extension type are implemented.

## Tag and TagTree objects

Each tag is stored as a ```Tag``` object instance, which is unique for that particular tag (but may be associated with multiple files). Each ```Tag``` object contains a reference to its parent and child tags, the tag name as a string and the files with that particular tag. 

All tag objects are stored within the `TagTree` object. This contains a dict to reference each tag by name (and vice versa) as well as the definition dicts for file extension <-> type/subtype tags. The tag tree will be extended once more extensive tag types are implemented.

## File type tagging

The first step in the classification process (for files) is to assign basic tags based on the 'type' of file in question. These are obtained by considering the extension of the file and matching this extension with predefined categories.

### File type category definitions

The file types are defined in the spreadsheets located in the src/data folder. The types and subtypes are defined in the [ExtensionTypeSubtypesDefinitions](../../djangoproject/src/fileorganiser/data/ExtensionTypeSubtypesDefinitions.xls) file. The associations for extensions are defined in [ExtensionSubtypeLabels](../../djangoproject/src/fileorganiser/data/ExtensionTypeSubtypesDefinitions.xls) which defines the subtype for each extension. 

These data files are loaded and translated into tag objects, where the subtypes are child tags of each parent type - eg the Documents tag has child tags Document, Spreadsheet, Presentation. The definitions may be changed in either file and will be loaded each time the code runs. Note: spelling (including case and spaces) must match in both definition files. These definitions are used to build the `Tag` objects for each type/subtype, and are stored in the `TagTree` object. 

## File type distributions

The file type and subtypes are used to plot several distributions showing the number/storage occupied by files of each type/subtype. These are implemented using Bokeh plots and include an interactive tooltip. 

# Grouping Utility: [grouper.py](../../djangoproject/src/grouper.py)

This script contains functionality for grouping files based on a defined group hierarchy. The main object is the ```Grouper``` class, which stores the group hierarchy, grouping methods and the proposed result of the grouping for preview. 

## Group methods

The methods used for each particular grouping operation are defined in this script. Each method simply takes a list of files and returns a dict containing ```{'GroupName':GroupItems}``` where ```'GroupName'``` is the name of the group (eg Documents, Audio Files, etc for the 'Type' group) and ```GroupItems``` is a list of the files in that group. 

Currently, group by Type, Subtype and Keyword are implemented. The Keyword group method utilises the method used for the tree pruning search, by building a dataframe of files and extracting matches to the keyword terms. All files not matching any of the supplied keywords are placed into the 'Other' group. 

The possible group methods are stored in the ```Grouper.groupMethods``` dict and referenced by name. For example, to invoke the ```GroupByType``` method use the following
```python
grouper.groupMethods['Type'](files:list[File])
```


## Nested groups:

The group hierarchy is defined by the user in the interface, which is parsed and used to create a nested dict containing {Group: GroupItems = {Subgroup: SubgroupItems = ...}}. The group items will either be files (end of recursion) or subgroups containing further items. 

The groups are created recursively using the ```CreateAllNestedGroups``` method, which performs the desired grouping at each level of recursion. 


# Gensim Models
## Script: [corpus.py](../../djangoproject/src/corpus.py)

This section contains information about the Gensim models that were beginning to be explored. The idea was to create a corpus using the list of file paths (each corpus document being a single file path) and use this to train various gensim models in order to 'learn' the language associations between the specific terms used within the file system.

For more specific informaiton about the gensim models, consult the [documentation](https://radimrehurek.com/gensim/auto_examples/index.html#documentation)

## File path corpus



The corpus (collection of 'documents') in this case simply consists of the list of all file paths, where each file path (document) has been split into individual tokenised parts. For example the path 
```python
C:/Users/My Documents/Finances/Account Statement January.pdf
```
would be split into the tokenised document (strings are lowered and split on delimiters such as spaces, underscores, CamelCase etc)
```python
['users', 'my', 'documents', 'finances', 'account', 'statement', 'january']
```
The collection of all such file paths on the system thus comprises the corpus, and each individual unique token is used to create a bag-of-words (bow) vector representation for each document. 

Frequently occuring patterns within this corpus should then lead to associations forming between certain words, such as 'finances' and 'account'. 

The corpus is stored in the `FileCorpus` object, which simply contains the list of tokenised documents (paths) and their bow vector representations

## Models

Various gensim models can be trained on such a corpus, and these are stored within the `ModelCollection` class. Each model (excluding the word2vec models) are stored as a double which contains the (model transformation, similarity matrix index). The model transformation is used to convert a bow vector into the respective model representation, which can then be passed into the similarity matrix in order to obtain a list of similar documents. 

Currently the models include tfidf(term frequency/inverse document frequency), LSI(latent semantic indexing) and a combined LSI after tfidf model.

The word2vec model is a single object, and the similarity queries are obtained using the .wv wordvector attribute of the w2v model. 

The phrases model is not yet fully understood. The model can transform the corpus and produce possible phrases from the available documents, however, queries are not yet working properly.

## Model Queries

The `ModelQuery` class contains the result of querying the various models in the `ModelCollection` with a given query document. The query is passed as a string and subsequently converted to the bow vector representation (Note: Any query terms which are not in the corpus dictionary will be ignored). The similarity matrix of the model is then used to determine similar documents and their respective scores. 


