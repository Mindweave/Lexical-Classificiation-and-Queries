---
Aliases: 
NoteType: ConceptClassification

hasFeatures: hasTextCriteriaBulletPoints
---
(SourceNotes:: [[00 Home Note]] )
SourceHighlight:: (No Highlight Selected)
(Template:: [[Utilities/Template Utilities/Templates Data/Classification Concept Preset Template Data.md|Classification Concept Preset Template Data]] )

# Classification: Academic Paper

## Description
- Research papers and academic essays. 
- Often reliable sources of peer-reviewed information. 

### Files with descriptions of this classification

## Indicators
### Text Indicators
#### Conditions to Attempt Classification (or leave blank)

#### Stop Classification

#### Automatically Classify
- title in text
	- Academic Paper
- title in file path
	- Academic Paper {{(File Path as Text)}} {{(Includes Text)}}
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
- Something unique to Academic Paper
- These exclude a regular piece of paper or a metaphorical use of paper. The only other paper I typically deal with is an academic paper
	- /(?<!\bjust )\bpaper(s)?\b/  {{(File Text as Text)}} {{(Includes Pure Regex)}} 
	- /(?<!\bpiece of )\bpaper(s)?\b/  {{(File Text as Text)}} {{(Includes Pure Regex)}}
	- /(?<!\bon )\bpaper(s)?\b/  {{(File Text as Text)}} {{(Includes Pure Regex)}}
	- /(?<!\bwith )\bpaper\b/  {{(File Text as Text)}} {{(Includes Pure Regex)}} 
- Common source for an academic paper
	- Google Scholar 
- Common source for an academic paper
	- JSTOR
- multiple authors in a paper
	- et al 
- Bracket followed by a last name followed by a year often indicates an academic paper
	- /\((\w|-)* (1|2)\d\d\d[^\d]/  {{(File Text as Text)}} {{(Includes Pure Regex)}} 

## Could be confused with these classifications