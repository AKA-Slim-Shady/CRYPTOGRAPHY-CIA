# CRYPTOGRAPHY-CIA

# Autokey Cipher (XOR Implementation)

This repository contains a Python implementation of the classic **Autokey Cipher**, but with a modern cryptographic twist: instead of using standard modular arithmetic to encrypt the letters, it uses the **Bitwise XOR (`^`)** operator.

## 1. The Hashing Function Used
Instead of the traditional "Division Hash" (using Modular 26 addition, e.g., `(Plaintext + Key) % 26`), this script uses the **Bitwise XOR (`^`)** operator. 

XOR (Exclusive OR) is a foundational operation in modern cryptography. It works by comparing the binary bits of two numbers and outputting a `1` only if the bits are different. One of the most powerful properties of XOR is that it is its own inverse. This means `(A ^ B) ^ B = A`. Because of this symmetry, the exact same mathematical function can be used for both encryption and decryption!

## 2. Steps for Doing Autokey using Bitwise XOR

**Step 1: Generate the Autokey**
Take your secret "primer" key and attach the plaintext message to the end of it, dropping spaces. Stop when the key matches the length of the message.
* **Plaintext:** `THE`
* **Primer Key:** `SEC`
* **Generated Key:** `SEC` (Since it matches the length of "THE")

**Step 2: Map to Dictionary Values**
Convert both the Plaintext and Key letters to their numeric index based on the dictionary.
* `T` = 19, `H` = 7, `E` = 4
* `S` = 18, `E` = 4, `C` = 2

**Step 3: Apply the XOR Hash**
Perform a Bitwise XOR between the Plaintext value and the Key value.
* `19 ^ 18 = 1`
* `7 ^ 4 = 3`
* `4 ^ 2 = 6`

**Step 4: Convert Back to Text**
Map the resulting numbers back to characters using the dictionary.
* `1` = B
* `3` = D
* `6` = G
* **Encrypted Text:** `BDG`

To decrypt, you simply run the exact same XOR steps: `Encrypted ^ Key = Plaintext` (`1 ^ 18 = 19`, which is `T`).

## 4. Python Implementation

'''
# Python program to implement Autokey Cipher using XOR

# Expanded Dictionary to 32 characters (A-Z + 1-6) 
# This prevents out-of-bounds errors when using bitwise XOR without modulo.
dict1 = {'A': 0, 'B': 1, 'C': 2, 'D': 3, 'E': 4,
         'F': 5, 'G': 6, 'H': 7, 'I': 8, 'J': 9,
         'K': 10, 'L': 11, 'M': 12, 'N': 13, 'O': 14,
         'P': 15, 'Q': 16, 'R': 17, 'S': 18, 'T': 19,
         'U': 20, 'V': 21, 'W': 22, 'X': 23, 'Y': 24, 'Z': 25,
         '1': 26, '2': 27, '3': 28, '4': 29, '5': 30, '6': 31}

dict2 = {0: 'A', 1: 'B', 2: 'C', 3: 'D', 4: 'E',
         5: 'F', 6: 'G', 7: 'H', 8: 'I', 9: 'J',
         10: 'K', 11: 'L', 12: 'M', 13: 'N', 14: 'O',
         15: 'P', 16: 'Q', 17: 'R', 18: 'S', 19: 'T',
         20: 'U', 21: 'V', 22: 'W', 23: 'X', 24: 'Y', 25: 'Z',
         26: '1', 27: '2', 28: '3', 29: '4', 30: '5', 31: '6'}


# This function generates the key
def generate_key(message, key):
    i = 0
    while True:
        if len(key) == len(message):
            break
        if message[i] == ' ':
            i += 1
        else:
            key += message[i]
            i += 1
    return key


# This function returns the encrypted text 
# generated using the Bitwise XOR hash function
def cipherText(message, key_new):
    cipher_text = ''
    i = 0
    for letter in message:
        if letter == ' ':
            cipher_text += ' '
        else:
            # XOR operation replaces the addition and modulo
            x = dict1[letter] ^ dict1[key_new[i]]
            i += 1
            cipher_text += dict2[x]
    return cipher_text


# This function decrypts the encrypted text
# XOR is its own inverse, so the decryption logic is identical to encryption!
def originalText(cipher_text, key_new):
    or_txt = ''
    i = 0
    for letter in cipher_text:
        if letter == ' ':
            or_txt += ' '
        else:
            # XOR operation reverses the encryption automatically
            x = dict1[letter] ^ dict1[key_new[i]]
            i += 1
            or_txt += dict2[x]
    return or_txt


def main():
    message = 'THE GERMAN ATTACK'
    key = 'SECRET'
    key_new = generate_key(message, key)
    
    cipher_text = cipherText(message, key_new)
    original_text = originalText(cipher_text, key_new)
    
    print("Plaintext Message =", message)
    print("Generated Autokey =", key_new)
    print("Encrypted Text    =", cipher_text)
    print("Decrypted Text    =", original_text)


# Executes the main function
if __name__ == '__main__':
    main()
'''

## 3. Why 32 Characters Instead of 26?
When you use modulo 26 arithmetic, the numbers are forced to wrap around, guaranteeing they never exceed 25. 

Bitwise XOR does not work like this. If you XOR two numbers between 0 and 25 (like `19 ^ 18 = 1`, or `25 ^ 15 = 22`), the result can theoretically go up to **31** (which is `11111` in binary). 

If we kept a 26-character dictionary, the script would eventually calculate a number like 28 and throw a `KeyError` because '28' doesn't exist in the alphabet. By padding the dictionary with numbers `1` through `6`, we expand the index to **32 characters** (a perfect power of 2: $2^5$). This completely contains the 5-bit XOR logic and prevents any out-of-bounds errors.

## 4. Vulnerabilities and Cryptanalysis
This specific implementation has two major vulnerabilities:

* **The Zero-Identity Flaw (XOR specific):** If a letter in the plaintext perfectly matches the letter in the key (e.g., `E` ^ `E`), the mathematical result of an XOR is always `0`. Because `0` maps to `A` in our dictionary, anytime the plaintext and key share a letter, the encrypted output will predictably be `A`. (This can be fixed in future versions by adding a constant "Salt" to the XOR equation).
* **The Crib-Dragging Attack (Autokey specific):** The fatal flaw of all Autokey ciphers is that *the key is made out of the message*. If an attacker guesses even a single word in your message (like guessing "ATTACK" is in the text), they can reverse the XOR to reveal a piece of the key. Because the key *is* the plaintext, they just revealed another piece of your message! They can "drag" this known text down the cipher to unzip the entire message in a chain reaction.

## 5. Why is this Better or Worse than Modulo 26?

**Why it is Better:**
* **Computational Efficiency:** Bitwise operations are significantly faster for computer processors to execute than division/modulo math.
* **Symmetry:** You do not need separate formulas for encryption and decryption. `+26 % 26` is no longer required to reverse the math. The code is much cleaner.

**Why it is Worse:**
* **The 'A' Problem:** As mentioned above, basic Modulo 26 doesn't have the Zero-Identity flaw. In Modulo 26, `E` (4) + `E` (4) = `I` (8). In XOR, `E` (4) ^ `E` (4) = `A` (0), creating an unwanted pattern if un-salted.
* **Non-Alphabetic Characters:** By forcing a 32-character dictionary, your ciphertext will sometimes output numbers (1-6). If you are trying to disguise your code as a standard A-Z alphabet block, the presence of numbers immediately gives away that an expanded dictionary or bitwise math is being used.
