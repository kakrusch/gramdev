# gramdev

## Overview

a short description of the phenomenon 

- main clause wh-questions in English, implement do-support, inflection differences and selectional properties based on the wh-word and which part of the C-structure it belongs to.

## Motivation - why I chose this phenomenon 

What it is, why I chose it, why it is challenging, etc.

- 

## Implementation approach/design

your implementation approach/design and the reasons for it (including usage of OT marks, restriction operators, testsuire design, etc), and the challenges that remain. You can have subheadings in this document.

#extra errors: tense mismatches, trans-mismatches, POSS-WH mismatches (both ways), subj-obj-do-nodo, no ?, non-wh-structure

   - new rule for WHQ:
      -   rule consists of a VP and then allowed stuff before 
      -   wh-words are banned in declaratives and vice verse

   - wh-words as pronouns (only gender and number in specific cases)

   - wh-word fronting via FOCUS and uncertainty paths

   - difference between subject and object wh-word origin:

      - SUBJ: main verb is inflected for case and number (usually 3rd person sg)

      - OBJ: do-support if there is no other aux and main verb is in infinitival form
      - 
   - extraction from adjuncts
     
   - extraction from PPs (with stranded P)

   - embedded wh-words


### Base:

- on the basis of the grammar from exercise 2, but including correct XCOMP and COMP rules and words. Also making all Verbs using morphological parser and remove nouns

- add Adjunct because-CP to base S rules, to have a hypothetical answer to "How" or "Why", add because to lexicon

Other Lexicon Changes:

- add a Question mark (?) that makes sentences interrogative type
- add inflected versions for do-support that agree with the sentence's subject 

### WH-words in the lexicon:
- all wh-words are treated like pronouns, but have `(^ PRON-TYPE) = int`
- all wh-words have capital and lowercased versions
- I added an additional feature (^REL) that can be either "+" or "-", to specify wh-words that can also be relative pronouns. This difference is relevant because those pronouns that can be relative must have an NP-position (such as the subject or Object).

Pronouns that can act as relative pronouns -> can come from subject or object (must be NP) 
- Who: personal pronoun -> 3rd sg agreement in SUBJ
- Whose and Which: possessive -> agreement with noun in SUBJ
- What: possessive and personal -> agreement like the counterparts

pronouns that do not act as relative pronouns -> not from subject or OBJ, agreement with subject 
- Where and When: personal -> can be within PP
- How and Why: personal -> cannot come from PP/strand a P

optionally relative pronouns:
- Who: like regular personal pronouns, but is of the type that can be relative
- Which and Whose: like possessive pronouns, but without PERSON or NUMBER features, as they don't have these features inherently and don't need to agree, as their sister-noun will agree if needed.
- what: alternatively either like "who" or like "which/whose"

  non-relative pronouns:
  - where and when: like personal pronouns but also without agreement, as they are always from adjuncts and don't need to agree
  - how and why: like where/when, but have an Inside-Out Functional uncertainty to designate them as not being the object of a preposition, and thus not allowing stranded Ps "Why did Mary appear from __?"



### Future work
 - fronted PPs: "from where did Mary appear?"
 - thematic role implementation: wh-words are constrained by the thematic roles they are able to ask about




## Files

List the files you have uploaded and their functions.
