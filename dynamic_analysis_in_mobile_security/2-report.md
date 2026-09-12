Report — Task 2: Android Cryptography — Interception and Decryption Objective

Extract a hidden flag from an Android application (app-release-task2.apk) that uses XOR encryption based on a Fibonacci calculation. No remote server is involved — the encryption is fully local.

Step 1 — Static Analysis of Java Bytecode

APK decompiled using jadx:

jadx ~/Desktop/app-release-task2.apk -d ~/Desktop/task2_jadx

Relevant files identified:

com/holberton/task3/MainActivityKt.java — contains the full decryption logic com/holberton/task3/MainActivity.java — simple UI launcher Step 2 — Algorithm Analysis

Three key functions found in MainActivityKt.java:

performslowDecryption() public static final String performslowDecryption() { byte[] decoded = Base64.getDecoder().decode( "cVZaW1dDQllZTFdRW1xeUlBbX21CWFtHalRZXUJFRFhNX1ZcbllGQ15cUUNSRFpcVks=" ); return xorDecrypt(new String(decoded, UTF_8), String.valueOf(slowRecursive(150))); } slowRecursive(int n) — Naive recursive Fibonacci public static final long slowRecursive(int i) { return i <= 1 ? i : slowRecursive(i - 1) + slowRecursive(i - 2); }

This computes Fibonacci(150) using an intentionally inefficient recursive method (exponential time complexity), making dynamic analysis slow.

xorDecrypt(String encryptedFlag, String key) Performs character-by-character XOR decryption: result[i] = key[i % key.length()] XOR encrypted[i] Step 3 — Decryption

Python reimplementation (without executing the app):

import base64

def fib(n): a, b = 0, 1 for _ in range(n): a, b = b, a + b return a

key = str(fib(150))

fib(150) = 9969216677189303386214405760200
encrypted = base64.b64decode( "cVZaW1dDQllZTFdRW1xeUlBbX21CWFtHalRZXUJFRFhNX1ZcbllGQ15cUUNSRFpcVks=" ).decode('utf-8')

flag = "".join(chr(ord(key[i % len(key)]) ^ ord(c)) for i, c in enumerate(encrypted)) print(flag) Result Flag: Holberton{fibonacci_slow_computation_optimization} Security Observations The encryption key is fully derivable from the source code — no external secret is used. slowRecursive is a naive Fibonacci implementation with O(2^n) complexity, intentionally used to slow down dynamic analysis. The flag is XOR-encrypted using a key derived from fib(150) converted to a string — easily reversible via static analysis. Tools Used Tool Purpose jadx Java bytecode decompilation python3 Algorithm reimplementation and decryption
