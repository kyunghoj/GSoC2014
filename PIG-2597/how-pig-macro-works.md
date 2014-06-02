How Pig Macro Works
===

A Brief Primer
---

A macro definition can appear anywhere in a Pig script
as long as it appears before its first use. A macro definition
can also include references to other macros, but recursive references
are not allowed.

Macro cannot be used inside a `FOREACH` nested block. Grunt shell commands
are not allowed in macros.

An example:
```
DEFINE my_macro(A, sortkey) RETURNS C {
    B = FILTER $A BY my_filter(*);
    $C = ORDER B BY $sortkey;
}
```
Only aliases A and C are visible from the outside of the macro.

Call Hierarchy
---

`QueryParserDriver.expandMacro(Tree)`
<-`QueryParserDriver.parse(String)`
  <-`PigServer.Graph.parseQuery()` or
  <-`PigServer.Graph.validateQuery()`


