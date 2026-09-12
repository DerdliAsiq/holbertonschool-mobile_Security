Report — Task 1: Hooking Native Android Functions

Objective Extract a hidden flag from the native code (JNI) of an Android application (task1_d.apk) without running the application, by combining static analysis of Java bytecode and the native binary.

Step 1 — Static Analysis of Java Bytecode

APK decompiled using jadx:

jadx ~/Desktop/task1_d.apk -d ~/Desktop/task1_jadx

Inspection of MainActivity.java shows that the application declares a native method and loads a native library:

public final native String getSecretMessage();

static { System.loadLibrary("native-lib"); }

The getSecretMessage() method is implemented in C inside the native library. It is never displayed in the UI, confirming that direct binary analysis is required.

Step 2 — Identifying the Native Library

Libraries extracted using apktool:

apktool d ~/Desktop/task1_d.apk -o ~/Desktop/task1_out find ~/Desktop/task1_out -name "*.so"

Found libraries for multiple architectures: x86_64, arm64-v8a, x86, armeabi-v7a.

Checking exported symbols:

nm -D ~/Desktop/task1_out/lib/x86_64/libnative-lib.so | grep -i "secret|jni"

Result:

0000000000000860 T Java_com_holberton_task2_1d_MainActivity_getSecretMessage Step 3 — Disassembly Analysis

Using objdump:

objdump -d ~/Desktop/task1_out/lib/x86_64/libnative-lib.so | grep -A 80 "getSecretMessage"

Algorithm inside getSecretMessage:

Copies 49 bytes (0x31) from the .rodata section at offset 0x5f0

For each byte at index i:

char[i] = char[i] - lit(i % 10) Sends the decrypted string back to the JVM using NewStringUTF

The function lit(n) computes the nth Fibonacci number iteratively:

lit(0..9) = [0, 1, 1, 2, 3, 5, 8, 13, 21, 34] Step 4 — Extracting Encrypted Data

Using:

readelf -x .rodata ~/Desktop/task1_out/lib/x86_64/libnative-lib.so

Raw bytes at offset 0x5f0:

48 70 6d 64 68 77 7c 7c 83 9d 6e 62 75 6b 75 6a 67 75 84 91 6b 6a 6f 69 62 6e 7b 6c 83 91 5f 65 6a 68 69 6a 7a 72 83 96 5f 62 75 61 64 71 74 8a 00 Step 5 — Decryption

Python reimplementation of the algorithm:

def fib(n): if n <= 1: return n a, b = 0, 1 for _ in range(2, n+1): a, b = b, a + b return b

fibs = [fib(i) for i in range(10)]

raw = [72, 112, 109, 100, 104, 119, 124, 124, 131, 157, 110, 98, 117, 107, 121, 106, 103, 117, 132, 145, 107, 106, 111, 105, 98, 110, 123, 108, 131, 145, 95, 101, 106, 104, 105, 106, 122, 114, 131, 150, 95, 98, 117, 97, 100, 113, 116, 138, 0]

result = "" for i, b in enumerate(raw): if b == 0: break result += chr(b - fibs[i % 10])

print(result) Final Flag Holberton{native_hooking_is_no_different_at_all} Tools Used Tool Purpose jadx Java bytecode decompilation apktool Extraction of native libraries nm / readelf Symbol and section analysis of .so objdump Disassembly of native function python3 Reimplementation and decryption of algorithm
