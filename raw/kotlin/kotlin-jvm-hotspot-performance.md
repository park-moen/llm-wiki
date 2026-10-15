# 5 Java HotSpot Virtual Machine Performance Enhancements

> Source: https://docs.oracle.com/en/java/javase/21/vm/java-hotspot-virtual-machine-performance-enhancements.html
> Collected: 2026-10-15
> Published: Unknown
> Scope: Java SE 21 chapter body; navigation omitted.
> Collection note: JVM/Java SE 21 or HotSpot-specific source. Do not generalize its runtime details to Kotlin/JS or Kotlin/Native, or confuse the abstract VM model with an exact allocation guarantee.

## 5 Java HotSpot Virtual Machine Performance Enhancements

This chapter describes the performance enhancements in the Oracle’s HotSpot Virtual Machine technology.

Topics:

- [Compact Strings](https://docs.oracle.com/en/java/javase/21/vm/java-hotspot-virtual-machine-performance-enhancements.html#GUID-D2E3DC58-D18B-4A6C-8167-4A1DFB4888E4 "The compact strings feature introduces a space-efficient internal representation for strings.")
- [Tiered Compilation](https://docs.oracle.com/en/java/javase/21/vm/java-hotspot-virtual-machine-performance-enhancements.html#GUID-85BA7DE7-4AF9-47D9-BFCF-379230C66412)
- [Compressed Ordinary Object Pointer](https://docs.oracle.com/en/java/javase/21/vm/java-hotspot-virtual-machine-performance-enhancements.html#GUID-932AD393-1C8C-4E50-8074-F81AD6FB2444)
- [Zero-Based Compressed Ordinary Object Pointers](https://docs.oracle.com/en/java/javase/21/vm/java-hotspot-virtual-machine-performance-enhancements.html#GUID-E9E4603D-2B6B-44CA-A87E-FB14F0721745)
- [Escape Analysis](https://docs.oracle.com/en/java/javase/21/vm/java-hotspot-virtual-machine-performance-enhancements.html#GUID-6BD8FCB5-995B-4AE9-BFAA-B2C7DE2BA5CD)

### Compact Strings

The compact strings feature introduces a space-efficient internal representation for strings.

Data from different applications suggests that strings are a major component of Java heap usage and that most `java.lang.String` objects contain only Latin-1 characters. Such characters require only one byte of storage. As a result, half of the space in the internal character arrays of `java.lang.String` objects are not used. The compact strings feature, introduced in Java SE 9 reduces the memory footprint, and reduces garbage collection activity. This feature can be disabled if you observe performance regression issues in an application.

The compact strings feature does not introduce new public APIs or interfaces. It modifies the internal representation of the `java.lang.String` class from a UTF-16 (two bytes) character array to a byte array with an additional field to identify character encoding. Other string-related classes, such as `AbstractStringBuilder`, `StringBuilder`, and `StringBuffer` are updated to use a similar internal representation.

In Java SE 9, the compact strings feature is enabled by default. Therefore, the `java.lang.String` class stores characters as one byte for each character, encoded as Latin-1. The additional character encoding field indicates the encoding that is used. The HotSpot VM string intrinsics are updated and optimized to support the internal representation.

You can disable the compact strings feature by using the `-XX:-CompactStrings` flag with the `java` command line. When the feature is disabled, the `java.lang.String` class stores characters as two bytes, encoded as UTF-16, and the HotSpot VM string intrinsics to use UTF-16 encoding.

### Tiered Compilation

Tiered compilation, introduced in Java SE 7, brings client VM startup speeds to the server VM. Without tired compilation, a server VM uses the interpreter to collect profiling information about methods that is sent to the compiler. With tiered compilation, the server VM also uses the client compiler to generate compiled versions of methods that collect profiling information about themselves. The compiled code is substantially faster than the interpreter, and the program executes with greater performance during the profiling phase. Often, startup is faster than the client VM startup speed because the final code produced by the server compiler might be available during the early stages of application initialization. Tiered compilation can also achieve better peak performance than a regular server VM, because, the faster profiling phase allows a longer period of profiling, which can yield better optimization.

Tiered compilation is enabled by default for the server VM. The 64-bit mode and [Compressed Ordinary Object Pointer](https://docs.oracle.com/en/java/javase/21/vm/java-hotspot-virtual-machine-performance-enhancements.html#GUID-932AD393-1C8C-4E50-8074-F81AD6FB2444) are supported. You can disable tiered compilation by using the `-XX:-TieredCompilation` flag with the `java` command.

To accommodate the additional profiling code that is generated with tiered compilation, the default size of code cache is multiplied by 5x. To organize and manage the larger space effectively, [segmented code cache](https://docs.oracle.com/en/java/javase/21/vm/java-hotspot-virtual-machine-performance-enhancements.html#GUID-1D9B26AD-8E0A-4771-90DA-A81A2C1F5B55) is used.

#### Segmented Code Cache

The code cache is the area of memory where the Java Virtual Machine stores generated native code. It is organized as a single heap data structure on top of a contiguous chunk of memory.

Instead of having a single code heap, the code cache is divided into segments, each containing compiled code of a particular type. This segmentation provides better control of the JVM memory footprint, shortens scanning time of compiled methods, significantly decreases the fragmentation of code cache, and improves performance.

The code cache is divided into the following three segments:

Table 5-1 Segmented Code Cache

| Code Cache Segments | Description | JVM Command-Line Arguments |
| --- | --- | --- |
| Non-method | This code heap contains non-method code such as compiler buffers and bytecode interpreter. This code type stays in the code cache forever. The code heap has a fixed size of 3 MB and remaining code cache is distributed evenly among the profiled and non-profiled code heaps. | `-XX:NonMethodCodeHeapSize` |
| Profiled | This code heap contains lightly optimized, profiled methods with a short lifetime. | `–XX:ProfiledCodeHeapSize` |
| Non-profiled | This code heap contains fully optimized, non-profiled methods with a potentially long lifetime. | `-XX:NonProfiledCodeHeapSize` |

### Compressed Ordinary Object Pointer

An ordinary object pointer (oop) in Java Hotspot parlance, is a managed pointer to an object. Typically, an oop is the same size as a native machine pointer, which is 64-bit on an LP64 system. On an ILP32 system, maximum heap size is less than 4 gigabytes, which is insufficient for many applications. On an LP64 system, the heap used by a given program might have to be around 1.5 times larger than when it is run on an ILP32 system. This requirement is due to the expanded size of managed pointers. Memory is inexpensive, but these days bandwidth and cache are in short supply, so significantly increasing the size of the heap and only getting just over the 4 gigabyte limit is undesirable.

Managed pointers in the Java heap point to objects that are aligned on 8-byte address boundaries. Compressed oops represent managed pointers (in many but not all places in the Java Virtual Machine (JVM) software) as 32-bit object offsets from the 64-bit Java heap base address. Because they're object offsets rather than byte offsets, oops can be used to address up to four billion objects (not bytes), or a heap size of up to about 32 gigabytes. To use them, they must be scaled by a factor of 8 and added to the Java heap base address to find the object to which they refer. Object sizes using compressed oops are comparable to those in ILP32 mode.

The term decode refer to the operation by which a 32-bit compressed oop is converted to a 64-bit native address and added into the managed heap. The term encode refers to that inverse operation.

Compressed oops is supported and enabled by default in Java SE 6u23 and later. In Java SE 7, compressed oops is enabled by default for 64-bit JVM processes when `-Xmx` isn't specified and for values of `-Xmx` less than 32 gigabytes. For JDK releases earlier than 6u23 release, use the `-XX:+UseCompressedOops` flag with the `java` command to enable the compressed oops.

### Zero-Based Compressed Ordinary Object Pointers

When the JVM uses compressed ordinary object pointers (oops) in a 64-bit JVM process, the JVM software sends a request to the operating system to reserve memory for the Java heap starting at virtual address zero. If the operating system supports such a request and can reserve memory for the Java heap at virtual address zero, then zero-based compressed oops are used.

When zero-based compressed oops are used, a 64-bit pointer can be decoded from a 32-bit object offset without including the Java heap base address. For heap sizes less than 4 gigabytes, the JVM software can use a byte offset instead of an object offset and thus also avoid scaling the offset by 8. Encoding a 64-bit address into a 32-bit offset is correspondingly efficient.

For Java heap sizes up to 26 gigabytes, the Linux and Windows operating systems
typically can allocate the Java heap at virtual address zero.

### Escape Analysis

Escape analysis is a technique by which the Java HotSpot Server Compiler can analyze the scope of a new object's uses and decide whether to allocate the object on the Java heap.

Escape analysis is supported and enabled by default in Java SE 6u23 and later.

The Java HotSpot Server Compiler implements the flow-insensitive escape analysis algorithm described in:

```
 [Choi99] Jong-Deok Choi, Manish Gupta, Mauricio Seffano,
          Vugranam C. Sreedhar, Sam Midkiff,
          "Escape Analysis for Java", Procedings of ACM SIGPLAN
          OOPSLA  Conference, November 1, 1999
```

An object's escape state, based on escape analysis, can be one of the following states:

- `GlobalEscape`: The object escapes the method and thread. For example, an object stored in a static field, stored in a field of an escaped object, or returned as the result of the current method.
- `ArgEscape`: The object is passed as an argument or referenced by an argument but does not globally escape during a call. This state is determined by analyzing the bytecode of the called method.
- `NoEscape`: The object is a scalar replaceable object, which means that its allocation could be removed from generated code.

After escape analysis, the server compiler eliminates the scalar replaceable object allocations and the associated locks from generated code. The server compiler also eliminates locks for objects that do not globally escape. It does not replace a heap allocation with a stack allocation for objects that do not globally escape.

The following examples describe some scenarios for escape analysis:

- The server compiler might eliminate certain object allocations. For example, a method makes a defensive copy of an object and returns the copy to the caller.

  ```
  public class Person {
    private String name;
    private int age;
    public Person(String personName, int personAge) {
      name = personName;
                  age = personAge;
    }
          
    public Person(Person p) { this(p.getName(), p.getAge()); }
    public int getName() { return name; }
    public int getAge() { return age; }
  }

  public class Employee {
    private Person person;
    
          // makes a defensive copy to protect against modifications by caller
          public Person getPerson() { return new Person(person) };
          
          public void printEmployeeDetail(Employee emp) {
            Person person = emp.getPerson();
            // this caller does not modify the object, so defensive copy was unnecessary
                  System.out.println ("Employee's name: " + person.getName() + "; age: "  + person.getAge());     
          }
  }
  ```

  The method makes a copy to prevent modification of the original object by the caller. If the compiler determines that the `getPerson` method is being invoked in a loop, then the compiler inlines that method. By using escape analysis, when the compiler determines that the original object is never modified, the compiler can optimize and eliminate the call to make a copy.
- The server compiler might eliminate synchronization blocks (lock elision) if it determines that an object is thread local. For example, methods of classes such as `StringBuffer` and `Vector` are synchronized because they can be accessed by different threads. However, in most scenarios, they are used in a thread local manner. In cases where the usage is thread local, the compiler can optimize and remove the synchronization blocks.
