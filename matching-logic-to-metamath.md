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
    imports INT

    configuration <matching-logic> $PGM:SpecList ~> . </matching-logic>
                  <metamath> .StmtSeq </metamath>

    rule <matching-logic> (Spec Specs):SpecList => Spec ~> Specs ... </matching-logic>
    rule <matching-logic> spec _SpecName Decls endspec => Decls ... </matching-logic>

    rule <matching-logic> Decl Decls:SpecDeclList => Decl ~> Decls ... </matching-logic>
```

We ignore these for now.
Later, we may want to only export a single spec and its transitively imported modules.

```k
    rule <matching-logic> imports _SpecName . => .K  ... </matching-logic>
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
    rule <matching-logic> notation Notation when Hypotheseses : Unfolding . => .K ... </matching-logic>
         <metamath>  Metamath => Metamath ++StmtSeq
                     $c SymbolNameToMathSymbol(getNotationName(Notation)) $.
                     notationSyntaxAssertionName(Notation)  $a #Pattern  PatternToMathSymbolSeq(Notation) $.
                     ${ ProvableListToHypothesisList(notationDesugarAssertionName(Notation), Hypotheseses) ++StmtSeq
                        notationDesugarAssertionName(Notation) $a #Notation (PatternToMathSymbolSeq(Notation) ++MathSymbolSeq PatternToMathSymbolSeq(Unfolding)) $.
                        .StmtSeq
                     $}
                     .StmtSeq
         </metamath>
```

```k
    syntax Stmt ::= ProvableToHypothesis(Label, Provable) [function, total]
    rule ProvableToHypothesis(Label, P:Pattern)                                      => Label $e |- PatternToMathSymbolSeq(P) $.
    rule ProvableToHypothesis(Label, < substitution Phi is Context [ Plug / Var ] >) => Label $e #Substitution MetaVarNameToMathSymbol(Phi) MetaVarNameToMathSymbol(Context) PatternToMathSymbolSeq(Plug) ++MathSymbolSeq MetaVarNameToMathSymbol(Var) $.
    rule ProvableToHypothesis(Label, < positive Var in Context >)                    => Label $e #Positive MetaVarNameToMathSymbol(Var) MetaVarNameToMathSymbol(Context) $.
    rule ProvableToHypothesis(_,     < disjoint Vars from MVar >)                     =>      $d MetaVarNameToMathSymbol(MVar) MetaVarNameListToMathSymbolSeq(Vars) $.

    syntax StmtSeq ::= ProvableListToHypothesisList(Label, ProvableList) [function, total]
    rule ProvableListToHypothesisList(Label, Ps) => ProvableListToHypothesisListN(Label, 0, Ps)

    syntax StmtSeq ::= ProvableListToHypothesisListN(Label, Int, ProvableList) [function, total]
    rule ProvableListToHypothesisListN(Label, N, P Ps)
      => ProvableToHypothesis(appendLabel(Label, "." +String Int2String(N)), P)
         ProvableListToHypothesisListN(Label, N, Ps)
    rule ProvableListToHypothesisListN(_, _, .ProvableList) => .StmtSeq

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

    syntax MathSymbolSeq ::= MetaVarNameListToMathSymbolSeq(MetaVarNameList) [total, function]
    rule MetaVarNameListToMathSymbolSeq(.MetaVarNameList) => .MathSymbolSeq
    rule MetaVarNameListToMathSymbolSeq(V Vs) => MetaVarNameToMathSymbol(V) ++MathSymbolSeq MetaVarNameListToMathSymbolSeq(Vs)

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
