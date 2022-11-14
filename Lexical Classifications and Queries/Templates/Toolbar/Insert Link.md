<%*
function getEditor(){
let MarkdownView = tp.obsidian.MarkdownView
const activeLeaf =
this.app.workspace.getActiveViewOfType(MarkdownView);

if (activeLeaf) {

const view = activeLeaf;

return view.editor;
}
}

async function replaceHighlightedText (textToReplace){
let MarkdownView = tp.obsidian.MarkdownView
const activeLeaf =
this.app.workspace.getActiveViewOfType(MarkdownView);

if (activeLeaf) {

const view = activeLeaf;

let editor = view.editor;

editor.replaceSelection(textToReplace);
}
}

try{
function leftDoubleBrackets() {
return "[" + "["
}

function rightDoubleBrackets() {
return "]" + "]"
}

function hasValue(variableToCheck){
return (typeof(variableToCheck)!="undefined" && variableToCheck != null && variableToCheck?.length != 0)
}

function getRecentFiles(){return this.app.plugins.plugins['recent-files-obsidian'].data.recentFiles}
function getRecentFilesTitles(){let flatFilesJSON = getRecentFiles();
/*Modal Output*/
var userVisibleSelections = [];
for (key of Object.keys(flatFilesJSON)){
userVisibleSelections.push(flatFilesJSON[key].basename);
}
return userVisibleSelections;
}
function getRecentFilesDetails(){let flatFilesJSON = getRecentFiles();
/*Modal Output*/
var outputSelections = [];
for (key of Object.keys(flatFilesJSON)){
outputSelections.push(key);
}
return outputSelections;
}
function useHighlightOrDefaultToThis(defaultText){
if(hasValue(tp.file.selection()) > 0){
return tp.file.selection();
}else{
return defaultText
}
}

function getRecentFilesLink(selectedFile){
let flatFilesJSON = getRecentFiles();
return leftDoubleBrackets()+flatFilesJSON[selectedFile].path+"|"+useHighlightOrDefaultToThis(flatFilesJSON[selectedFile].basename)+rightDoubleBrackets()
}

//select file
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

async function notifyUser(notificationText){
//notificationText can include html tags
let notice = new tp.obsidian.Notice("", 8000);
notice.noticeEl.innerHTML = notificationText;
}

async function selectFileFromFolder(selectedFolder,allowSkip){
let filesInFolder = await allFilesInFolder(selectedFolder);  let filesToSelect = ["(Search Recent Files)"];  let 
filesToSelectPage = ["(Selected Recent Files)"]; if(allowSkip){filesToSelect.push("(Skip)"); filesToSelectPage.push("(Skip)");} filesToSelect = addPageTitles(filesToSelect,filesInFolder); filesToSelectPage = addFullPages(filesToSelectPage,filesInFolder); let selectedFile = await tp.system.suggester(filesToSelect,filesToSelectPage,false,"Select file to add"); 
return leftDoubleBrackets()+selectedFile.file.path+"|"+useHighlightOrDefaultToThis(selectedFile.file.name)+rightDoubleBrackets();
}

async function selectObsidianFile(){ 
let file = ""; let searchFiles = "recentFiles";

do { if(file == "(Selected All Files)"){searchFiles = "All Files"}else{ searchFiles = "recentFiles"} if(searchFiles == "recentFiles"){ let suggestPrompts = ["(Search files in folder)"].concat(getRecentFilesTitles()); let suggestReturns = ["(Selected All Files)"].concat(getRecentFilesDetails()); file = await tp.system.suggester(suggestPrompts, suggestReturns,false,"Select File"); }else if(searchFiles == "All Files"){let allFolders = await app.plugins.plugins.dataview.api.pages().groupBy((page) => {return this.app.metadataCache.getFirstLinkpathDest(page.file.path, "").parent.path}); let foldersToSelect = []; for (let value of allFolders){foldersToSelect.push(value.key);} let selectedFolder = await tp.system.suggester(foldersToSelect,foldersToSelect,false,"Select folder to search"); 
if((await allFilesInFolder(selectedFolder))?.values?.length > 0){
let selectedFile = await selectFileFromFolder(selectedFolder,false);
if(selectedFile == "(Selected Recent Files)"){file = "(Selected Recent Files)"; continue;} file = selectedFile}else{
    await notifyUser("No files in selected folder")
    return "";
    }} }while(file == "(Selected All Files)" || file == "(Selected Recent Files)");

let fileSelected =""; if(searchFiles == "recentFiles"){fileSelected = getRecentFilesLink(file) }else{fileSelected = file; } 

return fileSelected;
}


var editor = getEditor();
var currentCursorPosition = editor.getCursor('head');
var linkToAdd = await selectObsidianFile();
await replaceHighlightedText(linkToAdd);
}catch(err){
console.error(err)
await replaceHighlightedText(tp.file.selection());
}finally{
//put cursor back to intended position
editor.focus()
editor.setCursor({line: currentCursorPosition.line,ch:currentCursorPosition.ch+(hasValue(linkToAdd?.length) ? linkToAdd?.length:0) })
}
//since it is meant to insert it should not replace if successful
%>