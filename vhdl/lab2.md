# Lab 2

## Aufgabe 1
```vhdl
entity quickAndDirty is
generic( width : positive := 16);
	port( S1, S2, S3 : in bit;
		  S7, S8     : out bit);
		  S4 : in bit_vector (width-1 downto 0);
		  S5, S6 : out bit_vector (width-1 downto 0));
end quickAndDirty;
```

## Aufgabe 2
### 2.1 XOR Entity
```vhdl
entity neq4 is
	port( A, B : in bit_vector(3 downto 0);
		  Y : out bit_vector(3 downto 0));
end neq4;
```
### 2.2 XOR Behaviour
```vhdl
architecture behaviour of neq4 is
	begin
	Bxor : process (A, B)
		begin
			if (A = '1' and B = '0') or (A = '0' and B = '1')#
   		then
     		C <= '1';
		else
     		C <= '0';
		end if;
	end process;
end behaviour;
```
### 2.3 XOR Dataflow
```vhdl
architecture dataflow of neq4 is
begin
	Y <= (A AND (NOT B) ) or (B AND (NOT A));
end dataflow;
```
### 2.4 XOR Logic
```vhdl
architecture logic of neq4 is
begin
	Y <= A xor B;
end logic;
```
### 2.5 XOR Structural
```vhdl
entity _xor is
	Port (A : in bit;
		  B : in bit;
		  Y : out bit);
end _xor;

architecture structural of _xor is

component _and
	Port (A : in bit;
		  B : in bit;
		  Y : out bit);
end component;

component _nand
	Port (A : in bit;
		  B : in bit;
		  Y : out bit);
end component;

component _or
	Port (A : in bit;
	      B : in bit;
	      Y : out bit);
end component;

signal S1, S2, Y: bit;

begin

U1: _nand port map(a => A, b => B, y => S1);
U2: _or port map(a => A, b => B, y => S2);
U3: _and port map(a => S1, b=> S2, y => Y);

end structural;
```
### XOR Testbench
```vhdl
entity _xor is
	Port (A : in bit;
		  B : in bit;
		  Y : out bit);
end _xor;

architecture structural of _xor is

component _nand
	Port (A : in bit;
		  Y : out bit);
end component;

component _and
	Port (A : in bit;
		  B : in bit;
		  Y : out bit);
end component;

component _or
	Port (A : in bit;
	      B : in bit;
	      Y : out bit);
end component;

signal S1, S2, S3, S4: bit;

begin

U1: _not port map(A, S1);
U2: _not port map(B, S2);
U3: _and port map(S1, B, S3);
U4: _and port map(A, S2, S4);
U5: _or port map(S3, S4, Y);

end structural;
```
### 4bit Xor
```vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;

entity top_xor_4bit is
	Port (A : in std_logic_vector(3 downto 0);
		  B : in std_logic_vector(3 downto 0);
		  Y : out std_logic);
end top_xor_4bit;

architecture structural of top_xor_4bit is

component comp_xor
	Port (a : in std_logic;
		   b : in std_logic;
		   y : out std_logic);
end component;

component comp_or_4bit
	Port (a : in std_logic;
	      b : in std_logic;
			c : in std_logic;
	      d : in std_logic;
	      y : out std_logic);
end component;

signal S1 : std_logic;
signal S2 : std_logic;
signal S3 : std_logic;
signal S4 : std_logic;

begin

U1: comp_xor port map(a => A(0), b => B(0), y => S1);
U2: comp_xor port map(a => A(1), b => B(1), y => S2);
U3: comp_xor port map(a => A(2), b => B(2), y => S3);
U4: comp_xor port map(a => A(3), b => B(3), y => S4);
U5: comp_or_4bit port map(a => S1, b => S2, c => S3, d => S4, y => Y);

end structural;
```
```vhdl
-- 4 Bit OR Component
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;

entity comp_or_4bit is
	port (a, b, c, d : in std_logic;
			y : out std_logic);
end comp_or_4bit;

architecture Behavioral of comp_or_4bit is
begin

	y <= (a OR b) OR (c OR d);
	
end Behavioral;
```
### 4 Bit XOR Testbench
```vhdl
USE ieee.std_logic_1164.ALL;

ENTITY tb_lab_2_8 IS
END tb_lab_2_8;
 
ARCHITECTURE behavior OF tb_lab_2_8 IS 
 
    -- Component Declaration for the Unit Under Test (UUT)
 
    COMPONENT top_xor_4bit
    PORT(
         A : IN  std_logic_vector(3 downto 0);
         B : IN  std_logic_vector(3 downto 0);
         Y : OUT  std_logic
        );
    END COMPONENT;
    
   --Inputs
   signal A : std_logic_vector(3 downto 0) := "0000";
   signal B : std_logic_vector(3 downto 0) := "0000";

 	--Outputs
   signal Y : std_logic;
   -- No clocks detected in port list. Replace <clock> below with 
   -- appropriate port name 
 
   constant clk : time := 100 ns;
 
BEGIN
 
	-- Instantiate the Unit Under Test (UUT)
   uut: top_xor_4bit PORT MAP (
          A => A,
          B => B,
          Y => Y
        );

   -- Stimulus process
   stim_proc: process
   begin		
      -- hold reset state for 100 ns.
      wait for 100 ns;	
		A <= "0001";
		B <= "0000";
		wait for clk;
		A <= "0000";
		B <= "0001";
		
		wait for clk;
		A <= "0001";
		B <= "0001";
		
      wait for clk;
		B <= "0100";

      wait for clk;
   end process;
END;
```
