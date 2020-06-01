File: readme.txt
Author: Zolboo Chuluunbaatar
----------------------

ATM
---
1a)

Defect description: when subtracting signed int from unsigned int, the signed int will be automatically converted into an unsigned int (which is a very large number in the case of our assignment) and the subtraction will for sure be greater than the minimum balance $50.
-------------------

Test case explanation (put test case in custom_tests!): ./samples/atm.c 300 zolboo would leave the balance -193. 
-------------------------------------------------------

Recommended fix: (signed)(cust->balance - amount) casting for the subtraction would fix the issue. 
----------------

1b)

Defect description:  strcmp(a,b) returns (positive) if the ASCII value of the first unmatched character is greater than second. Since we inputted a bogus name which is not equal to all of the names, the very last $eax address after the 449th strcmp would contain the return value from find_account(). So in order to lure the system, we choose some bogus word which is lexicographically greater than the very last account holder "zwzhang".
-------------------

Test case explanation (put test case in custom_tests!):  ./samples/atm 40 zzzzzz would give access to the 3rd person (I hope it is one of the TAs).
-------------------------------------------------------

Recommended fix: Inside find_account, we need to return a negative number when strcmp is not equal to zero. Therefore, in lookup_by_name, the $eax return value from find_account would be negative in the case of bogus name and would return NULL as desired. 
----------------


1c)

Defect description: We allocated on the stack where we would store the path[sizeof(BANK_DATA_DIR) + sizeof(filename) + 1] string. when we enter a long filename (account number) such that this would overwrite the return address 0x400be7 from

       callq 400b44 <read_secret_passcode>
to return address 0x400c0c, the function read_secret_passcode would return to
   	  return db->array + account_num;
in the lookup_by_number. This allows us to skip the following lines:

       int entered_code = strtol(arg2, NULL, 0);
       if (saved_code < 0 || saved_code != entered_code) {
       	  return NULL;
       }


-------------------

Test case explanation (put test case in custom_tests!): I looked up where in the %RSP stack, the return address 0x400be7 was and filled up all the addresses in front of it with placeholder characters.
300 $(python -c 'print "0" + "A"*30 +"\x00\x0c\x0c\x40"') 0
The first character is "0" because we want to steal from VAULT's account number 0. 
-------------------------------------------------------

Recommended fix:  instead of saving the path name in a buffer with only limited size, make sure that the path string is able to hold all the account number. (we could read *filename until the name is null terminated and store the length and use it for the path string's size.)
----------------



Bomb
----
2a)


I set breakpoints right after TEST, JE, JSE, JBE etc. Also I set breakpoints right at the line where there is
0xABCD      <explode_bomb>
When I know that in GDB, I reach the breakpoints at 0xABCD, I surely know that the bomb will explode if I run further.

2b)
	000000000040198b <level_1>:
	40198b:	 48 83 ec 08		sub    $0x8,%rsp
	40198f:	 e8 44 fd ff ff       	callq  4016d8 <get_input_line>
	401994:	 48 89 c6             	mov    %rax,%rsi
	401997:	 bf 58 27 40 00       	mov    $0x402758,%edi
	40199c:   e8 5f f6 ff ff        callq  401000 <strcmp@plt>
	4019a1: 85 c0                   test   %eax,%eax
	4019a3:  74 05                  je     4019aa <level_1+0x1f>
	4019a5:	  e8 62 fc ff ff        callq  40160c <explode_bomb>
		

Move $0x402758 to %edi (first argument) and  Move get_input_line to %rsi (second argument)
strcmp(first argument, second argument) is comparing the string stored at the memory address at 0x402758 with the user input line.
The string stored at 0x402758 address in gdb is:
Computers are useless. They can only give you answers.
We can reference %edi instead of $rdi because a pointer to character is 8 bytes, i.e, 32 bits. So %edi register could hold a pointer to a character (string). 



2c)


Arithmetic and logical instructions like XOR set flags for conditional jump instructions to use. For example, if you XOR value a and b, if the result is 0, then the ZF (Zero Flag) is set. The JLE instruction looks at the ZF flag, the SF (Sign Flag), and the OF (Overflow Flag). JLE would jump if either the ZF flag is set or SF is not equal to OF.




2d)

Winky exits when the user input line's string part's substring starting at the second character matches
one of the 64 words we read from the wordlist. 
count <- 0
while( true ) {
    match <- strcmp(current_word, input_string + 1)
    if (match == 0) break;
    count <- count + strcspn(current_word, input_word);
    go to the next word in the wordlist
}
// count has to be 48, and I found out that 'rdashpots' satisfies all the condition. 



2e)


    mov    $0x10,%edx
    lea    0x8(%rsp), %rsi
    mov    %r12, %rdi
    callq  401110 <strtoul@plt>
 These 4 lines are concerned with strtoul c function. Which has the following syntax:

unsigned long int strtoul(const char *str, char **endptr, int base)

So 0x8(%rsp) is a double pointer to char. The size should be 8 bytes.
From level 4's function, we read a line input and use the reference to that line for strtoul's first argument.
So 0x8(%rsp) is being populated with a pointer to some string after reading some hex from the user input line.

for example, if the user input line was
    -0xe1 0x1 0xf 0x1f 0x3f 0x7f 0xff 0x1ff 0x3ff 0x7ff 0xfff
in read_array strtoul would read from the second string which is
   "0x1 0xf 0x1f 0x3f 0x7f 0xff 0x1ff 0x3ff 0x7ff 0xfff"

So the hex number returned by strtoul is 0x1, and the 0x8(%rsp) would store a pointer to string
   "0xf 0x1f 0x3f 0x7f 0xff 0x1ff 0x3ff 0x7ff 0xfff".

The while loop only loops less than 0x14 = 20 times, but it has to be a number between 8 and 19.
The while loop also stops when we read two consecutive similar hex values. 



2f)


mycmp is using  kbnlv(some argument) function which gives you the argument's # of 1's at the end in its binary representation.
For instance if the number was 0x134f, the kbnlv would return 4 because F is 1111 in binary.


0x1 0xf 0x1f 0x3f 0x7f 0xff 0x1ff 0x3ff 0x7ff 0xfff

The array's each element is being compared to the next element using mycmp function.
The number of 1's in their binary representation is in ascending order.

Level_4 uses mycmp for checking if the array is in a sorted order in some binary representation sense.
Then we are using bsearch to find '0xb30614ff'.



