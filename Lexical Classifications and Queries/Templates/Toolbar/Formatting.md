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
userVisibleSelections = ["bold text", "italics text", "add bulleted list to selected lines", "add numbered list to selected lines", "heading 1","heading 2","heading 3","heading 4","heading 5","heading 6", "lowercase text", "uppercase text", "title case capitalizations text", "capitalize first letters text","link text to corresponding note","embed link text (transclusion)"]
outputSelections = ["bold text", "italics text", "bullet list", "number list", "heading 1","heading 2","heading 3","heading 4","heading 5","heading 6", "lowercase text", "uppercase text", "title case capitalizations text", "capitalize first letters text","link text to corresponding note","embed link text"]
}else{
userVisibleSelections = ["Add generic codeblock","add bulleted list to selected line", "add numbered list to selected lines", "heading 1","heading 2","heading 3","heading 4","heading 5","heading 6"]
outputSelections = ["Add generic codeblock", "bullet list", "number list", "heading 1","heading 2","heading 3","heading 4","heading 5","heading 6"]
}
let selectedFormatting = await tp.system.suggester(userVisibleSelections,outputSelections,false,"Select Formatting");

if(selectedFormatting == "Add generic codeblock"){
await activateCommand("cmenu-plugin:codeblock");
}else if(selectedFormatting == "bold text"){
await replaceHighlightedText(String.fromCharCode(42)+String.fromCharCode(42)+highlightedTextUnformatted+String.fromCharCode(42)+String.fromCharCode(42));
}else if(selectedFormatting == "italics text"){
await replaceHighlightedText(String.fromCharCode(42)+highlightedTextUnformatted+String.fromCharCode(42));
}else if(selectedFormatting == "bullet list"){
await addToStartofLines("- ");
}else if(selectedFormatting == "heading 1"){
await addToStartofLines("# ");
}else if(selectedFormatting == "heading 2"){
await addToStartofLines("## ");
}else if(selectedFormatting == "heading 3"){
await addToStartofLines("### ");
}else if(selectedFormatting == "heading 4"){
await addToStartofLines("#### ");
}else if(selectedFormatting == "heading 5"){
await addToStartofLines("##### ");
}else if(selectedFormatting == "heading 6"){
await addToStartofLines("###### ");
}else if(selectedFormatting == "lowercase text"){
await replaceHighlightedText(highlightedTextUnformatted.toLowerCase());
}else if(selectedFormatting == "lowercase text"){
await replaceHighlightedText(highlightedTextUnformatted.toLowerCase());
}else if(selectedFormatting == "uppercase text"){
await replaceHighlightedText(highlightedTextUnformatted.toUpperCase());
}else if(selectedFormatting == "title case capitalizations text"){
await replaceHighlightedText(highlightedTextUnformatted.split(' ')
   .map(w => {if( ['A', 'An', 'The', 'And', 'But', 'Or', 'For', 'Nor', 'As', 'At', 
  'By', 'For', 'From', 'In', 'Into', 'Near', 'Of', 'On', 'Onto', 'To', 'With'].some(r => r.toLowerCase() == w.toLowerCase()) ){return w.toLowerCase()}else{return w[0].toUpperCase() + w.substring(1).toLowerCase()}}).join(' '))
}else if(selectedFormatting == "capitalize first letters text"){
await replaceHighlightedText(highlightedTextUnformatted.split(' ')
   .map(w => w[0].toUpperCase() + w.substring(1).toLowerCase())
   .join(' '));
}else if(selectedFormatting == "embed link text"){
await replaceHighlightedText(String.fromCharCode(33)+String.fromCharCode(91)+String.fromCharCode(91)+highlightedTextUnformatted+String.fromCharCode(93)+String.fromCharCode(93));
}else if(selectedFormatting == "number list"){
await addNumberedListToStartofLines()
}else if(selectedFormatting == "link text to corresponding note"){
await replaceHighlightedText(String.fromCharCode(91)+String.fromCharCode(91)+highlightedTextUnformatted+String.fromCharCode(93)+String.fromCharCode(93));
}
}catch(err){
console.error(err);
await replaceHighlightedText(tp.file.selection());
}

%>