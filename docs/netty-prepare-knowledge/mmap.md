# mmap 内存映射

MappedByteBuffer 非堆内内存， 通过unsafe访问直接内存。这些数据避免了copy到堆内的开销。

```java
/**
 * 
 * <p> For most operating systems, mapping a file into memory is more
 * expensive than reading or writing a few tens of kilobytes of data via
 * the usual {@link #read read} and {@link #write write} methods.  From the
 * standpoint of performance it is generally only worth mapping relatively
 * large files into memory.  </p>
 * Using thie api when file segments bigger than 100KB. Eg, when transform big size file
 */
public abstract MappedByteBuffer map(MapMode mode, long position, long size)throws IOException;
```

## 堆外内存

使用堆外内存与对象池都能减少GC的暂停时间，这是它们唯一的共同点。生命周期短的可变对象，创建开销大，或者生命周期虽长但存在冗余的可变对象都比较适合使用对象池。生命周期适中，或者复杂的对象则比较适合由GC来进行处理。然而，中长生命周期的可变对象就比较棘手了，堆外内存则正是它们的菜。
ehcache、memcache中都有堆外内存的使用。

### ehchche
    1. 堆内存储：速度快，但是容量有限。
    2. 堆外（OffHeapStore）存储：被称为BigMemory，只在企业版本的Ehcache中提供，原理是利用nio的DirectByteBuffers实现，比存储到磁盘上快，而且完全不受GC的影响，可以保证响应时间的稳定性；但是direct buffer的在分配上的开销要比heap buffer大，而且要求必须以字节数组方式存储，因此对象必须在存储过程中进行序列化，读取则进行反序列化操作，它的速度大约比堆内存储慢一个数量级。

（注：direct buffer不受GC影响，**但是direct buffer归属的的JAVA对象是在堆上且能够被GC回收的**，一旦它被回收，JVM将释放direct buffer的堆外空间。）”

## 堆外内存的优点和缺点

堆外内存，其实就是不受JVM控制的内存。相比于堆内内存有几个优势： 
　　1. 减少了垃圾回收的工作，因为垃圾回收会暂停其他的工作（可能使用多线程或者时间片的方式，根本感觉不到） 。
　　2. 加快了复制的速度。因为堆内在flush到远程时，会先复制到直接内存（非堆内存），然后在发送；而堆外内存相当于省略掉了这个工作。 
　　3. 可以在进程间共享，减少JVM间的对象复制，使得JVM的分割部署更容易实现。
　　4. 可以扩展至更大的内存空间。比如超过1TB甚至比主存还大的空间。

　而福之祸所依，自然也有不好的一面： 
　　1. 堆外内存难以控制，如果内存泄漏，那么很难排查 
　　2. 堆外内存相对来说，不适合存储很复杂的对象。一般简单的对象或者扁平化的比较适合。

站在系统设计的角度来看，使用堆外内存可以为你的设计提供更多可能。最重要的提升并不在于性能，而是决定性的。

> 堆内在flush到远程时，会先复制到直接内存（非堆内存），然后在发送的说明：HeapByteBuffer与DirectByteBuffer，在原理上，前者可以看出分配的buffer是在heap区域的，其实真正flush到远程的时候会先拷贝得到直接内存，再做下一步操作（考虑细节还会到OS级别的内核区直接内存），其实发送静态文件最快速的方法是通过OS级别的send_file，只会经过OS一个内核拷贝，而不会来回拷贝；在NIO的框架下，很多框架会采用DirectByteBuffer来操作，这样分配的内存不再是在java heap上，而是在C heap上，经过性能测试，可以得到非常快速的网络交互，在大量的网络交互下，一般速度会比HeapByteBuffer要快速好几倍。
> 直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用，而且也可能导致OutOfMemoryError 异常出现，所以我们放到这里一起讲解。 
在JDK 1.4 中新加入了NIO（New Input/Output）类，引入了一种基于通道（Channel）与缓冲区（Buffer）的I/O 方式，它可以使用Native 函数库直接分配堆外内存，然后通过一个存储在Java 堆里面的DirectByteBuffer 对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在Java 堆和Native 堆中来回复制数据。

