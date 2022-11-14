<%* 
function getTFile(filePath){
return this.app.metadataCache.getFirstLinkpathDest(filePath, "");
}

function openFile(selectedTFile){
this.app.workspace.getLeaf().openFile(selectedTFile)
}

async function readFromTFile(selectedTFile){
return await this.app.vault.read(selectedTFile)
}

function getDataview(){
return app.plugins.plugins['dataview'].api
}

function delayTextReturn(textToReturn,delayInMilliseconds) {
//Page text needs time to update
//So rather have the button use a delay instead of the user delaying themselves
return new Promise((resolve, reject) => {
     setTimeout(() => {resolve(textToReturn);
     },delayInMilliseconds);
   });
}

let newFileText = "";
let timeNow = tp.date.now("YYYY-MM-DD_HH_mm_ss")
let newFileTitle = "Untitled "+timeNow;
let newFilePath = newFileTitle+".md";

await app.vault.create(newFilePath,"");
openFile(getTFile(newFilePath))

const dv = getDataview();
let folderlessFiles;  
let attemptsToRunFolderlessFiles = 0;
//sometimes the folderlessFiles returns zero since dataview plugin has not loaded yet (any way to know if it has then loop until property changes appropriately or wait for event?)
 do{
 //should always at least show the file just created. If not then the files have not been indexed yet
 await delayTextReturn("wait a second before trying and then repeat trying after another second",1000)
 folderlessFiles = await dv.pages().where(p=> (p.file.folder) == "").values;
 attemptsToRunFolderlessFiles+=1; /*run for 5 min then accept it isn't happening. User error or something. Should not happen since it automatically just created a file which does not have a folder but if the user deletes it faster than dataview loads then it could be an issue*/
 } while (folderlessFiles.length == 0 || attemptsToRunFolderlessFiles > 300)

for (let page of folderlessFiles){

if(page.file.path == newFilePath){
continue;
}

//other page data
let pageTFile = getTFile(page.file.path)
let pageText = await readFromTFile(pageTFile);


if( pageTFile.stat.size != 0){ //not organizing on startup anymore just delete blank files
continue; //below is skipped

//get title
let pathToAdd = "Scraps/Quick Notes/"+(page.file.name)+"."+(page.file.ext)

while(await tp.file.exists(pathToAdd)){
pathToAdd+=" "+"("+timeNow+")"
}

//edit note with meta data and any cleanup
let pageTextEdit = ""
//meta data
/* user may decide their own meta data
if(Object.keys(page.file.frontmatter).length == 0){
pageTextEdit+="---\nAliases:\nNoteType: QuickNote\n\nhasFeatures: hasAutoMove\n---"+"\n"
}
*/

if(pageText.search(/\n#+ /) == -1){
pageTextEdit+="# "+page.file.name+"\n";
}

pageTextEdit += pageText;

//update file with edits
app.vault.modify(pageTFile,pageTextEdit);

//create note
await app.fileManager.renameFile( pageTFile, pathToAdd );
}else{
app.vault.trash(pageTFile);
}

}

%>