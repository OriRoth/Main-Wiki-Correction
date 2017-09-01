Ternarization is a Micro abstraction technique which simplifies control. Ternarization is carried out by identifying a recurring pattern in two branches of a conditional statement, and replacing this recurrence by the ternary conditional evaluation operator.


## Ternarizing a conditional return statement ##

Instead of writing

```java
int abs(int x) {
    if (x < 0) 
        return -x;
    return x;
}
```

one should write

```java
int abs(int x) {
    return x > 0 ? x : -x;  
}
```



The famous factorial function

```java
int factorial(int n) {
    if (n <= 1)
        return n;
    else
        return n * fact(n - 1);
}
```

is thus rewritten as

```java
int factorial(int n) {
    return n <= 1 ? n : n * fact(n - 1);
}
```

Here is another example, taken from the JTL code development. The following code excerpt uses the string `s` twice.

```java
if (tail.isEmpty())
        return s;
else
    return s + ":" + tail.toString();
```

This repetition can be expressed by rewriting the above as:

```java
return s + (tail.isEmpty() ? "" : ":" + tail);
```

In the rewrite, we also discovered that the call to method toString was unnecessary.

## Ternarizing a conditional assignment ##

Similarly, instead of

```java
if (c)
   a = x;
else
   a = y;
```

write

```java
a = c ? x : y;
`

## Other ternarizations ##

Ternarization applies even more forcefully to express commonality within expressions and statements which occur at the two branches of a conditional. For example,

```java
if (z > y) 
    f(x + b,c);
else
    f(a + x,c);
```

write

```java
f(x + (z > y) ? b : a , c);
```

Ternarization is also a good opportunity to elicit the common instruction of the two branches of a conditional. Here is an actual example, drawn from method `start()` of class `org.apache.bcel.util.BCELFactory.` Instead of the code

```java
if (i instanceof BranchInstruction) {
    _out.println("    InstructionHandle ih_" + ih.getPosition() + ";");
} else {
    _out.print("    InstructionHandle ih_" + ih.getPosition() + " = ");
}
```

which it takes much pondering in understanding the difference between the two branches, one may more succinctly write

```java
_out.print("    InstructionHandle ih_" + ih.getPosition() + ";");
_out.print(i instanceof BranchInstruction ? ";\n" : " = ");
```

Here is an example drawn from the JTL project code.

```java
int mod;
if (m instanceof Method)
    mod = ((Method) m).getModifiers();
else
    mod = ((Constructor) m).getModifiers();
```

In the above example, the assignment to variable mod is repeated in both branches of the if statement. Similarly repeated is the call to `getModifiers()`. The fact that variable `mod` stores the modifiers of `m` regardless if `m` is a Method or a Constructor is blurred with the many tokens, characters and the control. The alternative,

```java
int mod = (m instanceof Method ?(Method) m :(Constructor) m).getModifiers();
```

is more concise, has fewer repetitions, and hence makes the intention clearer. Even further, placing the code in this form, reveales that the call to `getModifiers()` can only work since both class Method and class Constructor implement a common class or interface. Digging a bit into the `java.lang.reflect` library, we determine that this common super type is `interface Member`. Using this common interface, the conditional can be eliminated:

```java
int mod = ((Member) m).getModifiers();
```

## Example ##

The following function is drawn from the source code of JFLEX.

```java
public Action getHigherPriority(Action other) {
    if (other == null)
        return this;
    // the smaller the number the higher the priority
    if (other.priority > this.priority)
        return this;
    else
        return other;
}
```

Thanks to ternarization, this functions is reduced to a single simple statement:

```java
public Action getHigherPriority(Action other) {
    return other == null || other.priority > this.priority  ? this : other; 
}
```

And, the brevity of the code, justifies using generic names, obtaining an even shorter version

```java
public Action getHigherPriority(Action a) {
    return other == null || a.priority > priority  ? this : other; 
}
```

## Automatic ternarization ##

### Automatic ternarization of returns ###

Searching for

```
(?s)^(\t+)if\s*?\(([^;\{\}]*)\)(\s*?(?://[^\n]*$\s*?)*)\s*?return([^;\{\}]*);(\s*?(?://[^\n]*$\s*?)*)^\1(?:else\s*?)?return([^;\{\}]*);`
```

and replacing it with

```java
return $2 $3 ? $4 $5 : $6;
```

Will convert code snippets such as

```java
if (exp1) 
    return exp2;
else
    return exp3;
if (exp1) 
    return exp2;
return exp3;
```

to

```java
return exp1 ? exp2 : exp3;
```

The K&R indentation style is absolutely essential for this pattern to work correctly. Without it, the pattern may wrongly capture code such as

```java
if (exp1) while(exp) if (exp)
    return exp2;
else
    return exp3;
```

### Automatic ternarization of assignments ###

Searching for

```
(?s)^\t+(else\s*?)?if\s*?\(([^;\{\}]*)\)(\s*?(?://[^\n]*$\s*?)*)\s*?([a-zA-Z_][\.\w]*)\s*?=\s*?([^;\{\}]*);(\s*?(?://[^\n]*$\s*?)*)^\t+else\s*?\4\s*?=\s*?([^;\{\}]*);
```

and replacing it with

```java
$1 $4 = $2 ?  $5 : $7; $3 $6
```

will apply ternarization to code of the following format

```java
if (condition)
    var = expression1;
else
    var = expression2;
```

### Cautions ###

Regular expressions are tricky. Apply caution and care before using a global search and replace with these to modify your code.

* The regular expressions presented here assume the code is formatted strictly according to the Kernighan and Ritchie indentation style style. To be on the safe side, use and automatic code formatter, such as indent -kr, or Eclipse auto formatting together with the K&R setting.
* The replacement text does not preserve indentation. Applying the substitution will destroy the carefully indented code. This is yet another reason to use automatic indentation the tool of your choice.
* Beware that regular expression search may be easily fooled by comments and strings and may therefore produce wrong results: it may fail to recognize correct code which should be substituted; worse, it may change correct code in unexpected ways. It is prudent to inspect each such replacement manually and to apply some automatic checking to increase confidence that a global replacement, even if monitored by a human, did not cause the code to behave in unexpected ways.
