# Error Detection and Coding: Parity, CRC & FEC
## IT 327: Lab 8

## Background

Error Detection and Coding: Parity, CRC & FEC Error detection and coding is pervasive in digital communications; it is very rare that digital information is transmitted without some of these protective measures, except in public broadcasting. For this reason, we want to play around with it more than just in class. The best way (presently) is to do this with software.



## Objectives

* To experiment with parity and multi-bit errors in parity.
* To experiment with CRC and do some actual encoding.


## Part 1:


```python
from colorama import Fore, Style
from random import randint

c0 = Fore.GREEN
c1 = Fore.BLUE
nc = Style.RESET_ALL

def get_parity(data: int, parity_type: str='odd') -> bytes:
    '''
    True = odd parity
    that generates a parity bit (even or odd - you choose)
    for each byte of data and verify its correct operation
    '''
    parity = 0
    
    while data:
        parity =  ~parity
        data = data & (data - 1)
    if parity_type == 'odd':
        if parity % 2 == 0:
            return 0
        return 1
    if parity_type == 'even':
    
        if parity % 2 == 0:
            return 1
        return 0

check_parity = lambda data: [ True if get_parity(i) == get_parity((randint(0, 255) + i ) % 256) else False for i in data]  
```


```python
data = [0,1,2,3]
data = [randint(0, 9) for i in range(10)]
odd_parity = [get_parity(i) for i in data]
parity_checks = check_parity(data)

print('odd parities of data: {}\n\t{}{}\n\t{}{}\n'.format(c0,data, c1, odd_parity,nc))
print('parity checks: \n\t{}{}\n\t{}\n'.format(c1, parity_checks,nc))
```

    odd parities of data: [32m
    	[7, 1, 6, 2, 4, 7, 6, 8, 9, 3][34m
    	[1, 1, 0, 1, 1, 1, 0, 1, 0, 0][0m
    
    parity checks: 
    	[34m[False, False, True, False, False, False, True, True, False, True]
    	[0m
    


## Part 2:
* Write a program that generates an 8-bit CRC checksum for a data stream. An example of this program follows, in pseudocode. This same program may be used to check correct reception of the data stream. The simplest data stream for which this may be used is two bytes; simulate at least 5 different 2-byte data streams both with and without errors and log the results. Particularly interesting are error data streams which differ by only 1 bit from the correct data.
* Also, simulate at least one 32-byte data stream with at least one error and log the results. Include a screenshot of the results and discuss the probability of an undetected error occurring.


```python
from bitstring import BitArray
from bitarray import bitarray as bt
from random import randint



def get_CRC(b_len: int, p_len: int):
    x = randint(1,2**b_len) - 1
    poly = randint(1,2**p_len) - 1

    b_x = BitArray(uint=x, length=b_len).bin
    b_poly = BitArray(uint=poly, length=p_len).bin
    p_bit = str(get_parity(poly))
    padding = BitArray(uint=0, length=sum([int(i) for i in b_poly])).bin

    j = 0
    k = p_len
    _pad = ''
    for i in b_x:
        if k > b_len: break;

        q = int(b_x[j:k], 2)
        y = poly ^ q
        b_y = BitArray(uint=y, length=p_len).bin
        _b_x = b_x
        b_x = _b_x[:j] + b_y + b_x[k:]
        
        print('{}{}\n{}\n{}'.format(_pad, b_y, _b_x, b_x))
        k += 1
        j = k - p_len
        _pad = ''.join([_pad, ' '])
    crc = p_bit + _b_x + padding
    return crc
```

# 5 Different 2 Byte Streams:


```python
b_len = 2**3 + 1
p_len = 2**2
for i in range(5):
    print('{}Steam {}{}:'.format(c0, i, nc))
    print('........................................')
    crc = get_CRC(b_len, p_len)
    print('{}CRC:{}{}{}'.format(c1, crc, nc, '\n'))
    
```

    [32mSteam 0[0m:
    ........................................
    1010
    010100001
    101000001
     1011
    101000001
    110110001
      1001
    110110001
    111001001
       1101
    111001001
    111110101
        0101
    111110101
    111101011
         0100
    111101011
    111100100
    [34mCRC:01111010110000[0m
    
    [32mSteam 1[0m:
    ........................................
    1100
    111100010
    110000010
     1011
    110000010
    110110010
      0101
    110110010
    110101010
       1001
    110101010
    110100110
        0000
    110100110
    110100000
         0011
    110100000
    110100011
    [34mCRC:011010000000[0m
    
    [32mSteam 2[0m:
    ........................................
    0001
    010010100
    000110100
     0110
    000110100
    001100100
      1001
    001100100
    001001100
       0110
    001001100
    001011000
        1001
    001011000
    001010010
         0111
    001010010
    001010111
    [34mCRC:000101001000[0m
    
    [32mSteam 3[0m:
    ........................................
    0100
    001010110
    010010110
     1111
    010010110
    011110110
      1000
    011110110
    011000110
       0111
    011000110
    011011110
        1001
    011011110
    011010010
         0100
    011010010
    011010100
    [34mCRC:001101001000[0m
    
    [32mSteam 4[0m:
    ........................................
    0101
    101011001
    010111001
     0100
    010111001
    001001001
      0110
    001001001
    000110001
       0011
    000110001
    000001101
        1001
    000001101
    000010011
         1100
    000010011
    000011100
    [34mCRC:00000100110000[0m
    


# 32bit Stream


```python
b_len = 2**5 + 1
p_len = 2**2
print('{}Steam {}{}:'.format(c0, b_len, nc))
print('........................................')
crc = get_CRC(b_len, p_len)
print('CRC: {}{}{}{}'.format(c1, crc, nc, '\n'))
```

    [32mSteam 33[0m:
    ........................................
    0110
    100001010011110100100010001010011
    011001010011110100100010001010011
     0010
    011001010011110100100010001010011
    000101010011110100100010001010011
      1011
    000101010011110100100010001010011
    001011010011110100100010001010011
       1000
    001011010011110100100010001010011
    001100010011110100100010001010011
        1111
    001100010011110100100010001010011
    001111110011110100100010001010011
         0000
    001111110011110100100010001010011
    001110000011110100100010001010011
          1110
    001110000011110100100010001010011
    001110111011110100100010001010011
           0011
    001110111011110100100010001010011
    001110100111110100100010001010011
            1001
    001110100111110100100010001010011
    001110101001110100100010001010011
             1101
    001110101001110100100010001010011
    001110101110110100100010001010011
              0101
    001110101110110100100010001010011
    001110101101010100100010001010011
               0100
    001110101101010100100010001010011
    001110101100100100100010001010011
                0111
    001110101100100100100010001010011
    001110101100011100100010001010011
                 0000
    001110101100011100100010001010011
    001110101100000000100010001010011
                  1110
    001110101100000000100010001010011
    001110101100001110100010001010011
                   0011
    001110101100001110100010001010011
    001110101100001001100010001010011
                    1000
    001110101100001001100010001010011
    001110101100001010000010001010011
                     1110
    001110101100001010000010001010011
    001110101100001011110010001010011
                      0010
    001110101100001011110010001010011
    001110101100001011001010001010011
                       1011
    001110101100001011001010001010011
    001110101100001011010110001010011
                        1000
    001110101100001011010110001010011
    001110101100001011011000001010011
                         1110
    001110101100001011011000001010011
    001110101100001011011111001010011
                          0010
    001110101100001011011111001010011
    001110101100001011011100101010011
                           1011
    001110101100001011011100101010011
    001110101100001011011101011010011
                            1000
    001110101100001011011101011010011
    001110101100001011011101100010011
                             1111
    001110101100001011011101100010011
    001110101100001011011101111110011
                              0000
    001110101100001011011101111110011
    001110101100001011011101110000011
                               1110
    001110101100001011011101110000011
    001110101100001011011101110111011
                                0011
    001110101100001011011101110111011
    001110101100001011011101110100111
                                 1001
    001110101100001011011101110100111
    001110101100001011011101110101001
    CRC: [34m1001110101100001011011101110100111000[0m
    


## Conclusions
### Parity Bit Error Checking Seems limited to the pairity count of the parity bit itself, CRC looks much more reliable. Honestly, I had no idea how parity bit check worked until doing this lab. This has been my favorite lab thus far in this course. With that said, I am totally clear on how this applies to the rest of what we've learned in this class - hopefully, I'll see soon.

