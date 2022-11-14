%%I found it difficult to get the exclude outlinks working so I just tabled it. I've been working at it on and off for 4 hours. It's not worth that right now%%

---
Title: Search Classifications
Aliases: 
NoteType: ManualQuerySearch

hasFeatures: hasDataviewQuery 
---
(Template:: [[Utilities/Template Utilities/Rebuild Files/Rebuild Search Classifications.md|Rebuild Search Classifications]] )

# Search Classifications

## Criteria
### Found Classifications
(Has_at_least_one_of_these_found_classifications:: )
(Has_all_of_these_found_classifications:: )

(Missing_at_least_one_of_these_found_classifications:: )
(Missing_all_of_these_found_classifications:: )

### Links
#### Links within the file
(Has_at_least_one_of_these_outlinks:: )
(Has_all_of_these_outlinks:: )

(Missing_at_least_one_of_these_outlinks:: [[Twitter]])
(Missing_all_of_these_outlinks::)

#### Other files which link to the file 
(Has_at_least_one_of_these_inlinks:: )
(Has_all_of_these_inlinks:: )

(Missing_at_least_one_of_these_inlinks:: )
(Missing_all_of_these_inlinks:: )

### Files

### Metadata
#### Type and Root
##### Include pages with
(Any_of_these_note_types:: [""])

##### Exclude files with
(Has_any_of_these_note_types:: [""])

#### Date Created 
##### Include files which were
(Created_After:: )
(Created_Before:: )

```button
name Manually Refresh Query
type command
action Hotkeys for templates: Insert from Templater: Queries/Refresh Page
```
^button-1hc5

```button
name Clear Criteria
type command
action Hotkeys for templates: Insert from Templater: Queries/Clear Criteria
```
^button-g81f

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
returnDate = app.plugins.plugins['nldates-obsidian'].parseDate(returnDate)
returnDate = returnDate.moment.format("YYYY-MM-DD")
}

returnDate = dv.parse(`${returnDate}`)
if(returnDate == "Invalid date"){
//return info that the date is invalid (may change later)
return null
}else{
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

function findFiles(anyClassificationLinks,allClassificationLinks,excludingAnyClassificationLinks,excludingAllClassificationLinks,anyOutlinks,allOutlinks,excludingAnyOutlinks,excludingAllOutlinks,createdDateBefore,createdDateAfter){
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

//Outlinks Include Links
let anyOutlinksPaths = pathsFromListOfLinks(anyOutlinks)
let allOutlinksPaths = pathsFromListOfLinks(allOutlinks)
let anyOutlinksType = typeof(anyOutlinksPaths);
let allOutlinksType = typeof(allOutlinksPaths)
let anyOutlinksIsFiltered =  hasValue(anyOutlinksPaths); 
let allOutlinksIsFiltered = hasValue(allOutlinksPaths);
let includeOutlinksMatch = false;

//Outlinks Exclude Links
let excludingAnyOutlinksPaths = pathsFromListOfLinks(excludingAnyOutlinks)
let excludingAllOutlinksPaths = pathsFromListOfLinks(excludingAllOutlinks)
let excludingAnyOutlinksType = typeof(excludingAnyOutlinksPaths);
let excludingAllOutlinksType = typeof(excludingAllOutlinksPaths)
let excludingAnyOutlinksIsFiltered =  hasValue(excludingAnyOutlinksPaths); 
let excludingAllOutlinksIsFiltered = hasValue(excludingAllOutlinksPaths);
let excludeOutlinksMatch = true;

//Dates include values
let createdDateBeforeIsFiltered = hasValue(createdDateBefore)
let createdDateAfterIsFiltered = hasValue(createdDateAfter)
let createdDateBeforeMatch = false;
let createdDateAfterMatch = false;
let includeCreatedDate = false;

let returnDataview  = dv.pages('"Scraps"').where(p => typeof(p?.found_classifications) !="undefined" && p?.found_classifications != null ).where((p) =>{ 

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

//check inlinks and outlinks
//check outlinks
let found_outlinks = [];

let anyOutlinksMatch = false;
let allOutlinksMatch = false;
let excludingAnyOutlinksMatch = false;
let excludingAllOutlinksMatch = false;

if(hasValue(p.file?.outlinks) ){ //if nothing to filter or no filters
//add found_outlinks path
found_outlinks = pathsFromListOfLinks(p.file.outlinks.values);

//included outlinks
allOutlinksMatch = arrayHasRecordsOfOtherArray(allOutlinksIsFiltered,allOutlinksType,allOutlinksPaths,found_outlinks,true);
anyOutlinksMatch = arrayHasRecordsOfOtherArray(anyOutlinksIsFiltered,anyOutlinksType,anyOutlinksPaths,found_outlinks,false);
includeOutlinksMatch = ((!allOutlinksIsFiltered || allOutlinksMatch) && (!anyOutlinksIsFiltered || anyOutlinksMatch))

//excluding outlinks
excludingAllOutlinksMatch = arrayHasRecordsOfOtherArray(excludingAllOutlinksIsFiltered,excludingAllOutlinksType,excludingAllOutlinksPaths,found_outlinks,true);
excludingAnyOutlinksMatch = arrayHasRecordsOfOtherArray(excludingAnyOutlinksIsFiltered,excludingAnyOutlinksType,excludingAnyOutlinksPaths,found_outlinks,false);
if(excludingAnyOutlinksMatch){console.log("any match")}
excludeOutlinksMatch = (!excludingAnyOutlinksIsFiltered || excludingAnyOutlinksMatch) && (!excludingAllOutlinksIsFiltered || excludingAllOutlinksMatch) && (excludingAnyOutlinksIsFiltered || excludingAllOutlinksIsFiltered);
}else{
//check if there was a filter. If so then it would be missed and not matched
includeOutlinksMatch = !(anyOutlinksIsFiltered || allOutlinksIsFiltered);
excludeOutlinksMatch = (excludingAnyOutlinksIsFiltered || excludingAllOutlinksIsFiltered);
} 


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

let hasAtLeastOneFilter = (anyClassificationLinksIsFiltered || allClassificationLinksIsFiltered || excludingAnyClassificationLinksIsFiltered || excludingAllClassificationLinksIsFiltered || anyOutlinksIsFiltered || allOutlinksIsFiltered)

   return (includeClassificationMatch && !excludeClassificationMatch)  && (includeOutlinksMatch && !excludeOutlinksMatch) && includeCreatedDate && hasAtLeastOneFilter


    
   } )



return returnDataview;
}

let createdAfter = parseDate(dv.current().Created_After);
let createdBefore = parseDate(dv.current().Created_Before);

dv.el('p',createdBefore)
dv.el('p',  dv.compare(createdBefore, dateNow()) )
//console.log(app.plugins.plugins['nldates-obsidian'].parseDate(createdAfter))
//let parsedCreatedAfter = app.plugins.plugins['nldates-obsidian'].parseDate(createdAfter)
//let formattedCreatedAfter = parsedCreatedAfter.moment.format("YYYY-MM-DD")
//dv.el('p',dv.parse(`${formattedCreatedAfter}`))
//found classification groups variables



//found classifications variables
let anyClassificationLinks =getFileLinks((dv.current().Has_at_least_one_of_these_found_classifications))
let allClassificationLinks = getFileLinks((dv.current().Has_all_of_these_found_classifications))

let excludingAnyOfTheseClassificationLinks =getFileLinks((dv.current().Missing_at_least_one_of_these_found_classifications))
let excludingAllOfTheseClassificationLinks = getFileLinks((dv.current().Missing_all_of_these_found_classifications))


let anyOutlinks = (dv.current().Has_at_least_one_of_these_outlinks)
let allOutlinks = (dv.current().Has_all_of_these_outlinks)

let excludingAnyOutlinks = (dv.current().Missing_at_least_one_of_these_outlinks)
let excludingAllOutlinks = (dv.current().Missing_all_of_these_outlinks)

let groupOne = findFiles(anyClassificationLinks,allClassificationLinks,excludingAnyOfTheseClassificationLinks,excludingAllOfTheseClassificationLinks,anyOutlinks,allOutlinks,excludingAnyOutlinks,excludingAllOutlinks,createdBefore,createdAfter);
//console.log("group one")
//console.log(groupOne)

dv.table(["Link","Count References"],(
    groupOne).map(a => {
    let inlinkLength;
    if(typeof (a.file.inlinks?.path) == "undefined"){ //check found is array
        if(a.file.inlinks?.length >0){
        inlinkLength = a.file.inlinks;
        }else{
        inlinkLength = 0
        }
        }else{
         inlinkLength = a.file.inlinks.length;
        }
	let outlinkLength;
    if(typeof (a.file.outlinks?.path) == "undefined"){ //check found is array
        if(a.file.outlinks?.length >0){
        outlinkLength = a.file.outlinks;
        }else{
        outlinkLength = 0
        }
        }else{
         outlinkLength = a.file.outlinks.length;
        }
    
return [(a.file.link),inlinkLength+outlinkLength ]}))




```


