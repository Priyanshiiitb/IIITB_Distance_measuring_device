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


## C Code
```
int trigger;
int echo;
int pulse_counter = 0;
int switch_val;

void read();
void pulse_count();
void distance_calc();
void trigger_send();
void delay_10us();
void delay_100ms();

// Function to simulate displaying distance on an LCD
void lcd_display_distance(float distance) {
    // Replace this function with your actual LCD display code
    // Example: lcd_set_cursor(row, col); lcd_print("Distance: %.2f cm", distance);
    // Make sure to initialize and configure your LCD library appropriately
}

int main() {
    while (1) {
        read();
    }
    return 0;
}

void read() {
    asm(
        "andi %0, x30, 1\n\t"
        : "=r"(switch_val));
    if (switch_val) {
        pulse_count();
        pulse_counter = 0;
        delay_100ms();
    }
}

void pulse_count() {
    trigger_send();
    asm(
        "andi %0, x30, 2\n\t"
        : "=r"(echo));
    while (echo) {
        pulse_counter = pulse_counter + 1;
    }
    distance_calc();
}

void trigger_send() {
    int trigger_reg;
    trigger = 1;
    trigger_reg = trigger * 8;
    asm(
        "or x30, x30, %0\n\t"
        : "=r"(switch_val));
    delay_10us();
    trigger = 0;
    trigger_reg = trigger * 8;
    asm(
        "or x30, x30, %0\n\t"
        : "=r"(switch_val));
}

void distance_calc() {
    float distance;
    distance = pulse_counter * 0.0344 * 0.5;
    
    // Call the LCD display function to show the distance
    lcd_display_distance(distance);
}

void delay_10us() {
    for (int i = 0; i < 10; i++);
}

void delay_100ms() {
    for (int i = 0; i < 100000; i++);
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
```
Dis.o:     file format elf64-littleriscv


Disassembly of section .text:

00000000000100b0 <main>:
   100b0:	001f7713          	andi	a4,t5,1
   100b4:	002f7613          	andi	a2,t5,2
   100b8:	0007069b          	sext.w	a3,a4
   100bc:	0006051b          	sext.w	a0,a2
   100c0:	00bf6f33          	or	t5,t5,a1
   100c4:	fae1a423          	sw	a4,-88(gp) # 11e30 <switch_val>
   100c8:	fe068ee3          	beqz	a3,100c4 <main+0x14>
   100cc:	fab1a423          	sw	a1,-88(gp) # 11e30 <switch_val>
   100d0:	fa01a223          	sw	zero,-92(gp) # 11e2c <trigger>
   100d4:	fac1a023          	sw	a2,-96(gp) # 11e28 <echo>
   100d8:	00050463          	beqz	a0,100e0 <main+0x30>
   100dc:	0000006f          	j	100dc <main+0x2c>
   100e0:	f601a023          	sw	zero,-160(gp) # 11de8 <_edata>
   100e4:	fe1ff06f          	j	100c4 <main+0x14>

00000000000100e8 <register_fini>:
   100e8:	ffff0797          	auipc	a5,0xffff0
   100ec:	f1878793          	addi	a5,a5,-232 # 0 <main-0x100b0>
   100f0:	00078863          	beqz	a5,10100 <register_fini+0x18>
   100f4:	00000517          	auipc	a0,0x0
   100f8:	18850513          	addi	a0,a0,392 # 1027c <__libc_fini_array>
   100fc:	1380006f          	j	10234 <atexit>
   10100:	00008067          	ret

0000000000010104 <_start>:
   10104:	00002197          	auipc	gp,0x2
   10108:	d8418193          	addi	gp,gp,-636 # 11e88 <__global_pointer$>
   1010c:	f6018513          	addi	a0,gp,-160 # 11de8 <_edata>
   10110:	fb018613          	addi	a2,gp,-80 # 11e38 <__BSS_END__>
   10114:	40a60633          	sub	a2,a2,a0
   10118:	00000593          	li	a1,0
   1011c:	250000ef          	jal	ra,1036c <memset>
   10120:	00000517          	auipc	a0,0x0
   10124:	15c50513          	addi	a0,a0,348 # 1027c <__libc_fini_array>
   10128:	10c000ef          	jal	ra,10234 <atexit>
   1012c:	1ac000ef          	jal	ra,102d8 <__libc_init_array>
   10130:	00012503          	lw	a0,0(sp)
   10134:	00810593          	addi	a1,sp,8
   10138:	00000613          	li	a2,0
   1013c:	f75ff0ef          	jal	ra,100b0 <main>
   10140:	1080006f          	j	10248 <exit>

0000000000010144 <__do_global_dtors_aux>:
   10144:	f681c783          	lbu	a5,-152(gp) # 11df0 <completed.5468>
   10148:	04079463          	bnez	a5,10190 <__do_global_dtors_aux+0x4c>
   1014c:	ffff0797          	auipc	a5,0xffff0
   10150:	eb478793          	addi	a5,a5,-332 # 0 <main-0x100b0>
   10154:	02078863          	beqz	a5,10184 <__do_global_dtors_aux+0x40>
   10158:	ff010113          	addi	sp,sp,-16
   1015c:	00001517          	auipc	a0,0x1
   10160:	50c50513          	addi	a0,a0,1292 # 11668 <__FRAME_END__>
   10164:	00113423          	sd	ra,8(sp)
   10168:	00000097          	auipc	ra,0x0
   1016c:	000000e7          	jalr	zero # 0 <main-0x100b0>
   10170:	00813083          	ld	ra,8(sp)
   10174:	00100793          	li	a5,1
   10178:	f6f18423          	sb	a5,-152(gp) # 11df0 <completed.5468>
   1017c:	01010113          	addi	sp,sp,16
   10180:	00008067          	ret
   10184:	00100793          	li	a5,1
   10188:	f6f18423          	sb	a5,-152(gp) # 11df0 <completed.5468>
   1018c:	00008067          	ret
   10190:	00008067          	ret

0000000000010194 <frame_dummy>:
   10194:	ffff0797          	auipc	a5,0xffff0
   10198:	e6c78793          	addi	a5,a5,-404 # 0 <main-0x100b0>
   1019c:	00078c63          	beqz	a5,101b4 <frame_dummy+0x20>
   101a0:	f7018593          	addi	a1,gp,-144 # 11df8 <object.5473>
   101a4:	00001517          	auipc	a0,0x1
   101a8:	4c450513          	addi	a0,a0,1220 # 11668 <__FRAME_END__>
   101ac:	00000317          	auipc	t1,0x0
   101b0:	00000067          	jr	zero # 0 <main-0x100b0>
   101b4:	00008067          	ret

00000000000101b8 <lcd_display_distance>:
   101b8:	00008067          	ret

00000000000101bc <read>:
   101bc:	001f7793          	andi	a5,t5,1
   101c0:	faf1a423          	sw	a5,-88(gp) # 11e30 <switch_val>
   101c4:	0007879b          	sext.w	a5,a5
   101c8:	02078463          	beqz	a5,101f0 <read+0x34>
   101cc:	00df6f33          	or	t5,t5,a3
   101d0:	fad1a423          	sw	a3,-88(gp) # 11e30 <switch_val>
   101d4:	002f7793          	andi	a5,t5,2
   101d8:	fa01a223          	sw	zero,-92(gp) # 11e2c <trigger>
   101dc:	faf1a023          	sw	a5,-96(gp) # 11e28 <echo>
   101e0:	0007879b          	sext.w	a5,a5
   101e4:	00078463          	beqz	a5,101ec <read+0x30>
   101e8:	0000006f          	j	101e8 <read+0x2c>
   101ec:	f601a023          	sw	zero,-160(gp) # 11de8 <_edata>
   101f0:	00008067          	ret

00000000000101f4 <pulse_count>:
   101f4:	fa01a223          	sw	zero,-92(gp) # 11e2c <trigger>
   101f8:	00ef6f33          	or	t5,t5,a4
   101fc:	fae1a423          	sw	a4,-88(gp) # 11e30 <switch_val>
   10200:	002f7793          	andi	a5,t5,2
   10204:	faf1a023          	sw	a5,-96(gp) # 11e28 <echo>
   10208:	0007879b          	sext.w	a5,a5
   1020c:	00078463          	beqz	a5,10214 <pulse_count+0x20>
   10210:	0000006f          	j	10210 <pulse_count+0x1c>
   10214:	00008067          	ret

0000000000010218 <trigger_send>:
   10218:	fa01a223          	sw	zero,-92(gp) # 11e2c <trigger>
   1021c:	00ff6f33          	or	t5,t5,a5
   10220:	faf1a423          	sw	a5,-88(gp) # 11e30 <switch_val>
   10224:	00008067          	ret

0000000000010228 <distance_calc>:
   10228:	00008067          	ret

000000000001022c <delay_10us>:
   1022c:	00008067          	ret

0000000000010230 <delay_100ms>:
   10230:	00008067          	ret

0000000000010234 <atexit>:
   10234:	00050593          	mv	a1,a0
   10238:	00000693          	li	a3,0
   1023c:	00000613          	li	a2,0
   10240:	00000513          	li	a0,0
   10244:	2040006f          	j	10448 <__register_exitproc>

0000000000010248 <exit>:
   10248:	ff010113          	addi	sp,sp,-16
   1024c:	00000593          	li	a1,0
   10250:	00813023          	sd	s0,0(sp)
   10254:	00113423          	sd	ra,8(sp)
   10258:	00050413          	mv	s0,a0
   1025c:	298000ef          	jal	ra,104f4 <__call_exitprocs>
   10260:	f4818793          	addi	a5,gp,-184 # 11dd0 <_global_impure_ptr>
   10264:	0007b503          	ld	a0,0(a5)
   10268:	05853783          	ld	a5,88(a0)
   1026c:	00078463          	beqz	a5,10274 <exit+0x2c>
   10270:	000780e7          	jalr	a5
   10274:	00040513          	mv	a0,s0
   10278:	3a0000ef          	jal	ra,10618 <_exit>

000000000001027c <__libc_fini_array>:
   1027c:	fe010113          	addi	sp,sp,-32
   10280:	00813823          	sd	s0,16(sp)
   10284:	00001797          	auipc	a5,0x1
   10288:	40478793          	addi	a5,a5,1028 # 11688 <__fini_array_end>
   1028c:	00001417          	auipc	s0,0x1
   10290:	3f440413          	addi	s0,s0,1012 # 11680 <__init_array_end>
   10294:	408787b3          	sub	a5,a5,s0
   10298:	00913423          	sd	s1,8(sp)
   1029c:	00113c23          	sd	ra,24(sp)
   102a0:	4037d493          	srai	s1,a5,0x3
   102a4:	02048063          	beqz	s1,102c4 <__libc_fini_array+0x48>
   102a8:	ff878793          	addi	a5,a5,-8
   102ac:	00878433          	add	s0,a5,s0
   102b0:	00043783          	ld	a5,0(s0)
   102b4:	fff48493          	addi	s1,s1,-1
   102b8:	ff840413          	addi	s0,s0,-8
   102bc:	000780e7          	jalr	a5
   102c0:	fe0498e3          	bnez	s1,102b0 <__libc_fini_array+0x34>
   102c4:	01813083          	ld	ra,24(sp)
   102c8:	01013403          	ld	s0,16(sp)
   102cc:	00813483          	ld	s1,8(sp)
   102d0:	02010113          	addi	sp,sp,32
   102d4:	00008067          	ret

00000000000102d8 <__libc_init_array>:
   102d8:	fe010113          	addi	sp,sp,-32
   102dc:	00813823          	sd	s0,16(sp)
   102e0:	01213023          	sd	s2,0(sp)
   102e4:	00001417          	auipc	s0,0x1
   102e8:	38840413          	addi	s0,s0,904 # 1166c <__preinit_array_end>
   102ec:	00001917          	auipc	s2,0x1
   102f0:	38090913          	addi	s2,s2,896 # 1166c <__preinit_array_end>
   102f4:	40890933          	sub	s2,s2,s0
   102f8:	00113c23          	sd	ra,24(sp)
   102fc:	00913423          	sd	s1,8(sp)
   10300:	40395913          	srai	s2,s2,0x3
   10304:	00090e63          	beqz	s2,10320 <__libc_init_array+0x48>
   10308:	00000493          	li	s1,0
   1030c:	00043783          	ld	a5,0(s0)
   10310:	00148493          	addi	s1,s1,1
   10314:	00840413          	addi	s0,s0,8
   10318:	000780e7          	jalr	a5
   1031c:	fe9918e3          	bne	s2,s1,1030c <__libc_init_array+0x34>
   10320:	00001417          	auipc	s0,0x1
   10324:	35040413          	addi	s0,s0,848 # 11670 <__init_array_start>
   10328:	00001917          	auipc	s2,0x1
   1032c:	35890913          	addi	s2,s2,856 # 11680 <__init_array_end>
   10330:	40890933          	sub	s2,s2,s0
   10334:	40395913          	srai	s2,s2,0x3
   10338:	00090e63          	beqz	s2,10354 <__libc_init_array+0x7c>
   1033c:	00000493          	li	s1,0
   10340:	00043783          	ld	a5,0(s0)
   10344:	00148493          	addi	s1,s1,1
   10348:	00840413          	addi	s0,s0,8
   1034c:	000780e7          	jalr	a5
   10350:	fe9918e3          	bne	s2,s1,10340 <__libc_init_array+0x68>
   10354:	01813083          	ld	ra,24(sp)
   10358:	01013403          	ld	s0,16(sp)
   1035c:	00813483          	ld	s1,8(sp)
   10360:	00013903          	ld	s2,0(sp)
   10364:	02010113          	addi	sp,sp,32
   10368:	00008067          	ret

000000000001036c <memset>:
   1036c:	00f00313          	li	t1,15
   10370:	00050713          	mv	a4,a0
   10374:	02c37a63          	bgeu	t1,a2,103a8 <memset+0x3c>
   10378:	00f77793          	andi	a5,a4,15
   1037c:	0a079063          	bnez	a5,1041c <memset+0xb0>
   10380:	06059e63          	bnez	a1,103fc <memset+0x90>
   10384:	ff067693          	andi	a3,a2,-16
   10388:	00f67613          	andi	a2,a2,15
   1038c:	00e686b3          	add	a3,a3,a4
   10390:	00b73023          	sd	a1,0(a4)
   10394:	00b73423          	sd	a1,8(a4)
   10398:	01070713          	addi	a4,a4,16
   1039c:	fed76ae3          	bltu	a4,a3,10390 <memset+0x24>
   103a0:	00061463          	bnez	a2,103a8 <memset+0x3c>
   103a4:	00008067          	ret
   103a8:	40c306b3          	sub	a3,t1,a2
   103ac:	00269693          	slli	a3,a3,0x2
   103b0:	00000297          	auipc	t0,0x0
   103b4:	005686b3          	add	a3,a3,t0
   103b8:	00c68067          	jr	12(a3)
   103bc:	00b70723          	sb	a1,14(a4)
   103c0:	00b706a3          	sb	a1,13(a4)
   103c4:	00b70623          	sb	a1,12(a4)
   103c8:	00b705a3          	sb	a1,11(a4)
   103cc:	00b70523          	sb	a1,10(a4)
   103d0:	00b704a3          	sb	a1,9(a4)
   103d4:	00b70423          	sb	a1,8(a4)
   103d8:	00b703a3          	sb	a1,7(a4)
   103dc:	00b70323          	sb	a1,6(a4)
   103e0:	00b702a3          	sb	a1,5(a4)
   103e4:	00b70223          	sb	a1,4(a4)
   103e8:	00b701a3          	sb	a1,3(a4)
   103ec:	00b70123          	sb	a1,2(a4)
   103f0:	00b700a3          	sb	a1,1(a4)
   103f4:	00b70023          	sb	a1,0(a4)
   103f8:	00008067          	ret
   103fc:	0ff5f593          	andi	a1,a1,255
   10400:	00859693          	slli	a3,a1,0x8
   10404:	00d5e5b3          	or	a1,a1,a3
   10408:	01059693          	slli	a3,a1,0x10
   1040c:	00d5e5b3          	or	a1,a1,a3
   10410:	02059693          	slli	a3,a1,0x20
   10414:	00d5e5b3          	or	a1,a1,a3
   10418:	f6dff06f          	j	10384 <memset+0x18>
   1041c:	00279693          	slli	a3,a5,0x2
   10420:	00000297          	auipc	t0,0x0
   10424:	005686b3          	add	a3,a3,t0
   10428:	00008293          	mv	t0,ra
   1042c:	f98680e7          	jalr	-104(a3)
   10430:	00028093          	mv	ra,t0
   10434:	ff078793          	addi	a5,a5,-16
   10438:	40f70733          	sub	a4,a4,a5
   1043c:	00f60633          	add	a2,a2,a5
   10440:	f6c374e3          	bgeu	t1,a2,103a8 <memset+0x3c>
   10444:	f3dff06f          	j	10380 <memset+0x14>

0000000000010448 <__register_exitproc>:
   10448:	f4818793          	addi	a5,gp,-184 # 11dd0 <_global_impure_ptr>
   1044c:	0007b703          	ld	a4,0(a5)
   10450:	1f873783          	ld	a5,504(a4)
   10454:	06078063          	beqz	a5,104b4 <__register_exitproc+0x6c>
   10458:	0087a703          	lw	a4,8(a5)
   1045c:	01f00813          	li	a6,31
   10460:	08e84663          	blt	a6,a4,104ec <__register_exitproc+0xa4>
   10464:	02050863          	beqz	a0,10494 <__register_exitproc+0x4c>
   10468:	00371813          	slli	a6,a4,0x3
   1046c:	01078833          	add	a6,a5,a6
   10470:	10c83823          	sd	a2,272(a6)
   10474:	3107a883          	lw	a7,784(a5)
   10478:	00100613          	li	a2,1
   1047c:	00e6163b          	sllw	a2,a2,a4
   10480:	00c8e8b3          	or	a7,a7,a2
   10484:	3117a823          	sw	a7,784(a5)
   10488:	20d83823          	sd	a3,528(a6)
   1048c:	00200693          	li	a3,2
   10490:	02d50863          	beq	a0,a3,104c0 <__register_exitproc+0x78>
   10494:	00270693          	addi	a3,a4,2
   10498:	00369693          	slli	a3,a3,0x3
   1049c:	0017071b          	addiw	a4,a4,1
   104a0:	00e7a423          	sw	a4,8(a5)
   104a4:	00d787b3          	add	a5,a5,a3
   104a8:	00b7b023          	sd	a1,0(a5)
   104ac:	00000513          	li	a0,0
   104b0:	00008067          	ret
   104b4:	20070793          	addi	a5,a4,512
   104b8:	1ef73c23          	sd	a5,504(a4)
   104bc:	f9dff06f          	j	10458 <__register_exitproc+0x10>
   104c0:	3147a683          	lw	a3,788(a5)
   104c4:	00000513          	li	a0,0
   104c8:	00c6e633          	or	a2,a3,a2
   104cc:	00270693          	addi	a3,a4,2
   104d0:	00369693          	slli	a3,a3,0x3
   104d4:	0017071b          	addiw	a4,a4,1
   104d8:	30c7aa23          	sw	a2,788(a5)
   104dc:	00e7a423          	sw	a4,8(a5)
   104e0:	00d787b3          	add	a5,a5,a3
   104e4:	00b7b023          	sd	a1,0(a5)
   104e8:	00008067          	ret
   104ec:	fff00513          	li	a0,-1
   104f0:	00008067          	ret

00000000000104f4 <__call_exitprocs>:
   104f4:	fb010113          	addi	sp,sp,-80
   104f8:	f4818793          	addi	a5,gp,-184 # 11dd0 <_global_impure_ptr>
   104fc:	01813023          	sd	s8,0(sp)
   10500:	0007bc03          	ld	s8,0(a5)
   10504:	03313423          	sd	s3,40(sp)
   10508:	03413023          	sd	s4,32(sp)
   1050c:	01513c23          	sd	s5,24(sp)
   10510:	01613823          	sd	s6,16(sp)
   10514:	04113423          	sd	ra,72(sp)
   10518:	04813023          	sd	s0,64(sp)
   1051c:	02913c23          	sd	s1,56(sp)
   10520:	03213823          	sd	s2,48(sp)
   10524:	01713423          	sd	s7,8(sp)
   10528:	00050a93          	mv	s5,a0
   1052c:	00058b13          	mv	s6,a1
   10530:	00100a13          	li	s4,1
   10534:	fff00993          	li	s3,-1
   10538:	1f8c3903          	ld	s2,504(s8)
   1053c:	02090863          	beqz	s2,1056c <__call_exitprocs+0x78>
   10540:	00892483          	lw	s1,8(s2)
   10544:	fff4841b          	addiw	s0,s1,-1
   10548:	02044263          	bltz	s0,1056c <__call_exitprocs+0x78>
   1054c:	00349493          	slli	s1,s1,0x3
   10550:	009904b3          	add	s1,s2,s1
   10554:	040b0463          	beqz	s6,1059c <__call_exitprocs+0xa8>
   10558:	2084b783          	ld	a5,520(s1)
   1055c:	05678063          	beq	a5,s6,1059c <__call_exitprocs+0xa8>
   10560:	fff4041b          	addiw	s0,s0,-1
   10564:	ff848493          	addi	s1,s1,-8
   10568:	ff3416e3          	bne	s0,s3,10554 <__call_exitprocs+0x60>
   1056c:	04813083          	ld	ra,72(sp)
   10570:	04013403          	ld	s0,64(sp)
   10574:	03813483          	ld	s1,56(sp)
   10578:	03013903          	ld	s2,48(sp)
   1057c:	02813983          	ld	s3,40(sp)
   10580:	02013a03          	ld	s4,32(sp)
   10584:	01813a83          	ld	s5,24(sp)
   10588:	01013b03          	ld	s6,16(sp)
   1058c:	00813b83          	ld	s7,8(sp)
   10590:	00013c03          	ld	s8,0(sp)
   10594:	05010113          	addi	sp,sp,80
   10598:	00008067          	ret
   1059c:	00892783          	lw	a5,8(s2)
   105a0:	0084b703          	ld	a4,8(s1)
   105a4:	fff7879b          	addiw	a5,a5,-1
   105a8:	04878e63          	beq	a5,s0,10604 <__call_exitprocs+0x110>
   105ac:	0004b423          	sd	zero,8(s1)
   105b0:	fa0708e3          	beqz	a4,10560 <__call_exitprocs+0x6c>
   105b4:	31092783          	lw	a5,784(s2)
   105b8:	008a16bb          	sllw	a3,s4,s0
   105bc:	00892b83          	lw	s7,8(s2)
   105c0:	00d7f7b3          	and	a5,a5,a3
   105c4:	0007879b          	sext.w	a5,a5
   105c8:	00079e63          	bnez	a5,105e4 <__call_exitprocs+0xf0>
   105cc:	000700e7          	jalr	a4
   105d0:	00892783          	lw	a5,8(s2)
   105d4:	f77792e3          	bne	a5,s7,10538 <__call_exitprocs+0x44>
   105d8:	1f8c3783          	ld	a5,504(s8)
   105dc:	f92782e3          	beq	a5,s2,10560 <__call_exitprocs+0x6c>
   105e0:	f59ff06f          	j	10538 <__call_exitprocs+0x44>
   105e4:	31492783          	lw	a5,788(s2)
   105e8:	1084b583          	ld	a1,264(s1)
   105ec:	00d7f7b3          	and	a5,a5,a3
   105f0:	0007879b          	sext.w	a5,a5
   105f4:	00079c63          	bnez	a5,1060c <__call_exitprocs+0x118>
   105f8:	000a8513          	mv	a0,s5
   105fc:	000700e7          	jalr	a4
   10600:	fd1ff06f          	j	105d0 <__call_exitprocs+0xdc>
   10604:	00892423          	sw	s0,8(s2)
   10608:	fa9ff06f          	j	105b0 <__call_exitprocs+0xbc>
   1060c:	00058513          	mv	a0,a1
   10610:	000700e7          	jalr	a4
   10614:	fbdff06f          	j	105d0 <__call_exitprocs+0xdc>

0000000000010618 <_exit>:
   10618:	00000593          	li	a1,0
   1061c:	00000613          	li	a2,0
   10620:	00000693          	li	a3,0
   10624:	00000713          	li	a4,0
   10628:	00000793          	li	a5,0
   1062c:	05d00893          	li	a7,93
   10630:	00000073          	ecall
   10634:	00054463          	bltz	a0,1063c <_exit+0x24>
   10638:	0000006f          	j	10638 <_exit+0x20>
   1063c:	ff010113          	addi	sp,sp,-16
   10640:	00813023          	sd	s0,0(sp)
   10644:	00050413          	mv	s0,a0
   10648:	00113423          	sd	ra,8(sp)
   1064c:	4080043b          	negw	s0,s0
   10650:	00c000ef          	jal	ra,1065c <__errno>
   10654:	00852023          	sw	s0,0(a0)
   10658:	0000006f          	j	10658 <_exit+0x40>

000000000001065c <__errno>:
   1065c:	f5818793          	addi	a5,gp,-168 # 11de0 <_impure_ptr>
   10660:	0007b503          	ld	a0,0(a5)
   10664:	00008067          	ret
```
