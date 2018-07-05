### Parser
Parsing consists of lexical analyze (tokenizer) and syntactic analyze. 

Essentially, tokenizer is a DFA(Deterministic finite automaton) about which I do not have much to talk.

Rather, I would like to keep some notes after reading this [zhihu article](https://zhuanlan.zhihu.com/p/24035780)

Syntactic analyze is the process of taking a stream of tokens and converting them into an AST.

Given a stream of tokens. We need to design a set of rules and match tokens with those rules to give tokens 
syntax meaning.

To design the set of rules, we need to remove ambiguity, design priority of analyzing (how to split and group tokens),
and avoid left recursion.

If all above conditions are satisfied, we will be able to write descendant recursion to parse tokens. The key is "descendant",
the rules are normally defined recursively. Thus, the parsing is also using a set of mutual recursive functions. But each 
invocation of recursive parsing, the total number of unparsed tokens are decreasing.

Also LL(1) means we can decide which rules to apply without looking ahead of any token. Some rules, when parsing, 
need backtracing. Because some certain point of time, for current token, we have multiple choices of rules to apply.
In this case, what we can do is to try them one by one. Thus, this kind of rule needs backtrace to parse.


### Transformer
Transformer consists of two parts: traversor and visitor. Traversor controls how to visit each node in AST based on
current node type. Visitor controls how to transform the current node.
