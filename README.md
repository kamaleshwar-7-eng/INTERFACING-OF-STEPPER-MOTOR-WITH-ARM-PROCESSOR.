
# INTERFACING OF STEPPER MOTOR WITH ARM PROCESSOR (LPC1768)

## AIM
To interface a stepper motor with the ARM LPC1768 microcontroller and control its rotation using GPIO programming.

## COMPONENTS REQUIRED

### Hardware
- ARM LPC1768 development board
- Four-phase stepper motor
- ULN2003 motor driver (for a compatible unipolar motor)
- External motor power supply, as required by the motor
- Jumper wires
- Breadboard or connecting terminals

### Software
- Keil µVision 4 IDE
- ARM Compiler
- LPC1768 CMSIS device header and startup files
- `fromelf` utility for generating the BIN file

## PROCEDURE

1. Open Keil µVision and select **Project → New µVision Project**.
2. Create a project folder and save the project.
3. Select **NXP → LPC1768** in the device selection window.
4. Include the appropriate LPC17xx startup file.
5. Create a new C source file and save it as `main.c`.
6. Add `main.c` and the required system source file to Source Group 1.
7. Configure GPIO pins P1.20, P1.21, P1.22, and P1.23 as outputs.
8. Connect these GPIO pins to the four inputs of a compatible motor driver.
9. Connect the motor windings to the driver outputs according to the motor wiring diagram.
10. Connect the motor supply and establish a common ground between the driver and the LPC1768.
11. Write the program to energize the motor phases in sequence.
12. Build the project and resolve any compilation errors.
13. Configure the post-build command to generate a BIN file if required.
14. Flash the application to the LPC1768 using the appropriate programming tool.
15. Power the circuit and observe the stepper motor rotation.

## CONNECTION DETAILS

### LPC1768 to ULN2003 Driver

| LPC1768 GPIO | ULN2003 Input |
|---|---|
| P1.20 | IN1 |
| P1.21 | IN2 |
| P1.22 | IN3 |
| P1.23 | IN4 |
| GND | GND |

### Motor Driver Connections

| ULN2003 Pin/Terminal | Connection |
|---|---|
| OUT1–OUT4 | Corresponding motor winding ends |
| COM | Positive motor supply, for the internal flyback diodes |
| GND | Common ground |
| Motor common wire(s) | Motor supply positive, as required by the motor wiring |

**Important:** These connections assume a compatible unipolar stepper motor. A bipolar stepper motor requires a suitable dual H-bridge driver, such as an appropriate stepper driver module. Identify the motor winding configuration and rated voltage before connecting the hardware.

## PROGRAM

### main.c

```c
#include <LPC17xx.h>

#define MOTOR_MASK (0x0FU << 20)

/* Four-phase output sequence */
unsigned int step_sequence[4] =
{
    (1U << 20),
    (1U << 21),
    (1U << 22),
    (1U << 23)
};

void delay(void)
{
    volatile unsigned int i;

    for (i = 0; i < 500000; i++);
}

int main(void)
{
    unsigned int i;

    /* Select GPIO function for P1.20 to P1.23 */
    LPC_PINCON->PINSEL3 &= ~(0xFFU << 8);

    /* Configure P1.20 to P1.23 as outputs */
    LPC_GPIO1->FIODIR |= MOTOR_MASK;

    /* Initially switch all phases OFF */
    LPC_GPIO1->FIOCLR = MOTOR_MASK;

    while (1)
    {
        /* Energize motor phases sequentially */
        for (i = 0; i < 4; i++)
        {
            LPC_GPIO1->FIOCLR = MOTOR_MASK;
            LPC_GPIO1->FIOSET = step_sequence[i];

            delay();
        }
    }
}
```

## WORKING PRINCIPLE

1. The LPC1768 generates digital control signals through GPIO pins P1.20–P1.23.
2. The ULN2003 driver amplifies the control capability by switching the motor winding currents.
3. The motor phases are energized in a fixed sequence.
4. Each phase change moves the rotor by a step.
5. Repeating the sequence produces continuous rotation.
6. The rotation speed depends on the delay between phase changes.
7. The rotation direction can be reversed by reversing the phase sequence.

**Note:** The program uses a simple single-phase-at-a-time sequence. The actual rotation direction and operation depend on the motor's winding arrangement and the order in which the driver outputs are connected. The software delay is approximate and depends on the processor clock and compiler settings.

## KEIL µVISION CONFIGURATION

### Required Project Files

```text
INTERFACING-OF-STEPPER-MOTOR-WITH-ARM-PROCESSOR/
├── README.md
├── main.c
├── startup_LPC17xx.s
└── system_LPC17xx.c
```

The startup and system files must match the LPC1768 device and the installed CMSIS/Keil package.

### Generate the BIN File

1. Open **Options for Target → Target**.
2. Configure the flash memory settings according to the board and bootloader.
3. Build the project to generate the AXF file.
4. Open **Options for Target → User → After Build/Rebuild**.
5. Enter a command similar to the following, adjusting the AXF path to your project:

```bat
fromelf --bin ".\Objects\Stepper_Motor.axf" --output ".\Objects\stepper.bin"
```

6. Rebuild the project.
7. Verify that `stepper.bin` is generated in the output folder.

If your bootloader requires the application to start at `0x2000`, configure the linker and startup/vector-table handling accordingly. Do not change the address without confirming the bootloader requirements.

## OUTPUT

The stepper motor rotates continuously as the LPC1768 energizes its phases in sequence.

## RESULT

The stepper motor was interfaced with the ARM LPC1768 microcontroller using GPIO pins and a suitable motor driver. Sequential phase excitation was used to control the motor rotation.
