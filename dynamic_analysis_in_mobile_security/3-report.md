Report — Task 3: Revealing Hidden Android Functions Objective

Extract a hidden flag from an Android application (task3_d.apk) where the decryption function is never called during normal execution of the app.

Step 1 — Static Analysis of Java Bytecode

APK decompiled using jadx:

jadx ~/Desktop/task3_d.apk -d ~/Desktop/task3_jadx

Search for flag-related functions:

grep -r "hidden|secret|flag|decrypt|encode|Holberton"
~/Desktop/task3_jadx/sources/com --include="*.java" -l

Relevant file identified:

com/holberton/task4_d/MainActivityKt.java Step 2 — Identification of Hidden Function

Inside MainActivityKt.java, a private method with an intentionally obfuscated name is found:

private static final void aBcDeFgHiJkLmNoPqRsTuVwXyZ123456(Function1<? super String, Unit> function1)

This function is never invoked in the normal application flow. The UI only displays:

"Hmm it seems the interesting function is never called."

Step 3 — Decryption Algorithm Analysis

The function performs the following operations:

Step 3.1 — Base64 decoding 8CP4zSyn62t78lwwc383rxcgtv/UiMv3Pw+Mfw12LzXvorIpBypNK/oB7XvWNV0oWfoX Step 3.2 — Byte transformations per index i

For each byte:

int temp = value ^ 19 int temp2 = (((temp >> 2) | (temp << 6)) & 255) - (index * 3) temp2 = temp2 % 256 if temp2 < 0: temp2 += 256 char c = (char) ((temp2 * 183) % 256)

Finally, all characters are concatenated to form the flag.

Step 4 — Reimplementation & Decryption

Python implementation (without executing the app):

import base64

data = base64.b64decode( "8CP4zSyn62t78lwwc383rxcgtv/UiMv3Pw+Mfw12LzXvorIpBypNK/oB7XvWNV0oWfoX" ) decoded_bytes = [b & 0xFF for b in data]

flag_chars = [] for index, value in enumerate(decoded_bytes): temp = value ^ 19 temp2 = (((temp >> 2) | (temp << 6)) & 255) - (index * 3) temp2 = temp2 % 256 if temp2 < 0: temp2 += 256 flag_chars.append(chr((temp2 * 183) % 256))

print("".join(flag_chars)) Result Flag: Holberton{calling_uncalled_functions_is_now_known!} Security Observations The hidden function is private and static — not accessible through the UI, but fully visible after decompilation. The obfuscated name (aBcDeFgHiJkLmNoPqRsTuVwXyZ123456) is a weak hiding technique and provides no real protection against static analysis. All decryption logic is embedded in the binary — no external secrets are required to reconstruct the flag. Tools Used Tool Purpose jadx Java bytecode decompilation python3 Reimplementation of algorithm and decryption
