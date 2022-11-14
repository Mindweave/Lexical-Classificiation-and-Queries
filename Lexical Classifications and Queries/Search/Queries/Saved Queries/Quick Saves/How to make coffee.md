---
Aliases: 
NoteType: ManualQuerySearch

hasFeatures: hasDataviewQuery 
---
(Template:: [[Utilities/Template Utilities/Rebuild Files/Rebuild Search Classifications.md|Rebuild Search Classifications]] )

# Search Classifications

## Criteria
### Found Classifications
#### Found Classification Links (Separate links by commas)
(Has_at_least_one_of_these_found_classifications:: )
(Has_all_of_these_found_classifications:: [[Coffee]],[[Tutorial]])

(Missing_at_least_one_of_these_found_classifications:: )
(Missing_all_of_these_found_classifications:: )

### Files
#### File Links (Separate links by commas)
(Only_including_these_files:: )
(Excluding_these_files:: )

### File Paths
#### File Path Contains Text (Separate text by commas)
(Only_including_these_file_paths:: )
(Excluding_these_file_paths:: )

### Metadata
#### Date Created (accepts natural language or Year-Month-Day)
(Created_After:: )
(Created_Before:: )


```button
name Clear Criteria
type command
action Hotkeys for templates: Insert from Templater: Queries/Clear Criteria
```
^button-g81f

```button
name Save Query
type command
action Hotkeys for templates: Insert from Templater: Queries/Save Current Query
```
^button-muwv

```button
name Manually Refresh Query
type command
action Hotkeys for templates: Insert from Templater: Queries/Refresh Page
```
^button-1hc5
## Search Output

```dataviewjs

function removeSquareBracketsFromString(stringWithBrackets){return stringWithBrackets.replace(/[\[\]]+/g,"");}

function stringInObject (stringValue,ObjectValues){this.app.plugins.plugins['templater-obsidian'].templater.functions_generator.internal_functions.modules_array[0].static_object;
return ObjectValues.includes(stringValue)
}

function objectMatchesObject(Object1,Object2){
return Object1.every(r=> (Object2).includes(r))
}

function objectPartiallyMatchesObject(Object1,Object2){
	return (Object1).some(r=> (Object2).includes(r))
}

function stringEqualsString(stringValue1,stringValue2){
return stringValue1==stringValue2
}
function hasValue(variableToCheck){
    return (typeof(variableToCheck)!="undefined" && variableToCheck != null && variableToCheck?.length != 0)
}

function parseDate(dateToParse){
let returnDate = dateToParse;
if(hasValue(dateToParse)){
if(typeof(dv.parse(`${returnDate}`)?.isLuxonDateTime) == "undefined"){ //check to see if it is  a date by it having a date property
//needs formatting first from raw text
try{
returnDate = app.plugins.plugins['nldates-obsidian'].parseDate(returnDate)
returnDate = returnDate.moment.format("YYYY-MM-DD")
}catch(err){
return null
}

}

returnDate = dv.parse(`${returnDate}`)
if(returnDate == "Invalid date"){
//return info that the date is invalid (may change later)
return null
}else{
//return the parsed date
return returnDate
}
}else{
//did not have value
return null
}
}

function dateNow(){
let today = app.plugins.plugins['templater-obsidian'].templater.functions_generator.internal_functions.modules_array[0].static_object.now();
return  dv.parse(`${today}`);
}
    
function arrayHasRecordsOfOtherArray(mainArrayHasValues,mainArrayType,mainArray,comparisonArray,checkAll){
    if(mainArrayHasValues){
        if(mainArrayType == "object" && hasValue(comparisonArray)){
        if(checkAll){
            //than check all
            return objectMatchesObject(mainArray,comparisonArray);
        }else{
            //then check some
            return objectPartiallyMatchesObject(mainArray,comparisonArray);
        }
        }else{
        //isn't an array
        return false
        }
        }else{
        //could not have records since main array does not have any values
        return true;
        }
}

function pathsFromListOfLinks(listOfLinks){
    if(typeof (listOfLinks?.path) == "undefined"){ //check found is array
        if( typeof(listOfLinks?.map) == "undefined"){
    //does not have path but not an array. Likely error in classification (such as comma right after two dots)
    return [];
    }
        return  (listOfLinks.map(a => a?.path)).filter((a) => {return typeof(a) != "undefined"});
        }else{
        let blankArray = [];
		blankArray.push(listOfLinks?.path)
         return blankArray.filter(a => {return typeof(a) != "undefined"});
        }
}

function getFileLinks(file_classification_links){
let classification_links = file_classification_links == null ?  [] : [].concat(file_classification_links);
return classification_links;
}

function findFiles(anyClassificationLinks,allClassificationLinks,excludingAnyClassificationLinks,excludingAllClassificationLinks,onlyFilesToSearch,doNotSearchTheseFiles,createdDateBefore,createdDateAfter,includingFilePaths,excludingFilePaths){
//chcck type and if it has filters for logic per page

//Classification Include Links
let anyClassificationLinksPaths = pathsFromListOfLinks(anyClassificationLinks)
let allClassificationLinksPaths = pathsFromListOfLinks(allClassificationLinks)
let anyClassificationLinksType = typeof(anyClassificationLinksPaths);
let allClassificationLinksType = typeof(allClassificationLinksPaths)
let anyClassificationLinksIsFiltered =  hasValue(anyClassificationLinksPaths); 
let allClassificationLinksIsFiltered = hasValue(allClassificationLinksPaths);
let includeClassificationMatch = false;

//Classification Exclude Links
let excludingAnyOfTheseClassificationLinksPaths = pathsFromListOfLinks(excludingAnyClassificationLinks)
let excludingAllOfTheseClassificationLinksPaths = pathsFromListOfLinks(excludingAllClassificationLinks)
let excludingAnyOfTheseLinksType = typeof(excludingAnyOfTheseClassificationLinksPaths);
let excludingAllOfTheseLinksType = typeof(excludingAllOfTheseClassificationLinksPaths)
let excludingAnyClassificationLinksIsFiltered =  hasValue(excludingAnyOfTheseClassificationLinksPaths); 
let excludingAllClassificationLinksIsFiltered = hasValue(excludingAllOfTheseClassificationLinksPaths);
let excludeClassificationMatch = false;

//only these files
let onlyFilesPaths = pathsFromListOfLinks(onlyFilesToSearch);
let onlyFilesType = typeof(onlyFilesPaths);
let onlyFilesIsFiltered = hasValue(onlyFilesPaths);
let doNotSearchFilesPaths = pathsFromListOfLinks(doNotSearchTheseFiles);
let doNotSearchFilesType = typeof(onlyFilesPaths);
let doNotSearchFilesIsFiltered = hasValue(doNotSearchFilesPaths);

//file paths
let includingFilePathsArray = typeof(includingFilePaths?.split) == "undefined"?[]: includingFilePaths.split(",");
let includingFilePathsIsFiltered = (typeof(includingFilePathsArray) == "object" && includingFilePathsArray[0]?.length > 0)
let excludingFilePathsArray = typeof(excludingFilePaths?.split) == "undefined"?[]: excludingFilePaths.split(",");
let excludingFilePathsIsFiltered = (typeof(excludingFilePathsArray) == "object" && (excludingFilePathsArray)[0]?.length > 0)

//Dates include values
let createdDateBeforeIsFiltered = hasValue(createdDateBefore)
let createdDateAfterIsFiltered = hasValue(createdDateAfter)
let createdDateBeforeMatch = false;
let createdDateAfterMatch = false;
let includeCreatedDate = false;

let returnDataview  = dv.pages('"Scraps"').where(p => typeof(p?.found_classifications) !="undefined" && p?.found_classifications != null ).where((p) =>{ 

//check if it is on the list of included files
if(onlyFilesIsFiltered){
//do not even attempt to check if it is not on the list of included files
if(!arrayHasRecordsOfOtherArray(onlyFilesIsFiltered,onlyFilesType,onlyFilesPaths,[p.file.path],false)){
return false;
}
}

//check if it is on the list of excluded files
if(doNotSearchFilesIsFiltered){
//do not even attempt to check if it is on the list of excluded files
if(arrayHasRecordsOfOtherArray(doNotSearchFilesIsFiltered,doNotSearchFilesType,doNotSearchFilesPaths,[p.file.path],false)){
return false;
}
}

//check if it is in the list of included file paths
if(includingFilePathsIsFiltered){
//do not even attempt to check if it is on the list of excluded files
if(!(includingFilePathsArray.filter((a) => {return p.file.path.indexOf(a) != -1}).length > 0)){
return false; //was not in array
}
}

//check if it is not in the list of excluded file paths
if(excludingFilePathsIsFiltered){
//do not even attempt to check if it is on the list of excluded files
if((excludingFilePathsArray.filter((a) => {return p.file.path.indexOf(a) != -1}).length > 0)){
return false; //was in array
}
}

    let allClassificationLinksMatch = false;
    let anyClassificationLinksMatch = false;
    
    let found_classification_paths = pathsFromListOfLinks(p.found_classifications);
    
//check if the page includes classifications intended to be included
    allClassificationLinksMatch = arrayHasRecordsOfOtherArray(allClassificationLinksIsFiltered,anyClassificationLinksType,allClassificationLinksPaths,found_classification_paths,true);
    anyClassificationLinksMatch = arrayHasRecordsOfOtherArray(anyClassificationLinksIsFiltered,anyClassificationLinksType,anyClassificationLinksPaths,found_classification_paths,false);
    includeClassificationMatch = (!anyClassificationLinksIsFiltered 
 || anyClassificationLinksMatch) && (!allClassificationLinksIsFiltered || allClassificationLinksMatch );


//check if the page includes classifications which were exluded
	let excludingAllClassificationLinksMatch = false;
    let excludingAnyClassificationLinksMatch = false;
    
    excludingAllClassificationLinksMatch = arrayHasRecordsOfOtherArray(excludingAllClassificationLinksIsFiltered,excludingAllOfTheseLinksType,excludingAllOfTheseClassificationLinksPaths,found_classification_paths,true);
    excludingAnyClassificationLinksMatch = arrayHasRecordsOfOtherArray(excludingAnyClassificationLinksIsFiltered,excludingAnyOfTheseLinksType,excludingAnyOfTheseClassificationLinksPaths,found_classification_paths,false);
    //is filtered and has match
    excludeClassificationMatch = (!excludingAnyClassificationLinksIsFiltered || excludingAnyClassificationLinksMatch) && (!excludingAllClassificationLinksIsFiltered || excludingAllClassificationLinksMatch) && (excludingAnyClassificationLinksIsFiltered || excludingAllClassificationLinksIsFiltered);




if(hasValue(p.file?.ctime)){ 
if(createdDateBeforeIsFiltered){
createdDateBeforeMatch = dv.compare(p.file.ctime, createdDateBefore) == -1 
}else{
createdDateBeforeMatch = true;
}
if(createdDateAfterIsFiltered){
createdDateAfterMatch = dv.compare(p.file.ctime,createdDateAfter) >= 0
}else{
createdDateAfterMatch = true;
}
includeCreatedDate =  (createdDateAfterMatch || !createdDateAfterIsFiltered) && (createdDateBeforeMatch || !createdDateBeforeIsFiltered)
}else{
includeCreatedDate = !(createdDateBeforeIsFiltered || createdDateAfterIsFiltered)
}

let hasAtLeastOneFilter = (anyClassificationLinksIsFiltered || allClassificationLinksIsFiltered || excludingAnyClassificationLinksIsFiltered || excludingAllClassificationLinksIsFiltered)

   return (includeClassificationMatch && !excludeClassificationMatch) && includeCreatedDate && hasAtLeastOneFilter


    
   } )



return returnDataview;
}

function formatUserDate(userDate){
let dvDate = (dv.date(userDate).c);
return (dvDate.year+"-"+dvDate.month+"-"+dvDate.day)
}

let createdAfter = parseDate(dv.current().Created_After);
let createdBefore = parseDate(dv.current().Created_Before);

dv.el('p',"Date Range: "+"("+(createdAfter === null? "Start of Time":formatUserDate(createdAfter) )+")"+" to "+"("+(createdBefore === null? "End of Time":formatUserDate(createdBefore) )+")" )


//found classifications variables
let anyClassificationLinks =getFileLinks((dv.current().Has_at_least_one_of_these_found_classifications))
let allClassificationLinks = getFileLinks((dv.current().Has_all_of_these_found_classifications))

let excludingAnyOfTheseClassificationLinks =getFileLinks((dv.current().Missing_at_least_one_of_these_found_classifications))
let excludingAllOfTheseClassificationLinks = getFileLinks((dv.current().Missing_all_of_these_found_classifications))

//Limiting files variables
let onlyFilesToSearch = ((dv.current().Only_including_these_files))
let doNotSearchTheseFiles = ((dv.current().Excluding_these_files))

//Limiting folders variables
let includingFilePaths = ((dv.current().Only_including_these_file_paths))
let excludingFilePaths = ((dv.current().Excluding_these_file_paths))

let groupOne = findFiles(anyClassificationLinks,allClassificationLinks,excludingAnyOfTheseClassificationLinks,excludingAllOfTheseClassificationLinks,onlyFilesToSearch,doNotSearchTheseFiles,createdBefore,createdAfter,includingFilePaths,excludingFilePaths);

dv.table(["Link","Created Date"],(
    groupOne).map(a => {
    
    
return [(a.file.link),a.file.ctime]}))




```


