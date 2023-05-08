```k
requires "matching-logic.md"
requires "metamath.md"

module MATCHING-LOGIC-TO-METAMATH-SYNTAX
    imports MATCHING-LOGIC-SYNTAX
endmodule

module MATCHING-LOGIC-TO-METAMATH
    imports MATCHING-LOGIC
    imports METAMATH
    imports STRING

    configuration <matching-logic> $PGM:TheoryList ~> . </matching-logic>
                  <metamath> .StmtSeq </metamath>

    rule <matching-logic> (Theory Theories):TheoryList => Theory ~> Theories ... </matching-logic>
    rule <matching-logic> theory _TheoryName Decls endtheory => Decls ... </matching-logic>

    rule <matching-logic> Decl Decls:TheoryDeclList => Decl ~> Decls ... </matching-logic>
```

We ignore these for now.
Later, we may want to only export a single theory and its transitively imported modules.

```k
    rule <matching-logic> imports _TheoryName . => .K  ... </matching-logic>
```

## Meta Variables

Meta-variables are translated to floating hypothesis.

```k
    rule <matching-logic> metavar .MetaVarNameList : _:MetaVarType . => .K ... </matching-logic>
    rule <matching-logic> metavar V Vs:MetaVarNameList : MetaVarType .
                       => metavar   Vs:MetaVarNameList : MetaVarType .
                          ...
         </matching-logic>
         <metamath>  Metamath => Metamath ++StmtSeq
                     $v MetaVarNameToMathSymbol(V) $.
                     metavarSyntaxAssertionName(V, MetaVarType) $f MetaVarTypeToMathSymbol(MetaVarType) MetaVarNameToMathSymbol(V) $.
                     .StmtSeq
         </metamath>
```

## Notation

```k
    rule <matching-logic> notation Notation : Unfolding . => .K ... </matching-logic>
         <metamath>  Metamath => Metamath ++StmtSeq
                     $c SymbolNameToMathSymbol(getNotationName(Notation)) $.
                     notationSyntaxAssertionName(Notation)  $a #Pattern  PatternToMathSymbolSeq(Notation) $.
                     notationDesugarAssertionName(Notation) $a #Notation (PatternToMathSymbolSeq(Notation) ++MathSymbolSeq PatternToMathSymbolSeq(Unfolding)) $.
                     .StmtSeq
         </metamath>
```

```k
    syntax SymbolName ::= getNotationName(Pattern) [function]
    rule getNotationName((Name:SymbolName _Args)) => Name
    rule getNotationName(Name:SymbolName) => Name

    syntax MathSymbol ::= SymbolNameToMathSymbol(SymbolName) [total, function]
    rule SymbolNameToMathSymbol(SymbolName) => StringToMathSymbol(SymbolNameToString(SymbolName))

    syntax Label ::= metavarSyntaxAssertionName(MetaVarName, MetaVarType) [total, function]
    rule metavarSyntaxAssertionName(Name, Type) => StringToLabel(MetaVarNameToString(Name) +String "-is-" +String MetaVarTypeToLowerString(Type))

    syntax Label ::= notationSyntaxAssertionName(Pattern) [total, function]
    rule notationSyntaxAssertionName(Notation)
      => StringToLabel(SymbolNameToString(getNotationName(Notation)) +String "-is-pattern")

    syntax Label ::= notationDesugarAssertionName(Pattern) [total, function]
    rule notationDesugarAssertionName(Notation)
      => StringToLabel(SymbolNameToString(getNotationName(Notation)) +String "-is-sugar")


    syntax MathSymbolSeq ::= PatternToMathSymbolSeq(Pattern) [function, total]
    rule PatternToMathSymbolSeq(SymbolName) => SymbolNameToMathSymbol(SymbolName)
    rule PatternToMathSymbolSeq(( Ps:PatternList ):Pattern)
      => #token("(", "MathSymbol") (PatternListToMathSymbolSeq(Ps) ++MathSymbolSeq #token(")", "MathSymbol"))

    syntax MathSymbolSeq ::= PatternListToMathSymbolSeq(PatternList) [function, total]
    rule PatternListToMathSymbolSeq(.PatternList) => .MathSymbolSeq
    rule PatternListToMathSymbolSeq(P Ps) => PatternToMathSymbolSeq(P) ++MathSymbolSeq PatternListToMathSymbolSeq(Ps)

    syntax MathSymbol ::= MetaVarNameToMathSymbol(MetaVarName) [total, function]
    rule MetaVarNameToMathSymbol(Name) => StringToMathSymbol(MetaVarNameToString(Name))

    syntax MathSymbol ::= MetaVarTypeToMathSymbol(MetaVarType) [total, function]
    rule MetaVarTypeToMathSymbol(Pattern) => #Pattern
    rule MetaVarTypeToMathSymbol(EVar) => #ElementVariable
    rule MetaVarTypeToMathSymbol(SVar) => #SetVariable
```

```k
endmodule
```
