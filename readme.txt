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

Defect description:
-------------------

Test case explanation (put test case in custom_tests!):
-------------------------------------------------------

Recommended fix:
----------------

Bomb
----
2a)
2b)
2c)
2d)
2e)
2f)

