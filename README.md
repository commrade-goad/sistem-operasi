# Tugas 1 Sistem Operasi

## 1. How do the Task No. x behaves by setting and modifying the Priority if we have : 2024-04-04
the same delay and the same duration
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

- From that example we can say that if the `xTaskCreate` priority args set to be bigger then it will have the most priority and get to run first.
- priority is `uint8_t` with the alias of `UBaseType_t`;
- so any unsigned integer 8 bits are possible to be an args for the `xTaskCreate` function.
