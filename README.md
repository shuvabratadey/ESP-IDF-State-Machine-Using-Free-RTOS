# What is Free RTOS ?

FreeRTOS is an open-source real-time operating system (RTOS) designed for embedded systems. It is known for its small footprint, portability, and scalability, making it a popular choice for a wide range of microcontroller-based applications. Here are some key aspects of FreeRTOS:

* Architecture and Design
* Kernel Features
* Task Management
* Synchronization and Communication
* Memory Management
* Portability
* Community Support
* Licensing
* Real-Time Performance
* Ecosystem
  
Overall, FreeRTOS is a powerful and widely used open-source RTOS that continues to evolve, meeting the needs of developers working on diverse embedded applications.

## This are some basic Example of Free RTOS Codes in ESP-IDF
## 🌐Website Link[🔗Clike here](https://shuvabratadey.github.io/ESP-IDF-State-Machine-Using-Free-RTOS/)
## Include this Libraries First

``` c
#include "driver/gpio.h"
#include "esp_log.h"
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/queue.h"
#include "freertos/timers.h"
#include "esp_system.h"
#include "esp_spi_flash.h"
```

### Task Create

``` c
void my_task(void *pvParam)
{
 while(1)
 {
  printf("Hello Shuva, What's up?\n");
  fflush(stdout);
  vTaskDelay(500 / portTICK_PERIOD_MS);
 }
}

void app_main(void)
{
 xTaskCreate(my_task, "myTask", 1024, NULL, 1, NULL);
}
```

### Task Delete

```c
void my_task(void *pvParam)
{
 while(1)
 {
  printf("Hello Shuva, What's up?\n");
  fflush(stdout);
  vTaskDelay(500 / portTICK_PERIOD_MS);
 }
}

void app_main(void)
{
 TaskHandle_t myTaskHandeler = NULL;
 xTaskCreate(my_task, "myTask", 1024, NULL, 1, &myTaskHandeler);
 vTaskDelay(3000 / portTICK_PERIOD_MS);

 if(myTaskHandeler != NULL)
 {
  vTaskDelete(myTaskHandeler);
 }

 printf("Task has been deleted.");
 while(1)
 {
  printf("\nFrom main function.\n");
  fflush(stdout);
  vTaskDelay(2000 / portTICK_PERIOD_MS);
 }
}

```

### Task Suspension & Resume

``` c
TaskHandle_t TaskAHandle = NULL;

void TaskA(void *pvParameters)
{
    while (1)
    {
        printf("Task A is running!\n");
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

void TaskB(void *pvParameters)
{
    while (1)
    {
        printf("Task B: Suspending Task A\n");
        vTaskSuspend(TaskAHandle);

        vTaskDelay(pdMS_TO_TICKS(5000));  // Wait 5 seconds

        printf("Task B: Resuming Task A\n");
        vTaskResume(TaskAHandle);

        vTaskDelay(pdMS_TO_TICKS(5000));  // Repeat every 10 seconds
    }
}

void app_main()
{
    xTaskCreate(TaskA, "Task A", 2048, NULL, 1, &TaskAHandle);
    xTaskCreate(TaskB, "Task B", 2048, NULL, 2, NULL);  // Higher priority
}

```

## RTOS Semaphore

```c
SemaphoreHandle_t xSemaphore = NULL;

TaskHandle_t myTaskHandle = NULL;
TaskHandle_t myTaskHandle2 = NULL;

void Demo_Task(void *arg)
{
    while (1)
    {
        xSemaphoreTake(xSemaphore, portMAX_DELAY);
        printf("Message Sent! [%ld Sec] \n", pdTICKS_TO_MS(xTaskGetTickCount())/1000);
        vTaskDelay(pdMS_TO_TICKS(3000));
        xSemaphoreGive(xSemaphore);
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

void Demo_Task2(void *arg)
{
    while (1)
    {
        xSemaphoreTake(xSemaphore, portMAX_DELAY);
        printf("\033[0;34mReceived Message [%ld Sec] \n\033[0;37m", pdTICKS_TO_MS(xTaskGetTickCount())/1000);
        vTaskDelay(pdMS_TO_TICKS(2000));
        xSemaphoreGive(xSemaphore);
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

void app_main(void)
{
    xSemaphore = xSemaphoreCreateBinary();
    xSemaphoreGive(xSemaphore);
    xTaskCreate(Demo_Task, "Demo_Task", 4096, NULL, 10, &myTaskHandle);
    vTaskDelay(pdMS_TO_TICKS(500));
    xTaskCreatePinnedToCore(Demo_Task2, "Demo_Task2", 4096, NULL, 10, &myTaskHandle2, 1);
}
```

### Task Input Parameter

```c
void vPrintFunction(void *parameter)
{
 while(1)
 {
  printf("From myTask, Parameter = %d\n", *((int *)parameter));
  vTaskDelay(1000/portTICK_PERIOD_MS);
 }
}

void app_main()
{
    int num = 5;
 xTaskCreate(vPrintFunction, "myTask", 2048, (void *) &num, 1, NULL);
}
```

### Task Input Parameter as array

```c
void vPrintFunction(void *parameter)
{
 while(1)
 {
  printf("From myTask, 1st Parameter = %d\n", *((int *)parameter));
  printf("From myTask, 2nd Parameter = %d\n", *((int *)parameter + 1));
  printf("From myTask, 3rd Parameter = %d\n", *((int *)parameter + 2));
  printf("From myTask, 4th Parameter = %d\n", *((int *)parameter + 3));
  vTaskDelay(1000/portTICK_PERIOD_MS);
 }
}

void app_main()
{
    int num[] = {5, 6, 7, 8};
 xTaskCreate(vPrintFunction, "myTask", 2048, (void *) num, 1, NULL);
}
```

### Task Input Parameter as struct

```c
typedef struct struc{
int Member1;
 char Member2;
}xStruct;

void vPrintFunction(void *parameter)
{
 while(1)
 {
  printf("From myTask, 1st Parameter = %d\n", ((xStruct *)parameter)->Member1);
  printf("From myTask, 2nd Parameter = %c\n", ((xStruct *)parameter)->Member2);
  vTaskDelay(1000/portTICK_PERIOD_MS);
 }
}

void app_main()
{
 xStruct *obj = NULL;
 obj = (xStruct *)malloc(sizeof(xStruct));
 obj->Member1 = 5;
 obj->Member2 = 'S';
 xTaskCreate(vPrintFunction, "myTask", 2048, (void *) obj, 1, NULL);
}
```

### RTOS Queue (Queue Producer/Consumer Example)

``` c
#define QUEUE_SIZE 5

QueueHandle_t myQueue;

void ProducerTask(void *arg)
{
    int count = 0;
    while (1)
    {
        if (xQueueSend(myQueue, &count, pdMS_TO_TICKS(100)) == pdTRUE)
        {
            printf("Produced: %d\n", count);
            count++;
        }
        else
        {
            printf("Queue Full!\n");
        }
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

void ConsumerTask(void *arg)
{
    int rxValue;
    while (1)
    {
        if (xQueueReceive(myQueue, &rxValue, portMAX_DELAY))
        {
            printf("Consumed: %d\n", rxValue);
        }
    }
}

void app_main(void)
{
    myQueue = xQueueCreate(QUEUE_SIZE, sizeof(int));

    if (myQueue != NULL)
    {
        xTaskCreate(ProducerTask, "ProducerTask", 2048, NULL, 1, NULL);
        xTaskCreate(ConsumerTask, "ConsumerTask", 2048, NULL, 1, NULL);
    }
    else
    {
        printf("Failed to create queue.\n");
    }
}

```

### RTOS Software Timer

``` c
TimerHandle_t myTimer;

void TimerCallback(TimerHandle_t xTimer)
{
    static int counter = 0;
    printf("Timer Callback executed! Counter = %d\n", counter++);
}

void app_main(void)
{
    myTimer = xTimerCreate("MyTimer",                // Timer name
                           pdMS_TO_TICKS(2000),      // Timer period: 2 seconds
                           pdTRUE,                   // Auto-reload (periodic)
                           (void *)0,                // Timer ID
                           TimerCallback);           // Callback function

    if (myTimer != NULL)
    {
        xTimerStart(myTimer, 0);  // Start the timer immediately
        printf("Timer started!\n");
    }
    else
    {
        printf("Failed to create timer.\n");
    }
}
```

# State Machine

In FreeRTOS, as with any real-time operating system (RTOS), a state machine is a design pattern often used to model the behavior of a system. A state machine consists of a set of states, events, and transitions. In the context of FreeRTOS, these elements are typically implemented using tasks, queues, and other synchronization mechanisms provided by the RTOS.

Here's a general approach to implementing a state machine in FreeRTOS:

### Event Base State Machine

```c

```

This are some basic example of state machines and the actual implementation may vary depending on the complexity of your system and specific requirements. The key idea is to leverage FreeRTOS tasks and synchronization mechanisms to create a well-organized and efficient state machine for your embedded application.
