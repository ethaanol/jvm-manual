<h1 align="center">The JVM Manual</h1>

# Descriptors

The JVM has simple single-character names for each primitive type.

| Type | Descriptor |
| ---- | ---- |
| `long` | `J` |
| `int` | `I` |
| `short` | `S` |
| `byte` | `B` |
| `boolean` | `Z` |
| `float` | `F` |
| `double` | `D` |
| `void` | `V` 

#### Method descriptors
In the JVM descriptors are used to describe both methods and fields.
You can identify if a descriptor is a method descriptor if it for example contains parentheses.
If not it is a field descriptor.

Here is a simple method:

```java
int add(int a, int b) {
   return a + b;
}
```

The descriptor for this method would be:

`(II)I`

Even if a method has no arguments it will still contain parentheses like:

`()V` is an example of a void method that would look like this:

```java
void doSomething() {}
```


Inside of the parentheses we see two integer primitive types which in our source file are:

```java
(int a, int b)
```

The return type is specified at the ending of the descriptor which we can see is an integer primitive.

Here is a more complex method:

```java
String concat(String a, String b) {
   return a + b;
}
```

Now we've only seen primitive types yet, but what about parameters that aren't primitives like:

For example `java.lang.String` is obviously a class, so what would be the descriptor for the `concat()` method?

`(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;` is the descriptor.

Might seem scary but it isn't. In the JVM classes in descriptors need to have a prefix and a suffix,
the prefix being `L` and the suffix being `;` let's say you have a class in:

`me.myname.MyClass`

If you wanted to use it in a descriptor, it would be:

`Lme/myname/Myclass;`

#### Field descriptors
Field descriptors are the same as method descriptors just that they don't have parentheses because they aren't methods and obviously don't need arguments.

Here is an example field:

```java
String food = "Pancakes!"
```

The descriptor for it would simply be the class using the same prefix and suffix convention while containing **NO PARENTHESES**:

`Ljava/lang/String;`

# Core instructions

The JVM being well... a VM has bytecode which for simplicity I'll just call instructions.
Let's go over a few of the most important ones.

| Opcode | Type | Purpose | Frequency |
| ---- | ---- | ---- | ---- |
| `iload` | General | Loads a local variable of type primitive integer that was previously stored onto the stack. | High |
| `istore` | General | Stores a primitive integer value into a local variable that can later be loaded onto the stack using it's index. | High |
| `goto` | Flow | Jumps to a specified label. | Medium |
| `ifeq,ne` | Flow | Jumps to a specified label based on a condition. | Medium |
| `dup` | General | Duplicates the latest stack element. | Medium |
| `pop` | General | Pops the latest stack element off the stack. | High |
| `ireturn` | General | Returns an integer primitive from a method. | High |
| `iadd` | Arithmetic | Adds the latest 2 integer primitive type stack elements together and pushes the result. | Low |
| `isub` | Arithmetic | Subtracts the latest 2 integer primitive type stack elements and pushes the result. | Low |
| `ixor` | Bitwise | Performs an exclusive OR operation on the 2 latest integer primitive type stack elements. | Medium |
| `invokestatic` | Invocation | Invokes a static method from a class. | High |
| `invokevirtual` | Invocation | Invokes a method from a class using an instance of said class. | High |

# Invocation

In Java source we obviously invoke methods but how does that happen under the hood?
I will talk about 2 invocation instructions which are `invokestatic` and `invokevirtual` because explaining something like `invokedynamic` is not what I would like to do and it's also a bit harder to wrap your head around, either way you won't really see them as much.

### invokestatic

The syntax of the `invokestatic` instruction is like this:

```
invokestatic (Class name).(Method name) (Method descriptor)
```

Here is some code that invokes our previously seen `add()` method:

```java
public static int add(int a, int b) {
   return a + b;
}

public static void main(String[] args) {
   int result = add(5, 5);
}
```

Here are the instructions (bytecode) for our entry point:

```
l_entry:
  iconst_5
  iconst_5
  invokestatic Main.add (II)I
  istore_1
```

We can see iconst_5 is written twice meaning it loads 2 values (5, 5) onto the stack, then we can see those 2 values are used in the invocation to our add method which returns the result that we then store in a local variable.


### invokevirtual

The main kicker with `invokevirtual` is that it's used to invoke instance methods, meaning methods that aren't static.

Let's say we have this similar code block we had for `invokestatic` except that the `add()` method now does not have a `static` access modifier:

```java
public int add(int a, int b) {
   return a + b;
}

public void entry() {
   int result = add(5, 5);
}
```

Now for the cool part (the bytecode of `entry()`):

```
l_entry:
  aload this
  iconst_5
  iconst_5
  invokevirtual Main.add (II)I
  istore_1
```

We can now see that there is an additional instruction (`aload this`) now it's kind of weird because it gets resolved, in reality it can also be aload_0 but they mean the same thing.

`aload this` loads the instance of our current class, `invokevirtual` uses that instance to call our method. The rest is the same, in a few moments we will jump into some actual reverse engineering techniques. 

# Virtualization

The process of virtualization is very simple, we basically make a VM (virtual machine) that can interpret the JVM bytecode, this gives us a lot more power in terms of what information we can get.

A popular VM is SSVM by xxDark which you can find <a href="https://github.com/xxDark/SSVM">here<a>, it is very powerful and can be used for numerous things.
The reason we need a VM is so we can virtualize stuff like method calls.

Let's say we have this obfuscated piece of code:

```java
v7 = v5.void();
v14 = Main.import(v12, 556878214);
v17 = Main.boolean(v7, v14);
var1_1.setTitle(v17);
```

Now I mean this in the most respectful way possible, this is fucking dogshit but I can't really just introduce you into insanely obfuscated code, that will probably give you brain damage.

So as you could probably already figure out `setTitle()` takes a string.
However we don't see that string, it's a local variable that we can only get at runtime.
This is one of the situations where using virtualization is a good idea.
Now of course the naming here violates every single convention of Java but that's the whole point.

Now I won't put the actual implementation of `boolean()` as it is obviously massive.
Although I can see something very interesting:

```java
v4 = new throw();
v7 = v4.throw();
v9 = Main.import(v7, -821449344);
var2_2 = Cipher.getInstance(v9);
```

`import()`'s descriptor is:

```java
(Ljava/lang/String;I)Ljava/lang/String;
```

or simplified:

```java
String (String str, int num)
```

now this is a classic pattern of XOR encryption, and this is backed up by the fact that we see this in the `import()` method:

```java
v5 = (char)(s.toCharArray()[i] ^ num);
```

If you don't know, in Java the `^` character means the XOR operation.
Now since I have some experience I can already understand what this whole method does. 
in the descriptor (let's use the simplified one for simplicity) we can see that a String and an integer primitive are passed in, basically this means the string that is passed in is the data that needs to be encrypted and the integer is obviously the key.

So armed with this knowledge we can go back to the `boolean()` method:

```java
v4 = new throw();
v7 = v4.throw();
v9 = Main.import(v7, -821449344);
var2_2 = Cipher.getInstance(v9);
```

If you have any knowledge of Java then you know what `getInstance()` does.
Now `v9` in this case is a string meaning we can decrypt it by virtualizing the method call.
Now let's virtualize this piece of code:

```java
Main.import(v7, -821449344);
```

after virtualizing this using SSVM, the output is:

`AES/ECB/PKCS5Padding`

which means the code is:

```java
var2_2 = Cipher.getInstance("AES/ECB/PKCS5Padding");
```

which is about right.
Anyways now you know how to virtualize methods! This example was a bit off but you should be able to understand it.
# Hooking

Hooking is a core concept in reverse engineering.
I will only talk about Java 8 hooking here as Java 17 is a little different and uses the `.jmod` file format.

Classes that are part of Java's implementation (e.g. `java.lang.System`) in Java 8 are inside of a file called `rt.jar` which is located in your Java installation path, in my case it is in:

`C:/Program Files/Zulu/zulu-8/jre/lib`

it should be like that for every Java installation, of course if you use a Unix operating system the filesystem will be different but the actual path to the `rt.jar` will stay the same.

### General hooking (constructors, methods)

So let's say you found a piece of code that does something like this:

```java
/*
In a real world scenario this would likely be obfuscated which is why you need to hook.
*/
String information = "Super secret information";
JavaClass instance = new JavaClass(information);
```

in this case `JavaClass` is a general class from the Java implementation and could be anything.
Now what you will have to do to extract what the information string actually means is to hook the constructor of `JavaClass`, let's say the constructor of it looks like this:

```java
public JavaClass(String information) {
   // we perform something with the 'information' variable.
}
```

To hook this constructor what we have to do is in some way log the information that would be passed through the constructor at runtime, my usual go-to is the `println()` method:

```java
public JavaClass(String information) {
   System.out.println("extracted information: " + information);
   // we do something with the 'information' variable.
}
```

and there you go, now once we run this we will get whatever goes through this constructor printed right on our console.

The approach stays the same for methods, let's say we have a piece of code like this:

```java
String information = "Super secret information";
JavaClass.processInfo(information);
```

This is a static call meaning it uses `invokestatic` in bytecode which we learned about previously, although it does not matter if the method is invoked through an instance or statically.

Let's say that `processInfo()` looks something like this:

```java
public static processInfo(String information) {
   // we do something with the 'information' variable.
}
```

now the approach stays the same, all we have to do is add our `println()` call to the method's start.

```java
public static processInfo(String information) {
   System.out.println(information);
   // we do something with the 'information' variable.
}
```

and that is it! Now you know how to hook methods and constructors to print out any valuable information that is passed through them, although this isn't the only use for hooking, hooking can be used in many scenarios including changing code of a method or constructor to do more than it was originally intended to, or possibly less, whatever the case is hooking is a great technique to know about.
