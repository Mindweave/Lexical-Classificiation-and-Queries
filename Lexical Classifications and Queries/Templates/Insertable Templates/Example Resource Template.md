---
Aliases: 
NoteType: Resource
---
<%* 
function leftDoubleBrackets() {
return "[" + "["
}

function rightDoubleBrackets() {
return "]" + "]"
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
function getRecentFilesLink(selectedFile){
let flatFilesJSON = getRecentFiles();
return leftDoubleBrackets()+flatFilesJSON[selectedFile].path+"|"+flatFilesJSON[selectedFile].basename+rightDoubleBrackets()
}
let accessInstructionsType = await tp.system.suggester(["Custom Text","Go to website","Go to file in Obsidian"],["Custom Text","Go to website","Go to file in Obsidian"],false,"Access instructions"); 
let accessInstructions = ""; 
switch(accessInstructionsType){ 
case "Custom Text": let userInstructions = await tp.system.prompt("Please explain how to access the resource"); accessInstructions = userInstructions; break; 
case "Go to website": let website = await tp.system.prompt("Please enter the link to the website"); accessInstructions = "Go to: "+website; break; 
case "Go to file in Obsidian": let file = ""; let searchFiles = "recentFiles"; 
do { if(file == "(Selected All Files)"){searchFiles = "All Files"}else{ searchFiles = "recentFiles"} if(searchFiles == "recentFiles"){ let suggestPrompts = ["(Search files in folder)"].concat(getRecentFilesTitles()); let suggestReturns = ["(Selected All Files)"].concat(getRecentFilesDetails()); file = await tp.system.suggester(suggestPrompts, suggestReturns,false,"Select File"); }else if(searchFiles == "All Files"){let allFolders = await app.plugins.plugins.dataview.api.pages().groupBy((page) => {return this.app.metadataCache.getFirstLinkpathDest(page.file.path, "").parent.path}); let foldersToSelect = []; for (let value of allFolders){foldersToSelect.push(value.key);} let selectedFolder = await tp.system.suggester(foldersToSelect,foldersToSelect,false,"Select folder to search"); let filesInFolder = await app.plugins.plugins.dataview.api.pages(`"${selectedFolder}"`);  let filesToSelect = ["(Search Recent Files)"];  let filesToSelectPage = ["(Selected Recent Files)"]; for (let value of filesInFolder){filesToSelect.push(value.file.name); filesToSelectPage.push(value);} let selectedFile = await tp.system.suggester(filesToSelect,filesToSelectPage,false,"Select file to add"); if(selectedFile == "(Selected Recent Files)"){file = "(Selected Recent Files)"; continue;} file = selectedFile} }while(file == "(Selected All Files)" || file == "(Selected Recent Files)"); 
let fileSelected =""; if(searchFiles == "recentFiles"){fileSelected = getRecentFilesLink(file) }else{fileSelected = file.file.link; } accessInstructions = "Go to: "+fileSelected; break; let customTextAccessInstructions = await tp.system.prompt("How to access the resource?"); accessInstructions = customTextAccessInstructions; break; } 
let description = await tp.system.prompt("Please describe the content or characteristics of the resource"); 
let usageInstructions = await tp.system.prompt("Please give instructions for how to use and derive value from this resource");
%>
# Resource

## Access Instructions
<%* tR+=accessInstructions %>

## Description
<%* tR+=description %>

## Suggestions on Using the Resource
<%* tR+=usageInstructions %>

