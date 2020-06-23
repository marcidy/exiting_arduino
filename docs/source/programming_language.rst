.. _programming_language: 

Programming Lanauge
===================
Let's discuss what a pogrammming lanauge is.  

We write source code, which is a bunch of text in a file.  Compilation translates this text into a instructions that the target processor can run.   Instead of looking at it as text in a file, let's think about it more generally:  It's a long list of symbols, including spaces, new lines, etc.  Everything in the file is just a long list of single-characters.  How does the compilier differentiate all those symbols?

Syntax
------
The way the text in the file is written is extremely important.  It follows rules.  It's written is a specific language (let's say C++) and even within this language, it's written to confirm to a specific version of the language (a standard like C++11).  The language standard defines what text in the file represents valid syntax.  Our written and spoken languages evolve over time as well, so we also can understand different versions of the same programming lanauge.  Ye Olde Programme Langeuagee.

Syntax is how a thing is presented, as opposed to what a thing means (which is semantics).  

Valid Syntax
------------
In english, we use nouns, verbs, articles and adjectives in certain orders which are "correct".  This includes punctuation which break up meangingful units.  A syntactically valid sentence doesn't always mean something, but we recognize it as valid.

"The horse drove to the hospital to put it's wheels in the ocean" is valid english syntax.  The verbs, articles, nouns and prepositions are all in the right places, as are the spaces, commas, apostrophes, etc.  The meaning, or semantic content, is nonsense, but we can diagram the sentence based on the rules of English Grammar.  

"Drove man hospital I" isn't syntactically valid english, even though we can infer some meaning.  We can't easily digram that sentence in English based on the set of diagrams we are allowed to use.  

Abstract Syntax
---------------
"Abstract" can mean "The idea of", so Abstract Syntax is "The idea of Syntax", or talking about syntax without being specific.  The allowed sentence diagrams also represent Abstract Syntax.  They exist without any particular verb or noun being used.  In fact, they use words or symbols for "verb" and "noun" to be place holders for actual vers and nours.

This lets us talk about the syntax of a lanauge without the specific details of what's being said.  If we collect all the different allowable diagrams, we have all the possible valid ways to order the parts of speech.  The number of possible valid ways is much, much smaller than the number of valid sentences, because we don't have to talk about each sentence.  We can just talk about the structure of the sentence.

Programming lanauge work the same way.  They need use an abstract description of allowed syntax diagrams (a tree diagram, in fact) to formally specify how to write valid statements.  These rules are used by the compilier to test if the statement is valid, and if so, to determine how to start translating the statements piece by piece.

Computers can't infer.  They require rigid rules to operate.   Compilation is asking a computer to read text and perform some complex operations, and so rigid rules are required.  The details of these rules is a large topic, but is related to learning what is and isn't valid C++, and so it's related to the task of learning to program.

Grammar
-------
Lanauges are made up of symbols, like letters, numbers, and punctuation.  We know that words can start with letters, but not numbers or punctuation.  That's a rule.  Grammar is the collection of all the rules of what syntax is valid.  How to spell things, and how to place things in the proper order.

We also know that words are separated by punctuation like spaces, commas, periods, etc.  In a general sense, we can identify words (recognized by applying the rules in the Grammar), by looking at how the punctuation separates symbols from our alphabet.  These rules about how to separate valid words is also in the Grammar.  While we may not be familiar with the definition of all the words in the lanauge, we can still identify distinct words, and even what kind of word they might be, in an unfamiliar sentnece by applying these rules.  

Tokens
------
Let's call the act of separating valid words and symbols "tokenization", and valid words are "tokens".  Another term for this is "parsing".  

If we have rigid rules about the allowed symbols and rigid rules about how symbols near each other, we can parse a list of symbols into tokens without knowing what those tokens actually mean, but knowing what the token is supposed to do.  Is it a noun, like a variable name?  Is it an action, like a function call?


Compilation requires syntactical validity because part of the process is exactly like diagramming a sentence.  Diagramming a sentence doesn't requie knowledge about the sentence before it or after it, only the symbols inbetween certain punctuation.  Likewise, it doesnt' need to know the meaning of the sentnce, and it doesn't even need to know the definition of the words, just that certain words are expected in certain places (like a, the, an, of, etc), and that these placesments indicate that other words are nouns (the name of a variable for instance), and others are verbs (a function or operator).


Syntax, with repsect to programming, is the set of valid symbols we can use, and how we can order those symbols in a file.  Think spelling and punctuation.  Literally, think that.   The totality of valid combinations of syntax is called the Grammar of a language.  Remember that determining validity is applying a rule to a specific combination of symbols and determining if that is valid.  Different languages use different Grammars, but they all have some concept of valid syntax.  Learning a new language is a mix of learning the grammar while also learning the semantics ie the concepts which are invoked by valid grammar.

For the purposes of these guides, we'll define semantics at the machine code, or architecture level.  To get a grasp on what that means exactly, take a look at :ref:`architecture_introduction`.  Compilation will produce machine code based on a bunch of rules.  The machine we execute the code on will determine what that actually means (i.e. blink an LED.)

So, source files have to be written with a specific syntax which is allowed by a version of a programming language.  Let's assume we've done that from here on out.

Now we compile the program.  We tell the computer which syntax we are using (C++, c++11 version) and it then looks at the file and parses it.

