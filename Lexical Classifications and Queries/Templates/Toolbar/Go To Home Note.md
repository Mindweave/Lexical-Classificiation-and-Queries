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
function getTFile(filePath){
return this.app.metadataCache.getFirstLinkpathDest(filePath, "");
}

function openFile(selectedTFile){
this.app.workspace.getLeaf().openFile(selectedTFile)
}

openFile(getTFile("00 Home Note/00 Home Note.md"));
}catch(err){
console.error(err)
}finally{
await replaceHighlightedText(tp.file.selection());
}
%>