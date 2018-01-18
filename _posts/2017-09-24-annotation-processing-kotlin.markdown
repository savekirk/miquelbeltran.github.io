---
title: "Hello World of Annotation Processing in Kotlin"
layout: post
date: 2017-09-24 17:00
image: /assets/images/kotlin-android-extensions.png
headerImage: false
tag:
- kotlin
- android
category: blog
author: miquel
description: Simple annotation processing example with Kotlin
---

This is the result of some trial and error until I got a simple annotation processing example working, 
with the challenge of not using Java, but Kotlin instead. Hopefully will help you get started!

Sample code here: 
[https://github.com/miquelbeltran/kotlin-code-gen-sample](https://github.com/miquelbeltran/kotlin-code-gen-sample)

The goal is to generate a new class with the method getName() which will return the String “World”.

So if we annotate the class Hello, it will generate the class Generated_Hello, and it will contain the method getName() that returns “World”.

PS: You can jump directly into Building the processor for the good stuff.

<script src="https://gist.github.com/miquelbeltran/bf196cc941ec8a10b7702d80435b55d8.js"></script>

---

## Setup

First step is to setup your project. I went with a pure Kotlin project directly with IntelliJ IDEA.

In the root folder we won’t have any code, instead, create two modules inside the root folder, 
one that will contain our processor and another one that will contain our main code.

It’s important to do so, because you need to compile & process the processor before our main module!

On the root we will have two files: a build.gradle and a settings.gradle:

<script src="https://gist.github.com/miquelbeltran/01bbab4b9127eff4ecf7bedada49e87e.js"></script>

<script src="https://gist.github.com/miquelbeltran/3748e2a5b419beda98e52c1c95d9ac1f.js"></script>

We will have two modules: generator and sample-main

---

Let’s create the sample-main first:

- Create a Kotlin module with Gradle on IntelliJ
- Create a package folder (in my case work.beltran.sample)
- Add an empty Kotlin file Main.kt

Expect a structure similar to this one:


So far not much for a setup, we will go back to the Main.kt and the build.gradle later, let’s create the generator now.

---

Similarly, create a new module named generator with a package structure and add two Kotlin files:

- A class file called Generator
- An annotation called GenName

GenName is going to be our custom annotation. It will be a simple annotation without extra parameters. 
The way to create an annotation in Kotlin is by adding annotation in front of class.

<script src="https://gist.github.com/miquelbeltran/1b11a9584350f5582d55ec30c9945a53.js"></script>

We will look at the Generator later, now back to the main module.

---

## Main Module Code

On the Main.kt, create a class named Hello and annotate it with your new annotation.

<script src="https://gist.github.com/miquelbeltran/bf196cc941ec8a10b7702d80435b55d8.js"></script>

Secondly, generate a main method (you can just type main and hit Enter) and add a call to the Generated_Hello (which does not exist yet).

As you expect, the project will fail to resolve both GenName and Generated_Hello. So far it’s OK.

Now we will add the module dependency for GenName.

https://gist.github.com/miquelbeltran/c56905a7b142f2666412d7d3481c27e5#file-sample-main-build-gradle

On the main module build.gradle, you will have to add the dependency to the generator module, both as kapt and compileOnly.

There are two more tweaks you will need:

- Add generateStubs = true so we can refer to the generated classes (using Generated_Hello from our main method)
- Add the sourceSets pointing to the kotlinGenerated folder, so IntelliJ also sees the generated code (actually not required if you just build via command line gradle)

---

## Building the processor

Everything except our annotation processor is ready, let’s build it.

First, create a class that will extend AbstractProcessor:

```class Generator: AbstractProcessor()```

Our processor needs to implement the following methods:

- getSupportedAnnotationTypes: We tell the processor which annotations can we process. In our case just GenName.
- getSupportedSourceVersion: Which returns the source level supported version. I’ve found that leaving it as default won’t work and instead I need to return SourceVersion.latest.
- process: Here’s where the real work goes. What we need to do here is to actually generate the new source code and store it somewhere. In my example I use KotlinPoet to generate the class and then I use thekapt.kotlin.generated option to get the output folder.

<script src="https://gist.github.com/miquelbeltran/dafac1fafda8d20399173d0661e77372.js"></script>

Going step by step:

- On process, we will iterate over each annotated element, and we will use the class name and the package name to generate the resulting code. You can read more properties here to generate your code.
- On generateClass we are building the file and storing it. With KotlinPoet we create a new FileSpec, which includes a class with a function getName that returns “World”.
- Finally storing it into the kaptKotlinGeneratedDir.

Two important notes:

- You can print logs on the build process, which is heavily useful!
- As well, you can throw exceptions that will stop the build process. Also good to be sure no one is misusing the annotation.

One more thing: AutoService

In order to kapt to run our processor, we need to add a META-INF into our generator module indicating our processors. However AutoService from Google will generate this for us. Go for it and use it!

Also don’t forget, we need to add it to the generator build.gradle

<script src="https://gist.github.com/miquelbeltran/b81f2f630ca75b635655d02fb054d5c2.js"></script>

At the time of this article, 1.0-rc3 did not work, so I had to step back to 1.0-rc2.

Let’s build and check our generated file:

<script src="https://gist.github.com/miquelbeltran/d631bde38ccbd654e65a21e01a712f3c.js"></script>

If everything is alright, now you can run the main method, and you will see Hello World printed on the terminal.

```
Hello World
Process finished with exit code 0
```

## Summary

- Create two modules: one for your generator and one for your main code
- The tricky part is getting the gradle configuration right for both modules, pointing to the generated source files, generating stubs and the kapt dependency
- Don’t forget to generate or include the META-INF. That was one of my first mistakes. Nothing will be processed if it is missing
- In case of doubt, use logs/exceptions on your processor

Sample code here: [https://github.com/miquelbeltran/kotlin-code-gen-sample](https://github.com/miquelbeltran/kotlin-code-gen-sample)

##References

- KotlinPoet: https://github.com/square/kotlinpoet
- Here I found how to store the new Kotlin file: https://github.com/square/kotlinpoet/issues/105
- and https://github.com/JetBrains/kotlin-examples/blob/master/gradle/kotlin-code-generation/annotation-processor/src/main/java/TestAnnotationProcessor.kt
- Google’s AutoService: https://github.com/google/auto/tree/master/service
