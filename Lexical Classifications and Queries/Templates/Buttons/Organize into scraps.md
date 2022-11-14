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

const dv = getDataview();
let folderlessFiles;  
//sometimes the folderlessFiles returns zero since dataview plugin has not loaded yet (any way to know if it has then loop until property changes appropriately or wait for event?)
 do{
 //should always at least show the file just created. If not then the files have not been indexed yet
 await delayTextReturn("wait a second before trying and then repeat trying after another second",1000)
 folderlessFiles = await dv.pages().where(p=> (p.file.folder) == "").values;
 } while (folderlessFiles.length == 0)

for (let page of folderlessFiles){

//other page data
let pageTFile = getTFile(page.file.path)
let pageText = await readFromTFile(pageTFile);


if( pageText.replace(/\s/g,"").length != 0){ //if has some text then move otherwise delete

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
/*
Titles automatically added in 1.0
if(pageText.search(/\n#+ /) == -1){
pageTextEdit+="# "+page.file.name+"\n";
}
*/

pageTextEdit += pageText;

//update file with edits
app.vault.modify(pageTFile,pageTextEdit);

//create note
await app.fileManager.renameFile( pageTFile, pathToAdd );
}else{
//just organizes should not delete files
//app.vault.trash(pageTFile);
}

}

%>