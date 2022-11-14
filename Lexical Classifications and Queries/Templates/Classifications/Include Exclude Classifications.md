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

async function openFile(selectedTFile){
await this.app.workspace.getLeaf().openFile(selectedTFile)
}

async function readFromTFile(selectedTFile){

return await this.app.vault.read(selectedTFile)    
}

function enterCharacter(){
    return String.fromCharCode(10);
}

function reverseSlash(){
return String.fromCharCode(92);
}

function dollarSymbol(){
return String.fromCharCode(36);
}

function quotationSymbol(){
return String.fromCharCode(34);
}

function apostrapheSymbol(){
return String.fromCharCode(39);
}

function specialOptionBrackets(textToTurnIntoSpecialOption){
return "{{("+textToTurnIntoSpecialOption+")}}"
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

function delayTextReturn(textToReturn,delayInMilliseconds) {
//Page text needs time to update
//So rather have the button use a delay instead of the user delaying themselves
return new Promise((resolve, reject) => {
     setTimeout(() => {resolve(textToReturn);
     },delayInMilliseconds);
   });
}


async function requestIndicator(){

let specialOptions = "";
let fileText = "";
let readTextOrRegex = "";
let includeOrExclude = "";
let indicatorOptions = "";
let indicatorToAdd = "";
addOptionsText = "";
let textInPrompt = userHighlightUnformatted;

specialOptions = await tp.system.suggester(["No special options","Add special options"], ["No special options","Add special options"],false,"Use special options?") 

if(specialOptions == "No special options"){
indicatorToAdd = await tp.system.prompt("Enter text to use in "+selectedClassificationTitle+" classification",textInPrompt);
}else{

addOptionsText = "";

fileText = await tp.system.suggester(["Document Text","Document File Path"], ["{{(File Text as Text)}}","{{(File Path as Text)}}"],false,"Text to classify")

if(fileText == "{{(File Path as Text)}}"){
textInPrompt = sourceTFile.path + " | "+ userHighlightUnformatted;
}

addOptionsText+= " "+fileText

readTextOrRegex = await tp.system.suggester(["Basic Text", "Pure Regex"], ["Text","Pure Regex"],false,"Classify using indicator type");

if(readTextOrRegex == "Text"){
indicatorToAdd = await tp.system.prompt("Enter text to use in "+selectedClassificationTitle+" classification",textInPrompt);
}else if(readTextOrRegex == "Pure Regex"){

let manualOrPreset = await tp.system.suggester(["Manual","Preset"], ["Manual","Preset"],false,"Enter Regular Expression Manually or with Preset")
if(manualOrPreset == "Manual"){ //if manual or preset
indicatorToAdd = await tp.system.prompt("Enter regex to use in "+selectedClassificationTitle+" classification",textInPrompt);
}else{
let preset = await tp.system.suggester(["Word or Phrase starting with","Word or Phrase ending with","Word or Phrase containing","Word or Phrase later followed by other Word or Phrase in a sentence","Word or Phrase but NOT directly followed by other Word or Phrase","Word or Phrase but NOT directly preceded by other Word or Phrase"], ["Word starting with","Word ending with","Word containing","Sentence sequence","Negative followup","Negative Preceding"],false,"Select Regular Expression preset");

if(preset == "Word starting with"){
let startingText =  await tp.system.prompt("Enter text to use at the start of the word",textInPrompt);
let caseInsensitive = await tp.system.suggester(["No","Yes"], [true,false],false,"Is this case-sensitive?");
indicatorToAdd = "/"+reverseSlash()+"b"+"("+startingText+")"+"/";
indicatorToAdd += caseInsensitive ? "i" : "";

}else if(preset == "Word ending with"){
let startingText =  await tp.system.prompt("Enter text to use at the end of the word",textInPrompt);
let caseInsensitive = await tp.system.suggester(["No","Yes"], [true,false],false,"Is this case-sensitive?");
indicatorToAdd = "/("+startingText+")"+reverseSlash()+"b"+"/";
indicatorToAdd += caseInsensitive ? "i" : "";
}else if(preset == "Word containing"){
let startingText =  await tp.system.prompt("Enter text in the word",textInPrompt);
let caseInsensitive = await tp.system.suggester(["No","Yes"], [true,false],false,"Is this case-sensitive?");
indicatorToAdd = "/"+ reverseSlash()+"b["+ reverseSlash()+"w"+ reverseSlash()+"-"+ reverseSlash()+"d]*("+startingText+")["+ reverseSlash()+"w"+ reverseSlash()+"-"+ reverseSlash()+"d]*"+ reverseSlash()+"b"+"/";
indicatorToAdd += caseInsensitive ? "i" : "";
}else if(preset == "Sentence sequence"){
let startingText =  await tp.system.prompt("Enter text to use in the sentence",textInPrompt);
let laterText = await tp.system.prompt("Enter text later in the sentence",textInPrompt);
let caseInsensitive = await tp.system.suggester(["No","Yes"], [true,false],false,"Is this case-sensitive?");
indicatorToAdd = "/"+reverseSlash()+"b"+startingText+"(?=[^"+ reverseSlash()+"w])(([^"+ reverseSlash()+"n"+ reverseSlash()+"t"+ reverseSlash()+"r"+ reverseSlash()+"f"+ reverseSlash()+"."+ reverseSlash()+"?"+ reverseSlash()+"!])|["+ reverseSlash()+"."+ reverseSlash()+"?"+ reverseSlash()+"!](?! )|(?:"+ reverseSlash()+"b([Dd]r|[Mm]r|[Mm]s|[Mm]rs|[Ii]"+ reverseSlash()+".[Ee]|[Ee]"+ reverseSlash()+".[Gg]|[Ee][Tt] [Aa][Ll])"+ reverseSlash()+".))+(?<=[^"+ reverseSlash()+"w])"+laterText+ reverseSlash()+"b"+"/"

}else if(preset == "Negative followup"){
let startingText =  await tp.system.prompt("Enter text to use in the sentence",textInPrompt);
let laterText = await tp.system.prompt("Enter text not found directly after",textInPrompt);
let caseInsensitive = await tp.system.suggester(["No","Yes"], [true,false],false,"Is this case-sensitive?");
indicatorToAdd = "/"+reverseSlash()+"b"+startingText+reverseSlash()+"b"+"((?!"+reverseSlash()+"b"+laterText+ reverseSlash()+"b"+"))"+"/"

}else if(preset == "Negative Preceding"){
let startingText =  await tp.system.prompt("Enter text to use in the sentence",textInPrompt);
let earlierText = await tp.system.prompt("Enter text not found directly before",textInPrompt);
let caseInsensitive = await tp.system.suggester(["No","Yes"], [true,false],false,"Is this case-sensitive?");
indicatorToAdd = "/"+"(?<!"+reverseSlash()+"b"+earlierText+reverseSlash()+"b"+")"+reverseSlash()+"b"+startingText+ reverseSlash()+"b"+"/"

}

}
}

includeOrExclude = await tp.system.suggester(["Found in text","Not Found in text"],["Includes","Excludes"],false,"Classify when indicator is")

addOptionsText += " "+specialOptionBrackets(includeOrExclude+" "+readTextOrRegex)

if(readTextOrRegex == "Text"){
//will need to turn this into a loop to account for other text indicator options like trailing whitespace
indicatorOptions = await tp.system.suggester(["None","Case-Sensitive"],["None","{{(Case-Sensitive)}}"],false,"Modify indicator");
}
if(indicatorOptions != "None"){
addOptionsText+= " "+indicatorOptions
}

}
return indicatorToAdd+" "+addOptionsText;
}

let editor = await getEditor()
let userHighlightUnformatted = editor.getSelection();
let userHighlightedText = (userHighlightUnformatted.length > 0);
let allClassificationsInFolderToDebug;
let allClassificationsDebugTitles;
let allClassificationsDebugPaths;
let selectedClassification;
let selectedClassificationPath;
let selectedClassificationTitle;
let createdNewClassification = false;
let sourceTFile = this.app.workspace.getActiveFile();
let endTFile;

let existingTFile;
let currentFileText;
let currentFileTextEdited;

//find out which classification to add to

allClassificationsInFolderToDebug = await app.plugins.plugins.dataview.api.pages('"Classifications/Text Classifications"').where(p => typeof(p.NoteType) != "undefined").values /*separated from all classifications in folder in case some need to be excluded from debug*/

allClassificationsDebugTitles = allClassificationsInFolderToDebug.map(a => {return a.file.name+(a.file.aliases.values.length>0 ? (" "+"("+a.file.aliases.values+")"):"");});

allClassificationsDebugPaths = allClassificationsInFolderToDebug.map(a => a.file.path);

selectedClassification = await tp.system.suggester(["(Create New File)"].concat(allClassificationsDebugTitles),["User Requested to Create New Files for the Selected Classification"].concat(allClassificationsInFolderToDebug),false, "Choose classification to add to")

if(selectedClassification == "User Requested to Create New Files for the Selected Classification"){
createdNewClassification = true;
let useHighlightForTitle =  await tp.system.suggester(["Use Highlight for New Classification Title","Use Custom Text for New Classification Title"],[true,false],false, "New Classification Title?");
if(!useHighlightForTitle){
//stop highlight
editor.setCursor(0,0);
editor.blur();
}

await activateCommand("obsidian-hotkeys-for-templates:templater:From Highlight/Classification Template.md");
//wait for new file to be recognized
(await delayTextReturn("Moving paths",1000))

//remove trailing space returned by TFile between title and path
selectedClassificationPath = this.app.workspace.getActiveFile().path.replace(/\/\s/,"/");

selectedClassificationTitle = this.app.workspace.getActiveFile().basename

}else{
selectedClassificationPath = selectedClassification.file.path;

selectedClassificationTitle = selectedClassification.file.name;
}


let headingSelected = await tp.system.suggester(["(Do not add text indicator)","Conditions to Attempt Classification (or leave blank)","Stop Classification","Automatically Classify"],
["(Do not add text indicator)","#### Stop Classification","#### Automatically Classify","## Could be confused with these classifications"],false,"Heading for this text indicator") /*return heading after to add to bottom */

if(headingSelected != "(Do not add text indicator)"){
//do add a text indicator and update the classification page
let indicators = [];
let addAdditionalIndicator = false;
do{
indicators.push(await requestIndicator());
addAdditionalIndicator = await tp.system.suggester(["Done. Add explanation","Add additional indicator"],
[false, true],false,"Would you like to add an additional indicator?")
}while(addAdditionalIndicator);

if(indicators.length>0){ //needs to have valid text indicator
let explanationOfIndicator = await tp.system.prompt("Briefly explain why the text indicator is used to classify "+selectedClassificationTitle);

let indicatorHeading = "- "+explanationOfIndicator;
let indicatorSubBullet = String.fromCharCode(9)+"- "+indicators.join(String.fromCharCode(10)+String.fromCharCode(9)+"- "); //String.fromCharCode(9)+"- "+indicatorToAdd+" "+addOptionsText;
 existingTFile = getTFile(selectedClassificationPath);
 currentFileText = await readFromTFile(existingTFile);
 currentFileTextEdited = currentFileText;

if(currentFileText.indexOf(enterCharacter()+headingSelected) != -1){
currentFileTextEdited = currentFileText.replace(enterCharacter()+headingSelected,indicatorHeading+enterCharacter()+indicatorSubBullet+enterCharacter()+enterCharacter()+headingSelected);
}else if(currentFileText.indexOf(headingSelected) != -1){
currentFileTextEdited = currentFileText.replace(headingSelected,enterCharacter()+indicatorHeading+enterCharacter()+indicatorSubBullet+enterCharacter()+enterCharacter()+headingSelected);
}else{
    let notice = new tp.obsidian.Notice("", 8000);
    notice.noticeEl.innerHTML = `<b>Error</b><br/>${selectedClassificationTitle} missing ${headingSelected}<br/>`;
}

await this.app.vault.modify(existingTFile,currentFileTextEdited);
}
}
let navigationAfterEdit;
if(createdNewClassification){
navigationAfterEdit = await tp.system.suggester(["Stay in current file","Go back to source file"],[false,true],false,"Navigation")
}else{
navigationAfterEdit = await tp.system.suggester(["Stay in current file","Go to edited classification"],[false,true],false,"Navigation")
}
if(navigationAfterEdit){
if(createdNewClassification){
await openFile(sourceTFile);
}else{
await openFile(existingTFile);
}
}





endTFile = this.app.workspace.getActiveFile();

//replace highlight if still in same file, did not leave file, and still has a highlight
if(userHighlightedText && tp.file.selection().length > 0){
if(sourceTFile.path == endTFile.path && selectedClassification != "User Requested to Create New Files for the Selected Classification"){
//still in same file so highlighted text needs to be replaced
await replaceHighlightedText(userHighlightUnformatted);
}
}

//TODO check it has not already been added using get indicators from classify text code
}catch(err){
console.error(err);
await replaceHighlightedText(tp.file.selection());
}

%>