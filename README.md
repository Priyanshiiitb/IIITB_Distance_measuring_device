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

![Screenshot from 2023-10-10 21-17-19](https://github.com/Priyanshiiitb/IIITB_Distance_measuring_device/assets/140998626/aaa1e025-f71d-4412-9e55-24eeb39c2bd1)



## C Code
```


int division(int dividend, int divisor);

int main() {
    int trig; // bit 0
    int echo; // bit 1
    int clk_freq = 10000000;
    int distance;
    int dummy = 0xFFFFFFFE;
    unsigned int i;
    unsigned int duration;

    while (1) {
        int trig_local = 1; // Declare trig as a local variable

        // Apply a bitwise mask to manipulate bit 0 of x30
        asm volatile(
            "and x30, x30, %1\n\t"
            "or x30, x30, %0\n\t"
            :
            : "r"(trig_local), "r"(dummy)
            : "x30"
        );

        for (i = 0; i < 10000000; i++);

        int trig_local2 = 0; // Declare trig as a local variable

        // Apply a bitwise mask to manipulate bit 0 of x30
        asm volatile(
            "and x30, x30, %1\n\t"
            "or x30, x30, %0\n\t"
            :
            : "r"(trig_local2), "r"(dummy)
            : "x30"
        );
        i = 0;

        int echo_local;
        asm volatile(
            "addi x10, x30, 0\n\t"

            // Apply a bitwise mask to extract bit 1 of x10
            "and %0, x10, 2\n\t"
            : "=r"(echo_local)
            :
            : "x10"
        );

        while (echo_local != 1) {
            i++;
            asm volatile(
                "addi x10, x30, 0\n\t"
                "and %0, x10, 2\n\t"
                : "=r"(echo_local)
                :
                : "x10"
            );
        }

        duration = division(i, clk_freq);
        distance = duration * 172;
    }

    return 0;
}

int division(int dividend, int divisor) {
    int quotient = 0;

    while (dividend >= divisor) {
        dividend -= divisor;
        quotient++;
    }

    return quotient;
}

```


### Assembly Code Generation

**To use the RISC-V gcc compiler or simulator**
```
riscv64-unkown-elf-gcc -march=rv32i -mabi=ilp32 -ffreestanding  -nostdlib -o ./output.o DISTANCE.c
riscv64-unknown-elf-objdump -d  -r output.o > assembly.txt

```

## Assembly Code


```



output.o:     file format elf32-littleriscv


Disassembly of section .text:

00010054 <main>:
   10054:	fd010113          	addi	sp,sp,-48
   10058:	02112623          	sw	ra,44(sp)
   1005c:	02812423          	sw	s0,40(sp)
   10060:	03010413          	addi	s0,sp,48
   10064:	009897b7          	lui	a5,0x989
   10068:	68078793          	addi	a5,a5,1664 # 989680 <__global_pointer$+0x977ce4>
   1006c:	fef42223          	sw	a5,-28(s0)
   10070:	ffe00793          	li	a5,-2
   10074:	fef42023          	sw	a5,-32(s0)
   10078:	00100793          	li	a5,1
   1007c:	fcf42e23          	sw	a5,-36(s0)
   10080:	fdc42783          	lw	a5,-36(s0)
   10084:	fe042703          	lw	a4,-32(s0)
   10088:	00ef7f33          	and	t5,t5,a4
   1008c:	00ff6f33          	or	t5,t5,a5
   10090:	fe042623          	sw	zero,-20(s0)
   10094:	0100006f          	j	100a4 <main+0x50>
   10098:	fec42783          	lw	a5,-20(s0)
   1009c:	00178793          	addi	a5,a5,1
   100a0:	fef42623          	sw	a5,-20(s0)
   100a4:	fec42703          	lw	a4,-20(s0)
   100a8:	009897b7          	lui	a5,0x989
   100ac:	67f78793          	addi	a5,a5,1663 # 98967f <__global_pointer$+0x977ce3>
   100b0:	fee7f4e3          	bgeu	a5,a4,10098 <main+0x44>
   100b4:	fc042c23          	sw	zero,-40(s0)
   100b8:	fd842783          	lw	a5,-40(s0)
   100bc:	fe042703          	lw	a4,-32(s0)
   100c0:	00ef7f33          	and	t5,t5,a4
   100c4:	00ff6f33          	or	t5,t5,a5
   100c8:	fe042623          	sw	zero,-20(s0)
   100cc:	000f0513          	mv	a0,t5
   100d0:	00257793          	andi	a5,a0,2
   100d4:	fef42423          	sw	a5,-24(s0)
   100d8:	01c0006f          	j	100f4 <main+0xa0>
   100dc:	fec42783          	lw	a5,-20(s0)
   100e0:	00178793          	addi	a5,a5,1
   100e4:	fef42623          	sw	a5,-20(s0)
   100e8:	000f0513          	mv	a0,t5
   100ec:	00257793          	andi	a5,a0,2
   100f0:	fef42423          	sw	a5,-24(s0)
   100f4:	fe842703          	lw	a4,-24(s0)
   100f8:	00100793          	li	a5,1
   100fc:	fef710e3          	bne	a4,a5,100dc <main+0x88>
   10100:	fec42783          	lw	a5,-20(s0)
   10104:	fe442583          	lw	a1,-28(s0)
   10108:	00078513          	mv	a0,a5
   1010c:	038000ef          	jal	ra,10144 <division>
   10110:	00050793          	mv	a5,a0
   10114:	fcf42a23          	sw	a5,-44(s0)
   10118:	fd442703          	lw	a4,-44(s0)
   1011c:	00070793          	mv	a5,a4
   10120:	00179793          	slli	a5,a5,0x1
   10124:	00e787b3          	add	a5,a5,a4
   10128:	00279793          	slli	a5,a5,0x2
   1012c:	40e787b3          	sub	a5,a5,a4
   10130:	00279793          	slli	a5,a5,0x2
   10134:	40e787b3          	sub	a5,a5,a4
   10138:	00279793          	slli	a5,a5,0x2
   1013c:	fcf42823          	sw	a5,-48(s0)
   10140:	f39ff06f          	j	10078 <main+0x24>

00010144 <division>:
   10144:	fd010113          	addi	sp,sp,-48
   10148:	02812623          	sw	s0,44(sp)
   1014c:	03010413          	addi	s0,sp,48
   10150:	fca42e23          	sw	a0,-36(s0)
   10154:	fcb42c23          	sw	a1,-40(s0)
   10158:	fe042623          	sw	zero,-20(s0)
   1015c:	0200006f          	j	1017c <division+0x38>
   10160:	fdc42703          	lw	a4,-36(s0)
   10164:	fd842783          	lw	a5,-40(s0)
   10168:	40f707b3          	sub	a5,a4,a5
   1016c:	fcf42e23          	sw	a5,-36(s0)
   10170:	fec42783          	lw	a5,-20(s0)
   10174:	00178793          	addi	a5,a5,1
   10178:	fef42623          	sw	a5,-20(s0)
   1017c:	fdc42703          	lw	a4,-36(s0)
   10180:	fd842783          	lw	a5,-40(s0)
   10184:	fcf75ee3          	bge	a4,a5,10160 <division+0x1c>
   10188:	fec42783          	lw	a5,-20(s0)
   1018c:	00078513          	mv	a0,a5
   10190:	02c12403          	lw	s0,44(sp)
   10194:	03010113          	addi	sp,sp,48
   10198:	00008067          	ret

```
## Number of unique instruction
```
Number of different instructions: 18
List of unique instructions:
add
ret
jal
mv
sub
lui
lw
bgeu
bge
j
and
bne
andi
or
addi
slli
sw
li

```
## Spike Code-
```
#include <stdio.h>

int division(int dividend, int divisor);

int main() {
    int clk_freq = 10000000;
    int distance = 0; // Initialize distance to 0
    int echo = 0;     // Initialize echo to 0
    int trig = 1;     // Simulate trig as 1
    unsigned int i;
    unsigned int duration;
    int maxIterations = 10;  // Set the maximum number of iterations

    for (int iteration = 0; iteration < maxIterations; iteration++) {
        // Simulate a change in echo_local using inline assembly
        asm volatile(
            "li %0, 1\n\t"  // Simulate echo_local becoming 1
            : "=r"(echo)
            :
        );

        // Simulate changing the value of trig using inline assembly
        asm volatile(
            "li %0, 0\n\t"  // Simulate trig becoming 0
            : "=r"(trig)
            :
        );

        i = 0;

        // Simulate the process of waiting for echo_local to become 1
        while (echo != 1) {
            i++;

            // Simulate a change in echo_local using inline assembly
            asm volatile(
                "li %0, 1\n\t"  // Simulate echo_local becoming 1
                : "=r"(echo)
                :
            );
        }

        duration = division(i, clk_freq);
        distance += duration * 172; // Simulate a change in distance

        // Print the values of echo, trig, and distance
        printf("Echo: %d, Trig: %d, Distance: %d\n", echo, trig, distance);
    }

    return 0;
}

int division(int dividend, int divisor) {
    int quotient = 0;

    while (dividend >= divisor) {
        dividend -= divisor;
        quotient++;
    }

    return quotient;
}
```
### output-

![Screenshot from 2023-10-26 11-49-53](https://github.com/Priyanshiiitb/IIITB_Distance_measuring_device/assets/140998626/8d4c83ab-5740-4f3c-aa30-c3b59f8abd0e)


![Screenshot from 2023-10-30 10-33-46](https://github.com/Priyanshiiitb/IIITB_Distance_measuring_device/assets/140998626/445c0bfa-faa7-4eb8-a65f-109f5bb35924)

### Synthesis-
Synthesis is the process of converting RTL code into a gate-level netlist. It involves mapping the functionality specified in the RTL code to a library of standard cells, such as NAND, NOR, XOR gates, etc., provided by the target technology. Synthesis takes place in multiple steps :

-Converting RTL into simple logic gates.
-Mapping those gates to actual technology-dependent logic gates available in the technology libraries.
-Optimizing the mapped netlist keeping the constraints set by the designer intact.

### Gate Level Simulation-
Gate level Simulation(GLS) is done at the late level of Design cycle. This is run after the RTL code is synthesized into Netlist. Netlist is translation from RTL into Gates and connection wirings with full functional and timing behaviour. Netlist is logically same as RTL code, therefore, same test bench can be used for it.We perform this to verify logical correctness of the design after synthesizing it. Also ensuring the timing of the design is met. Folllowing are the commands to we need to convert Rtl code to netlist in yosys for that commands are :

```
read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80_256.lib 
read_verilog processor.v 
synth -top wrapper
dfflibmap -liberty sky130_fd_sc_hd__tt_025C_1v80_256.lib 
abc -liberty sky130_fd_sc_hd__tt_025C_1v80_256.lib
write_verilog synth_processor_test.v

```
![Screenshot from 2023-11-06 09-21-15](https://github.com/Priyanshiiitb/IIITB_Distance_measuring_device/assets/140998626/a3d0a26e-2572-4fbd-8327-0c19fe4a788b)

### Word of Thanks

I sciencerly thank Mr. Kunal Gosh(Founder/VSD) for helping me out to complete this flow smoothly.

## Acknowledgement
1. Kunal Ghosh, VSD Corp. Pvt. Ltd.
2. Skywater Foundry
3. Mayank Kabra

## Reference
1. https://github.com/SakethGajawada/RISCV-GNU
