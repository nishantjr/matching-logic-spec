We define the syntax of Metamath as per the syntax
[manual](https://us.metamath.org/downloads/metamath.pdf), Section 4.1.3.

```k
module METAMATH-TOKENS-ABSTRACT
    imports BUILTIN-ID-TOKENS
    syntax Path       [token]
    syntax Label      [token]
    syntax MathSymbol [token]

    // Explicit tokens useful for matching logic.
    syntax MathSymbol ::= "#Pattern"            [token]
                        | "#Symbol"             [token]
                        | "#Variable"           [token]
                        | "#ElementVariable"    [token]
                        | "#SetVariable"        [token]

                        | "#Positive"           [token]
                        | "#Negative"           [token]
                        | "#Fresh"              [token]
                        | "#ApplicationContext" [token]
                        | "#Substitution"       [token]
                        | "#Notation"           [token]
                        | "|-"                  [token]
endmodule

module METAMATH-TOKENS
    imports METAMATH-TOKENS-ABSTRACT
    syntax #Layout ::= r"\\$\\(([^\\$]*(\\$[^\\(])?)*\\$\\)" // "$(" (not-dollar* (dollar not-close-paren)?)* ")$"
                     | r"([\\ \\n\\r\\t])" // Whitespace
    syntax Path       ::= MathSymbol [token]
    syntax Label      ::= MathSymbol [token]
    syntax MathSymbol ::= r"[!-#%-|][!-#%-|]*" [prec(1), token]
                          | #LowerId [token]
                          | #UpperId [token]
                          | "(" [token]
                          | ")" [token]
endmodule

module METAMATH-SYNTAX-ABSTRACT
    imports METAMATH-TOKENS-ABSTRACT

    syntax Metamath ::= StmtSeq
    syntax StmtSeq ::= List{Stmt, ""}   [klabel(StmtSeq)]

    syntax Stmt ::= "$[" Path "$]"
    syntax Stmt ::= "${" StmtSeq "$}" [format(%1%i%n  %2%n%3%d%n)]

    syntax Stmt ::= "$c" MathSymbolSeq "$."  [format(%1 %2 %3%n)]   // Constant statement
    syntax Stmt ::= "$v" MathSymbolSeq "$."  [format(%1 %2 %3%n)]   // Variable statement

    syntax Stmt ::= Label "$f" MathSymbol MathSymbol "$."    [format(%1 %2 %3 %4 %5%n)]  // Floating Hypothesis
    syntax Stmt ::= Label "$e" MathSymbol MathSymbolSeq "$." [format(%1 %2 %3 %4 %5%n)] // Essential Hypothesis

    syntax Stmt ::= "$d" MathSymbolSeq "$." [format(%1 %2 %3%n)]  // Simple+Compound disjointness

    syntax Stmt ::= Label "$a" MathSymbol MathSymbolSeq "$."  [format(%1 %2 %3 %4 %5%n)] // Axiom statement
    syntax Stmt ::= Label "$p" MathSymbol MathSymbolSeq "$=" LabelSeq "$." [format(%1 %2 %3 %4 %5 %6 %7%n)] // Proof statement

    syntax MathSymbolSeq ::= List{MathSymbol, ""} [klabel(MathSymbolSeq)]
    syntax LabelSeq      ::= List{Label, ""}      [klabel(LabelSeq)]
endmodule

module METAMATH-SYNTAX
    imports METAMATH-SYNTAX-ABSTRACT
    imports METAMATH-TOKENS
endmodule

module METAMATH
    imports METAMATH-SYNTAX-ABSTRACT
    imports STRING-SYNTAX

    syntax StmtSeq ::= StmtSeq "++StmtSeq" StmtSeq [total, function]
    rule .StmtSeq ++StmtSeq Stmts2 => Stmts2
    rule (Stmt Stmts1) ++StmtSeq Stmts2 => Stmt (Stmts1 ++StmtSeq Stmts2)

    syntax MathSymbolSeq ::= MathSymbolSeq "++MathSymbolSeq" MathSymbolSeq [total, function, left, avoid]
    rule .MathSymbolSeq ++MathSymbolSeq MathSymbols2 => MathSymbols2
    rule (MathSymbol MathSymbols1) ++MathSymbolSeq MathSymbols2 => MathSymbol (MathSymbols1 ++MathSymbolSeq MathSymbols2)

    syntax Label ::= StringToLabel(String) [function, total, hook(STRING.string2token)]
    syntax MathSymbol ::= StringToMathSymbol(String) [function, total, hook(STRING.string2token)]
endmodule
```
