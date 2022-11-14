<%*
function getTFile(filePath){
return this.app.metadataCache.getFirstLinkpathDest(filePath, "");
}
async function readFromTFile(selectedTFile){
return await this.app.vault.read(selectedTFile)
}

function openFile(selectedTFile){
this.app.workspace.getLeaf().openFile(selectedTFile)
}
function getEditorStartPosition(){
return  {line: 0, ch:0}
}
function getEditorEndPosition(editor){
let lineLength = editor.getLine(editor.lastLine()).length 
return {line: editor.lastLine(), ch: lineLength}
}
async function replaceEntireFile (textToReplace){
let MarkdownView = tp.obsidian.MarkdownView
const activeLeaf =
this.app.workspace.getActiveViewOfType(MarkdownView);
let editor;
if (activeLeaf) {

let view = activeLeaf;

editor = view.editor;

//editor.getRange(getEditorStartPosition(),getEditorEndPosition(editor));
editor.replaceRange(textToReplace,getEditorStartPosition(),getEditorEndPosition(editor));
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

async function getEditor() {

let MarkdownView = tp.obsidian.MarkdownView

const activeLeaf =

this.app.workspace.getActiveViewOfType(MarkdownView);

if (activeLeaf) {

const view = activeLeaf;

return view.editor;

}

}

console.log("starting")
let SourceNotesReplaceTFile = getTFile("Utilities/Template Utilities/Rebuild Files/Rebuild Search Classifications.md")

let textToAdd = await readFromTFile(SourceNotesReplaceTFile)

let editor = await getEditor();
editor.setValue(textToAdd)
editor.blur();
editor.scrollIntoView({from: {line: 0, ch: 0},to: {line: 0, ch: 0}}); //go to top of file

%>