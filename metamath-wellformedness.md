```k
require "metamath.md"
```

```k
module MATCHING-LOGIC-SYNTAX-ABSTRACT
    imports METAMATH-SYNTAX-ABSTRACT
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

    syntax Label ::= "var-is-pattern" [token]
                   | "symbol-is-pattern" [token]
                   | "element-var-is-var" [token]
                   | "set-var-is-var" [token]
endmodule

module MATCHING-LOGIC-SYNTAX
    imports METAMATH-SYNTAX
    imports MATCHING-LOGIC-SYNTAX-ABSTRACT
endmodule

module MATCHING-LOGIC
    imports MATCHING-LOGIC-SYNTAX-ABSTRACT
    imports BOOL
```

```k
    configuration <k> $PGM:Metamath </k>
                  <metavariables> .Map </metavariables>
```

```k
    rule <k> S:Stmt Ss:StmtSeq => S ~> Ss ... </k>
```

# Syntax of Matching Logic

For each built-in, a constructor is defined.

```k
    rule <k> $c MathSymbol $. => .K ... </k> requires isBuiltin(MathSymbol)
```

We have the various subsorting inclusions:

```k
    rule <k> element-var-is-var $a #Variable EVar$. => .K ... </k>
         <metavariables> EVar:MathSymbol |-> #ElementVariable ... </metavariables>
    rule <k> set-var-is-var $a #Variable SVar$. => .K ... </k>
         <metavariables> SVar:MathSymbol |-> #SetVariable ... </metavariables>
    rule <k> var-is-pattern $a #Pattern Var$. => .K ... </k>
         <metavariables> Var:MathSymbol |-> #Variable ... </metavariables>
    rule <k> symbol-is-pattern $a #Pattern Symbol$. => .K ... </k>
         <metavariables> Symbol:MathSymbol |-> #Symbol ... </metavariables>
```

# Meta variables

We collect the type of each metavariable.

```k
    rule <k> $v Sym Syms:MathSymbolSeq $.
          => $v     Syms:MathSymbolSeq $.
             ...
         </k>
         <metavariables> .Map => (Sym |-> #UnknownMetaVarType)
                         ...
         </metavariables>
    rule <k> $v .MathSymbolSeq $. => .K ... </k>

    // TODO: Enforce that the label for this hypothesis is the correct one.
    rule <k> _Label $f MVarType Sym $. => .K ... </k>
         <metavariables> Sym |-> (#UnknownMetaVarType => MVarType) ... </metavariables>
      requires isMetaVarType(MVarType)

    syntax KItem ::= "#UnknownMetaVarType"
```

# Helper functions

```k
    syntax Bool ::= isBuiltin(MathSymbol) [total, function]
    rule isBuiltin(#Positive) => true
    rule isBuiltin(#Negative) => true
    rule isBuiltin(#Fresh) => true
    rule isBuiltin(#ApplicationContext) => true
    rule isBuiltin(#Substitution) => true
    rule isBuiltin(#Notation) => true
    rule isBuiltin(|-) => true
    rule isBuiltin(Sym) => isMetaVarType(Sym) [owise]

    syntax Bool ::= isMetaVarType(MathSymbol) [total, function]
    rule isMetaVarType(#Pattern) => true
    rule isMetaVarType(#Symbol) => true
    rule isMetaVarType(#Variable) => true
    rule isMetaVarType(#ElementVariable) => true
    rule isMetaVarType(#SetVariable) => true
    rule isMetaVarType(_) => false [owise]
```

```k
endmodule
```
