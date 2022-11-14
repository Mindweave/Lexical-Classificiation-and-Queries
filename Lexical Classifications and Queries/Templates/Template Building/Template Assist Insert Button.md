<%*
/* Required Functions*/
function escapeRegExp(string) {

return string.replace(/([.*+?^=!:${}()|\[\]\/\\])/g, "\\$1");

}
async function getEditor(){
let MarkdownView = tp.obsidian.MarkdownView
const activeLeaf =
this.app.workspace.getActiveViewOfType(MarkdownView);

if (activeLeaf) {

let view = activeLeaf;

let editor = view.editor;

return editor

}
}
function getTFile(filePath){
return this.app.metadataCache.getFirstLinkpathDest(filePath, "");
}
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

function delayTextReturn(textToReturn,delayInMilliseconds) {
//Page text needs time to update
//So rather have the button use a delay instead of the user delaying themselves
return new Promise((resolve, reject) => {
     setTimeout(() => {resolve(textToReturn);
     },delayInMilliseconds);
   });
}

async function readFromTFile(selectedTFile) {

return await this.app.vault.read(selectedTFile)

}

async function runCommand(commandID){
let saveCommandDefinition =this.app.commands.commands[commandID];

var save = saveCommandDefinition?.callback;

if(typeof save === "undefined"){
//sometimes function is held in check callback
save = saveCommandDefinition?.checkCallback;
}

if (typeof save === "function") {
await save()
}
}
let editor = await getEditor()
let highlightedTextUnformatted = tp.file.selection();
let userHighlightedText = (highlightedTextUnformatted.length > 0);
var selectedInsertID;
let currentTFile = this.app.workspace.getActiveFile();
let currentDataviewFile = this.app.plugins.plugins.dataview.api.page(currentTFile.path)
let endTFile;

let templatesInsertIDs = ["Basic template website from highlight","Note Title","Note Title (No Prepend)","Highlight","Note Source","Note Source Title","Note Source Note Type","Text Prompt","Note Source Note Root","Note End Folder","URL Encoded Highlight","Template used by note","Create new basic template","Add Template Data from Template"];
let templatesInsertIDPrompts = ["Basic template website from highlight","Note Title","Note Title (No Prepend)","Highlight","Note Source","Note Source Title","Note Source Note Type","Text Prompt","Note Source Note Root","Note End Folder","URL Encoded Highlight","Template used by note","Create new basic template","Add Template Data from Template"];

let basicInsertIDs = ["editor:insert-callout","nldates-obsidian:nlp-now","nldates-obsidian:nlp-today","nldates-obsidian:nlp-time"];
let basicInsertIDPrompts = ["Insert Callout","Insert the current date and time","Insert today's date","Insert the current time"];

let userInsertIDs  = ["(Done)"];
let userInsertIDPrompts = ["(Done)"];

if(currentDataviewFile.file.path.search(/Templates\//i) > -1){
userInsertIDs = userInsertIDs.concat(templatesInsertIDs);
userInsertIDPrompts = userInsertIDPrompts.concat(templatesInsertIDPrompts);
}

if(true){
//basic insert applies to all notes (can filter this down if there are some which do not apply)

}

if(userHighlightedText){
selectedInsertID = await tp.system.suggester(userInsertIDPrompts,userInsertIDs)
}else{
//wait for editor to update. Seems like it is taking time to update text and using old text if pause is not given for sufficient time 
selectedInsertID = await tp.system.suggester(userInsertIDPrompts,userInsertIDs)
}

//go through each insert command
switch(selectedInsertID){
case "Note Title":
	tR+="VarStart_NoteTitle_VarEnd"
	break;
case "Note Title (No Prepend)":
	tR+="VarStart_NoteTitleNoPrepend_VarEnd"
	break;
case "Highlight":
	tR+="VarStart_NoteHighlight_VarEnd";
	break;
case "Note Source":
	tR+="VarStart_NoteSource_VarEnd";
	break;
case "Note Source Title":
	tR+="VarStart_NoteSourceTitle_VarEnd";
	break;
case "Note Source Note Type":
	tR+="VarStart_NoteSourceType_VarEnd";
	break;
case "Note Source Note Root":
	tR+="VarStart_NoteSourceRoot_VarEnd";
	break;
case "Note End Folder":
	tR+="VarStart_NoteEndFolderPath_VarEnd";
	break;
case "URL Encoded Highlight":
	tR+="VarStart_URLEncodedHighlight_VarEnd";
	break;
case "Template used by note":
	tR+="VarStart_NoteTemplate_VarEnd";
	break;
case "Create new basic template":
	console.log("creating basic template")
	let basicTemplate = await readFromTFile(getTFile("Template Utilities/Base Template"));
	let newTemplateTitle = await tp.system.prompt("Template Name");
	await tp.file.create_new(basicTemplate,newTemplateTitle,true);
	break;
	console.log("should not reach")
case "Basic template website from highlight":
	let searchedURL = (await tp.system.prompt("What is the URL with the text search?"));
	let searchedTextWithSpace = (await tp.system.prompt("What is the text searched in the URL? (must contain a space)"));
	let urlName = await tp.system.prompt("What is the name of this URL?");
	let foundFirstWord = searchedURL.indexOf(searchedTextWithSpace.split(" ")[0])+searchedTextWithSpace.split(" ")[0].length;
	let foundSecondWord = searchedURL.indexOf(searchedTextWithSpace.split(" ")[1],foundFirstWord);
	let spaceCharacter = searchedURL.substring(foundFirstWord,foundSecondWord);
	let joinedOnSpaceCharacter = searchedTextWithSpace.split(" ").join(spaceCharacter);
	let regexString = escapeRegExp(joinedOnSpaceCharacter);
	let regexObject = new RegExp(regexString,'gi');
	let urlFormattedWithVariable = searchedURL.replace(regexObject,`"+searchText+"`);
	let textToAdd = "["+urlName+"]"+`(<%*  searchText = searchTextStatic; searchText = searchText.replace(/\%20/g,"${spaceCharacter}"); tR += "<"+"${urlFormattedWithVariable}"+">" %>)`
	tR+= textToAdd;
	break;
case "Text Prompt":
	let promptNumber = [];
	for (let i = 1; i<25;i++){
	promptNumber.push(i);
	}
	let numberSelected = await tp.system.suggester(promptNumber,promptNumber,false,"Text Prompt Number",promptNumber.length);
	tR+="VarStart_Prompt"+numberSelected+"_VarEnd";
	break;
case "Add Template Data from Template":
	await runCommand("obsidian-hotkeys-for-templates:templater:Template Building/Passthrough Templates.md");
	break;
}


endTFile = this.app.workspace.getActiveFile();
//replace highlight if still in same file and still has a highlight

if(currentTFile.path == endTFile.path){
//still in same file so highlighted text needs to be replaced (in most cases does not though just overwrite)
//await replaceHighlightedText(highlightedTextUnformatted);
}


%>