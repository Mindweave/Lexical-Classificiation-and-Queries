<%*
async function replaceHighlightedText (textToReplace){
let MarkdownView = tp.obsidian.MarkdownView
const activeLeaf =
this.app.workspace.getActiveViewOfType(MarkdownView);

if (activeLeaf) {

const view = activeLeaf;

const editor = view.editor;

editor.replaceSelection(textToReplace);
}
}

try{

 /*Required Functions*/
function getFoldersIn(folderPath){
let ar = []
let paths = []
function extract(node){
    if(typeof(node?.extension) == "undefined"){
    //is a folder not a file
        ar.push(node);
    if(typeof(node?.children)!="undefined"){
        for(let child of node.children){
        //has children iterate deeper
        extract(child)
        }
    }else{
    //not parent
    ar.push(node)
    }
    }
}
extract(app.vault.getAbstractFileByPath(folderPath))
paths = ar.map(a => a.path);
return (paths)
}

async function createFileLocationIfBlank(fileLocation){
let fileLocationExists = await this.app.vault.exists(fileLocation)
if(!fileLocationExists){
await this.app.vault.createFolder(fileLocation);
} 
}

async function selectFolder(){

let allFolders = getFoldersIn("Search/Queries/Saved Queries"); let currentFolder =  this.app.workspace.getActiveFile().parent.path; let foldersToSelect = ["(Current Folder:"+" "+currentFolder+")","(Create new folder)"]; let foldersToSelectValue = [currentFolder,"(Create new folder)"]; for (let value of allFolders){if(value == currentFolder){continue;}foldersToSelect.push(value);foldersToSelectValue.push(value);} let selectedFolder = await tp.system.suggester((foldersToSelect),(foldersToSelectValue),false,"Select folder to search");
if(selectedFolder != "(Create new folder)"){
if(selectedFolder[selectedFolder.length-1] != "/"){
return selectedFolder+"/"
}
return selectedFolder
}else{
let startingFolderText = await tp.system.suggester((allFolders),(allFolders),false,"Select folder as starting text");
let folderText = await tp.system.prompt("Edit new folder file path",startingFolderText,false);
await createFileLocationIfBlank(folderText);
if(folderText[folderText.length-1] != "/"){
return folderText+"/"
}
return folderText
}
}




function isValidHttpUrl(string) {
/*https://stackoverflow.com/questions/5717093/check-if-a-javascript-string-is-a-url */
  let url;
  
  try {
    url = new URL(string);
  } catch (_) {
    return false;  
  }

  return url.protocol === "http:" || url.protocol === "https:";
}

function removeSpecialCharactersFromTitle(noteTitle){
//some characters cannot be used in a title this removes them
noteTitle = noteTitle.replace(/[*"\/\\<>:|?\r\n\t\f\v\[\]]/gm," ");
noteTitle = noteTitle.replace(/\s+/gm," "); //only one space
return noteTitle.trim(); //no surrounding spaces
}

function enterCharacter(){
    return String.fromCharCode(10);
}

function leftDoubleBrackets() {
return "[" + "["
}

function rightDoubleBrackets() {
return "]" + "]"
}

function removeSquareBracketsFromString(stringWithBrackets){return stringWithBrackets.replace(/[\[\]]+/g,"");}

async function getCurrentSourceNotesOfFile(filePath){
let fileInfo = await app.plugins.plugins.dataview.api.page(filePath)
let fileSourceNotes = fileInfo?.sourcenotes;

if(typeof fileSourceNotes?.length == "undefined"){
//is a single link not an array or missing source note
if(typeof (fileSourceNotes?.path) == "undefined"){
//is missing source note
return []
}else{
return [].concat(fileSourceNotes?.path);
}
}else{
return fileSourceNotes.map(a => a.path); //return the paths of the source notes links
}
}

function stringInObject (stringValue,ObjectValues){
return ObjectValues.includes(stringValue)
}

function titleFormattedText(mySentence){ 

let words = mySentence.split(" ");

for (let i = 0; i < words.length; i++) {
 if(words[i]){
 words[i] = words[i][0].toUpperCase() + words[i].substr(1);
}

}

return words.join(" "); }

async function getEditor(){

let MarkdownView = tp.obsidian.MarkdownView

let activeLeaf = this.app.workspace.getActiveViewOfType(MarkdownView);

if (activeLeaf) {

let view = activeLeaf;

let editor = view.editor;

return editor;
}

}


async function replaceHighlightedText (textToReplace,editor){

editor.replaceSelection(textToReplace);
}

function getTFile(filePath){

return this.app.metadataCache.getFirstLinkpathDest(filePath, "");
}

function openFile(selectedTFile){

this.app.workspace.getLeaf().openFile(selectedTFile)

}

function hasValue(variableToCheck){
return (typeof(variableToCheck)!="undefined" && variableToCheck != null && variableToCheck?.length != 0)
}

async function readFromTFile(selectedTFile){

return await this.app.vault.read(selectedTFile)

}


async function getNoteTitle(userHighlightedText,userHighlightedTextUnformatted,promptInitialText){

//Templates with unique IDs
//old code

let isTextHighlighted = userHighlightedText.length > 0 && (userHighlightedText != "(No Highlight Selected)" || userHighlightedTextUnformatted == "(No Highlight Selected)") ;

let maxHighlightTitleCharacterLength = 150;

let initialText; 

if( promptInitialText == userHighlightedText){
initialText = (userHighlightedText?.length > 0 && isTextHighlighted) ? titleFormattedText(userHighlightedText):"";
}else{
initialText = promptInitialText;
}

//custom text
let userTitleInsteadofHighlight = await tp.system.prompt("File title?"+"(Text, Extract Title from URL, or Leave Blank)",initialText);
if(hasValue(userTitleInsteadofHighlight)){
if(isValidHttpUrl(userTitleInsteadofHighlight)){
return removeSpecialCharactersFromTitle(await tp.user.website("title",tp,userTitleInsteadofHighlight));
}else{
return userTitleInsteadofHighlight
}
}else{
let timeNow = tp.date.now("YYYY-MM-DD_HH_mm_ss")
let newFileTitle = "Untitled "+timeNow;
return newFileTitle; 
}
}


function formatHighlightedText(userUnformattedHighlightedText){

let isTextHighlighted = userUnformattedHighlightedText.length > 0;
let formattedHighlight = userUnformattedHighlightedText;

if(isTextHighlighted){

/*Format highlighted text*/

formattedHighlight = removeSquareBracketsFromString(formattedHighlight); //highlight sometimes used for title (brackets not allowed) may need to remove other symbols too

formattedHighlight = formattedHighlight.replace(/\s+/gm," "); //only one space and no multiple lines

formattedHighlight = formattedHighlight.trim(); //no surrounding spaces

return formattedHighlight

}else{

return "(No Highlight Selected)"

}

}




/*Getting current file information */

let currentPageTFile = app.workspace.getActiveFile();

let currentPageDataview = await app.plugins.plugins.dataview.api.page(currentPageTFile.path)


var userHighlightedText = (tp.file.selection().length > 0);

var thisTemplate = tp.config.template_file.basename

/*Modal Output*/
var noteEndFolderPath = "";
noteEndFolderPath = await selectFolder();


/*Getting text to pass through from current file and insert into template*/


var highlightedTextUnformatted = tp.file.selection();
var highlightedText = "";
highlightedText = formatHighlightedText(highlightedTextUnformatted);


var noteEndFilePath = "";

var editor = await getEditor()

 //get title and make sure it has the correct format and does not already exist if work has already been done in the note (such as entering prompts)
var noteTitle = "";
var noteTitleNoPrepend = "";
//variables used in testing if title is valid
var titleAlreadyExists = false;
var titleContainsSpecialCharacters = false;
var titleInvalidLength = false;
var modifyTitle = false;
var userRequestsToChangeFolder = false;


do{
modifyTitle = false;
if(userRequestsToChangeFolder){
//use previous run note title as prompt initial text
noteEndFolderPath = await selectFolder();
userRequestsToChangeFolder = false;
noteTitleNoPrepend = await getNoteTitle(highlightedText,highlightedTextUnformatted,noteTitleNoPrepend);
}else{
noteTitleNoPrepend = await getNoteTitle(highlightedText,highlightedTextUnformatted,highlightedText);
}

if(noteTitleNoPrepend?.length == 0 || typeof(noteTitleNoPrepend?.length) == "undefined" ){
//did not receive valid response start again. Likely from website title not providing title
noteTitle = ""
}else{
noteTitle = noteTitleNoPrepend;

noteTitle = noteTitle.trim()

}


titleContainsSpecialCharacters = noteTitle.search(/[*"\/\\<>:|?\r\n\t\f\v\[\]]/) != -1;

if(titleContainsSpecialCharacters){
modifyTitle = await tp.system.suggester(["Edit Title","Remove special characters"],[true,false],false,"Title contains special characters")
if(!modifyTitle){
//remove special characters and replace them with a space but only one space
noteTitle =  removeSpecialCharactersFromTitle(noteTitle)
//check if all have been removed
titleContainsSpecialCharacters = noteTitle.search(/[*"\/\\<>:|?\r\n\t\f\v\[\]]/) != -1;
}
}

titleInvalidLength = (noteTitle.length > 150 || noteTitle.length == 0);
if(titleInvalidLength && !modifyTitle){
modifyTitle = await tp.system.suggester(["Edit Title"],[true],false,"Title length invalid "+(noteTitle.length == 0 ? "(no text)":"(over 150 characters)"));
continue;
}


noteEndFilePath = noteEndFolderPath + noteTitle+".md"

titleAlreadyExists = await tp.file.exists(noteEndFilePath);

//saving the prompts by requesting a title that does not already exist
if(titleAlreadyExists){
let actionForAfterTitleAlreadyExists = await tp.system.suggester(["Edit Title","Change selected folder","Go to existing file"],["Edit Title","Change selected folder","Go to existing file"],false,"Title already exists");
modifyTitle = actionForAfterTitleAlreadyExists == "Edit Title";
userRequestsToChangeFolder = actionForAfterTitleAlreadyExists == "Change selected folder";
//user already accepted to edit file path (title or file folder) or go to the existing file. So skip other checks
continue;
}

}while ( (titleContainsSpecialCharacters) || (modifyTitle) || (userRequestsToChangeFolder))

//Get note end link after title and path have been decided

var noteEndLink = "";
noteEndLink = leftDoubleBrackets()+noteEndFolderPath + noteTitle+"|"+noteTitle+rightDoubleBrackets();


//when activating a template it deletes currently highlighted text, this add it back in and adds square brackets around if it matches the highlight

let linkHighlightManually = false;
if(highlightedTextUnformatted?.length > 0){
//ask if user wants to link highlight
 linkHighlightManually = await tp.system.suggester(["Yes, link highlight","No, do not link highlight"],[true,false],false,"Link highlight to file?");
}

if(linkHighlightManually){
let highlightIntoLink =   leftDoubleBrackets()+noteEndFolderPath + noteTitle+"|"+highlightedTextUnformatted.trim()+rightDoubleBrackets();

//add spaces back in outside of the link
if(highlightedTextUnformatted[0] == " "){
highlightIntoLink = " "+highlightIntoLink
}
if(highlightedTextUnformatted[highlightedTextUnformatted.length-1] == " "){
highlightIntoLink = highlightIntoLink+" "
}

await replaceHighlightedText(highlightIntoLink,editor); 

}else{
await replaceHighlightedText(highlightedTextUnformatted,editor); 
}

if(await tp.file.exists(noteEndFilePath)){

const notice = new tp.obsidian.Notice("", 8000);

notice.noticeEl.innerHTML = `<b>File Already Exists</b><br/>Opening File: <br/>${noteTitle}`;

let existingTFile = getTFile(noteEndFilePath)

openFile(existingTFile);

}else{

//check if the end folder path exists and if not then create the folder
 createFileLocationIfBlank(noteEndFolderPath);
 
//create template at end folder path and go to the newly created file

let newFileText = await readFromTFile(this.app.workspace.getActiveFile()); //get current file text
await tp.file.create_new(newFileText, noteEndFolderPath+noteTitle,true);

}

//Editor Instructions post-file creation
editor.focus(); //so cursor is in file
}catch(err){
console.error(err)
}finally{
await replaceHighlightedText(tp.file.selection());
}
%>