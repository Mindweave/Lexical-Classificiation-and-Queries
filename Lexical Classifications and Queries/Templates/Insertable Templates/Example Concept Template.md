---
Aliases: 
NoteType: ConceptNote
hasFeatures: hasFlexibleWikipediaPage
---

# Concept: <% tp.file.title %>

## Description
<%* tp.file.cursor(1) %>

## Components
- 

## Questions
-  

## Applications
- 

----
## Web Info
<%*
function removeSpecialCharactersFromTitle(noteTitle){
//some characters cannot be used in a title this removes them
noteTitle = noteTitle.replace(/[*"\/\\<>:|?\r\n\t\f\v\[\]]/gm," ");
noteTitle = noteTitle.replace(/\s+/gm," "); //only one space
return noteTitle;
}

let wikipediaBeforeTitleFormatted = `<div style=" position: left;">`+String.fromCharCode(10)+`<iframe width="100%" height = "100%" allowfullscreen src ="https://en.wikipedia.org/w/index.php?search=`;
let wikipediaAfterTitleFormatted = `&ns0=1"  style=" height: 500px; resize: both; border:7px solid #5c73f2;  border-radius: 8px;"></iframe>`+String.fromCharCode(10)+`</div>`;
let rawTitle = tp.file.title;
let titleFormatted = removeSpecialCharactersFromTitle(rawTitle);
titleFormatted = encodeURIComponent(titleFormatted);
let userSelection = await tp.system.suggester(["Add default Wikipedia","Customize Wikipedia search text","Do not add Wikipedia"],["Add default Wikipedia","Customize Wikipedia search text","Do not add Wikipedia"],false,"Add Wikipedia iFrame to file");
if(userSelection == "Add default Wikipedia"){
tR+=wikipediaBeforeTitleFormatted+titleFormatted+wikipediaAfterTitleFormatted;
}else if(userSelection == "Customize Wikipedia search text"){
let userFormatted = await tp.system.prompt("Wikipedia Search Text",removeSpecialCharactersFromTitle(rawTitle));
tR+=wikipediaBeforeTitleFormatted+userFormatted+wikipediaAfterTitleFormatted;
}else{
tR+="User opted not to Include Wikipedia"
}

%>