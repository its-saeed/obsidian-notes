[source](https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/)
#unicode #string #utf-8 #ascii #utf-16
## ASCII
- In the original system of 128 characters, the binary codes were 7 bits long.
- Today, ASCII uses 8-bit codes to maintain compatibility with modern computers that use 8-bit bytes.
- The extra bit in these codes is usually set to 0.
- Codes 32-127 are all printable ASCII characters, representing letters, numbers, punctuation marks, plus some special characters like ^, [, \, ~
- Codes below 32 were called _unprintable_ and were used for control characters.
	- Like 7 for beep
![[Pasted image 20250421070612.png]]

- Because bytes have room for up to eight bits, lots of people got to thinking, “gosh, we can use the codes 128-255 for our own purposes.”
- The trouble was, _lots_ of people had this idea at the same time, and they had their own ideas of what should go where in the space from 128 to 255.
- Eventually this OEM free-for-all got codified in the ANSI standard.
	- Everybody agreed on what to do below 128, same as ASCII
	- For chars >= 128 code pages were invented
		- Based on where you lived, you had a code page
		- But for example, Hebrew and Greek on the same computer was a complete impossibility
	- In Asia DBCS was used because some alphabets had thousands of characters
		- Double byte character set
		- in which _some_ letters were stored in one byte and others took two.
	- As soon as you start to share your doc with other part of the world it doesn't work anymore.
## Unicode
Unicode was a brave effort to create a single character set that included every reasonable writing system on the planet.

☠️**Incorrect belief**: Unicode is simply a 16-bit code where each character takes 16 bits and therefore there are 65,536 possible characters. **This is not, actually, correct.**