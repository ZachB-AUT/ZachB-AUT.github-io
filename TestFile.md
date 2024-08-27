# XV6:
XV6 runs on Sv39 RISC-V
Only uses the least significant 36 bits of the memory address

Page table is 4kb, and has $2^{27}$ entries (PTE's, Page Table Entries)

First 12 bits are offset (Location within page), rest (27 bits) is page index.
`satp` is the pointer to the start of the page table.
Sv39 has a three level page table. With 9 bits per level of page table.

## Flags:
Each page has a set of flags indicating certain properties of the page (such as
executable, readable writable, etc).

# Memory allocation and Swap Space.
To allocate more memory for a program, the `sbrk()` function is called.
This takes an argument of number of bytes to allocate.

In the case where allocated virtual memory is greater than the amount of
physical memory of the computer, the computer can move some of the data into the
slower long-term storage. This is called "Swap Space". The page table entry for
this section of memory will have an "invalid" flag set, which indicates it is no
longer in memory

Then, when trying to access a page when it is not loaded into physical memory, a
"Page Fault" is triggered, which causes the computer to load that page into
physical memory again.

The operating system keeps track of the number of free pages in memory.
When the number of free pages is less than the low water mark (LW), the "Swap
Daemon" begins loading unused pages into swap space until the high water mark
(HW) is reached.

There are some number of ways of determining *which* pages to move into swap space, such as least frequently used or FIFO.

These algorithms are measured against the Optimal algorithm, which is a hypothetical situation where the OS knows ahead of time which pages will be required at each time. 

> [!note]- Links on OPT
> Read these!
> [geeksforgeeks](https://www.geeksforgeeks.org/optimal-page-replacement-algorithm/)
> [Wikipedia](https://en.wikipedia.org/wiki/Page_replacement_algorithm#The_theoretically_optimal_page_replacement_algorithm)

But TLDR, This algorithm always swaps out pages that are required furthest away in time.

This is impossible (Or at least impractical) to implement this algorithm in real life, as it requires you to have perfect knowledge of which pages will be accessed at which time.

Other algorithms such as FIFO are much easier to implement, but less efficient.

The most common algorithm us called Least Recently Used (LRU).
In this system, the OS keeps track of which page has been used the least in the past period (variable length) and swaps out the page that is least accessed.

# On-Demand Paging:

## Copy on Write:
When a process calls `fork` it makes a complete copy of the parent processes
memory. It then deletes this memory when calling `exec`. Instead, it is possible
for the memory to be only a pointer to the original file, only making new memory when it attempts to write. this is called "Copy on Write (COW)"
This is a method of On Demand Paging.

This means that the unused portion of the heap/stack doesn't exist!

### Pros and cons:
- Pros:
	- More efficient use of physical memory
	- Faster program startup time
- Cons:
	- File must be open (and available) whole time program is executed.

# Interprocess communication:

Often there are several processes working together to achieve some function.
To do this, they often need to share information.

This can be done with "Shared memory", a section of memory all processes are allowed to read and write.

This can also be done by a broker or message queue.

This is covered more in COMP713.

This can also be done with...
## Pipes:

Pipes are a simple interprocess communication tool used in unix.

Simply, they take the output of one command and feed it into another. 
For example:
```bash
ls -l | grep "*.txt"
```
This command takes the output of `ls -l` and feeds it into `grep`, matching any line that represents a .txt file.

This can also be used with the `pipe(int fd[])` system call, where `fd[]` is a two
element array, where each element is an integer representing one of the standard
lines such as stdin, stdout, or stderr. `fd[0]` is the read end, and `fd[1]` is the write end.

Note, creating a pipe must be done before calling `fork()`

### Example:

```c
#include <sys/types.h>  
#include <stdio.h>  
#include <string.h>  
#include <unistd.h>  
int main(void) {  
	char msg[12]=”Greetings”;  
	char rd_msg[12];  
	int fd[2];  
	pid_t pid;  
	if (pipe(fd) == -1) 
	{  
		fprintf(stderr,”Pipe failed”);  
		return 1;  
	}  
	if (pid = fork() < 0) 
	{  
		fprintf(stderr,”Fork failed”);  
		return 1;  
	}
	if (pid > 0) 
	{  
		close(fd[0]);  
		write(fd[1],msg,strlen(msg)+1);  
		close(fd[1]);
	}  
	else 
	{  
		close(fd[1]);  
		read(fd[0],rd_msg,12);  
		printf(“read %s\n”,rd_msg);  
		close(fd[0];  
	}
}
```

It is best practice to always close a pipe after opening it.
It is possible for communication to be bidirectional, although in the command line pipes are one way.
## Signals:
We didn't cover this, more on this next week.

# Misc
Learn more about system calls!
When a process forks, the child process inherits all the parents open files and file descriptor.
In linux, everything is a file. Pipes are a special kind of file with two ends.

