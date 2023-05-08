```k
module MATCHING-LOGIC-SYNTAX-ABSTRACT
    syntax TheoryName [token]
    syntax TheoremName [token]
    syntax MetaVarName [token]
    syntax SymbolName [token]

    syntax SymbolName ::= "in" [token]

    syntax Theory ::= "theory" TheoryName TheoryDeclList "endtheory"
    syntax TheoryList ::= List{Theory, ""} [klabel(TheoryList)]

    syntax TheoryDecl ::= "imports" TheoryName "."

    syntax TheoryDeclList ::= List{TheoryDecl, ""} [klabel(TheoryDeclList)]

    syntax TheoryDecl ::= "metavar" MetaVarNameList ":" MetaVarType "."
    syntax MetaVarType ::= "Pattern" | "EVar" | "SVar"

    syntax TheoryDecl ::= "notation" Pattern ":" Pattern "."
                        | "notation" Pattern "when" ProvableList ":" Pattern "."

    syntax TheoryDecl ::= "symbol" SymbolName "."
    syntax TheoryDecl ::= "axiom" TheoremName ":" ProvableList "."

    syntax Provable ::= Pattern
                      | "<" "disjoint"     MetaVarNameList "from" MetaVarName ">"
                      | "<" "positive"     MetaVarName     "in"   MetaVarName ">"
                      | "<" "substitution" MetaVarName     "is"   MetaVarName "[" Pattern "/" MetaVarName "]" ">"
    syntax ProvableList ::= List{Provable, ""} [klabel(ProvableList)]

    syntax Pattern ::= SymbolName
                     | "(" PatternList ")"
    syntax PatternList ::= List{Pattern, ""} [klabel(PatternList)]
    syntax MetaVarNameList ::= List{MetaVarName, ""} [klabel(MetaVarNameList)]
endmodule

module MATCHING-LOGIC-SYNTAX
    imports BUILTIN-ID-TOKENS
    imports MATCHING-LOGIC-SYNTAX-ABSTRACT
    syntax Name ::= r"[A-Za-z][a-zA-Z0-9-]*" [token, prec(1)]
                  | #LowerId [token]
                  | #UpperId [token]

    syntax TheoryName  ::= Name [token]
    syntax TheoremName ::= Name [token]
    syntax MetaVarName ::= Name [token]
    syntax SymbolName  ::= Name [token]
endmodule

module MATCHING-LOGIC
    imports MATCHING-LOGIC-SYNTAX-ABSTRACT
    imports STRING-SYNTAX

    syntax String ::= SymbolNameToString(SymbolName)    [function, total, hook(STRING.token2string)]
    syntax String ::= MetaVarNameToString(MetaVarName)  [function, total, hook(STRING.token2string)]

    syntax String ::= MetaVarTypeToString(MetaVarType) [total, function]
    rule MetaVarTypeToString(Pattern) => "Pattern"
    rule MetaVarTypeToString(EVar) => "Evar"
    rule MetaVarTypeToString(SVar) => "Svar"

    syntax String ::= MetaVarTypeToLowerString(MetaVarType) [total, function]
    rule MetaVarTypeToLowerString(Pattern) => "pattern"
    rule MetaVarTypeToLowerString(EVar) => "element-variable"
    rule MetaVarTypeToLowerString(SVar) => "set-variable"
endmodule
```
