---
enableToc: true
title: "System Rules on the Command Line"
date: 2020-04-18T10:30:21+01:00
tags: ["Java", "Linux"]
keywords: ["JUnit", "Testing"] 
categories: ["Unix workbench"]
description: "Using JUnit from the command line"
draft: false
---

## Introduction

This is a more of a follow up to my previous post [Barebones Guide to JUnit on the Command Line](https://www.preciouschicken.com/blog/posts/barebones-guide-to-junit-on-the-command-line/) rather than an entirely new topic.  [System Rules](https://stefanbirkner.github.io/system-rules/index.html) is a "a collection of [JUnit](https://junit.org/junit5/) rules for testing code that uses java.lang.System" - in other words a way to use JUnit to test for cases where you want to use a `System.out.print` or similar.  The majority of the documentation is how to integrate it with an IDE, this is a quick reference on how to use it at the Linux command line.  At time of writing I'm using Ubuntu 18.04.4 LTS and OpenJDK 11.0.6.

## Download relevant Java Archive (JAR) files

You will need to place `junit-platform-console-standalone-1.6.0.jar` and `system-rules-1.19.0.jar` in your current working directory.  These can be downloaded from [JUnit Maven repo](https://repo1.maven.org/maven2/org/junit/platform/junit-platform-console-standalone/1.6.0/) and [Get System Rules](https://stefanbirkner.github.io/system-rules/download.html) respectively.  The repo at JUnit isn't particularly human readable (just a long list of files), so make sure you download the correct one.  There may be later versions by the time you read this.

## Create a sample java file

Using your text editor create a file called `SystemOut.java` and copy and paste the following code in it:

```java
public class SystemOut {
	public void PrintToSystem(boolean input) {
		if (input) {
			System.out.print("Hello from PrintToSystem.");
		}
	}
}
```

## Create a sample test file

Creating a file called `SystemOutTest.java` copy the following:

```java
import static org.junit.Assert.assertEquals;
import org.junit.contrib.java.lang.system.SystemOutRule;
import org.junit.Test;
import org.junit.Rule;

public class SystemOutTest {
	@Rule
	public SystemOutRule systemOutRule = new SystemOutRule().enableLog();

	@Test
	public void testPrintToSystem() {
		SystemOut systemOut = new SystemOut();
		systemOut.PrintToSystem(true);
		assertEquals("Hello from PrintToSystem.", systemOutRule.getLog());
	}
}
```

## Compile

Run the following from the command line:

```bash
javac SystemOut.java
javac -cp .:junit-platform-console-standalone-1.6.0.jar:system-rules-1.19.0.jar SystemOutTest.java
```

This assumes all files are in the same directory. The compiler will create two new class files.

## Run the tests

Finally at the command line run:

```bash
java -jar junit-platform-console-standalone-1.6.0.jar --class-path .:system-rules-1.19.0.jar -c SystemOutTest
```

This should output:

```bash
Hello from PrintToSystem.
Thanks for using JUnit! Support its development at https://junit.org/sponsoring

╷
├─ JUnit Jupiter ✔
└─ JUnit Vintage ✔
   └─ SystemOutTest ✔
      └─ testPrintToSystem ✔

Test run finished after 42 ms
[         3 containers found      ]
[         0 containers skipped    ]
[         3 containers started    ]
[         0 containers aborted    ]
[         3 containers successful ]
[         0 containers failed     ]
[         1 tests found           ]
[         0 tests skipped         ]
[         1 tests started         ]
[         0 tests aborted         ]
[         1 tests successful      ]
[         0 tests failed          ]
```

## Conclusion

We're done.  If this helped, or you have a pickup, please comment below.
