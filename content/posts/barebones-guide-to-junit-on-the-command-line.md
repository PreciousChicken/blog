---
title: "Barebones Guide to JUnit on the Command Line"
date: 2020-02-15T22:19:39Z
tags: ["Java", "Linux"]
keywords: ["JUnit", "Testing"] 
categories: ["Unix workbench"]
description: "Using JUnit from the command line"
draft: false
---

### Introduction

[JUnit](https://junit.org) is a testing framework for Java.  It is primarily aimed for IDEs, but with some perseverance it can be used on the command line.

### ConsoleLauncher

You will need [ConsoleLauncher](https://junit.org/junit5/docs/current/user-guide/#running-tests-console-launcher), a java executable, to run JUnit from the command line.  At time of writing the latest version is `junit-platform-console-standalone-1.6.0.jar` and can be downloaded from the [Maven Repository](https://repo1.maven.org/maven2/org/junit/platform/junit-platform-console-standalone/1.6.0/).  Download this file to your working directory.

### Create a sample java file

Using your text editor of choice create a file called `SampleUnit.java` and copy and paste the following code in it:

```java
public class SampleUnit {
    
    public int addInts(int a, int b){
        return a + b;
    }

    public boolean validAboveZero(int a) {
	    if (a>0) {
		    return true;
	    }
	    return false;
    }
}
```

### Create a sample test file

Creating a file called `SampleUnitTest.java` copy the following:

```java
import static org.junit.Assert.assertFalse;
import static org.junit.Assert.assertTrue;
import static org.junit.Assert.assertEquals;
import org.junit.Test;

public class SampleUnitTest {

    @Test
    public void testAddInts() {
        SampleUnit sampleUnit = new SampleUnit();
        int result = sampleUnit.addInts(2, 2);
	// The below should fail.
        assertEquals("Not adding correctly!", 5, result);
    }

    @Test
    public void testValidAboveZero() {
        SampleUnit sampleUnit = new SampleUnit();
        boolean result = sampleUnit.validAboveZero(2);
        assertTrue(result);
    }

    @Test
    public void testValidFail() {
        boolean result = false;
        assertFalse(result);
    }
}
```

### Compile

Run the following from the command line:

```bash
javac SampleUnit.java
javac -cp junit-platform-console-standalone-1.6.0.jar:. SampleUnitTest.java
```

This assumes the JUnit jar, `SampleUnit.java` and `SampleUnitTest.java` are all in the same directory.  The compiler will create two new files: `SampleUnit.class` and `SampleUnitTest.class`.

### Run the tests

Finally at the command line run:

```bash
java -jar junit-platform-console-standalone-1.6.0.jar --class-path . -c SampleUnitTest
```

This should output something like this:

```bash
Thanks for using JUnit! Support its development at https://junit.org/sponsoring

╷
├─ JUnit Jupiter ✔
└─ JUnit Vintage ✔
   └─ SampleUnitTest ✔
      ├─ testAddInts ✘ Not adding correctly! expected:<5> but was:<4>
      ├─ testValidFail ✔
      └─ testValidAboveZero ✔

Failures (1):
  JUnit Vintage:SampleUnitTest:testAddInts
    MethodSource [className = 'SampleUnitTest', methodName = 'testAddInts', methodParameterTypes = '']
    => java.lang.AssertionError: Not adding correctly! expected:<5> but was:<4>
       org.junit.Assert.fail(Assert.java:89)
       org.junit.Assert.failNotEquals(Assert.java:835)
       org.junit.Assert.assertEquals(Assert.java:647)
       SampleUnitTest.testAddInts(SampleUnitTest.java:13)
       java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
       java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
       java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
       java.base/java.lang.reflect.Method.invoke(Method.java:566)
       org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:59)
       org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
       [...]

Test run finished after 43 ms
[         3 containers found      ]
[         0 containers skipped    ]
[         3 containers started    ]
[         0 containers aborted    ]
[         3 containers successful ]
[         0 containers failed     ]
[         3 tests found           ]
[         0 tests skipped         ]
[         3 tests started         ]
[         0 tests aborted         ]
[         2 tests successful      ]
[         1 tests failed          ]
```

### ConsoleLauncher Options Frustration

The ConsoleLauncher has an extensive list of command line [options](https://junit.org/junit5/docs/current/user-guide/#running-tests-console-launcher-options) for providing arguments.  Above I've gone for `--class-path . -c SampleUnitTest` defined respectively as: 

```bash
-cp, --classpath, --class-path=PATH[;|:PATH...] Provide additional classpath entries -- for example, for adding engines and their dependencies. This option can be repeated.
-c, --select-class=CLASS   Select a class for test discovery. This option can be repeated.
```

Why these two specifically? A simple reason - after a couple of hours reading [stack overflow](https://stackoverflow.com/questions/52373469/how-to-launch-junit-5-platform-from-the-command-line-without-maven-gradle/52373592#52373592) and trying various random combinations, it appears to be the only option I could get to work.  This may be due to my incompetence,  my setup (openjdk 11.0.6) or a bug.

