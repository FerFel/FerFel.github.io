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
end behaviour;
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

	process(imm, grp, opc)
	begin
	
		if ((imm = '1') and (grp = "000"))
		then
			pc_inc <= '1';
			pc_load <= '1';
		else
			pc_inc <= '0';
			pc_load <= '1';
		end if;
		
	end process;
end architecture;
```