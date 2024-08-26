# Grammar Development Final project
*_Kascha Kruschwitz_* 

Github link:[ https://github.com/kakrusch/gramdev.git](https://github.com/kakrusch/gramdev.git)

## Overview and Motivation


- main clause wh-questions in English, implement do-support, inflection differences and selectional properties based on the wh-word and which part of the structure it asks about
- implement 8 wh-words: Who, Whose, Which, What, Where, When, How and Why
- if asking about subject: inflected verb based on q-NP, if from any other position: do-support and infinitival verb
- all the wh-words have their own properties
- based on LFG implementations from *A Grammar Writer's Cookbook* and the *Handbook of Lexical Functional Grammar*


- Motivation: seems like a straight-forward phenomenon, but there is a lot of selectional properties to look out for and a lot more that I could not include in this project
- wanted to use this as an opportunity to explore how to use different restrictors in XLE
- wanted to understand LFG more, especially how it deals with phenomena that in Minimalism are due to movement, such as wh-movement and preposition-stranding

## Implementation approach/design

In order to implement English wh-questions, I used my grammar from exercise two and extended it, including new phrase-structure rules and lexical entries. I included rules for XCOMP and COMP arguments and added the corresponding verbs to the lexicon. Further, verbal inflections are handled via the tokenizer and the Lexical Lookup Model is implemented, such that nouns are not in the lexicon and are instead recognized automatically. 

I implemented several basic but task-specific changes to the lexicon. Initially, I added the restriction `(! (POSS) PRON-TYPE) ~= int` to the base sentence's `(S)` subject NP, to disallow interrogative pronouns in the subject position of a regular sentence. To further ensure that the wh-questions are not parsed as regular sentences, a question mark was added to the lexicon, which specifies that a sentence has an interrogative sentence type. Another addition to the lexicon is different inflected versions of the auxiliary *do*, to allow do-support in wh-questions if the wh-word originates in a non-subject position. The last basic change is the addition of an adjunct CP with the complementizer *because* in the `VP` rule, to have an answer structure in place for responses to the wh-words *how* and *why*. 


### WH-words in the lexicon:
The first half of the implementation included adding wh-words to the lexicon, each with different specific properties. All wh-words are treated as pronouns, with `(^ PRED) = 'PRO'`. They pattern either like possessive or personal pronouns, but each have the property `(^ PRON-TYPE) = int` to mark them as interrogative. All wh-words also have both a capital and a lowercase version. I mainly used existing features from the XLE Documentation but implemented a `(^REL)= +/-` feature to specify those wh-words that can also be relative pronouns, such as in "Mary, **who** is a cat, appeared". This project does not implement relative clauses but the distinction is relevant, as those wh-words that are optionally relative can be used to ask about NPs in `SUBJ` or `OBJ` positions, whereas the other wh-words cannot. For example, "Who appeared?" is `+REL` and grammatical, but "Where appeared?" is `-REL` and ungrammatical.

The wh-words that can act as relative pronouns and are specified as `(^REL)= +` are: *who, whose, which*, and *what*. As mentioned, these can ask about the `SUBJ` and `OBJ` of the sentence, but cannot come from adjuncts that do not have a preposition. *Who* uses the regular pronoun template but is specified as 3rd person sg, as this is how it agrees when in subject position. *Which* and *Whose* use the possessive pronoun structure, but are not specified for person or number, as they can combine with any N and, if present in subject position, agreement happens with the N's features, not the wh-word's. The final optionally relative wh-word is *what*, which can be either personal-like or possessive-like, and is thus specified as either like *who* or like *which/whose*.


Wh-words that cannot act as relative pronouns, at least not a referential one, and are specified as `(^REL)= -` include *where*, *when*, *why*, and *how*. All of these act like personal, not possessive, pronouns. Crucially, these cannot be the `SUBJ` or `OBJ` of the main verb, but can replace unspecified and non-PP adjuncts. This pattern is specified in the rules, but as a consequence they have no Person or Number features, as they are never in subject position and thus don't trigger agreement. All four wh-words thus have almost identical specifications except for one key difference. *Where* and *When* can replace CPs or PPs and can also be the object of a semantic PP, such as in "from where". *How* and *Why* can also replace CPs or PPs, but crucially cannot be the object of a semantic PP. Thus, they are specified with the additional Inside-Out Functional Uncertainty `~((ADJUNCT OBJ ^ ) PTYPE)`, that specifies exactly the aforementioned restriction, disallowing structures like "from how".


### WHQ Rule - another ROOT option
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
- there has to be one wh-word, but there cannot be more than one: if there are more they remain in-situ
    - additional wh-words may remain in-situ, so long as a single word is fronted and all argument positions are filled
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

### testsuite?

  
### Future work
 - fronted PPs: "from where did Mary appear?"
 - thematic role implementation: wh-words are constrained by the thematic roles they are able to ask about, I made rough destinctions, but using thematic roles would allow for much more fine grained distinctions




## Files

wh_english.lfg  -> full grammar

wh_testsuite.lfg  -> testsuite for the grammar

xlerc         -> load grammar automatically and keyboard shortcuts (*g* to reload grammar, *t* to parse the testsuite)

common.templates.lfg  -> use basic English templates

basic-parse-tok.fst  -> basic english tokenizer

english.infl.patch.full.fst -> basic english analyser (detokenizer)



## Sources:
Dalrymple, M. (2024). *Handbook of Lexical Functional Grammar*. Language Science Press.

Butt, M., King, T., Niño, M. and Segond F. (1999). *A Grammar Writer’s Cookbook*. CSLI Publications.

Crouch, D., Dalrymple, M., Kaplan, R., King, T., Maxwell, J. and Newman, P., (1993). *XLE Documentation*. Palo Alto Research Center (PARC).
