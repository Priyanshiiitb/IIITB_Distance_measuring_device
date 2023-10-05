# IIITB_Distance_measuring_device
This github repository summarizes the progress made in the ASIC class regarding the RISC-V project.
## Aim 
The aim of a project that measures distance using an ultrasonic sensor is to measure the distance of an object in inaccessible areas. Ultrasonic sensors are non-intrusive and can detect clear or shiny targets that are obscured to some vision-based sensors. This design is made utilising a specialized RISC-V processor.

## Ultrasonic Sensor:
An ultrasonic Sensor is a device used to measure the distance between the sensor and an object without physical contact. This device works based on time-to-distance conversion.

![Screenshot from 2023-10-05 15-41-16](https://github.com/Priyanshiiitb/IIITB_Distance_measuring_device/assets/140998626/c2aaa31a-121c-4f72-a2b2-2d10d02f02c8)


## Working Principle of Ultrasonic Sensor:
Ultrasonic sensors measure distance by sending and receiving the ultrasonic wave. The ultrasonic sensor has a sender to emit the ultrasonic waves and a receiver to receive the ultrasonic waves. The transmitted ultrasonic wave travels through the air and is reflected by hitting the Object. Arduino calculates the time taken by the ultrasonic pulse wave to reach the receiver from the sender. 

We know that the speed of sound in air is nearly 344 m/s,

So, the known parameters are time and speed (constant). Using these parameters, we can calculate the distance traveled by the sound wave.

Formula: Distance = Speed * Time

In the code, the “duration” variable stores the time taken by the sound wave traveling from the emitter to the receiver. That is double the time to reach the object, whereas the sensor returns the total time including sender to object and object to receiver. Then, the time taken to reach the object is half of the time taken to reach the receiver. 


## Circuit Diagram

![Screenshot from 2023-10-05 15-40-41](https://github.com/Priyanshiiitb/IIITB_Distance_measuring_device/assets/140998626/17ee778d-7e91-48fd-bb0b-5dc389bdd2e6)

## C code

## C Code
```
#include <stdio.h>
#include <stdint.h>

// Define the GPIO pin numbers for trig and echo.
#define trigPin 3
#define echoPin 2

long duration; // Variable to store time taken for the pulse to reach the receiver
int distance;  // Variable to store the calculated distance

void pinMode(int pin, int mode) {
    // Simulate pinMode (not used in this code)
}

void digitalWrite(int pin, int value) {
    // Simulate digitalWrite using inline assembly
    int gpio_reg;
    if (value == 1) {
        gpio_reg = 1 << pin;
        asm("or x30, x30, %0" : "=r"(gpio_reg));
    } else {
        gpio_reg = ~(1 << pin);
        asm("and x30, x30, %0" : "=r"(gpio_reg));
    }
}

void delay(int milliseconds) {
    // Simulate delay using a simple loop (adjust based on your hardware)
    for (volatile int i = 0; i < milliseconds * 1000; i++) {
        // Adjust this loop based on your microcontroller's speed
    }
}

unsigned long micros() {
    // Simulate micros using a simple counter (adjust based on your hardware)
    static unsigned long counter = 0;
    return counter++;
}

// Simulate reading the state of the echo pin (replace with actual code)
int readEchoPin() {
    // Replace this with your code to read the echo pin state (HIGH or LOW)
    // Example: return digitalRead(echoPin);
    return 0; // Simulated state
}

int main() {
    // Setup code
    pinMode(trigPin, 1); // Set trigPin as OUTPUT
    pinMode(echoPin, 0); // Set echoPin as INPUT

    printf("Distance measurement using C.\n");
    delay(500); // Delay for 500ms

    while (1) {
        digitalWrite(trigPin, 0);
        delay(2); // Wait for 2ms

        digitalWrite(trigPin, 1);
        delay(10); // Keep trigPin "ON" for 10ms to generate a pulse
        digitalWrite(trigPin, 0);

        // Measure the duration of the pulse using the echo pin
        while (readEchoPin() == 0)
            ; // Wait for the pulse to start
        unsigned long startTime = micros();

        while (readEchoPin() == 1)
            ; // Wait for the pulse to end
        unsigned long endTime = micros();

        duration = endTime - startTime;

        // Calculate distance using the formula (speed of sound = 0.0344 cm/microsecond)
        distance = duration * 0.0344 / 2;

        printf("Distance: %d cm\n", distance);
        delay(100); // Delay for 100ms before the next measurement
    }

    return 0;
}
```

## Assembly Code

### Assembly Code Generation

**To use the RISC-V gcc compiler or simulator**

    `riscv64-unknown-elf-gcc <compiler option -O1 ; Ofast> -mabi=lp64  -march=rv64i  -o <object filename.o> <Cfilename.c>`

   Here -01 gives 15 instructions set while -0fast gives us 12 instructions set
    
**To list the details of a file**
  
  ```
ls -ltr <filename.o>

```

**To disassemble the object file**

```

riscv64-unknown-elf-objdump  -d  <object filename.o> (or)
riscv64-unknown-elf-objdump  -d <object filename.o> | less

```
