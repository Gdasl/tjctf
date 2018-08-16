# BadCipher

My friend insisted on using his own cipher program to encrypt this flag, but I don't think it's very secure. Unfortunately, he is quite good at Code Golf, and it seems like he tried to make the program as short (and confusing!) as possible before he sent it.

I don't know the key length, but I do know that the only thing in the plaintext is a flag. Can you break his cipher for me?

## Original program
``` python
def encrypt(m,k,_):
 r,o,u,x,h=range,ord,chr,"".join,hex
 l=len(k);s=[m[i::l]for i in r(l)] 
 for i in r(l):
  a,e=0,""
  for c in s[i]:
   a=o(c)^o(k[i])^(a>>2)
   e+=u(a)
  s[i]=e
 return x(h((1<<8)+o(f))[3:]for f in x(x(y)for y in zip(*s)))
 
flag = '473c23192d4737025b3b2d34175f66421631250711461a7905342a3e365d08190215152f1f1e3d5c550c12521f55217e500a3714787b6554'
```

## Decrypting the program

This is clearly a simple xor block cipher with IV 0. To make it a bit easier I rewrote the program in a legible way:
```python
def encrypt(m,k):
    l=len(k)
    s=[m[i::l]for i in range(l)]
    for i in range(l):
        a=0
        e=""
        for c in s[i]:
            a=ord(c)^ord(k[i])^(a>>2)
            e+=chr(a)
        s[i]=e
    string1 = zip(*s)
    string2 = "".join("".join(y)for y in string1)
    else:
        enci = ""
        for f in string2:
             
            tmp = (1<<8)+ord(f) 
            tmp_h = hex(tmp)[3:]
            
            enci += tmp_h
        return enci
 ```
 
 Rapidly we see that this is more of a hash than an actual cipher. The reason is the ```zip" (*s)```. As explained by a helpful StackOverflow contributor ```The iterator stops when the shortest input iterable is exhausted````. This means that if for some reason the length of the message is not a multiple of the key, the resulting array in line 3 ```s=[m[i::l]for i in range(l)]``` will have at least one element of shorter length than the rest. During the zip operation, this will result in all information contained in the last byte of all other elements of the array to be lost.
 
 This caused me quite some grief while testing because I couldn't understand how the resulting encrypted string could be so much shorter than the input. In the meantime however, I realized that in order to get the first bytes of the key this wouldn't matter, for as long as the length of the key wasn't exceeded. I knoew that the flag would start with 'tjctf{' so I wrote the following script:
 
 ```python
 def finchr(ex,texttofind,flag):
    for i in range(255):
            keyer = ex + chr(i)
            if encrypt(flag,keyer,1).decode('hex')[0:len(keyer)] == texttofind[0:len(keyer)]:
                    return i
            else:
                pass
		
def brutus(txt,flag):
    keykey = ""
    for i in range(len(txt)):
        tmp = finchr(keykey,txt[0:len(keykey)+1],flag)
        keykey += chr(tmp)
    return keykey
 
 fir_key = brutus('tjctf{',flag.decode('hex'))
 ```
 
 This sucessfully gave me the initial 6 chars: ```3V@mK<```. At this point however, I couldn't rely on that kind of bruteforcing anymore since the flags have been shown to contain digits and letters. At this point my best bet was to devise a second bruteforcing method. In this case, I wrote a decryption script, then just incremented the key by one char each time and tested all possible combinations for order 32 to 130 (it still had to be printable I figured), decrypted the flag using that key and checked if it was printable:

```python
def decrypt(m,k):
    l=len(k)
    s=[m[i::l]for i in range(l)] #always results in arr of length l
    for i in range(l):
        a=0
        e=""
        for c in s[i]:
            cur_k = k[i]
            tmp = ord(c)^ord(cur_k)^(a>>2)
            a = ord(c)
            e+=chr(tmp)
        s[i]=e
    string1 = zip(*s)
    string2 = "".join("".join(y)for y in string1)
    dec = ""
    for f in string2:
        tmp = (1<<8)+ord(f)
        tmp_h = hex(tmp)[3:]
        dec += tmp_h
    return dec.decode('hex')
    
 def checkifprint(stri):
    for i in stri:
        if i not in good_bubu:
            return False
            break
    return True
            
 ```
 
 In the end the key turned out to be 8 chars (that's when I got the most ```checkifprint=True``` and the final```}```) leaving me with about 4 possibilities. I picked the most likely one:
 
flag : ```tjctf{m4ybe_Wr1t3ing_mY_3ncRypT10N_MY5elf_W4Snt_v_sm4R7}```
key : ```3V@mK<[0```


 
