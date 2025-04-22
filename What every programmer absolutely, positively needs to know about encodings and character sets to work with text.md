[source](https://kunststube.net/encoding/)
#string #utf-8 #unicode
## Getting the basics straight
- A computer cannot store "letters", "numbers", "pictures" or anything else.
- The only thing it can store and work with are _bits_.
- To use bits to represent anything at all besides bits, we need rules.

### Encoding
We need to convert a sequence of bits into something like letters, numbers and pictures using an _encoding scheme_, or _encoding_ for short.

```
01100010 01101001 01110100 01110011
b        i        t        s
```

The above encoding scheme happens to be ASCII.

The ASCII encoding specifies a table translating bytes into human readable letters. Here's a short excerpt of that table:

|       bits | character |
| ---------: | --------- |
| `01000001` | A         |
| `01000010` | B         |
| `01000011` | C         |
| `01000100` | D         |
| `01000101` | E         |
| `01000110` | F         |
In ASCII:
- There are 95 human readable characters specified in the ASCII table, including:
	- the letters A through Z both in upper and lower case,
	- the numbers 0 through 9,
	- a handful of punctuation marks and characters like the dollar symbol, the ampersand and a few others.
- It also includes 33 values for things like space, line feed, tab, backspace and so on.
	- These are not _printable_
	- A number of values are only useful to a computer, like codes to signify the start or end of a text.
- In total there are 128 characters defined in the ASCII encoding.
	- It fills up 7bits
```
01001000 01100101 01101100 01101100 01101111 00100000
  01010111 01101111 01110010 01101100 01100100
  
"Hello World"
```
⚠️ Although ASCII had 1 more bit left (128 more free rooms) to use, it was not enough to encode all of the characters of the rest of the world.
### Important terms
**Encode**
to use something to represent something else. An _encoding_ is the set of rules with which to convert something from one representation to another.

**character set, charset**
The set of characters that can be encoded. "The ASCII encoding encompasses a character set of 128 characters."

**code page**
A "page" of codes that map a character to a number or bit sequence. A.k.a. "the table". Essentially synonymous to "encoding".

## Multi-byte encodings
- To create a table that maps characters to letters for a language that uses more than 256 characters, one byte simply isn't enough.
- Using two bytes (16 bits), it's possible to encode 65,536 distinct values.
- BIG-5 is such a double-byte encoding for Chinese language.
- GB1830 is another
- A lot of encodings were invented

## Unicode to the confusion
- Finally one encoding created to unify all encoding standards. This standard is Unicode.
- It basically defines a ginormous table of 1,114,112 code points that can be used for all sorts of letters and symbols.
- Using Unicode, you can write a document containing virtually any language using any character you can type into a computer.

❓So, how many bits does Unicode use to encode all these characters? _None._ Because Unicode is not an encoding.

- _Unicode_ first and foremost defines a table of _code points_ for characters.
- How these code points are actually _encoded into bits_ is a different topic.
- To represent 1,114,112 different values, two bytes aren't enough. Three bytes are, but three bytes are often awkward to work with, so four bytes would be the comfortable minimum.

**But there's a problem**
If the letter "A" was always encoded to `00000000 00000000 00000000 01000001`, "B" always to `00000000 00000000 00000000 01000010` and so on, any document would bloat to four times the necessary size.

To optimize this, there are several ways to encode Unicode code points into bits.

**UTF-32**
is such an encoding that encodes all Unicode code points using 32 bits. 
- Very simple to implement
- Wastes a lot of space.

**UTF-16 and UTF-8**
- are _variable-length encodings_
	- If a character can be represented using a single byte (because its code point is a very small number), UTF-8 will encode it with a single byte.
	- If it requires two bytes, it will use two bytes and so on.

**How it works?**
It has elaborate ways to use the highest bits in a byte to signal how many bytes a character consists of.

UTF-16 is in the middle, using at least two bytes, growing to up to four bytes as necessary.

| character | encoding |                                  bits |
| --------- | -------- | ------------------------------------: |
| A         | UTF-8    |                            `01000001` |
| A         | UTF-16   |                   `00000000 01000001` |
| A         | UTF-32   | `00000000 00000000 00000000 01000001` |
| あ         | UTF-8    |          `11100011 10000001 10000010` |
| あ         | UTF-16   |                   `00110000 01000010` |
| あ         | UTF-32   | `00000000 00000000 00110000 01000010` |

**Remember**
**Unicode** is a large table mapping characters to numbers and the different **UTF encodings** specify how these numbers are encoded as bits.

### Code points
- Characters are referred to by their "Unicode code point".
- Unicode code points are written in hexadecimal (to keep the numbers shorter), preceded by a "U+"
-  The character Ḁ has the Unicode code point U+1E00.
![[Pasted image 20250421084741.png]]
## TL;DR

- Any character can be encoded in many different bit sequences
- and any particular bit sequence can represent many different characters, depending on which encoding is used to read or write them.
	- The reason is simply because different encodings use different numbers of bits per characters and different values to represent different characters.

|bits|encoding|characters|
|--:|---|---|
|`11000100 01000010`|Windows Latin 1|ÄB|
|`11000100 01000010`|Mac Roman|ƒB|
|`11000100 01000010`|GB18030|腂|

|characters|encoding|bits|
|---|---|--:|
|Føö|Windows Latin 1|`01000110 11111000 11110110`|
|Føö|Mac Roman|`01000110 10111111 10011010`|
|Føö|UTF-8|`01000110 11000011 10111000 11000011 10110110`|

## Misconceptions, confusions and problems
### Why in god's name are my characters garbled?!

```
ÉGÉìÉRÅ[ÉfÉBÉìÉOÇÕìÔÇµÇ≠Ç»Ç¢
```
**There's one and only one reason for it:**
Your text editor, browser, word processor or whatever else that's trying to read the document is assuming the wrong encoding.

**The computer always needs to be told what encoding some text is in. Otherwise it can't know**
### My document doesn't make sense in any encoding!
The document has mostly likely been converted incorrectly at some point.
For example, we decoded it using a wrong encoding and then we saved it in another encoding like UTF-8.

### So how to handle encodings correctly?
- _Know_ what encoding a certain piece of text, that is, a certain byte sequence, is in, then interpret it with that encoding.
- If you need to convert from one encoding to another, do so cleanly using tools that are specialized for that.
### Unicode all the way
Precisely because of that, there's virtually no excuse in this day and age not to be using Unicode all the way.

### UTF-8 and ASCII
The ingenious thing about UTF-8 is that it's binary compatible with ASCII
- All characters available in the ASCII encoding only take up a single byte in UTF-8 and they're the exact same bytes as are used in ASCII.
- Any character not in ASCII takes up two or more bytes in UTF-8.

[[How To Encode in UTF-8]]