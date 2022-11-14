---
Title: VarStart_NoteTitleNoPrepend_VarEnd
Aliases: 
NoteRoot: TemplateUtilities
NoteType: TemplateData

hasFeatures: hasHeader, hasNoteType, hasAliases, hasNoteRoot, hasTemplateInformation, hasTemplateDescription, hasTemplateUses, hasTemplateTips, hasTemplateHeadingContains, hasHowTheTemplateWorks, hasTemplateInspiration, SystemPrompts, SystemPromptVariables
---
# Template Data: VarStart_NoteTitleNoPrepend_VarEnd
## Data
### Dataview
- TemplateLink:: VarStart_NoteSource_VarEnd
- EndFolderPath:: <%* let allFolders = await app.plugins.plugins.dataview.api.pages().groupBy((page) => {return this.app.metadataCache.getFirstLinkpathDest(page.file.path, "").parent.path}); let foldersToSelect = []; for (let value of allFolders){foldersToSelect.push(value.key);} console.log(foldersToSelect); let selectedFolder =  await tp.system.suggester(foldersToSelect,foldersToSelect,false,"Select folder text"); let folderToUse = await tp.system.prompt("Select folder",selectedFolder); tR+=folderToUse %>
- TitlePrepend:: <%* let titlePrependPrompt = "VarStart_NoteSourceTitle_VarEnd".replace(/template$/i,"").trim(); titlePrependPrompt = "("+titlePrependPrompt+")"; let titlePrepend = await tp.system.prompt("Title Prepend",titlePrependPrompt); tR += titlePrepend %>
- TemplateName:: <%* let templateNamePrompt = "VarStart_NoteSourceTitle_VarEnd".replace(/template$/i,"").trim(); let templateName = await tp.system.prompt("Template Name",templateNamePrompt); tR += templateName %>
- TemplateAction:: <%* let templateActionPrompt = "VarStart_NoteSourceTitle_VarEnd".replace(/template$/i,"").trim(); let templateAction = await tp.system.prompt("Action of using template",templateActionPrompt); tR += templateAction %>
- TemplateNoteType:: <%* let templateNoteTypePrompt = "VarStart_NoteSourceType_VarEnd".replace(/template$/i,"").trim(); let templateNoteType = "VarStart_NoteSourceType_VarEnd"; if(templateNoteType.length == 0){ templateNoteType = await tp.system.prompt("Note Type",templateNoteTypePrompt);} tR += templateNoteType; %>
- TemplateTextPrompts:: []
- TemplateTextPromptsBeforeTitle:: 0
- TemplateRequestHighlightAsTitle:: 0
- TemplateRequiresHighlight:: 1
- TemplateIncludeDateInTitle:: 0

# Description

## What is it useful for

## Tips on how to use the template 

## What each heading in the template should contain

## Template inspiration

## How the template works

## Template authors and acknowledgements

