---
layout: page
title: Unicode and UTF encodings explanation
---

### Contents

- Purpose of the document.
- Coding history. ASCII.
- The history of Unicode creation. UCS and UTF.
- Basic coding principles in Unicode.
- Translation of a character into machine code. Definition of coding.
- UTF-8. Coding algorithm.
- UTF-16. Coding algorithm.
- Byte order. Unicode byte order marker.
- General information about UTF-32.
- Comparison of UTF-8 and UTF-16. Findings.

### Purpose of the document

After reading the document, the reader should have a clear understanding of the Unicode character encoding standard, and also know how Unicode characters are translated into machine code. Also, this document will describe the encoding principles in UTF-8 and UTF-16, briefly UTF-32, the main differences between the encodings, the pros and cons of each.

The document is designed for technical specialists with an understanding of binary and hexadecimal number systems.

### Coding history. ASCII.

Coding is the presentation of an informational message in the form of a code. The simplest coding method, known since ancient times, is a signal fire. 

The first tool that can convey the alphabet and punctuation is Samuel Morse code. The term “code table” appears in the Morse code. This is a table of correspondence between symbols and their codes. 

The evolution of Morse code is the Bodo code. 5 pulses were used to transmit each character. In fact, the Bodo code is the world's first binary information coding system. The size of each character was fixed at 5 bits.

In 1936, the United States created the ASCII (American Standart Code for Information Interchange) coding table. ASCII is a table of correspondence of characters to their codes, size 8 by 16 cells.

The ASCII table, with the exception of Latin letters and punctuation, contains control characters, such as the beginning of the text (SOT), line feed (LF), the end of the transmission (EOT), and others - total 33 characters. 

In ASCII, the story of the development of coding could end if humanity spoke only English and did not use symbols like ¿- “Inverted Question Mark”. However, we know that there are thousands of languages ​​in the world, and many special characters. Came up a logical idea - to combine all existing alphabets and all special characters into one table. 

So the Unicode standard appeared.

### The history of Unicode creation. UCS and UTF.

Unicode is a character encoding standard that includes almost all the written languages ​​of the world. The standard was proposed by the Unicode Consortium (Unicode Inc.) in 1991. 

The standard consists of two parts: Universal Character Set, UCS, and Universal Transformation Format, UTF - a set of encodings.

In simple terms, UCS is a table where each character has its own hexadecimal code, prefixed with U+. 

UTF is an algorithm for translating the hexadecimal code of UCS to binary. In other words, this is a translation of the intermediate U+ code into binary language that the computer understands.

### Basic coding principles in Unicode. 

Symbols in Unicode have a specific name - “code point”. For example, the Latin letter E will be represented by the code point U+ 0045. It is worth noting that the lowercase letter “e”, as well as the Cyrillic characters “Е” and “е” will have their own codes in the table - despite the same letter inscription.

Each code point also has additional characteristics, such as:

- HTML and CSS word codes.
- The section of the Unicode table in which the code point is located.
- Name in Unicode (E - Latin Capital Letter E).

All code points form a set called “code space”. This space consists of 1,114,112 code points, of which 128,237 are occupied - that is only 12%. 

Below is a map of the Unicode code space. Each small field (square) of the card contains 16 * 16 = 256 code points. In turn of, each large field contains 65536 code points. The total number of large fields is 17. Unicode also reserves “private points” - fields for the internal needs of applications.

Figure 1. Unicode code points location map.

Blue points in the table indicate already defined points, green - private code points. The largest space is the free fields, they are marked white.

The first large field (upper left square) is used most often. It is called the “Basic Multilingual Plane (BMP)”, and contains almost all the characters that are used in modern texts. BMP includes Latin letters, Korean, Japanese and Chinese alphabets, Cyrillic, and other languages. It is easy to guess that the BMP is used most often. 

The second field contains more specific languages, for example, Egyptian hieroglyphs, as well as emoji. The third field contains rare Chinese characters. 

Figure 2. Main Unicode fields.

### Translation of a character into machine code. Definition of coding.

We found that each character in Unicode is represented by a code word in the range U + 0000 - U + 10FFFF. Our next task is to understand how the hexadecimal code is translated into a binary, understandable to the computer. UTF encoding formats are used for this conversion. They are also called “encodings”.

To represent the code point in binary form, we can use 1, 2 or 4 bytes (8, 16, and 32 bits, respectively). This is a simplified definition of the encodings UTF-8, UTF-16, UTF-32.

It should be noted that in Unicode is used a Byte Order Marker (abbreviated as BOM) - special character inserted at the beginning of the text stream, indicating that the file (stream) uses Unicode. BOM is also used to specify the encoding and byte order of characters.

Consider the different encodings using the example of the Latin letter E:

```
Code point: U + 0045
Hexadecimal number: 45
Decimal number: 69 (serial number in the Unicode table)

UTF-8:                             01000101
UTF-16:                   00000000 01000101
UTF-32: 00000000 00000000 00000000 01000101
```

It is easy to see that only one working byte is required to encode a code point from the BMP. In the UTF-16 and UTF-32 encodings, all bytes except the first one are occupied by zeros and do not make sense. Why do we need all this “heavy” encodings, if there is UTF-8? 

A detailed explanation of this will be given below.


### UTF-8. Coding algorithm.

UTF-8 is the most common coding in web space. This encoding uses 1 to 4 bytes to represent a character, and is fully ASCII compatible. UTF-8 is widely used on UNIX-like systems.

The encoding algorithm in UTF-8 is divided into several stages.
First you need to find out how many bytes are needed to encode the character. The correspondence table is used for this:


|Code point range |Required number of bytes|
|-----------------|------------------------|
|00000000-0000007F|1                       |
|00000080-000007FF|2                       |
|00000800-0000FFFF|3                       |
|00010000-0010FFFF|4                       |


As we already explained above, only one byte is required for the Latin letter E, because this code point is located in BMP. 

For the “tick ✓” symbol, 3 bytes are required already, as it lies in the third range.

Next, you need to set the higher bits of the first byte to the corresponding value:

|Higher bits|Number of bytes for encoding|
|-----------|----------------------------|
|0xxxxxxx  |1                           |
|110xxxxx  |2                           |
|1110xxxx  |3                           |
|11110xxx  |4                           |

It is also necessary to determine the most significant bits in the intermediate bytes (2-4). If more than two bytes are required for encoding, the first two bits in bytes 2-4 always take the value 10xxxxxxx.


|Number of Bytes| Significant Bits| Pattern|
|---------------|-----------------|--------|
|1              |7                |0xxxxxxx|
|2              |11               |110xxxxx 10xxxxxx|
|3              |16               |1110xxxx 10xxxxxx 10xxxxxx|
|4              |21               |11110xxx 10xxxxxx 10xxxxxx 10xxxxxx|


Thus, we found out that for the “tick” symbol the first byte will be equal to 1110xxxx. The second and third bytes will begin with 10xxxxxx.

The final step in character encoding will be to set the significant bits to match Unicode characters. You need to start filling with the least significant bits of the character number, putting them in the least significant bits of the last byte, and then continue from right to left until the first byte. The free bits of the first byte are filled with zeros.

As a result, we got a binary representation for the tick symbol. These are 3 bytes:
```
11100010 10011100 10010011
```
We figured out a way to represent code points in UTF-8 encoding. Below we consider working with the UTF-16 encoding.

### UTF-16. Coding algorithm.

UTF-16 is a character encoding method in which characters are encoded by a set of double-byte words. The range of values from U+ 0000 to U+ FFFF is written in two bytes. For example, the Latin letter E will be written like this:
```
U+ 0045 	00000000 01000101
```
Note that the first byte is completely filled by zeros.

With two bytes, only 65535 code points can be represented. However, we know that there are significantly more characters in Unicode. For the representation of code points in a range greater than U + FFFF, 4 bytes are already used. The encoding of these bytes occurs using “surrogate pairs”. 

A surrogate pair is a two-byte Unicode character that, in combination with another surrogate pair, gives a Unicode character in the range above U + FFFF. Surrogate pairs are can be “upper” and “lower” (“leading” and “trailing” in other words). 

The range of the upper surrogate pairs: U+ 800 - U+ DBFF
The range of the lower surrogate pairs: U+ DC00 - U+ DFFF

Let's look at an example of how the coding of characters above U+ FFFF is made with surrogate pairs. To do this, we turn to the symbols of the book “Canon of Changes”, which was written in China around 700 BC.

The 𝌡 symbol - a tetragram of changes, will be represented in UTF-16 as follows:

D8 34 DF 21	11011000 00110100 11011111 00100001

Here, the first two bytes are the upper surrogate pair U+ D834, while the second two bytes U+ DF21 are the lower surrogate pair.

The U+ D834 and U+ DF21 characters themselves are not significant in Unicode. In other words, there is no letter or graphic representation for these characters. They are reserved specifically for the compilation of surrogate UTF-16 encoding pairs, and only work together.

Consider also the “tick” symbol from the UTF-8 encoding section. In UTF-16, this symbol will be represented as follows:

U+ 2713 	00100111 00010011
 
Please note that the “tick” character in UTF-16 encoding only takes 2 bytes, while in UTF-8 it took 3 bytes to encode.

In UTF-16 encoding, the byte order may be different. This order depends on the processor architecture, on which the data is processed. The symbol of changes 𝌡 can be represented in two versions:

D8 34 DF 21	11011000 00110100 11011111 00100001
34 D8 21 DF	00110100 11011000 00100001 11011111

The first option is called Big Endian (BE), the second - Little Endian (LE). What these formats mean and how the processor distinguishes them will be discussed in the next chapter.

Byte order. Unicode byte order marker.

Different types of processors use different byte orders:

Big Endian is the “high to low” byte order. It corresponds to the usual order of writing Arabic numbers - from left to right. This byte order is used in SPARC, Motorola, IBM processors, as well as in the TCP/IP protocol.

The order of Little Endian, in turn, is “from low to high”. For example, the number 123 in this order would be written as 321. This byte order is used in family of x86 CPU-s, as well as in USB and PCI interfaces.

The question arises - how does the processor determine which order of bytes to use to work with the information block? For this, a special character was introduced in Unicode - U+ FEFF. We already know it - this is a Byte Order Marker (BOM). 

BOM for encoding UTF-16BE: 0xFE 0xFF
BOM for encoding UTF-16LE: 0xFF 0xFE

It should be noted that in Unicode there is no U+ FFFE character. This is done to uniquely determine BOM symbol, and therefore, byte order. 

UTF-8 encoding does not use BOM to determine byte order. The standard implies adding a BOM to the beginning of a file encoded in UTF-8. This is necessary to unambiguously determine the fact that the file is UTF-8 encoded. Other encodings - UTF-16 and UTF-32 requires mandatory using BOM.

General information about UTF-32.

The Unicode standard can also be encoded in UTF-32. This encoding always uses 4 bytes to represent any Unicode character. In other words, even the “E” character from the BMP in UTF-32 will look like 00000000 00000000 00000000 01000101. 

As you can see, in this example the first three bytes are “excess”. Hence the main disadvantage of the encoding is that the text on the computer disk will take up too much space. This is especially noticeable when working with BMP code points.

The UTF-32 encoding must also have a BOM at the beginning of the text, and can be either Big Endian or Little Endian.

Comparison of UTF-8 and UTF-16. Findings.

Now we know the features of the UTF-8 and UTF-16 encodings, and we can make the following observations and conclusions:

UTF-8 is ideal for working with the Latin alphabet and ASCII control characters, because only 1 byte is required to encode these characters. If your product contains Latin letters (even mixed with other languages), UTF-8 would be a great choice.

In case if your product will work with Cyrillic, Greek and Hebrew, both encodings will use 2 bytes to represent the code point. If you plan to work mainly with Asian languages, the choice of UTF-16 will be preferable. You require only 2 bytes to write a character, instead of three in UTF-8.

If your product is designed to work with Web, or on Unix systems, choosing UTF-8 will be preferable. In turn, Windows, Java, Python, C # use UTF-16 as the internal encoding.

UTF-8 is independent of byte order. UTF-16 can have two byte order options, and it necessarily requires the use of BOM. In UTF-8, BOM is often not used at all.

Computers mainly communicate in the ASCII range, and here UTF-8 has the full advantage.

UTF-16 is better for presenting data in memory, as the byte order will not matter. Indexing by code points will be performed faster. 

The code point in UTF-8 can contain from 1 to 4 bytes, which makes it difficult to manipulate the string (for example, calculating the number of characters in a string).

UTF-8 is a self-synchronizing encoding. This means that if any byte in the string is damaged, only one character will be invalid. Rest of the string will be correctly presented.


