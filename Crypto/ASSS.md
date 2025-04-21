# crypto/ASSS

## Challenge Summary:

Welcome to the Amazing SSS! Join now to get your share of the secret. The flag is 66 bytes long. <br>
`ncat --ssl asss.atreides.b01lersc.tf 8443`

We're given the **ASSS.py** source code below: 
```python
from Crypto.Util.number import getPrime, bytes_to_long

def evaluate_poly(poly:list, x:int, s:int):
    return s + sum(co*x**(i+1) for i, co in enumerate(poly))

s = bytes_to_long(open("./flag.txt", "rb").read())
a = getPrime(64)
poly = [a*getPrime(64) for _ in range(1, 20)]
share = getPrime(64)

print(f"Here is a ^_^: {a}")
print(f"Here is your share ^_^: ({share}, {evaluate_poly(poly, share, s)})")
```

## Analysis

**Server**: Gives 64-bit prime `(a)` and share `((x, y))`, where `(y = P(x))`, `P(x) = s + c₀x + c₁x² + ... + c₁₈x¹⁹`, with each `cᵢ = a * pᵢ`.
Since all coefficients are multiples of a: `P(x) ≡ s (mod a)`
The flag `s` is 528 bits long (66 bytes)

By collecting at least **9 unique `(aᵢ, yᵢ)` pairs**, we can recover `s` using the Chinese Remainder Theorem (CRT) over a total modulus > 528 bits.

## Solution

### Steps:
1. Collect 9+ unique `(aᵢ, yᵢ)` pairs.
2. Use CRT to solve `s ≡ yᵢ (mod aᵢ)`.
3. Convert `(s)` to bytes.

```python
from Crypto.Util.number import long_to_bytes
import pwn

def crt(residues, moduli):
    # Implementation of Chinese Remainder Theorem
    from math import prod
    N = prod(moduli)
    total = 0
    for r, m in zip(residues, moduli):
        Ni = N // m
        inv = pow(Ni, -1, m)  # Modular inverse of Ni mod m
        total += r * Ni * inv
    return total % N

# Lists to store moduli (a_i) and residues (y_i)
moduli = []
residues = []

while len(moduli) < 9:
    conn = pwn.remote('asss.atreides.b01lersc.tf', 8443, ssl=True)
    # Receive output (example: "Here is a ^_^: 12345\nHere is your share ^_^: (6789, 98765)")
    lines = conn.recvall().decode().strip().split('\n')
    a = int(lines[0].split(': ')[1])
    share = eval(lines[1].split(': ')[1])  # Parse (x, y)
    x, y = share
    # Store a and y (since y ≡ s mod a)
    if a not in moduli:  # Ensure distinct moduli
        moduli.append(a)
        residues.append(y)
    conn.close()

# Apply CRT
s = crt(residues, moduli)

# Convert to bytes
flag = long_to_bytes(s)
if flag.startswith(b'bctf{') and len(flag) == 66:
    print("Flag:", flag.decode())
else:
    print("Need more shares or incorrect reconstruction.")
```
