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
| `invokestatic` | Invocation | Invokes a static method from a class. |  |
| `invokevirtual` | Invocation | Invokes a method from a class using an instance of said class. |  |

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

`aload this` loads the instance of our current class, `invokevirtual` uses that instance to call our method. The rest is the same.
