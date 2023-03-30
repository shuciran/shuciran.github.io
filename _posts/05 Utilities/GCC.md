### Basic Compilation
```bash
gcc -o exploit exploit.c
```

### 32-Bit Compilation
```bash
gcc -m32 -Wl,--hash-style=both -o exploit exploit.c
```