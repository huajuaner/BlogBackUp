---
layout: blog
title: FreeRTOS
date: 2022-01-09 15:42:36
tags:

---

## preliminary

> 之前已经开始过一版了，但限于种种原因，没有坚持学完，这次希望以项目驱动，demo驱动，减少引用来自网页tutorial 的图片，常需要使用的函数另做整理，只看只记录必要的信息。
>
> 目前使用RTOS的主要动机是因为原本的代码结构太过丑陋，不便于调试及整理合并。
>
> 对架构的问题：
>
> - 多个Task之间是内存保护的么？
>   - 内存保护是RTOS建立在MPU（Memory Protection Unit）基础之上的，为可选项，由IC提供硬件，RTOS提供软件Support
>   - 特别的，我们所选用的STM32G431RBT6是Cortex-M4 附有MPU的
> - processor存在内核态，用来进行系统中断。
> - Task 的上下文包括栈（还有什么？
>
> 总结：
>
> - 应该时刻注意RTOS是<u>实时</u>的<u>嵌入式</u>系统，其目的与一般的操作系统不同。

## Demos

### CORTEX_STM32F103_Keil

共含有三个task：

- Fast Interrupt Test：使用定时器产生高频中断（20000Hz），用另一个定时器作为标准时间，统计中断定时器的抖动问题。
- LCD task：作为写LCD屏幕的抽象层。
- Check task：5s一次，更改LCD的显示内容。

引出的问题：

- [ ] 定时器的使用以及为什么另一个定时器可视为标准时间。
- [ ] 5s一次，这个时间是基于哪个时钟。
- [x] task 之间使用queue 传输信息。
- [x] 代码规范，部分函数前命名为v，部分函数前命名为x

## 正文

### 程序主体 Task 

> Co-routine 主要是为了降低对RAM的要求，随着目前设备的发展，已经很少会用，如果进一步有需求，再说。

一个RTOS系统可以看作由多个TASK构成，其中有一个负责分发时间片以及切换上下文的TASK，称为Scheduler。

- [ ] 进一步研究task implementation

Task state共有四种，分别为running, ready, blocked 以及suspended。其中挂起vTaskSuspend()可以挂起任意Task（当前Task或其他任意状态的Task）。当configUSE_TIME_SLICING未定义或者被定义为1时，才会同一优先级的多个Task时分复用。而Task scheduling 共面对三种情况：

- Scheduling algorithm for single-core: 
  - "Fixed priority"
  - "Preemptive" 抢占式，高优先级可以随时中断并抢占处理器时间。可以通过配置<u>configUSE_PREEMPTION</u>关闭。
  - "Round-robin" 
  - "Time sliced" 可以通过配置<u>configUSE_TIME_SLICING</u>关闭。
  - Tips：需要合理分配优先级，且高优先级的Task不能陷入loop，避免低优先级Task陷入starvation.
- Scheduling algorithm for asymmetric multicore(AMP): Each processor core runs its own instance of FreeRTOS.
  - 通过使用 <u>stream or message buffer</u> as the inter-core communication primitive.
- Scheduling algorithm for symmetric multicore(SMP): There is only one instance of FreeRTOS.
  - 同一时间可以运行多个Task。

#### Task Implementation

```c++
  void vATaskFunction( void *pvParameters )
  {
      for( ;; )
      {
          /* Psudeo code showing a task waiting for an event 
            with a block time. If the event occurs, process it.  
            If the timeout expires before the event occurs, then 
            the system may be in an error state, so handle the
            error.  Here the pseudo code "WaitForEvent()" could 
            replaced with xQueueReceive(), ulTaskNotifyTake(), 
            xEventGroupWaitBits(), or any of the other FreeRTOS 
            communication and synchronisation primitives. */
          if( WaitForEvent( EventObject, TimeOut ) == pdPASS )
          {
              -- Handle event here. --
          }
          else
          {
              -- Clear errors, or take actions here. --
          }
      }

      /* If the task needs to be terminated, use vTaskDelete instead of return */
      vTaskDelete( NULL );
  }
//PS: use event-driven instead of looping 
```

### intermediary for inter-task communications: Queue, Semaphore, Mutex

#### Queue

用于在task之间，以及task与interrupt之间传递消息。（Interrupt 与 Task 采用不同抽象）。

- 传值
- The kernel is fully responsible for allocating the memory used.
- The implementation is naturally suited for use in a memory protected environment. A task that is restricted to a protected memory area can pass data to a task that is restricted to a different protected memory area because invoking the RTOS by calling the queue send function will raise the microcontroller privilege level. The queue storage area is only accessed by the RTOS (with full privileges). 为什么要做privilege？同一时间只有一个线程在工作，不会打乱读写（存疑，看是否为元操作）-->如果需要全局变量是否也要做抽象（或者mutex对读写上锁）。
- 在中断内需要调用不同的接口。(中断是routine类)
- 如果进行当前不能进行的操作，当前Task 会进入Block状态。

#### Semaphore Vs. Mutex

<u>TIP: 'Task Notifications' can provide a light weight alternative to binary semaphores in many situations</u>

- [x] Mutexes 更适合实现互斥资源访问，而 Binary Semaphores 更适合实现同步，为什么？

因为Mutex 实现了优先级机制，并且<u>Mutex不可用于中断</u>，而semaphore 并没有。

$Definition$ Priority Inheritance: 当一个更高优先级的task申请mutex时，将目前正占有该mutex的task的优先级暂时升高到相同。以避免出现priority inversion.

$Definition$ Deferred Interrupt: The Peripheral interrupt will immediately send data to a queue and block, the post processing will occur when the processor is free. 简单说，这样可以保证各task按时正常进行。

- [ ] Counting Semaphores 是否可用于传感器计数，每次increment 1，decrement 多个且可变。最简单也可以用多个信号量代替实现。

### Task Notification 

<u>TIP: 如果使用Stream and Message Buffers, the task notification at array index 0 is preserved.</u>

相较于前一章所提到的用于传输信息的实例，Task Notification 更快且更节省空间。

#### 代替Binary Semaphore

``` c++
/* This is an example of a transmit function in a generic
peripheral driver.  An RTOS task calls the transmit function,
then waits in the Blocked state (so not using an CPU time)
until it is notified that the transmission is complete.  The
transmission is performed by a DMA, and the DMA end interrupt
is used to notify the task. */

/* Stores the handle of the task that will be notified when the
transmission is complete. */
static TaskHandle_t xTaskToNotify = NULL;

/* The index within the target task's array of task notifications
to use. */
const UBaseType_t xArrayIndex = 1;

/* The peripheral driver's transmit function. */
void StartTransmission( uint8_t *pcData, size_t xDataLength )
{
    /* At this point xTaskToNotify should be NULL as no transmission
    is in progress.  A mutex can be used to guard access to the
    peripheral if necessary. */
    configASSERT( xTaskToNotify == NULL );

    /* Store the handle of the calling task. */
    xTaskToNotify = xTaskGetCurrentTaskHandle();

    /* Start the transmission - an interrupt is generated when the
    transmission is complete. */
    vStartTransmit( pcData, xDatalength );
}
/*-----------------------------------------------------------*/

/* The transmit end interrupt. */
void vTransmitEndISR( void )
{
BaseType_t xHigherPriorityTaskWoken = pdFALSE;

    /* At this point xTaskToNotify should not be NULL as
    a transmission was in progress. */
    configASSERT( xTaskToNotify != NULL );

    /* Notify the task that the transmission is complete. */
    vTaskNotifyGiveIndexedFromISR( xTaskToNotify,
                                   xArrayIndex,
                                   &xHigherPriorityTaskWoken );
	//不是信号量给过去之后这个就被抢占了？
    //所以中断不会被抢占，可为什么要切换上下文
    
    /* There are no transmissions in progress, so no tasks
    to notify. */
    xTaskToNotify = NULL;

    /* If xHigherPriorityTaskWoken is now set to pdTRUE then a
    context switch should be performed to ensure the interrupt
    returns directly to the highest priority task.  The macro used
    for this purpose is dependent on the port in use and may be
    called portEND_SWITCHING_ISR(). */
    // 为什么要手动切换上下文？？
    portYIELD_FROM_ISR( xHigherPriorityTaskWoken );
}
/*-----------------------------------------------------------*/

/* The task that initiates the transmission, then enters the
Blocked state (so not consuming any CPU time) to wait for it
to complete. */
void vAFunctionCalledFromATask( uint8_t ucDataToTransmit,
                                size_t xDataLength )
{
uint32_t ulNotificationValue;
const TickType_t xMaxBlockTime = pdMS_TO_TICKS( 200 );

    /* Start the transmission by calling the function shown above. */
    StartTransmission( ucDataToTransmit, xDataLength );

    /* Wait to be notified that the transmission is complete.  Note
    the first parameter is pdTRUE, which has the effect of clearing
    the task's notification value back to 0, making the notification
    value act like a binary (rather than a counting) semaphore.  */
    ulNotificationValue = ulTaskNotifyTakeIndexed( xArrayIndex,
                                                   pdTRUE,
                                                   xMaxBlockTime );

    if( ulNotificationValue == 1 )
    {
        /* The transmission ended as expected. */
    }
    else
    {
        /* The call to ulTaskNotifyTake() timed out. */
    }
}

```

### Stream & Message Buffers

正常只支持单个task读，单个task写，如果需要多个，自行上锁，这两个功能并不是默认开启的。

- [ ] **FreeRTOS/Demo/Common/Minimal/StreamBufferInterrupt.c**

### Software Timers

> Efficiency Consideration：
>
> - does not execute timer callback functions from an interrupt context
> - does not consume **any** processing time unless a timer has actually expired
> - does not add any processing overhead to the tick interrupt
> - does not walk any link list structures while interrupts are disabled

这个功能也是可选的，将Timer 抽象为Timer Service Task，功能主要经由API移交给该task。

### memory allocating 

- For Static Memory:  <u>configSUPPORT_STATIC_ALLOCATION</u> needs to be set to 1, and the available function name is like <u>xTaskCreateStatic()</u>
- For Dynamic Memory: <u>configSUPPORT_DYNAMIC_ALLOCATION</u> needs to be set to 1 or left undefined, the available function name is like <u>xTaskCreate()</u>

考虑到各式的嵌入式系统有不同的需求，将heap management schemes的实现方式交由用户决定，同时提供了五种可选方案：

- heap_1 - the very simplest, does not permit memory to be freed.
- heap_2 - permits memory to be freed, but does not coalescence adjacent free blocks.
- heap_3 - simply wraps the standard malloc() and free() for thread safety.
- heap_4 - coalescences adjacent free blocks to avoid fragmentation. Includes absolute address placement option.
- heap_5 - as per heap_4, with the ability to span the heap across multiple non-adjacent memory areas.

同样的，应对栈溢出的解决方案也留待用户自行决定。

## Secondary Docs

### Idle Tasks 

系统自动创建，为全场最低优先级，用于Garbage Collection。为了保证系统时刻有可以运行的Task，Idle Task千不能万不能陷入Block状态，同样，其他任何Task 都不能设置做Loop。

<u>Tips: 使用vTaskDelete()</u> 结束Task，会自动被Idle Task 清理。

### Hook Functions

挂钩函数，可以由某些Task的特定生命进程唤醒，例如：

- Idle Hook Function

大概是若Idle Task 都没有Garbage to collect时，会触发Idle Hook Function，例如可将系统进入低功耗模式

- Tick Hook Function
- Malloc Failed Hook Function
- Daemon Task Startup Hook

### MPU Support 

FreeRTOS 共提供两种方式使得应用更安全：

- 区分Task 分别运行在privileged模式以及unprivileged模式
- 限制Unprivileged Task 对例如RAM, exectuable, peripherals 的访问

综上，FreeRTOS的解决方式更类似于权限管理，而非细粒度的每个Task存在其私有空间。

### Thread Local Storage Pointers (TLS)

- [x] 作用类似于static 函数，尚不明确具体用途，需要测试static 是否可用

可以获取其他Task 的某个index 的TLS。

### Blocking on Multiple RTOS objects 

Enable tasks to be blocked on multiple queues and/or semaphores.

### Deferred Interrupt Handling 

为减少中断所占用的时长，将数据后处理的工作放在一些优先级较低（较ISR而言）的Task中进行。

- [ ] 可以用来处理IMU数据，处理变向及stopper问题。

1. Centralised Deferred Interrupt Handling：

2. Application Controlled Deferred Interrupt Handling

> https://www.freertos.org/RTOS_Task_Notification_As_Counting_Semaphore.html 
>
> 第二个实例是否可以用于监测stopper？

本质区别是，第一类是每次interrupt后由daemon task 新建一个function来处理数据，第二类则是存在一个Task，一直在等待这个信号量。

### Task implementation

Real Time Operating System 和普通操作系统对多线程的需求不同，需要保证某些Task 在某时刻之前完成，引入了优先级的概念，而时分复用只会出现在相同优先级的task之间。
