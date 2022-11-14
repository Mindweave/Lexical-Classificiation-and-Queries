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
/* Required Functions*/
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

function delayTextReturn(textToReturn,delayInMilliseconds) {
//Page text needs time to update
//So rather have the button use a delay instead of the user delaying themselves
return new Promise((resolve, reject) => {
     setTimeout(() => {resolve(textToReturn);
     },delayInMilliseconds);
   });
}

async function activateCommand(commandID){
let saveCommandDefinition =this.app.commands.commands[commandID];

let save = saveCommandDefinition?.callback;

if(typeof save === "undefined"){
//sometimes function is held in check callback
save = saveCommandDefinition?.checkCallback;
}

if (typeof save === "function") {
await save()
}
}

async function readFromTFile(selectedTFile) {

return await this.app.vault.read(selectedTFile)

}
let editor = await getEditor()
let highlightedTextUnformatted = tp.file.selection();
let userHighlightedText = (highlightedTextUnformatted.length > 0);
var commandID;
var saveCommandDefinition;
let currentTFile = this.app.workspace.getActiveFile();
let endTFile;

if(userHighlightedText){
commandID = await tp.system.suggester(["Search Classifications","Search for Files with Matching Classifications"],["obsidian-hotkeys-for-templates:templater:Queries/Go to Search Classifications.md","obsidian-hotkeys-for-templates:templater:Queries/Go to Search Similar Classifications.md"]);

}else{
//wait for editor to update. Seems like it is taking time to update text and using old text if pause is not given for sufficient time 
commandID = await tp.system.suggester(["Search Classifications","Search for Files with Matching Classifications"],["obsidian-hotkeys-for-templates:templater:Queries/Go to Search Classifications.md","obsidian-hotkeys-for-templates:templater:Queries/Go to Search Similar Classifications.md"]);
/*
commandID = await delayTextReturn("obsidian-hotkeys-for-templates:templater:Classifications/Classify Text Classifications.md",1500);
*/

}

if(commandID == null  && (tp.file.selection()).length > 0){
//user did not make selection but is not an error
await replaceHighlightedText(tp.file.selection());
}

await activateCommand(commandID);

endTFile = this.app.workspace.getActiveFile();
//replace highlight if still in same file and still has a highlight
/*
let cursorStart = editor.getCursor("from")
let cursorEnd = editor.getCursor("to")
let hasHighlight = cursorStart != cursorEnd
let cursorIsAtTop = cursorStart?.line == 0 && cursorStart?.ch == 0 && cursorEnd?.line == 0 && cursorEnd?.ch == 0
*/

//include exclude replaces the text itself these ones do not
if(userHighlightedText && (commandID == "obsidian-hotkeys-for-templates:templater:Debug classification source.md" || commandID == "obsidian-hotkeys-for-templates:templater:Classifications/Classify Text Classifications.md" ) ){
if(currentTFile.path == endTFile.path){
//still in same file so highlighted text needs to be replaced
await replaceHighlightedText(highlightedTextUnformatted);
}
}

}catch(err){
console.error(err);
await replaceHighlightedText(tp.file.selection());
}

%>