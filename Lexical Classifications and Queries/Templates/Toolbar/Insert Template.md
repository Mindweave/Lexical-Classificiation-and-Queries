<%*
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

this.app.plugins.plugins['templater-obsidian'].settings.templates_folder = "Templates/Insertable Templates"

try {  
  //tryCode - Code block to run
  await activateCommand("templater-obsidian:insert-templater")  
}  
catch(err) {  
  //catchCode - Code block to handle errors
  let notice = new tp.obsidian.Notice("", 8000).noticeEl.innerHTML = `<b>Error</b><br/>Template did not successfully load<br/>`;
}  
finally {  
 // finallyCode - Code block to be executed regardless of the try result  
this.app.plugins.plugins['templater-obsidian'].settings.templates_folder = "Templates"
}

%>