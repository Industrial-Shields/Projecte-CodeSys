# CODESYS project templates for Industrial Shield's Raspberry PLCs
**by Industrial Shields**

This repository contains project templates for CODESYS, set up to work with Raspberry PLCs

## Initialisation
An IS PLC requires the i2c bus to be active in order for the GPIOS to work. This must be initiated in the PLC main program with the method `i2c_init()` once every time the PLC starts:
```
setup : BOOL := TRUE;
```
```
IF setup THEN
  i2c_init();
  setup := FALSE;
END_IF
```

## PLC GPIOs
A custom set of Function Blocks is required, which can be found on the project templates.
Analog and Relay PLCs internally differ on how to reference the GPIOs location. Therefore, the specific Function Block must be used.
### Analog PLC
Analog PLCs use the GPIOs() function block. This must be declared on the variable as `Var : GPIOs := GPIOs(WSTR)`.

WSTR must be the name of the IOs in the PLC. Some valid names would be "I0.12", "Q2.2", "A0.6"...

**List of methods**
* **GPIOs.read_analog()** reads an analog input. Returns a UINT.
* **GPIOs.read_digital()** reads a digital input. Returns a BOOL.
* **GPIOs.write_analog(UINT)** writes an analog output. Requires an UINT, returns a BOOL.
* **GPIOs.write_analog(BOOL)** writes a digital output. Requires an BOOL, returns a BOOL.

### Relay PLC
Relay PLCs use the GPIOsR() function block. It must be initialised as `Var : GPIOsR := GPIOsR(WSTR)`.

WSTR must be the name of the IOs in the PLC. It works the same as GPIOs FB, with the addition of relay declaration. For example: "R0.5"

**List of methods**
* **GPIOsR.read_analog()** reads an analog input. Returns a UINT.
* **GPIOsR.read_digital()** reads a digital input. Returns a BOOL.
* **GPIOsR.read_relay()** reads the state of a relay. It's not a voltage reading passing through the relay. Returns a BOOL.
* **GPIOsR.write_analog(UINT)** writes an analog output. Requires an UINT, returns a BOOL.
* **GPIOsR.write_analog(BOOL)** writes a digital output. Requires an BOOL, returns a BOOL.
* **GPIOsR.write_relay(BOOL)** writes to a relay. Requires a BOOL, returns a BOOL.

### Working Example
```
PROGRAM PLC_PRG
VAR
Var1 : GPIOs := GPIOs(“I0.7”);
Var2 : GPIOs := GPIOs(“Q0.0”);
i : UINT;
i2: BOOL := TRUE;
END_VAR
```
```
i := Var1.read();
Var2.write(i2);
```
## Direct GPIOs
Direct GPIOs must be declared differently from other GPIOs. 

List of direct GPIOs found in RPI PLCs:
| Analog PLCs | | Relay PLCs | |
| --- | --- | --- | --- |
| Name | RPI pin | Name | RPI pin |
| I0.5 | 13 | I0.0 | 13 |
| I0.6 | 12 | I0.1 | 12 |
| I1.5 | 27 | I1.0 | 27 |
| I1.6 | 5  | I1.1 | 5  |
| I2.5 | 26 | I2.0 | 26 |
| I2.6 | 4  | I2.1 | 4  |

A series of steps must be followed in order to add a direct GPIO to the project:
### 1. Declare variable in the program header
```
PROGRAM PLC_PRG
VAR
DirectVar : BOOL;
i : BOOL;
END_VAR
```
### 2. Add it to the Raspberry's GPIOs table

In GPIOs_A_B (GPIOs A/B) --> GPIOs Parameters:
* Set desired GPIO to Input or Output state

In GPIOs_A_B (GPIOs A/B) --> GPIOs I/O Mapping:
* Set variable. It should look something like `Applicaton.PLC_PRG.DirectVar`
### 3. Use in main program
```
i2 := DirectVar;
```
