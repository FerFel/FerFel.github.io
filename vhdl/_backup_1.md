Lab 3
----

## Aufgabe 1 - Entwurf einer einfachen ALU
Die Alu besteht aus zwei Teilschaltugen, **logischem** und **arithmetischem** Teil.
Beide Teilschaltungen werten den opCode **opc** aus und verknüpfen dementsprechend die signale **a** und **b**.
Ob das arithmetische oder logische Ergebnis an den Ausgang **y** geleitet wird, hängt von **anl** ab.
|operation|opcode|
|---|---|---|
|AND|0000|
|OR|0001|
|NAND|0010|
|NOR|0011|
|XOR|0100|
|XNOR|0101|
|NOT|0111|

```vhdl
entity ice_alu is
	generic(width : positive := 16);
	port(a, b : in std_logic_vector(width-1 downto 0)
		 y : out std_logic_vector(width-1 downto 0)
		 opc : in std_logic_vector(3 downto 0)
		 anl : in std_logic);
end entity;

architecture behaviour of ice_alu is

begin

	process(a, b, opc)
	
	begin
        case opc is
            when "0000" => 
                y <= a AND b;
            when "0001" =>
                y <= a OR b;
            when "0010" =>
                y <= a NAND b;
            when "0011" =>
                y <= a NOR b;
            when "0100" =>
                y <= a XOR b;
            when "0101" =>
                y <= a XNOR b;
            when "0111"=>
                y <= a NOT b;
        end case;
	end process;
end behaviour;
```
## Aufgabe 2 - Barrel-Shifter
- Shift functions are found in numeric_std package file
- Shift functions can perform both logical (zero-fill) and arithmetic (keep sign) shifts
- Type of shift depends on input to function. Unsigned=Logical, Signed=Arithmetic

|operation|opcode|beschreibung|
|----|----|----|----|
|LSL|0000|logischer Links-Shift|
|LSR|0001|logischer Rechts-Shift|
|ASR|0010|arithmetischer Rechts-Shift|
|ROL|0011|Links-Rollen|
|ROR|0100|Rechts-Rollen|
```vhdl
library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;   

component ice_shifter
	port(a : in std_logic_vector(15 downto 0);
		 pos : in std_logic_vector(3 downto 0);
		 y : out std_logic_vector(15 downto 0);
		 opc : in std_logic_vector(3 downto 0));
end component;

architecture behaviour of ice shifter is
begin
	process(a, b, opc, pos)
	begin
        case opc is
            when "0000" => 
                y <= shift_left(unsigned(a), pos);
            when "0001" =>
                y <= shift_right(unsigned(a), pos);
        end case;
	end process;
end behaviour;
```

Lab 4
----
## Aufgabe 1 - Adressdekoder mit Enable
- Im Falle des ICE-Prozessors ist ein 3-Port-Speicher vorgesehen, der es erlaubt zu einem Zeitpunkt auf beliebige Register lesend (Read Ports) und auf ein Register schreibend zuzugreifen.
- Die Auswahl eines lesenden Zugriffs erfolgt einfach über einen großen Multiplexer, der in Abhängigkeit von der Leseadresse das entsprechende Quellregister aus den Leseport durchschaltet.

1. Aktivierung einer Leitung **sel** erfolung nur wenn **en** aktiv
2. wenn **en** inaktiv sind alle bits von **sel** = **0**
3. Register mit Adresse **addr** wird beschrieben durch ein bit von **sel** = **1**

```vhdl
entity ice_rfdecode is
	port(addr : in std_logic_vector(3 downto 0);
		 en : in std_logic;
		 sel : out std_logic_vector(15 downto 0));
end entity;

architecture behaviour of ice_rfdecode is
begin
	
	process(addr, en, sel)
	begin
        if(en = '1')
        then 
        	sel(addr) <= '1';
       	else
       		sel <= "0000000000000000";
	end process;
```
## Aufgabe 2 - Befehlsdekoder
Der ICE-Prozessor ist ein einfacher Single-Cycle-Prozessor, d. h. die typischen Phasen einer Instruktionsverarbeitung finden in einem Taktzyklus statt:
1. **Instruction Fetch**
2. **Instruction Decode**
3. **Execute**
4. **Write Back**

- Befehlsdekoder implementiert sodass er nur **einen** Befehl  = **JUMP** dekodieren und die Steuerleitungen des Befehlszählers(lab5) entsprechend setzten kann
- ICE-CPU verwendet Instruktionswort fester Breite **3 Byte = 24 Bit**

#### Allgemeines Instruktionswort 

|I|G-Code|I-Code|Befehlsformat spezifisch|
|:----:|:----:|:----:|:----:|
|b23|b22 b21 b20|b19 b18 b17 b16|b15 b14 b13 b12 b11 b10 b9 b8|b7 b6 b5 b4 b3 b2 b1 b0|

- Immediate Bit (b23)
- Befehlsgruppe (b22 bis b20 **G-Code**)
- Instruktionen (b19 bis b16 **I-Code**)

#### Befehlsformat I für JUMP
|I|G-Code|I-Code|Register-ID|Immediate|
|:-:|:----:|:----:|:----:|:----:|:----:|
|1|0 0 0|- - - -|d3 d2 d1 d0|i11 i10 i9 i8|i7 i6 i5 i4 i3 i2 i1 i0|

- Befehlsformat I bezieht sich auf Befehlsgruppe 0 und einige Memory-Instructions
- In den untersten 12 Bit ist die absolute Zielsprungadresse kodiert (i11 bis i0)
- Quellregister-ID ist in den Bits b15 bits b12 spezifizierst (für bedingte Sprünge), der **JUMP**-Befehl ignoriert dieses Feld
- Das **Immediate**-Bit muss beim Befehlsformat I gesetzt sein, da innerhalb der Memory-Befehlsgruppe eine Unterscheidung zwischen absoluten und Registern indirekten Speicheroperationen durchgeführt wird

|beschreibung|port|richtung|breite|
|----|----|----|----|
|Befehlt enthält immediate Wert|imm|in|1|
|Instruktionsgruppencode|grp|in|2 downto 0|
|Operationscode|opc|in|3 downto 0|
|Programmzähler aktivieren|pc_inc|out|1|
|Programmzähler laden|pc_load|out|1|

```vhdl
-- Entity Declaration des Befehlsdecoders
entity ice_idecode is
	port(imm : in std_logic;
		 grp : in std_logic_vector(2 downto 0);
		 opc : in std_logic_vector(3 downto 0);
		 pc_inc, pc_load : out std_logic);
end entity;
```
```vhdl
-- Architecture Befehlsdecoder
architecture behaviour of ice_idecode is
begin
	process( )
		if (imm = '1') and (grp = "000") and (I-CODE??);
		then
			pc_inc = '1';
			pc_load = '1';
		else
			pc_inc = '0';
			pc_load = '1';
	end process;
end behaviour;
```
Lab 5
----
## Aufgabe 1 - Register mit Enable
- Register mit 16 Bit Standartbreite aber variable
- positiv flankengetriggert
- asynchromenem Rücksetzteingang (active high)
#### Register Aufbau Entity
|Beschreibung|Port|Richtung|Breite|
|----|----|----|----|----|
|Takt|clk|in|1|
|Reset|reset|in|1|
|D|d|in|width-1 downto 0|
|Q|q|out|width-1 downto 0|
|Enable|en|in|1|

#### Rising Edge Function
- function **rising_edge** ( signal s : std_ulogic ) return boolean;
- Detects the rising edge of a **std_ulogic** or **std_logic** signal. It will return true when the signal changes from a low value ('0' or 'L') to a high value ('1' or 'H').

```vhdl
-- Entity Delcaration Register mit Enable
entity ice_reg is
	generic(width : positive := 16);
	port(clk, reset, en : in std_logic;
		 d : in std_logic_vector(width-1 downto 0);
		 q : out std_logic_vector(width-1 downto 0));
end entity;
```
```vhdl
-- Architecture Register mit Enable
architecture behaviour of ice_reg is 
begin
	process(clk, reset)
	begin
		if reset = '1' then
			q <= x"0000000000000000";
		elsif rising_edge(clk) then
        	if ld = '1' then
        		q <= d;
        	end if;
        end if;
    end process;
end behaviour;
```
## Aufgabe 2 - Befehlszähler
- Befehlszähler (program counter, pc, instruction pointer, ip)
- enthält je nach Architektur die Speicheradresse des derzeitigen bzw. nächsten Befehls
- Im ICE-Prozessor soll er 12 Bit Adressen generieren die am Ausgang **pc** stehen
- wenn **inc** gesetzt ist wird entweder die inkementierte Adresse bzw. das Sprungziel übernommen 
- soll ein Sprungziel übernommen werden muss zusätzlich zu **inc** auch **load** aktiv sein
#### Aufbau instruction pointer entity
|Beschreibung|Port|Richtung|Breite|
|----|----|----|----|
|Takt|clk|in|1|
|Reset|reset|in|1|
|Adresse inc bzw laden|inc|in|1|
|Sprungadresse laden|load|in|1|
|Sprungadresse|imm|in|11 downto 0|
|aktuelle Befehlsadresse|pc|out|11 downto 0|
```vhdl
-- Entity Declaration Instruction Pointer
entity ice_pc is
	port(clk, reset, inc, load : in std_logic;
		 imm : in std_logic_vector(11 downto 0);
		 pc : out std_logic_vector(11 downto 0));
end entity;
```
```vhdl
-- Architecture Instruction Pointer
architecture behaviour of ice_pc is
begin
	process(clk, reset)
	begin
		-- reset pc auf zero
		if(reset = '1') then
			pc <= "0000000000000000";
		if rising_edge(clk) then
			-- nächsten Befehl laden
			if(inc = '1') and (load = '1') then
				pc <= imm;
			-- addresse inkrementieren
			elsif(inc = '1') then
            	pc <= std_logic_vector(pc+1)
            end if;
     end process;
end behaviour;
```
lab 6
----


​			