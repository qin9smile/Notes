Memory Footprint (Pages)
Each Page has typically 16KB Size
page type is clean or dirty

App Memory Use = number of Pages * page size

```c++
int *array = malloc(20000 * sizeof(int)); // 6 pages
```

memory map file on disk but load on memory.

Read-Only files are clean

Kernel manages when file is in RAM.

An app, their  footprint and profile is  Dirty, Compressed and Clean

In Clean Memory, data that can be paged out of memory
Data can be memory mapped files(image, blob, model,) or frameworks.
(Every framework has __DATA_CONST data section)
now every thing is clean, but if you do any runtime shenanigans like swizzling, that can actually make it dirty.

Dirty Memory, memory written by an app.
All heap allocations (Such as Decoded image buffers，Frameworks，)
(Frameworks have a data section called __DATA, __DATA_DIRTY also count towards dirty memory.)


Framework link twice (memory and dirty memory)

If Framework use singletons or global initializers are a greate way to reduce amount of memory they use because a singleton's always going to be in memory after it has been created, and these initializers are alse run whenever the 
framework is linked or the class is loaded.

Compressed memory 
No traditional disk swap
memory compressor
* compresses unaccessed pages
* decompresses pages upon access


Use NSCache instead of dictionary is a thread safe way to store cached objects.


When we talk about memory footprint usually talk about dirty and compressed memory ,the clean memory is not in count.

Footprint limits
* Limits vary by device
* Apps has a fairly high footprint limit
* Extensions have a much lower limit

Exception upon exceeding limit
* EXC_RESOURCE)EXCEPTION

Profiling Footprint
* Allocations
* Leaks
* VMTracker (separate tracks for dirty and swapped with compressed memory)
* Virtual memory trace