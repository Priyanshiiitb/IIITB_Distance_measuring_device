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
        trig = 1;
        asm volatile(
            "and x30, x30, %1\n\t"
            "or x30, x30, %0\n\t"
            :
            : "r"(trig), "r"(dummy)
            : "x30"
        );
        for (i = 0; i < 10000000; i++);

        trig = 0;
        asm volatile(
            "and x30, x30, %1\n\t"
            "or x30, x30, %0\n\t"
            :
            : "r"(trig), "r"(dummy)
            : "x30"
        );
        i = 0;

        asm volatile(
            "addi x10, x30, 0\n\t"
            "and %0, x10, 2\n\t"
            : "=r"(echo)
            :
            : "x10"
        );

        while (echo != 1) {
            i++;
            asm volatile(
                "addi x10, x30, 0\n\t"
                "and %0, x10, 2\n\t"
                : "=r"(echo)
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

## Assembly Code

### Assembly Code Generation

**To use the RISC-V gcc compiler or simulator**
```
  riscv32-unknown-elf-gcc -march=rv32i -mabi=ilp32 -ffreestanding -nostdlib -o ./out filename.c
  riscv32-unknown-elf-objdump -d -r out > assembly.txt

```




```
distance.o:     file format elf32-littleriscv


Disassembly of section .text:

00010074 <main>:
   10074:	fd010113          	add	sp,sp,-48
   10078:	02112623          	sw	ra,44(sp)
   1007c:	02812423          	sw	s0,40(sp)
   10080:	03010413          	add	s0,sp,48
   10084:	009897b7          	lui	a5,0x989
   10088:	68078793          	add	a5,a5,1664 # 989680 <__global_pointer$+0x977cc4>
   1008c:	fef42223          	sw	a5,-28(s0)
   10090:	ffe00793          	li	a5,-2
   10094:	fef42023          	sw	a5,-32(s0)
   10098:	00100793          	li	a5,1
   1009c:	fcf42e23          	sw	a5,-36(s0)
   100a0:	fdc42783          	lw	a5,-36(s0)
   100a4:	fe042703          	lw	a4,-32(s0)
   100a8:	00ef7f33          	and	t5,t5,a4
   100ac:	00ff6f33          	or	t5,t5,a5
   100b0:	fe042423          	sw	zero,-24(s0)
   100b4:	0100006f          	j	100c4 <main+0x50>
   100b8:	fe842783          	lw	a5,-24(s0)
   100bc:	00178793          	add	a5,a5,1
   100c0:	fef42423          	sw	a5,-24(s0)
   100c4:	fe842703          	lw	a4,-24(s0)
   100c8:	009897b7          	lui	a5,0x989
   100cc:	67f78793          	add	a5,a5,1663 # 98967f <__global_pointer$+0x977cc3>
   100d0:	fee7f4e3          	bgeu	a5,a4,100b8 <main+0x44>
   100d4:	fc042e23          	sw	zero,-36(s0)
   100d8:	fdc42783          	lw	a5,-36(s0)
   100dc:	fe042703          	lw	a4,-32(s0)
   100e0:	00ef7f33          	and	t5,t5,a4
   100e4:	00ff6f33          	or	t5,t5,a5
   100e8:	fe042423          	sw	zero,-24(s0)
   100ec:	000f0513          	mv	a0,t5
   100f0:	00257793          	and	a5,a0,2
   100f4:	fef42623          	sw	a5,-20(s0)
   100f8:	01c0006f          	j	10114 <main+0xa0>
   100fc:	fe842783          	lw	a5,-24(s0)
   10100:	00178793          	add	a5,a5,1
   10104:	fef42423          	sw	a5,-24(s0)
   10108:	000f0513          	mv	a0,t5
   1010c:	00257793          	and	a5,a0,2
   10110:	fef42623          	sw	a5,-20(s0)
   10114:	fec42703          	lw	a4,-20(s0)
   10118:	00100793          	li	a5,1
   1011c:	fef710e3          	bne	a4,a5,100fc <main+0x88>
   10120:	fe842783          	lw	a5,-24(s0)
   10124:	fe442583          	lw	a1,-28(s0)
   10128:	00078513          	mv	a0,a5
   1012c:	038000ef          	jal	10164 <division>
   10130:	00050793          	mv	a5,a0
   10134:	fcf42c23          	sw	a5,-40(s0)
   10138:	fd842703          	lw	a4,-40(s0)
   1013c:	00070793          	mv	a5,a4
   10140:	00179793          	sll	a5,a5,0x1
   10144:	00e787b3          	add	a5,a5,a4
   10148:	00279793          	sll	a5,a5,0x2
   1014c:	40e787b3          	sub	a5,a5,a4
   10150:	00279793          	sll	a5,a5,0x2
   10154:	40e787b3          	sub	a5,a5,a4
   10158:	00279793          	sll	a5,a5,0x2
   1015c:	fcf42a23          	sw	a5,-44(s0)
   10160:	f39ff06f          	j	10098 <main+0x24>

00010164 <division>:
   10164:	fd010113          	add	sp,sp,-48
   10168:	02812623          	sw	s0,44(sp)
   1016c:	03010413          	add	s0,sp,48
   10170:	fca42e23          	sw	a0,-36(s0)
   10174:	fcb42c23          	sw	a1,-40(s0)
   10178:	fe042623          	sw	zero,-20(s0)
   1017c:	0200006f          	j	1019c <division+0x38>
   10180:	fdc42703          	lw	a4,-36(s0)
   10184:	fd842783          	lw	a5,-40(s0)
   10188:	40f707b3          	sub	a5,a4,a5
   1018c:	fcf42e23          	sw	a5,-36(s0)
   10190:	fec42783          	lw	a5,-20(s0)
   10194:	00178793          	add	a5,a5,1
   10198:	fef42623          	sw	a5,-20(s0)
   1019c:	fdc42703          	lw	a4,-36(s0)
   101a0:	fd842783          	lw	a5,-40(s0)
   101a4:	fcf75ee3          	bge	a4,a5,10180 <division+0x1c>
   101a8:	fec42783          	lw	a5,-20(s0)
   101ac:	00078513          	mv	a0,a5
   101b0:	02c12403          	lw	s0,44(sp)
   101b4:	03010113          	add	sp,sp,48
   101b8:	00008067          	ret
```
## Number of unique instruction
```
Number of different instructions: 12
List of unique instructions:
add
addi
and
bgeu
j
li
lw
mv
or
sll
sub
sw
```
## Word of thanks
I sciencerly thank Mr. Kunal Gosh(Founder/VSD) for helping me out to complete this flow smoothly.

## Acknowledgement
1. Kunal Ghosh, VSD Corp. Pvt. Ltd.
2. Skywater Foundry
3. Mayank Kabra

## Reference
1. https://github.com/SakethGajawada/RISCV-GNU
