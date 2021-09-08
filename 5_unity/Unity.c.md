|/\* =========================================================================|
| :- |
||`    `Unity Project - A Test Framework for C|
||`    `Copyright (c) 2007-21 Mike Karlesky, Mark VanderVoord, Greg Williams|
||`    `[Released under MIT License. Please refer to license.txt for details]|
||============================================================================ \*/|
||<p></p><p></p>|
||#include "unity.h"|
||#include <stddef.h>|
||<p></p><p></p>|
||#ifdef AVR|
||#include <avr/pgmspace.h>|
||#else|
||#define PROGMEM|
||#endif|
||<p></p><p></p>|
||/\* If omitted from header, declare overrideable prototypes here so they're ready for use \*/|
||#ifdef UNITY\_OMIT\_OUTPUT\_CHAR\_HEADER\_DECLARATION|
||void UNITY\_OUTPUT\_CHAR(int);|
||#endif|
||<p></p><p></p>|
||/\* Helpful macros for us to use here in Assert functions \*/|
||#define UNITY\_FAIL\_AND\_BAIL   { Unity.CurrentTestFailed  = 1; UNITY\_OUTPUT\_FLUSH(); TEST\_ABORT(); }|
||#define UNITY\_IGNORE\_AND\_BAIL { Unity.CurrentTestIgnored = 1; UNITY\_OUTPUT\_FLUSH(); TEST\_ABORT(); }|
||#define RETURN\_IF\_FAIL\_OR\_IGNORE if (Unity.CurrentTestFailed || Unity.CurrentTestIgnored) TEST\_ABORT()|
||<p></p><p></p>|
||struct UNITY\_STORAGE\_T Unity;|
||<p></p><p></p>|
||#ifdef UNITY\_OUTPUT\_COLOR|
||const char PROGMEM UnityStrOk[]                            = "\033[42mOK\033[00m";|
||const char PROGMEM UnityStrPass[]                          = "\033[42mPASS\033[00m";|
||const char PROGMEM UnityStrFail[]                          = "\033[41mFAIL\033[00m";|
||const char PROGMEM UnityStrIgnore[]                        = "\033[43mIGNORE\033[00m";|
||#else|
||const char PROGMEM UnityStrOk[]                            = "OK";|
||const char PROGMEM UnityStrPass[]                          = "PASS";|
||const char PROGMEM UnityStrFail[]                          = "FAIL";|
||const char PROGMEM UnityStrIgnore[]                        = "IGNORE";|
||#endif|
||static const char PROGMEM UnityStrNull[]                   = "NULL";|
||static const char PROGMEM UnityStrSpacer[]                 = ". ";|
||static const char PROGMEM UnityStrExpected[]               = " Expected ";|
||static const char PROGMEM UnityStrWas[]                    = " Was ";|
||static const char PROGMEM UnityStrGt[]                     = " to be greater than ";|
||static const char PROGMEM UnityStrLt[]                     = " to be less than ";|
||static const char PROGMEM UnityStrOrEqual[]                = "or equal to ";|
||static const char PROGMEM UnityStrNotEqual[]               = " to be not equal to ";|
||static const char PROGMEM UnityStrElement[]                = " Element ";|
||static const char PROGMEM UnityStrByte[]                   = " Byte ";|
||static const char PROGMEM UnityStrMemory[]                 = " Memory Mismatch.";|
||static const char PROGMEM UnityStrDelta[]                  = " Values Not Within Delta ";|
||static const char PROGMEM UnityStrPointless[]              = " You Asked Me To Compare Nothing, Which Was Pointless.";|
||static const char PROGMEM UnityStrNullPointerForExpected[] = " Expected pointer to be NULL";|
||static const char PROGMEM UnityStrNullPointerForActual[]   = " Actual pointer was NULL";|
||#ifndef UNITY\_EXCLUDE\_FLOAT|
||static const char PROGMEM UnityStrNot[]                    = "Not ";|
||static const char PROGMEM UnityStrInf[]                    = "Infinity";|
||static const char PROGMEM UnityStrNegInf[]                 = "Negative Infinity";|
||static const char PROGMEM UnityStrNaN[]                    = "NaN";|
||static const char PROGMEM UnityStrDet[]                    = "Determinate";|
||static const char PROGMEM UnityStrInvalidFloatTrait[]      = "Invalid Float Trait";|
||#endif|
||const char PROGMEM UnityStrErrShorthand[]                  = "Unity Shorthand Support Disabled";|
||const char PROGMEM UnityStrErrFloat[]                      = "Unity Floating Point Disabled";|
||const char PROGMEM UnityStrErrDouble[]                     = "Unity Double Precision Disabled";|
||const char PROGMEM UnityStrErr64[]                         = "Unity 64-bit Support Disabled";|
||static const char PROGMEM UnityStrBreaker[]                = "-----------------------";|
||static const char PROGMEM UnityStrResultsTests[]           = " Tests ";|
||static const char PROGMEM UnityStrResultsFailures[]        = " Failures ";|
||static const char PROGMEM UnityStrResultsIgnored[]         = " Ignored ";|
||#ifndef UNITY\_EXCLUDE\_DETAILS|
||static const char PROGMEM UnityStrDetail1Name[]            = UNITY\_DETAIL1\_NAME " ";|
||static const char PROGMEM UnityStrDetail2Name[]            = " " UNITY\_DETAIL2\_NAME " ";|
||#endif|
||/\*-----------------------------------------------|
||` `\* Pretty Printers & Test Result Output Handlers|
||` `\*-----------------------------------------------\*/|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||/\* Local helper function to print characters. \*/|
||static void UnityPrintChar(const char\* pch)|
||{|
||`    `/\* printable characters plus CR & LF are printed \*/|
||`    `if ((\*pch <= 126) && (\*pch >= 32))|
||`    `{|
||`        `UNITY\_OUTPUT\_CHAR(\*pch);|
||`    `}|
||`    `/\* write escaped carriage returns \*/|
||`    `else if (\*pch == 13)|
||`    `{|
||`        `UNITY\_OUTPUT\_CHAR('\\');|
||`        `UNITY\_OUTPUT\_CHAR('r');|
||`    `}|
||`    `/\* write escaped line feeds \*/|
||`    `else if (\*pch == 10)|
||`    `{|
||`        `UNITY\_OUTPUT\_CHAR('\\');|
||`        `UNITY\_OUTPUT\_CHAR('n');|
||`    `}|
||`    `/\* unprintable characters are shown as codes \*/|
||`    `else|
||`    `{|
||`        `UNITY\_OUTPUT\_CHAR('\\');|
||`        `UNITY\_OUTPUT\_CHAR('x');|
||`        `UnityPrintNumberHex((UNITY\_UINT)\*pch, 2);|
||`    `}|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||/\* Local helper function to print ANSI escape strings e.g. "\033[42m". \*/|
||#ifdef UNITY\_OUTPUT\_COLOR|
||static UNITY\_UINT UnityPrintAnsiEscapeString(const char\* string)|
||{|
||`    `const char\* pch = string;|
||`    `UNITY\_UINT count = 0;|
||<p></p><p></p>|
||`    `while (\*pch && (\*pch != 'm'))|
||`    `{|
||`        `UNITY\_OUTPUT\_CHAR(\*pch);|
||`        `pch++;|
||`        `count++;|
||`    `}|
||`    `UNITY\_OUTPUT\_CHAR('m');|
||`    `count++;|
||<p></p><p></p>|
||`    `return count;|
||}|
||#endif|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityPrint(const char\* string)|
||{|
||`    `const char\* pch = string;|
||<p></p><p></p>|
||`    `if (pch != NULL)|
||`    `{|
||`        `while (\*pch)|
||`        `{|
||#ifdef UNITY\_OUTPUT\_COLOR|
||`            `/\* print ANSI escape code \*/|
||`            `if ((\*pch == 27) && (\*(pch + 1) == '['))|
||`            `{|
||`                `pch += UnityPrintAnsiEscapeString(pch);|
||`                `continue;|
||`            `}|
||#endif|
||`            `UnityPrintChar(pch);|
||`            `pch++;|
||`        `}|
||`    `}|
||}|
||/\*-----------------------------------------------\*/|
||void UnityPrintLen(const char\* string, const UNITY\_UINT32 length)|
||{|
||`    `const char\* pch = string;|
||<p></p><p></p>|
||`    `if (pch != NULL)|
||`    `{|
||`        `while (\*pch && ((UNITY\_UINT32)(pch - string) < length))|
||`        `{|
||`            `/\* printable characters plus CR & LF are printed \*/|
||`            `if ((\*pch <= 126) && (\*pch >= 32))|
||`            `{|
||`                `UNITY\_OUTPUT\_CHAR(\*pch);|
||`            `}|
||`            `/\* write escaped carriage returns \*/|
||`            `else if (\*pch == 13)|
||`            `{|
||`                `UNITY\_OUTPUT\_CHAR('\\');|
||`                `UNITY\_OUTPUT\_CHAR('r');|
||`            `}|
||`            `/\* write escaped line feeds \*/|
||`            `else if (\*pch == 10)|
||`            `{|
||`                `UNITY\_OUTPUT\_CHAR('\\');|
||`                `UNITY\_OUTPUT\_CHAR('n');|
||`            `}|
||`            `/\* unprintable characters are shown as codes \*/|
||`            `else|
||`            `{|
||`                `UNITY\_OUTPUT\_CHAR('\\');|
||`                `UNITY\_OUTPUT\_CHAR('x');|
||`                `UnityPrintNumberHex((UNITY\_UINT)\*pch, 2);|
||`            `}|
||`            `pch++;|
||`        `}|
||`    `}|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityPrintNumberByStyle(const UNITY\_INT number, const UNITY\_DISPLAY\_STYLE\_T style)|
||{|
||`    `if ((style & UNITY\_DISPLAY\_RANGE\_INT) == UNITY\_DISPLAY\_RANGE\_INT)|
||`    `{|
||`        `if (style == UNITY\_DISPLAY\_STYLE\_CHAR)|
||`        `{|
||`            `/\* printable characters plus CR & LF are printed \*/|
||`            `UNITY\_OUTPUT\_CHAR('\'');|
||`            `if ((number <= 126) && (number >= 32))|
||`            `{|
||`                `UNITY\_OUTPUT\_CHAR((int)number);|
||`            `}|
||`            `/\* write escaped carriage returns \*/|
||`            `else if (number == 13)|
||`            `{|
||`                `UNITY\_OUTPUT\_CHAR('\\');|
||`                `UNITY\_OUTPUT\_CHAR('r');|
||`            `}|
||`            `/\* write escaped line feeds \*/|
||`            `else if (number == 10)|
||`            `{|
||`                `UNITY\_OUTPUT\_CHAR('\\');|
||`                `UNITY\_OUTPUT\_CHAR('n');|
||`            `}|
||`            `/\* unprintable characters are shown as codes \*/|
||`            `else|
||`            `{|
||`                `UNITY\_OUTPUT\_CHAR('\\');|
||`                `UNITY\_OUTPUT\_CHAR('x');|
||`                `UnityPrintNumberHex((UNITY\_UINT)number, 2);|
||`            `}|
||`            `UNITY\_OUTPUT\_CHAR('\'');|
||`        `}|
||`        `else|
||`        `{|
||`            `UnityPrintNumber(number);|
||`        `}|
||`    `}|
||`    `else if ((style & UNITY\_DISPLAY\_RANGE\_UINT) == UNITY\_DISPLAY\_RANGE\_UINT)|
||`    `{|
||`        `UnityPrintNumberUnsigned((UNITY\_UINT)number);|
||`    `}|
||`    `else|
||`    `{|
||`        `UNITY\_OUTPUT\_CHAR('0');|
||`        `UNITY\_OUTPUT\_CHAR('x');|
||`        `UnityPrintNumberHex((UNITY\_UINT)number, (char)((style & 0xF) \* 2));|
||`    `}|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityPrintNumber(const UNITY\_INT number\_to\_print)|
||{|
||`    `UNITY\_UINT number = (UNITY\_UINT)number\_to\_print;|
||<p></p><p></p>|
||`    `if (number\_to\_print < 0)|
||`    `{|
||`        `/\* A negative number, including MIN negative \*/|
||`        `UNITY\_OUTPUT\_CHAR('-');|
||`        `number = (~number) + 1;|
||`    `}|
||`    `UnityPrintNumberUnsigned(number);|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------|
||` `\* basically do an itoa using as little ram as possible \*/|
||void UnityPrintNumberUnsigned(const UNITY\_UINT number)|
||{|
||`    `UNITY\_UINT divisor = 1;|
||<p></p><p></p>|
||`    `/\* figure out initial divisor \*/|
||`    `while (number / divisor > 9)|
||`    `{|
||`        `divisor \*= 10;|
||`    `}|
||<p></p><p></p>|
||`    `/\* now mod and print, then divide divisor \*/|
||`    `do|
||`    `{|
||`        `UNITY\_OUTPUT\_CHAR((char)('0' + (number / divisor % 10)));|
||`        `divisor /= 10;|
||`    `} while (divisor > 0);|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityPrintNumberHex(const UNITY\_UINT number, const char nibbles\_to\_print)|
||{|
||`    `int nibble;|
||`    `char nibbles = nibbles\_to\_print;|
||<p></p><p></p>|
||`    `if ((unsigned)nibbles > UNITY\_MAX\_NIBBLES)|
||`    `{|
||`        `nibbles = UNITY\_MAX\_NIBBLES;|
||`    `}|
||<p></p><p></p>|
||`    `while (nibbles > 0)|
||`    `{|
||`        `nibbles--;|
||`        `nibble = (int)(number >> (nibbles \* 4)) & 0x0F;|
||`        `if (nibble <= 9)|
||`        `{|
||`            `UNITY\_OUTPUT\_CHAR((char)('0' + nibble));|
||`        `}|
||`        `else|
||`        `{|
||`            `UNITY\_OUTPUT\_CHAR((char)('A' - 10 + nibble));|
||`        `}|
||`    `}|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityPrintMask(const UNITY\_UINT mask, const UNITY\_UINT number)|
||{|
||`    `UNITY\_UINT current\_bit = (UNITY\_UINT)1 << (UNITY\_INT\_WIDTH - 1);|
||`    `UNITY\_INT32 i;|
||<p></p><p></p>|
||`    `for (i = 0; i < UNITY\_INT\_WIDTH; i++)|
||`    `{|
||`        `if (current\_bit & mask)|
||`        `{|
||`            `if (current\_bit & number)|
||`            `{|
||`                `UNITY\_OUTPUT\_CHAR('1');|
||`            `}|
||`            `else|
||`            `{|
||`                `UNITY\_OUTPUT\_CHAR('0');|
||`            `}|
||`        `}|
||`        `else|
||`        `{|
||`            `UNITY\_OUTPUT\_CHAR('X');|
||`        `}|
||`        `current\_bit = current\_bit >> 1;|
||`    `}|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||#ifndef UNITY\_EXCLUDE\_FLOAT\_PRINT|
||/\*|
||` `\* This function prints a floating-point value in a format similar to|
||` `\* printf("%.7g") on a single-precision machine or printf("%.9g") on a|
||` `\* double-precision machine.  The 7th digit won't always be totally correct|
||` `\* in single-precision operation (for that level of accuracy, a more|
||` `\* complicated algorithm would be needed).|
||` `\*/|
||void UnityPrintFloat(const UNITY\_DOUBLE input\_number)|
||{|
||#ifdef UNITY\_INCLUDE\_DOUBLE|
||`    `static const int sig\_digits = 9;|
||`    `static const UNITY\_INT32 min\_scaled = 100000000;|
||`    `static const UNITY\_INT32 max\_scaled = 1000000000;|
||#else|
||`    `static const int sig\_digits = 7;|
||`    `static const UNITY\_INT32 min\_scaled = 1000000;|
||`    `static const UNITY\_INT32 max\_scaled = 10000000;|
||#endif|
||<p></p><p></p>|
||`    `UNITY\_DOUBLE number = input\_number;|
||<p></p><p></p>|
||`    `/\* print minus sign (does not handle negative zero) \*/|
||`    `if (number < 0.0f)|
||`    `{|
||`        `UNITY\_OUTPUT\_CHAR('-');|
||`        `number = -number;|
||`    `}|
||<p></p><p></p>|
||`    `/\* handle zero, NaN, and +/- infinity \*/|
||`    `if (number == 0.0f)|
||`    `{|
||`        `UnityPrint("0");|
||`    `}|
||`    `else if (isnan(number))|
||`    `{|
||`        `UnityPrint("nan");|
||`    `}|
||`    `else if (isinf(number))|
||`    `{|
||`        `UnityPrint("inf");|
||`    `}|
||`    `else|
||`    `{|
||`        `UNITY\_INT32 n\_int = 0, n;|
||`        `int exponent = 0;|
||`        `int decimals, digits;|
||`        `char buf[16] = {0};|
||<p></p><p></p>|
||`        `/\*|
||`         `\* Scale up or down by powers of 10.  To minimize rounding error,|
||`         `\* start with a factor/divisor of 10^10, which is the largest|
||`         `\* power of 10 that can be represented exactly.  Finally, compute|
||`         `\* (exactly) the remaining power of 10 and perform one more|
||`         `\* multiplication or division.|
||`         `\*/|
||`        `if (number < 1.0f)|
||`        `{|
||`            `UNITY\_DOUBLE factor = 1.0f;|
||<p></p><p></p>|
||`            `while (number < (UNITY\_DOUBLE)max\_scaled / 1e10f)  { number \*= 1e10f; exponent -= 10; }|
||`            `while (number \* factor < (UNITY\_DOUBLE)min\_scaled) { factor \*= 10.0f; exponent--; }|
||<p></p><p></p>|
||`            `number \*= factor;|
||`        `}|
||`        `else if (number > (UNITY\_DOUBLE)max\_scaled)|
||`        `{|
||`            `UNITY\_DOUBLE divisor = 1.0f;|
||<p></p><p></p>|
||`            `while (number > (UNITY\_DOUBLE)min\_scaled \* 1e10f)   { number  /= 1e10f; exponent += 10; }|
||`            `while (number / divisor > (UNITY\_DOUBLE)max\_scaled) { divisor \*= 10.0f; exponent++; }|
||<p></p><p></p>|
||`            `number /= divisor;|
||`        `}|
||`        `else|
||`        `{|
||`            `/\*|
||`             `\* In this range, we can split off the integer part before|
||`             `\* doing any multiplications.  This reduces rounding error by|
||`             `\* freeing up significant bits in the fractional part.|
||`             `\*/|
||`            `UNITY\_DOUBLE factor = 1.0f;|
||`            `n\_int = (UNITY\_INT32)number;|
||`            `number -= (UNITY\_DOUBLE)n\_int;|
||<p></p><p></p>|
||`            `while (n\_int < min\_scaled) { n\_int \*= 10; factor \*= 10.0f; exponent--; }|
||<p></p><p></p>|
||`            `number \*= factor;|
||`        `}|
||<p></p><p></p>|
||`        `/\* round to nearest integer \*/|
||`        `n = ((UNITY\_INT32)(number + number) + 1) / 2;|
||<p></p><p></p>|
||#ifndef UNITY\_ROUND\_TIES\_AWAY\_FROM\_ZERO|
||`        `/\* round to even if exactly between two integers \*/|
||`        `if ((n & 1) && (((UNITY\_DOUBLE)n - number) == 0.5f))|
||`            `n--;|
||#endif|
||<p></p><p></p>|
||`        `n += n\_int;|
||<p></p><p></p>|
||`        `if (n >= max\_scaled)|
||`        `{|
||`            `n = min\_scaled;|
||`            `exponent++;|
||`        `}|
||<p></p><p></p>|
||`        `/\* determine where to place decimal point \*/|
||`        `decimals = ((exponent <= 0) && (exponent >= -(sig\_digits + 3))) ? (-exponent) : (sig\_digits - 1);|
||`        `exponent += decimals;|
||<p></p><p></p>|
||`        `/\* truncate trailing zeroes after decimal point \*/|
||`        `while ((decimals > 0) && ((n % 10) == 0))|
||`        `{|
||`            `n /= 10;|
||`            `decimals--;|
||`        `}|
||<p></p><p></p>|
||`        `/\* build up buffer in reverse order \*/|
||`        `digits = 0;|
||`        `while ((n != 0) || (digits <= decimals))|
||`        `{|
||`            `buf[digits++] = (char)('0' + n % 10);|
||`            `n /= 10;|
||`        `}|
||`        `while (digits > 0)|
||`        `{|
||`            `if (digits == decimals) { UNITY\_OUTPUT\_CHAR('.'); }|
||`            `UNITY\_OUTPUT\_CHAR(buf[--digits]);|
||`        `}|
||<p></p><p></p>|
||`        `/\* print exponent if needed \*/|
||`        `if (exponent != 0)|
||`        `{|
||`            `UNITY\_OUTPUT\_CHAR('e');|
||<p></p><p></p>|
||`            `if (exponent < 0)|
||`            `{|
||`                `UNITY\_OUTPUT\_CHAR('-');|
||`                `exponent = -exponent;|
||`            `}|
||`            `else|
||`            `{|
||`                `UNITY\_OUTPUT\_CHAR('+');|
||`            `}|
||<p></p><p></p>|
||`            `digits = 0;|
||`            `while ((exponent != 0) || (digits < 2))|
||`            `{|
||`                `buf[digits++] = (char)('0' + exponent % 10);|
||`                `exponent /= 10;|
||`            `}|
||`            `while (digits > 0)|
||`            `{|
||`                `UNITY\_OUTPUT\_CHAR(buf[--digits]);|
||`            `}|
||`        `}|
||`    `}|
||}|
||#endif /\* ! UNITY\_EXCLUDE\_FLOAT\_PRINT \*/|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||static void UnityTestResultsBegin(const char\* file, const UNITY\_LINE\_TYPE line)|
||{|
||#ifdef UNITY\_OUTPUT\_FOR\_ECLIPSE|
||`    `UNITY\_OUTPUT\_CHAR('(');|
||`    `UnityPrint(file);|
||`    `UNITY\_OUTPUT\_CHAR(':');|
||`    `UnityPrintNumber((UNITY\_INT)line);|
||`    `UNITY\_OUTPUT\_CHAR(')');|
||`    `UNITY\_OUTPUT\_CHAR(' ');|
||`    `UnityPrint(Unity.CurrentTestName);|
||`    `UNITY\_OUTPUT\_CHAR(':');|
||#else|
||#ifdef UNITY\_OUTPUT\_FOR\_IAR\_WORKBENCH|
||`    `UnityPrint("<SRCREF line=");|
||`    `UnityPrintNumber((UNITY\_INT)line);|
||`    `UnityPrint(" file=\"");|
||`    `UnityPrint(file);|
||`    `UNITY\_OUTPUT\_CHAR('"');|
||`    `UNITY\_OUTPUT\_CHAR('>');|
||`    `UnityPrint(Unity.CurrentTestName);|
||`    `UnityPrint("</SRCREF> ");|
||#else|
||#ifdef UNITY\_OUTPUT\_FOR\_QT\_CREATOR|
||`    `UnityPrint("file://");|
||`    `UnityPrint(file);|
||`    `UNITY\_OUTPUT\_CHAR(':');|
||`    `UnityPrintNumber((UNITY\_INT)line);|
||`    `UNITY\_OUTPUT\_CHAR(' ');|
||`    `UnityPrint(Unity.CurrentTestName);|
||`    `UNITY\_OUTPUT\_CHAR(':');|
||#else|
||`    `UnityPrint(file);|
||`    `UNITY\_OUTPUT\_CHAR(':');|
||`    `UnityPrintNumber((UNITY\_INT)line);|
||`    `UNITY\_OUTPUT\_CHAR(':');|
||`    `UnityPrint(Unity.CurrentTestName);|
||`    `UNITY\_OUTPUT\_CHAR(':');|
||#endif|
||#endif|
||#endif|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||static void UnityTestResultsFailBegin(const UNITY\_LINE\_TYPE line)|
||{|
||`    `UnityTestResultsBegin(Unity.TestFile, line);|
||`    `UnityPrint(UnityStrFail);|
||`    `UNITY\_OUTPUT\_CHAR(':');|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityConcludeTest(void)|
||{|
||`    `if (Unity.CurrentTestIgnored)|
||`    `{|
||`        `Unity.TestIgnores++;|
||`    `}|
||`    `else if (!Unity.CurrentTestFailed)|
||`    `{|
||`        `UnityTestResultsBegin(Unity.TestFile, Unity.CurrentTestLineNumber);|
||`        `UnityPrint(UnityStrPass);|
||`    `}|
||`    `else|
||`    `{|
||`        `Unity.TestFailures++;|
||`    `}|
||<p></p><p></p>|
||`    `Unity.CurrentTestFailed = 0;|
||`    `Unity.CurrentTestIgnored = 0;|
||`    `UNITY\_PRINT\_EXEC\_TIME();|
||`    `UNITY\_PRINT\_EOL();|
||`    `UNITY\_FLUSH\_CALL();|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||static void UnityAddMsgIfSpecified(const char\* msg)|
||{|
||`    `if (msg)|
||`    `{|
||`        `UnityPrint(UnityStrSpacer);|
||<p></p><p></p>|
||#ifdef UNITY\_PRINT\_TEST\_CONTEXT|
||`        `UNITY\_PRINT\_TEST\_CONTEXT();|
||#endif|
||#ifndef UNITY\_EXCLUDE\_DETAILS|
||`        `if (Unity.CurrentDetail1)|
||`        `{|
||`            `UnityPrint(UnityStrDetail1Name);|
||`            `UnityPrint(Unity.CurrentDetail1);|
||`            `if (Unity.CurrentDetail2)|
||`            `{|
||`                `UnityPrint(UnityStrDetail2Name);|
||`                `UnityPrint(Unity.CurrentDetail2);|
||`            `}|
||`            `UnityPrint(UnityStrSpacer);|
||`        `}|
||#endif|
||`        `UnityPrint(msg);|
||`    `}|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||static void UnityPrintExpectedAndActualStrings(const char\* expected, const char\* actual)|
||{|
||`    `UnityPrint(UnityStrExpected);|
||`    `if (expected != NULL)|
||`    `{|
||`        `UNITY\_OUTPUT\_CHAR('\'');|
||`        `UnityPrint(expected);|
||`        `UNITY\_OUTPUT\_CHAR('\'');|
||`    `}|
||`    `else|
||`    `{|
||`        `UnityPrint(UnityStrNull);|
||`    `}|
||`    `UnityPrint(UnityStrWas);|
||`    `if (actual != NULL)|
||`    `{|
||`        `UNITY\_OUTPUT\_CHAR('\'');|
||`        `UnityPrint(actual);|
||`        `UNITY\_OUTPUT\_CHAR('\'');|
||`    `}|
||`    `else|
||`    `{|
||`        `UnityPrint(UnityStrNull);|
||`    `}|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||static void UnityPrintExpectedAndActualStringsLen(const char\* expected,|
||`                                                  `const char\* actual,|
||`                                                  `const UNITY\_UINT32 length)|
||{|
||`    `UnityPrint(UnityStrExpected);|
||`    `if (expected != NULL)|
||`    `{|
||`        `UNITY\_OUTPUT\_CHAR('\'');|
||`        `UnityPrintLen(expected, length);|
||`        `UNITY\_OUTPUT\_CHAR('\'');|
||`    `}|
||`    `else|
||`    `{|
||`        `UnityPrint(UnityStrNull);|
||`    `}|
||`    `UnityPrint(UnityStrWas);|
||`    `if (actual != NULL)|
||`    `{|
||`        `UNITY\_OUTPUT\_CHAR('\'');|
||`        `UnityPrintLen(actual, length);|
||`        `UNITY\_OUTPUT\_CHAR('\'');|
||`    `}|
||`    `else|
||`    `{|
||`        `UnityPrint(UnityStrNull);|
||`    `}|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------|
||` `\* Assertion & Control Helpers|
||` `\*-----------------------------------------------\*/|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||static int UnityIsOneArrayNull(UNITY\_INTERNAL\_PTR expected,|
||`                               `UNITY\_INTERNAL\_PTR actual,|
||`                               `const UNITY\_LINE\_TYPE lineNumber,|
||`                               `const char\* msg)|
||{|
||`    `/\* Both are NULL or same pointer \*/|
||`    `if (expected == actual) { return 0; }|
||<p></p><p></p>|
||`    `/\* print and return true if just expected is NULL \*/|
||`    `if (expected == NULL)|
||`    `{|
||`        `UnityTestResultsFailBegin(lineNumber);|
||`        `UnityPrint(UnityStrNullPointerForExpected);|
||`        `UnityAddMsgIfSpecified(msg);|
||`        `return 1;|
||`    `}|
||<p></p><p></p>|
||`    `/\* print and return true if just actual is NULL \*/|
||`    `if (actual == NULL)|
||`    `{|
||`        `UnityTestResultsFailBegin(lineNumber);|
||`        `UnityPrint(UnityStrNullPointerForActual);|
||`        `UnityAddMsgIfSpecified(msg);|
||`        `return 1;|
||`    `}|
||<p></p><p></p>|
||`    `return 0; /\* return false if neither is NULL \*/|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------|
||` `\* Assertion Functions|
||` `\*-----------------------------------------------\*/|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityAssertBits(const UNITY\_INT mask,|
||`                     `const UNITY\_INT expected,|
||`                     `const UNITY\_INT actual,|
||`                     `const char\* msg,|
||`                     `const UNITY\_LINE\_TYPE lineNumber)|
||{|
||`    `RETURN\_IF\_FAIL\_OR\_IGNORE;|
||<p></p><p></p>|
||`    `if ((mask & expected) != (mask & actual))|
||`    `{|
||`        `UnityTestResultsFailBegin(lineNumber);|
||`        `UnityPrint(UnityStrExpected);|
||`        `UnityPrintMask((UNITY\_UINT)mask, (UNITY\_UINT)expected);|
||`        `UnityPrint(UnityStrWas);|
||`        `UnityPrintMask((UNITY\_UINT)mask, (UNITY\_UINT)actual);|
||`        `UnityAddMsgIfSpecified(msg);|
||`        `UNITY\_FAIL\_AND\_BAIL;|
||`    `}|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityAssertEqualNumber(const UNITY\_INT expected,|
||`                            `const UNITY\_INT actual,|
||`                            `const char\* msg,|
||`                            `const UNITY\_LINE\_TYPE lineNumber,|
||`                            `const UNITY\_DISPLAY\_STYLE\_T style)|
||{|
||`    `RETURN\_IF\_FAIL\_OR\_IGNORE;|
||<p></p><p></p>|
||`    `if (expected != actual)|
||`    `{|
||`        `UnityTestResultsFailBegin(lineNumber);|
||`        `UnityPrint(UnityStrExpected);|
||`        `UnityPrintNumberByStyle(expected, style);|
||`        `UnityPrint(UnityStrWas);|
||`        `UnityPrintNumberByStyle(actual, style);|
||`        `UnityAddMsgIfSpecified(msg);|
||`        `UNITY\_FAIL\_AND\_BAIL;|
||`    `}|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityAssertGreaterOrLessOrEqualNumber(const UNITY\_INT threshold,|
||`                                           `const UNITY\_INT actual,|
||`                                           `const UNITY\_COMPARISON\_T compare,|
||`                                           `const char \*msg,|
||`                                           `const UNITY\_LINE\_TYPE lineNumber,|
||`                                           `const UNITY\_DISPLAY\_STYLE\_T style)|
||{|
||`    `int failed = 0;|
||`    `RETURN\_IF\_FAIL\_OR\_IGNORE;|
||<p></p><p></p>|
||`    `if ((threshold == actual) && (compare & UNITY\_EQUAL\_TO)) { return; }|
||`    `if ((threshold == actual))                               { failed = 1; }|
||<p></p><p></p>|
||`    `if ((style & UNITY\_DISPLAY\_RANGE\_INT) == UNITY\_DISPLAY\_RANGE\_INT)|
||`    `{|
||`        `if ((actual > threshold) && (compare & UNITY\_SMALLER\_THAN)) { failed = 1; }|
||`        `if ((actual < threshold) && (compare & UNITY\_GREATER\_THAN)) { failed = 1; }|
||`    `}|
||`    `else /\* UINT or HEX \*/|
||`    `{|
||`        `if (((UNITY\_UINT)actual > (UNITY\_UINT)threshold) && (compare & UNITY\_SMALLER\_THAN)) { failed = 1; }|
||`        `if (((UNITY\_UINT)actual < (UNITY\_UINT)threshold) && (compare & UNITY\_GREATER\_THAN)) { failed = 1; }|
||`    `}|
||<p></p><p></p>|
||`    `if (failed)|
||`    `{|
||`        `UnityTestResultsFailBegin(lineNumber);|
||`        `UnityPrint(UnityStrExpected);|
||`        `UnityPrintNumberByStyle(actual, style);|
||`        `if (compare & UNITY\_GREATER\_THAN) { UnityPrint(UnityStrGt);       }|
||`        `if (compare & UNITY\_SMALLER\_THAN) { UnityPrint(UnityStrLt);       }|
||`        `if (compare & UNITY\_EQUAL\_TO)     { UnityPrint(UnityStrOrEqual);  }|
||`        `if (compare == UNITY\_NOT\_EQUAL)   { UnityPrint(UnityStrNotEqual); }|
||`        `UnityPrintNumberByStyle(threshold, style);|
||`        `UnityAddMsgIfSpecified(msg);|
||`        `UNITY\_FAIL\_AND\_BAIL;|
||`    `}|
||}|
||<p></p><p></p>|
||#define UnityPrintPointlessAndBail()       \|
||{                                          \|
||`    `UnityTestResultsFailBegin(lineNumber); \|
||`    `UnityPrint(UnityStrPointless);         \|
||`    `UnityAddMsgIfSpecified(msg);           \|
||`    `UNITY\_FAIL\_AND\_BAIL; }|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityAssertEqualIntArray(UNITY\_INTERNAL\_PTR expected,|
||`                              `UNITY\_INTERNAL\_PTR actual,|
||`                              `const UNITY\_UINT32 num\_elements,|
||`                              `const char\* msg,|
||`                              `const UNITY\_LINE\_TYPE lineNumber,|
||`                              `const UNITY\_DISPLAY\_STYLE\_T style,|
||`                              `const UNITY\_FLAGS\_T flags)|
||{|
||`    `UNITY\_UINT32 elements  = num\_elements;|
||`    `unsigned int length    = style & 0xF;|
||`    `unsigned int increment = 0;|
||<p></p><p></p>|
||`    `RETURN\_IF\_FAIL\_OR\_IGNORE;|
||<p></p><p></p>|
||`    `if (num\_elements == 0)|
||`    `{|
||`        `UnityPrintPointlessAndBail();|
||`    `}|
||<p></p><p></p>|
||`    `if (expected == actual)|
||`    `{|
||`        `return; /\* Both are NULL or same pointer \*/|
||`    `}|
||<p></p><p></p>|
||`    `if (UnityIsOneArrayNull(expected, actual, lineNumber, msg))|
||`    `{|
||`        `UNITY\_FAIL\_AND\_BAIL;|
||`    `}|
||<p></p><p></p>|
||`    `while ((elements > 0) && (elements--))|
||`    `{|
||`        `UNITY\_INT expect\_val;|
||`        `UNITY\_INT actual\_val;|
||<p></p><p></p>|
||`        `switch (length)|
||`        `{|
||`            `case 1:|
||`                `expect\_val = \*(UNITY\_PTR\_ATTRIBUTE const UNITY\_INT8\*)expected;|
||`                `actual\_val = \*(UNITY\_PTR\_ATTRIBUTE const UNITY\_INT8\*)actual;|
||`                `increment  = sizeof(UNITY\_INT8);|
||`                `break;|
||<p></p><p></p>|
||`            `case 2:|
||`                `expect\_val = \*(UNITY\_PTR\_ATTRIBUTE const UNITY\_INT16\*)expected;|
||`                `actual\_val = \*(UNITY\_PTR\_ATTRIBUTE const UNITY\_INT16\*)actual;|
||`                `increment  = sizeof(UNITY\_INT16);|
||`                `break;|
||<p></p><p></p>|
||#ifdef UNITY\_SUPPORT\_64|
||`            `case 8:|
||`                `expect\_val = \*(UNITY\_PTR\_ATTRIBUTE const UNITY\_INT64\*)expected;|
||`                `actual\_val = \*(UNITY\_PTR\_ATTRIBUTE const UNITY\_INT64\*)actual;|
||`                `increment  = sizeof(UNITY\_INT64);|
||`                `break;|
||#endif|
||<p></p><p></p>|
||`            `default: /\* default is length 4 bytes \*/|
||`            `case 4:|
||`                `expect\_val = \*(UNITY\_PTR\_ATTRIBUTE const UNITY\_INT32\*)expected;|
||`                `actual\_val = \*(UNITY\_PTR\_ATTRIBUTE const UNITY\_INT32\*)actual;|
||`                `increment  = sizeof(UNITY\_INT32);|
||`                `length = 4;|
||`                `break;|
||`        `}|
||<p></p><p></p>|
||`        `if (expect\_val != actual\_val)|
||`        `{|
||`            `if ((style & UNITY\_DISPLAY\_RANGE\_UINT) && (length < (UNITY\_INT\_WIDTH / 8)))|
||`            `{   /\* For UINT, remove sign extension (padding 1's) from signed type casts above \*/|
||`                `UNITY\_INT mask = 1;|
||`                `mask = (mask << 8 \* length) - 1;|
||`                `expect\_val &= mask;|
||`                `actual\_val &= mask;|
||`            `}|
||`            `UnityTestResultsFailBegin(lineNumber);|
||`            `UnityPrint(UnityStrElement);|
||`            `UnityPrintNumberUnsigned(num\_elements - elements - 1);|
||`            `UnityPrint(UnityStrExpected);|
||`            `UnityPrintNumberByStyle(expect\_val, style);|
||`            `UnityPrint(UnityStrWas);|
||`            `UnityPrintNumberByStyle(actual\_val, style);|
||`            `UnityAddMsgIfSpecified(msg);|
||`            `UNITY\_FAIL\_AND\_BAIL;|
||`        `}|
||`        `/\* Walk through array by incrementing the pointers \*/|
||`        `if (flags == UNITY\_ARRAY\_TO\_ARRAY)|
||`        `{|
||`            `expected = (UNITY\_INTERNAL\_PTR)((const char\*)expected + increment);|
||`        `}|
||`        `actual = (UNITY\_INTERNAL\_PTR)((const char\*)actual + increment);|
||`    `}|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||#ifndef UNITY\_EXCLUDE\_FLOAT|
||/\* Wrap this define in a function with variable types as float or double \*/|
||#define UNITY\_FLOAT\_OR\_DOUBLE\_WITHIN(delta, expected, actual, diff)                           \|
||`    `if (isinf(expected) && isinf(actual) && (((expected) < 0) == ((actual) < 0))) return 1;   \|
||`    `if (UNITY\_NAN\_CHECK) return 1;                                                            \|
||`    `(diff) = (actual) - (expected);                                                           \|
||`    `if ((diff) < 0) (diff) = -(diff);                                                         \|
||`    `if ((delta) < 0) (delta) = -(delta);                                                      \|
||`    `return !(isnan(diff) || isinf(diff) || ((diff) > (delta)))|
||`    `/\* This first part of this condition will catch any NaN or Infinite values \*/|
||#ifndef UNITY\_NAN\_NOT\_EQUAL\_NAN|
||`  `#define UNITY\_NAN\_CHECK isnan(expected) && isnan(actual)|
||#else|
||`  `#define UNITY\_NAN\_CHECK 0|
||#endif|
||<p></p><p></p>|
||#ifndef UNITY\_EXCLUDE\_FLOAT\_PRINT|
||`  `#define UNITY\_PRINT\_EXPECTED\_AND\_ACTUAL\_FLOAT(expected, actual) \|
||`  `{                                                               \|
||`    `UnityPrint(UnityStrExpected);                                 \|
||`    `UnityPrintFloat(expected);                                    \|
||`    `UnityPrint(UnityStrWas);                                      \|
||`    `UnityPrintFloat(actual); }|
||#else|
||`  `#define UNITY\_PRINT\_EXPECTED\_AND\_ACTUAL\_FLOAT(expected, actual) \|
||`    `UnityPrint(UnityStrDelta)|
||#endif /\* UNITY\_EXCLUDE\_FLOAT\_PRINT \*/|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||static int UnityFloatsWithin(UNITY\_FLOAT delta, UNITY\_FLOAT expected, UNITY\_FLOAT actual)|
||{|
||`    `UNITY\_FLOAT diff;|
||`    `UNITY\_FLOAT\_OR\_DOUBLE\_WITHIN(delta, expected, actual, diff);|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityAssertEqualFloatArray(UNITY\_PTR\_ATTRIBUTE const UNITY\_FLOAT\* expected,|
||`                                `UNITY\_PTR\_ATTRIBUTE const UNITY\_FLOAT\* actual,|
||`                                `const UNITY\_UINT32 num\_elements,|
||`                                `const char\* msg,|
||`                                `const UNITY\_LINE\_TYPE lineNumber,|
||`                                `const UNITY\_FLAGS\_T flags)|
||{|
||`    `UNITY\_UINT32 elements = num\_elements;|
||`    `UNITY\_PTR\_ATTRIBUTE const UNITY\_FLOAT\* ptr\_expected = expected;|
||`    `UNITY\_PTR\_ATTRIBUTE const UNITY\_FLOAT\* ptr\_actual = actual;|
||<p></p><p></p>|
||`    `RETURN\_IF\_FAIL\_OR\_IGNORE;|
||<p></p><p></p>|
||`    `if (elements == 0)|
||`    `{|
||`        `UnityPrintPointlessAndBail();|
||`    `}|
||<p></p><p></p>|
||`    `if (expected == actual)|
||`    `{|
||`        `return; /\* Both are NULL or same pointer \*/|
||`    `}|
||<p></p><p></p>|
||`    `if (UnityIsOneArrayNull((UNITY\_INTERNAL\_PTR)expected, (UNITY\_INTERNAL\_PTR)actual, lineNumber, msg))|
||`    `{|
||`        `UNITY\_FAIL\_AND\_BAIL;|
||`    `}|
||<p></p><p></p>|
||`    `while (elements--)|
||`    `{|
||`        `if (!UnityFloatsWithin(\*ptr\_expected \* UNITY\_FLOAT\_PRECISION, \*ptr\_expected, \*ptr\_actual))|
||`        `{|
||`            `UnityTestResultsFailBegin(lineNumber);|
||`            `UnityPrint(UnityStrElement);|
||`            `UnityPrintNumberUnsigned(num\_elements - elements - 1);|
||`            `UNITY\_PRINT\_EXPECTED\_AND\_ACTUAL\_FLOAT((UNITY\_DOUBLE)\*ptr\_expected, (UNITY\_DOUBLE)\*ptr\_actual);|
||`            `UnityAddMsgIfSpecified(msg);|
||`            `UNITY\_FAIL\_AND\_BAIL;|
||`        `}|
||`        `if (flags == UNITY\_ARRAY\_TO\_ARRAY)|
||`        `{|
||`            `ptr\_expected++;|
||`        `}|
||`        `ptr\_actual++;|
||`    `}|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityAssertFloatsWithin(const UNITY\_FLOAT delta,|
||`                             `const UNITY\_FLOAT expected,|
||`                             `const UNITY\_FLOAT actual,|
||`                             `const char\* msg,|
||`                             `const UNITY\_LINE\_TYPE lineNumber)|
||{|
||`    `RETURN\_IF\_FAIL\_OR\_IGNORE;|
||<p></p><p></p>|
||<p></p><p></p>|
||`    `if (!UnityFloatsWithin(delta, expected, actual))|
||`    `{|
||`        `UnityTestResultsFailBegin(lineNumber);|
||`        `UNITY\_PRINT\_EXPECTED\_AND\_ACTUAL\_FLOAT((UNITY\_DOUBLE)expected, (UNITY\_DOUBLE)actual);|
||`        `UnityAddMsgIfSpecified(msg);|
||`        `UNITY\_FAIL\_AND\_BAIL;|
||`    `}|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityAssertFloatSpecial(const UNITY\_FLOAT actual,|
||`                             `const char\* msg,|
||`                             `const UNITY\_LINE\_TYPE lineNumber,|
||`                             `const UNITY\_FLOAT\_TRAIT\_T style)|
||{|
||`    `const char\* trait\_names[] = {UnityStrInf, UnityStrNegInf, UnityStrNaN, UnityStrDet};|
||`    `UNITY\_INT should\_be\_trait = ((UNITY\_INT)style & 1);|
||`    `UNITY\_INT is\_trait        = !should\_be\_trait;|
||`    `UNITY\_INT trait\_index     = (UNITY\_INT)(style >> 1);|
||<p></p><p></p>|
||`    `RETURN\_IF\_FAIL\_OR\_IGNORE;|
||<p></p><p></p>|
||`    `switch (style)|
||`    `{|
||`        `case UNITY\_FLOAT\_IS\_INF:|
||`        `case UNITY\_FLOAT\_IS\_NOT\_INF:|
||`            `is\_trait = isinf(actual) && (actual > 0);|
||`            `break;|
||`        `case UNITY\_FLOAT\_IS\_NEG\_INF:|
||`        `case UNITY\_FLOAT\_IS\_NOT\_NEG\_INF:|
||`            `is\_trait = isinf(actual) && (actual < 0);|
||`            `break;|
||<p></p><p></p>|
||`        `case UNITY\_FLOAT\_IS\_NAN:|
||`        `case UNITY\_FLOAT\_IS\_NOT\_NAN:|
||`            `is\_trait = isnan(actual) ? 1 : 0;|
||`            `break;|
||<p></p><p></p>|
||`        `case UNITY\_FLOAT\_IS\_DET: /\* A determinate number is non infinite and not NaN. \*/|
||`        `case UNITY\_FLOAT\_IS\_NOT\_DET:|
||`            `is\_trait = !isinf(actual) && !isnan(actual);|
||`            `break;|
||<p></p><p></p>|
||`        `default: /\* including UNITY\_FLOAT\_INVALID\_TRAIT \*/|
||`            `trait\_index = 0;|
||`            `trait\_names[0] = UnityStrInvalidFloatTrait;|
||`            `break;|
||`    `}|
||<p></p><p></p>|
||`    `if (is\_trait != should\_be\_trait)|
||`    `{|
||`        `UnityTestResultsFailBegin(lineNumber);|
||`        `UnityPrint(UnityStrExpected);|
||`        `if (!should\_be\_trait)|
||`        `{|
||`            `UnityPrint(UnityStrNot);|
||`        `}|
||`        `UnityPrint(trait\_names[trait\_index]);|
||`        `UnityPrint(UnityStrWas);|
||#ifndef UNITY\_EXCLUDE\_FLOAT\_PRINT|
||`        `UnityPrintFloat((UNITY\_DOUBLE)actual);|
||#else|
||`        `if (should\_be\_trait)|
||`        `{|
||`            `UnityPrint(UnityStrNot);|
||`        `}|
||`        `UnityPrint(trait\_names[trait\_index]);|
||#endif|
||`        `UnityAddMsgIfSpecified(msg);|
||`        `UNITY\_FAIL\_AND\_BAIL;|
||`    `}|
||}|
||<p></p><p></p>|
||#endif /\* not UNITY\_EXCLUDE\_FLOAT \*/|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||#ifndef UNITY\_EXCLUDE\_DOUBLE|
||static int UnityDoublesWithin(UNITY\_DOUBLE delta, UNITY\_DOUBLE expected, UNITY\_DOUBLE actual)|
||{|
||`    `UNITY\_DOUBLE diff;|
||`    `UNITY\_FLOAT\_OR\_DOUBLE\_WITHIN(delta, expected, actual, diff);|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityAssertEqualDoubleArray(UNITY\_PTR\_ATTRIBUTE const UNITY\_DOUBLE\* expected,|
||`                                 `UNITY\_PTR\_ATTRIBUTE const UNITY\_DOUBLE\* actual,|
||`                                 `const UNITY\_UINT32 num\_elements,|
||`                                 `const char\* msg,|
||`                                 `const UNITY\_LINE\_TYPE lineNumber,|
||`                                 `const UNITY\_FLAGS\_T flags)|
||{|
||`    `UNITY\_UINT32 elements = num\_elements;|
||`    `UNITY\_PTR\_ATTRIBUTE const UNITY\_DOUBLE\* ptr\_expected = expected;|
||`    `UNITY\_PTR\_ATTRIBUTE const UNITY\_DOUBLE\* ptr\_actual = actual;|
||<p></p><p></p>|
||`    `RETURN\_IF\_FAIL\_OR\_IGNORE;|
||<p></p><p></p>|
||`    `if (elements == 0)|
||`    `{|
||`        `UnityPrintPointlessAndBail();|
||`    `}|
||<p></p><p></p>|
||`    `if (expected == actual)|
||`    `{|
||`        `return; /\* Both are NULL or same pointer \*/|
||`    `}|
||<p></p><p></p>|
||`    `if (UnityIsOneArrayNull((UNITY\_INTERNAL\_PTR)expected, (UNITY\_INTERNAL\_PTR)actual, lineNumber, msg))|
||`    `{|
||`        `UNITY\_FAIL\_AND\_BAIL;|
||`    `}|
||<p></p><p></p>|
||`    `while (elements--)|
||`    `{|
||`        `if (!UnityDoublesWithin(\*ptr\_expected \* UNITY\_DOUBLE\_PRECISION, \*ptr\_expected, \*ptr\_actual))|
||`        `{|
||`            `UnityTestResultsFailBegin(lineNumber);|
||`            `UnityPrint(UnityStrElement);|
||`            `UnityPrintNumberUnsigned(num\_elements - elements - 1);|
||`            `UNITY\_PRINT\_EXPECTED\_AND\_ACTUAL\_FLOAT(\*ptr\_expected, \*ptr\_actual);|
||`            `UnityAddMsgIfSpecified(msg);|
||`            `UNITY\_FAIL\_AND\_BAIL;|
||`        `}|
||`        `if (flags == UNITY\_ARRAY\_TO\_ARRAY)|
||`        `{|
||`            `ptr\_expected++;|
||`        `}|
||`        `ptr\_actual++;|
||`    `}|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityAssertDoublesWithin(const UNITY\_DOUBLE delta,|
||`                              `const UNITY\_DOUBLE expected,|
||`                              `const UNITY\_DOUBLE actual,|
||`                              `const char\* msg,|
||`                              `const UNITY\_LINE\_TYPE lineNumber)|
||{|
||`    `RETURN\_IF\_FAIL\_OR\_IGNORE;|
||<p></p><p></p>|
||`    `if (!UnityDoublesWithin(delta, expected, actual))|
||`    `{|
||`        `UnityTestResultsFailBegin(lineNumber);|
||`        `UNITY\_PRINT\_EXPECTED\_AND\_ACTUAL\_FLOAT(expected, actual);|
||`        `UnityAddMsgIfSpecified(msg);|
||`        `UNITY\_FAIL\_AND\_BAIL;|
||`    `}|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityAssertDoubleSpecial(const UNITY\_DOUBLE actual,|
||`                              `const char\* msg,|
||`                              `const UNITY\_LINE\_TYPE lineNumber,|
||`                              `const UNITY\_FLOAT\_TRAIT\_T style)|
||{|
||`    `const char\* trait\_names[] = {UnityStrInf, UnityStrNegInf, UnityStrNaN, UnityStrDet};|
||`    `UNITY\_INT should\_be\_trait = ((UNITY\_INT)style & 1);|
||`    `UNITY\_INT is\_trait        = !should\_be\_trait;|
||`    `UNITY\_INT trait\_index     = (UNITY\_INT)(style >> 1);|
||<p></p><p></p>|
||`    `RETURN\_IF\_FAIL\_OR\_IGNORE;|
||<p></p><p></p>|
||`    `switch (style)|
||`    `{|
||`        `case UNITY\_FLOAT\_IS\_INF:|
||`        `case UNITY\_FLOAT\_IS\_NOT\_INF:|
||`            `is\_trait = isinf(actual) && (actual > 0);|
||`            `break;|
||`        `case UNITY\_FLOAT\_IS\_NEG\_INF:|
||`        `case UNITY\_FLOAT\_IS\_NOT\_NEG\_INF:|
||`            `is\_trait = isinf(actual) && (actual < 0);|
||`            `break;|
||<p></p><p></p>|
||`        `case UNITY\_FLOAT\_IS\_NAN:|
||`        `case UNITY\_FLOAT\_IS\_NOT\_NAN:|
||`            `is\_trait = isnan(actual) ? 1 : 0;|
||`            `break;|
||<p></p><p></p>|
||`        `case UNITY\_FLOAT\_IS\_DET: /\* A determinate number is non infinite and not NaN. \*/|
||`        `case UNITY\_FLOAT\_IS\_NOT\_DET:|
||`            `is\_trait = !isinf(actual) && !isnan(actual);|
||`            `break;|
||<p></p><p></p>|
||`        `default: /\* including UNITY\_FLOAT\_INVALID\_TRAIT \*/|
||`            `trait\_index = 0;|
||`            `trait\_names[0] = UnityStrInvalidFloatTrait;|
||`            `break;|
||`    `}|
||<p></p><p></p>|
||`    `if (is\_trait != should\_be\_trait)|
||`    `{|
||`        `UnityTestResultsFailBegin(lineNumber);|
||`        `UnityPrint(UnityStrExpected);|
||`        `if (!should\_be\_trait)|
||`        `{|
||`            `UnityPrint(UnityStrNot);|
||`        `}|
||`        `UnityPrint(trait\_names[trait\_index]);|
||`        `UnityPrint(UnityStrWas);|
||#ifndef UNITY\_EXCLUDE\_FLOAT\_PRINT|
||`        `UnityPrintFloat(actual);|
||#else|
||`        `if (should\_be\_trait)|
||`        `{|
||`            `UnityPrint(UnityStrNot);|
||`        `}|
||`        `UnityPrint(trait\_names[trait\_index]);|
||#endif|
||`        `UnityAddMsgIfSpecified(msg);|
||`        `UNITY\_FAIL\_AND\_BAIL;|
||`    `}|
||}|
||<p></p><p></p>|
||#endif /\* not UNITY\_EXCLUDE\_DOUBLE \*/|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityAssertNumbersWithin(const UNITY\_UINT delta,|
||`                              `const UNITY\_INT expected,|
||`                              `const UNITY\_INT actual,|
||`                              `const char\* msg,|
||`                              `const UNITY\_LINE\_TYPE lineNumber,|
||`                              `const UNITY\_DISPLAY\_STYLE\_T style)|
||{|
||`    `RETURN\_IF\_FAIL\_OR\_IGNORE;|
||<p></p><p></p>|
||`    `if ((style & UNITY\_DISPLAY\_RANGE\_INT) == UNITY\_DISPLAY\_RANGE\_INT)|
||`    `{|
||`        `if (actual > expected)|
||`        `{|
||`            `Unity.CurrentTestFailed = (((UNITY\_UINT)actual - (UNITY\_UINT)expected) > delta);|
||`        `}|
||`        `else|
||`        `{|
||`            `Unity.CurrentTestFailed = (((UNITY\_UINT)expected - (UNITY\_UINT)actual) > delta);|
||`        `}|
||`    `}|
||`    `else|
||`    `{|
||`        `if ((UNITY\_UINT)actual > (UNITY\_UINT)expected)|
||`        `{|
||`            `Unity.CurrentTestFailed = (((UNITY\_UINT)actual - (UNITY\_UINT)expected) > delta);|
||`        `}|
||`        `else|
||`        `{|
||`            `Unity.CurrentTestFailed = (((UNITY\_UINT)expected - (UNITY\_UINT)actual) > delta);|
||`        `}|
||`    `}|
||<p></p><p></p>|
||`    `if (Unity.CurrentTestFailed)|
||`    `{|
||`        `UnityTestResultsFailBegin(lineNumber);|
||`        `UnityPrint(UnityStrDelta);|
||`        `UnityPrintNumberByStyle((UNITY\_INT)delta, style);|
||`        `UnityPrint(UnityStrExpected);|
||`        `UnityPrintNumberByStyle(expected, style);|
||`        `UnityPrint(UnityStrWas);|
||`        `UnityPrintNumberByStyle(actual, style);|
||`        `UnityAddMsgIfSpecified(msg);|
||`        `UNITY\_FAIL\_AND\_BAIL;|
||`    `}|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityAssertNumbersArrayWithin(const UNITY\_UINT delta,|
||`                                   `UNITY\_INTERNAL\_PTR expected,|
||`                                   `UNITY\_INTERNAL\_PTR actual,|
||`                                   `const UNITY\_UINT32 num\_elements,|
||`                                   `const char\* msg,|
||`                                   `const UNITY\_LINE\_TYPE lineNumber,|
||`                                   `const UNITY\_DISPLAY\_STYLE\_T style,|
||`                                   `const UNITY\_FLAGS\_T flags)|
||{|
||`    `UNITY\_UINT32 elements = num\_elements;|
||`    `unsigned int length   = style & 0xF;|
||`    `unsigned int increment = 0;|
||<p></p><p></p>|
||`    `RETURN\_IF\_FAIL\_OR\_IGNORE;|
||<p></p><p></p>|
||`    `if (num\_elements == 0)|
||`    `{|
||`        `UnityPrintPointlessAndBail();|
||`    `}|
||<p></p><p></p>|
||`    `if (expected == actual)|
||`    `{|
||`        `return; /\* Both are NULL or same pointer \*/|
||`    `}|
||<p></p><p></p>|
||`    `if (UnityIsOneArrayNull(expected, actual, lineNumber, msg))|
||`    `{|
||`        `UNITY\_FAIL\_AND\_BAIL;|
||`    `}|
||<p></p><p></p>|
||`    `while ((elements > 0) && (elements--))|
||`    `{|
||`        `UNITY\_INT expect\_val;|
||`        `UNITY\_INT actual\_val;|
||<p></p><p></p>|
||`        `switch (length)|
||`        `{|
||`            `case 1:|
||`                `expect\_val = \*(UNITY\_PTR\_ATTRIBUTE const UNITY\_INT8\*)expected;|
||`                `actual\_val = \*(UNITY\_PTR\_ATTRIBUTE const UNITY\_INT8\*)actual;|
||`                `increment  = sizeof(UNITY\_INT8);|
||`                `break;|
||<p></p><p></p>|
||`            `case 2:|
||`                `expect\_val = \*(UNITY\_PTR\_ATTRIBUTE const UNITY\_INT16\*)expected;|
||`                `actual\_val = \*(UNITY\_PTR\_ATTRIBUTE const UNITY\_INT16\*)actual;|
||`                `increment  = sizeof(UNITY\_INT16);|
||`                `break;|
||<p></p><p></p>|
||#ifdef UNITY\_SUPPORT\_64|
||`            `case 8:|
||`                `expect\_val = \*(UNITY\_PTR\_ATTRIBUTE const UNITY\_INT64\*)expected;|
||`                `actual\_val = \*(UNITY\_PTR\_ATTRIBUTE const UNITY\_INT64\*)actual;|
||`                `increment  = sizeof(UNITY\_INT64);|
||`                `break;|
||#endif|
||<p></p><p></p>|
||`            `default: /\* default is length 4 bytes \*/|
||`            `case 4:|
||`                `expect\_val = \*(UNITY\_PTR\_ATTRIBUTE const UNITY\_INT32\*)expected;|
||`                `actual\_val = \*(UNITY\_PTR\_ATTRIBUTE const UNITY\_INT32\*)actual;|
||`                `increment  = sizeof(UNITY\_INT32);|
||`                `length = 4;|
||`                `break;|
||`        `}|
||<p></p><p></p>|
||`        `if ((style & UNITY\_DISPLAY\_RANGE\_INT) == UNITY\_DISPLAY\_RANGE\_INT)|
||`        `{|
||`            `if (actual\_val > expect\_val)|
||`            `{|
||`                `Unity.CurrentTestFailed = (((UNITY\_UINT)actual\_val - (UNITY\_UINT)expect\_val) > delta);|
||`            `}|
||`            `else|
||`            `{|
||`                `Unity.CurrentTestFailed = (((UNITY\_UINT)expect\_val - (UNITY\_UINT)actual\_val) > delta);|
||`            `}|
||`        `}|
||`        `else|
||`        `{|
||`            `if ((UNITY\_UINT)actual\_val > (UNITY\_UINT)expect\_val)|
||`            `{|
||`                `Unity.CurrentTestFailed = (((UNITY\_UINT)actual\_val - (UNITY\_UINT)expect\_val) > delta);|
||`            `}|
||`            `else|
||`            `{|
||`                `Unity.CurrentTestFailed = (((UNITY\_UINT)expect\_val - (UNITY\_UINT)actual\_val) > delta);|
||`            `}|
||`        `}|
||<p></p><p></p>|
||`        `if (Unity.CurrentTestFailed)|
||`        `{|
||`            `if ((style & UNITY\_DISPLAY\_RANGE\_UINT) && (length < (UNITY\_INT\_WIDTH / 8)))|
||`            `{   /\* For UINT, remove sign extension (padding 1's) from signed type casts above \*/|
||`                `UNITY\_INT mask = 1;|
||`                `mask = (mask << 8 \* length) - 1;|
||`                `expect\_val &= mask;|
||`                `actual\_val &= mask;|
||`            `}|
||`            `UnityTestResultsFailBegin(lineNumber);|
||`            `UnityPrint(UnityStrDelta);|
||`            `UnityPrintNumberByStyle((UNITY\_INT)delta, style);|
||`            `UnityPrint(UnityStrElement);|
||`            `UnityPrintNumberUnsigned(num\_elements - elements - 1);|
||`            `UnityPrint(UnityStrExpected);|
||`            `UnityPrintNumberByStyle(expect\_val, style);|
||`            `UnityPrint(UnityStrWas);|
||`            `UnityPrintNumberByStyle(actual\_val, style);|
||`            `UnityAddMsgIfSpecified(msg);|
||`            `UNITY\_FAIL\_AND\_BAIL;|
||`        `}|
||`        `/\* Walk through array by incrementing the pointers \*/|
||`        `if (flags == UNITY\_ARRAY\_TO\_ARRAY)|
||`        `{|
||`            `expected = (UNITY\_INTERNAL\_PTR)((const char\*)expected + increment);|
||`        `}|
||`        `actual = (UNITY\_INTERNAL\_PTR)((const char\*)actual + increment);|
||`    `}|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityAssertEqualString(const char\* expected,|
||`                            `const char\* actual,|
||`                            `const char\* msg,|
||`                            `const UNITY\_LINE\_TYPE lineNumber)|
||{|
||`    `UNITY\_UINT32 i;|
||<p></p><p></p>|
||`    `RETURN\_IF\_FAIL\_OR\_IGNORE;|
||<p></p><p></p>|
||`    `/\* if both pointers not null compare the strings \*/|
||`    `if (expected && actual)|
||`    `{|
||`        `for (i = 0; expected[i] || actual[i]; i++)|
||`        `{|
||`            `if (expected[i] != actual[i])|
||`            `{|
||`                `Unity.CurrentTestFailed = 1;|
||`                `break;|
||`            `}|
||`        `}|
||`    `}|
||`    `else|
||`    `{ /\* handle case of one pointers being null (if both null, test should pass) \*/|
||`        `if (expected != actual)|
||`        `{|
||`            `Unity.CurrentTestFailed = 1;|
||`        `}|
||`    `}|
||<p></p><p></p>|
||`    `if (Unity.CurrentTestFailed)|
||`    `{|
||`        `UnityTestResultsFailBegin(lineNumber);|
||`        `UnityPrintExpectedAndActualStrings(expected, actual);|
||`        `UnityAddMsgIfSpecified(msg);|
||`        `UNITY\_FAIL\_AND\_BAIL;|
||`    `}|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityAssertEqualStringLen(const char\* expected,|
||`                               `const char\* actual,|
||`                               `const UNITY\_UINT32 length,|
||`                               `const char\* msg,|
||`                               `const UNITY\_LINE\_TYPE lineNumber)|
||{|
||`    `UNITY\_UINT32 i;|
||<p></p><p></p>|
||`    `RETURN\_IF\_FAIL\_OR\_IGNORE;|
||<p></p><p></p>|
||`    `/\* if both pointers not null compare the strings \*/|
||`    `if (expected && actual)|
||`    `{|
||`        `for (i = 0; (i < length) && (expected[i] || actual[i]); i++)|
||`        `{|
||`            `if (expected[i] != actual[i])|
||`            `{|
||`                `Unity.CurrentTestFailed = 1;|
||`                `break;|
||`            `}|
||`        `}|
||`    `}|
||`    `else|
||`    `{ /\* handle case of one pointers being null (if both null, test should pass) \*/|
||`        `if (expected != actual)|
||`        `{|
||`            `Unity.CurrentTestFailed = 1;|
||`        `}|
||`    `}|
||<p></p><p></p>|
||`    `if (Unity.CurrentTestFailed)|
||`    `{|
||`        `UnityTestResultsFailBegin(lineNumber);|
||`        `UnityPrintExpectedAndActualStringsLen(expected, actual, length);|
||`        `UnityAddMsgIfSpecified(msg);|
||`        `UNITY\_FAIL\_AND\_BAIL;|
||`    `}|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityAssertEqualStringArray(UNITY\_INTERNAL\_PTR expected,|
||`                                 `const char\*\* actual,|
||`                                 `const UNITY\_UINT32 num\_elements,|
||`                                 `const char\* msg,|
||`                                 `const UNITY\_LINE\_TYPE lineNumber,|
||`                                 `const UNITY\_FLAGS\_T flags)|
||{|
||`    `UNITY\_UINT32 i = 0;|
||`    `UNITY\_UINT32 j = 0;|
||`    `const char\* expd = NULL;|
||`    `const char\* act = NULL;|
||<p></p><p></p>|
||`    `RETURN\_IF\_FAIL\_OR\_IGNORE;|
||<p></p><p></p>|
||`    `/\* if no elements, it's an error \*/|
||`    `if (num\_elements == 0)|
||`    `{|
||`        `UnityPrintPointlessAndBail();|
||`    `}|
||<p></p><p></p>|
||`    `if ((const void\*)expected == (const void\*)actual)|
||`    `{|
||`        `return; /\* Both are NULL or same pointer \*/|
||`    `}|
||<p></p><p></p>|
||`    `if (UnityIsOneArrayNull((UNITY\_INTERNAL\_PTR)expected, (UNITY\_INTERNAL\_PTR)actual, lineNumber, msg))|
||`    `{|
||`        `UNITY\_FAIL\_AND\_BAIL;|
||`    `}|
||<p></p><p></p>|
||`    `if (flags != UNITY\_ARRAY\_TO\_ARRAY)|
||`    `{|
||`        `expd = (const char\*)expected;|
||`    `}|
||<p></p><p></p>|
||`    `do|
||`    `{|
||`        `act = actual[j];|
||`        `if (flags == UNITY\_ARRAY\_TO\_ARRAY)|
||`        `{|
||`            `expd = ((const char\* const\*)expected)[j];|
||`        `}|
||<p></p><p></p>|
||`        `/\* if both pointers not null compare the strings \*/|
||`        `if (expd && act)|
||`        `{|
||`            `for (i = 0; expd[i] || act[i]; i++)|
||`            `{|
||`                `if (expd[i] != act[i])|
||`                `{|
||`                    `Unity.CurrentTestFailed = 1;|
||`                    `break;|
||`                `}|
||`            `}|
||`        `}|
||`        `else|
||`        `{ /\* handle case of one pointers being null (if both null, test should pass) \*/|
||`            `if (expd != act)|
||`            `{|
||`                `Unity.CurrentTestFailed = 1;|
||`            `}|
||`        `}|
||<p></p><p></p>|
||`        `if (Unity.CurrentTestFailed)|
||`        `{|
||`            `UnityTestResultsFailBegin(lineNumber);|
||`            `if (num\_elements > 1)|
||`            `{|
||`                `UnityPrint(UnityStrElement);|
||`                `UnityPrintNumberUnsigned(j);|
||`            `}|
||`            `UnityPrintExpectedAndActualStrings(expd, act);|
||`            `UnityAddMsgIfSpecified(msg);|
||`            `UNITY\_FAIL\_AND\_BAIL;|
||`        `}|
||`    `} while (++j < num\_elements);|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityAssertEqualMemory(UNITY\_INTERNAL\_PTR expected,|
||`                            `UNITY\_INTERNAL\_PTR actual,|
||`                            `const UNITY\_UINT32 length,|
||`                            `const UNITY\_UINT32 num\_elements,|
||`                            `const char\* msg,|
||`                            `const UNITY\_LINE\_TYPE lineNumber,|
||`                            `const UNITY\_FLAGS\_T flags)|
||{|
||`    `UNITY\_PTR\_ATTRIBUTE const unsigned char\* ptr\_exp = (UNITY\_PTR\_ATTRIBUTE const unsigned char\*)expected;|
||`    `UNITY\_PTR\_ATTRIBUTE const unsigned char\* ptr\_act = (UNITY\_PTR\_ATTRIBUTE const unsigned char\*)actual;|
||`    `UNITY\_UINT32 elements = num\_elements;|
||`    `UNITY\_UINT32 bytes;|
||<p></p><p></p>|
||`    `RETURN\_IF\_FAIL\_OR\_IGNORE;|
||<p></p><p></p>|
||`    `if ((elements == 0) || (length == 0))|
||`    `{|
||`        `UnityPrintPointlessAndBail();|
||`    `}|
||<p></p><p></p>|
||`    `if (expected == actual)|
||`    `{|
||`        `return; /\* Both are NULL or same pointer \*/|
||`    `}|
||<p></p><p></p>|
||`    `if (UnityIsOneArrayNull(expected, actual, lineNumber, msg))|
||`    `{|
||`        `UNITY\_FAIL\_AND\_BAIL;|
||`    `}|
||<p></p><p></p>|
||`    `while (elements--)|
||`    `{|
||`        `bytes = length;|
||`        `while (bytes--)|
||`        `{|
||`            `if (\*ptr\_exp != \*ptr\_act)|
||`            `{|
||`                `UnityTestResultsFailBegin(lineNumber);|
||`                `UnityPrint(UnityStrMemory);|
||`                `if (num\_elements > 1)|
||`                `{|
||`                    `UnityPrint(UnityStrElement);|
||`                    `UnityPrintNumberUnsigned(num\_elements - elements - 1);|
||`                `}|
||`                `UnityPrint(UnityStrByte);|
||`                `UnityPrintNumberUnsigned(length - bytes - 1);|
||`                `UnityPrint(UnityStrExpected);|
||`                `UnityPrintNumberByStyle(\*ptr\_exp, UNITY\_DISPLAY\_STYLE\_HEX8);|
||`                `UnityPrint(UnityStrWas);|
||`                `UnityPrintNumberByStyle(\*ptr\_act, UNITY\_DISPLAY\_STYLE\_HEX8);|
||`                `UnityAddMsgIfSpecified(msg);|
||`                `UNITY\_FAIL\_AND\_BAIL;|
||`            `}|
||`            `ptr\_exp++;|
||`            `ptr\_act++;|
||`        `}|
||`        `if (flags == UNITY\_ARRAY\_TO\_VAL)|
||`        `{|
||`            `ptr\_exp = (UNITY\_PTR\_ATTRIBUTE const unsigned char\*)expected;|
||`        `}|
||`    `}|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||<p></p><p></p>|
||static union|
||{|
||`    `UNITY\_INT8 i8;|
||`    `UNITY\_INT16 i16;|
||`    `UNITY\_INT32 i32;|
||#ifdef UNITY\_SUPPORT\_64|
||`    `UNITY\_INT64 i64;|
||#endif|
||#ifndef UNITY\_EXCLUDE\_FLOAT|
||`    `float f;|
||#endif|
||#ifndef UNITY\_EXCLUDE\_DOUBLE|
||`    `double d;|
||#endif|
||} UnityQuickCompare;|
||<p></p><p></p>|
||UNITY\_INTERNAL\_PTR UnityNumToPtr(const UNITY\_INT num, const UNITY\_UINT8 size)|
||{|
||`    `switch(size)|
||`    `{|
||`        `case 1:|
||`            `UnityQuickCompare.i8 = (UNITY\_INT8)num;|
||`            `return (UNITY\_INTERNAL\_PTR)(&UnityQuickCompare.i8);|
||<p></p><p></p>|
||`        `case 2:|
||`            `UnityQuickCompare.i16 = (UNITY\_INT16)num;|
||`            `return (UNITY\_INTERNAL\_PTR)(&UnityQuickCompare.i16);|
||<p></p><p></p>|
||#ifdef UNITY\_SUPPORT\_64|
||`        `case 8:|
||`            `UnityQuickCompare.i64 = (UNITY\_INT64)num;|
||`            `return (UNITY\_INTERNAL\_PTR)(&UnityQuickCompare.i64);|
||#endif|
||<p></p><p></p>|
||`        `default: /\* 4 bytes \*/|
||`            `UnityQuickCompare.i32 = (UNITY\_INT32)num;|
||`            `return (UNITY\_INTERNAL\_PTR)(&UnityQuickCompare.i32);|
||`    `}|
||}|
||<p></p><p></p>|
||#ifndef UNITY\_EXCLUDE\_FLOAT|
||/\*-----------------------------------------------\*/|
||UNITY\_INTERNAL\_PTR UnityFloatToPtr(const float num)|
||{|
||`    `UnityQuickCompare.f = num;|
||`    `return (UNITY\_INTERNAL\_PTR)(&UnityQuickCompare.f);|
||}|
||#endif|
||<p></p><p></p>|
||#ifndef UNITY\_EXCLUDE\_DOUBLE|
||/\*-----------------------------------------------\*/|
||UNITY\_INTERNAL\_PTR UnityDoubleToPtr(const double num)|
||{|
||`    `UnityQuickCompare.d = num;|
||`    `return (UNITY\_INTERNAL\_PTR)(&UnityQuickCompare.d);|
||}|
||#endif|
||<p></p><p></p>|
||/\*-----------------------------------------------|
||` `\* printf helper function|
||` `\*-----------------------------------------------\*/|
||#ifdef UNITY\_INCLUDE\_PRINT\_FORMATTED|
||static void UnityPrintFVA(const char\* format, va\_list va)|
||{|
||`    `const char\* pch = format;|
||`    `if (pch != NULL)|
||`    `{|
||`        `while (\*pch)|
||`        `{|
||`            `/\* format identification character \*/|
||`            `if (\*pch == '%')|
||`            `{|
||`                `pch++;|
||<p></p><p></p>|
||`                `if (pch != NULL)|
||`                `{|
||`                    `switch (\*pch)|
||`                    `{|
||`                        `case 'd':|
||`                        `case 'i':|
||`                            `{|
||`                                `const int number = va\_arg(va, int);|
||`                                `UnityPrintNumber((UNITY\_INT)number);|
||`                                `break;|
||`                            `}|
||#ifndef UNITY\_EXCLUDE\_FLOAT\_PRINT|
||`                        `case 'f':|
||`                        `case 'g':|
||`                            `{|
||`                                `const double number = va\_arg(va, double);|
||`                                `UnityPrintFloat((UNITY\_DOUBLE)number);|
||`                                `break;|
||`                            `}|
||#endif|
||`                        `case 'u':|
||`                            `{|
||`                                `const unsigned int number = va\_arg(va, unsigned int);|
||`                                `UnityPrintNumberUnsigned((UNITY\_UINT)number);|
||`                                `break;|
||`                            `}|
||`                        `case 'b':|
||`                            `{|
||`                                `const unsigned int number = va\_arg(va, unsigned int);|
||`                                `const UNITY\_UINT mask = (UNITY\_UINT)0 - (UNITY\_UINT)1;|
||`                                `UNITY\_OUTPUT\_CHAR('0');|
||`                                `UNITY\_OUTPUT\_CHAR('b');|
||`                                `UnityPrintMask(mask, (UNITY\_UINT)number);|
||`                                `break;|
||`                            `}|
||`                        `case 'x':|
||`                        `case 'X':|
||`                        `case 'p':|
||`                            `{|
||`                                `const unsigned int number = va\_arg(va, unsigned int);|
||`                                `UNITY\_OUTPUT\_CHAR('0');|
||`                                `UNITY\_OUTPUT\_CHAR('x');|
||`                                `UnityPrintNumberHex((UNITY\_UINT)number, 8);|
||`                                `break;|
||`                            `}|
||`                        `case 'c':|
||`                            `{|
||`                                `const int ch = va\_arg(va, int);|
||`                                `UnityPrintChar((const char \*)&ch);|
||`                                `break;|
||`                            `}|
||`                        `case 's':|
||`                            `{|
||`                                `const char \* string = va\_arg(va, const char \*);|
||`                                `UnityPrint(string);|
||`                                `break;|
||`                            `}|
||`                        `case '%':|
||`                            `{|
||`                                `UnityPrintChar(pch);|
||`                                `break;|
||`                            `}|
||`                        `default:|
||`                            `{|
||`                                `/\* print the unknown format character \*/|
||`                                `UNITY\_OUTPUT\_CHAR('%');|
||`                                `UnityPrintChar(pch);|
||`                                `break;|
||`                            `}|
||`                    `}|
||`                `}|
||`            `}|
||#ifdef UNITY\_OUTPUT\_COLOR|
||`            `/\* print ANSI escape code \*/|
||`            `else if ((\*pch == 27) && (\*(pch + 1) == '['))|
||`            `{|
||`                `pch += UnityPrintAnsiEscapeString(pch);|
||`                `continue;|
||`            `}|
||#endif|
||`            `else if (\*pch == '\n')|
||`            `{|
||`                `UNITY\_PRINT\_EOL();|
||`            `}|
||`            `else|
||`            `{|
||`                `UnityPrintChar(pch);|
||`            `}|
||<p></p><p></p>|
||`            `pch++;|
||`        `}|
||`    `}|
||}|
||<p></p><p></p>|
||void UnityPrintF(const UNITY\_LINE\_TYPE line, const char\* format, ...)|
||{|
||`    `UnityTestResultsBegin(Unity.TestFile, line);|
||`    `UnityPrint("INFO");|
||`    `if(format != NULL)|
||`    `{|
||`        `UnityPrint(": ");|
||`        `va\_list va;|
||`        `va\_start(va, format);|
||`        `UnityPrintFVA(format, va);|
||`        `va\_end(va);|
||`    `}|
||`    `UNITY\_PRINT\_EOL();|
||}|
||#endif /\* ! UNITY\_INCLUDE\_PRINT\_FORMATTED \*/|
||<p></p><p></p>|
||<p></p><p></p>|
||/\*-----------------------------------------------|
||` `\* Control Functions|
||` `\*-----------------------------------------------\*/|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityFail(const char\* msg, const UNITY\_LINE\_TYPE line)|
||{|
||`    `RETURN\_IF\_FAIL\_OR\_IGNORE;|
||<p></p><p></p>|
||`    `UnityTestResultsBegin(Unity.TestFile, line);|
||`    `UnityPrint(UnityStrFail);|
||`    `if (msg != NULL)|
||`    `{|
||`        `UNITY\_OUTPUT\_CHAR(':');|
||<p></p><p></p>|
||#ifdef UNITY\_PRINT\_TEST\_CONTEXT|
||`        `UNITY\_PRINT\_TEST\_CONTEXT();|
||#endif|
||#ifndef UNITY\_EXCLUDE\_DETAILS|
||`        `if (Unity.CurrentDetail1)|
||`        `{|
||`            `UnityPrint(UnityStrDetail1Name);|
||`            `UnityPrint(Unity.CurrentDetail1);|
||`            `if (Unity.CurrentDetail2)|
||`            `{|
||`                `UnityPrint(UnityStrDetail2Name);|
||`                `UnityPrint(Unity.CurrentDetail2);|
||`            `}|
||`            `UnityPrint(UnityStrSpacer);|
||`        `}|
||#endif|
||`        `if (msg[0] != ' ')|
||`        `{|
||`            `UNITY\_OUTPUT\_CHAR(' ');|
||`        `}|
||`        `UnityPrint(msg);|
||`    `}|
||<p></p><p></p>|
||`    `UNITY\_FAIL\_AND\_BAIL;|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityIgnore(const char\* msg, const UNITY\_LINE\_TYPE line)|
||{|
||`    `RETURN\_IF\_FAIL\_OR\_IGNORE;|
||<p></p><p></p>|
||`    `UnityTestResultsBegin(Unity.TestFile, line);|
||`    `UnityPrint(UnityStrIgnore);|
||`    `if (msg != NULL)|
||`    `{|
||`        `UNITY\_OUTPUT\_CHAR(':');|
||`        `UNITY\_OUTPUT\_CHAR(' ');|
||`        `UnityPrint(msg);|
||`    `}|
||`    `UNITY\_IGNORE\_AND\_BAIL;|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityMessage(const char\* msg, const UNITY\_LINE\_TYPE line)|
||{|
||`    `UnityTestResultsBegin(Unity.TestFile, line);|
||`    `UnityPrint("INFO");|
||`    `if (msg != NULL)|
||`    `{|
||`      `UNITY\_OUTPUT\_CHAR(':');|
||`      `UNITY\_OUTPUT\_CHAR(' ');|
||`      `UnityPrint(msg);|
||`    `}|
||`    `UNITY\_PRINT\_EOL();|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||/\* If we have not defined our own test runner, then include our default test runner to make life easier \*/|
||#ifndef UNITY\_SKIP\_DEFAULT\_RUNNER|
||void UnityDefaultTestRun(UnityTestFunction Func, const char\* FuncName, const int FuncLineNum)|
||{|
||`    `Unity.CurrentTestName = FuncName;|
||`    `Unity.CurrentTestLineNumber = (UNITY\_LINE\_TYPE)FuncLineNum;|
||`    `Unity.NumberOfTests++;|
||`    `UNITY\_CLR\_DETAILS();|
||`    `UNITY\_EXEC\_TIME\_START();|
||`    `if (TEST\_PROTECT())|
||`    `{|
||`        `setUp();|
||`        `Func();|
||`    `}|
||`    `if (TEST\_PROTECT())|
||`    `{|
||`        `tearDown();|
||`    `}|
||`    `UNITY\_EXEC\_TIME\_STOP();|
||`    `UnityConcludeTest();|
||}|
||#endif|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnitySetTestFile(const char\* filename)|
||{|
||`	`Unity.TestFile = filename;|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||void UnityBegin(const char\* filename)|
||{|
||`    `Unity.TestFile = filename;|
||`    `Unity.CurrentTestName = NULL;|
||`    `Unity.CurrentTestLineNumber = 0;|
||`    `Unity.NumberOfTests = 0;|
||`    `Unity.TestFailures = 0;|
||`    `Unity.TestIgnores = 0;|
||`    `Unity.CurrentTestFailed = 0;|
||`    `Unity.CurrentTestIgnored = 0;|
||<p></p><p></p>|
||`    `UNITY\_CLR\_DETAILS();|
||`    `UNITY\_OUTPUT\_START();|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||int UnityEnd(void)|
||{|
||`    `UNITY\_PRINT\_EOL();|
||`    `UnityPrint(UnityStrBreaker);|
||`    `UNITY\_PRINT\_EOL();|
||`    `UnityPrintNumber((UNITY\_INT)(Unity.NumberOfTests));|
||`    `UnityPrint(UnityStrResultsTests);|
||`    `UnityPrintNumber((UNITY\_INT)(Unity.TestFailures));|
||`    `UnityPrint(UnityStrResultsFailures);|
||`    `UnityPrintNumber((UNITY\_INT)(Unity.TestIgnores));|
||`    `UnityPrint(UnityStrResultsIgnored);|
||`    `UNITY\_PRINT\_EOL();|
||`    `if (Unity.TestFailures == 0U)|
||`    `{|
||`        `UnityPrint(UnityStrOk);|
||`    `}|
||`    `else|
||`    `{|
||`        `UnityPrint(UnityStrFail);|
||#ifdef UNITY\_DIFFERENTIATE\_FINAL\_FAIL|
||`        `UNITY\_OUTPUT\_CHAR('E'); UNITY\_OUTPUT\_CHAR('D');|
||#endif|
||`    `}|
||`    `UNITY\_PRINT\_EOL();|
||`    `UNITY\_FLUSH\_CALL();|
||`    `UNITY\_OUTPUT\_COMPLETE();|
||`    `return (int)(Unity.TestFailures);|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------|
||` `\* Command Line Argument Support|
||` `\*-----------------------------------------------\*/|
||#ifdef UNITY\_USE\_COMMAND\_LINE\_ARGS|
||<p></p><p></p>|
||char\* UnityOptionIncludeNamed = NULL;|
||char\* UnityOptionExcludeNamed = NULL;|
||int UnityVerbosity            = 1;|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||int UnityParseOptions(int argc, char\*\* argv)|
||{|
||`    `int i;|
||`    `UnityOptionIncludeNamed = NULL;|
||`    `UnityOptionExcludeNamed = NULL;|
||<p></p><p></p>|
||`    `for (i = 1; i < argc; i++)|
||`    `{|
||`        `if (argv[i][0] == '-')|
||`        `{|
||`            `switch (argv[i][1])|
||`            `{|
||`                `case 'l': /\* list tests \*/|
||`                    `return -1;|
||`                `case 'n': /\* include tests with name including this string \*/|
||`                `case 'f': /\* an alias for -n \*/|
||`                    `if (argv[i][2] == '=')|
||`                    `{|
||`                        `UnityOptionIncludeNamed = &argv[i][3];|
||`                    `}|
||`                    `else if (++i < argc)|
||`                    `{|
||`                        `UnityOptionIncludeNamed = argv[i];|
||`                    `}|
||`                    `else|
||`                    `{|
||`                        `UnityPrint("ERROR: No Test String to Include Matches For");|
||`                        `UNITY\_PRINT\_EOL();|
||`                        `return 1;|
||`                    `}|
||`                    `break;|
||`                `case 'q': /\* quiet \*/|
||`                    `UnityVerbosity = 0;|
||`                    `break;|
||`                `case 'v': /\* verbose \*/|
||`                    `UnityVerbosity = 2;|
||`                    `break;|
||`                `case 'x': /\* exclude tests with name including this string \*/|
||`                    `if (argv[i][2] == '=')|
||`                    `{|
||`                        `UnityOptionExcludeNamed = &argv[i][3];|
||`                    `}|
||`                    `else if (++i < argc)|
||`                    `{|
||`                        `UnityOptionExcludeNamed = argv[i];|
||`                    `}|
||`                    `else|
||`                    `{|
||`                        `UnityPrint("ERROR: No Test String to Exclude Matches For");|
||`                        `UNITY\_PRINT\_EOL();|
||`                        `return 1;|
||`                    `}|
||`                    `break;|
||`                `default:|
||`                    `UnityPrint("ERROR: Unknown Option ");|
||`                    `UNITY\_OUTPUT\_CHAR(argv[i][1]);|
||`                    `UNITY\_PRINT\_EOL();|
||`                    `return 1;|
||`            `}|
||`        `}|
||`    `}|
||<p></p><p></p>|
||`    `return 0;|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||int IsStringInBiggerString(const char\* longstring, const char\* shortstring)|
||{|
||`    `const char\* lptr = longstring;|
||`    `const char\* sptr = shortstring;|
||`    `const char\* lnext = lptr;|
||<p></p><p></p>|
||`    `if (\*sptr == '\*')|
||`    `{|
||`        `return 1;|
||`    `}|
||<p></p><p></p>|
||`    `while (\*lptr)|
||`    `{|
||`        `lnext = lptr + 1;|
||<p></p><p></p>|
||`        `/\* If they current bytes match, go on to the next bytes \*/|
||`        `while (\*lptr && \*sptr && (\*lptr == \*sptr))|
||`        `{|
||`            `lptr++;|
||`            `sptr++;|
||<p></p><p></p>|
||`            `/\* We're done if we match the entire string or up to a wildcard \*/|
||`            `if (\*sptr == '\*')|
||`                `return 1;|
||`            `if (\*sptr == ',')|
||`                `return 1;|
||`            `if (\*sptr == '"')|
||`                `return 1;|
||`            `if (\*sptr == '\'')|
||`                `return 1;|
||`            `if (\*sptr == ':')|
||`                `return 2;|
||`            `if (\*sptr == 0)|
||`                `return 1;|
||`        `}|
||<p></p><p></p>|
||`        `/\* Otherwise we start in the long pointer 1 character further and try again \*/|
||`        `lptr = lnext;|
||`        `sptr = shortstring;|
||`    `}|
||<p></p><p></p>|
||`    `return 0;|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||int UnityStringArgumentMatches(const char\* str)|
||{|
||`    `int retval;|
||`    `const char\* ptr1;|
||`    `const char\* ptr2;|
||`    `const char\* ptrf;|
||<p></p><p></p>|
||`    `/\* Go through the options and get the substrings for matching one at a time \*/|
||`    `ptr1 = str;|
||`    `while (ptr1[0] != 0)|
||`    `{|
||`        `if ((ptr1[0] == '"') || (ptr1[0] == '\''))|
||`        `{|
||`            `ptr1++;|
||`        `}|
||<p></p><p></p>|
||`        `/\* look for the start of the next partial \*/|
||`        `ptr2 = ptr1;|
||`        `ptrf = 0;|
||`        `do|
||`        `{|
||`            `ptr2++;|
||`            `if ((ptr2[0] == ':') && (ptr2[1] != 0) && (ptr2[0] != '\'') && (ptr2[0] != '"') && (ptr2[0] != ','))|
||`            `{|
||`                `ptrf = &ptr2[1];|
||`            `}|
||`        `} while ((ptr2[0] != 0) && (ptr2[0] != '\'') && (ptr2[0] != '"') && (ptr2[0] != ','));|
||<p></p><p></p>|
||`        `while ((ptr2[0] != 0) && ((ptr2[0] == ':') || (ptr2[0] == '\'') || (ptr2[0] == '"') || (ptr2[0] == ',')))|
||`        `{|
||`            `ptr2++;|
||`        `}|
||<p></p><p></p>|
||`        `/\* done if complete filename match \*/|
||`        `retval = IsStringInBiggerString(Unity.TestFile, ptr1);|
||`        `if (retval == 1)|
||`        `{|
||`            `return retval;|
||`        `}|
||<p></p><p></p>|
||`        `/\* done if testname match after filename partial match \*/|
||`        `if ((retval == 2) && (ptrf != 0))|
||`        `{|
||`            `if (IsStringInBiggerString(Unity.CurrentTestName, ptrf))|
||`            `{|
||`                `return 1;|
||`            `}|
||`        `}|
||<p></p><p></p>|
||`        `/\* done if complete testname match \*/|
||`        `if (IsStringInBiggerString(Unity.CurrentTestName, ptr1) == 1)|
||`        `{|
||`            `return 1;|
||`        `}|
||<p></p><p></p>|
||`        `ptr1 = ptr2;|
||`    `}|
||<p></p><p></p>|
||`    `/\* we couldn't find a match for any substrings \*/|
||`    `return 0;|
||}|
||<p></p><p></p>|
||/\*-----------------------------------------------\*/|
||int UnityTestMatches(void)|
||{|
||`    `/\* Check if this test name matches the included test pattern \*/|
||`    `int retval;|
||`    `if (UnityOptionIncludeNamed)|
||`    `{|
||`        `retval = UnityStringArgumentMatches(UnityOptionIncludeNamed);|
||`    `}|
||`    `else|
||`    `{|
||`        `retval = 1;|
||`    `}|
||<p></p><p></p>|
||`    `/\* Check if this test name matches the excluded test pattern \*/|
||`    `if (UnityOptionExcludeNamed)|
||`    `{|
||`        `if (UnityStringArgumentMatches(UnityOptionExcludeNamed))|
||`        `{|
||`            `retval = 0;|
||`        `}|
||`    `}|
||<p></p><p></p>|
||`    `return retval;|
||}|
||<p></p><p></p>|
||#endif /\* UNITY\_USE\_COMMAND\_LINE\_ARGS \*/|
||/\*-----------------------------------------------\*/|

