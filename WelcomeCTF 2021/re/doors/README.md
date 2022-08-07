# Chall Name

## Solution

Door 1: enter 9708102
Door 2: enter 992428
Door 3: enter 2187324
Door 4: enter 612381

Flag: greyhats{0p3n_th3m_w17h_c0d3}

## Deeper explanations

The given code has a few "doors", which are different checks.

Door 1:

```python
def door1():
    global flag

    print("Door 1/4")
    print("Enter the passcode:")
    try:
        key = int(input())
    except:
        print("What are you doing...")
        sys.exit(1)

    if (key & 0xfff) == 0x246 and (key >> 12) == 0x942:
        print("Correct!\n")
    else:
        print("I don't know you. Go away.")
        sys.exit(1)

    flag += decode(0, key) + "\_"
```

1. `key & 0xfff == 0x246` checks that the least significant 12 bits must correpsond to `0x246`. (Decimal: `582`, Binary: `0010 0100 0110`)
2. `(key>>12)` shifts `key` by 12 bits to the right and checks if the remaining bits of `key` is equals to `0x942`. (Decimal: irrelevant, Binary: `1001 0100 0010`)
3. Both conditions must be satisfied. However, they can be treated as two separate problems: the former checks the first 12 bits, the latter checks the remaining bits.
4. From 1, the first 12 bits of key must be `0010 0100 0110`.
5. From 2, the next 12 bits must be `1001 0100 0010`.
6. Putting 1 and 2 together, key must be of the binary value: `0000 .... 1001 0100 0010 0010 0100 0110` = `9708102` in decimal.

Door 2:

```python
def door2():
    global flag

    print("Door 2/4")
    print("Enter the passcode:")
    try:
        key = int(input())
    except:
        print("What are you doing...")
        sys.exit(1)

    #key%1000=428 && 992000=((189^861)*1000)
    #FINAL ANS: 992428
    if (key % 1000) ** 3 == 78402752 and ((key // 1000) ^ 861) == 189:
        print("Correct!\n") 
    else:
        print("I don't know you. Go away.")
        sys.exit(1)

    flag += decode(1, key) + "\_"
```

1. `(key % 1000) ** 3 == 78402752` takes the first 3 decimal digits and raises it to the power of 3, then checks if it corresponds to decimal `78402752`.
2. `((key // 1000) ^ 861) == 189` discards the first 3 decimal digits and XORs the resultant value with decimal value `861`, then checks if it corresponds to decimal `189`.
3. Both conditions must be satisfied. However, they can be treated as two separate problems once again. The former checks the first 3 decimal digits, the latter checks the remaining decimal digits.
4. From 1, cube root `78402752` to get `428`. 
5. From 2, it is known that `a^b^b=a`. Since `(key//1000^861)` = `189`, simply perform `189^861` to get `992`.
6. The decimal value to be input is `992 428`.

Door 3:

```python
def door3():
    global flag

    print("Door 3/4")
    print("Enter the passcode:")
    try:
        key = int(input())
    except:
        print("What are you doing...")
        sys.exit(1)

    random.seed(52318)
    pos = [i for i in range(7)]
    random.shuffle(pos)
    #pos is now [6,0,3,4,5,1,2]

    key_ = 0
    #7th digit: 2
    #1st digit: 4
    #4th digit: 7
    #5th digit: 8
    #6th digit: 1
    #2nd digit: 2
    #3rd digit: 3

    #FINAL ANS: 2187324
    for p in pos:
        dg = key % (10 ** (p + 1)) #extract first p+1 digit. (eg p=6, %10 000 000 means up to 9 999 999)
        dg //= 10 ** p  #// floor division , dg = floor(dg/(10 power p)) #EXTRACT P-th digit! eg 9 876 543 gives 9
        key -= dg * (10 ** p) #remove the involved digit in the 2nd step from key
        key_ *= 10 #shift existing keys to the left
        key_ += dg #add extracted digit in

    if key != 0:
        print("I don't know you. Go away.")
        sys.exit(1)

    if key_ == 2478123:
        print("Correct!\n")
    else:
        print("I don't know you. Go away.")
        sys.exit(1)

    flag += decode(2, key) + "_"
```

1. Note that `random.shuffle(pos)` always produces `[6, 0, 3, 4, 5, 1, 2]` since the `random()` call has been seeded. 
2. `dg` can be expanded to digit value. The first 3 lines in the `for p in pos:` loops extracts a value associated with a particular n-th digit.
3. `dg` is inserted into the least significant digit of `key_`. Then, `key_` is multiplied by 10 (to do a decimal shift left by 1)
4. In short, the code above reassembles `key` into `key_` by swapping values in various digits. The swap is determined by the `pos` array. However, since the `random` call is seeded, `random.shuffle(pos)` produces the same array every time: `[6, 0, 3, 4, 5, 1, 2]`.
5. Given that `key_` must be `2478123`, and the array `pos` represents the order of digits extracted from `key`, it is possible to reassemble the original `key`. 
6. The 6th digit (0-based indexing) of `key` is extracted first, so the most significant digit of `key_` must be the 6th digit of `key`. (`2`)
7. The 0th digit of `key` is extracted next, so the second most significant digit of `key_` must be the 0th digit of `key`. (`4`)
8. Repeat this process to reconstruct the original `key`: `2187324`.

Door 4:

```python
def door4():
    global flag

    print("Door 4/4")
    print("Enter the passcode:")
    try:
        key = int(input())
    except:
        print("What are you doing...")
        sys.exit(1)

    hhs = [
        b'919c5ab6d04ee9d1c81335692d2ed68e',
        b'9f7e8841c1d64dfde51953fccecde2cf',
        b'bcb34c837111773983f4860ee51cb1b6',
        b'e48b14700f10e3d8f3ad25b73a0d20db',
        b'835f2ff9473cfc787d4f1c08ad5037c9',
        b'c3e3b06937f539a2e52b834a4f1d27fc',
    ]

    import hashlib
    import binascii
    for i in range(6):
        dg = key % 10
        key //= 10
        hh = binascii.hexlify(hashlib.md5(f"{i}_{dg}_{key}".encode()).digest())

        if hhs[i] != hh:
            print("I don't know you. Go away.")
            sys.exit(1)

    if key != 0:
        print("I don't know you. Go away.")
        sys.exit(1)

    print("Correct!\n")
    flag += decode(3, key) + "}"
```

1. The code takes `key` and extracts the least significant decimal digit. 
2. A string, `{i}_{dg}_{key}` goes through a round of `md5` hashing and is compared with preset values.
3. This is repeated for the next least 5 significant digits of `key`. 
4. A brute force script could be used to calculate the required `key`. The below script simply tries possible values for `key` and subjects it to the hashing process to ensure that the calculated md5 hashes are correct.

```python
import hashlib
import binascii

hhs = [
    b'919c5ab6d04ee9d1c81335692d2ed68e',
    b'9f7e8841c1d64dfde51953fccecde2cf',
    b'bcb34c837111773983f4860ee51cb1b6',
    b'e48b14700f10e3d8f3ad25b73a0d20db',
    b'835f2ff9473cfc787d4f1c08ad5037c9',
    b'c3e3b06937f539a2e52b834a4f1d27fc',
]

for originalkey in range(1000000,0,-1):
    for i in range(6):
        key=originalkey
        dg = key % 10 #get 1st digit
        key //= 10 #shift key to the right by 1 digit
        passcode=f"{i}_{dg}_{key}"
        hh = binascii.hexlify(hashlib.md5(passcode.encode()).digest())
        if hh==hhs[i]:
            print("solution found: key = "+str(originalkey))
            print(hh)
            break
        if (key==0):
            break
```

The original key is `612381`.