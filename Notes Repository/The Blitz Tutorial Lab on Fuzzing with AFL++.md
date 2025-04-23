**Initial Notes**
- As a fuzzing target this lab uses an old version of `libtiff`
- A git repository of the exercise files is available [here](https://github.com/BenH11235/libtiff-fuzzing-lab).

# Exercise 0 — Dry Run
- First, we must obtain a copy of the obsolete version 4.0.6 of libtiff.
```sh
cd projects
wget https://github.com/vadz/libtiff/archive/Release-v4-0-6.zip
unzip Release-v4-0-6.zip
cd libtiff-Release-v4-0-6/
- `./configure i686-pc-linux-gnu CFLAGS='-m32 -g' CXXFLAGS='-m32 -g'` # To compile in 32-bit target
make clean # In the case you've already done a build
make
```

**If your build failed, do the following changes**
1. Remove `-V` and `-qversion` from `configure` file
2. `sudo apt install gcc-multilib libc6-dev-i386 g++-multilib`

We're about to add our own main to the project, so move this file to stop compiler from complaining:
```sh
mv ./libtiff/mkg3states.o ./libtiff/mkg3states.o.bkp
```

## Exercise Files
- [MakeFile](https://github.com/BenH11235/libtiff-fuzzing-lab/blob/main/0_dry_run/Makefile)
- [loadtiff.c](https://github.com/BenH11235/libtiff-fuzzing-lab/blob/main/0_dry_run/loadtiff.c)

We need to change the `laodtiff.c` somehow it can load a simple tiff file successfully. But again we need to install new packages:

```sh
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install zlib1g-dev:i386

# Then make loadtiff.c
make loadtiff
./loadtiff libtiff-fuzzing-lab/samples/brain.tif

# You should see `TIFF load successful`
```

# Exercise 1 — Custom Compilation with AFL++ and ASAN
- Fuzzing is an easy concept, but is not so easy to implement.
- It involves feeding a program different inputs and observing if and how its execution path changes.
	- Easy to explain to a 5-year-old
	- But hard to explain to computers
		- “Take this program, and whenever you see an `if` statement, you should — ”
		- there is no API that can elegantly engage with the problem on that level.
		- AFL++ solves this in the hard way.

**How AFL++ Works**
- It reads the program source
- painstakingly assembles a meta-program that supports running the original program again and again, 
- trying various inputs, and reacting based on the original program behavior.
- So, AFL++ is a compiler
- Under the hood, it uses genetic approach to generating program inputs:
	- **inputs that caused the program to reach a novel state of execution are used as a baseline when trying to find further such inputs**

**What is ASAN**
- Stands for Address Sanitizer
- A Programming tool by google that detects memory corruption bugs, lib buffer overflow and user-after-free
- Similarly to AFL++, it intercedes during the compilation process
- But as opposed to AFL++ which changes the whole nature of the program, ASAN mainly makes the experience of running into memory corruption bugs cozier.
	- It add some context and color to segmentation fault outputs.
- Programs compiled with ASAN will produce pretty-prints and detailed diagnostics when they encounter a memory management error.