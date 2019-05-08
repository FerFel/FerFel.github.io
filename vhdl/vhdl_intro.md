# First Look at VHDL

### A Comparator
Accepts two 8-bit inputs, compares them, and produces a 1-bit result(1 for match, 0 for  difference).

```vhdl
-- Eight-bit comparator
entity compare is
	port(A, B: in bit_vector(0 to 7);
		 Y: out bit);
end compare;

architecture compare1 of compare is
begin

	EQ <= '1' when (A=B) else '0';

end compare1;
```

### Entities and Architectures
Every VHDL design consists of at least one entity/architecture pair. This combination is sometimes reffered to as **design entity**. 
An **entity declaration** describes the circuit as it appears from the *outside*, analogous to a symbol on a schematic.
The **architecture declaration** describes the actual function, analogous to a lower-level schematic than the entity. Its name has to be unique, followed by the name of the entity it is bound to.

### Data Types
Every Data Type in VHDL hast a defined set of values and a defines set of valid operations.

|Data Type | Values | Example |
|----|----|----|----|
|bit | '1', '0' | Q <= '1'|
|bit_vector| (array of bits) | DataOut <="00100011";|
|Booleon|True, False|EQ <= True; |
|Integer | -2, -1, 0, 1, 2 | Count <= Count + 2|
|Real | 1.0, -1.0E5 |V1=V2/5.3|
|Time|1 ua, 7 ns, 100 ps| Q <= '1' after 6 ns;|
|Character| 'a', 'b', '2', '$'; |CharData <= 'X'|
|String| (Array of Bits) |Msg <= "MEM: " & Addr|

### Design Units
Design Units in VHDL are segments of VHDL code that can be compiled separately and stored in a library. There are five types of design units:
1. entities
2. architectures
3. packages
4. package bodies
5. configurations
Entities and Architecture are the only two design units that you must have in VHDL design description, packages and configurations are optional.

### Entites
Entites define the external specification of a circuit or subcircuit. Entities must be provided with  **unique name** and a **port list**, defining the input and output ports. Each port must be given a **name**, **direction**, and **type**. Optionally a special type of paramater list, **generic list**, may be included.
```vhdl
entity fulladder is
	port (X: in bit;
		  Y: in bit;
		  Cin: in bit;
		  Cout: out bit;
		  Sum: out bit);
end fulladder;
```
### Architectures
Architectures describe the underlying function and/or structure of the circuit. Each architecture must be bound by name with one entity in the design.
VHDL allows you to create **more than one** alternate architecture for each entity.
This feature is useful for simulatio and team project environments.
An architecture declaration consists of zero or more declarations of items such as **intermediate signals**, **components** that will be referenced in the architecture, **local functions** and **constants**, followed by a **begin** statement, a series of concurrent statements, and an **end** statement.
```vhdl
architecture concurrent of fulladder is 
begin
	Sum <= X xor Y xor Cin;
	Cout <= (X and Y) or (X and Cin) or (Y and Cin);
end concurrent;
```
### Packages and Package Bodies
A VDHL package declaration is identified by the **package** keyword, and is used to collect commonly-used declarations for use globally among different design units. You can think of a package as a comon storage area, one used to store such things as **type declarations**, **constants**, **global subprograms**. Items defines within a package can be made visible to any other design unit in the complete VHDL design, and they can be compiled into libraries for late re-use.
A package consists of two basic parts: a **package declaration** and an optional **package body**. The relationship between a package and package body is somewhat akin to the relationship between an entity and its corresponding architecture.
```vhdl
package conversion is 
	function to_vector(size: integer; num:integer) return std_logic_vector;
end conversion;
	
package body conversion is
	function to_vector(size: integer; num: interger) return std_logic_vector is 
		variable ret: std_logic_vector(1 to size);
		variable a: integer;
	begin
		a := num;
		for i in size downto 1 loop
			if ((a mod 2) =1) then
				ret(i) := '1';
			else
				ret(i) :='0';
			end if;
			a := a / 2;
		end loop;
		return ret;
	end to_vector;
end conversion;
```
### Configurations
Configurations Declarations are roughly analogous to a parts list for your design. It specifies which architectures are to be bound to which entities, and it allows you to change how components are connected in your design description at the time of simulation. Configuration declarations are always optional.
```vhdl
configuration this_build of rcomp is
	for structure
		for COMP1: compare use entity work.compare(compare1);
		for ROT1: rotate use untity work.rotate(rotate1);
	end for;
end this_build;
```

### Levels of Abstraction (Styles)
#### Behavior 
This is the highest level of abstraciton. It describes the circuit in its **operation over time**.
The concept of time is the critical distinction between behavioral descriptions of circuits and lower-level descriptions. Examples include *state diagrams*, *timing diagrams* and *algorithmic descriptions*.
In behavrioral description the concept of time may be expressed precisely, with actual delays between related events (such as propagation delays between gates and wires), or such by sequentially descripted operations.

#### Dataflow
Describes the circuit in terms of how data moves through the system, that is, how information is passed between registers in the circuit. Dataflow level is often called **register transfer logic** (**RTL**). So the Dataflow Level is all about connectionc registers. VDHL has no build in registers, when using dataflow they must first be created (or obtained), either as components of in the form of subprograms (functions/procedures).

#### Structure
The Structure Level of abstraction can be used to describe very low-level descriptions of a circuit (transistor-level) or a very high-level description (block diagram).
In a gate-level description components such as logic gates and flip-flops might be connected in some logical structure to create the circuit. This is often called a **netlist**. For a higher-level circuit, components might just be connected to segment the design in manageable parts.
Structure-level VHDL is very useful for managing complexity (=> **Top-down** approach).

### Signal Assignments
```vhdl
-- CONDITIONAL SIGNAL ASSIGNMENT
architecture mux1 of mux is
begin 
	Y <= A when (Sel = "00") else
		 B when (Sel = "01") else
		 C when (Sel = "10") else
		 D when (Sel = "11");
end mux1;
```
```vhdl
-- SELECTED SIGNAL ASSIGNMENT
architecture mux1 of mux is
begin 
	Y <= A when "00",
		 B when "01",
		 C when "10",
		 D when "11";
end mux1;
```
### Using a Process
The process statement in VHDL is the primary means by which **sequential operations** can be described. 
```vhdl
architecture arch_name of ent_name is
begin
	process_name: process(sensitivity_list)
		local_declaration;
		local_declaration;
		...
	begin
		sequential statement;
		sequential statement;
		sequential statement;
		...
	end process;
end arch_name;
```
A process statement consists of:
- optional process name (identifier followed by colon character)
- the **process** keyword
- optional sensitivity list, indicating which signals result in the process being executed when there is some event detected (is required if there is no **wait** statement)
- optional declarations section, local objects and subprograms
- **begin** keyword
- sequence of statements when the process runs
- **end process** statement

### Generics
Generics can pass instance-specific information other than actual port connections to an entity. Generics are very useful for making design units more general-ourpose or for annotating information.
```vhdl
entity RAM is
generic(
 data_width : integer := 64; -- default value
 addr_width : integer := 12; -- default value
 Taa : time := 50; 
 Toe : time := 40
 ); 
port(
 oeb, wrb, csb : in std_logic;
 data : inout std_logic_vector(data_width-1 downto 0);
 addr : in std_logic_vector(addr_width-1 downto 0)
 );
end RAM;
```
```vhdl
RAM1 : RAM 
generic map(
 data_width => 32,
 addr_width => 20,
 Taa => 30 ns,
 Toe => 35 ns)
port map(
 oeb => oeb, wrb => wrb, csb => csb,
 data => data ,
 addr => addr );
```
 ```vhdl 
 RAM2 : RAM -- using the default values
port map(
 oeb => oeb, wrb => wrb, csb => csb,
 data => data ,
 addr => addr );
 ```


