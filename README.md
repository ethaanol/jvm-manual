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
