---
Aliases: 
NoteType: ConceptClassification

hasFeatures: hasTextCriteriaBulletPoints
---
(SourceNotes:: VarStart_NoteSource_VarEnd )
SourceHighlight:: VarStart_NoteHighlight_VarEnd
(Template:: VarStart_NoteTemplate_VarEnd )

# Classification: VarStart_NoteTitleNoPrepend_VarEnd

## Description
- 

### Files with descriptions of this classification

## Indicators
### Text Indicators
#### Conditions to Attempt Classification (or leave blank)

#### Stop Classification

#### Automatically Classify
<%* let addTitle = await tp.system.suggester(["Yes, add indicator title in text","No do not add indicator title in text"],["Yes, add indicator","No do not add indicator"],false,"Add title included in text as indicator to automatically classify?");
if(addTitle == "Yes, add indicator"){
tR+="- title in text"+String.fromCharCode(10)+String.fromCharCode(9)+"- VarStart_NoteTitleNoPrepend_VarEnd"
}

let addTitleToFilePath = await tp.system.suggester(["Yes, add indicator title in file path","No do not add indicator title in file path"],["Yes, add indicator","No do not add indicator"],false,"Add title included in file path as indicator to automatically classify?");

if(addTitleToFilePath == "Yes, add indicator"){
if(addTitle == "Yes, add indicator"){
//need space between indicators
tR+=String.fromCharCode(10);
}
tR+="- title in file path"+String.fromCharCode(10)+String.fromCharCode(9)+"- VarStart_NoteTitleNoPrepend_VarEnd"+" "+"{{(File Path as Text)}} {{(Includes Text)}}"
}
%>
- Plural
- Regex Identification
- Group
- Singular
- Possessive
- Present Tense
- Past Tense
- Future Tense
- Formal usage
- Slang usage
- Direct reference
- Direct usage
- Phrase
- Synonym
- Broad Regex Identification
- Common misspelling 
- Indirect Reference
- Indirect usage
- Common example
- Something unique to VarStart_NoteTitleNoPrepend_VarEnd

## Could be confused with these classifications