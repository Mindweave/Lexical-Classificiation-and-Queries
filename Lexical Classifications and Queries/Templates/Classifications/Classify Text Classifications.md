<%*

/*Initial variables get type of template actions (debug or not)*/
function getDataview() {
return app.plugins.plugins['dataview'].api
}

var allClassificationsInFolderToDebug;

var allClassificationsDebugTitles;

var allClassificationsDebugPaths;

var selectedClassificationPath;

var thisTemplate = tp.config.template_file.basename

var templateMode;

const dv = getDataview();

if (thisTemplate == "Classify using text indicators") {

templateMode = "Classify";

} else if (thisTemplate == "Debug classification source") {

templateMode = "DebugSource"

}

if (templateMode == "DebugSource") {

await tp.system.prompt("In Debug mode. The list of found_classifications will not be affected. (Press enter to continue) ")

allClassificationsInFolderToDebug = await dv.pages('"Classifications/Text Classifications"').where(p => typeof (p.NoteType) != "undefined").values /*separated from all classifications in folder in case some need to be excluded from debug*/

allClassificationsDebugTitles = allClassificationsInFolderToDebug.map(a => { return a.file.name + (a.file.aliases.values.length > 0 ? (" " + "(" + a.file.aliases.values + ")") : ""); });

allClassificationsDebugPaths = allClassificationsInFolderToDebug.map(a => a.file.path);

selectedClassificationPath = await tp.system.suggester(allClassificationsDebugTitles, allClassificationsDebugPaths, "choose classification to debug")

}

/*Required Functions*/
function leftDoubleBrackets() {

return "[" + "["

}

function rightDoubleBrackets() {

return "]" + "]"

}


function hasValue(variableToCheck){
return (typeof(variableToCheck)!="undefined" && variableToCheck != null && variableToCheck?.length != 0)
}

async function allFilesInFolder(selectedFolder){
return await app.plugins.plugins.dataview.api.pages(`"${selectedFolder}"`);
}
function isIterable(input) {  
//https://futurestud.io/tutorials/check-if-a-value-is-iterable-in-javascript-or-node-js
  if (input === null || input === undefined) {
    return false
  }

  return typeof input[Symbol.iterator] === 'function'
}

function addPageTitles(startingArray,pages){
if( isIterable(pages) ){
let filesToSelect = startingArray;
for (let value of pages){filesToSelect.push(value.file.name); }
return filesToSelect;
}else{
return pages.file.name
}
}

function addFullPages(startingArray,pages){
let filesToSelectPage = startingArray;
for (let value of pages){filesToSelectPage.push(value);}
return filesToSelectPage;
}

async function selectFileFromFolder(selectedFolder,allowSkip){
let filesInFolder = await allFilesInFolder(selectedFolder);  let filesToSelect = ["(Search Recent Files)"];  let 
filesToSelectPage = ["(Selected Recent Files)"]; if(allowSkip){filesToSelect.push("(Skip)"); filesToSelectPage.push("(Skip)");} filesToSelect = addPageTitles(filesToSelect,filesInFolder); filesToSelectPage = addFullPages(filesToSelectPage,filesInFolder); let selectedFile = await tp.system.suggester(filesToSelect,filesToSelectPage,false,"Select file to add"); 
return selectedFile;
}

async function selectFolder(){

let allFolders = await app.plugins.plugins.dataview.api.pages('"Scraps"').groupBy((page) => {return this.app.metadataCache.getFirstLinkpathDest(page.file.path, "").parent.path}); let currentFolder =  this.app.workspace.getActiveFile().parent.path; let foldersToSelect = ["(Current Folder:"+" "+currentFolder+")"]; let foldersToSelectValue = [currentFolder]; for (let value of allFolders){if(value.key == currentFolder){continue;}foldersToSelect.push(value.key);foldersToSelectValue.push(value.key);} let selectedFolder = await tp.system.suggester((foldersToSelect),(foldersToSelectValue),false,"Select folder to search");
if(selectedFolder[selectedFolder.length-1] != "/"){
return selectedFolder+"/"
}
return selectedFolder
}

function removeSquareBracketsFromString(stringWithBrackets) { return stringWithBrackets.replace(/[\[\]]+/g, ""); }

function getTFile(filePath) {

return this.app.metadataCache.getFirstLinkpathDest(filePath, "");

}

async function readFromTFile(selectedTFile) {

return await this.app.vault.read(selectedTFile)

}

function isSubset(array1, array2) {

return array2.every((element) => array1.includes(element));

}

function isIntersecting(array1, array2) {

return array2.some((element) => array1.includes(element));

}


function standardizedPunctuation(textUnformatted) {
let textFormatted = textUnformatted;
textFormatted = textFormatted.replaceAll("’", "'");
textFormatted = textFormatted.replaceAll("‘", "'");
textFormatted = textFormatted.replaceAll('“', '"');
textFormatted = textFormatted.replaceAll('”', '"');
return textFormatted
}

async function getTextIndicatorsFromClassification(classificationPath, classificationSourceText, startOfTextIndicators, endOfTextIndicators) {

/*Getting indicators*/

let location = classificationPath;

let textInFile = classificationSourceText;

let startOfSearchText = startOfTextIndicators; //"#### Automatically Classify"


let startOfSearch = textInFile.indexOf(startOfSearchText)

let endOfSearchText = endOfTextIndicators; //"#### Request Before Classification"


let endOfSearch = textInFile.indexOf(endOfSearchText)

let currentHeader = 0;

let startOfIndicatorText = String.fromCharCode(9) + "-" + " "


let startOfIndicatorTextLength = startOfIndicatorText.length


let endOfIndicatorText = String.fromCharCode(10);

let nextHeaderText = String.fromCharCode(10) + "-" + " "


let nextPossibleHeader = 0


let indicatorsArray = [];

let innerIndicatorsArray = [];

if (startOfSearch == -1 || endOfSearch == -1) {

//did not find text to search through

return [];

}

/*reducing file size to search through (possible optimization) need to adjust search values*/

//textInFile = textInFile.substring(startOfSearch,endOfSearch+endOfSearchText.length)

let foundNextClassification = startOfSearch

//Get list of indicators

while (foundNextClassification < endOfSearch && foundNextClassification != -1) {

if (foundNextClassification != startOfSearch) { //still before header

//save header

currentHeader = textInFile.substring(startOfSearch, foundNextClassification).lastIndexOf(nextHeaderText); //last header up to found classification

//save classification

innerIndicatorsArray.push(textInFile.substring(foundNextClassification + startOfIndicatorTextLength, textInFile.indexOf(endOfIndicatorText, foundNextClassification)));

}

foundNextClassification = textInFile.indexOf(startOfIndicatorText, foundNextClassification + 1); //start after last classification to find next

nextPossibleHeader = textInFile.substring(startOfSearch, foundNextClassification).lastIndexOf(nextHeaderText);

if (currentHeader != nextPossibleHeader && innerIndicatorsArray.length != 0) {

//if any indicator was found between last one indent header

indicatorsArray.push(innerIndicatorsArray)

innerIndicatorsArray = [];

}

}

//Capture last indicator since it wont cross a heading which triggers inputting the value

if (innerIndicatorsArray.length != 0) {

indicatorsArray.push(innerIndicatorsArray)

}

//Output list of indicators for testing

return indicatorsArray;

}

function escapeRegExp(string) {

return string.replace(/([.*+?^=!:${}()|\[\]\/\\])/g, "\\$1");

}

function regexObject(regexText) {

//console.log("Initial Regex Text");

//console.log(regexText);

let slashCharacter = String.fromCharCode(47);

let regexTextLength = regexText.length;

let firstValueIsSlash = regexText[0] == slashCharacter;

let lastValueIsSlash = regexText[regexTextLength - 1] == slashCharacter;

let lastSlashPosition = regexText.lastIndexOf(slashCharacter);

let textAfterLastSlashPosition;

let hasModifierg = false;

let hasModifieri = false;

let hasModifierm = false;

let modifiers = "";

let regexTextFormatted = regexText;

//no slashes so no possible modifiers

if (lastSlashPosition == -1) {

return new RegExp(regexTextFormatted)

}

if (!lastValueIsSlash && lastSlashPosition > 0 && ((regexTextLength - lastSlashPosition) < 4)) {

//has space for modifiers after last slash position

//last slash position is not error (-1) or the first value (0)

//test after last slash does it have the modifier


textAfterLastSlashPosition = (regexText.substring(lastSlashPosition + 1));

hasModifierg = (textAfterLastSlashPosition.indexOf("g") != -1)

hasModifieri = (textAfterLastSlashPosition.indexOf("i") != -1)

hasModifierm = (textAfterLastSlashPosition.indexOf("m") != -1)

}

if (firstValueIsSlash) {

regexTextFormatted = regexTextFormatted.substring(1, regexTextFormatted.length);

}

if (lastValueIsSlash) {

//basic removal of slash


regexTextFormatted = regexTextFormatted.substring(0, regexTextFormatted.length - 1);

return new RegExp(regexTextFormatted);

} else if (lastSlashPosition != 0 && lastSlashPosition != -1) {

if (!(hasModifierg || hasModifieri || hasModifierm)) {

//last slash is not for modifiers


new RegExp(regexTextFormatted)

} else {

//last slash is for modifiers


regexTextFormatted = regexTextFormatted.substring(0, lastSlashPosition - 1);

}

} else {

//invalid last slash position

//return as is

new RegExp(regexTextFormatted);

}

if (hasModifierg) {

modifiers += "g"

}

if (hasModifieri) {

modifiers += "i"

}

if (hasModifierm) {

modifiers += "m"

}

if (modifiers.length > 0) {

// console.log(new RegExp(regexTextFormatted, modifiers))

return new RegExp(regexTextFormatted, modifiers)

} else {

// console.log(new RegExp(regexTextFormatted))

return new RegExp(regexTextFormatted)

}

}

function getHeadingFromClassificationAction(classificationAction){
if(classificationAction == "autoStop"){
return "automatically stopping"
}else if(classificationAction == "conditionsToAttempt"){
return "attempting classification"
}else if(classificationAction == "autoClassify"){
return "automatically classifying"
}
}

async function meetsTextIndicatorCriteria(textIndicator, indicatorSpecialOptions, textToSearch, textToSearch_default, activeFilePath,classificationFilePath,classificationAction,userSelection,criteriaNumber) {

let foundIndicatorSpecialOptions = [].concat(indicatorSpecialOptions);

let indicatorFormatted = textIndicator;

let textToSearchFormatted = textToSearch;

let specialOptionIndicatorCheckTypes = ["{{(Includes Text)}}", "{{(Excludes Text)}}", "{{(Includes Pure Regex)}}", "{{(Excludes Pure Regex)}}"] /* TODO add includes link and includes found classification*/

let changeSearchTextType = ["{{(Case-Sensitive)}}", "{{(Keep Punctuation Format)}}", "{{(File Path as Text)}}", "{{(File Text as Text)}}"] //add date created and add date modified as text and use templater date moment to parse the date (is before date, is after date, is on date, is day greater than, is hour greater than, is year greater than, is month greater than, is weekday, is weekend)

let changeIndicatorType = ["{{(Case-Sensitive)}}", "{{(Allow Trailing Whitespace)}}", "{{(Keep Punctuation Format)}}", "{{(Keep Obsidian Symbols)}}"];

let foundIndicatorInText = false; //start with false to confirm one is found

var regex;

var matchedValue = "";

//assuming no indicator to search in the text would include the special options

if (foundIndicatorSpecialOptions.length > 0 && indicatorFormatted.indexOf("{{(") != -1) {

let firstSpecialOption = indicatorFormatted.indexOf("{{(");

indicatorFormatted = indicatorFormatted.substring(0, firstSpecialOption)

}

//Changes to text to search

if (isIntersecting(foundIndicatorSpecialOptions, changeSearchTextType)) {

//check for what changes need to be made to the text

//text needs to be file path if this special option is found

if (isIntersecting(foundIndicatorSpecialOptions, ["{{(File Path as Text)}}"])) {

//case sensitivity and trailing whitespace still apply to a file path as well as text

textToSearchFormatted = activeFilePath


}

//keep punctuation standardized unless specified otherwise
if (isSubset(foundIndicatorSpecialOptions, ["{{(Keep Punctuation Format)}}"])) {

//keep indicator as is

} else {

//replace IOS smart punctuation. Smart apostrophes and quotes
textToSearchFormatted = standardizedPunctuation(textToSearchFormatted);
}

//remove obsidian symbols used for functionality unless specified otherwise
if (isSubset(foundIndicatorSpecialOptions, ["{{(Keep Obsidian Symbols)}}"])) {

//keep indicator as is

} else {

//replace Obsidian symbols. Links are replaced with their aliases shown on the editor, footnotes are removed using regex. 
textToSearchFormatted = textToSearchFormatted; //TODO
}


//lower so search is case-insensitive (could use Regex but this code is easier to maintain and read)

if (isSubset(foundIndicatorSpecialOptions, ["{{(Case-Sensitive)}}"])) {

//keep text as is

} else {

//lower text to be case-insensitive

textToSearchFormatted = textToSearchFormatted.toLowerCase();

}

} else {

//use default text format. Case-insensitive with normal punctuation
textToSearchFormatted = textToSearch_default;
/*
textToSearchFormatted = textToSearchFormatted.toLowerCase();
textToSearchFormatted = standardizedPunctuation(textToSearchFormatted);
*/
}

//If it is Pure Regex there should be no changes to the indicator

if (isIntersecting(foundIndicatorSpecialOptions, ["{{(Includes Pure Regex)}}", "{{(Excludes Pure Regex)}}"])) {

//cleanup regex

regex = indicatorFormatted.trim();

regex = regexObject(regex);

foundIndicatorInText = textToSearchFormatted.search(regex) != -1;//new RegExp(regex).test(textToSearchFormatted);

if (foundIndicatorInText) {

matchedValue = textToSearchFormatted.match(regex)[0]

}

} else {

//Changes to indicator

if (isIntersecting(foundIndicatorSpecialOptions, changeIndicatorType)) {

//check for what changes need to be made to the indicator


//trim before lowering, reduces characters

if (isSubset(foundIndicatorSpecialOptions, ["{{(Allow trailing whitespace)}}"])) {

//keep indicator as is

} else {

indicatorFormatted = indicatorFormatted.trim()

}

if (isSubset(foundIndicatorSpecialOptions, ["{{(Keep Punctuation Format)}}"])) {

//keep indicator as is

} else {

//replace IOS smart punctuation. Smart apostrophes and quotes
indicatorFormatted = standardizedPunctuation(indicatorFormatted);
}

//lower so search is case-insensitive (could use Regex but this code is easier to maintain and read)

if (isSubset(foundIndicatorSpecialOptions, ["{{(Case-Sensitive)}}"])) {

//keep indicator as is

} else {

indicatorFormatted = indicatorFormatted.toLowerCase();

}

} else {

//use default indicator. Case-insensitive and trimmed with normal punctuation

indicatorFormatted = indicatorFormatted.trim().toLowerCase();
indicatorFormatted = standardizedPunctuation(indicatorFormatted);

}

if (isIntersecting(foundIndicatorSpecialOptions, ["{{(File Path as Text)}}"])) {

//is found in the text. Since file paths are not separated by words, using an index of catches any part of the file path

foundIndicatorInText = (textToSearchFormatted.indexOf(indicatorFormatted) != -1)


if (foundIndicatorInText) {

matchedValue = textIndicator; //original indicator

}

} else {
//String.fromCharCode(92) is for the backslash \ symbol
regex = '([^a-zA-Z0-9_]|^)'; //'\\b'; word boundary was catching non-word symbols when they were in the classification

regex += escapeRegExp(indicatorFormatted);

regex += '([^a-zA-Z0-9_]|' + String.fromCharCode(92) + 'Z|$)';

foundIndicatorInText = new RegExp(regex).test(textToSearchFormatted);


/*
classificationAction,userSelection
console.log("found indicator")
console.log(indicatorFormatted);
console.log(new RegExp(regex));
console.log(textToSearchFormatted);
console.log(foundIndicatorInText);
*/
if (foundIndicatorInText) {

matchedValue = textIndicator; //original indicator

}

}

//End changes to indicator and non pure regex checks

}

//switch results if they should be excluded

if (isIntersecting(foundIndicatorSpecialOptions, ["{{(Excludes Pure Regex)}}", "{{(Excludes Text)}}"])) {

foundIndicatorInText = !foundIndicatorInText

matchedValue = textIndicator; //was not found in text so was not added previously

} else {

foundIndicatorInText = foundIndicatorInText

}



if(userSelection == "Walkthrough classification in current file"){
let initialText = "";
let headingText = getHeadingFromClassificationAction(classificationAction);
let startingCriteriaOrAnd = (criteriaNumber == 0) ? "":"AND ";
let metCriteria = foundIndicatorInText ? "met":"NOT met";
initialText = startingCriteriaOrAnd+"Criteria"+" "+metCriteria+" for "+headingText+": ";

let waitOnClassificationProcess = false;
do{
let userChoice = await tp.system.suggester(["Continue","Copy Criteria","Copy Regex","Copy Text Classified"],["Continue","Copy Criteria","Copy Regex","Copy Text Classified"],false,initialText+textIndicator);
waitOnClassificationProcess = (userChoice != "Continue")
if(userChoice == "Copy Criteria"){
await navigator.clipboard.writeText(textIndicator);
}
if(userChoice == "Copy Regex"){
await navigator.clipboard.writeText(new RegExp(regex));
}
if(userChoice == "Copy Text Classified"){
await navigator.clipboard.writeText(textToSearchFormatted);
}
}while(waitOnClassificationProcess)

}


if (foundIndicatorInText) {

return matchedValue

} else {

return ""

}

/*return (new RegExp("\\b"+textIndicator.toLowerCase()+"\\b").test(textToSearchLower))*/

}

function returnListOfSpecialOptionsFromText(withinText) {

let startSplitValue = "{{(";

let endSplitValue = ")}}";

let SplitArray = withinText.split(startSplitValue);

for (let i = 0; i < SplitArray.length; i++) {

if (SplitArray[i].indexOf(endSplitValue) == -1) {

SplitArray[i] = ""; continue;

}

SplitArray[i] = startSplitValue + SplitArray[i].substring(0, SplitArray[i].indexOf(endSplitValue)) + endSplitValue;

}


/*Remove sections which are not links*/


let splitArrayToString = SplitArray.toString();

if (splitArrayToString.charAt(0) == ",") {

splitArrayToString = splitArrayToString.substring(1, splitArrayToString.length);

}

if (splitArrayToString.charAt(splitArrayToString.length - 1) == ",") {

splitArrayToString = splitArrayToString.substring(0, splitArrayToString.length);

}

return splitArrayToString.split(",");

}

async function foundTextIndicatorValuesInCurrentText(foundIndicators, activeFileText, activeFileTextDefault, requestNeeded, classificationName,classificationFilePath, activeFilePath,classificationAction,userSelection) {
//Search current file text

let matchesFound = [];

let indicatorMeetsCriteria = false;

for (let i = 0; i < foundIndicators.length; i++) {
if(userSelection == "Walkthrough classification in current file"){
let promptText = "";
if(i == 0){
promptText = "Checking first criteria group of "+getHeadingFromClassificationAction(classificationAction)
}else{
promptText = "OR Checking next criteria group of "+getHeadingFromClassificationAction(classificationAction)
}
let continueChecking = await tp.system.suggester(["Continue","Stop Checking Criteria"],
["Continue","Stop Checking Criteria"],false,promptText);
if(continueChecking != "Continue"){
break;
}

}
for (let j = 0; j < foundIndicators[i].length; j++) {

let indicatorSpecialOptions = returnListOfSpecialOptionsFromText(foundIndicators[i][j]);

let matchedValue = await meetsTextIndicatorCriteria(foundIndicators[i][j], indicatorSpecialOptions, activeFileText, activeFileTextDefault, activeFilePath,classificationFilePath,classificationAction,userSelection,j);

if (matchedValue.length > 0) {

matchesFound.push(matchedValue);

}

}

//i array

if (matchesFound.length == (foundIndicators[i].length)) { /*all matches found under header*/


break;

}

matchesFound = [];

}

if (matchesFound.length != 0) {

if (requestNeeded) {

let classifyAgreed = await tp.system.prompt(("Classify as: " + String.fromCharCode(34) + classificationName + String.fromCharCode(34) + "?" + " Found in text: " + String.fromCharCode(34) + matchesFound.toString().replace(/\s/g, " ") + String.fromCharCode(34) + " (enter text for yes or blank for no)"))


if (classifyAgreed.length > 0) {

return matchesFound;

} else {

return [];

}

} else {

//return true for classification

return matchesFound;

}

} else {

//return blank for not classifying

return [];

}

}

function returnUniqueArray(nonUniqueArray){
return [...new Set(nonUniqueArray)];
}

function addCommentedFoundClassificationsSection(currentTFile,currentTFileText, foundClassificationsArray) {
let foundClassificationsCleaned = "{{( found_classifications:: " + foundClassificationsArray.map(a => {return leftDoubleBrackets()+a.path+"|"+a.basename+rightDoubleBrackets()}).join(",") + ")}}";
if (templateMode != "DebugSource") {
let formattedTextToAdd = (String.fromCharCode(10)+String.fromCharCode(10) + String.fromCharCode(37) + String.fromCharCode(37) + String.fromCharCode(10) + foundClassificationsCleaned + String.fromCharCode(10) + String.fromCharCode(37) + String.fromCharCode(37))
app.vault.modify(currentTFile,currentTFileText+formattedTextToAdd)

}

}

function replaceFoundClassificationSection(currentTFile,currentTFileText) {

if (templateMode != "DebugSource") {
let formattedTextToAdd = (String.fromCharCode(10) + String.fromCharCode(37) + String.fromCharCode(37) + String.fromCharCode(10) + textToAdd + String.fromCharCode(10) + String.fromCharCode(37) + String.fromCharCode(37))
app.vault.modify(currentTFile,currentTFileText+formattedTextToAdd)

}

}

var filesToClassify;
var userCurrentFile = app.workspace.getActiveFile()
let optionsToClassify;
let optionsToClassifySelected;
if(userCurrentFile.path == "00 Home Note/00 Home Note.md"){
optionsToClassify = ["Classify all markdown files in Scraps folder", "Classify markdown files in folder","Classify all unclassified markdown files in Scraps folder"];
optionsToClassifySelected = ["Classify all markdown files in Scraps folder", "Classify markdown files in folder","Classify all unclassified markdown files in Scraps folder"];
}else{
optionsToClassify = ["Classify current file", "Walkthrough classification in current file"];
optionsToClassifySelected = ["Classify current file", "Walkthrough classification in current file"];
}

var userFilesSelection = await tp.system.suggester(optionsToClassify,optionsToClassifySelected); 
var selectedClassificationToDebug; //only applicable in debug
//edit to have options
if(userFilesSelection == "Classify all markdown files in Scraps folder"){
filesToClassify = dv.pages('"Scraps"').where(p=>p.file.ext == "md");
}else if (userFilesSelection == "Classify current file"){
filesToClassify = [dv.page(userCurrentFile.path)];
}else if(userFilesSelection == "Classify all unclassified markdown files in Scraps folder"){
filesToClassify = dv.pages('"Scraps"').where(p=>p.file.ext == "md" && !hasValue(p?.found_classifications) );
}else if(userFilesSelection == "Walkthrough classification in current file"){
selectedClassificationToDebug = await selectFileFromFolder("Classifications/Text Classifications",false);
filesToClassify = [dv.page(userCurrentFile.path)];
}else if(userFilesSelection == "Classify markdown files in folder"){
let userFolderSelection = await selectFolder();
if(userFolderSelection[userFolderSelection.length - 1] == "/"){
//dataview does not accept slash at the end of a file path
userFolderSelection = userFolderSelection.substring(0,userFolderSelection.length - 1);
}
filesToClassify = dv.pages(`"${userFolderSelection}"`).where(p=>p.file.ext == "md");
}

var userClassificationNotice;
var classifiedFiles = 0;
var requiredToClassify = filesToClassify?.length;
//notify user that classification has begun
if(typeof(filesToClassify) != "undefined"){
if (userFilesSelection == "Walkthrough classification in current file"){
new tp.obsidian.Notice("Walkthrough started", 8000);
}else{
new tp.obsidian.Notice("Classification started",8000);
userClassificationNotice = new tp.obsidian.Notice("Classified Files\n("+classifiedFiles+"/"+requiredToClassify+")",0);
}
}

for (let page of filesToClassify ){

//Get needed elements from file
let activeFilePath = page.file.path;

let activeFileTFile = getTFile(activeFilePath);

let activeFileText = "";

activeFileText = await readFromTFile(activeFileTFile);

// do not include found classifications in text to search
//remove any existing classifications (the whole comment field it sits within)
let activeFileTextCleaned = activeFileText.replace(/(%%\s*)?{{\(\s+Found_Classifications.*\)}}(\s*%%)?/i,"");
//remove trailing new lines
activeFileTextCleaned = activeFileTextCleaned.replace(/\n+$/,"");


let activeFileTextCleaned_default = activeFileTextCleaned;
activeFileTextCleaned_default = activeFileTextCleaned_default.toLowerCase();
activeFileTextCleaned_default = standardizedPunctuation(activeFileTextCleaned_default);


//Get all relevant classifications

let allClassificationsInFolder;
if(userFilesSelection == "Walkthrough classification in current file"){
allClassificationsInFolder = [selectedClassificationToDebug]
}else{
allClassificationsInFolder = await dv.pages('"Classifications/Text Classifications"').where(p => typeof (p.NoteType) != "undefined").values
}

let relevantClassifications = []


for (let i = 0; i < allClassificationsInFolder.length; i++) {

relevantClassifications.push(allClassificationsInFolder[i].file.path)

}

//Iterate through all relevant classifications
var classificationsFoundInSearch = [];

for (let i = 0; i < relevantClassifications.length; i++) {

//Begin searching text for individual classification


let classificationStatus = "unclassified";

let classificationPath = relevantClassifications[i];

let classificationTFile = getTFile(classificationPath);

let classificationTitle = classificationTFile.basename;

let textInFile = await readFromTFile(classificationTFile);

let conditionsToAttempt = await getTextIndicatorsFromClassification(classificationPath, textInFile, "#### Conditions to Attempt Classification", "#### Stop Classification");

let autoStopIndicators = await getTextIndicatorsFromClassification(classificationPath, textInFile, "#### Stop Classification", "#### Automatically Classify");

let autoClassifyIndicators = await getTextIndicatorsFromClassification(classificationPath, textInFile, "#### Automatically Classify", "## Could be confused with these classifications");

//Order of checking classifications

//pre-filter condition to attempt classification

if (classificationStatus == "unclassified") {

let conditionToAttemptMatches = await foundTextIndicatorValuesInCurrentText(conditionsToAttempt, activeFileTextCleaned, activeFileTextCleaned_default, false, classificationTitle,classificationPath, activeFilePath,"conditionsToAttempt",userFilesSelection);

if (conditionToAttemptMatches.length > 0 || conditionsToAttempt.length == 0) {

//continue attempting classification because found conditions or had no conditions
if (userFilesSelection == "Walkthrough classification in current file") {

await tp.system.prompt(("(Debug Source) " + "Conditions to attempt classification found: " + String.fromCharCode(34) + classificationTitle + String.fromCharCode(34) + "." + " Found in text: " + String.fromCharCode(34) + conditionToAttemptMatches + String.fromCharCode(34) + " (press enter to continue)"))

}

if (autoStopIndicators.length > 0 && classificationStatus == "unclassified") {

let autoStopMatches = await foundTextIndicatorValuesInCurrentText(autoStopIndicators, activeFileTextCleaned, activeFileTextCleaned_default, false, classificationTitle,classificationPath, activeFilePath,"autoStop",userFilesSelection);

if (autoStopMatches.length > 0) {

classificationStatus = "Stop";

if (userFilesSelection == "Walkthrough classification in current file") {

await tp.system.prompt(("(Debug Source) " + "Stopped Classification as: " + String.fromCharCode(34) + classificationTitle + String.fromCharCode(34) + "." + " Found in text: " + String.fromCharCode(34) + autoStopMatches + String.fromCharCode(34) + " (press enter to continue)"))

}

}

}

if (autoClassifyIndicators.length > 0 && classificationStatus == "unclassified") {

let autoMatches = await foundTextIndicatorValuesInCurrentText(autoClassifyIndicators, activeFileTextCleaned, activeFileTextCleaned_default, false, classificationTitle,classificationPath, activeFilePath,"autoClassify",userFilesSelection)

if (autoMatches.length > 0) {

classificationStatus = "Classified"

if (userFilesSelection == "Walkthrough classification in current file") {

await tp.system.prompt(("(Debug Source) " + "Classified as: " + String.fromCharCode(34) + classificationTitle + String.fromCharCode(34) + "." + " Found in text: " + String.fromCharCode(34) + autoMatches + String.fromCharCode(34) + " (press enter to continue)"))

}

}

}


}

}

//console.log("Status: " + classificationPath + " is " + classificationStatus)

if (classificationStatus == "Classified") {
classificationsFoundInSearch.push(classificationTFile);

}

}

//After all classifications have been added in the file

//completed classification of another file
classifiedFiles++;
if (userFilesSelection != "Walkthrough classification in current file"){
userClassificationNotice.setMessage("Classified Files\n("+classifiedFiles+"/"+requiredToClassify+")");
}
//Add found classifications dataview field or replace if it is already present


if (userFilesSelection == "Walkthrough classification in current file"){
//does not affect classification
}else{
addCommentedFoundClassificationsSection(activeFileTFile,activeFileTextCleaned,returnUniqueArray(classificationsFoundInSearch));
}


//after classification of single file
}
//after classification of all selected files

//notify user of status and impact

if (userFilesSelection == "Walkthrough classification in current file"){
new tp.obsidian.Notice("Walkthrough completed", 8000);
}else{
userClassificationNotice.hide();
new tp.obsidian.Notice("Classification completed", 8000);
new tp.obsidian.Notice("Classified ("+requiredToClassify+") Files ", 8000);
}


%>