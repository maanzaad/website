---
layout: docpage
navgroup: docs
title: "Gendarme Rules: Smells"
---

[Gendarme]({{ site.github.url }}/old_site/Gendarme "Gendarme")'s refactoring suggestion rules are located in the **Gendarme.Rules.Smells.dll** assembly. Latest sources are available from [anonymous SVN](http://anonsvn.mono-project.com/viewcvs/trunk/mono-tools/gendarme/rules/Gendarme.Rules.Smells/).

<table>
<col width="100%" />
<tbody>
<tr class="odd">
<td align="left"><h2>Table of contents</h2>
<ul>
<li><a href="#rules">1 Rules</a>
<ul>
<li><a href="#avoidcodeduplicatedinsameclassrule">1.1 AvoidCodeDuplicatedInSameClassRule</a></li>
<li><a href="#avoidcodeduplicatedinsiblingclassesrule">1.2 AvoidCodeDuplicatedInSiblingClassesRule</a></li>
<li><a href="#avoidlargeclassesrule">1.3 AvoidLargeClassesRule</a></li>
<li><a href="#avoidlongmethodsrule">1.4 AvoidLongMethodsRule</a></li>
<li><a href="#avoidlongparameterlistsrule">1.5 AvoidLongParameterListsRule</a></li>
<li><a href="#avoidmessagechainsrule">1.6 AvoidMessageChainsRule</a></li>
<li><a href="#avoidspeculativegeneralityrule">1.7 AvoidSpeculativeGeneralityRule</a></li>
<li><a href="#avoidswitchstatementsrule">1.8 AvoidSwitchStatementsRule</a></li>
</ul></li>
<li><a href="#feedback">2 Feedback</a></li>
</ul></td>
</tr>
</tbody>
</table>

Rules
=====

### AvoidCodeDuplicatedInSameClassRule

This rule checks for duplicated code in the same class.

**Bad** example:

``` csharp
public class MyClass {
    private IList myList;
 
    public MyClass () {
        myList = new ArrayList ();
        myList.Add ("Foo");
        myList.Add ("Bar");
        myList.Add ("Baz");
    }
 
    public void MakeStuff () {
        foreach (string value in myList) {
            Console.WriteLine (value);
        }
        myList.Add ("FooReplied");
    }
 
    public void MakeMoreStuff () {
        foreach (string value in myList) {
            Console.WriteLine (value);
        }
        myList.Remove ("FooReplied");
    }
}
```

**Good** example:

``` csharp
public class MyClass {
    private IList myList;
 
    public MyClass () {
        myList = new ArrayList ();
        myList.Add ("Foo");
        myList.Add ("Bar");
        myList.Add ("Baz");
    }
 
    private void PrintValuesInList () {
        foreach (string value in myList) {
            Console.WriteLine (value);
        }
    }
 
    public void MakeStuff () {
        PrintValuesInList ();
        myList.Add ("FooReplied");
    }
 
    public void MakeMoreStuff () {
        PrintValuesInList ();
        myList.Remove ("FooReplied");
    }
}
```

### AvoidCodeDuplicatedInSiblingClassesRule

This rule looks for code duplicated in sibling subclasses.

**Bad** example:

``` csharp
public class BaseClassWithCodeDuplicated {
    protected IList list;
}
 
public class OverriderClassWithCodeDuplicated : BaseClassWithCodeDuplicated {
    public void CodeDuplicated ()
    {
        foreach (int i in list) {
            Console.WriteLine (i);
        }
        list.Add (1);
    }
}
 
public class OtherOverriderWithCodeDuplicated : BaseClassWithCodeDuplicated {
    public void OtherMethod ()
    {
        foreach (int i in list) {
            Console.WriteLine (i);
        }
        list.Remove (1);
    }
}
```

**Good** example:

``` csharp
public class BaseClassWithoutCodeDuplicated {
    protected IList list;
 
    protected void PrintValuesInList ()
    {
        foreach (int i in list) {
            Console.WriteLine (i);
        }
    }
}
 
public class OverriderClassWithoutCodeDuplicated : BaseClassWithoutCodeDuplicated {
    public void SomeCode ()
    {
        PrintValuesInList ();
        list.Add (1);
    }
}
 
public class OtherOverriderWithoutCodeDuplicated : BaseClassWithoutCodeDuplicated {
    public void MoreCode ()
    {
        PrintValuesInList ();
        list.Remove (1);
    }
}
```

### AvoidLargeClassesRule

This rule allows developers to measure the classes size. When a class is trying to doing a lot of work, then you probabily have the Large Class smell. This rule will fire if a type contains too many fields (over 25 by default) or has fields with common prefixes. If the rule does fire then the type should be reviewed to see if new classes should be extracted from it.

**Bad** example:

``` csharp
public class MyClass {
    int x, x1, x2, x3;
    string s, s1, s2, s3;
    DateTime bar, bar1, bar2;
    string[] strings, strings1;
}
```

**Good** example:

``` csharp
public class MyClass {
    int x;
    string s;
    DateTime bar;
}
```

### AvoidLongMethodsRule

This rule allows developers to measure the method size. The short sized methods allows you to maintain your code better, if you have long sized methods perhaps you have the Long Method smell. The rule will skip some well known methods, because they are autogenerated:

-   **Gtk.Bin::Build ()**
-   **Gtk.Window::Build ()**
-   **Gtk.Dialog::Build ()**
-   **System.Windows.Forms::InitializeComponents ()**

This will work for classes that extend those types. If debugging symbols (e.g. Mono .mdb or MS .pdb) are available then the rule will compute the number of logical source line of code (SLOC). This number represent the lines where 'SequencePoint' are present in the code. By default the maximum SLOC is defined to 40 lines. Otherwise the rule falls back onto an IL-SLOC approximation. It's quite hard to determine how many SLOC exists based on the IL (e.g. LINQ). The metric being used is based on a screen (1024 x 768) full of source code where the number of IL instructions were counted. By default the maximum number of IL instructions is defined to be 165.

**Bad** example:

``` csharp
public void LongMethod ()
{
    Console.WriteLine ("I'm writting a test, and I will fill a screen with some useless code");
    IList list = new ArrayList ();
    list.Add ("Foo");
    list.Add (4);
    list.Add (6);
 
    IEnumerator listEnumerator = list.GetEnumerator ();
    while (listEnumerator.MoveNext ()) {
        Console.WriteLine (listEnumerator.Current);
    }
 
    try {
        list.Add ("Bar");
        list.Add ('a');
    }
    catch (NotSupportedException exception) {
        Console.WriteLine (exception.Message);
        Console.WriteLine (exception);
    }
 
    foreach (object value in list) {
        Console.Write (value);
        Console.Write (Environment.NewLine);
    }
 
    int x = 0;
    for (int i = 0; i < 100; i++) {
        x++;
    }
    Console.WriteLine (x);
 
    string useless = "Useless String";
    if (useless.Equals ("Other useless")) {
        useless = String.Empty;
        Console.WriteLine ("Other useless string");
    }
 
    useless = String.Concat (useless," 1");
    for (int j = 0; j < useless.Length; j++) {
        if (useless[j] == 'u') {
            Console.WriteLine ("I have detected an u char");
            } else {
                Console.WriteLine ("I have detected an useless char");
            }
        }
 
        try {
            foreach (string environmentVariable in Environment.GetEnvironmentVariables ().Keys) {
                Console.WriteLine (environmentVariable);
            }
        }
        catch (System.Security.SecurityException exception) {
            Console.WriteLine (exception.Message);
            Console.WriteLine (exception);
        }
 
        Console.WriteLine ("I will add more useless code !!");
        try {
            if (!(File.Exists ("foo.txt"))) {
                File.Create ("foo.txt");
                File.Delete ("foo.txt");
            }
        }
        catch (IOException exception) {
            Console.WriteLine (exception.Message);
            Console.WriteLine (exception);
        }
    }
```

**Good** example:

``` csharp
public void ShortMethod ()
{
    try {
        foreach (string environmentVariable in Environment.GetEnvironmentVariables ().Keys) {
            Console.WriteLine (environmentVariable);
        }
    }
    catch (System.Security.SecurityException exception) {
        Console.WriteLine (exception.Message);
        Console.WriteLine (exception);
    }
}
```

### AvoidLongParameterListsRule

This rule allows developers to measure the parameter list size in a method. If you have methods with a lot of parameters, perhaps you have a Long Parameter List smell. This rule counts the method's parameters, and compares it against a maximum value. If you have an overloaded method, then the rule will get the shortest overload and compare the shortest overload against the maximum value. Other time, it's quite hard determine a long parameter list. By default, a method with 6 or more arguments will be flagged as a defect.

**Bad** example:

``` csharp
public void MethodWithLongParameterList (int x, char c, object obj, bool j, string f,
float z, double u, short s, int v, string[] array)
{
    // Method body ...
}
```

**Good** example:

``` csharp
public void MethodWithoutLongParameterList (int x, object obj)
{
    // Method body....
}
```

### AvoidMessageChainsRule

This rule checks for the Message Chain smell. This can cause problems because it means that your code is heavily coupled to the navigation structure.

**Bad** example:

``` csharp
public void Method (Person person)
{
    person.GetPhone ().GetAreaCode ().GetCountry ().Language.ToFrench ("Hello world");
}
```

**Good** example:

``` csharp
public void Method (Language language)
{
    language.ToFrench ("Hello world");
}
```

### AvoidSpeculativeGeneralityRule

This rule allows developers to avoid the Speculative Generality smell. Be careful if you are developing a new framework or a new library, because this rule only inspects the assembly, then if you provide an abstract base class for extend by third party people, then the rule can warn you. You can ignore the message in this special case. We detect this smell by looking for:

-   Abstract classes without responsibility
-   Unnecessary delegation.
-   Unused parameters.

 **Bad** example:

``` csharp
// An abstract class with only one subclass.
public abstract class AbstractClass {
    public abstract void MakeStuff ();
}
 
public class OverriderClass : AbstractClass {
    public override void MakeStuff ()
    {
    }
}
```

If you use Telephone class only in one client, perhaps you don't need this kind of delegation.

``` csharp
public class Person {
    int age;
    string name;
    Telephone phone;
}
 
public class Telephone {
    int areaCode;
    int phone;
}
```

**Good** example:

``` csharp
public abstract class OtherAbstractClass{
    public abstract void MakeStuff ();
}
 
public class OtherOverriderClass : OtherAbstractClass {
    public override void MakeStuff ()
    {
    }
}
 
public class YetAnotherOverriderClass : OtherAbstractClass {
    public override void MakeStuff ()
    {
    }
}
```

### AvoidSwitchStatementsRule

This rule checks for the Switch Statements smell. This can lead to code duplication, because the same switch could be repeated in various places in your program. Also, if need to do a little change, you may have to change every switch statement. The preferred way to do this is with virtual methods and polymorphism.

**Bad** example:

``` csharp
int balance = 0;
foreach (Movie movie in movies) {
    switch (movie.GetTypeCode ()) {
        case MovieType.OldMovie: {
            balance += movie.DaysRented * movie.Price / 2;
            break;
        }
        case MovieType.ChildMovie: {
            //its an special bargain !!
            balance += movie.Price;
            break;
        }
        case MovieType.NewMovie: {
            balance += (movie.DaysRented + 1) * movie.Price;
            break:
        }
    }
}
```

**Good** example:

``` csharp
abstract class Movie {
    abstract int GetPrice ();
}
class OldMovie : Movie {
    public override int GetPrice ()
    {
        return DaysRented * Price / 2;
    }
}
class ChildMovie : Movie {
    public override int GetPrice ()
    {
        return movie.Price;
    }
}
class NewMovie : Movie {
    public override int GetPrice ()
    {
        return (DaysRented + 1) * Price;
    }
}
 
int balance = 0;
foreach (Movie movie in movies) {
    balance += movie.GetPrice ()
}
```

**Notes**

-   This rule is available since Gendarme 2.4

Feedback
========

Please report any documentation errors, typos or suggestions to the [Gendarme Google Group](http://groups.google.com/group/gendarme). Thanks!
