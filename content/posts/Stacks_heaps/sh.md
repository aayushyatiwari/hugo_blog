---
   title: "Stacks and Heaps"
   date: 2026-01-16
---

# Data management during compiler process

When compiling a piece of code, the data in the code is stored in *segments*.

## There are five types of segments:
1. *stack*
2. *heap*
3. *data*
4. *code*
5. *BSS*    

We will focus on stack and heap in this essay.  
## Stack
> Stack allocation is the process of allocating memory for local variables and function calls in **call stack**.    
> Each function gets some memory in the stack to store variables in it.    
> Since the memory is handled by the system, its faster. But the memory is less as compared to heaps.    
> The size required is already known before execution. The compiler allocates some memory.  
#### So, what does the stack store?
- for every function, it stores the local variables, return addresses.  
The programmer does not have to worry about allocation or deallocation of the memory, in the case of stack memory.  
*After the function call is done, all the memory is flushed out.*    
- Also, the stack memory is allocated in contigous manner.  

```cpp
int main(){
    // all these go on the stack
    int x = 10;
    int b[10];
    int n = x;
}
```


## Heap
> Heaps are used to *dynamically* allocate memory.  
> Whenever you think about heaps, I'd like you to think about vectors in c++. How do you think they can shrink and expand?  
> Unlike in stack, a programmer has to make sure they delete heap memory after the function is done executing. There is no automatic deletion!    
> Heaps can be a little tricky to use because if you don't deallocate the memory, you might run into issues like **heap-overflow**.     
> Heaps are also *less secure* than stacks because all the threads can get access to the memory of heaps, which can lead to data leaks and can be abused by people.    
> You can use ***malloc*** to dynamically allocate memory in heaps.     
> Heap memory is cleaned using **garbage collectors** in java/python or using **free()** in C/C++.  
> All the new ***objects*** created are in the heap memory.
---
<ins>The name heap has no relation to the *heap data structure*; it simply refers to a large pool of memory available for dynamic allocation.</ins>


```cpp
int main(){
    // this goes on heap 
    int *x = new int[10];
}
```

### Case study example.

```python
class Employee:
    def __init__(self, id, name):
        self.id = id
        self.name = name

def call_emp(id, emp_name):
    return Employee(id, emp_name)

if __name__ == "__main__":
    id, empName = 21, "tiwariji"

    person = call_emp(id, empName)

```
- At run-time, all classes are stored on heap.
- The main method is stored in a stack.
- `Employee` of the call_emp function is called a reference variable and points to the object in heap memory.   
- When `call_emp()` is called from main, a new stack frame is created on top of the previous stack frame.   
And that is how we work our ways with stacks and heaps!     


<center>Interesting note:    

---
In an academic paper, researchers dig into 797 open source C/C++ binaries and tried to figure out the **Qh** (the amount of obejects kept on the heap). They wanted to see if people were still using heaps in their code.  
What they found was people were still using heaps quite extensively!      
Heaps are VERY important because of their ability to resize and also the fact that if you want to create an object that ***outlives*** the function its created in, you'd want a heap.   
Although,   
I am quoting here from the paper something a little hilarious if you ask me.    
*** 
**"There is yet another reason, though, why programmers may allocate on the heap: they are not aware of the costs. In our study, we didn’t analyze whether every allocation on the heap is motivated, but we suspect that a decent amount of objects may be placed on the heap by mistake."**
***

</center>   
Well. It would be nice to understand when you're keeping your objects on the heap right?    

~ Aayushya Tiwari

## References:
[GeeksForGeeks](https://www.geeksforgeeks.org/dsa/stack-vs-heap-memory-allocation/)     
[Medium](https://medium.com/fhinkel/confused-about-stack-and-heap-2cf3e6adb771)     
[Simplilearn](https://www.simplilearn.com/tutorials/data-structure-tutorial/stacks-vs-heap)     
[ResearchPaper](https://arxiv.org/abs/2403.06695)