# Lecture 15: Context Sensitive Analysis

* Dupliate variables/procedures
* Missing definitions of variables/procedures
* Types?

## Missing Variable Definitions

Keep track of all variables that have been declared - create a symbol table!

* Traverse the tree & find all declarations of variables
  * `dcl -> Type ID`
  * Store the variable name (string) & type (string)
    * `map<string, string> symbTable;`
  * Before inserting tinto the symbol table, check that an entry for this variable does not exist
    * Error if it's already in the symbol table
* We could traverse the tree again once the symbol table has been created
  * Looking for uses of variables
    * factor -> ID
    * lvalue -> ID

```none
Interesting problem in wlp4:

int x = 5;
x = 10;
int y = x + 2; <- error...
```

Since all declarations in WLP4 must appear before any other statements, it is possible to do this in one pass

```c++
// WLP4 CODE

int f () {
  int x = 0;
  return 1;
}

int wain (int a, int b) {
  int x = 0;
  return 1;
}

// OK as x is a local variable in x and wain respectively
```

## Handling Scopes

Create a symbol table for each procedure.

```c++
// WLP4 CODE

int f () {
  int x = 0;
  return 1;
}

int wain (int a, int b) {
  return X; // NOT OK - X UNDEFINED
}
```

When checking for the use of endeclared variables, make sure we check the correct symbol table

* Procedures
  * Multiple definitions
  * Procedure calls to undeclared procedures

Create a map that maps procedure names to its variable's symbol table

```none
map<string, map<string, string>> topLevelSymbolTable
```

## Filling the top level symbol table

1. Traverse the tree looking for procedures

    * `procedure -> INT ID LPAREN ...`
      * The name is ID
    * `main -> INT WAIN LPAREN`

2. Check that the top level symbol table does not already have an entry for this procedure

3. If found, add an entry to this table

Let's store the types of the parameters of each procedure.

* paramList -> dcl
* paramList -> dcl COMMA paramList
* paramList -> ë

The "signature" is simply a list of all parameter  & variable symbol tables. All we need is a `vector<string>`.

**Option 1**: create a map from procedure name to vector of parameter types: `map<string, vector<string>>`. 

**Option 2**: Use the top level smybol table:

```c++
map<string, pair<vector<string>, map<string, string>>>
// Maps the procedure name to a pair of the parameter types 
```

## Checking Type Errors

Types allow us to assign interpretation to the contents of some memory address. A good type system (stype rules) prevents us from re-interpreting the contents or something else.

```c
int *a = NULL;
a = 7; // Rule is broken! Expected integer pointer - TYPE ERROR
```

## WLP4 types

We have `int` and `int*`.

## Context Sensitive Analysis Type Correctness

* Determine the types associated with variables/expressions
* Ensure operators are applied to operands of the correct type
* We already have types for variables
* Determine type of every expression by applying type rules
* If no rule is applicable, or if an expression's type does not match the context it is used, TYPE ERROR.

```none
string typeof(Tree &t) {
  for each child c in t.children
    typeof(c); // recursively determine types for subtrees

  Use t.rule to determine what type rule is applicable, use the
  computed type sof the children to determine if type rule
  is satisfied
}
```