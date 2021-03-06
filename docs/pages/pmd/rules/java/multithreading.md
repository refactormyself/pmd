---
title: Multithreading
summary: Rules that flag issues when dealing with multiple threads of execution.
permalink: pmd_rules_java_multithreading.html
folder: pmd/rules/java
sidebaractiveurl: /pmd_rules_java.html
editmepath: ../pmd-java/src/main/resources/category/java/multithreading.xml
keywords: Multithreading, AvoidSynchronizedAtMethodLevel, AvoidThreadGroup, AvoidUsingVolatile, DoNotUseThreads, DontCallThreadRun, DoubleCheckedLocking, NonThreadSafeSingleton, UnsynchronizedStaticDateFormatter, UseConcurrentHashMap, UseNotifyAllInsteadOfNotify
language: Java
---
## AvoidSynchronizedAtMethodLevel

**Since:** PMD 3.0

**Priority:** Medium (3)

Method-level synchronization can cause problems when new code is added to the method.
Block-level synchronization helps to ensure that only the code that needs synchronization
gets it.

**This rule is defined by the following XPath expression:**
```
//MethodDeclaration[@Synchronized='true']
```

**Example(s):**

``` java
public class Foo {
  // Try to avoid this:
  synchronized void foo() {
  }
  // Prefer this:
  void bar() {
    synchronized(this) {
    }
  }

  // Try to avoid this for static methods:
  static synchronized void fooStatic() {
  }

  // Prefer this:
  static void barStatic() {
    synchronized(Foo.class) {
    }
  }
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/multithreading.xml/AvoidSynchronizedAtMethodLevel" />
```

## AvoidThreadGroup

**Since:** PMD 3.6

**Priority:** Medium (3)

Avoid using java.lang.ThreadGroup; although it is intended to be used in a threaded environment
it contains methods that are not thread-safe.

**This rule is defined by the following XPath expression:**
```
//AllocationExpression/ClassOrInterfaceType[pmd-java:typeof(@Image, 'java.lang.ThreadGroup')]|
//PrimarySuffix[contains(@Image, 'getThreadGroup')]
```

**Example(s):**

``` java
public class Bar {
    void buz() {
        ThreadGroup tg = new ThreadGroup("My threadgroup");
        tg = new ThreadGroup(tg, "my thread group");
        tg = Thread.currentThread().getThreadGroup();
        tg = System.getSecurityManager().getThreadGroup();
    }
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/multithreading.xml/AvoidThreadGroup" />
```

## AvoidUsingVolatile

**Since:** PMD 4.1

**Priority:** Medium High (2)

Use of the keyword 'volatile' is generally used to fine tune a Java application, and therefore, requires
a good expertise of the Java Memory Model. Moreover, its range of action is somewhat misknown. Therefore,
the volatile keyword should not be used for maintenance purpose and portability.

**This rule is defined by the following XPath expression:**
```
//FieldDeclaration[contains(@Volatile,'true')]
```

**Example(s):**

``` java
public class ThrDeux {
  private volatile String var1; // not suggested
  private          String var2; // preferred
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/multithreading.xml/AvoidUsingVolatile" />
```

## DoNotUseThreads

**Since:** PMD 4.1

**Priority:** Medium (3)

The J2EE specification explicitly forbids the use of threads.

**This rule is defined by the following XPath expression:**
```
//ClassOrInterfaceType[@Image = 'Thread' or @Image = 'Runnable']
```

**Example(s):**

``` java
// This is not allowed
public class UsingThread extends Thread {

}

// Neither this,
public class OtherThread implements Runnable {
    // Nor this ...
    public void methode() {
        Runnable thread = new Thread(); thread.run();
    }
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/multithreading.xml/DoNotUseThreads" />
```

## DontCallThreadRun

**Since:** PMD 4.3

**Priority:** Medium Low (4)

Explicitly calling Thread.run() method will execute in the caller's thread of control.  Instead, call Thread.start() for the intended behavior.

**This rule is defined by the following XPath expression:**
```
//StatementExpression/PrimaryExpression
[
    PrimaryPrefix
    [
        ./Name[ends-with(@Image, '.run') or @Image = 'run']
        and substring-before(Name/@Image, '.') =//VariableDeclarator/VariableDeclaratorId/@Image
            [../../../Type/ReferenceType/ClassOrInterfaceType[typeof(@Image, 'java.lang.Thread', 'Thread')]]
        or (./AllocationExpression/ClassOrInterfaceType[typeof(@Image, 'java.lang.Thread', 'Thread')]
        and ../PrimarySuffix[@Image = 'run'])
    ]
]
```

**Example(s):**

``` java
Thread t = new Thread();
t.run();            // use t.start() instead
new Thread().run(); // same violation
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/multithreading.xml/DontCallThreadRun" />
```

## DoubleCheckedLocking

**Since:** PMD 1.04

**Priority:** High (1)

Partially created objects can be returned by the Double Checked Locking pattern when used in Java.
An optimizing JRE may assign a reference to the baz variable before it calls the constructor of the object the
reference points to.

Note: With Java 5, you can make Double checked locking work, if you declare the variable to be `volatile`.

For more details refer to: <http://www.javaworld.com/javaworld/jw-02-2001/jw-0209-double.html>
or <http://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html>

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.multithreading.DoubleCheckedLockingRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/multithreading/DoubleCheckedLockingRule.java)

**Example(s):**

``` java
public class Foo {
    /*volatile */ Object baz = null; // fix for Java5 and later: volatile
    Object bar() {
        if (baz == null) { // baz may be non-null yet not fully created
            synchronized(this) {
                if (baz == null) {
                    baz = new Object();
                }
              }
        }
        return baz;
    }
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/multithreading.xml/DoubleCheckedLocking" />
```

## NonThreadSafeSingleton

**Since:** PMD 3.4

**Priority:** Medium (3)

Non-thread safe singletons can result in bad state changes. Eliminate
static singletons if possible by instantiating the object directly. Static
singletons are usually not needed as only a single instance exists anyway.
Other possible fixes are to synchronize the entire method or to use an
[initialize-on-demand holder class](https://en.wikipedia.org/wiki/Initialization-on-demand_holder_idiom).

Refrain from using the double-checked locking pattern. The Java Memory Model doesn't
guarantee it to work unless the variable is declared as `volatile`, adding an uneeded
performance penalty. [Reference](http://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html)

See Effective Java, item 48.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.multithreading.NonThreadSafeSingletonRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/multithreading/NonThreadSafeSingletonRule.java)

**Example(s):**

``` java
private static Foo foo = null;

//multiple simultaneous callers may see partially initialized objects
public static Foo getFoo() {
    if (foo==null) {
        foo = new Foo();
    }
    return foo;
}
```

**This rule has the following properties:**

|Name|Default Value|Description|Multivalued|
|----|-------------|-----------|-----------|
|checkNonStaticFields|false|Check for non-static fields.  Do not set this to true and checkNonStaticMethods to false.|no|
|checkNonStaticMethods|true|Check for non-static methods.  Do not set this to false and checkNonStaticFields to true.|no|

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/multithreading.xml/NonThreadSafeSingleton" />
```

## UnsynchronizedStaticDateFormatter

**Since:** PMD 3.6

**Priority:** Medium (3)

SimpleDateFormat instances are not synchronized. Sun recommends using separate format instances
for each thread. If multiple threads must access a static formatter, the formatter must be
synchronized either on method or block level.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.multithreading.UnsynchronizedStaticDateFormatterRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/multithreading/UnsynchronizedStaticDateFormatterRule.java)

**Example(s):**

``` java
public class Foo {
    private static final SimpleDateFormat sdf = new SimpleDateFormat();
    void bar() {
        sdf.format(); // poor, no thread-safety
    }
    synchronized void foo() {
        sdf.format(); // preferred
    }
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/multithreading.xml/UnsynchronizedStaticDateFormatter" />
```

## UseConcurrentHashMap

**Since:** PMD 4.2.6

**Priority:** Medium (3)

**Minimum Language Version:** Java 1.5

Since Java5 brought a new implementation of the Map designed for multi-threaded access, you can
perform efficient map reads without blocking other threads.

**This rule is defined by the following XPath expression:**
```
//Type[../VariableDeclarator/VariableInitializer//AllocationExpression/ClassOrInterfaceType[@Image != 'ConcurrentHashMap']]
/ReferenceType/ClassOrInterfaceType[@Image = 'Map']
```

**Example(s):**

``` java
public class ConcurrentApp {
  public void getMyInstance() {
    Map map1 = new HashMap();           // fine for single-threaded access
    Map map2 = new ConcurrentHashMap(); // preferred for use with multiple threads

    // the following case will be ignored by this rule
    Map map3 = someModule.methodThatReturnMap(); // might be OK, if the returned map is already thread-safe
  }
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/multithreading.xml/UseConcurrentHashMap" />
```

## UseNotifyAllInsteadOfNotify

**Since:** PMD 3.0

**Priority:** Medium (3)

Thread.notify() awakens a thread monitoring the object. If more than one thread is monitoring, then only
one is chosen.  The thread chosen is arbitrary; thus its usually safer to call notifyAll() instead.

**This rule is defined by the following XPath expression:**
```
//StatementExpression/PrimaryExpression
[PrimarySuffix/Arguments[@ArgumentCount = '0']]
[
    PrimaryPrefix[
        ./Name[@Image='notify' or ends-with(@Image,'.notify')]
        or ../PrimarySuffix/@Image='notify'
        or (./AllocationExpression and ../PrimarySuffix[@Image='notify'])
    ]
]
```

**Example(s):**

``` java
void bar() {
    x.notify();
    // If many threads are monitoring x, only one (and you won't know which) will be notified.
    // use instead:
    x.notifyAll();
  }
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/multithreading.xml/UseNotifyAllInsteadOfNotify" />
```

