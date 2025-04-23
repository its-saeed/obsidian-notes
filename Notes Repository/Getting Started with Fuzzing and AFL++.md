[source](https://comp.anu.edu.au/courses/comp2120/tutorials/06-fuzzing/)

#afl #fuzzing #test
# Fuzzing
Fuzzing is an automated program testing technique where:

1. We feed program with malformed inputs
2. Monitor program for crashes
## Input Generation
### Mutational
It requires a set of seed inputs, from where they can randomly mutate these inputs.
If the seed inputs are `hi` and `pass`, a sample set of inputs could be
```
hi
hihi
hi123
_2hi_
pass
passhi
...
```
**Advantages**
- Requires no a prior knowledge about the program structure
- Very fast in generating new inputs

**Disadvantages**
- Generated inputs may fail parsing stage of the main program, so many “progress” may not be achieved
### Generational
Generate seeds from a model/grammar of the target program’s input language.
For example, if we define the grammar for valid C in BNF form for fuzzer, it could generate the following inputs
```c
void f() {}
void fx(){  ;}
void f(int a) { a = 3333333333333333333333333; }
....
```
**Advantages**
- Generates more “valid” inputs according to the underlying input grammar
- Does not require an initial set of seeds

**Disadvantages**
- Requires domain-specific knowledge of target to generate grammar
## Fuzzing categories
1. **Blackbox**:
	- No knowledge of the program structure
	- Treating the target as a black box
	- Leads to high throughput, but low progress
2. **Greybox**:
	- Leverages light-weight program instrumentation to extract + monitor runtime information.
	- This runtime information is used as feedback to guide input.
	- The specific runtime information includes control flow coverage (which conditional jumps were taken), and data-flow coverage (how data is manipulated by the target program) generation
3. **Whitebox**:
	- Apply heavy-weight program analysis to collect runtime information.
	- They collect a lot of information to better guide the fuzzer towards more coverage. 
	- However, they have a huge overhead when it comes to runtime. 
# American Fuzzy Lop
AFL++ is the successor to AFL, a very popular *mutational*, *coverage-guided*, *greybox* fuzzer.

**coverage-guided fuzzer** means:
	
- It gathers coverage information for each mutated input in order to discover new execution paths and potential bugs. 
- When source code is available AFL can use instrumentation, inserting function calls at the beginning of each basic block (functions, loops, etc.)
- To enable instrumentation for our target application, we need to compile the code with AFL's compilers.

## How It Works
1. First instrumenting the binary (i.e. adding specific machine code into a program without changing it’s behavior) by doing static analysis to collect runtime logs. 
2. Then run this instrumented binary with some seed inputs
3. Based on the behavior of the input results it generates further new inputs.
![[Pasted image 20250416073914.png]]
## How To Use AFL++
### Create AFL++ Docker Container[](https://comp.anu.edu.au/courses/comp2120/tutorials/06-fuzzing/#create-afl-docker-container)

1. To have the compiled version directly, you can pull the image directly from Dockerhub:

```sh
docker pull aflplusplus/aflplusplus
```
2. To launch the Docker image:
```sh
$ docker run -ti -v $HOME:/home aflplusplus/aflplusplus
$ export $HOME="/home"
```
3. You should now be in the `/AFLplusplus` directory
4. Navigate to the directory of the test program
5. Create a build directory and change into it
    
```sh
$ mkdir build && cd build
```
    
6. Add AFL++ tooling to the compiler for your executable (note that `CC` and `CXX` are just aliases we setup to pass to `cmake` which builds the result):
    
```sh
CC=/AFLplusplus/afl-clang-fast CXX=/AFLplusplus/afl-clang-fast++ cmake ..
```

`afl-clang-fast/++` is just one example of compilers you can use with AFL++ - different compilers have different advantages.

7. Use the `make` build system to compile the executable:
```sh
make
```
8. Now, we will be using the following flags to run `AFL++`:
	- `-i` - `AFL++` uses a seeds directory which contains the list of initial sample inputs to test, and based on the program control flow, update it’s inputs. We will create the `seeds/` directory alongside `build/`, and populate it with a random input with the `dd` utility
	```sh
	    $ cd ..
	    $ mkdir seeds && cd seeds
	    $ dd if=/dev/urandom of=seed_i bs=64 count=10
	```
	
	- `-s` - AFL uses a non-deterministic testing algorithm, so two fuzzing sessions are never the same. You can make your test deterministic by using `s` like `-s 123`
	- `-o` - the output directory to have crashes, hangs, queues, etc

	** The final command to run is as follows**
	
	```sh
	/AFLplusplus/afl-fuzz -i <full path for the seeds directory> -s 123 -o out -m none -- ./simple_crash
	```
	
	**After a while, we would see some crashes**
	
	Go to the `out/default/crashes` folder, and analyze the respective crashes starting from `id:0000...`.

9. You can use `gdb`, and from within that, one can use input redirection to run the crashed input. 
	- For example `r < out/default/crashes/<crash_id>`. It will show the emitted signal on crash.
	- To further debug with more options, also add `set(CMAKE_BUILD_TYPE Debug)` to the `CMakeLists.txt` 
	- You can add more compiler instrumentations like `-fsanitize=address`

How to replicate xpdf bug using afl++ [[2.Exercise 1 - Xpdf]]