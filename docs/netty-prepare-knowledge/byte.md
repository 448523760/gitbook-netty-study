# byte 

在Java中，byte类型的e79fa5e98193e58685e5aeb931333433616136数据是8位带符号的二进制数。最高位表示正负，0为正，1为负。

* byte，整型，1个字节，范围：-2的7次方 ~ 2的7次方-1；
* short，整型，2个字节，范围：-2的15次方 ~ 2的15次方-1；
* int，整型，4个字节，范围：-2的31次方 ~ 2的31次方-1；
* long，整型，8个字节，范围：-2的63次方 ~ 2的63次方-1；
* float，浮点型，4个字节，范围：3.402823e+38 ~ 1.401298e-45；
* double，浮点型，8个字节，范围：1.797693e+308~ 4.9000000e-324；
* char，文本型，2个字节，范围：0~2的16次方-1；
* boolean，布尔型，1个字节，范围：true/false；

## 其他基础类型转byte

long 是8个byte，在写入buffer时，先写一个字节，然后写入index+1，data右移8bit。相当于在byte[]中写8次，每次写完一个byte的时候需要右移到下一个byte位。

`(byte)x` 将x强转位byte，**只会保留x的最低8位**

```java
private static byte long7(long x) { return (byte)(x >> 56); }
    private static byte long6(long x) { return (byte)(x >> 48); }
    private static byte long5(long x) { return (byte)(x >> 40); }
    private static byte long4(long x) { return (byte)(x >> 32); }
    private static byte long3(long x) { return (byte)(x >> 24); }
    private static byte long2(long x) { return (byte)(x >> 16); }
    private static byte long1(long x) { return (byte)(x >>  8); }
    private static byte long0(long x) { return (byte)(x      ); }
    
    static void putLongL(ByteBuffer bb, int bi, long x) {
        bb._put(bi + 7, long7(x));
        bb._put(bi + 6, long6(x));
        bb._put(bi + 5, long5(x));
        bb._put(bi + 4, long4(x));
        bb._put(bi + 3, long3(x));
        bb._put(bi + 2, long2(x));
        bb._put(bi + 1, long1(x));
        bb._put(bi    , long0(x));
    }
```


