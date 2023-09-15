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

## Maintaining an interrupt daisy chain 
on the Z80 involves connecting multiple devices in a series such that each device's interrupt output is connected to the interrupt input of the next device in the chain. The Z80's `INT` and `IORQ` lines are also connected to this chain. 

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

## Other notable examples 
related to Z80 interrupt handling could cover different interrupt modes, like `IM 0` (Interrupt Mode 0) and `IM 1` (Interrupt Mode 1), or specific use-cases like handling keyboard inputs, or timing functions using interrupts. Below are some additional examples:

### 1. Interrupt Mode 0 (IM 0)

In this mode, the data put on the data bus during an interrupt acknowledge cycle is executed as an opcode. Typically, a single `RST` instruction is placed on the bus.

```assembly
ORG 0000h
LD SP, 0xF000     ; Initialize the Stack Pointer
DI                ; Disable interrupts during initialization
IM 0              ; Set to Interrupt Mode 0
EI                ; Enable interrupts

MAIN:
    ; Your main loop
    JR MAIN

; ISR would typically be a single instruction like RST placed on the data bus by hardware.
; Let's assume RST 30h is placed on the data bus, then ISR at 0030h will be called.
ORG 0x0030
ISR_IM0:
    PUSH AF
    ; Your ISR code for IM 0
    POP AF
    RET
```

### 2. Interrupt Mode 1 (IM 1)

In this mode, upon receiving an interrupt, the Z80 will automatically jump to location `0x0038`. This mode is easier to handle compared to `IM 0` or `IM 2`.

```assembly
ORG 0000h
LD SP, 0xF000     ; Initialize the Stack Pointer
DI                ; Disable interrupts during initialization
IM 1              ; Set to Interrupt Mode 1
EI                ; Enable interrupts

MAIN:
    ; Your main loop
    JR MAIN

; ISR at 0x0038 for IM 1
ORG 0x0038
ISR_IM1:
    PUSH AF
    ; Your ISR code for IM 1
    POP AF
    RET
```

### 3. Handling Keyboard Inputs using Interrupts

```assembly
ORG 0000h
LD SP, 0xF000     ; Initialize the Stack Pointer
DI                ; Disable interrupts during initialization
IM 1              ; Set to Interrupt Mode 1
EI                ; Enable interrupts

MAIN:
    ; Your main loop
    JR MAIN

; ISR for handling keyboard inputs
ORG 0x0038
KEYBOARD_ISR:
    PUSH AF
    ; Read the character from the keyboard port (e.g., IN A, (0xFF))
    ; Process the keypress
    POP AF
    RET
```

### 4. Using Interrupts for Timing Functions

```assembly
ORG 0000h
LD SP, 0xF000     ; Initialize the Stack Pointer
DI                ; Disable interrupts during initialization
IM 1              ; Set to Interrupt Mode 1
EI                ; Enable interrupts

; Initialize timer counter
LD A, 0x00
LD (0xF100), A    ; Let's say 0xF100 is our timer counter

MAIN:
    ; Your main loop
    JR MAIN

; ISR for incrementing a timer
ORG 0x0038
TIMER_ISR:
    PUSH AF
    LD A, (0xF100)
    INC A
    LD (0xF100), A ; Increment timer counter
    POP AF
    RET
```

In all of these examples, the ISRs push the `AF` register pair onto the stack to preserve its value and then pop it back before returning. Depending on what your ISR is doing, you might also need to push and pop other registers.
