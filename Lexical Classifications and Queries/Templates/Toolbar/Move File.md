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

function removeSpecialCharactersFromFilePath(noteTitle){
//some characters cannot be used in a title this removes them
noteTitle = noteTitle.replace(/[*"\\<>:|?\r\n\t\f\v\[\]]/gm," ");
noteTitle = noteTitle.replace(/\s+/gm," "); //only one space
return noteTitle.trim(); //no surrounding spaces
}

async function selectFolder(){

let allFolders = getFoldersIn("Scraps"); let currentFolder =  this.app.workspace.getActiveFile().parent.path; let foldersToSelect = ["(Current Folder:"+" "+currentFolder+")","(Create new folder)"]; let foldersToSelectValue = [currentFolder,"(Create new folder)"]; for (let value of allFolders){if(value == currentFolder){continue;}foldersToSelect.push(value);foldersToSelectValue.push(value);} let selectedFolder = await tp.system.suggester((foldersToSelect),(foldersToSelectValue),false,"Select folder to search");
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


function openFile(selectedTFile){
this.app.workspace.getLeaf().openFile(selectedTFile)
}

/*Getting current file information */
let currentPageTFile = app.workspace.getActiveFile();

/*Modal Output*/
var noteEndFolderPath = "";

 //get title and make sure it has the correct format and does not already exist if work has already been done in the note (such as entering prompts)

//variables used in testing if title is valid
var filePathContainsSpecialCharacters = false;
var filePathInvalidLength = false;
var modifyFilePath = false;
var userRequestsToChangeFolder = true;
let actionForAfterTitleAlreadyExists = false;

let noteTitle = currentPageTFile.basename;
let noteEndFilePath = "";//noteEndFolderPath + noteTitle


do{
if(modifyFilePath){
noteTitle = await tp.system.prompt("Edit title",noteTitle,false);
modifyFilePath = false;
}

if(userRequestsToChangeFolder || modifyFilePath){
//use previous run note title as prompt initial text
noteEndFolderPath = await selectFolder();
userRequestsToChangeFolder = false;
}

noteEndFilePath = noteEndFolderPath + noteTitle
filePathContainsSpecialCharacters = noteEndFilePath.search(/[*"\\<>:|?\r\n\t\f\v\[\]]/) != -1;

if(filePathContainsSpecialCharacters){
modifyFilePath = await tp.system.suggester(["Edit File Path","Remove special characters"],[true,false],false,"File Path contains special characters")
if(!modifyFilePath){
//remove special characters and replace them with a space but only one space
noteEndFilePath =  removeSpecialCharactersFromFilePath(noteEndFilePath)
//check if all have been removed
filePathContainsSpecialCharacters = noteEndFilePath.search(/[*"\\<>:|?\r\n\t\f\v\[\]]/) != -1;
}
}

filePathInvalidLength = (noteEndFilePath.length > 250 || noteEndFilePath.length == 0); //seems to be max
if(filePathInvalidLength && !modifyFilePath){
modifyFilePath = await tp.system.suggester(["Edit File Path"],[true],false,"File Path length invalid "+(noteEndFilePath.length == 0 ? "(no text)":"(over 250 characters)"));
continue;
}

titleAlreadyExists = await tp.file.exists(noteEndFilePath+".md"); //should always be markdown


//saving the prompts by requesting a title that does not already exist
if(titleAlreadyExists){
actionForAfterTitleAlreadyExists = await tp.system.suggester(["Edit Title","Change selected folder","Go to existing file"],["Edit Title","Change selected folder","Go to existing file"],false,"Title already exists");
modifyFilePath = actionForAfterTitleAlreadyExists == "Edit Title";
userRequestsToChangeFolder = actionForAfterTitleAlreadyExists == "Change selected folder";
//user already accepted to edit file path (title or file folder) or go to the existing file. So skip other checks
continue;
}

}while ( (filePathContainsSpecialCharacters) || (modifyFilePath) || (userRequestsToChangeFolder))

if(actionForAfterTitleAlreadyExists == "Go to existing file"){
let existingTFile = tp.file.find_tfile(noteEndFilePath)

openFile(existingTFile);
}else{
await tp.file.move(noteEndFilePath,currentPageTFile)
}

}catch(err){
console.error(err);
}finally{
await replaceHighlightedText(tp.file.selection());
}

%>