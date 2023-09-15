# tec-interrupts
z80 interrupts


example of Z80 assembly code that demonstrates interrupt control along with stack operations using the Stack Pointer (SP). 

```assembly
; Z80 Interrupt Control and Stack Example

; Define memory addresses for ISR and interrupt vector
ORG 0000h        ; Start address for the program
ISR_ADDR equ 0038h ; Interrupt Service Routine (ISR) address
INT_VECTOR equ 0066h ; Interrupt vector address

; Initialize stack pointer (SP) to a safe location
LD SP, 0xFFFF   ; Initialize SP to the top of available RAM

; Enable interrupts
EI              ; Enable interrupts

; Main program loop
MAIN:
    ; Your main program logic goes here

    ; Polling a condition or performing other tasks

    ; Jump to the ISR if a specific condition is met
    ; For example, check a flag in the zero flag (Z)
    JR Z, CALL_ISR

    ; Continue with your main program
    ; ...

    ; Loop back to MAIN
    JR MAIN

; Define the Interrupt Service Routine (ISR)
CALL_ISR:
    ; Push registers onto the stack that you want to preserve
    PUSH AF         ; Push the accumulator and flags onto the stack

    ; Your ISR code goes here
    ; Handle the interrupt and perform necessary tasks

    ; Pop the registers back from the stack
    POP AF          ; Pop the accumulator and flags

    ; Return from the ISR
    RETI            ; Return from interrupt and re-enable interrupts

; Define the Interrupt Vector
; This tells the CPU where to jump when an interrupt occurs
; The value 0038h is the address of the CALL_ISR label
    ORG INT_VECTOR
    DW ISR_ADDR

; End of program
```

In this example:

1. We define memory addresses for the ISR and the interrupt vector. The ISR address (ISR_ADDR) is where our interrupt service routine is located, and the interrupt vector address (INT_VECTOR) is where the CPU will look to find the ISR address when an interrupt occurs.

2. We initialize the Stack Pointer (SP) to 0xFFFF, which is often used as the top of available RAM. You should adjust this to the actual memory location where you want to set up your stack.

3. We enable interrupts using the `EI` (Enable Interrupts) instruction.

4. In the `MAIN` loop, you can have your main program logic. When a specific condition is met (in this case, the Z flag is checked), we jump to the `CALL_ISR` label to handle the interrupt.

5. Inside the `CALL_ISR` label, we push the registers onto the stack to preserve their values, perform the ISR tasks, pop the registers back from the stack, and use `RETI` (Return from Interrupt) to return from the ISR.

6. Finally, we define the interrupt vector using the `ORG INT_VECTOR` directive and specify the address of the `CALL_ISR` label as the interrupt handler address.

Please note that this is a simplified example, and in a real-world scenario, you may need to set up more complex interrupt handling routines and possibly handle multiple interrupt sources. Additionally, you should adjust memory addresses and stack initialization to match your specific hardware and memory layout.
