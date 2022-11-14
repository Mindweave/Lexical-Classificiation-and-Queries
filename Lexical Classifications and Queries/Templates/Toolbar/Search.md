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

function getEditor(){
let MarkdownView = tp.obsidian.MarkdownView
const activeLeaf =
this.app.workspace.getActiveViewOfType(MarkdownView);

if (activeLeaf) {

const view = activeLeaf;

return view.editor;
}
}

async function addToStartofLines (textToAddToStart){

let editor = getEditor();
let numberOfLines = editor.getSelection().split(String.fromCharCode(10)).length 
let cursorPosition = editor.getCursor()
let cursorStart = cursorPosition["line"]
let lines = []
for(let i = 0;i<numberOfLines;i++){
let lineToEdit =cursorStart+i 
let currentLine = editor.getLine(lineToEdit).toString()
lines.push(currentLine)
}
editor.setSelection(cursorPosition,cursorPosition);
for(let i = 0;i<numberOfLines;i++){
let lineToEdit =cursorStart+i 
editor.setLine(lineToEdit,textToAddToStart+lines[i])
}
editor.focus()
editor.setCursor({line: cursorPosition["line"]+numberOfLines,ch: cursorPosition["ch"]+lines[-1]+textToAddToStart.length})
}

async function addNumberedListToStartofLines (){

let editor = getEditor();
let numberOfLines = editor.getSelection().split(String.fromCharCode(10)).length 
let cursorPosition = editor.getCursor()
let cursorStart = cursorPosition["line"]
let lines = []
for(let i = 0;i<numberOfLines;i++){
let lineToEdit =cursorStart+i 
let currentLine = editor.getLine(lineToEdit).toString()
lines.push(currentLine)
}
editor.setSelection(cursorPosition,cursorPosition);
for(let i = 0;i<numberOfLines;i++){
let lineToEdit =cursorStart+i 
editor.setLine(lineToEdit,(i+1)+". "+lines[i])
}
editor.focus()
editor.setCursor({line: cursorPosition["line"]+numberOfLines,ch: cursorPosition["ch"]+lines[-1]+(lines.length).toString().length+2})
}



let highlightedTextUnformatted = tp.file.selection();
let isTextHighlighted = highlightedTextUnformatted.length > 0;

let userVisibleSelections;
let outputSelections;

if(isTextHighlighted){
userVisibleSelections = ["Vault Search (Omnisearch)", "Boolean Search (Vantage)","Search In-File","Basic Search In-File", "Search and Replace In-File"]
outputSelections = ["Vault Search (Omnisearch)", "Boolean Search (Vantage)","Search In-File","Basic Search In-File", "Search and Replace In-File"]
}else{
userVisibleSelections = ["Vault Search (Omnisearch)", "Boolean Search (Vantage)","Search In-File","Basic Search In-File", "Search and Replace In-File"]
outputSelections = ["Vault Search (Omnisearch)", "Boolean Search (Vantage)","Search In-File","Basic Search In-File", "Search and Replace In-File"]
}
let selectedFormatting = await tp.system.suggester(userVisibleSelections,outputSelections,false,"Select Search");

if(selectedFormatting == "Vault Search (Omnisearch)"){
await activateCommand("omnisearch:show-modal");
}else if(selectedFormatting == "Boolean Search (Vantage)"){
await activateCommand("vantage-obsidian:build-search");
}else if(selectedFormatting == "Search In-File"){
await activateCommand("omnisearch:show-modal-infile");
}else if(selectedFormatting == "Basic Search In-File"){
await activateCommand("editor:open-search");
}else if(selectedFormatting == "Search and Replace In-File"){
await activateCommand("editor:open-search-replace");
}

}catch(err){
console.error(err);
await replaceHighlightedText(tp.file.selection());
}

%>