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