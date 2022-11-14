<%*
function standardizedPunctuation(textUnformatted) {
let textFormatted = textUnformatted;
textFormatted = textFormatted.replaceAll("’", "'");
textFormatted = textFormatted.replaceAll("‘", "'");
textFormatted = textFormatted.replaceAll('“', '"');
textFormatted = textFormatted.replaceAll('”', '"');
return textFormatted
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

try{
 /*Required Functions*/

function isValidHttpUrl(string) {
/*https://stackoverflow.com/questions/5717093/check-if-a-javascript-string-is-a-url */
  let url;
  
  try {
    url = new URL(string);
  } catch (_) {
    return false;  
  }

  return url.protocol === "http:" || url.protocol === "https:";
}

function removeSpecialCharactersFromTitle(noteTitle){
//some characters cannot be used in a title this removes them
noteTitle = noteTitle.replace(/[*"\/\\<>:|?\r\n\t\f\v\[\]]/gm," ");
noteTitle = noteTitle.replace(/\s+/gm," "); //only one space
return noteTitle;
}

function enterCharacter(){
    return String.fromCharCode(10);
}

function leftDoubleBrackets() {
return "[" + "["
}

function rightDoubleBrackets() {
return "]" + "]"
}

function removeSquareBracketsFromString(stringWithBrackets){return stringWithBrackets.replace(/[\[\]]+/g,"");}

async function getCurrentSourceNotesOfFile(filePath){
let fileInfo = await app.plugins.plugins.dataview.api.page(filePath)
let fileSourceNotes = fileInfo?.sourcenotes;

if(typeof fileSourceNotes?.length == "undefined"){
//is a single link not an array or missing source note
if(typeof (fileSourceNotes?.path) == "undefined"){
//is missing source note
return []
}else{
return [].concat(fileSourceNotes?.path);
}
}else{
return fileSourceNotes.map(a => a.path); //return the paths of the source notes links
}
}

function stringInObject (stringValue,ObjectValues){
return ObjectValues.includes(stringValue)
}

function titleFormattedText(mySentence){ 

let words = mySentence.split(" ");

for (let i = 0; i < words.length; i++) {
 if(words[i]){
 words[i] = words[i][0].toUpperCase() + words[i].substr(1);
}

}

return words.join(" "); }

async function getEditor(){

let MarkdownView = tp.obsidian.MarkdownView

let activeLeaf = this.app.workspace.getActiveViewOfType(MarkdownView);

if (activeLeaf) {

let view = activeLeaf;

let editor = view.editor;

return editor;
}

}

function replaceUniqueSectionOfEditor(startingValue, endingValue, replacementText){
console.log("replacing unique section of editor")
for(let i = 0;i<editor.lineCount();i++){
if(editor.getLine(i).indexOf(startingValue) > -1 && //get unique value 
   editor.getLine(i).indexOf("replaceUniqueSectionOfEditor") == -1){ //Not in query
startingPosition = {
        line: i,
        ch: editor.getLine(i).indexOf(startingValue),
      }
break;
}
}

for(let i = 0;i<editor.lineCount();i++){
if(editor.getLine(i).indexOf(endingValue) > -1 && //get unique value 
   editor.getLine(i).indexOf("replaceUniqueSectionOfEditor") == -1){ //Not in query
endingPosition = {
        line: i,
        ch: editor.getLine(i).indexOf(endingValue)+endingValue.length,
      }
break;
}
}
if(typeof(endingPosition) == "object" && typeof(startingPosition) == "object"){
if(endingPosition.line>startingPosition.line || (endingPosition.line == startingPosition.line && endingPosition.ch >= startingPosition.ch ) ){
console.log("replacing");
editor.replaceRange(replacementText,startingPosition,endingPosition)
}else{console.log("incorrect order");}
}else{
console.log("didn't find")
}
//editor.replaceRange(replacementText,startingPosition,endingPosition)
//replaceUniqueSectionOfEditor("abcdefg","abcdez","wow")
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

function hasValue(variableToCheck){
return (typeof(variableToCheck)!="undefined" && variableToCheck != null && variableToCheck?.length != 0)
}

function prependNoteTitle(templateData){

let titlePrepend = templateData?.TitlePrepend

if(!hasValue(titlePrepend)){
titlePrepend = " ";
}

if(titlePrepend.charAt(titlePrepend.length-1)==" "){

//make sure there is a space before the prepend and the actual title

return titlePrepend

}else{

return titlePrepend+" "

}

}

async function getNoteTitle(userHighlightedText,userHighlightedTextUnformatted,templateData){

//Templates with unique IDs
if(["DemarcatingDefinition","DemarcatingTypeOf"].includes(templateData?.TemplateNoteType)){
return tp.date.now("YYYY-MM-DD_HH_mm_ss")
}

let isTextHighlighted = userHighlightedText.length > 0 && (userHighlightedText != "(No Highlight Selected)" || userHighlightedTextUnformatted == "(No Highlight Selected)") ;

let maxHighlightTitleCharacterLength = 150;

if(isTextHighlighted && userHighlightedText.length<= maxHighlightTitleCharacterLength && (templateData?.TemplateRequestHighlightAsTitle == 0 || typeof(templateData?.TemplateRequestHighlightAsTitle) == "undefined")){

return titleFormattedText(userHighlightedText);

}else if(isTextHighlighted && userHighlightedText.length<= maxHighlightTitleCharacterLength && (templateData?.TemplateRequestHighlightAsTitle == 1 || typeof(templateData?.TemplateRequestHighlightAsTitle) == "undefined")){

let useHighlightAsTitle = await tp.system.suggester(["Use highlight for title","Use custom text for title"],[true,false]);
if(useHighlightAsTitle){
    return titleFormattedText(userHighlightedText);
}else{
    //custom text
    let userTitleInsteadofHighlight = await tp.system.prompt("What is the title for the new "+templateData?.TemplateName+"?");
	if(isValidHttpUrl(userTitleInsteadofHighlight)){
	return removeSpecialCharactersFromTitle(await tp.user.website("title",tp,userTitleInsteadofHighlight));
	}else{
	return userTitleInsteadofHighlight
	}
}
}else{
let userTitle = await tp.system.prompt("What is the title for the new "+templateData?.TemplateName+"?");

//Websites get title
if(isValidHttpUrl(userTitle)){
	return removeSpecialCharactersFromTitle(await tp.user.website("title",tp,userTitle));
	}else{
	return userTitle;
	}


}

}

function addDateToTitleIfNeeded(templateData,currentNoteTitle){

if(templateData?.TemplateIncludeDateInTitle == 1){

return  currentNoteTitle + " ("+tp.date.now("YYYY-MM-DD_HH-mm")+")"

}

return currentNoteTitle

}

function updateNoteSourceFileWithLink(linkPath,linkName){

return leftDoubleBrackets() +linkPath+"|"+linkName+ rightDoubleBrackets()

}

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

function replaceRemainingVariableFields(currentTemplateText){

return currentTemplateText.replaceAll(/\bvarstart_[\d\w_\[\]]*_varend\b/gi,"");

}


async function basicReplaceTemplatePromptWithUserInput(arrayOfPrompts,templateTextToEdit){

let promptsOutput = []
if(arrayOfPrompts != undefined){
for(let i = 0; i<arrayOfPrompts.length;i++){

promptsOutput.push(await tp.system.prompt(arrayOfPrompts[i]));

/*Replace custom variables in template with acquired data*/

templateTextToEdit = templateTextToEdit.replaceAll("VarStart_Prompt"+(i+1)+"_VarEnd",promptsOutput[i]);

}
}

return templateTextToEdit;

}

async function currentFileEdits(dvCurrentFile,templateData,endNoteLink){

if(dvCurrentFile?.NoteType == "FreeFormAnswer" && templateData?.TemplateNoteType == "FreeFormQuestion"){
replaceUniqueSectionOfEditor("## Relevant Questions (links):", "## Relevant Questions (links):", "## Relevant Questions (links):"+enterCharacter()+"-"+" "+endNoteLink);
}

if(dvCurrentFile?.NoteType == "FreeFormQuestion" && templateData?.TemplateNoteType == "FreeFormAnswer"){
//console.log("in question add answer")
replaceUniqueSectionOfEditor("## Answered By (links):", "## Answered By (links):", "## Answered By (links):"+enterCharacter()+"-"+" "+endNoteLink);
}
}

async function createFileLocationIfBlank(fileLocation){
let fileLocationExists = await this.app.vault.exists(fileLocation)
if(!fileLocationExists){
await this.app.vault.createFolder(fileLocation);
} 
}

/*Getting current file information */
let currentPageTFile = app.workspace.getActiveFile();

let currentPageDataview = await app.plugins.plugins.dataview.api.page(currentPageTFile.path)

/*Getting template*/
console.log("does it break before adding variables?");

var templatesFolderPath="Templates/";

var templatesSuffix = ".md"

var selectedTemplatePath;

var selectedTemplateName;

var selectedTemplateTFile;

var currentFolder = tp.file.folder();

var templateText;

var templateTFile;



var selectedTemplate;


var userHighlightedText = (tp.file.selection().length > 0);

var thisTemplate = tp.config.template_file.basename

var templatesWhereTemplateIsAutoSelected = ["Note Significance Template"]
/*
console.log("Templates check needed")
console.log(!templatesWhereTemplateIsAutoSelected.includes(thisTemplate))
console.log(templatesWhereTemplateIsAutoSelected);
console.log(thisTemplate)
*/
/*Requirements for templates shown in list*/
if(!templatesWhereTemplateIsAutoSelected.includes(thisTemplate) ){ //list of templates to automatically select template

/*Modal Output*/

var userVisibleSelections = [];

var outputSelections = [];

let templatesData = await app.plugins.plugins.dataview.api.pages('"Utilities/Template Utilities/Templates Data"')

//filter due to template name
if(thisTemplate== "Classification Template"){

templatesData = templatesData.where(p => {return (p.EndFolderPath.indexOf("Classifications/Text Classifications") != -1) })

}else if(thisTemplate == "Online Searches Template"){
templatesData = templatesData.where(p => {return (p.EndFolderPath.indexOf("Search/Online Searches") != -1) })

}

for (page of templatesData){

if(

//Does not need highlight or has highlight

((page?.TemplateRequiresHighlight == 0) || userHighlightedText)

){

userVisibleSelections.push(page?.TemplateAction);

outputSelections.push(page?.file.path);

}

}

selectedTemplate = await tp.system.suggester(userVisibleSelections,outputSelections,false,"Select Template");  //gets file path
console.log("getting file path")
console.log(selectedTemplate)
selectedTemplate = await app.plugins.plugins.dataview.api.page(selectedTemplate); //gets template data page
console.log("Page itself")
console.log(Object.keys(selectedTemplate))
}else{
//is a template which is selected automatically
if(thisTemplate == "Note Significance Template"){
let matchingPage = (await app.plugins.plugins.dataview.api.pages('"Utilities/Template Utilities/Templates Data"').where(p => p.TemplateName == "Note Significance"))[0]
selectedTemplate = await app.plugins.plugins.dataview.api.page(matchingPage.file.path);
console.log("selected template automatically")
}else{
//is a template which is selected automatically
console.log("Template selected not found. Was added to templatesWhereTemplateIsAutoSelected but did not provide template to auto select")
}

}

selectedTemplateName = selectedTemplate.TemplateName
console.log("link")
console.log(selectedTemplate.TemplateLink)
selectedTemplatePath = selectedTemplate.TemplateLink.path;

console.log("does it break after suggesting template");

/*Getting text from selected Template*/

templateTFile = getTFile(selectedTemplatePath);

templateText = await this.app.vault.read(templateTFile)

/*Getting text to pass through from current file and insert into template*/


var highlightedTextUnformatted = tp.file.selection();
var highlightedText = "";
highlightedText = formatHighlightedText(highlightedTextUnformatted);

var noteSourcePath = "";
noteSourcePath = currentPageTFile.path;

var noteSourceName = "";
noteSourceName = currentPageTFile.basename

var noteSourceFile = "";
noteSourceFile = updateNoteSourceFileWithLink(noteSourcePath,noteSourceName);

var noteEndFolderPath = "";
noteEndFolderPath = selectedTemplate?.EndFolderPath;

var noteTemplate = selectedTemplate.file.link.toString();

var noteEndFilePath = "";

var editor = await getEditor()

await replaceHighlightedText(highlightedTextUnformatted,editor); //when activating a template it deletes currently highlighted text, this add it back in

//get user input on text prompts
var textPrompts = [];
textPrompts = selectedTemplate?.TemplateTextPrompts;
textPrompts = Object.values(JSON.parse(textPrompts)) //format into array
var textPromptsBeforeTitle =  selectedTemplate?.TemplateTextPromptsBeforeTitle == 1 ? true : false;
if(textPrompts != undefined &&  textPromptsBeforeTitle){
templateText = await basicReplaceTemplatePromptWithUserInput(textPrompts, templateText);
}

 //get title and make sure it has the correct format and does not already exist if work has already been done in the note (such as entering prompts)
var noteTitle = "";
var noteTitleNoPrepend = "";
//variables used in testing if title is valid
var titleAlreadyExists = false;
var titleContainsSpecialCharacters = false;
var titleInvalidLength = false;
var modifyTitle = false;
do{
modifyTitle = false;
noteTitle = prependNoteTitle(selectedTemplate);

noteTitleNoPrepend = await getNoteTitle(highlightedText,highlightedTextUnformatted,selectedTemplate);

if(noteTitleNoPrepend?.length == 0 || typeof(noteTitleNoPrepend?.length) == "undefined" ){
//did not receive valid response start again. Likely from website title not providing title
noteTitle = ""
}else{
noteTitle += noteTitleNoPrepend;

noteTitle = addDateToTitleIfNeeded(selectedTemplate,noteTitle);

noteTitle = noteTitle.trim()

}

noteEndFilePath = noteEndFolderPath + noteTitle+".md"

titleAlreadyExists = tp.file.exists(noteEndFilePath);

//saving the prompts by requesting a title that does not already exist
if(textPromptsBeforeTitle && titleAlreadyExists){
modifyTitle = await tp.system.suggester(["Edit Title","Go to existing file"],[true,false],false,"Title already exists");
//user already accepted to edit title or go to the other file. So skip other checks
continue;
}

titleContainsSpecialCharacters = noteTitle.search(/[*"\/\\<>:|?\r\n\t\f\v\[\]]/) != -1;

if(titleContainsSpecialCharacters){
modifyTitle = await tp.system.suggester(["Edit Title","Remove special characters"],[true,false],false,"Title contains special characters")
if(!modifyTitle){
//remove special characters and replace them with a space but only one space
noteTitle =  removeSpecialCharactersFromTitle(noteTitle)
//check if all have been removed
titleContainsSpecialCharacters = noteTitle.search(/[*"\/\\<>:|?\r\n\t\f\v\[\]]/) != -1;
}
}

titleInvalidLength = (noteTitle.length > 150 || noteTitle.length == 0);
if(titleInvalidLength && !modifyTitle){
modifyTitle = await tp.system.suggester(["Edit Title"],[true],false,"Title length invalid "+(noteTitle.length == 0 ? "(no text)":"(over 150 characters)"))
}

}while ( (titleContainsSpecialCharacters) || (modifyTitle) )

//Get note end link after title and path have been decided

var noteEndLink = "";
noteEndLink = leftDoubleBrackets()+noteEndFolderPath + noteTitle+"|"+noteTitle+rightDoubleBrackets();

//get text prompts after user determines title
if(textPrompts != undefined &&  !textPromptsBeforeTitle){
templateText = await basicReplaceTemplatePromptWithUserInput(textPrompts, templateText);
}


//Replace prompts
templateText = templateText.replaceAll("VarStart_NoteTitle_VarEnd",noteTitle);

templateText = templateText.replaceAll("VarStart_NoteTitleNoPrepend_VarEnd",noteTitleNoPrepend);

templateText = templateText.replaceAll("VarStart_NoteHighlight_VarEnd",highlightedText);

templateText = templateText.replaceAll("VarStart_NoteSource_VarEnd",noteSourceFile);

templateText = templateText.replaceAll("VarStart_URLEncodedHighlight_VarEnd",encodeURIComponent(standardizedPunctuation(highlightedText)));

templateText = templateText.replaceAll("VarStart_NoteEndFolderPath_VarEnd",noteEndFolderPath);

templateText = templateText.replaceAll("VarStart_NoteTemplate_VarEnd",noteTemplate);

//replace remaining prompts using regex match starts with ends with variable text

templateText = replaceRemainingVariableFields(templateText);

console.log("does it break before adding file?");

//actions to take on current file before creating new file

await currentFileEdits(currentPageDataview,selectedTemplate,noteEndLink)
console.log("does it break after current file edits?");
//start creating the new file or use existing
console.log("Note end file path?");
console.log(noteEndFilePath)
if(await tp.file.exists(noteEndFilePath)){

const notice = new tp.obsidian.Notice("", 8000);

notice.noticeEl.innerHTML = `<b>File Already Exists</b><br/>Opening File: <br/>${noteTitle}`;

let existingTFile = getTFile(noteEndFilePath)

//add current source note to source notes if not already found

let currentSourceNotes = await getCurrentSourceNotesOfFile(noteEndFilePath);

if(!stringInObject(noteSourcePath,currentSourceNotes)){
//if current file is not already a source then add and open otherwise just open
let currentFileText = await readFromTFile(existingTFile);
let currentFileTextEdited = currentFileText.replace("SourceNotes::","SourceNotes::"+noteSourceFile+",");

await this.app.vault.modify(existingTFile,currentFileTextEdited);
}

openFile(existingTFile);

}else{

//check if the end folder path exists and if not then create the folder
 createFileLocationIfBlank(noteEndFolderPath);
 
//create template at end folder path and go to the newly created file
await tp.file.create_new(""+templateText+"", noteEndFolderPath+"/"+noteTitle,true);

}

//Editor Instructions post-file creation
editor.blur(); //so cursor is not anywhere in file but is in title
}catch(err){
console.error(err);
await replaceHighlightedText(tp.file.selection());
}

%>