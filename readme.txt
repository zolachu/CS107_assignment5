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

Recommended fix: (signed) casting for the subtraction would fix the issue. 
----------------

1b)

Defect description:
-------------------

Test case explanation (put test case in custom_tests!):
-------------------------------------------------------

Recommended fix:
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

