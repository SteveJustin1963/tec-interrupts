# tec-interrupts


example specifically for the Z80 CPU. Here are some points to note for Z80-specific implementation:

1. The Z80 CPU automatically jumps to address `0x0038` for a maskable interrupt (Mode 1), so the ISR should ideally be located there unless you're setting up a more advanced interrupt handling mechanism. 

2. The `RETI` instruction is designed for systems that use more than one interrupt source or use prioritized interrupts. If your system only has one interrupt source and you don't need to maintain the interrupt daisy chain, a simple `RET` would suffice.

Here's a rewritten version with these Z80-specific considerations:

```assembly
; Z80 Specific Interrupt Control and Stack Example

ORG 0000h         ; Start address for the program
LD SP, 0xF000     ; Initialize Stack Pointer to a known location (tailor this as per your hardware)

DI                ; Disable interrupts during initialization
EI                ; Enable interrupts once initialization is done

; Main program loop starts here
MAIN:
    ; Main program logic
    ; Perform tasks, poll devices etc.
  
    ; Loop back to MAIN
    JR MAIN

; Interrupt Service Routine (ISR) starts here
; On Z80, by default, a maskable interrupt jumps to address 0x0038
ORG 0x0038        
ISR:
    PUSH AF        ; Save accumulator and flags onto the stack
    PUSH BC        ; Save additional registers if needed
    
    ; Perform the tasks required during an interrupt here
    
    POP BC         ; Restore saved registers
    POP AF         ; Restore accumulator and flags
    RET            ; Return from ISR (consider RETI if you have more than one interrupt source)

; End of program
```

In this revised example:

- The program's ORG starts at `0000h`, typical for many Z80 applications.
- I've changed the Stack Pointer initialization to `0xF000` for the sake of the example. In a real-world scenario, the stack should be initialized to an appropriate location based on your specific hardware configuration.
- The `DI` and `EI` instructions are used to disable and enable interrupts. This is important to ensure that an interrupt doesn't occur during system initialization.
- The ISR is specifically placed at `0x0038`, which is where the Z80 CPU looks by default when a maskable interrupt occurs.
- I've replaced `RETI` with `RET` since, in a single interrupt source system for the Z80, `RET` is often sufficient.

With this setup, the Z80's default interrupt behavior should invoke the ISR when an interrupt occurs, without needing to set an interrupt vector table or jump to the ISR manually.

Maintaining an interrupt daisy chain on the Z80 involves connecting multiple devices in a series such that each device's interrupt output is connected to the interrupt input of the next device in the chain. The Z80's `INT` and `IORQ` lines are also connected to this chain. 

Devices in the chain typically require an `EI` (Enable Interrupts) command to enable their individual interrupt outputs. A device can interrupt the CPU only if all preceding devices in the chain allow it.

In the daisy chain mechanism, when an interrupt occurs, the Z80 CPU outputs a series of `IORQ` pulses to each device in the chain. Each device decides whether to respond based on its interrupt priority. If a device decides to interrupt, it places its interrupt vector on the data bus.

The following assembly code provides a simple example to maintain an interrupt daisy chain using the Z80's `IM 2` mode, which allows for vectored interrupts. In this mode, the Z80 expects an interrupt vector from the device that is interrupting.

```assembly
; Z80 Interrupt Daisy Chain Example

ORG 0000h         ; Start address for the program

; Initialize stack pointer to a known location
LD SP, 0xF000

; Disable interrupts initially
DI

; Initialization code for daisy-chained devices goes here
; (e.g., sending EI command to each device)

; Set interrupt mode to IM 2 (Vectored interrupt)
IM 2

; Initialize the I register for the base address of the vector table
LD I, 0x70

; Enable interrupts
EI

; Main program loop starts here
MAIN:
    ; Main program logic
    ; Perform tasks, poll devices, etc.
  
    ; Loop back to MAIN
    JR MAIN

; Default ISR Handler (This will be called if no device responds)
ORG 0x0038
DEFAULT_ISR:
    PUSH AF
    ; Handle unexpected interrupt
    POP AF
    RETI

; Vector Table
; In IM 2, Z80 looks up the vector address = (I register content) << 8 + (Vector supplied by device)
ORG 0x7000       ; Starting at the base address set in the I register
    DW DEVICE1_ISR
    DW DEVICE2_ISR
    ;... More ISR addresses can go here

; Device 1 ISR
ORG 0x8000
DEVICE1_ISR:
    PUSH AF
    ; Handle interrupt from Device 1
    POP AF
    RETI

; Device 2 ISR
ORG 0x8100
DEVICE2_ISR:
    PUSH AF
    ; Handle interrupt from Device 2
    POP AF
    RETI

; End of program
```

In this example:

- I used the `IM 2` instruction to set the Z80 to interrupt mode 2, which is for vectored interrupts.
- The `I` register is initialized with `0x70`. In interrupt mode 2, the interrupt vector is formed by combining the contents of the `I` register with the byte supplied by the peripheral device. The resulting address will point to a location in the vector table (`ORG 0x7000` in this case) that contains the address of the ISR to execute.
- `DEVICE1_ISR` and `DEVICE2_ISR` are example ISRs for two daisy-chained devices. These ISRs would handle interrupts from their respective devices.
- I've also included a default ISR (`DEFAULT_ISR`) at address `0x0038`. This ISR will be executed if no device in the daisy chain acknowledges the interrupt.

Note: The addresses (`ORG`) for ISRs, the vector table, and other elements are illustrative. In a real-world application, these would be defined based on your specific hardware and system design.

