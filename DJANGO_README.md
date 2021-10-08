# File Organiser Django Interface

This readme is for the file organiser django interface. For information about the file organiser functionality consult the [FileOrganiserReadme](./FILEORGANISER_README.md).

## Summary:
#### This django project contains the FileOrganiser application interface
#### The application used the Uni-Soft insights application as a basic template, borrowing CSS and JS files for the nav and sidebar functionality (not yet implemented)


# Interface

## Index page
The main index page simply contains a text input form where the user specifies the root folder to scan. 

## Tree display page

Intermediate page that displays an interactive tree of the scanned root (using jsTree, see details below) and histograms of the file Type/Subtype distributions. From this page the user can select to configure and start the duplicate search or visit the file grouping utility. 


## Duplicate results page

Page for displaying the duplicate finding results. Displays the sorted list of duplicated files with size information (previously saved to a file) and the list of folder pair similarity scores.

## File grouping Utility

Page for configuring and previewing the results of the file grouping utility. The interactive table is used to define the group hierarchy. The preview of the grouping results are displayed using a jsTree instance for the user to verify the changes. 

From this page the user can restore previously grouped files to their original locations using a backup file (generated once files are grouped).

# Views

## scan_tree()
This view (function based) handles the tree display page and also the duplicate search configuration (configuration options only shown once file duplicate utility button is pressed). This view renders the 'treeview.html' template.

### Utilisation

1. From the index page input a root to scan. The view scan_tree is then called which constructs the appropriate `Tree` and the information is stored within a `TreeModel` class.

        Note: This `TreeModel` must be global to the `views.py` scope, since it does not persist when re-calling the view to access the duplicate results page.
        

1. The JSON strings for the jsTree are calculated using the methods in `general.py` whenever a fobject is constructed. However, they cannot all be collected until the entire tree is constructed (without a global reference to the tree). Instead, they are collected after constructing the tree by looping through the pathDict (could likely be optimised)

1. The file Type/Subtype distribution plots are created using Bokeh

1. The 'search' input field may be used to filter the tree based on a fuzzy keyword search. The view is called again and will update the jsTree and Bokeh plots to respect the filtered tree.

## scan_duplicates()

This view triggers the duplicate search and displays the results using the template 'duplicates.html'. The search is configured using a form on the tree view page. 

- This view stores the results of the duplicate search (as string lines) in the cache so that the user can toggle between the file/folder pair results. 

## grouping_utility()

This view controls the grouping utility page which starts with the 'fileorganiser/grouping/start.html' template. 

- This view simply displays the page that promps the user to select the folder(s) within which all files will be grouped. 
- This page also has the 'Restore from backup file' option. 

## grouping_config()

This view controls the grouping configuration page which uses the 'fileorganiser/grouping/config.html' template.

- This page prompts the user to specify the destination for the grouped files (a new folder is created in this destination that contains the first level of group folders in the hierarchy).

- The user can optionally specify the group hierarchy from this page. The functionality for adding/removing rows is implemented in javascript within the template.
- Since the Keyword group requires additional input, an extra column is added with a text input if this group is selected in the dropdown (all rows are updated whenever a dropdown is changed) - this still has some minor issues when adding/deleting additional rows.
- When the user clicks the 'Preview Groups' button the group heirarchy is parsed from the table using javascript `submitGroupHierarchy`. This reads the selected dropdown items and creates a csv string with the respective order. This string is then stored in a hidden form variable before the form is actually submitted (from this function).

## grouping_preview()

This view displays the preview of the grouped files (using a jstree instance), where the user has the option to accept or cancel the changes. 

- Once changes have been accepted, the page then displays an option to revert and move all files back, or download the backup file.

### Restore from backup

The start page of the grouping utility page also contains the 'restore from backup' functionality, where the user can select a backup file which will move any grouped files back to their original locations. This requires that files have not moved since they were originally grouped. 

# jsTree

Application specific information regarding the jsTree plugin. Consult the [jsTree documentation](https://www.jstree.com/) for more general information.

## Integration

The plugin operates using the ```jstree.js``` file located in the static/js directory. The required CSS file is ```jsTree_style.css``` located in static/css (separate from style.css for convenience during development - build single css file for deployment). 

## Data

The data required to construct the tree consists of a JSON string for each fobject, where the JSON must specify a unique id string (the path suffices) and the id of the parent. The JSON strings for each fobject are calculated on construction, an example is shown below (this shows the dict used to construct the JSON using `json.dumps(dict)`)
```python
JSONdict = {
        "id"          : str(path.as_posix()),          
        "parent"      : parent_id,         
        "icon"        : icon,           
        "text"        : str(path.name) + write_storage_info(fobject),
    }
```
The `id` and `parent` fields are always required, whilst the others will be autogenerated if not present (eg open/closed status of the node). It is possible to add additional fields here (to be accessed later) as they will be ignored by jsTree. 

NOTE: The master root folder is specified by assigning `parent = '#'`

The `icon` argument is used to define the icon for each node, by default this is taken from an image, this project instead uses a fontawesome glyphicon.

The `text` argument specifies the text to display on the node, and here includes the storage information (Net: _ Top: _ ) obtained using `general.write_storage_info`

## Data construction

1. JSON data for each fobject is created when constructing a fobject, using `general.individual_jstree_json` which does the following: 
    1. Sets the `id` as the absolute path using `path.as_posix()` 
    1. The `parent` id is set using `path.parent`. 
    1. If the folder is the root of a tree (`isTree=True`), the parent id must be set as `'#'`
    1. The name string uses the `path.name` in addition to the `general.write_storage_info` method. 
    1. The `icon` is set depending on the fobject type, using a fontawesome glyphicon 
1. JSON data for each fobject is stored as a string, and once the entire tree has been constructed these strings are collected into a single array `treeJson` (by looping through the pathDict) using `general.all_jstree_json`. 
1. `treeJson` is an array of JSON _strings_, which is passed to the template. The following must be used ``` {{ treeJson|safe }}``` to ensure extra escape characters are removed. 
    ```javascript
    var stringArray = {{ treeJson|safe }};
    ```
    NOTE: this line will produce syntax errors within the javascript block, however it _is_ correct.
    
1. This array of JSON strings is parsed (in javascript) to an array of JSON objects using 
    ```javascript  
    var jsonArray = stringArray.map(JSON.parse)
    ```


## Instantiation
To create a jsTree instance using the passed context `treeJson` (JSON string array):
1. Create an empty ```<div>``` container with an appropriate id, eg ``id=jstree_instance``
1. The following instantiates the tree, using the passed context treeJson: NOTE this __IS__ correct and syntax errors should be ignored
```javascript
<script>
    $(function () {
    // Parse the JSON string array
    var stringArray = {{ treeJson|safe }}; 
    var jsonArray = stringArray.map(JSON.parse)
    // jsTree instance for the div with id=jstree_instance
    $('#jstree').jstree({
      'core' : {
        'data' : jsonArray,         // JSON data array
        "check_callback" : true,    // Ensures events are captured
        "themes" : {
          "variant" : "large"       // Large row size theme
        }
      }, 
      'plugins' : ['sort']          // Use the sort plugin
    });
</script>
```
## Sorting

The jsTree sorts the children of each node alphabetically by default. To sort the children based on their size property the default sort function was modified in order to sort the children in descending size order. The function is located starting at line 7863 within [jstree.js](../../djangoproject/appsuite/static/js/jstree.js), which compares two child nodes in the tree.

The name string that is passed to each node is of the form 
```
File/Foldername    ---- (XXX GB)
```
In order to parse the size part of this string the following steps are implemented

1. Split the string on `"----"` (relies on this sequence not appearing anywhere else in the string) and take the last entry, which should be of the form (XXX GB)
1. A regular expression is used to find the order of magnitude, by matching against the following characters
    ```javascript
    var symbolsRe = [/[b]/g, /[K]/g, /[M]/g, /[G]/g, /[T]/g];
    ```
1. Once the order of magnitude has been found, the number portion is matched using 
    ```javascript
    var numRe = /([0-9]{1,3})/g;
    ```
    from which the actual size can be computed.
1. The sizes are then used to compare the pair of entries passed to the sort function (two child nodes). The function returns +1 if the first passed node size is less than the second, and -1 if the opposite. This is then used to determine the order in the tree. 

