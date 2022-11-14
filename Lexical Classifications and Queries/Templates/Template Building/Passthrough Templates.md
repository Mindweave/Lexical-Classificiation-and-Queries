<%*
async function getEditor(){

let MarkdownView = tp.obsidian.MarkdownView

let activeLeaf = this.app.workspace.getActiveViewOfType(MarkdownView);

if (activeLeaf) {

let view = activeLeaf;

let editor = view.editor;

return editor;
}

}
async function replaceHighlightedText (textToReplace,editor){

editor.replaceSelection(textToReplace);
}

function leftDoubleBrackets() {
return "[" + "["
}

function rightDoubleBrackets() {
return "]" + "]"
}

function getTFile(filePath){

return this.app.metadataCache.getFirstLinkpathDest(filePath, "");
}

async function readFromTFile(selectedTFile){

return await this.app.vault.read(selectedTFile)

}

function openFile(selectedTFile){

this.app.workspace.getLeaf().openFile(selectedTFile)

}

function removeSquareBracketsFromString(stringWithBrackets){return stringWithBrackets.replace(/[\[\]]+/g,"");}

function formatHighlightedText(userUnformattedHighlightedText){

let isTextHighlighted = userUnformattedHighlightedText.length > 0;
let formattedHighlight = userUnformattedHighlightedText;

if(isTextHighlighted){

/*Format highlighted text*/

formattedHighlight = removeSquareBracketsFromString(formattedHighlight); //highlight sometimes used for title (brackets not allowed) may need to remove other symbols too

formattedHighlight = formattedHighlight.replace(/\s+/gm," "); //only one space and no multiple lines

formattedHighlight = formattedHighlight.trim(); //no surrounding spaces

return formattedHighlight

}else{

return "(No Highlight Selected)"

}

}

function variableReplaceHighlight(highlight, replaceInText){
return replaceInText.replaceAll("VarStart_NoteHighlight_VarEnd",highlight)
}

function variableReplaceSourceTitle(sourceTitle, replaceInText){
return replaceInText.replaceAll("VarStart_NoteSourceTitle_VarEnd",sourceTitle)
}

function variableReplaceSourceType(sourceType, replaceInText){
return replaceInText.replaceAll("VarStart_NoteSourceType_VarEnd",sourceType)
}

function variableReplaceSourceRoot(sourceRoot, replaceInText){
return replaceInText.replaceAll("VarStart_NoteSourceRoot_VarEnd",sourceRoot)
}

function variableReplaceSourceLink(sourcelink, replaceInText){
return replaceInText.replaceAll("VarStart_NoteSource_VarEnd",sourcelink)
}

function variableReplaceSourceTitleNoPrepend(sourceTitleNoPrepend, replaceInText){
return replaceInText.replaceAll("VarStart_NoteTitleNoPrepend_VarEnd",sourceTitleNoPrepend)
}

function replaceRemainingVariableFields(replaceInText){
return replaceInText.replaceAll(/\bvarstart_[\d\w_\[\]]*_varend\b/gi,"");
}

function getLinkFromPathAndTitle(path,title){
return (leftDoubleBrackets())+path+"|"+title+(rightDoubleBrackets())
}

function goToExisitingFile(existingFilePath){

let existingTFile = getTFile(existingFilePath)

const notice = new tp.obsidian.Notice("", 8000);

notice.noticeEl.innerHTML = `<b>File Already Exists</b><br/>Opening File: <br/>${existingTFile.basename}`;

openFile(existingTFile);
}
console.log("does it break before starting")
let currentPageTFile = app.workspace.getActiveFile();

let currentPageDataview = await app.plugins.plugins.dataview.api.page(currentPageTFile.path)
console.log("does it break after getting page info")
let userHighlightedText = (tp.file.selection().length > 0);
let userHighlight = tp.file.selection();
let userHighlightedFormatted = formatHighlightedText(userHighlight);

let noteSourceTitle = currentPageDataview.file.name;

let noteSourceTitleNoPrepend = noteSourceTitle; //templates do not have prepend

let noteSourcePath = currentPageDataview.file.path;

let noteSourceLink = getLinkFromPathAndTitle(noteSourcePath,noteSourceTitleNoPrepend)

let noteSourceNoteRoot = typeof(currentPageDataview?.noteroot) == "undefined" ? "":currentPageDataview?.noteroot;

let noteSourceNoteType = typeof(currentPageDataview?.notetype) == "undefined" ? "":currentPageDataview?.notetype;

//Getting Template Text

let endFolderPath = "Utilities/Template Utilities/Templates Data"

let templateText =  await readFromTFile(getTFile("Templates/Template Building/Input Template Data"))

console.log("does it break after adding variables")

let locationToAddFile = endFolderPath+"/"+noteSourceTitle+" Data"
console.log(locationToAddFile)
//Replace highlighted text before so that it is not deleted
var editor = await getEditor()
replaceHighlightedText(tp.file.selection(),editor)

//Check if file already exists
let fileWhereItAlreadyExists = await app.plugins.plugins.dataview.api.pages('"Utilities/Template Utilities/Templates Data"').where(p => p.templatelink.path == noteSourcePath)
let alreadyExists = (fileWhereItAlreadyExists.length>0 || await tp.file.exists(locationToAddFile))

console.log("does it break before checking if file exists"+alreadyExists)

if(alreadyExists){
let existingFilePath ="";
if(fileWhereItAlreadyExists.length>0){
existingFilePath = fileWhereItAlreadyExists[0].file.path
}else{
existingFilePath = locationToAddFile;
}
goToExisitingFile(existingFilePath)
}else{

//Passthrough
templateText = variableReplaceHighlight(userHighlightedFormatted, templateText);
templateText = variableReplaceSourceTitle(noteSourceTitle, templateText);
templateText = variableReplaceSourceType(noteSourceNoteType, templateText);
templateText = variableReplaceSourceRoot(noteSourceNoteRoot, templateText);
templateText = variableReplaceSourceLink(noteSourceLink, templateText)
templateText = variableReplaceSourceTitleNoPrepend(noteSourceTitleNoPrepend, templateText)
templateText = replaceRemainingVariableFields(templateText)

//Create template with variables passed through
await tp.file.create_new(""+templateText+"", locationToAddFile,true);
}
%>