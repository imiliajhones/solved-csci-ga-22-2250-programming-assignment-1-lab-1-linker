Download Link: https://assignmentchef.com/product/solved-csci-ga-22-2250-programming-assignment-1-lab-1-linker
<br>
You are to implement a <strong>two-pass linker</strong> and submit the <strong>source </strong>code, which we will compile and run. Submit your source code together with a Makefile <strong>as a ZIP file</strong> through NYU Classes assignment. <strong>Please do <u>not</u> submit inputs or outputs.</strong> Your program must take one input parameter which will be the name of an input file to be processed. All output should go to standard output. The languages of choice for labs are C/C++ only. You may develop your lab on any machine you wish,     but you must ensure that it compiles and runs on the NYU system assigned to the course (<em>linserv)</em> where it will be graded [https://cims.nyu.edu/webapps/content/systems/resources/computeservers]. It is your responsibility to make sure it executes on those machines. Note, when you work on <em>linserv1</em> the default GCC/G++ compiler is v4.8.5. If you use advanced features there are later versions available. To do so see at the end, but make sure your submission is clear which version it depends on, so the Graders can switch. We realize you code on your own machine and transferring to <em>linserv1</em> machine exposes occasionally some linker errors. In that case use static linking (such as not finding the appropriate libraries at runtime), though hopefully that is now resolved with the “module” approach discussed at the end.




In general, a linker takes individually compiled code/object modules and creates a single executable by resolving external symbol references (e.g. variables and functions) and module relative addressing by assigning global addresses after placing the modules’ object code at global addresses.




Rather than dealing with complex x86 tool chains, we assume a target machine with the following properties: (a) word addressable, (b) addressable memory of 512 words, and (c) each word consisting of up to 4 decimal digits. [ <em>I know that is a really strange machine, but I once saw an UFO too.</em> ].




The input to the linker is a file containing a <strong>sequence of tokens</strong> (symbols and integers and instruction type characters). Don’t assume tokens that make up a section to be on one line, don’t make assumptions about how much space separates tokens or that lines are non-empty for that matter or that each input conforms syntactically.  Symbols always begin with alpha characters followed by optional alphanumerical characters, i.e.[a-Z][a-Z0-9]*. Valid symbols can be up to 16 characters. Integers are decimal based. Instruction type characters are (I, A, R, E). Token delimiters are ‘ ‘, ‘t’ or ‘
’.




The input file to the linker is structured as a series of “object module” definitions.

Each “object module” definition contains three parts (in fixed order): definition list, use list, and program text.




<ul>

 <li><strong><em>definition list</em></strong> consists of a count <em>defcount</em> followed by <em>defcount</em> pairs (S, R) where S is the symbol being defined and R is the relative word address (offset) to which the symbol refers in the module.</li>

 <li><strong><em>use list</em></strong> consists of a count <em>usecount </em>followed by <em>usecount </em>symbols that are referred to in this module. These could include symbols defined in the <em>definition list</em> of any module (prior or subsequent or not at all).</li>

 <li><strong><em>program text</em> </strong>consists of a count <em>codecount </em>followed by <em>codecount</em> pairs (<strong>type, instr</strong>), where <em>instr</em> is an upto 4-digit instruction (integer) and <em>type</em> is a single character indicating <strong>I</strong>mmediate, <strong>A</strong>bsolute, <strong>R</strong>elative, or <strong>E</strong> <em>codecount</em> is thus the length of the module.</li>

</ul>




An instruction is composed of an integer that is separated into an opcode (op/1000) and an operand (op mod 1000). The opcode always remains unchanged by the linker. (Since the instruction value is supposed to be 4 or less digits, read an integer and ensure opcode &lt; 10, see errorcodes below). The operand is modified/retained based on the instruction type in the <em>program text</em> as follows:

(<strong>I</strong>) an immediate operand is unchanged;

(<strong>A</strong>) operand is an absolute address which will never be changed in pass2; however it can’t be “&gt;=” the machine size (512); (<strong>R</strong>) operand is a relative address which is relocated by replacing the relative address with the absolute address of that relative address after the modules global address has been determined.

(<strong>E</strong>) operand is an external address which is represented as an index into the uselist. For example, a reference in the program text with operand K represents the Kth symbol in the use list, using 0-based counting, e.g., if the use list is ‘‘2 f g’’, then an instruction ‘‘E 7000’’ refers to f, and an instruction ‘‘E 5001’’ refers to g. You must identify to which global address the symbol is assigned and then replace the operand with that global address.




The linker must process the input twice (that is why it is called two-pass) (to preempt the favored question: “Can I do it in one pass?” → NO). <strong>Pass One</strong> parses the input and verifies the correct syntax and determines the base address for each module and the absolute address for each defined symbol, storing the latter in a symbol table. The first module has base address zero; the base address for module X+1 is equal to the base address of module X plus the length of module X (defined as the number of instructions in a module. The absolute address for symbol S defined in module M is the base address of M plus the relative address of S within M. After pass one print the symbol table (including errors related to it (see rule2 later)). Do not store parsed tokens, the only data you should and need to store between passes is the symboltable.

<strong>Pass Two</strong> again parses the input and uses the base addresses and the symbol table entries created in pass one to generate the actual output by relocating relative addresses and resolving external references.

You must clearly mark your two passes in the code through comments and/or proper function naming.




<strong>Other requirements: error detection, limits, and space used. </strong>

<strong> </strong>

To receive full credit, you must check the input for various errors. All errors/warnings should follow the message catalog provided below. We will do a textual difference against a reference implementation to grade your program. Any reported difference will indicate a non-compliance with the instructions provided and is reported as an error and results in deductions. You should continue processing after encountering an error/warning (other than a syntax error) and you should be able to detect multiple errors in the same run.

<ol>

 <li>You should stop processing if a syntax error is detected in the input, print a syntax error message with the line number and the character offset in the input file where observed. A syntax error is defined as a missing token (e.g. 4 used symbols are defined but only 3 are given) or an unexpected token. Stop processing and exit.</li>

 <li>If a symbol is defined multiple times, print an error message and use the value given in the first definition. Error message to appear as part of printing the symbol table (following symbol=value printout on the same line)</li>

 <li>If a symbol is used in an E-instruction but not defined, print an error message and use the value zero.</li>

 <li>If a symbol is defined but not used, print a warning message and continue.</li>

 <li>If an address appearing in a definition exceeds the size of the module, print a warning message and treat the address given as 0 (relative to the module).</li>

 <li>If an external address is too large to reference an entry in the use list, print an error message and treat the address as immediate.</li>

 <li>If a symbol appears in a use list but it not actually used in the module (i.e., not referred to in an E-type address), print a warning message and continue.</li>

 <li>If an absolute address exceeds the size of the machine, print an error message and use the absolute value zero.</li>

 <li>If a relative address exceeds the size of the module, print an error message and use the module relative value zero (that means you still need to remap “0” that to the correct absolute address).</li>

 <li>If an illegal immediate value (I) is encountered (i.e. more than 4 numerical digits, aka &gt;= 10000), print an error and convert the value to 9999.</li>

 <li>If an illegal opcode is encountered (i.e. more than 4 numerical digits, aka &gt;= 10000), print an error and convert the &lt;opcode,operand&gt; to 9999.</li>

</ol>




The following exact limits are in place.

<ol>

 <li>Accepted symbols should be upto 16 characters long (not including terminations e.g. ‘ ’), any longer symbol names are erroneous.</li>

 <li>a uselist or deflist should support 16 definitions, but not more and an error should be raised.</li>

 <li>number instructions are unlimited (hence the two pass system), but in reality they are limited to the machine size.</li>

 <li>Symbol table should support at least 256 symbols (reference program supports exactly 256 symbols).</li>

</ol>




There are several sample inputs and outputs provided as part of the sample input files / output files (see NYU Classes). The first (<em>input-1) </em>is shown below and the second (<em>input-2) </em>is a re-formatted version of the first. They both produce the same output as the input is token-based and hence present the same content to the linker. Some of the input sets contain errors that you are to detect as described above. Note that when you have questions regarding errors, please first make sure the structure of the input is not messing with your mind.

We will run your lab on these (and other) input sets. Please submit the SOURCE code for your lab, together with a README and a Makefile (required) describing how to compile and run it. Your program must accept one command line argument giving the name of the input file (which must accept a full path as we are running this through a grading harness);




<ul>

 <li>xy 2</li>

 <li>z xy</li>

 <li>R 1004 I 5678  E 2000  R 8002  E 7001 0</li>

</ul>

1 z

<ul>

 <li>R 8001 E 1000  E 1000  E 3000  R 1002  A 1010</li>

</ul>

0

<ul>

 <li>z</li>

 <li>R 5001 E 4000</li>

 <li>z 2</li>

 <li>xy z</li>

 <li>A 8000 E 1001  E 2000</li>

</ul>




<strong>Your output is expected to strictly follow this format (with exception of empty lines):  </strong>

<strong> </strong>

Symbol Table xy=2 z=15




Memory Map

000: 1004

001: 5678

002: 2015

003: 8002

004: 7002

005: 8006

006: 1015

007: 1015

008: 3015

009: 1007

010: 1010

011: 5012

012: 4015

013: 8000

014: 1015

015: 2002




The following output is heavily annotated for clarity and class discussion. Your output is <strong><u>not</u></strong> expected to be this fancy. It should help you understand the operation and mapping of symbols etc.




Symbol Table xy=2 z=15

Memory Map

+0

0:      R 1004        1004+0 =  1004

1:      I 5678                  5678

2: xy:  E 2000 -&gt;z              2015

3:      R 8002        8002+0 =  8002

4:      E 7001 -&gt;xy             7002

+5

0:      R 8001        8001+5 =  8006

1:      E 1000 -&gt;z              1015

2:      E 1000 -&gt;z              1015

3:      E 3000 -&gt;z              3015

4:      R 1002        1002+5 =  1007

5:      A 1010                  1010 +11

0:      R 5001        5001+11=  5012

1:      E 4000 -&gt;z              4015

+13

0:      A 8000                  8000

1:      E 1001 -&gt;z              1015

2 z:    E 2000 -&gt;xy             2002







Note that even an empty program should have the “Symbol Table” and “Memory Map” line.




We grade by using a “diff –b –B -E” against the reference output created by my test program using a grading harness.

Inputs will be the ones provided in NYU Classes as well as other once and will be checked for several of the error conditions.

It is imperative that you match the output as generated by the ref program to allow for automated testing.

For a test case to pass you must catch ALL warning/errors and generate the correct output for a given input file.




Example:




Symbol Table

X21=3

X31=4




Memory Map

000: 1003

001: 1003

002: 1003

003: 2000  Error: Absolute address exceeds machine size; zero used 004: 3000  Error: Relative address exceeds module size; zero used




Warning: Module 3: X31 was defined but never used




Parse errors should abort processing.

Error messages must be following the instruction as shown above.

Warnings message locations are defined further down. Module counting starts at 1.




I provide in C the code to print parse errors, which also gives you an indication what is considered a parse error.




<table width="509">

 <tbody>

  <tr>

   <td width="240">void __parseerror(int errcode) {     static char* errstr[] = {</td>

   <td width="269"></td>

  </tr>

  <tr>

   <td width="240">        “NUM_EXPECTED”,</td>

   <td width="269">// Number expect</td>

  </tr>

  <tr>

   <td width="240">        “SYM_EXPECTED”,</td>

   <td width="269">// Symbol Expected</td>

  </tr>

  <tr>

   <td width="240">        “ADDR_EXPECTED”,</td>

   <td width="269">// Addressing Expected which is A/E/I/R</td>

  </tr>

  <tr>

   <td width="240">        “SYM_TOO_LONG”,</td>

   <td width="269">// Symbol Name is too long</td>

  </tr>

  <tr>

   <td width="240">        “TOO_MANY_DEF_IN_MODULE”,</td>

   <td width="269">// &gt; 16</td>

  </tr>

 </tbody>

</table>

“TOO_MANY_USE_IN_MODULE”,     // &gt; 16

“TOO_MANY_INSTR”,             // total num_instr exceeds memory size (512)

};

printf(“Parse Error line %d offset %d: %s
”, linenum, lineoffset, errstr[errcode]); }

(Note: line numbers start with 1 and offsets in the line start with 1, offsets should indicate the first character offset of the token that is wrong, not the last). Tabs count as one character.

Error messages have the following text and should appear right at the end of the line you are printing out




“Error: Absolute address exceeds machine size; zero used”                                                       (see rule 8)

“Error: Relative address exceeds module size; zero used”                                                          (see rule 9)          “Error: External address exceeds length of uselist; treated as immediate”                                 (see rule 6)

“Error: %s is not defined; zero used”                         (insert the symbol name for %s)              (see rule 3)

“Error: This variable is multiple times defined; first value used”                                               (see rule 2)

“Error: Illegal immediate value; treated as 9999”                                                                               (see rule 10)

“Error: Illegal opcode; treated as 9999”                                                                                                (see rule 11)




Warning messages have the following text and are on a separate line.




“Warning: Module %d: %s too big %d (max=%d) assume zero relative
”                                (see rule 5)

“Warning: Module %d: %s appeared in the uselist but was not actually used
”                        (see rule 7)

“Warning: Module %d: %s was defined but never used
”                                                          (see rule 4)




Locations for these warnings are:

Rule 5: to be printed after each module in pass1

Rule 7: to be printed after each module in pass2 (so actually interspersed in the memory map (see  out-9) Rule 4: to be printed after pass 2 (i.e. all modules have been processed)   ( see out-3).




<strong>Parse Error Location: </strong>




Parse errors are to be located at the first character of the wrong token, or if end-of-file is reached at the end of file location. There is one special case when the eof ends with `
’. My expectation is that the line number reported actually exists in the file and that an editor (e.g. vi) can jump to it. In this particular case the linenumber to be reported is the last line read and the last position of that line, not the next line and offset 1 (see <em>input-12 </em> for an example). The error is at the very last position of the line. Reason is when one does a linecount on the file  (“wc –l input-12”)  it shows 3.

<strong> </strong>

<strong>Hint: </strong>for each parser error and warning error mentioned above you should have code checking for that.




<strong>Writing a parser and other important information. </strong>

<strong> </strong>

Here are some hints on writing a parser. In general one could write this parser using   lex/yacc, however this is so simple that I suggest writing a simple recursive decent parser, in particular since you have to parse the input twice. In lex and yacc you also need to handle the error locations to conform to the specification. It would have a structure similar to this (all pseudoCode). This structure is simply copied twice and the actions taken in each path are different.




ReadFile() {  while (!eof) { createModule() = { readDefList(); readUseList(); readInstList(); } }

ReadDefList() { numDefs=readInt(); for (i=0;i&lt;numDefs(); i++) { readDef(); }

ReadDef() { str= readSym(); val=readInt(); createSymbol(str,val); }




Or  even better and more concise.




Pass1() {       while (!eof) { createModule(); int defcount = readInt(); for (int i=0;i&lt;defcount;i++) {                  Symbol sym = readSym();                  int val = readInt();                  createSymbol(sym,val);

}

int usecount = readInt();             for (int i=0;i&lt;usecount;i++) {               char addressmode = ReadIEAR();               int  operand = ReadNumber();

:  // various checks             }       : }




Of course you will have to deal with the errors etc.

The first pass does syntax and early error checking and creates the symbol table, the second pass performs additional error checking and instruction transformation. In order to go over the file a second time reset the file pointer or close and reopen the file. You should not store tokens or deflist or uselist or instructions during the first pass in order to pass to pass2. Only metadata, such as list of modules and symbol table can be stored.




<strong><u>To start off</u></strong>, write a token parser that simply parses the input, prints the token and the position in the file found and finally prints the final position in the file (as you will need that for error reporting) and verify it correctly recognizes tokens and line and lineoffsets. The functions you need to study for this are getline() or fgets()  (reads a full line into a buffer) or C++ equivalent and strtok(), which tokenizes input buffers. Please use Linux’s build in help :   “man strtok” and understand how a new line is seeded and continued in subsequent calls.

Once you have the getToken() function written and verified that your token locations are properly reported, then layer the readInt(), readSymbol() , readIAER() functions on top of it and build your real parser. Once you have the first pass written, reset the input file, copy the pass1() to pass2() and rewrite. Note, in pass2 certain error checking doesn’t have be done anymore. Instead you are now rewriting the instructions.




<strong>Writing a Makefile </strong>

<strong> </strong>

A makefile should be named either “makefile” or “Makefile”. <em>make</em> will look for either of these files during building.

Here is a good introduction: <a href="http://www.cs.colby.edu/maxwell/courses/tutorials/maketutor/"><em>http://www.cs.colby.edu/maxwell/courses/tutorials/maketutor/</em></a> <u> with more sophisticated ones.</u>




The simplest Makefile you can think of contains how it builds the executable and how it cleans up (here it assumes one source file only and builds the <em>linker </em>executable):

linker: linker.c

g++ -g linker.cpp -o linker




clean:

rm -f linker *~




Here is a more sophisticated one assumes multiple inputfiles:  file1.c .. file3.c  and allows to specify the compiler version




CFLAGS=-g        # -g = debug,  -O2 for optimized code    CPPFLAGS for g


linker: file1.c file2.c file3.c

$(CC)  $(CFLAGS) –o linker file1.c file2.c file3.c




There are certainly more sophisticated once to write.

<strong>Testing your program before submission: </strong>




As part of the labsamples.tar in NYU Classes you will see a <em>runit.sh</em> and a <em>gradeit.sh</em> script (the same we will use for grading albeit with more inputs). Understand the script what it does. If you can’t pass this script, things will be flagged during the grading process as well. Don’t assume at just because you pass for these inputs, it will pass for all inputs. Carefully go through the rules above and ensure you have covered each rule with some code (if/then/else) and it is at the right location.




Execute as follows:

Create yourself a directory &lt;your-outdir&gt; where your outputs will be created. <em><u>&gt; cd labsamples</u> </em>

&gt; <em>./runit.sh &lt;your-outdir&gt; &lt;your-executable and optional arguments&gt;</em>          # make your output directory different

The above will create all the outputs for the available inputs (1-19)

&gt; <em>./gradeit.sh  <strong>.</strong>  &lt;your-outdir&gt;</em>        #  note the 2<sup>nd</sup> argument is a DOT for the local directory where the reference output is.




The above will compare the reference outputs (out-[1-19]) with the ones created with your program and tell you how many you got right and which one are wrong

There will be a file called &lt;your-outdir&gt;/LOG that contains which cases you got wrong and where the differences are. If you want to analyze further please run the “diff –b –B –E” by hand on a particular output pair.




Please only submit your source code and if necessary instructions.

The reference program used during grading is located on <em>/home/frankeh/Public/linker</em> on cims systems. So feel free to try it in order to answer any questions you might have on what is expected for a particular input.




<strong>Switching compiler versions on courses2/3: </strong>

<strong> </strong>

gcc –v <strong>or</strong> g++ -v  # will tell you the currently active gcc version module avail gcc#  This will show you the list of all available modules module load gcc-6.3.0  # loads indicated gcc version (must be in avail list) module unload gcc-6.3.0  # reverts back to default one

# If you want to load a different version – first unload otherwise they stack