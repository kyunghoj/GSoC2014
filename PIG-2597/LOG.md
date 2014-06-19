2014-06-10
===

`QueryParser.g` starts with the following definition:

```
query : statement* EOF -> ^( QUERY statement* )
;
```

`QUERY` seems like an imaginary token. 

Anyway, the output of the parser is `AST`, but it does not need to be the case
if it is a `Grunt` command.

However, what if a `Grunt` command is included in a macro?

2014-06-09
===

> Pig allows three modes of user interaction:
>
> 1. Interactive mode: In this mode, the user is presented with an (...)
>
> 2. Batch mode: In this mode, a user submits a pre-written script containing
> a series of Pig commands, typically ending with STORE (...)
>
> 3. Embedded mode: Pig is also provided as a Java library allowing Pig Latin
> commands to be submitted via method invocations from a Java program. (...)
>

Keep in mind that it needs to support embedded mode. 

One way to think about is to implement a single parser that parses everything
(Grunt command as well as Pig Latin). If it sees a Pig Latin script, then 
it will build a logical plan. But a Grunt command comes, it can just execute the 
command and do not add to a logical plan.

`GruntParser.processPig(String)` is called from `PigScriptParser.parse()`. 

`PigScriptParser.parse()` is called from:
 * `GruntParser.loadScript(String, boolean, ...)`,
 * `GruntParser.parseOnly()`, or
 * `GruntParser.parseStopOnError(boolean)`.

Doing
---

Created `PigScriptLexer.g` be copying `QueryLexer.g`. Adding Grunt commands to the lexer.

But the first job is to understand ANTLR lexer and parser generation.

2014-05-31
===

Goal
---
 * ANTLR-based Grunt parser: by itself is not super-complicated
 * Use all commands in Macro, especially `REGISTER` and parameter substitution (https://issues.apache.org/jira/browse/PIG-2597?focusedCommentId=13936422&page=com.atlassian.jira.plugin.system.issuetabpanels:comment-tabpanel#comment-13936422)
 

Macro
---

`REGISTER` command
---
`void processRegister(String jar)` in `class GruntParser` deals with it.
It is basically a frontend to `PigServer.registerJar(path)` or
`PigServer.registerCode(...)`


Development Environment Configuration
---
I forked apache/pig repo in github.com and cloned to `~/git/gsoc2014/pig` directory.
Then, I added `remote`. Please see "Fork a repo" (https://help.github.com/articles/fork-a-repo) for the future reference.

```
kyunghoj@jeju:~/git/gsoc2014/pig$ git checkout -b trunk origin/trunk
Branch trunk set up to track remote branch trunk from origin.
Switched to a new branch 'trunk'
kyunghoj@jeju:~/git/gsoc2014/pig$ git branch
  branch-0.1
* trunk
kyunghoj@jeju:~/git/gsoc2014/pig$ git remote add upstream https://github.com/apache/pig.git
kyunghoj@jeju:~/git/gsoc2014/pig$ git fetch upstream
remote: Counting objects: 512, done.
remote: Compressing objects: 100% (375/375), done.
remote: Total 512 (delta 288), reused 96 (delta 44)
Receiving objects: 100% (512/512), 415.96 KiB | 371 KiB/s, done.
Resolving deltas: 100% (288/288), done.
From https://github.com/apache/pig
 * [new branch]      branch-0.1 -> upstream/branch-0.1
 * [new branch]      branch-0.10 -> upstream/branch-0.10
 * [new branch]      branch-0.11 -> upstream/branch-0.11
 * [new branch]      branch-0.12 -> upstream/branch-0.12
 * [new branch]      branch-0.13 -> upstream/branch-0.13
 * [new branch]      branch-0.2 -> upstream/branch-0.2
 * [new branch]      branch-0.3 -> upstream/branch-0.3
 * [new branch]      branch-0.4 -> upstream/branch-0.4
 * [new branch]      branch-0.5 -> upstream/branch-0.5
 * [new branch]      branch-0.6 -> upstream/branch-0.6
 * [new branch]      branch-0.7 -> upstream/branch-0.7
 * [new branch]      branch-0.8 -> upstream/branch-0.8
 * [new branch]      branch-0.9 -> upstream/branch-0.9
 * [new branch]      branch-X.Y -> upstream/branch-X.Y
 * [new branch]      load-store-redesign -> upstream/load-store-redesign
 * [new branch]      multiquery -> upstream/multiquery
 * [new branch]      plan       -> upstream/plan
 * [new branch]      pre-multiquery-phase2 -> upstream/pre-multiquery-phase2
 * [new branch]      pretypes   -> upstream/pretypes
 * [new branch]      tez        -> upstream/tez
 * [new branch]      trunk      -> upstream/trunk
```

Then, the changes from the upstream should be merged.

```
kyunghoj@jeju:~/git/gsoc2014/pig$ git merge upstream/trunk
```

Additionally, `javacc-4.2.jar` should be deleted from `.classpath` and `perf` 
should be excluded from `build.xml`. 

```
kyunghoj@jeju:~/git/gsoc2014/pig$ git diff build.xml
diff --git a/build.xml b/build.xml
index 7dcbe75..63cbfea 100644
--- a/build.xml
+++ b/build.xml
@@ -320,7 +320,7 @@
                 <source path="${src.shims.dir}"/>
                 <source path="${src.shims.test.dir}"/>
                 <source path="tutorial/src"/>
-                <source path="${test.src.dir}" excluding="e2e/pig/udfs/java/|resources/"/>
+                <source path="${test.src.dir}" excluding="e2e/pig/udfs/java/|perf/|resources/"/>
                 <output path="${build.dir.eclipse-main-classes}" />
                 <library pathref="eclipse.classpath" exported="true" />
                 <!--library pathref="classpath" exported="false"/-->
```

List of relevant pacackages and classes
---
 * `Grunt`
 * `GruntParser` <-- `PigScriptParser`
 * `DryRunGruntParser`
 * `org.apache.pig.parser.*` (ANTLR-based parser and its helper classes)

Memo
---
 * Rewrite PigScriptParser and merge with QueryParser?

 
TODO
---
 * Add one or two test codes and see if they work

2014-05-23
===

Related packages
---

 * org.apache.pig.tools.pigscript: JavaCC-based parser generator code
 * org.apache.pig.tools.grunt: (kind of) frontend to Pig, PigScriptParser, ...
 * org.apache.pig.parser: ANTLR-based Query parser and LP Generator code

Plan
---
 * Merge them into one package, (maybe) org.apache.pig.parser

How Pig Macro Works
===

A Brief Primer
---

A macro definition can appear anywhere in a Pig script
as long as it appears before its first use. A macro definition
can also include references to other macros, but recursive references
are not allowed.

Macro cannot be used inside a `FOREACH` nested block. Grunt shell commands
are not allowed in macros (need to be fixed in this project).

An example:
```
DEFINE my_macro(A, sortkey) RETURNS C {
    B = FILTER $A BY my_filter(*);
    $C = ORDER B BY $sortkey;
}
```

Only aliases A and C are visible from the outside of the macro.

What is "inline macro"?

Call Hierarchy
---

`QueryParserDriver.expandMacro(Tree)`
<-`QueryParserDriver.parse(String)`
  <-`PigServer.Graph.parseQuery()` or
  <-`PigServer.Graph.validateQuery()`


