# Translating a subset of Java to C

## Goal

In this project, you must translate a very small subset of Java to pure C using ANTLR and Java code that you write. The subset has very few statements and almost no expressions, focusing instead on classes and methods. You will learn not only about one which translation but also how polymorphism is implemented using so-called `vtables`, which C++ also uses. It requires a deep understanding of C types as well.

To get started, please familiarize yourself with the [Java translator starter kit](https://github.com/USF-CS652-starterkits/parrt-vtable). The main program is `JTran.java`.

## Discussion

### Sample input

To get a flavor of the Java subset, take a look at the following `.j` files, which demonstrate all of the tricky polymorphism and dynamic binding your translated C code must implement.

```java
// tests/cs652/j/polymorph.j
class Animal {
    int ID;
    int getID() { return ID; }
    void speak() { printf("%d\n", getID()); }
}

class Dog extends Animal {
    void speak() { printf("woof!\n"); }
}

Animal a;
Dog d;
d = new Dog();
a = d; // should cast to Animal *
a.speak(); // prints woof!
d.speak(); // prints woof!
```

Check out the [expected C code](https://github.com/USF-CS652-starterkits/parrt-vtable/blob/master/tests/cs652/j/polymorph.c).

```java
// tests/cs652/j/vtable_check.j
class Animal {
    int getID() { return 1; }
    int foo() { return getID(); }
}

class Dog extends Animal {
    int getID() { return 2; }
}

class Pekinese extends Dog {
    int getID() { return 3; }
}

Pekinese d;
d = new Pekinese();
printf("%d\n", d.foo()); // must print 3
```

Check out the [expected C code](https://github.com/USF-CS652-starterkits/parrt-vtable/blob/master/tests/cs652/j/vtable_check.c).

A file consists of zero or more class definitions file optionally by a main program followed by end of file:

```
grammar J;

file:   classDeclaration* main EOF
    ;
```

You can see all of the [sample inputs I used for testing](https://github.com/USF-CS652-starterkits/parrt-vtable/tree/master/tests/cs652/j).

## Tasks

### Creating the J grammar

Mechanically, your goal is to fill in the `J.g4` grammar by looking at all of the examples and the standard [ANTLR Java grammar](https://github.com/antlr/grammars-v4/blob/master/java/Java.g4). (I used as a template to cut it down to my `J.g4`.) Learning how to examine exemplars of a language and construct a suitable grammar is important but here are a few details that matter in terms of symbol table management and type analysis.

* Assume all input is syntactically and semantically valid J(ava) code with the exception that statements existing outside of class definitions are considered the main program. Other than that, assume Java syntax and semantics.
* Only integers and floating-point literals are valid. For floating-point numbers, don't worry about negation or exponents: just match and integer on either side of a decimal point.
* Identifiers are just the usual upper and lowercase letters, underscores, and digits (but not in the first position).
* Allow `null` and `this`
* Constructor definitions are not allowed but we still use syntax `new T()` (without parameters) to create objects.
* There are no access modifiers like `public`; everything is assumed to be `public`.
* To print things out there is a single predefined function called `printf(STRING)` or with variable number of arguments `print(STRING, args...)`.
* String literals are only allowed as the first argument of `printf()` calls. Strings should allow a single escape of `\"` but none other.
* There are no variable initializers like `int x=1;`. You must do `int x; x=1;`.
* There are no operators and so you don't have to worry about operator precedence but you do have to support criticize expressions for grouping.

### Defining scopes and symbols

2. Define J symbol table objects using `src/org/antlr/symbols` objects as superclasses as necessary:
	JArg.java
	JClass.java
	JField.java
	JMethod.java
	JObjectType.java
	JPrimitiveType.java
	JVar.java
2. `DefineScopesAndSymbols.java`

### Computing expression types

3. `SetScopes.java`
4. `ComputeTypes.java`

### Constructing a model

### Generating C code from the model

## Testing

All [sample inputs I used for testing](https://github.com/USF-CS652-starterkits/parrt-vtable/tree/master/tests/cs652/j) are available. For each test `T`, you will find `T.j`, `T.c`, and `T.txt` where `T.txt` is the expected output after you compile and execute the program. You can run all of the tests like this:

```bash
./bild.py -debug tests
```

Remember, the definition of “working” is when your grammar correctly parses all of the .j files. If you use the -tree option from the command line with JTran.java that I provide, it will pop up a visual of the parse tree for you.