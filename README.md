# Grammar Development Final project
*_Kascha Kruschwitz_* 

Github link:[ https://github.com/kakrusch/gramdev.git](https://github.com/kakrusch/gramdev.git)

## Overview and Motivation

This project focuses on English main-clause wh-questions, specifically focusing on the selectional properties and inflectional differences of different wh-words and the part of the sentence they are asking about.


- implement 8 wh-words: Who, Whose, Which, What, Where, When, How and Why
- if asking about subject: inflected verb based on q-NP, if from any other position: do-support and infinitival verb
- all the wh-words have their own properties (eg. some allow preposition stranding, some can be the subject etc..)
- based on LFG implementations from *A Grammar Writer's Cookbook* and the *Handbook of Lexical Functional Grammar*


The phenomenon seemed relatively straightforward, but there were a lot of selectional properties to look out for and a lot more that I could not include in this project. I wanted to focus on this topic because I was aware of how wh-questions and their properties are explained using movement in Minimalism. With this project, I wanted to explore that same phenomenon but from the perspective of LFG, especially phenomena like wh-stranding and displaced arguments. Further, I saw this as an oportunity to explore different restrictors in XLE, to explain the selectional properties of each wh-word.


## Implementation approach/design

In order to implement English wh-questions, I used my grammar from exercise two and extended it, including new phrase-structure rules and lexical entries. I included rules for XCOMP and COMP arguments and added the corresponding verbs to the lexicon. Further, verbal inflections are handled via the tokenizer and the Lexical Lookup Model is implemented, such that nouns are not in the lexicon and are instead recognized automatically. 

I implemented several basic but task-specific changes to the lexicon. Initially, I added the restriction `(! (POSS) PRON-TYPE) ~= int` to the base sentence's `(S)` subject NP, to disallow interrogative pronouns in the subject position of a regular sentence. To further ensure that the wh-questions are not parsed as regular sentences, a question mark was added to the lexicon, which specifies that a sentence has an interrogative sentence type. Another addition to the lexicon is different inflected versions of the auxiliary *do*, to allow do-support in wh-questions if the wh-word originates in a non-subject position. The last basic change is the addition of an adjunct CP with the complementizer *because* in the `VP` rule, to have an answer structure in place for responses to the wh-words *how* and *why*. 


### WH-words in the lexicon:
The first half of the implementation included adding wh-words to the lexicon, each with different specific properties. All wh-words are treated as pronouns, with `(^ PRED) = 'PRO'`. They pattern either like possessive or personal pronouns, but each have the property `(^ PRON-TYPE) = int` to mark them as interrogative. All wh-words also have both a capital and a lowercase version. I mainly used existing features from the XLE Documentation but implemented a `(^REL)= +/-` feature to specify those wh-words that can also be relative pronouns, such as in "Mary, **who** is a cat, appeared". This project does not implement relative clauses but the distinction is relevant, as those wh-words that are optionally relative can be used to ask about NPs in `SUBJ` or `OBJ` positions, whereas the other wh-words cannot. For example, "Who appeared?" is `+REL` and grammatical, but "Where appeared?" is `-REL` and ungrammatical.

The wh-words that can act as relative pronouns and are specified as `(^REL)= +` are: *who, whose, which*, and *what*. As mentioned, these can ask about the `SUBJ` and `OBJ` of the sentence, but cannot come from adjuncts that do not have a preposition. *Who* uses the regular pronoun template but is specified as 3rd person sg, as this is how it agrees when in subject position. *Which* and *Whose* use the possessive pronoun structure, but are not specified for person or number, as they can combine with any N and, if present in subject position, agreement happens with the N's features, not the wh-word's. The final optionally relative wh-word is *what*, which can be either personal-like or possessive-like, and is thus specified as either like *who* or like *which/whose*.


Wh-words that cannot act as relative pronouns, at least not a referential one, and are specified as `(^REL)= -` include *where*, *when*, *why*, and *how*. All of these act like personal, not possessive, pronouns. Crucially, these cannot be the `SUBJ` or `OBJ` of the main verb, but can replace unspecified and non-PP adjuncts. This pattern is specified in the rules, but as a consequence they have no Person or Number features, as they are never in subject position and thus don't trigger agreement. All four wh-words thus have almost identical specifications except for one key difference. *Where* and *When* can replace CPs or PPs and can also be the object of a semantic PP, such as in "from where". *How* and *Why* can also replace CPs or PPs, but crucially cannot be the object of a semantic PP. Thus, they are specified with the additional Inside-Out Functional Uncertainty `~((ADJUNCT OBJ ^ ) PTYPE)`, which specifies exactly the aforementioned restriction, disallowing structures like "from how".


### WHQ Rule 

In the c-structure rules, I introduced a rule called WHQ, which specifies the structure of wh-questions. This rule is not called  SInt, as an English grammar would need a different rule for polar interrogatives, which have a different structure. The basic structure of the rule is as follows:

- Some kind of wh-word or phrase, with or without an auxiliary
- An optional NP-subject, that is mandatory if the wh-phrase itseld is not the subject
- A mandatory VP
- Some kind of optional preposition, to allow for preposition stranding 
- A mandatory question mark, to mark the sentence as an interrogative sentence


#### The WH-phrase 

English wh-phrases consist of a single fronted wh-phrase. If there are multiple wh-phrases in a sentence, one of those is fronted, while the others remain in-situ. This is achieved in the current grammar by marking the fronted wh-phrase using (^FOCUS), which marks it as being moved from its base position into the interrogative sentence position. As only a single NP can be in this focused position, only one wh-phrase is fronted. This NP is also constrained to always contain an interrogative preposition. 

There are then two overarching options for the fronted wh-phrase. The wh-word can either be in subject position, such that the main verb is inflected based on the wh-phrase, or the wh-word can have any other grammatical function, which leads to do-support and an uninflected main verb. The first option is defined by marking the focused NP as `(^SUBJ)` and forcing the main verb to be inflected using `(^ VFORM) ~= inf`. Further, as mentioned previously, only optionally relative pronouns can be in this position, which is marked using `(!REL)=c+`. 

The second option contains an `NP` and an `AUX` for do-support. The auxiliary specifies that the verb is uninflected using `(^ VFORM) =c inf`. There are then four options for which wh-words have which grammatical functions. These are defined using functional uncertainties and constrained using OT marks. The first option is specified using `(^ {XCOMP|COMP}* {OBJ2|OBL-TO|OBL}) = !)`. Any wh-word can fulfill any of the specified functions and can even be embedded within multiple CPs. The second option is `(^ {XCOMP|COMP}* {OBJ}) = ! (!REL)=c+`, which allows only optionally relative wh-words to be in object position. The third option allows only non-relative type pronouns to replace any adjunct, using ` !$(^ADJUNCT) (!REL)=c- @(OT-MARK Q-NREL)`. The OT mark here makes sure that these parses are dispreferred over parses where the wh-phrase is an argument of the main verb. Non-relative type wh-words, but also the personal-pronoun-like version of *what* can be an argument of type CP, which is specified in `(^ {XCOMP|COMP}* {XCOMP|COMP}) = ! {(!REL)=c-|(! PRON-FORM) = what (! POSS PRED) ~= 'PRO'}`. Again, these can also originally be embedded in higher CPs. Finally, most wh-words, except *how* and *why*, can be in object position of an adjunct PP if there is an overt preposition, such as in "from where". This is specified by an off-path constraint that forces the presence of a semantic PP, and thus some preposition: `(^ ADJUNCT: (<-PTYPE)=sem; OBJ )= !`. This option also contains an OT mark that prefers this option over the other types of adjuncts but ensures that the satisfaction of an argument position is preferred over being an adjunct.


#### Preposition stranding
The presence of a PP with a preposition and a wh-phrase is so far only possible in-situ. However, the wh-phrase may also be fronted, but leave the preposition behind in its base position, such as in "Where did he appear from?". This is especially important for the PP adjuncts, which must be defined as semantic somehow. Thus, an optional preposition can be inserted after the VP. If available as an argument, the stranded preposition forces the `OBL` or `OBL-TO` function on the wh-phrase. If neither is available, the wh-word is treated as part of an adjunct PP. *why* and *how* are disallowed with preposition stranding, as they are specified in the lexicon as not being able to combine with a preposition.

  
### Future work

The presented projects only scratches the surface of the constraints on the realization of each wh-word. Future projects should take more features and possible constraints into account, such as specifying the thematic roles that wh-words can occupy and which main verbs can license which wh-word. Further, there are other structures, such as fronted PPs with wh-phrases ("from where did Mary appear?") and embedded wh-phrases ("I wonder where Mary appeared"), which could act as extensions to the current project.




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
