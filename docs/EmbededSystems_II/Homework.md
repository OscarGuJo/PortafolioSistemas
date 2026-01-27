# ASSIGNMENTS OF THE COURSE

---

## Assignment 1: Tasks 1-5

---

### Objective

The goal of this exercise is to train you to identify logical FreeRTOS tasks from system behavior, even when no RTOS code is shown.

You should focus on:

-Timing requirements
-Blocking behavior
-Safety and criticality
-Independent execution flows

Think in terms of "what must happen independently", not functions or lines of code.

---

### Description

---

The system:

-Reads a temperature sensor every 50 ms
-Sends sensor data via Wi-Fi every 2 seconds
-Monitors an emergency button continuously
-Blinks a status LED at 1 Hz
-Stores error messages when failures occur

Assume:

-The system runs on a microcontroller
-Timing matters
-Some operations may block (Wi-Fi, storage)

### Exercise 1 - Identify Logical Tasks

List the logical tasks that exist in this system.

---

### Exercise 2 - Task Characteristics

For each task you identified, answer the following:

-Is it time-critical? (Yes/No)
-Can it block safely? (Yes/No)
-What happens if this task is delayed?

Write short, technical answers.

---

### Excersice 3 - Priority Reasoning

---

Assign a relative priority to each task:

-High
-Medium
-Low

Then justify each choice in one sentece.



### Exercise 4 - Design Judgment (Trick Question)

Which of the following should NOT necessarily be implemented as a FreeRTOS task?

 -Emergency button monitoring
 -Wi-Fi trnsmission
 -Error logging
 -Status LED blinking

Explain why in 2-3 sentences.