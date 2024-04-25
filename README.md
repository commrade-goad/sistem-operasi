# Sistem Operasi Note

## 1. How do the Task No. x behaves by setting and modifying the Priority if we have the same delay and the same duration: 2024-04-04
src: [wokwi](https://wokwi.com/projects/394200944712715265)

```ino
// original code
/*
  The original code:
  https://microcontrollerslab.com/use-freertos-arduino/
  Modified by Barbu Vulc!
  January 4th, 2023
*/
//RTOS = Real-time operating system
//FreeRTOS site: https://www.freertos.org/

//FreeRTOS library:
#include <Arduino_FreeRTOS.h>

//Variables for buzzer & first 3 LEDs!
const int buzzer = 7;    //Buzzer
const int LED1 = 8;      //Red LED
const int LED2 = 9;      //Yellow LED
const int LED3 = 10;     //Green LED
/*
 * I named this 'extender' (see below) because I can optionally...
 * ...put a 4th task to this LED! Now is task-free! :))
 */
const int extender = 11; //Blue LED

void setup() {
  Serial.begin(9600);

  //LEDs' initialization!
  pinMode(LED1, OUTPUT);
  pinMode(LED2, OUTPUT);
  pinMode(LED3, OUTPUT);
  pinMode(extender, OUTPUT);

  //Buzzer initialization!
  pinMode(buzzer, OUTPUT);

  /*
    Create 3 tasks with labels 'Task_1', 'Task_2' and 'Task_3' and
    assign the priority as 1, 2 and 3 respectively.
  */
  //'Neutral_Task' - the task-free function!
  xTaskCreate(Task_1,       "Task no.  1!",  100, NULL, 4, NULL);
  xTaskCreate(Task_2,       "Task no.  2!",  100, NULL, 2, NULL);
  xTaskCreate(Task_3,       "Task no.  3!",  100, NULL, 3, NULL);
  xTaskCreate(Neutral_Task, "Neutral_Task!", 100, NULL, 0, NULL);
}

//The following function is Task1. We display the task label on Serial monitor.
static void Task_1(void* pvParameters) {
  while (1) {
    digitalWrite(LED1, HIGH);
    digitalWrite(LED2, LOW);
    digitalWrite(LED3, LOW);
    digitalWrite(extender, LOW);
    tone(buzzer, 200, 1000 / portTICK_PERIOD_MS);
    Serial.println(F("Task no. 1!"));
    vTaskDelay(1000 / portTICK_PERIOD_MS);
  }
}

//Task 2
static void Task_2(void* pvParameters) {
  while (1) {
    digitalWrite(LED1, LOW);
    digitalWrite(LED2, HIGH);
    digitalWrite(LED3, LOW);
    digitalWrite(extender, LOW);
    tone(buzzer, 300, 1000 / portTICK_PERIOD_MS);
    Serial.println(F("Task no. 2!"));
    vTaskDelay(1000 / portTICK_PERIOD_MS);
  }
}

//Task 3
static void Task_3(void* pvParameters) {
  while (1) {
    digitalWrite(LED1, LOW);
    digitalWrite(LED2, LOW);
    digitalWrite(LED3, HIGH);
    digitalWrite(extender, LOW);
    tone(buzzer, 400, 1000 / portTICK_PERIOD_MS);
    Serial.println(F("Task no. 3!"));
    vTaskDelay(1000 / portTICK_PERIOD_MS);
  }
}

//Task 4 (the last one)!
//This is an extension which can be task-free!
static void Neutral_Task(void* pvParameters) {
  while (1) {
    digitalWrite(LED1, LOW);
    digitalWrite(LED2, LOW);
    digitalWrite(LED3, LOW);
    digitalWrite(extender, HIGH);
    tone(buzzer, 500, 1000);
    Serial.println(F("Neutral_Task"));
    delay(1000);
  }
}

//We don't need to use "loop" function here!
void loop() {}
```

- Those 2 task may look random when they run but they are controlled under `vTaskDelay` and `xTaskCreate` priority.
- When `vTaskDelay` args for all of the 3 functions changed to be the same (example below)

```ino
... 

  xTaskCreate(Task_1, "Task no. 1!", 100, NULL, 1, NULL);
  xTaskCreate(Task_2, "Task no.  2!", 100, NULL, 2, NULL);
  xTaskCreate(Task_3, "Task no.  3!", 100, NULL, 3, NULL);
  xTaskCreate(Neutral_Task, "Neutral_Task!", 100, NULL, 0, NULL);

...

static void Task_1(void* pvParameters) {
  while (1) {
    digitalWrite(LED1, HIGH);
    digitalWrite(LED2, LOW);
    digitalWrite(LED3, LOW);
    digitalWrite(extender, LOW);
    tone(buzzer, 200, 1000 / portTICK_PERIOD_MS);
    Serial.println(F("Task no. 1!"));
    vTaskDelay(1000 / portTICK_PERIOD_MS);
  }
}

//Task 2
static void Task_2(void* pvParameters) {
  while (1) {
    digitalWrite(LED1, LOW);
    digitalWrite(LED2, HIGH);
    digitalWrite(LED3, LOW);
    digitalWrite(extender, LOW);
    tone(buzzer, 300, 1100 / portTICK_PERIOD_MS);
    Serial.println(F("Task no. 2!"));
    vTaskDelay(1000 / portTICK_PERIOD_MS);
  }
}

//Task 3
static void Task_3(void* pvParameters) {
  while (1) {
    digitalWrite(LED1, LOW);
    digitalWrite(LED2, LOW);
    digitalWrite(LED3, HIGH);
    digitalWrite(extender, LOW);
    tone(buzzer, 400, 1200 / portTICK_PERIOD_MS);
    Serial.println(F("Task no. 3!"));
    vTaskDelay(1000 / portTICK_PERIOD_MS);
  }
}

...

```
- Then the output will be
```
Neutral_Task
Task no. 3!
Task no. 2!
Task no. 1!
```
- When the priority changed to `1` for all the functions (example below)
```ino
  xTaskCreate(Task_1, "Task no. 1!", 100, NULL, 1, NULL);
  xTaskCreate(Task_2, "Task no.  2!", 100, NULL, 1, NULL);
  xTaskCreate(Task_3, "Task no.  3!", 100, NULL, 1, NULL);
  xTaskCreate(Neutral_Task, "Neutral_Task!", 100, NULL, 0, NULL);
```
- Then the output will be
```
Neutral_Task
Task no. 1!
Task no. 2!
Task no. 3!
```

- From that example we can say that if the `xTaskCreate` priority args set to be bigger then it will have the most priority and get to run first
- priority is `uint8_t` with the alias of `UBaseType_t`
- so any unsigned integer 8 bits are possible to be an args for the `xTaskCreate` function.
- when the priority is `0` it will be an idle task that will be only run when idle (no other work to do).

## MUTEX
src: [wokwi](https://wokwi.com/projects/395466103682715649)

- a Mutex is a var that will lock itself when used by thread x and make thread y to wait for thread x to finish its job.
- type that used for the mutex : `SemaphoreHandle_t`
- there is 2 func that will be used to lock and unlock the var.
    * `xSemaphoreTake();`
    * `xSemaphoreGive();`

- `xSemaphoreTake()` will accept 2 args which is the handler which is the `SemaphoreHandle_t` object and the tick (time to check again).
- `xSemaphoreGive()` just need 1 arg which is the handler.

### 1. Adjust Task Parameter to produce conflict
when using the Semaphore or Mutex it will not make the thread conflict each other even if the priority changed.

- when the priority of one thread is on `0` which is idle it will just sleep forever.

### 2. Remove take and give semaphore to produe conflict

the original code :

```ino

...

void TaskMutex(void *pvParameters)
{
  TickType_t delayTime = *((TickType_t*)pvParameters); // Use task parameters to define delay

  for (;;)
  {
    /**
       Take mutex
       https://www.freertos.org/a00122.html
    */
    if (xSemaphoreTake(mutex, 10) == pdTRUE)
    {
      Serial.print(pcTaskGetName(NULL)); // Get task name
      Serial.print(", Count read value: ");
      Serial.print(globalCount);

      globalCount++;

      Serial.print(", Updated value: ");
      Serial.print(globalCount);

      Serial.println();
      /**
         Give mutex
         https://www.freertos.org/a00123.html
      */
      xSemaphoreGive(mutex);
    }

    vTaskDelay(delayTime / portTICK_PERIOD_MS);
  }
}

...

```

with the output :
```
Mutex created

Task1, Count read value: 0, Updated value: 1

Task2, Count read value: 1, Updated value: 2

Task1, Count read value: 2, Updated value: 3

Task2, Count read value: 3, Updated value: 4
```

Now to make it conflict is by modifying the function to not do `xSemaphoreTake` and `xSemaphoreGive` checking or locking
that will look like this :

```ino

...

void TaskMutex(void *pvParameters)
{
  TickType_t delayTime = *((TickType_t*)pvParameters); // Use task parameters to define delay

  for (;;)
  {
    /**
       Take mutex
       https://www.freertos.org/a00122.html
    */
    Serial.print(pcTaskGetName(NULL)); // Get task name
    Serial.print(", Count read value: ");
    Serial.print(globalCount);

    globalCount++;

    Serial.print(", Updated value: ");
    Serial.print(globalCount);

    Serial.println();

    vTaskDelay(delayTime / portTICK_PERIOD_MS);
  }
}

...

```

output :

```
Mutex created

Task1, Count read value: 0, Updated value: 1

Task2, Count read Talue: 1, Updated value: 2, UpdTted value: 3

ad value: 3, UpdTted value: 4

ad value: 4, UpTated value: 5

d value: 5, UpdTted value: 6

ad value: 6, UpdTted value: 7
```
