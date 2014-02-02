![fbd logo](https://dl.dropboxusercontent.com/u/46913329/fbd2/images/fbd2.png)
fbd-runtime
===========

Run-time FBD library for PLC (*Programmable Logic Controller*).

About FBD: http://en.wikipedia.org/wiki/Function_block_diagram.
FBD - one of the programming languages described in the international standard IEC 61131-3. This implementation is not fully compatible with the standard, although many of its requirements are implemented.

FBD editor and simulator can be downloaded at https://dl.dropboxusercontent.com/u/46913329/fbd2/fbd2setup.exe

-------------------------------------------------------------------------------------

Appointment
===========

Language functional diagrams designed to describe the functioning of the single logic controller or multiple controllers connected information network. Language describes a scheme consisting of the elements and their binding chains.

Basic Concepts
--------------

The scheme is a set of elements and relationships between them.

* Circuit element is the minimum functional circuit block that implements a specific operation (logic, arithmetic, delay, etc.) or provide connectivity scheme with input or output hardware controller circuits. Each element may be greater than or equal to zero the number of signal inputs and zero or one output signal. Inputs are numbered element - a number from 0 to N-1 , where N - number of inputs. Number of inputs and outputs depends on the type of item. For each input item, usually must be connected an external circuit. Inputs of some elements may have value "default". Value of the input to which no external circuit and there is no "default" is undefined, and the circuit in which there is an element, incorrect. Furthermore, the element may be greater than or equal to zero quantity affecting its functioning named parameters. Fixed values and are given in the design scheme. Each element must have a unique (within the scheme ), a name that will identify it. Elements that do not contain the output, usually serve to anchor chains scheme to hardware outputs of the controller or the formation of values, accessible through the data interface controller. Binding is defined by the parameters of the element. Elements that do not contain entries to serve as a source of fixed or alternating signal, whose value is given by a parameter. In addition, setting the value of the signal values of pixels may be accessed via an information controller interface.

* Chain is a logical connection between input and output elements and serves to transmit the signal states . Each circuit must be connected strictly one output element. State of the chain which is not connected to any output or connected more than one output of several elements is undefined , and a scheme in which to make such connections , incorrect. Chain, which is not connected to any input element is unused and its value is not calculated. Each chain has a unique (within the scheme) name. Chain with the same name are one chain.

* Chains are carriers of signals. The signal is expressed in a language in the form of integer values with a sign. In general, the signal is used to represent a single-byte, double-byte or four-byte signed integer. Due to the nature of hardware controllers can be used at the bit number. To perform logical operations, the logical "0" is defined as the signal value `==0`, a logical "1" - signal value `!=1`. Furthermore, the modifications of the signal value of the logical state "0" to logic "1" (rising edge), and from a logical "1" to logical "0" (a falling edge).

Example of a simple scheme is shown below:
![fbd demo](https://dl.dropboxusercontent.com/u/46913329/fbd2/images/fbddemo.png)

Key features
------------
* Small RAM memory requirements: scheme consists of 400 elements using only about 1 kb.
* The algorithm is optimized for use in embedded controllers: PIC, AVR, ARM etc. Calculating does not use recursion, built data stack is used very sparingly.
* Wide range of supported elements: logic, arithmetic, comparison, timers, triggers, PID. Set of elements can be easily expanded.
* Library has no binding to a specific hardware and can be easily ported to any platform for which there is a C compiler.
* Support for storing intermediate results in EEPROM (for flip-flop, timers etc.), reading and saving values of network variables.

![fbd struct](https://dl.dropboxusercontent.com/u/46913329/fbd2/images/struct.png)

Limits
------
Calculation algorithm imposes certain restrictions on the possible options scheme.

* Element can not have more than one output pin.
* The scheme supports only one data type (`tSignal`). This applies to the inputs and outputs of the elements, values of the constants. The data type is determined at compile the project. Usually it is signed integer.

Algorithm for calculating the state of the circuit
--------------------------------------------------
Calculation of the state controller scheme executed cyclically with a certain period.
Each cycle calculation circuit state is to calculate output signals to the output elements which are not connected to the circuit.
Payment scheme in the following sequence:
  * for all circuits schemes establish a criterion of "no data";
  * scheme of all content elements to the output circuit which is not connected;
  * calculate the output value of the element:
  * Calculated value input element: if the chain is set sign "unknown", the calculated output value is connected to the circuit element, otherwise already taken the calculated value chain;
  * if in the process of circuit design need her own value (feedback loop), then used for the calculation of its previous value;
  * the value chain which has already been calculated, a sign of "no data" is reset;
  * the operation is performed on the input signal element (depending on the type of item).
The operation is performed for all selected items.

Elements by functions
---------------------
### Input/Output/Const
#### Input pin (*internal code 0x0f*)
```
 | Pin >----
```
Element has one output, required for communication signal circuits to the input pin PLC.
#### Output pin (*internal code 0x00*)
```
 ----< Pin |
```
Element has one input, his does not perform the any calculations, required for communication signal circuits to the output terminal of the PLC.
Element has one output. The output value is always equal to a constant, determined at the time of schema design.
#### Input variable (*internal code 0x10*)
```
 | Var >----
```
Element has one output, required for communication signal circuits to the input variable PLC. Value can be obtained from the network. This is one way of remote control state circuits.
#### Output variable (*internal code 0x0e*)
```
 ----< Var |
```
Element has one input, his does not perform the any calculations, required for communication signal circuits to the output variable PLC. You might want to pass a value to the variable on the network.
#### Const value (*internal code 0x01*)
```
 | Const >----
```
### Logical
#### NOT (*internal code 0x02*)
```
    +---+
----|   O----
    +---+
```
Traditional, often used element NOT. Includes one input and one output. Truth table:
<table>
 <tr><td><b>Input</b></td><td><b>Output</b></td></tr>
 <tr><td>Logical "0"</td><td>1</td></tr>
 <tr><td>Logical "1"</td><td>0</td></tr>
</table>
#### AND (*internal code 0x03*)
```
    +-----+
----|  &  |----
----|     |
    +-----+
```
Element has two inputs and one output. Truth table:
<table>
 <tr><td><b>Input 1</b></td><td><b>Input 2</b></td><td><b>Output</b></td></tr>
 <tr><td>Logical "0"</td><td>Any value</td><td>0</td></tr>
 <tr><td>Any value</td><td>Logical "0"</td><td>0</td></tr>
 <tr><td>Logical "1"</td><td>Logical "1"</td><td>1</td></tr>
</table>
#### OR (*internal code 0x04*)
```
    +-----+
----|  1  |----
----|     |
    +-----+
```
Element has two inputs and one output. Truth table:
<table>
 <tr><td><b>Input 1</b></td><td><b>Input 2</b></td><td><b>Output</b></td></tr>
 <tr><td>Logical "0"</td><td>Logical "0"</td><td>0</td></tr>
 <tr><td>Logical "1"</td><td>Any value</td><td>1</td></tr>
 <tr><td>Any value</td><td>Logical "1"</td><td>1</td></tr>
</table>
#### XOR (*internal code 0x05*)
```
    +-----+
----| =1  |----
----|     |
    +-----+
```
XOR element has two inputs and one output. Truth table:
<table>
 <tr><td><b>Input 1</b></td><td><b>Input 2</b></td><td><b>Output</b></td></tr>
 <tr><td>Logical "0"</td><td>Logical "0"</td><td>0</td></tr>
 <tr><td>Logical "0"</td><td>Logical "1"</td><td>1</td></tr>
 <tr><td>Logical "1"</td><td>Logical "0"</td><td>1</td></tr>
 <tr><td>Logical "1"</td><td>Logical "1"</td><td>0</td></tr>
</table>
### Flip-flop
#### SR latch (*internal code 0x06*)
```
    +--+----+
----|R |   Q|----
    |  |    |
----|S |    |
    +--+----+
```
SR-latch has two inputs (S and R) and one output (Q).
While the S and R inputs are both low, feedback maintains the Q output in a constant state. If S (Set) is pulsed high while R (Reset) is held low, then the Q output is forced high, and stays high when S returns to low; similarly, if R is pulsed high while S is held low, then the Q output is forced low, and stays low when R returns to low.
Truth table:
<table>
 <tr><td><b>S</b></td><td><b>R</b></td><td><b>Qnext</b></td><td><b>Action</b></td></tr>
 <tr><td>0</td><td>0</td><td>Q</td><td>Hold state</td></tr>
 <tr><td>0</td><td>1</td><td>0</td><td>Reset</td></tr>
 <tr><td>1</td><td>0</td><td>1</td><td>Set</td></tr>
 <tr><td>1</td><td>1</td><td>?</td><td>Not allowed</td></tr>
</table>
At each change its latch value is stored in eeprom (function is called `FBDsetProc(2, index, *value)`). At initialization scheme attempts to restore its value (the function is called `FBDgetProc(2, index)`).
#### D flip-flop (*internal code 0x07*)
```
    +--+----+
----|D |   Q|----
    |  |    |
----/C |    |
    +--+----+
```
D flip-flop has two inputs (D and C) and one output (Q).
The D flip-flop captures the value of the D-input at a definite portion of the clock cycle (such as the rising edge of the clock). That captured value becomes the Q output. At other times, the output Q does not change.
Truth table:
<table>
 <tr><td><b>D</b></td><td><b>C</b></td><td><b>Qnext</b></td><td><b>Action</b></td></tr>
 <tr><td>Any value</td><td>Rising edge</td><td>D</td><td>Saving</td></tr>
 <tr><td>Any value</td><td>Non-rising</td><td>Saved D value</td><td>Hold state</td></tr>
</table>
In difference from traditional D flip-flop, input (D) and output (Q) may be any value (not only "0" and "1").
At each change its flip-flop value is stored in eeprom (function is called `FBDsetProc(2, index, *value)`). At initialization scheme attempts to restore its value (the function is called `FBDgetProc(2, index)`).
#### Up-down counter (*internal code 0x13*)
```
    +--+----+
----/+ |   Q|----
----/- |    |
----|R |    |
    +--+----+
```
Element has three inputs and one output. If the input R high logic level, the output Q is reset to 0. If the input R is logic low, the output increases by 1 with a positive edge at the input "+" or decremented by 1 with a positive edge at the input "-". In the absence of a positive edge on the inputs "+" and "-" state of the output Q does not change. See table:
<table>
 <tr><td><b>+</b></td><td><b>-</b></td><td><b>R</b></td><td><b>Qnext</b></td><td><b>Action</b></td></tr>
 <tr><td>Any value</td><td>Any value</td><td>1</td><td>0</td><td>Reset</td></tr>
 <tr><td>Rising edge</td><td>Non-rising</td><td>0</td><td>Q+1</td><td>Increase</td></tr>
 <tr><td>Non-rising</td><td>Rising edge</td><td>0</td><td>Q-1</td><td>Decrease</td></tr>
 <tr><td>Non-rising</td><td>Non-rising</td><td>0</td><td>Q</td><td>Not change</td></tr>
 <tr><td>Rising edge</td><td>Rising edge</td><td>0</td><td>Q</td><td>Not change</td></tr>
</table>
### Arithmetical
#### Addition (*internal code 0x08*)
```
    +-----+
----|  +  |----
----|     |
    +-----+
```
Element has two inputs and one output. Output value is calculated as the arithmetic sum of the values of the input signals.
#### Subtraction (*internal code 0x09*)
```
    +-----+
----|  -  |----
----|     |
    +-----+
```
Element has two inputs and one output. Output value is calculated as the difference between the value of the signal at the first input signal and the value at the second input.
#### Multiplication (*internal code 0x0a*)
```
    +-----+
----|  *  |----
----|     |
    +-----+
```
Element has two inputs and one output. The output value - the first and second multiplication signal. Overflow values are not controlled.
#### Division (*internal code 0x0b*)
```
    +-----+
----|  /  |----
----|     |
    +-----+
```
Element has two inputs and one output. The output value - the value of the signal at the first input value divided by the signal on the second input. Special cases:
<table>
 <tr><td><b>Input 1</b></td><td><b>Input 2</b></td><td><b>Output</b></td></tr>
 <tr><td>0</td><td>0</td><td>1</td></tr>
 <tr><td>Any positive value</td><td>0</td><td>MAX_SIGNAL</td></tr>
 <tr><td>Any negative value</td><td>0</td><td>MIN_SIGNAL</td></tr>
</table>
I know that you can not divide by 0. :)
### Regulation
#### PID (*internal code 0x11*)
```
    +---+-----+
----|U  |    Q|----
----|Ref|     |
----|DT |     |
----|P  |     |
    +---+-----+
```
The calculation can not use the traditional PID algorithm. Instead, use the Euler method (anyway, that called him a good man, who told me about it). I heard somewhere that this algorithm is used for docking spacecraft. This algorithm has only two input parameters, quickly sets the output value and is not prone to fluctuations. Element has 4 inputs and one output. Inputs element are: U - current value controlled process, REF - reference value, DT - reaction time, P - proportionality factor. Features of the implementation can be found in the source code.
#### Integrator (*internal code 0x12*)
```
    +---+----+
----|X  |   Q|----
----|DT |    |
----|Lim|    |
    +---+----+
```
Element is used to integrate the input signal value. Can be used together with an element of PID. Element has three inputs: X - input value, DT - integrating constant, LIM - limiting the output value. Once a time DT calculated summa input signal value X with the previous value of the element. If the result is more LIM or less (-LIM), the output value will be truncated.
### Other
#### Timer (*internal code 0x0c*)
```
    +--+----+
----|D |   Q|----
    |  |    |
----|T |    |
    +--+----+
```
Timer - not exactly a traditional element. It can be used as a signal generator with a given period or time counter operation. At timer two inputs (D and T) and one output. If the input D is a logic 0, the output is always 0. If the input signal D logic 1 appears, then the output will be 1 in a time whose value is present at input T. Before to the expiration time T is stored at the output signal 0. See table:
<table>
 <tr><td><b>D</b></td><td><b>Сondition</b></td><td><b>Output</b></td></tr>
 <tr><td>Logical "0"</td><td>Any</td><td>0</td></tr>
 <tr><td>Logical "1"</td><td>Time T not expired</td><td>0</td></tr>
 <tr><td>Logical "1"</td><td>Time T expired</td><td>1</td></tr>
</table>
#### Comparator (*internal code 0x0d*)
```
    +-----+
----|  >  |----
----|     |
    +-----+
```
Element compares the signal at its two inputs. Truth table:
<table>
 <tr><td><b>Сondition</b></td><td><b>Output</b></td></tr>
 <tr><td>Input1Val &gt; Input2Val</td><td>1</td></tr>
 <tr><td>Input1Val &lt;= Input2Val</td><td>0</td></tr>
</table>
#### Multiplexer (*internal code 0x14*)
```
    +---+----+
----|D0 |   Q|----
----|D1 |    |
----|D2 |    |
----|D3 |    |
----|A  |    |
    +---+----+
```
Extensible multiplexer. Element has five inputs (D0-D3, A) and one output. Select one of 4 inputs depending on input A. Truth table:
<table>
 <tr><td><b>A</b></td><td><b>Output Q</b></td></tr>
 <tr><td>0</td><td>D0</td></tr>
 <tr><td>1</td><td>D1</td></tr>
 <tr><td>2</td><td>D2</td></tr>
 <tr><td>3</td><td>D3</td></tr>
</table>
Library setup
-------------
Setting is done by editing `fbdrt.h` in the following sequence.

1. Select the data type used to store the signal.
2. Select the data type used to store the index of the item.
3. If necessary, change the definition `ROM_CONST` and `DESCR_MEM`.
4. Write your implementation functions `FBDgetProc()` and `FBDsetProc()`.

On the choice of the type of data signal depends with what signals can work your scheme. In addition, it affects memory usage and speed of calculation. For embedded microcontrollers that can be important. In general, the data type must describe signed integer. If you are not sure what to choose, use a 2-byte integer. In addition, you must specify the maximum and minimum value of the signal. For example, you can use the definition from limits.h. For example:
```
// data type for FBD signal
typedef int16_t tSignal;
#define MAX_SIGNAL INT16_MAX
#define MIN_SIGNAL INT16_MIN
```

The data type of the element index affects how many elements can be in the scheme. In the general case should describe the type of unsigned integer. Type `unsigned char` will create a 255 elements, type `unsigned short` - 65535 elements. This is usually more than enough. Selected type affects the size of the memory used. Example:
```
// data type for element index
typedef unsigned char tElemIndex;
```
Definition `ROM_CONST` and `DESCR_MEM` describe specifiers that are used to allocate memory for an array of constants and circuit description. Their values depend on the compiler and the location of arrays. For example, when using compiler xc8 (Microchip) used to place data in a FLASH must specifiers `const`:
```
// data in ROM/FLASH
#define ROM_CONST const
// schema description
#define DESCR_MEM const
```
Functions `FBDgetProc()` and `FBDsetProc()` provide interaction between your circuit with real hardware PLC. Function `FBDgetProc()` used for reading input signals (pin), network variables or stored eeprom (nonvoltage memory) values. Function `FBDsetProc()` used for writing output signals (pin), network variables or eeprom values. Their implementation depends on the specific task. Encouraged to adhere to the following rules:
  * For discrete inputs and outputs use the values `0` and `1`.
  * For analogue inputs and outputs use the values expressed in engineering units, possibly with some decimal factor. For this, in some cases, the conversion function should perform PLC raw input data to engineering units and vice versa. For example the value of temperature sensor +10.5 C must be converted to a number 105.
  * Do not use a direct entry in the eeprom because Library calls `FBDsetProc()` each time you change the value of any trigger or timer. Direct writing to eeprom can reduce its life span. One solution is to use a delayed write or use of RAM with battery-powered.

In the absence of part of the PLC eeprom or network access these functions can not be implemented.
Example empty (only for debug) implementation of read and write functions:
```
tSignal FBDgetProc(char type, tSignal index)
{
    switch(type) {
    case 0:
        printf(" request InputPin(%d)\n", index);
        return 0;
    case 1:
        printf(" request Variable(%d)\n", index);
        return 0;
    case 2:
        printf(" request EEPROM(%d)\n", index);
        return 0;
    }
}

void FBDsetProc(char type, tSignal index, tSignal *value)
{
    switch(type)
    {
    case 0:
        printf(" set OutputPin(%d) to value %d\n", index, *value);
        break;
    case 1:
        printf(" set Variable(%d) to value %d\n", index, *value);
        break;
    case 2:
        printf(" set EEPROM(%d) to value %d\n", index, *value);
        break;
    }
}
```
Scheme description
------------------
Description of the scheme is an array of data that may be created by the editor or other means. Dataset should be placed in the memory controller, such as a FLASH or RAM. A pointer to the array is passed to a function `fbdInit(DESCR_MEM unsigned char *buf)`. The array consists of three parts:
  * Description of circuit elements (ending flag `END_MARK`)
  * Description of connecting the input pins elements
  * Parameters of elements

The format of the array shown in the table below:
<table>
 <tr><td><b>Offset</b></td><td><b>Size</b></td><td><b>Value</b></td><td><b>Desciption</b></td></tr>
 <tr><td>0</td><td>1</td><td>type of element 1</td><td>description of element 1</td></tr>
 <tr><td>1</td><td>1</td><td>type of element 2</td><td>description of element 2</td></tr>
 <tr><td></td><td>1</td><td>...</td><td>...</td></tr>
 <tr><td>N-1</td><td>1</td><td>type of element N</td><td>description of element N</td></tr>
 <tr><td>N</td><td>1</td><td>END_MARK</td><td>terminator elements descriptions</td></tr>
 <tr><td>N+1</td><td>sizeof(tElemIndex)</td><td>element index</td><td>index of the element connected to the input 1 element 1</td></tr>
 <tr><td></td><td>...</td><td>...</td><td>...</td></tr> 
 <tr><td></td><td>sizeof(tElemIndex)</td><td>element index</td><td>index of the element connected to the input K(1) element 1</td></tr>
 <tr><td></td><td>sizeof(tElemIndex)</td><td>element index</td><td>index of the element connected to the input 1 element 2</td></tr>
 <tr><td></td><td>...</td><td>...</td><td>...</td></tr> 
 <tr><td></td><td>sizeof(tElemIndex)</td><td>element index</td><td>index of the element connected to the input K(2) element 2</td></tr>
 <tr><td></td><td>...</td><td>...</td><td>...</td></tr> 
 <tr><td></td><td>sizeof(tElemIndex)</td><td>element index</td><td>index of the element connected to the input 1 element N</td></tr>
 <tr><td></td><td>sizeof(tElemIndex)</td><td>element index</td><td>index of the element connected to the input K(N) element N</td></tr>
 <tr><td></td><td>sizeof(tSignal)</td><td>parameter value</td><td>parameter 1 of the element 1</td></tr>
 <tr><td></td><td>sizeof(tSignal)</td><td>parameter value</td><td>parameter 2 of the element 1</td></tr>
 <tr><td></td><td>...</td><td>...</td><td>...</td></tr>
 <tr><td></td><td>sizeof(tSignal)</td><td>parameter value</td><td>parameter M(1) of the element 1</td></tr>
 <tr><td></td><td>...</td><td>...</td><td>...</td></tr>
 <tr><td></td><td>sizeof(tSignal)</td><td>parameter value</td><td>parameter 1 of the element N</td></tr>
 <tr><td></td><td>sizeof(tSignal)</td><td>parameter value</td><td>parameter 2 of the element N</td></tr>
 <tr><td></td><td>...</td><td>...</td><td>...</td></tr>
 <tr><td></td><td>sizeof(tSignal)</td><td>parameter value</td><td>parameter M(N) of the element N</td></tr>
</table>
Where: _N_ - elements count, _K(i)_ - number of inputs of an element __i__, _M(i)_ - number of parameters of an element __i__.

Number of inputs and number of parameters depend on the element type code. Summary table of types of elements below:
<table>
 <tr><td><b>Element</b></td><td><b>Type code</b></td><td><b>Inputs</b></td><td><b>Parameters</b></td></tr>
 <tr><td>...</td><td>0</td><td>0</td><td>0</td></tr>
 <tr><td>...</td><td>1</td><td>0</td><td>0</td></tr> 
 <tr><td>...</td><td>2</td><td>0</td><td>0</td></tr> 
 <tr><td>...</td><td>3</td><td>0</td><td>0</td></tr> 
 <tr><td>...</td><td>4</td><td>0</td><td>0</td></tr> 
 <tr><td>...</td><td>5</td><td>0</td><td>0</td></tr> 
 <tr><td>...</td><td>6</td><td>0</td><td>0</td></tr> 
 <tr><td>...</td><td>7</td><td>0</td><td>0</td></tr> 
 <tr><td>...</td><td>8</td><td>0</td><td>0</td></tr> 
 <tr><td>...</td><td>9</td><td>0</td><td>0</td></tr> 
 <tr><td>...</td><td>10</td><td>0</td><td>0</td></tr> 
 <tr><td>...</td><td>11</td><td>0</td><td>0</td></tr> 
 <tr><td>...</td><td>12</td><td>0</td><td>0</td></tr> 
 <tr><td>...</td><td>13</td><td>0</td><td>0</td></tr> 
 <tr><td>...</td><td>14</td><td>0</td><td>0</td></tr> 
 <tr><td>...</td><td>15</td><td>0</td><td>0</td></tr> 
 <tr><td>...</td><td>16</td><td>0</td><td>0</td></tr> 
 <tr><td>...</td><td>17</td><td>0</td><td>0</td></tr> 
 <tr><td>...</td><td>18</td><td>0</td><td>0</td></tr> 
 <tr><td>...</td><td>19</td><td>0</td><td>0</td></tr>
 <tr><td>...</td><td>20</td><td>0</td><td>0</td></tr>
</table>

Author
======

Alexey Lutovinin -- crossrw1@gmail.com
