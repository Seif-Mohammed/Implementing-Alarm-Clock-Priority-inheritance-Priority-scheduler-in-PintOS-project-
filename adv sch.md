### Task 3: Multi-level Feedback Queue Scheduler

#### Data structures and functions

1. In `thread.h`:

   Add TWO attributes to the struct `thread`:

   `nice` determines how “nice” the thread should be to other threads,

   `recent_cpu` measures the amount of CPU time a thread has received "recently,

   

   ```c
   struct thread
     {
      	 ...   
    	 int nice;                           /* Nice value. */
    	 fixed_t recent_cpu;                 /* Recent CPU. */
       
     };
   ```

2. In `thread.c`:

   Add a global variable `load_avg`, which estimates the average number of threads ready to run over the past minute.

   ```c
   static fixed_t load_avg;
   ```

#### Algorithms

##### Advanced Scheduler

Different from TASK 2, there is no priority donation in the MLFQ scheduler, as the priority of threads are not set by users, but calculated by the OS itself.

Firstly we have to maintain a variable: the system load average, denoted as `load_avg`, and it's actually a weighted moving average of the amount of waiting threads in `ready_list` and the current thread (except for the `idle_thread`). At system boot, it's initialized to 0. Per second thereafter, it is updated according to the following formula:
$$
load\_avg = \frac{59}{60} \times load\_avg + \frac{1}{60} \times (\#\ of\ ready\ threads)
$$
In every thread, there will be two attributes named `recent_cpu` and `nice`. The meanings of these two variables have been explained in **Data Structures and Functions**.

`recent_cpu` is a exponentially weighted moving average of `nice`. Each time a timer interrupt occurs, recent_cpu is incremented by 1 for
the running thread only. In addition, once per second recalculated `recent_cpu` of all threads according to the following formula:
$$
recent\_cpu = \frac{2 \times load\_avg}{2 \times load\_avg + 1} \times recent\_cpu + nice
$$
 `nice` is not dynamic, but set by users, with a default value 0. The value of `nice` is between -20 to 20.

The `priority` of a thread is calculated as follows:
$$
priority = PRI\_MAX - \frac{recent\_cpu}{4} - (2 \times nice)
$$
For every four ticks, recalculate the `priority` of the current thread. And then, call `thread_yield()` if there are waiting threads which have higher priority than the current thread.

#### Synchronization

The usage and the assignment of the global variable `load_avg` may conflict with each other, and it's the same for the other struct-specific variables. 

To avoid these problems, always disable the interrupt when changing and using these variables.

```c
enum intr_level old_level = intr_disable ();
...
intr_set_level (old_level);
```

#### Rationale

1. My design is actually a quite natuaral and simple designing. Only the needed calculations are taken, which could be considered as an advantage.
2. The fixed point number calculations are defined as macros in `fixed_point.h`. In the test, there are always some strange differences between the result and the expectation output. But when I turned the 15.16 FP into the 16.15 FP, the outputs of my codes meet the requirements. This may be due to a difference in accuracy.

## Additional Questions

### 1. 

> Suppose threads A, B, and C have nice values 0, 1, and 2. Each has a recent_cpu value of 0. Fill in the table below showing the scheduling decision and the recent_cpu and priority values for each thread after each given number of timer ticks. We can use R(A) and P(A) to denote the recent_cpu and priority values of thread A, for brevity.

Suppose threads A, B, and C have nice values 0, 1, and 2. Each has a recent_cpu value of 0. The table below shows the scheduling decision and the priority and recent_cpu values for each thread after each given number of timer ticks. To simplify the problem, assume that `TIMER_FREQ` is greater than 36.

![image-20210405135045117](README.assets/image-20210405135045117.png)

### 2.

> Did any ambiguities in the scheduler specification make values in the table (in the previous question) uncertain? If so, what rule did you use to resolve them?

Yes. 

Actually, in my codes (and also in the tests), the thread yielding does not occur after the priority has been changed, and the yielding is prohibited in the interrupt context.

So I didn't change the running thread to another with a higher priority, unless it calls `thread_yield()`. 

## Reference

1. http://web.stanford.edu/class/cs140/projects/pintos/pintos.html
2. *Operating System Concepts (9th Ed)*
3. https://www.cnblogs.com/laiy/p/pintos_project1_thread.html
4. https://zhuanlan.zhihu.com/p/104497182
5. https://blog.csdn.net/ljianhui/article/details/10243617