[source](https://stackoverflow.com/a/27939161/848068)

Suppose we want to encode this Chinese character in UTF-8:

| Description         | Value             |
| ------------------- | ----------------- |
| A Chinese character | 汉                 |
| Its Unicode value   | U+6C49            |
| 6C49 in binary      | 01101100 01001001 |
Now we need to store `01101100 01001001` in some way. One way is UTF-8. The rules of this encoding is as follows:

| 1st Byte | 2nd Byte | 3rd Byte | 4th Byte | Number of Free Bits | Maximum Expressible Unicode Value |
| -------- | -------- | -------- | -------- | ------------------- | --------------------------------- |
| 0xxxxxxx |          |          |          | 7                   | 007F hex (127)                    |
| 110xxxxx | 10xxxxxx |          |          | (5+6)=11            | 07FF hex (2047)                   |
| 1110xxxx | 10xxxxxx | 10xxxxxx |          | (4+6+6)=16          | FFFF hex (65535)                  |
| 11110xxx | 10xxxxxx | 10xxxxxx | 10xxxxxx | (3+6+6+6)=21        | 10FFFF hex (1,114,111)            |
Each byte in UTF-8 comes with a header. First you need to know how many bits you need store your original data. Here we need 16 bit because `01101100 01001001` is 16 bits. So we must you the third row of UTF-8 because it has 16 free bits.

| Header | Place holder | Fill in our Binary | Result   |
| ------ | ------------ | ------------------ | -------- |
| 1110   | xxxx         | 0110               | 11100110 |
| 10     | xxxxxx       | 110001             | 10110001 |
| 10     | xxxxxx       | 001001             | 10001001 |
So the final output is:
```
11100110 10110001 10001001
```
