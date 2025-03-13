# CODESYS project templates for Industrial Shield's Raspberry PLCs
**by Industrial Shields**

This repository contains project templates for CODESYS, set up to work with Raspberry PLCs v6.
None of those implementations are the standard for CODESYS, and use paradigms closer to regular programming instead of PLC programming.

**Warning!! None of this methods are standard for CODESYS and aren't recommended to be used on a commercial environment**

There are currently 2 methods for accessing the GPIOs. They're referred in this README as "old" and "most recent".

# "Old" method
This method is the quickest implementation for accessing the RPIPLC's GPIOs. The catch is it only works with RPIPLC v6.

* [Initialisation](https://github.com/Industrial-Shields/Projecte-CodeSys/blob/main/README.md#initialisation)
* [GPIOs (Structured Text)](https://github.com/Industrial-Shields/Projecte-CodeSys/blob/main/README.md#plc-gpios)
* [Ladder](https://github.com/Industrial-Shields/Projecte-CodeSys/blob/main/README.md#ladder)

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
| Analog PLC | Relay PLC | RPI pin |
| --- | --- | --- |
| I0.5 | I0.0 | 13 |
| I0.6 | I0.1 | 12 |
| I1.5 | I1.0 | 27 |
| I1.6 | I1.1 | 5  |
| I2.5 | I2.0 | 26 |
| I2.6 | I2.1 | 4  |

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
Update "GPIOs A/B" to "GPIOs B+/Pi2"

In GPIOs_A_B (GPIOs B+/Pi2) --> GPIOs Parameters:
* Set desired GPIO to Input or Output state

In GPIOs_A_B (GPIOs B+/Pi2) --> GPIOs I/O Mapping:
* Set variable. It should look something like `Applicaton.PLC_PRG.DirectVar`
### 3. Use in main program
```
i2 := DirectVar;
```

# "Most recent" method
It consists on a library that calls at the functions on `~/test/RPIPLC_V*/` to get and set values on the GPIOs. With this, it can work with v4 RPIPLCs.

This library uses a lot of concatenations, therefore it's slow and not recommended for serious usage.

- [ ] An update for this version is currently being worked on to enhance library speed.


## Usage
### Declaring GPIOs
The GPIOs in this are declared as an object that acts on them all, instead of individual instances
```
PROGRAM PLC_PRG
VAR
rpiplc : Rpiplc.GPIOs('RPIPLC_21','V4');
END_VAR
```
The model of RPIPLC and version must be declared in a string.
### Communication
The GPIOs object has some methods:
* **GPIOs.read_analog(STRING)** Requires a STRING with a GPIO (ex: 'I0.5'). Returns a UINT.
* **GPIOs.read_analog(STRING)** Requires a STRING with a GPIO (ex: 'I0.1'). Returns a UINT.
* **GPIOs.write_analog(STRING, UINT)** Requires a STRING with a GPIO (ex: 'A0.6') and a value UINT to be written. Returns a STRING with the console feedback.
* **GPIOs.write_digital(STRING, BOOL)** Requires a STRING with a GPIO (ex: 'Q0.2') and a value BOOL to be written. Returns a STRING with the console feedback.
* **GPIOs.write_relay(STRING, BOOL)** Requires a STRING with a GPIO (ex: 'R0.1') and a value UINT to be written. Returns a STRING with the console feedback.
### Example

```
PROGRAM PLC_PRG
VAR
rpiplc : Rpiplc.GPIOs('RPIPLC_21','V4');
pin : STRING := 'Q0.1';
i : BOOL := false;
END_VAR
```
```
rpiplc.write_digital(pin,i);
i := NOT(i);
```


# Ladder
The GPIOs of the PLC can be interacted by using the Ladder blocks "EXECUTE".

Drag the Execution block at the ladder diagram and write in Structured Text with any of the previously described functions or methods.

**Caution**: The 12c_init function must be always called as well in the first code execution when using the "old" method. A simple way to implement this behavor is having a BOOL variable activating an Execute block with the 12c_init() and setting said variable to FALSE.
