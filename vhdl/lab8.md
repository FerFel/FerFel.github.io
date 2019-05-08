# lab 8
## Aufgabe 1 - Registerspeicher

- component **ice_rfdecode** (Name:dec)
- 16 Instanze der Komponente **ice_reg** mittels **generate**
- internen signal **s_rf** ist std_logic_vector(15 downto 0) 
- vhdl Realisierung Leseport-Multiplexer

```vhdl
entity ice_rf_gen is
	generic(width : positiv := 16);
	port(CLK : in std_logic;
		 RESET : in std_logic;
		 rp0_addr : in std_logic_vector(3 downto 0);
		 rp0_data : out std_logic_vector(width-1 downto 0);
		 rp1_data : in std_logic_vector(3 downto 0);
		 rd1_data : out std_logic_vector(width-1 downto 0);
		 wp_addr : in std_logic_vector(3 downto 0);
		 wp_data : in std_logic_vector(width-1 downto 0);
		 wp_en : in std_logic));
end entity;

architecture behaviour of ice_rf_gen is

-- signals
signal s_rf : std_logic_vector(15 downto 0);
signal s_sel : std_logic_vector(15 downto 0);

-- components
component ice_rfdecode


package bus_mux_pkg is
        type bus_array is array(natural range <>) of std_logic_vector;
end package;

component ice_mux is
	generic(width : positiv := 16);
	port(mux_in : in bus_array(15 downto 0)(width-1 downto 0);
		 rp_addr : in std_logic_vector(3 downto 0);
		 rp_data : out std_logic_vector(width-1 downto 0));
end component;

architecture dataflow of ice_mux is
begin

	rp_data <= mux_in(to_integer(unsigned(rp_addr)));
	
end dataflow;

-- structural port map
U1 : dec port map(addr => ADDR, en => EN, sel => SEL);
U2 : mux port map(s_rf => mux_in, rp_data => rp0_data, rp_addr => rp0_addr);
U2 : mux port map(s_rf => mux_in, rp_data => rp1_data, rp_addr => r1_addr);

-- generate 16 register
begin
	gen_reg:
	for i in 0 to 15 generate
		regx : reg port map
			(wp_data(i), s_sel(i), RESET, CLK, s_rf(i));
		end generate gen_reg;
end gen_reg;
```