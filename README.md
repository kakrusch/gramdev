# Grammar Development Final project

## Overview

a short description of the phenomenon 

- main clause wh-questions in English, implement do-support, inflection differences and selectional properties based on the wh-word and which part of the C-structure it belongs to
- implement 8 wh-words: Who, Whose, Which, What, Where, When, How and Why
- 


## Motivation - why I chose this phenomenon 

What it is, why I chose it, why it is challenging, etc.

- seems like a straight-forward phenomenon, but there is a lot of selectional properties to look out for and a lot more that I could not include in this project

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

- in S's NP: (! (POSS) PRON-TYPE) ~= int
     - disallow interrogative type pronouns in SUBJ of normal sentence

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

### WHQ Rule 
I introduced a rule called WHQ (WH-question), to generate wh-questions (as opposed to polar questions).

     - 
- Base outline of the rule:
     - some kind of wh-word: either a wh-word and do-support or a wh-word in subject position without Auxiliary
     - an optional NP-subject, if the WH-word is not the subject
     - a VP that follows the rules of a regular VP
     - some kind of optional Preposition, to allow for P-stranding
     - a question mark to mark the sentence as an interrogative
 
WH-word rule
- wh-word/phrase has (^FOCUS) feature to mark it as being moved from its base position & also is constrained to be an interrogative pronoun
- option 1: NP is (^SUBJ)
   -  verb MUST be inflected: (^ VFORM) ~= inf
   -  only the optionally-relative wh-words can be in this position (eg. whose cat/who appeared)
-  Option 2: NP is from any other position and must thus have do-support (as I currently don't have any other auxiliaries)
   -  thus this option has an auxiliary that states the verb must be infinitival form: (^ VFORM) =c inf
   -  There are then 4 options for the wh-word specified by functional uncertainty
         1. (^ {XCOMP|COMP}* {XCOMP|COMP|OBJ2|OBL-TO|OBL}) = !): wh-word replaces any of the specified categories fully, and this origin might be embedded in higher complement clauses
         2. (^ {XCOMP|COMP}* {OBJ}) = ! (!REL)=c+ : wh-word can be an object, but only if it is an optionally-relative type 
         3. !$(^ADJUNCT) @(OT-MARK Q-NREL) (!REL)=c- : non-relative pronouns can fully replace any adjunct (CP or PP)
               - the OT mark marks it as the less preferred option if a position where it replaces an argument is available
         4. (^ ADJUNCT: (<-PTYPE)=sem; OBJ)= ! @(OT-MARK Q-Ad): for most wh-words/phrases (except how and why) the wh-word can be in the object position of an adjunct PP, as specified by the off-path constraint that forces it to have a semantic preposition (which are specified at the end)
               - the OT mark marks it as preferred over the other adjunct type, but not more than when it replaces an argument
   

- PP:
      - 1. If the wh-word is an OBL-TO, the "to" can be stranded
      - 2. if the wh-word is an Oblique, the P can be stranded
      - 3. If the wh-word is an adjunct, it can strand its P and thus specificy a pred type (except for how and why because of their lexicon)


  
### Future work
 - fronted PPs: "from where did Mary appear?"
 - thematic role implementation: wh-words are constrained by the thematic roles they are able to ask about




## Files

wh_english.lfg  -> full grammar

wh_testsuite.lfg  -> testsuite for the grammar

xlerc         -> load grammar automatically and keyboard shortcuts

common.templates.lfg  -> use basic English templates

basic-parse-tok.fst  -> basic english tokenizer

english.infl.patch.full.fst -> basic english analyser (detokenizer)







