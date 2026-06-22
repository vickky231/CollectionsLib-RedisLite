# Duration: 2 hours

Goal:
Implement a dynamic array in C++ with core operations including push_back, pop_back, get, set, display, getSize, isEmpty, and clear.

Problem Encountered:

1 .g++.exe: error: main.cpp: No such file or directory
   g++.exe: fatal error: no input files
   compilation terminated.

2. dynamicArray.cpp: In function 'int main()':
   dynamicArray.cpp:153:9: error: 'class DynamicArray<int>' has no member named 'pushback'; did you mean 'pushBack'?
     arr.pushback(50);
         ^~~~~~~~

3. Managing dynamic memory safely while resizing the array. Initial issues included    incorrect   size updates after push_back and confusion between capacity and size during element insertion and deletion.

What I Tried:

Implemented basic structure using raw pointers and dynamic memory allocation
Created separate variables for size and capacity
Added resize() logic to double capacity when full
Debugged push_back() to ensure correct insertion index
Tested edge cases like popping from empty array and accessing invalid indices
Used print/debug statements to track size vs capacity changes

Outcome:
Successfully built a working dynamic array supporting all required operations:

push_back and pop_back work correctly with resizing support
get and set safely access elements with boundary checks
display prints all valid elements
getSize, isEmpty, and clear correctly reflect array state
Gained a clearer understanding of manual memory management and dynamic resizing logic.