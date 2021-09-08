|\* ==========================================|
| :- |
||`    `Unity Project - A Test Framework for C|
||`    `Copyright (c) 2007-21 Mike Karlesky, Mark VanderVoord, Greg Williams|
||`    `[Released under MIT License. Please refer to license.txt for details]|
||========================================== \*/|
||<p></p><p></p>|
||#ifndef UNITY\_FRAMEWORK\_H|
||#define UNITY\_FRAMEWORK\_H|
||#define UNITY|
||<p></p><p></p>|
||#define UNITY\_VERSION\_MAJOR    2|
||#define UNITY\_VERSION\_MINOR    5|
||#define UNITY\_VERSION\_BUILD    4|
||#define UNITY\_VERSION          ((UNITY\_VERSION\_MAJOR << 16) | (UNITY\_VERSION\_MINOR << 8) | UNITY\_VERSION\_BUILD)|
||<p></p><p></p>|
||#ifdef \_\_cplusplus|
||extern "C"|
||{|
||#endif|
||<p></p><p></p>|
||#include "unity\_internals.h"|
||<p></p><p></p>|
||/\*-------------------------------------------------------|
||` `\* Test Setup / Teardown|
||` `\*-------------------------------------------------------\*/|
||<p></p><p></p>|
||/\* These functions are intended to be called before and after each test.|
||` `\* If using unity directly, these will need to be provided for each test|
||` `\* executable built. If you are using the test runner generator and/or|
||` `\* Ceedling, these are optional. \*/|
||void setUp(void);|
||void tearDown(void);|
||<p></p><p></p>|
||/\* These functions are intended to be called at the beginning and end of an|
||` `\* entire test suite.  suiteTearDown() is passed the number of tests that|
||` `\* failed, and its return value becomes the exit code of main(). If using|
||` `\* Unity directly, you're in charge of calling these if they are desired.|
||` `\* If using Ceedling or the test runner generator, these will be called|
||` `\* automatically if they exist. \*/|
||void suiteSetUp(void);|
||int suiteTearDown(int num\_failures);|
||<p></p><p></p>|
||/\*-------------------------------------------------------|
||` `\* Test Reset and Verify|
||` `\*-------------------------------------------------------\*/|
||<p></p><p></p>|
||/\* These functions are intended to be called before during tests in order|
||` `\* to support complex test loops, etc. Both are NOT built into Unity. Instead|
||` `\* the test runner generator will create them. resetTest will run teardown and|
||` `\* setup again, verifying any end-of-test needs between. verifyTest will only|
||` `\* run the verification. \*/|
||void resetTest(void);|
||void verifyTest(void);|
||<p></p><p></p>|
||/\*-------------------------------------------------------|
||` `\* Configuration Options|
||` `\*-------------------------------------------------------|
||` `\* All options described below should be passed as a compiler flag to all files using Unity. If you must add #defines, place them BEFORE the #include above.|
|||
||` `\* Integers/longs/pointers|
||` `\*     - Unity attempts to automatically discover your integer sizes|
||` `\*       - define UNITY\_EXCLUDE\_STDINT\_H to stop attempting to look in <stdint.h>|
||` `\*       - define UNITY\_EXCLUDE\_LIMITS\_H to stop attempting to look in <limits.h>|
||` `\*     - If you cannot use the automatic methods above, you can force Unity by using these options:|
||` `\*       - define UNITY\_SUPPORT\_64|
||` `\*       - set UNITY\_INT\_WIDTH|
||` `\*       - set UNITY\_LONG\_WIDTH|
||` `\*       - set UNITY\_POINTER\_WIDTH|
|||
||` `\* Floats|
||` `\*     - define UNITY\_EXCLUDE\_FLOAT to disallow floating point comparisons|
||` `\*     - define UNITY\_FLOAT\_PRECISION to specify the precision to use when doing TEST\_ASSERT\_EQUAL\_FLOAT|
||` `\*     - define UNITY\_FLOAT\_TYPE to specify doubles instead of single precision floats|
||` `\*     - define UNITY\_INCLUDE\_DOUBLE to allow double floating point comparisons|
||` `\*     - define UNITY\_EXCLUDE\_DOUBLE to disallow double floating point comparisons (default)|
||` `\*     - define UNITY\_DOUBLE\_PRECISION to specify the precision to use when doing TEST\_ASSERT\_EQUAL\_DOUBLE|
||` `\*     - define UNITY\_DOUBLE\_TYPE to specify something other than double|
||` `\*     - define UNITY\_EXCLUDE\_FLOAT\_PRINT to trim binary size, won't print floating point values in errors|
|||
||` `\* Output|
||` `\*     - by default, Unity prints to standard out with putchar.  define UNITY\_OUTPUT\_CHAR(a) with a different function if desired|
||` `\*     - define UNITY\_DIFFERENTIATE\_FINAL\_FAIL to print FAILED (vs. FAIL) at test end summary - for automated search for failure|
|||
||` `\* Optimization|
||` `\*     - by default, line numbers are stored in unsigned shorts.  Define UNITY\_LINE\_TYPE with a different type if your files are huge|
||` `\*     - by default, test and failure counters are unsigned shorts.  Define UNITY\_COUNTER\_TYPE with a different type if you want to save space or have more than 65535 Tests.|
|||
||` `\* Test Cases|
||` `\*     - define UNITY\_SUPPORT\_TEST\_CASES to include the TEST\_CASE macro, though really it's mostly about the runner generator script|
|||
||` `\* Parameterized Tests|
||` `\*     - you'll want to create a define of TEST\_CASE(...) which basically evaluates to nothing|
|||
||` `\* Tests with Arguments|
||` `\*     - you'll want to define UNITY\_USE\_COMMAND\_LINE\_ARGS if you have the test runner passing arguments to Unity|
|||
||` `\*-------------------------------------------------------|
||` `\* Basic Fail and Ignore|
||` `\*-------------------------------------------------------\*/|
||<p></p><p></p>|
||#define TEST\_FAIL\_MESSAGE(message)                                                                 UNITY\_TEST\_FAIL(\_\_LINE\_\_, (message))|
||#define TEST\_FAIL()                                                                                UNITY\_TEST\_FAIL(\_\_LINE\_\_, NULL)|
||#define TEST\_IGNORE\_MESSAGE(message)                                                               UNITY\_TEST\_IGNORE(\_\_LINE\_\_, (message))|
||#define TEST\_IGNORE()                                                                              UNITY\_TEST\_IGNORE(\_\_LINE\_\_, NULL)|
||#define TEST\_MESSAGE(message)                                                                      UnityMessage((message), \_\_LINE\_\_)|
||#define TEST\_ONLY()|
||#ifdef UNITY\_INCLUDE\_PRINT\_FORMATTED|
||#define TEST\_PRINTF(message, ...)                                                                  UnityPrintF(\_\_LINE\_\_, (message), \_\_VA\_ARGS\_\_)|
||#endif|
||<p></p><p></p>|
||/\* It is not necessary for you to call PASS. A PASS condition is assumed if nothing fails.|
||` `\* This method allows you to abort a test immediately with a PASS state, ignoring the remainder of the test. \*/|
||#define TEST\_PASS()                                                                                TEST\_ABORT()|
||#define TEST\_PASS\_MESSAGE(message)                                                                 do { UnityMessage((message), \_\_LINE\_\_); TEST\_ABORT(); } while(0)|
||<p></p><p></p>|
||/\* This macro does nothing, but it is useful for build tools (like Ceedling) to make use of this to figure out|
||` `\* which files should be linked to in order to perform a test. Use it like TEST\_FILE("sandwiches.c") \*/|
||#define TEST\_FILE(a)|
||<p></p><p></p>|
||/\*-------------------------------------------------------|
||` `\* Test Asserts (simple)|
||` `\*-------------------------------------------------------\*/|
||<p></p><p></p>|
||/\* Boolean \*/|
||#define TEST\_ASSERT(condition)                                                                     UNITY\_TEST\_ASSERT(       (condition), \_\_LINE\_\_, " Expression Evaluated To FALSE")|
||#define TEST\_ASSERT\_TRUE(condition)                                                                UNITY\_TEST\_ASSERT(       (condition), \_\_LINE\_\_, " Expected TRUE Was FALSE")|
||#define TEST\_ASSERT\_UNLESS(condition)                                                              UNITY\_TEST\_ASSERT(      !(condition), \_\_LINE\_\_, " Expression Evaluated To TRUE")|
||#define TEST\_ASSERT\_FALSE(condition)                                                               UNITY\_TEST\_ASSERT(      !(condition), \_\_LINE\_\_, " Expected FALSE Was TRUE")|
||#define TEST\_ASSERT\_NULL(pointer)                                                                  UNITY\_TEST\_ASSERT\_NULL(    (pointer), \_\_LINE\_\_, " Expected NULL")|
||#define TEST\_ASSERT\_NOT\_NULL(pointer)                                                              UNITY\_TEST\_ASSERT\_NOT\_NULL((pointer), \_\_LINE\_\_, " Expected Non-NULL")|
||#define TEST\_ASSERT\_EMPTY(pointer)                                                                 UNITY\_TEST\_ASSERT\_EMPTY(    (pointer), \_\_LINE\_\_, " Expected Empty")|
||#define TEST\_ASSERT\_NOT\_EMPTY(pointer)                                                             UNITY\_TEST\_ASSERT\_NOT\_EMPTY((pointer), \_\_LINE\_\_, " Expected Non-Empty")|
||<p></p><p></p>|
||/\* Integers (of all sizes) \*/|
||#define TEST\_ASSERT\_EQUAL\_INT(expected, actual)                                                    UNITY\_TEST\_ASSERT\_EQUAL\_INT((expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_INT8(expected, actual)                                                   UNITY\_TEST\_ASSERT\_EQUAL\_INT8((expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_INT16(expected, actual)                                                  UNITY\_TEST\_ASSERT\_EQUAL\_INT16((expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_INT32(expected, actual)                                                  UNITY\_TEST\_ASSERT\_EQUAL\_INT32((expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_INT64(expected, actual)                                                  UNITY\_TEST\_ASSERT\_EQUAL\_INT64((expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_UINT(expected, actual)                                                   UNITY\_TEST\_ASSERT\_EQUAL\_UINT( (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_UINT8(expected, actual)                                                  UNITY\_TEST\_ASSERT\_EQUAL\_UINT8( (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_UINT16(expected, actual)                                                 UNITY\_TEST\_ASSERT\_EQUAL\_UINT16( (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_UINT32(expected, actual)                                                 UNITY\_TEST\_ASSERT\_EQUAL\_UINT32( (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_UINT64(expected, actual)                                                 UNITY\_TEST\_ASSERT\_EQUAL\_UINT64( (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_size\_t(expected, actual)                                                 UNITY\_TEST\_ASSERT\_EQUAL\_UINT((expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_HEX(expected, actual)                                                    UNITY\_TEST\_ASSERT\_EQUAL\_HEX32((expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_HEX8(expected, actual)                                                   UNITY\_TEST\_ASSERT\_EQUAL\_HEX8( (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_HEX16(expected, actual)                                                  UNITY\_TEST\_ASSERT\_EQUAL\_HEX16((expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_HEX32(expected, actual)                                                  UNITY\_TEST\_ASSERT\_EQUAL\_HEX32((expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_HEX64(expected, actual)                                                  UNITY\_TEST\_ASSERT\_EQUAL\_HEX64((expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_CHAR(expected, actual)                                                   UNITY\_TEST\_ASSERT\_EQUAL\_CHAR((expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_BITS(mask, expected, actual)                                                   UNITY\_TEST\_ASSERT\_BITS((mask), (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_BITS\_HIGH(mask, actual)                                                        UNITY\_TEST\_ASSERT\_BITS((mask), (UNITY\_UINT)(-1), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_BITS\_LOW(mask, actual)                                                         UNITY\_TEST\_ASSERT\_BITS((mask), (UNITY\_UINT)(0), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_BIT\_HIGH(bit, actual)                                                          UNITY\_TEST\_ASSERT\_BITS(((UNITY\_UINT)1 << (bit)), (UNITY\_UINT)(-1), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_BIT\_LOW(bit, actual)                                                           UNITY\_TEST\_ASSERT\_BITS(((UNITY\_UINT)1 << (bit)), (UNITY\_UINT)(0), (actual), \_\_LINE\_\_, NULL)|
||<p></p><p></p>|
||/\* Integer Not Equal To (of all sizes) \*/|
||#define TEST\_ASSERT\_NOT\_EQUAL\_INT(threshold, actual)                                               UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_INT((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_NOT\_EQUAL\_INT8(threshold, actual)                                              UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_INT8((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_NOT\_EQUAL\_INT16(threshold, actual)                                             UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_INT16((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_NOT\_EQUAL\_INT32(threshold, actual)                                             UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_INT32((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_NOT\_EQUAL\_INT64(threshold, actual)                                             UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_INT64((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_NOT\_EQUAL\_UINT(threshold, actual)                                              UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_UINT((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_NOT\_EQUAL\_UINT8(threshold, actual)                                             UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_UINT8((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_NOT\_EQUAL\_UINT16(threshold, actual)                                            UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_UINT16((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_NOT\_EQUAL\_UINT32(threshold, actual)                                            UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_UINT32((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_NOT\_EQUAL\_UINT64(threshold, actual)                                            UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_UINT64((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_NOT\_EQUAL\_size\_t(threshold, actual)                                            UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_UINT((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_NOT\_EQUAL\_HEX8(threshold, actual)                                              UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_HEX8((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_NOT\_EQUAL\_HEX16(threshold, actual)                                             UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_HEX16((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_NOT\_EQUAL\_HEX32(threshold, actual)                                             UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_HEX32((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_NOT\_EQUAL\_HEX64(threshold, actual)                                             UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_HEX64((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_NOT\_EQUAL\_CHAR(threshold, actual)                                              UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_CHAR((threshold), (actual), \_\_LINE\_\_, NULL)|
||<p></p><p></p>|
||/\* Integer Greater Than/ Less Than (of all sizes) \*/|
||#define TEST\_ASSERT\_GREATER\_THAN(threshold, actual)                                                UNITY\_TEST\_ASSERT\_GREATER\_THAN\_INT((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_THAN\_INT(threshold, actual)                                            UNITY\_TEST\_ASSERT\_GREATER\_THAN\_INT((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_THAN\_INT8(threshold, actual)                                           UNITY\_TEST\_ASSERT\_GREATER\_THAN\_INT8((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_THAN\_INT16(threshold, actual)                                          UNITY\_TEST\_ASSERT\_GREATER\_THAN\_INT16((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_THAN\_INT32(threshold, actual)                                          UNITY\_TEST\_ASSERT\_GREATER\_THAN\_INT32((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_THAN\_INT64(threshold, actual)                                          UNITY\_TEST\_ASSERT\_GREATER\_THAN\_INT64((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_THAN\_UINT(threshold, actual)                                           UNITY\_TEST\_ASSERT\_GREATER\_THAN\_UINT((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_THAN\_UINT8(threshold, actual)                                          UNITY\_TEST\_ASSERT\_GREATER\_THAN\_UINT8((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_THAN\_UINT16(threshold, actual)                                         UNITY\_TEST\_ASSERT\_GREATER\_THAN\_UINT16((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_THAN\_UINT32(threshold, actual)                                         UNITY\_TEST\_ASSERT\_GREATER\_THAN\_UINT32((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_THAN\_UINT64(threshold, actual)                                         UNITY\_TEST\_ASSERT\_GREATER\_THAN\_UINT64((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_THAN\_size\_t(threshold, actual)                                         UNITY\_TEST\_ASSERT\_GREATER\_THAN\_UINT((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_THAN\_HEX8(threshold, actual)                                           UNITY\_TEST\_ASSERT\_GREATER\_THAN\_HEX8((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_THAN\_HEX16(threshold, actual)                                          UNITY\_TEST\_ASSERT\_GREATER\_THAN\_HEX16((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_THAN\_HEX32(threshold, actual)                                          UNITY\_TEST\_ASSERT\_GREATER\_THAN\_HEX32((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_THAN\_HEX64(threshold, actual)                                          UNITY\_TEST\_ASSERT\_GREATER\_THAN\_HEX64((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_THAN\_CHAR(threshold, actual)                                           UNITY\_TEST\_ASSERT\_GREATER\_THAN\_CHAR((threshold), (actual), \_\_LINE\_\_, NULL)|
||<p></p><p></p>|
||#define TEST\_ASSERT\_LESS\_THAN(threshold, actual)                                                   UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_INT((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_THAN\_INT(threshold, actual)                                               UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_INT((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_THAN\_INT8(threshold, actual)                                              UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_INT8((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_THAN\_INT16(threshold, actual)                                             UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_INT16((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_THAN\_INT32(threshold, actual)                                             UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_INT32((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_THAN\_INT64(threshold, actual)                                             UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_INT64((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_THAN\_UINT(threshold, actual)                                              UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_UINT((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_THAN\_UINT8(threshold, actual)                                             UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_UINT8((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_THAN\_UINT16(threshold, actual)                                            UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_UINT16((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_THAN\_UINT32(threshold, actual)                                            UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_UINT32((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_THAN\_UINT64(threshold, actual)                                            UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_UINT64((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_THAN\_size\_t(threshold, actual)                                            UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_UINT((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_THAN\_HEX8(threshold, actual)                                              UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_HEX8((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_THAN\_HEX16(threshold, actual)                                             UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_HEX16((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_THAN\_HEX32(threshold, actual)                                             UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_HEX32((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_THAN\_HEX64(threshold, actual)                                             UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_HEX64((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_THAN\_CHAR(threshold, actual)                                              UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_CHAR((threshold), (actual), \_\_LINE\_\_, NULL)|
||<p></p><p></p>|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL(threshold, actual)                                            UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT(threshold, actual)                                        UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT8(threshold, actual)                                       UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT8((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT16(threshold, actual)                                      UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT16((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT32(threshold, actual)                                      UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT32((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT64(threshold, actual)                                      UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT64((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT(threshold, actual)                                       UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT8(threshold, actual)                                      UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT8((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT16(threshold, actual)                                     UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT16((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT32(threshold, actual)                                     UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT32((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT64(threshold, actual)                                     UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT64((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_size\_t(threshold, actual)                                     UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_HEX8(threshold, actual)                                       UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_HEX8((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_HEX16(threshold, actual)                                      UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_HEX16((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_HEX32(threshold, actual)                                      UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_HEX32((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_HEX64(threshold, actual)                                      UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_HEX64((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_CHAR(threshold, actual)                                       UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_CHAR((threshold), (actual), \_\_LINE\_\_, NULL)|
||<p></p><p></p>|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL(threshold, actual)                                               UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_INT((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_INT(threshold, actual)                                           UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_INT((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_INT8(threshold, actual)                                          UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_INT8((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_INT16(threshold, actual)                                         UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_INT16((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_INT32(threshold, actual)                                         UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_INT32((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_INT64(threshold, actual)                                         UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_INT64((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_UINT(threshold, actual)                                          UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_UINT((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_UINT8(threshold, actual)                                         UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_UINT8((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_UINT16(threshold, actual)                                        UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_UINT16((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_UINT32(threshold, actual)                                        UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_UINT32((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_UINT64(threshold, actual)                                        UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_UINT64((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_size\_t(threshold, actual)                                        UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_UINT((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_HEX8(threshold, actual)                                          UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_HEX8((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_HEX16(threshold, actual)                                         UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_HEX16((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_HEX32(threshold, actual)                                         UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_HEX32((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_HEX64(threshold, actual)                                         UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_HEX64((threshold), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_CHAR(threshold, actual)                                          UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_CHAR((threshold), (actual), \_\_LINE\_\_, NULL)|
||<p></p><p></p>|
||/\* Integer Ranges (of all sizes) \*/|
||#define TEST\_ASSERT\_INT\_WITHIN(delta, expected, actual)                                            UNITY\_TEST\_ASSERT\_INT\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_INT8\_WITHIN(delta, expected, actual)                                           UNITY\_TEST\_ASSERT\_INT8\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_INT16\_WITHIN(delta, expected, actual)                                          UNITY\_TEST\_ASSERT\_INT16\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_INT32\_WITHIN(delta, expected, actual)                                          UNITY\_TEST\_ASSERT\_INT32\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_INT64\_WITHIN(delta, expected, actual)                                          UNITY\_TEST\_ASSERT\_INT64\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_UINT\_WITHIN(delta, expected, actual)                                           UNITY\_TEST\_ASSERT\_UINT\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_UINT8\_WITHIN(delta, expected, actual)                                          UNITY\_TEST\_ASSERT\_UINT8\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_UINT16\_WITHIN(delta, expected, actual)                                         UNITY\_TEST\_ASSERT\_UINT16\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_UINT32\_WITHIN(delta, expected, actual)                                         UNITY\_TEST\_ASSERT\_UINT32\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_UINT64\_WITHIN(delta, expected, actual)                                         UNITY\_TEST\_ASSERT\_UINT64\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_size\_t\_WITHIN(delta, expected, actual)                                         UNITY\_TEST\_ASSERT\_UINT\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_HEX\_WITHIN(delta, expected, actual)                                            UNITY\_TEST\_ASSERT\_HEX32\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_HEX8\_WITHIN(delta, expected, actual)                                           UNITY\_TEST\_ASSERT\_HEX8\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_HEX16\_WITHIN(delta, expected, actual)                                          UNITY\_TEST\_ASSERT\_HEX16\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_HEX32\_WITHIN(delta, expected, actual)                                          UNITY\_TEST\_ASSERT\_HEX32\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_HEX64\_WITHIN(delta, expected, actual)                                          UNITY\_TEST\_ASSERT\_HEX64\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_CHAR\_WITHIN(delta, expected, actual)                                           UNITY\_TEST\_ASSERT\_CHAR\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, NULL)|
||<p></p><p></p>|
||/\* Integer Array Ranges (of all sizes) \*/|
||#define TEST\_ASSERT\_INT\_ARRAY\_WITHIN(delta, expected, actual, num\_elements)                        UNITY\_TEST\_ASSERT\_INT\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_INT8\_ARRAY\_WITHIN(delta, expected, actual, num\_elements)                       UNITY\_TEST\_ASSERT\_INT8\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_INT16\_ARRAY\_WITHIN(delta, expected, actual, num\_elements)                      UNITY\_TEST\_ASSERT\_INT16\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_INT32\_ARRAY\_WITHIN(delta, expected, actual, num\_elements)                      UNITY\_TEST\_ASSERT\_INT32\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_INT64\_ARRAY\_WITHIN(delta, expected, actual, num\_elements)                      UNITY\_TEST\_ASSERT\_INT64\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_UINT\_ARRAY\_WITHIN(delta, expected, actual, num\_elements)                       UNITY\_TEST\_ASSERT\_UINT\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_UINT8\_ARRAY\_WITHIN(delta, expected, actual, num\_elements)                      UNITY\_TEST\_ASSERT\_UINT8\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_UINT16\_ARRAY\_WITHIN(delta, expected, actual, num\_elements)                     UNITY\_TEST\_ASSERT\_UINT16\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_UINT32\_ARRAY\_WITHIN(delta, expected, actual, num\_elements)                     UNITY\_TEST\_ASSERT\_UINT32\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_UINT64\_ARRAY\_WITHIN(delta, expected, actual, num\_elements)                     UNITY\_TEST\_ASSERT\_UINT64\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_size\_t\_ARRAY\_WITHIN(delta, expected, actual, num\_elements)                     UNITY\_TEST\_ASSERT\_UINT\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_HEX\_ARRAY\_WITHIN(delta, expected, actual, num\_elements)                        UNITY\_TEST\_ASSERT\_HEX32\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_HEX8\_ARRAY\_WITHIN(delta, expected, actual, num\_elements)                       UNITY\_TEST\_ASSERT\_HEX8\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_HEX16\_ARRAY\_WITHIN(delta, expected, actual, num\_elements)                      UNITY\_TEST\_ASSERT\_HEX16\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_HEX32\_ARRAY\_WITHIN(delta, expected, actual, num\_elements)                      UNITY\_TEST\_ASSERT\_HEX32\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_HEX64\_ARRAY\_WITHIN(delta, expected, actual, num\_elements)                      UNITY\_TEST\_ASSERT\_HEX64\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_CHAR\_ARRAY\_WITHIN(delta, expected, actual, num\_elements)                       UNITY\_TEST\_ASSERT\_CHAR\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, NULL)|
||<p></p><p></p>|
||<p></p><p></p>|
||/\* Structs and Strings \*/|
||#define TEST\_ASSERT\_EQUAL\_PTR(expected, actual)                                                    UNITY\_TEST\_ASSERT\_EQUAL\_PTR((expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_STRING(expected, actual)                                                 UNITY\_TEST\_ASSERT\_EQUAL\_STRING((expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_STRING\_LEN(expected, actual, len)                                        UNITY\_TEST\_ASSERT\_EQUAL\_STRING\_LEN((expected), (actual), (len), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_MEMORY(expected, actual, len)                                            UNITY\_TEST\_ASSERT\_EQUAL\_MEMORY((expected), (actual), (len), \_\_LINE\_\_, NULL)|
||<p></p><p></p>|
||/\* Arrays \*/|
||#define TEST\_ASSERT\_EQUAL\_INT\_ARRAY(expected, actual, num\_elements)                                UNITY\_TEST\_ASSERT\_EQUAL\_INT\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_INT8\_ARRAY(expected, actual, num\_elements)                               UNITY\_TEST\_ASSERT\_EQUAL\_INT8\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_INT16\_ARRAY(expected, actual, num\_elements)                              UNITY\_TEST\_ASSERT\_EQUAL\_INT16\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_INT32\_ARRAY(expected, actual, num\_elements)                              UNITY\_TEST\_ASSERT\_EQUAL\_INT32\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_INT64\_ARRAY(expected, actual, num\_elements)                              UNITY\_TEST\_ASSERT\_EQUAL\_INT64\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_UINT\_ARRAY(expected, actual, num\_elements)                               UNITY\_TEST\_ASSERT\_EQUAL\_UINT\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_UINT8\_ARRAY(expected, actual, num\_elements)                              UNITY\_TEST\_ASSERT\_EQUAL\_UINT8\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_UINT16\_ARRAY(expected, actual, num\_elements)                             UNITY\_TEST\_ASSERT\_EQUAL\_UINT16\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_UINT32\_ARRAY(expected, actual, num\_elements)                             UNITY\_TEST\_ASSERT\_EQUAL\_UINT32\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_UINT64\_ARRAY(expected, actual, num\_elements)                             UNITY\_TEST\_ASSERT\_EQUAL\_UINT64\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_size\_t\_ARRAY(expected, actual, num\_elements)                             UNITY\_TEST\_ASSERT\_EQUAL\_UINT\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_HEX\_ARRAY(expected, actual, num\_elements)                                UNITY\_TEST\_ASSERT\_EQUAL\_HEX32\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_HEX8\_ARRAY(expected, actual, num\_elements)                               UNITY\_TEST\_ASSERT\_EQUAL\_HEX8\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_HEX16\_ARRAY(expected, actual, num\_elements)                              UNITY\_TEST\_ASSERT\_EQUAL\_HEX16\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_HEX32\_ARRAY(expected, actual, num\_elements)                              UNITY\_TEST\_ASSERT\_EQUAL\_HEX32\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_HEX64\_ARRAY(expected, actual, num\_elements)                              UNITY\_TEST\_ASSERT\_EQUAL\_HEX64\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_PTR\_ARRAY(expected, actual, num\_elements)                                UNITY\_TEST\_ASSERT\_EQUAL\_PTR\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_STRING\_ARRAY(expected, actual, num\_elements)                             UNITY\_TEST\_ASSERT\_EQUAL\_STRING\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_MEMORY\_ARRAY(expected, actual, len, num\_elements)                        UNITY\_TEST\_ASSERT\_EQUAL\_MEMORY\_ARRAY((expected), (actual), (len), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_CHAR\_ARRAY(expected, actual, num\_elements)                               UNITY\_TEST\_ASSERT\_EQUAL\_CHAR\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||<p></p><p></p>|
||/\* Arrays Compared To Single Value \*/|
||#define TEST\_ASSERT\_EACH\_EQUAL\_INT(expected, actual, num\_elements)                                 UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_INT((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EACH\_EQUAL\_INT8(expected, actual, num\_elements)                                UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_INT8((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EACH\_EQUAL\_INT16(expected, actual, num\_elements)                               UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_INT16((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EACH\_EQUAL\_INT32(expected, actual, num\_elements)                               UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_INT32((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EACH\_EQUAL\_INT64(expected, actual, num\_elements)                               UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_INT64((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EACH\_EQUAL\_UINT(expected, actual, num\_elements)                                UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_UINT((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EACH\_EQUAL\_UINT8(expected, actual, num\_elements)                               UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_UINT8((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EACH\_EQUAL\_UINT16(expected, actual, num\_elements)                              UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_UINT16((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EACH\_EQUAL\_UINT32(expected, actual, num\_elements)                              UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_UINT32((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EACH\_EQUAL\_UINT64(expected, actual, num\_elements)                              UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_UINT64((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EACH\_EQUAL\_size\_t(expected, actual, num\_elements)                              UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_UINT((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EACH\_EQUAL\_HEX(expected, actual, num\_elements)                                 UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_HEX32((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EACH\_EQUAL\_HEX8(expected, actual, num\_elements)                                UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_HEX8((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EACH\_EQUAL\_HEX16(expected, actual, num\_elements)                               UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_HEX16((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EACH\_EQUAL\_HEX32(expected, actual, num\_elements)                               UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_HEX32((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EACH\_EQUAL\_HEX64(expected, actual, num\_elements)                               UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_HEX64((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EACH\_EQUAL\_PTR(expected, actual, num\_elements)                                 UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_PTR((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EACH\_EQUAL\_STRING(expected, actual, num\_elements)                              UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_STRING((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EACH\_EQUAL\_MEMORY(expected, actual, len, num\_elements)                         UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_MEMORY((expected), (actual), (len), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EACH\_EQUAL\_CHAR(expected, actual, num\_elements)                                UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_CHAR((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||<p></p><p></p>|
||/\* Floating Point (If Enabled) \*/|
||#define TEST\_ASSERT\_FLOAT\_WITHIN(delta, expected, actual)                                          UNITY\_TEST\_ASSERT\_FLOAT\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_FLOAT(expected, actual)                                                  UNITY\_TEST\_ASSERT\_EQUAL\_FLOAT((expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_FLOAT\_ARRAY(expected, actual, num\_elements)                              UNITY\_TEST\_ASSERT\_EQUAL\_FLOAT\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EACH\_EQUAL\_FLOAT(expected, actual, num\_elements)                               UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_FLOAT((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_FLOAT\_IS\_INF(actual)                                                           UNITY\_TEST\_ASSERT\_FLOAT\_IS\_INF((actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_FLOAT\_IS\_NEG\_INF(actual)                                                       UNITY\_TEST\_ASSERT\_FLOAT\_IS\_NEG\_INF((actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_FLOAT\_IS\_NAN(actual)                                                           UNITY\_TEST\_ASSERT\_FLOAT\_IS\_NAN((actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_FLOAT\_IS\_DETERMINATE(actual)                                                   UNITY\_TEST\_ASSERT\_FLOAT\_IS\_DETERMINATE((actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_FLOAT\_IS\_NOT\_INF(actual)                                                       UNITY\_TEST\_ASSERT\_FLOAT\_IS\_NOT\_INF((actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_FLOAT\_IS\_NOT\_NEG\_INF(actual)                                                   UNITY\_TEST\_ASSERT\_FLOAT\_IS\_NOT\_NEG\_INF((actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_FLOAT\_IS\_NOT\_NAN(actual)                                                       UNITY\_TEST\_ASSERT\_FLOAT\_IS\_NOT\_NAN((actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_FLOAT\_IS\_NOT\_DETERMINATE(actual)                                               UNITY\_TEST\_ASSERT\_FLOAT\_IS\_NOT\_DETERMINATE((actual), \_\_LINE\_\_, NULL)|
||<p></p><p></p>|
||/\* Double (If Enabled) \*/|
||#define TEST\_ASSERT\_DOUBLE\_WITHIN(delta, expected, actual)                                         UNITY\_TEST\_ASSERT\_DOUBLE\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_DOUBLE(expected, actual)                                                 UNITY\_TEST\_ASSERT\_EQUAL\_DOUBLE((expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EQUAL\_DOUBLE\_ARRAY(expected, actual, num\_elements)                             UNITY\_TEST\_ASSERT\_EQUAL\_DOUBLE\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_EACH\_EQUAL\_DOUBLE(expected, actual, num\_elements)                              UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_DOUBLE((expected), (actual), (num\_elements), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_DOUBLE\_IS\_INF(actual)                                                          UNITY\_TEST\_ASSERT\_DOUBLE\_IS\_INF((actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_DOUBLE\_IS\_NEG\_INF(actual)                                                      UNITY\_TEST\_ASSERT\_DOUBLE\_IS\_NEG\_INF((actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_DOUBLE\_IS\_NAN(actual)                                                          UNITY\_TEST\_ASSERT\_DOUBLE\_IS\_NAN((actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_DOUBLE\_IS\_DETERMINATE(actual)                                                  UNITY\_TEST\_ASSERT\_DOUBLE\_IS\_DETERMINATE((actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_DOUBLE\_IS\_NOT\_INF(actual)                                                      UNITY\_TEST\_ASSERT\_DOUBLE\_IS\_NOT\_INF((actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_DOUBLE\_IS\_NOT\_NEG\_INF(actual)                                                  UNITY\_TEST\_ASSERT\_DOUBLE\_IS\_NOT\_NEG\_INF((actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_DOUBLE\_IS\_NOT\_NAN(actual)                                                      UNITY\_TEST\_ASSERT\_DOUBLE\_IS\_NOT\_NAN((actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_DOUBLE\_IS\_NOT\_DETERMINATE(actual)                                              UNITY\_TEST\_ASSERT\_DOUBLE\_IS\_NOT\_DETERMINATE((actual), \_\_LINE\_\_, NULL)|
||<p></p><p></p>|
||/\* Shorthand \*/|
||#ifdef UNITY\_SHORTHAND\_AS\_OLD|
||#define TEST\_ASSERT\_EQUAL(expected, actual)                                                        UNITY\_TEST\_ASSERT\_EQUAL\_INT((expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_NOT\_EQUAL(expected, actual)                                                    UNITY\_TEST\_ASSERT(((expected) != (actual)), \_\_LINE\_\_, " Expected Not-Equal")|
||#endif|
||#ifdef UNITY\_SHORTHAND\_AS\_INT|
||#define TEST\_ASSERT\_EQUAL(expected, actual)                                                        UNITY\_TEST\_ASSERT\_EQUAL\_INT((expected), (actual), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_NOT\_EQUAL(expected, actual)                                                    UNITY\_TEST\_FAIL(\_\_LINE\_\_, UnityStrErrShorthand)|
||#endif|
||#ifdef UNITY\_SHORTHAND\_AS\_MEM|
||#define TEST\_ASSERT\_EQUAL(expected, actual)                                                        UNITY\_TEST\_ASSERT\_EQUAL\_MEMORY((&expected), (&actual), sizeof(expected), \_\_LINE\_\_, NULL)|
||#define TEST\_ASSERT\_NOT\_EQUAL(expected, actual)                                                    UNITY\_TEST\_FAIL(\_\_LINE\_\_, UnityStrErrShorthand)|
||#endif|
||#ifdef UNITY\_SHORTHAND\_AS\_RAW|
||#define TEST\_ASSERT\_EQUAL(expected, actual)                                                        UNITY\_TEST\_ASSERT(((expected) == (actual)), \_\_LINE\_\_, " Expected Equal")|
||#define TEST\_ASSERT\_NOT\_EQUAL(expected, actual)                                                    UNITY\_TEST\_ASSERT(((expected) != (actual)), \_\_LINE\_\_, " Expected Not-Equal")|
||#endif|
||#ifdef UNITY\_SHORTHAND\_AS\_NONE|
||#define TEST\_ASSERT\_EQUAL(expected, actual)                                                        UNITY\_TEST\_FAIL(\_\_LINE\_\_, UnityStrErrShorthand)|
||#define TEST\_ASSERT\_NOT\_EQUAL(expected, actual)                                                    UNITY\_TEST\_FAIL(\_\_LINE\_\_, UnityStrErrShorthand)|
||#endif|
||<p></p><p></p>|
||/\*-------------------------------------------------------|
||` `\* Test Asserts (with additional messages)|
||` `\*-------------------------------------------------------\*/|
||<p></p><p></p>|
||/\* Boolean \*/|
||#define TEST\_ASSERT\_MESSAGE(condition, message)                                                    UNITY\_TEST\_ASSERT(       (condition), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_TRUE\_MESSAGE(condition, message)                                               UNITY\_TEST\_ASSERT(       (condition), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_UNLESS\_MESSAGE(condition, message)                                             UNITY\_TEST\_ASSERT(      !(condition), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_FALSE\_MESSAGE(condition, message)                                              UNITY\_TEST\_ASSERT(      !(condition), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_NULL\_MESSAGE(pointer, message)                                                 UNITY\_TEST\_ASSERT\_NULL(    (pointer), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_NOT\_NULL\_MESSAGE(pointer, message)                                             UNITY\_TEST\_ASSERT\_NOT\_NULL((pointer), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EMPTY\_MESSAGE(pointer, message)                                                UNITY\_TEST\_ASSERT\_EMPTY(    (pointer), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_NOT\_EMPTY\_MESSAGE(pointer, message)                                            UNITY\_TEST\_ASSERT\_NOT\_EMPTY((pointer), \_\_LINE\_\_, (message))|
||<p></p><p></p>|
||/\* Integers (of all sizes) \*/|
||#define TEST\_ASSERT\_EQUAL\_INT\_MESSAGE(expected, actual, message)                                   UNITY\_TEST\_ASSERT\_EQUAL\_INT((expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_INT8\_MESSAGE(expected, actual, message)                                  UNITY\_TEST\_ASSERT\_EQUAL\_INT8((expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_INT16\_MESSAGE(expected, actual, message)                                 UNITY\_TEST\_ASSERT\_EQUAL\_INT16((expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_INT32\_MESSAGE(expected, actual, message)                                 UNITY\_TEST\_ASSERT\_EQUAL\_INT32((expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_INT64\_MESSAGE(expected, actual, message)                                 UNITY\_TEST\_ASSERT\_EQUAL\_INT64((expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_UINT\_MESSAGE(expected, actual, message)                                  UNITY\_TEST\_ASSERT\_EQUAL\_UINT( (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_UINT8\_MESSAGE(expected, actual, message)                                 UNITY\_TEST\_ASSERT\_EQUAL\_UINT8( (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_UINT16\_MESSAGE(expected, actual, message)                                UNITY\_TEST\_ASSERT\_EQUAL\_UINT16( (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_UINT32\_MESSAGE(expected, actual, message)                                UNITY\_TEST\_ASSERT\_EQUAL\_UINT32( (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_UINT64\_MESSAGE(expected, actual, message)                                UNITY\_TEST\_ASSERT\_EQUAL\_UINT64( (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_size\_t\_MESSAGE(expected, actual, message)                                UNITY\_TEST\_ASSERT\_EQUAL\_UINT( (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_HEX\_MESSAGE(expected, actual, message)                                   UNITY\_TEST\_ASSERT\_EQUAL\_HEX32((expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_HEX8\_MESSAGE(expected, actual, message)                                  UNITY\_TEST\_ASSERT\_EQUAL\_HEX8( (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_HEX16\_MESSAGE(expected, actual, message)                                 UNITY\_TEST\_ASSERT\_EQUAL\_HEX16((expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_HEX32\_MESSAGE(expected, actual, message)                                 UNITY\_TEST\_ASSERT\_EQUAL\_HEX32((expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_HEX64\_MESSAGE(expected, actual, message)                                 UNITY\_TEST\_ASSERT\_EQUAL\_HEX64((expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_BITS\_MESSAGE(mask, expected, actual, message)                                  UNITY\_TEST\_ASSERT\_BITS((mask), (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_BITS\_HIGH\_MESSAGE(mask, actual, message)                                       UNITY\_TEST\_ASSERT\_BITS((mask), (UNITY\_UINT32)(-1), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_BITS\_LOW\_MESSAGE(mask, actual, message)                                        UNITY\_TEST\_ASSERT\_BITS((mask), (UNITY\_UINT32)(0), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_BIT\_HIGH\_MESSAGE(bit, actual, message)                                         UNITY\_TEST\_ASSERT\_BITS(((UNITY\_UINT32)1 << (bit)), (UNITY\_UINT32)(-1), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_BIT\_LOW\_MESSAGE(bit, actual, message)                                          UNITY\_TEST\_ASSERT\_BITS(((UNITY\_UINT32)1 << (bit)), (UNITY\_UINT32)(0), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_CHAR\_MESSAGE(expected, actual, message)                                  UNITY\_TEST\_ASSERT\_EQUAL\_CHAR((expected), (actual), \_\_LINE\_\_, (message))|
||<p></p><p></p>|
||/\* Integer Not Equal To (of all sizes) \*/|
||#define TEST\_ASSERT\_NOT\_EQUAL\_INT\_MESSAGE(threshold, actual, message)                              UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_INT((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_NOT\_EQUAL\_INT8\_MESSAGE(threshold, actual, message)                             UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_INT8((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_NOT\_EQUAL\_INT16\_MESSAGE(threshold, actual, message)                            UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_INT16((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_NOT\_EQUAL\_INT32\_MESSAGE(threshold, actual, message)                            UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_INT32((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_NOT\_EQUAL\_INT64\_MESSAGE(threshold, actual, message)                            UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_INT64((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_NOT\_EQUAL\_UINT\_MESSAGE(threshold, actual, message)                             UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_UINT((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_NOT\_EQUAL\_UINT8\_MESSAGE(threshold, actual, message)                            UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_UINT8((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_NOT\_EQUAL\_UINT16\_MESSAGE(threshold, actual, message)                           UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_UINT16((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_NOT\_EQUAL\_UINT32\_MESSAGE(threshold, actual, message)                           UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_UINT32((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_NOT\_EQUAL\_UINT64\_MESSAGE(threshold, actual, message)                           UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_UINT64((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_NOT\_EQUAL\_size\_t\_MESSAGE(threshold, actual, message)                           UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_UINT((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_NOT\_EQUAL\_HEX8\_MESSAGE(threshold, actual, message)                             UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_HEX8((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_NOT\_EQUAL\_HEX16\_MESSAGE(threshold, actual, message)                            UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_HEX16((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_NOT\_EQUAL\_HEX32\_MESSAGE(threshold, actual, message)                            UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_HEX32((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_NOT\_EQUAL\_HEX64\_MESSAGE(threshold, actual, message)                            UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_HEX64((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_NOT\_EQUAL\_CHAR\_MESSAGE(threshold, actual, message)                             UNITY\_TEST\_ASSERT\_NOT\_EQUAL\_CHAR((threshold), (actual), \_\_LINE\_\_, (message))|
||<p></p><p></p>|
||<p></p><p></p>|
||/\* Integer Greater Than/ Less Than (of all sizes) \*/|
||#define TEST\_ASSERT\_GREATER\_THAN\_MESSAGE(threshold, actual, message)                               UNITY\_TEST\_ASSERT\_GREATER\_THAN\_INT((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_THAN\_INT\_MESSAGE(threshold, actual, message)                           UNITY\_TEST\_ASSERT\_GREATER\_THAN\_INT((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_THAN\_INT8\_MESSAGE(threshold, actual, message)                          UNITY\_TEST\_ASSERT\_GREATER\_THAN\_INT8((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_THAN\_INT16\_MESSAGE(threshold, actual, message)                         UNITY\_TEST\_ASSERT\_GREATER\_THAN\_INT16((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_THAN\_INT32\_MESSAGE(threshold, actual, message)                         UNITY\_TEST\_ASSERT\_GREATER\_THAN\_INT32((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_THAN\_INT64\_MESSAGE(threshold, actual, message)                         UNITY\_TEST\_ASSERT\_GREATER\_THAN\_INT64((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_THAN\_UINT\_MESSAGE(threshold, actual, message)                          UNITY\_TEST\_ASSERT\_GREATER\_THAN\_UINT((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_THAN\_UINT8\_MESSAGE(threshold, actual, message)                         UNITY\_TEST\_ASSERT\_GREATER\_THAN\_UINT8((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_THAN\_UINT16\_MESSAGE(threshold, actual, message)                        UNITY\_TEST\_ASSERT\_GREATER\_THAN\_UINT16((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_THAN\_UINT32\_MESSAGE(threshold, actual, message)                        UNITY\_TEST\_ASSERT\_GREATER\_THAN\_UINT32((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_THAN\_UINT64\_MESSAGE(threshold, actual, message)                        UNITY\_TEST\_ASSERT\_GREATER\_THAN\_UINT64((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_THAN\_size\_t\_MESSAGE(threshold, actual, message)                        UNITY\_TEST\_ASSERT\_GREATER\_THAN\_UINT((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_THAN\_HEX8\_MESSAGE(threshold, actual, message)                          UNITY\_TEST\_ASSERT\_GREATER\_THAN\_HEX8((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_THAN\_HEX16\_MESSAGE(threshold, actual, message)                         UNITY\_TEST\_ASSERT\_GREATER\_THAN\_HEX16((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_THAN\_HEX32\_MESSAGE(threshold, actual, message)                         UNITY\_TEST\_ASSERT\_GREATER\_THAN\_HEX32((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_THAN\_HEX64\_MESSAGE(threshold, actual, message)                         UNITY\_TEST\_ASSERT\_GREATER\_THAN\_HEX64((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_THAN\_CHAR\_MESSAGE(threshold, actual, message)                          UNITY\_TEST\_ASSERT\_GREATER\_THAN\_CHAR((threshold), (actual), \_\_LINE\_\_, (message))|
||<p></p><p></p>|
||#define TEST\_ASSERT\_LESS\_THAN\_MESSAGE(threshold, actual, message)                                  UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_INT((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_THAN\_INT\_MESSAGE(threshold, actual, message)                              UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_INT((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_THAN\_INT8\_MESSAGE(threshold, actual, message)                             UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_INT8((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_THAN\_INT16\_MESSAGE(threshold, actual, message)                            UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_INT16((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_THAN\_INT32\_MESSAGE(threshold, actual, message)                            UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_INT32((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_THAN\_INT64\_MESSAGE(threshold, actual, message)                            UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_INT64((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_THAN\_UINT\_MESSAGE(threshold, actual, message)                             UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_UINT((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_THAN\_UINT8\_MESSAGE(threshold, actual, message)                            UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_UINT8((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_THAN\_UINT16\_MESSAGE(threshold, actual, message)                           UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_UINT16((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_THAN\_UINT32\_MESSAGE(threshold, actual, message)                           UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_UINT32((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_THAN\_UINT64\_MESSAGE(threshold, actual, message)                           UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_UINT64((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_THAN\_size\_t\_MESSAGE(threshold, actual, message)                           UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_UINT((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_THAN\_HEX8\_MESSAGE(threshold, actual, message)                             UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_HEX8((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_THAN\_HEX16\_MESSAGE(threshold, actual, message)                            UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_HEX16((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_THAN\_HEX32\_MESSAGE(threshold, actual, message)                            UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_HEX32((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_THAN\_HEX64\_MESSAGE(threshold, actual, message)                            UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_HEX64((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_THAN\_CHAR\_MESSAGE(threshold, actual, message)                             UNITY\_TEST\_ASSERT\_SMALLER\_THAN\_CHAR((threshold), (actual), \_\_LINE\_\_, (message))|
||<p></p><p></p>|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_MESSAGE(threshold, actual, message)                           UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT\_MESSAGE(threshold, actual, message)                       UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT8\_MESSAGE(threshold, actual, message)                      UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT8((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT16\_MESSAGE(threshold, actual, message)                     UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT16((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT32\_MESSAGE(threshold, actual, message)                     UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT32((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT64\_MESSAGE(threshold, actual, message)                     UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_INT64((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT\_MESSAGE(threshold, actual, message)                      UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT8\_MESSAGE(threshold, actual, message)                     UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT8((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT16\_MESSAGE(threshold, actual, message)                    UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT16((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT32\_MESSAGE(threshold, actual, message)                    UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT32((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT64\_MESSAGE(threshold, actual, message)                    UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT64((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_size\_t\_MESSAGE(threshold, actual, message)                    UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_UINT((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_HEX8\_MESSAGE(threshold, actual, message)                      UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_HEX8((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_HEX16\_MESSAGE(threshold, actual, message)                     UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_HEX16((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_HEX32\_MESSAGE(threshold, actual, message)                     UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_HEX32((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_HEX64\_MESSAGE(threshold, actual, message)                     UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_HEX64((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_GREATER\_OR\_EQUAL\_CHAR\_MESSAGE(threshold, actual, message)                      UNITY\_TEST\_ASSERT\_GREATER\_OR\_EQUAL\_CHAR((threshold), (actual), \_\_LINE\_\_, (message))|
||<p></p><p></p>|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_MESSAGE(threshold, actual, message)                              UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_INT((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_INT\_MESSAGE(threshold, actual, message)                          UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_INT((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_INT8\_MESSAGE(threshold, actual, message)                         UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_INT8((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_INT16\_MESSAGE(threshold, actual, message)                        UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_INT16((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_INT32\_MESSAGE(threshold, actual, message)                        UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_INT32((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_INT64\_MESSAGE(threshold, actual, message)                        UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_INT64((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_UINT\_MESSAGE(threshold, actual, message)                         UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_UINT((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_UINT8\_MESSAGE(threshold, actual, message)                        UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_UINT8((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_UINT16\_MESSAGE(threshold, actual, message)                       UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_UINT16((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_UINT32\_MESSAGE(threshold, actual, message)                       UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_UINT32((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_UINT64\_MESSAGE(threshold, actual, message)                       UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_UINT64((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_size\_t\_MESSAGE(threshold, actual, message)                       UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_UINT((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_HEX8\_MESSAGE(threshold, actual, message)                         UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_HEX8((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_HEX16\_MESSAGE(threshold, actual, message)                        UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_HEX16((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_HEX32\_MESSAGE(threshold, actual, message)                        UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_HEX32((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_HEX64\_MESSAGE(threshold, actual, message)                        UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_HEX64((threshold), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_LESS\_OR\_EQUAL\_CHAR\_MESSAGE(threshold, actual, message)                         UNITY\_TEST\_ASSERT\_SMALLER\_OR\_EQUAL\_CHAR((threshold), (actual), \_\_LINE\_\_, (message))|
||<p></p><p></p>|
||/\* Integer Ranges (of all sizes) \*/|
||#define TEST\_ASSERT\_INT\_WITHIN\_MESSAGE(delta, expected, actual, message)                           UNITY\_TEST\_ASSERT\_INT\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_INT8\_WITHIN\_MESSAGE(delta, expected, actual, message)                          UNITY\_TEST\_ASSERT\_INT8\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_INT16\_WITHIN\_MESSAGE(delta, expected, actual, message)                         UNITY\_TEST\_ASSERT\_INT16\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_INT32\_WITHIN\_MESSAGE(delta, expected, actual, message)                         UNITY\_TEST\_ASSERT\_INT32\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_INT64\_WITHIN\_MESSAGE(delta, expected, actual, message)                         UNITY\_TEST\_ASSERT\_INT64\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_UINT\_WITHIN\_MESSAGE(delta, expected, actual, message)                          UNITY\_TEST\_ASSERT\_UINT\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_UINT8\_WITHIN\_MESSAGE(delta, expected, actual, message)                         UNITY\_TEST\_ASSERT\_UINT8\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_UINT16\_WITHIN\_MESSAGE(delta, expected, actual, message)                        UNITY\_TEST\_ASSERT\_UINT16\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_UINT32\_WITHIN\_MESSAGE(delta, expected, actual, message)                        UNITY\_TEST\_ASSERT\_UINT32\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_UINT64\_WITHIN\_MESSAGE(delta, expected, actual, message)                        UNITY\_TEST\_ASSERT\_UINT64\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_size\_t\_WITHIN\_MESSAGE(delta, expected, actual, message)                        UNITY\_TEST\_ASSERT\_UINT\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_HEX\_WITHIN\_MESSAGE(delta, expected, actual, message)                           UNITY\_TEST\_ASSERT\_HEX32\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_HEX8\_WITHIN\_MESSAGE(delta, expected, actual, message)                          UNITY\_TEST\_ASSERT\_HEX8\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_HEX16\_WITHIN\_MESSAGE(delta, expected, actual, message)                         UNITY\_TEST\_ASSERT\_HEX16\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_HEX32\_WITHIN\_MESSAGE(delta, expected, actual, message)                         UNITY\_TEST\_ASSERT\_HEX32\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_HEX64\_WITHIN\_MESSAGE(delta, expected, actual, message)                         UNITY\_TEST\_ASSERT\_HEX64\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_CHAR\_WITHIN\_MESSAGE(delta, expected, actual, message)                          UNITY\_TEST\_ASSERT\_CHAR\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, (message))|
||<p></p><p></p>|
||/\* Integer Array Ranges (of all sizes) \*/|
||#define TEST\_ASSERT\_INT\_ARRAY\_WITHIN\_MESSAGE(delta, expected, actual, num\_elements, message)       UNITY\_TEST\_ASSERT\_INT\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_INT8\_ARRAY\_WITHIN\_MESSAGE(delta, expected, actual, num\_elements, message)      UNITY\_TEST\_ASSERT\_INT8\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_INT16\_ARRAY\_WITHIN\_MESSAGE(delta, expected, actual, num\_elements, message)     UNITY\_TEST\_ASSERT\_INT16\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_INT32\_ARRAY\_WITHIN\_MESSAGE(delta, expected, actual, num\_elements, message)     UNITY\_TEST\_ASSERT\_INT32\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_INT64\_ARRAY\_WITHIN\_MESSAGE(delta, expected, actual, num\_elements, message)     UNITY\_TEST\_ASSERT\_INT64\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_UINT\_ARRAY\_WITHIN\_MESSAGE(delta, expected, actual, num\_elements, message)      UNITY\_TEST\_ASSERT\_UINT\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_UINT8\_ARRAY\_WITHIN\_MESSAGE(delta, expected, actual, num\_elements, message)     UNITY\_TEST\_ASSERT\_UINT8\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_UINT16\_ARRAY\_WITHIN\_MESSAGE(delta, expected, actual, num\_elements, message)    UNITY\_TEST\_ASSERT\_UINT16\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_UINT32\_ARRAY\_WITHIN\_MESSAGE(delta, expected, actual, num\_elements, message)    UNITY\_TEST\_ASSERT\_UINT32\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_UINT64\_ARRAY\_WITHIN\_MESSAGE(delta, expected, actual, num\_elements, message)    UNITY\_TEST\_ASSERT\_UINT64\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_size\_t\_ARRAY\_WITHIN\_MESSAGE(delta, expected, actual, num\_elements, message)    UNITY\_TEST\_ASSERT\_UINT\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_HEX\_ARRAY\_WITHIN\_MESSAGE(delta, expected, actual, num\_elements, message)       UNITY\_TEST\_ASSERT\_HEX32\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_HEX8\_ARRAY\_WITHIN\_MESSAGE(delta, expected, actual, num\_elements, message)      UNITY\_TEST\_ASSERT\_HEX8\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_HEX16\_ARRAY\_WITHIN\_MESSAGE(delta, expected, actual, num\_elements, message)     UNITY\_TEST\_ASSERT\_HEX16\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_HEX32\_ARRAY\_WITHIN\_MESSAGE(delta, expected, actual, num\_elements, message)     UNITY\_TEST\_ASSERT\_HEX32\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_HEX64\_ARRAY\_WITHIN\_MESSAGE(delta, expected, actual, num\_elements, message)     UNITY\_TEST\_ASSERT\_HEX64\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_CHAR\_ARRAY\_WITHIN\_MESSAGE(delta, expected, actual, num\_elements, message)      UNITY\_TEST\_ASSERT\_CHAR\_ARRAY\_WITHIN((delta), (expected), (actual), num\_elements, \_\_LINE\_\_, (message))|
||<p></p><p></p>|
||<p></p><p></p>|
||/\* Structs and Strings \*/|
||#define TEST\_ASSERT\_EQUAL\_PTR\_MESSAGE(expected, actual, message)                                   UNITY\_TEST\_ASSERT\_EQUAL\_PTR((expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_STRING\_MESSAGE(expected, actual, message)                                UNITY\_TEST\_ASSERT\_EQUAL\_STRING((expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_STRING\_LEN\_MESSAGE(expected, actual, len, message)                       UNITY\_TEST\_ASSERT\_EQUAL\_STRING\_LEN((expected), (actual), (len), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_MEMORY\_MESSAGE(expected, actual, len, message)                           UNITY\_TEST\_ASSERT\_EQUAL\_MEMORY((expected), (actual), (len), \_\_LINE\_\_, (message))|
||<p></p><p></p>|
||/\* Arrays \*/|
||#define TEST\_ASSERT\_EQUAL\_INT\_ARRAY\_MESSAGE(expected, actual, num\_elements, message)               UNITY\_TEST\_ASSERT\_EQUAL\_INT\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_INT8\_ARRAY\_MESSAGE(expected, actual, num\_elements, message)              UNITY\_TEST\_ASSERT\_EQUAL\_INT8\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_INT16\_ARRAY\_MESSAGE(expected, actual, num\_elements, message)             UNITY\_TEST\_ASSERT\_EQUAL\_INT16\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_INT32\_ARRAY\_MESSAGE(expected, actual, num\_elements, message)             UNITY\_TEST\_ASSERT\_EQUAL\_INT32\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_INT64\_ARRAY\_MESSAGE(expected, actual, num\_elements, message)             UNITY\_TEST\_ASSERT\_EQUAL\_INT64\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_UINT\_ARRAY\_MESSAGE(expected, actual, num\_elements, message)              UNITY\_TEST\_ASSERT\_EQUAL\_UINT\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_UINT8\_ARRAY\_MESSAGE(expected, actual, num\_elements, message)             UNITY\_TEST\_ASSERT\_EQUAL\_UINT8\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_UINT16\_ARRAY\_MESSAGE(expected, actual, num\_elements, message)            UNITY\_TEST\_ASSERT\_EQUAL\_UINT16\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_UINT32\_ARRAY\_MESSAGE(expected, actual, num\_elements, message)            UNITY\_TEST\_ASSERT\_EQUAL\_UINT32\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_UINT64\_ARRAY\_MESSAGE(expected, actual, num\_elements, message)            UNITY\_TEST\_ASSERT\_EQUAL\_UINT64\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_size\_t\_ARRAY\_MESSAGE(expected, actual, num\_elements, message)            UNITY\_TEST\_ASSERT\_EQUAL\_UINT\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_HEX\_ARRAY\_MESSAGE(expected, actual, num\_elements, message)               UNITY\_TEST\_ASSERT\_EQUAL\_HEX32\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_HEX8\_ARRAY\_MESSAGE(expected, actual, num\_elements, message)              UNITY\_TEST\_ASSERT\_EQUAL\_HEX8\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_HEX16\_ARRAY\_MESSAGE(expected, actual, num\_elements, message)             UNITY\_TEST\_ASSERT\_EQUAL\_HEX16\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_HEX32\_ARRAY\_MESSAGE(expected, actual, num\_elements, message)             UNITY\_TEST\_ASSERT\_EQUAL\_HEX32\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_HEX64\_ARRAY\_MESSAGE(expected, actual, num\_elements, message)             UNITY\_TEST\_ASSERT\_EQUAL\_HEX64\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_PTR\_ARRAY\_MESSAGE(expected, actual, num\_elements, message)               UNITY\_TEST\_ASSERT\_EQUAL\_PTR\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_STRING\_ARRAY\_MESSAGE(expected, actual, num\_elements, message)            UNITY\_TEST\_ASSERT\_EQUAL\_STRING\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_MEMORY\_ARRAY\_MESSAGE(expected, actual, len, num\_elements, message)       UNITY\_TEST\_ASSERT\_EQUAL\_MEMORY\_ARRAY((expected), (actual), (len), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_CHAR\_ARRAY\_MESSAGE(expected, actual, num\_elements, message)              UNITY\_TEST\_ASSERT\_EQUAL\_CHAR\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||<p></p><p></p>|
||/\* Arrays Compared To Single Value\*/|
||#define TEST\_ASSERT\_EACH\_EQUAL\_INT\_MESSAGE(expected, actual, num\_elements, message)                UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_INT((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EACH\_EQUAL\_INT8\_MESSAGE(expected, actual, num\_elements, message)               UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_INT8((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EACH\_EQUAL\_INT16\_MESSAGE(expected, actual, num\_elements, message)              UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_INT16((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EACH\_EQUAL\_INT32\_MESSAGE(expected, actual, num\_elements, message)              UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_INT32((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EACH\_EQUAL\_INT64\_MESSAGE(expected, actual, num\_elements, message)              UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_INT64((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EACH\_EQUAL\_UINT\_MESSAGE(expected, actual, num\_elements, message)               UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_UINT((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EACH\_EQUAL\_UINT8\_MESSAGE(expected, actual, num\_elements, message)              UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_UINT8((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EACH\_EQUAL\_UINT16\_MESSAGE(expected, actual, num\_elements, message)             UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_UINT16((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EACH\_EQUAL\_UINT32\_MESSAGE(expected, actual, num\_elements, message)             UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_UINT32((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EACH\_EQUAL\_UINT64\_MESSAGE(expected, actual, num\_elements, message)             UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_UINT64((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EACH\_EQUAL\_size\_t\_MESSAGE(expected, actual, num\_elements, message)             UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_UINT((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EACH\_EQUAL\_HEX\_MESSAGE(expected, actual, num\_elements, message)                UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_HEX32((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EACH\_EQUAL\_HEX8\_MESSAGE(expected, actual, num\_elements, message)               UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_HEX8((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EACH\_EQUAL\_HEX16\_MESSAGE(expected, actual, num\_elements, message)              UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_HEX16((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EACH\_EQUAL\_HEX32\_MESSAGE(expected, actual, num\_elements, message)              UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_HEX32((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EACH\_EQUAL\_HEX64\_MESSAGE(expected, actual, num\_elements, message)              UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_HEX64((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EACH\_EQUAL\_PTR\_MESSAGE(expected, actual, num\_elements, message)                UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_PTR((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EACH\_EQUAL\_STRING\_MESSAGE(expected, actual, num\_elements, message)             UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_STRING((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EACH\_EQUAL\_MEMORY\_MESSAGE(expected, actual, len, num\_elements, message)        UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_MEMORY((expected), (actual), (len), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EACH\_EQUAL\_CHAR\_MESSAGE(expected, actual, num\_elements, message)               UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_CHAR((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||<p></p><p></p>|
||/\* Floating Point (If Enabled) \*/|
||#define TEST\_ASSERT\_FLOAT\_WITHIN\_MESSAGE(delta, expected, actual, message)                         UNITY\_TEST\_ASSERT\_FLOAT\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_FLOAT\_MESSAGE(expected, actual, message)                                 UNITY\_TEST\_ASSERT\_EQUAL\_FLOAT((expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_FLOAT\_ARRAY\_MESSAGE(expected, actual, num\_elements, message)             UNITY\_TEST\_ASSERT\_EQUAL\_FLOAT\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EACH\_EQUAL\_FLOAT\_MESSAGE(expected, actual, num\_elements, message)              UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_FLOAT((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_FLOAT\_IS\_INF\_MESSAGE(actual, message)                                          UNITY\_TEST\_ASSERT\_FLOAT\_IS\_INF((actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_FLOAT\_IS\_NEG\_INF\_MESSAGE(actual, message)                                      UNITY\_TEST\_ASSERT\_FLOAT\_IS\_NEG\_INF((actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_FLOAT\_IS\_NAN\_MESSAGE(actual, message)                                          UNITY\_TEST\_ASSERT\_FLOAT\_IS\_NAN((actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_FLOAT\_IS\_DETERMINATE\_MESSAGE(actual, message)                                  UNITY\_TEST\_ASSERT\_FLOAT\_IS\_DETERMINATE((actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_FLOAT\_IS\_NOT\_INF\_MESSAGE(actual, message)                                      UNITY\_TEST\_ASSERT\_FLOAT\_IS\_NOT\_INF((actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_FLOAT\_IS\_NOT\_NEG\_INF\_MESSAGE(actual, message)                                  UNITY\_TEST\_ASSERT\_FLOAT\_IS\_NOT\_NEG\_INF((actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_FLOAT\_IS\_NOT\_NAN\_MESSAGE(actual, message)                                      UNITY\_TEST\_ASSERT\_FLOAT\_IS\_NOT\_NAN((actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_FLOAT\_IS\_NOT\_DETERMINATE\_MESSAGE(actual, message)                              UNITY\_TEST\_ASSERT\_FLOAT\_IS\_NOT\_DETERMINATE((actual), \_\_LINE\_\_, (message))|
||<p></p><p></p>|
||/\* Double (If Enabled) \*/|
||#define TEST\_ASSERT\_DOUBLE\_WITHIN\_MESSAGE(delta, expected, actual, message)                        UNITY\_TEST\_ASSERT\_DOUBLE\_WITHIN((delta), (expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_DOUBLE\_MESSAGE(expected, actual, message)                                UNITY\_TEST\_ASSERT\_EQUAL\_DOUBLE((expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EQUAL\_DOUBLE\_ARRAY\_MESSAGE(expected, actual, num\_elements, message)            UNITY\_TEST\_ASSERT\_EQUAL\_DOUBLE\_ARRAY((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_EACH\_EQUAL\_DOUBLE\_MESSAGE(expected, actual, num\_elements, message)             UNITY\_TEST\_ASSERT\_EACH\_EQUAL\_DOUBLE((expected), (actual), (num\_elements), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_DOUBLE\_IS\_INF\_MESSAGE(actual, message)                                         UNITY\_TEST\_ASSERT\_DOUBLE\_IS\_INF((actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_DOUBLE\_IS\_NEG\_INF\_MESSAGE(actual, message)                                     UNITY\_TEST\_ASSERT\_DOUBLE\_IS\_NEG\_INF((actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_DOUBLE\_IS\_NAN\_MESSAGE(actual, message)                                         UNITY\_TEST\_ASSERT\_DOUBLE\_IS\_NAN((actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_DOUBLE\_IS\_DETERMINATE\_MESSAGE(actual, message)                                 UNITY\_TEST\_ASSERT\_DOUBLE\_IS\_DETERMINATE((actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_DOUBLE\_IS\_NOT\_INF\_MESSAGE(actual, message)                                     UNITY\_TEST\_ASSERT\_DOUBLE\_IS\_NOT\_INF((actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_DOUBLE\_IS\_NOT\_NEG\_INF\_MESSAGE(actual, message)                                 UNITY\_TEST\_ASSERT\_DOUBLE\_IS\_NOT\_NEG\_INF((actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_DOUBLE\_IS\_NOT\_NAN\_MESSAGE(actual, message)                                     UNITY\_TEST\_ASSERT\_DOUBLE\_IS\_NOT\_NAN((actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_DOUBLE\_IS\_NOT\_DETERMINATE\_MESSAGE(actual, message)                             UNITY\_TEST\_ASSERT\_DOUBLE\_IS\_NOT\_DETERMINATE((actual), \_\_LINE\_\_, (message))|
||<p></p><p></p>|
||/\* Shorthand \*/|
||#ifdef UNITY\_SHORTHAND\_AS\_OLD|
||#define TEST\_ASSERT\_EQUAL\_MESSAGE(expected, actual, message)                                       UNITY\_TEST\_ASSERT\_EQUAL\_INT((expected), (actual), \_\_LINE\_\_, (message))|
||#define TEST\_ASSERT\_NOT\_EQUAL\_MESSAGE(expected, actual, message)                                   UNITY\_TEST\_ASSERT(((expected) != (actual)), \_\_LINE\_\_, (message))|
||#endif|
||#ifdef UNITY\_SHORTHAND\_AS\_INT|
||#define TEST\_ASSERT\_EQUAL\_MESSAGE(expected, actual, message)                                       UNITY\_TEST\_ASSERT\_EQUAL\_INT((expected), (actual), \_\_LINE\_\_, message)|
||#define TEST\_ASSERT\_NOT\_EQUAL\_MESSAGE(expected, actual, message)                                   UNITY\_TEST\_FAIL(\_\_LINE\_\_, UnityStrErrShorthand)|
||#endif|
||#ifdef  UNITY\_SHORTHAND\_AS\_MEM|
||#define TEST\_ASSERT\_EQUAL\_MESSAGE(expected, actual, message)                                       UNITY\_TEST\_ASSERT\_EQUAL\_MEMORY((&expected), (&actual), sizeof(expected), \_\_LINE\_\_, message)|
||#define TEST\_ASSERT\_NOT\_EQUAL\_MESSAGE(expected, actual, message)                                   UNITY\_TEST\_FAIL(\_\_LINE\_\_, UnityStrErrShorthand)|
||#endif|
||#ifdef  UNITY\_SHORTHAND\_AS\_RAW|
||#define TEST\_ASSERT\_EQUAL\_MESSAGE(expected, actual, message)                                       UNITY\_TEST\_ASSERT(((expected) == (actual)), \_\_LINE\_\_, message)|
||#define TEST\_ASSERT\_NOT\_EQUAL\_MESSAGE(expected, actual, message)                                   UNITY\_TEST\_ASSERT(((expected) != (actual)), \_\_LINE\_\_, message)|
||#endif|
||#ifdef UNITY\_SHORTHAND\_AS\_NONE|
||#define TEST\_ASSERT\_EQUAL\_MESSAGE(expected, actual, message)                                       UNITY\_TEST\_FAIL(\_\_LINE\_\_, UnityStrErrShorthand)|
||#define TEST\_ASSERT\_NOT\_EQUAL\_MESSAGE(expected, actual, message)                                   UNITY\_TEST\_FAIL(\_\_LINE\_\_, UnityStrErrShorthand)|
||#endif|
||<p></p><p></p>|
||/\* end of UNITY\_FRAMEWORK\_H \*/|
||#ifdef \_\_cplusplus|
||}|
||#endif|
||#endif|

