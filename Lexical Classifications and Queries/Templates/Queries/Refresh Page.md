<%*
function hasValue(variableToCheck){
return (typeof(variableToCheck)!="undefined" && variableToCheck != null && variableToCheck?.length != 0)
}

//dataview requires the user to have all the code visible on screen (I suppose) which requires scrolling past it all or highlighting it all. This highlights the page
async function getEditor(){

let MarkdownView = tp.obsidian.MarkdownView

let activeLeaf = app.workspace.getActiveViewOfType(MarkdownView);

if (activeLeaf) {

let view = activeLeaf;

let editor = view.editor;

return editor;
}

}

let editor = await getEditor();
let currentCursorPosition = (editor.getCursor())
editor.setSelection({line: 0,ch: 0},{line: editor.lineCount(), ch: editor.lastLine.length})
if(hasValue(currentCursorPosition)){
//editor.setCursor({line: 0,ch: 0})
editor.setCursor(currentCursorPosition);
}else{
editor.blur()
} 


%>