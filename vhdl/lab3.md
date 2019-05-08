

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
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;

entity ice_alu is
	generic(width : positive := 16);
	port(a, b : in std_logic_vector(width-1 downto 0);
		 y : out std_logic_vector(width-1 downto 0);
		 opc : in std_logic_vector(3 downto 0);
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
				-- NOT ???
            -- when "0111"=>
            --   y <= a NOT b;
				 when others =>
					 y <= "0000000000000000";
        end case;
	end process;
end behaviour;
```
### ALU Testbench
```vhdl
LIBRARY ieee;
USE ieee.std_logic_1164.ALL;
 
-- Uncomment the following library declaration if using
-- arithmetic functions with Signed or Unsigned values
--USE ieee.numeric_std.ALL;
 
ENTITY tb_ice_alu IS
END tb_ice_alu;
 
ARCHITECTURE behavior OF tb_ice_alu IS 
 
    -- Component Declaration for the Unit Under Test (UUT)
 
    COMPONENT ice_alu
	 generic(width : positive := 16);
    PORT(
         a : IN  std_logic_vector(width-1 downto 0);
         b : IN  std_logic_vector(width-1 downto 0);
         y : OUT  std_logic_vector(width-1 downto 0);
         opc : IN  std_logic_vector(3 downto 0);
         anl : IN  std_logic
        );
    END COMPONENT;
    
   --Inputs
   signal a : std_logic_vector(15 downto 0) := (others => '0');
   signal b : std_logic_vector(15 downto 0) := (others => '0');
   signal opc : std_logic_vector(3 downto 0) := (others => '0');
   signal anl : std_logic := '0';

 	--Outputs
   signal y : std_logic_vector(15 downto 0);
   -- No clocks detected in port list. Replace <clock> below with 
   -- appropriate port name 
 
   constant clk : time := 100 ns;
 
BEGIN
 
	-- Instantiate the Unit Under Test (UUT)
   uut: ice_alu PORT MAP (
          a => a,
          b => b,
          y => y,
          opc => opc,
          anl => anl
        );

	-- Stimulus process
	
   stim_proc: process
   begin
		-- AND
		wait for clk;	
		opc <= "0000";
		A <= "0000000000000001";
		B <= "0000000000000010";
		wait for clk;
		
		-- OR
		opc <="0001";
		A <= "0000000000000001";
		B <= "0000000000000010";
		wait for clk;
		
		-- NAND
		opc <="0010";
		A <= "0000000000000001";
		B <= "0000000000000010";
		wait for clk;
		
		-- NOR
		opc <="0011";
		A <= "0000000000000001";
		B <= "0000000000000010";
		wait for clk;
		
		-- XOR
		opc <="0100";
		A <= "0000000000000001";
		B <= "0000000000000010";
		wait for clk;
		
		-- XNOR
		opc <="0101";
		A <= "0000000000000001";
		B <= "0000000000000010";
		wait for clk;
		
		-- NOT
		opc <="0111";
		A <= "0000000000000001";
		B <= "0000000000000010";
		wait for clk;
		
   end process;

END;
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

entity ice_shifter is
	port(a : in std_logic_vector(15 downto 0);
		 pos : in std_logic_vector(3 downto 0);
		 y : out std_logic_vector(15 downto 0);
		 opc : in std_logic_vector(3 downto 0));
end entity;

architecture behaviour of ice_shifter is
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
