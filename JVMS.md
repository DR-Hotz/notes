# Java Virtual Machine Specification

## Run-time data areas

	pc register per thread in jvm

	jvm stack per thread in jvm

	jvm heap is shared among all jvm threads

	method area is shared among all jvm threads

	A run-time constant pool(symbol table+) is a per-class or per-interface run-time representation of the constant_pool table in a class file; it is constructed when the class or interface is created by jvm

	Native method stack(C stack) supports native methods, it's typically allocated per thread when each thread is created.

## Frames

### Basics

	A new frame is created each time a method is invoked, and it's destroyed when its method invocation completes

	contains:
		its own array of local variables(compile-time determined)
		its own operand stack(compile-time determined)
		a reference to the runtime constant pool of the class of the current method

	A frame may be extended with additional implementation-specific information, such as debugging information.

	Thus the size of the frame data structure depends only on the implementation of the Java Virtual Machine,  the memory for these structures can be allocated simultaneously on method invocation

### Local Variables

	On instance method invocation, local variable 0 is always used to pass a reference to the object on which the instance method is being invoked (**this** in the Java programming language)

### Operand Stack(last-in-first-out (LIFO))

	The operand stack is empty when the frame that contains it is created

### Dynamic Linking


## Exception

	An exception in the Java Virtual Machine is represented by an instance of the class Throwable or one of its subclasses

	The Java Virtual Machine throws an exception for one of three reasons:
		1.	An athrow instruction was executed
		2.	An abnormal execution condition was synchronously detected by the JVM, these exceptions are not thrown at an arbitrary point in the program, but only synchronously after execution of an instruction 
		3.	An asynchronous exception occurred :
			1)	The stop method of class Thread or ThreadGroup was invoked(one thread to affect another thread or all the threads in a specified thread group, which could be occurred at any point)
			2)	An internal error occurred in the Java Virtual Machine implementation

	








