# lab 9
## UART
- Start bit
- 5 bis maximal 9 Datenbits
- optionales Parity bit
- Stopp bit
- asynchron, kein eigenes Taktsignal, synchronisierung über Flanke des Start/Stop bits bzw eingestellte baudrate

|0|1|2|3|4|5|6|7|8|9|10|
|-|-|-|-|-|-|-|-|-|-|-|-|-|
|**Start**|x1|x2|x3|x4|x5|x6|x7|x8|**Parity**|**Stop**|

## ICE UART
|0|1|2|3|4|5|6|7|8|9|10|
|-|-|-|-|-|-|-|-|-|-|-|-|-|
|**Start**|x0|x1|x2|x3|x4|x5|x6|x7|x8|**Stop**|

## Schnittstelle
```vhdl
entity ice_uart is
	port(clk : in std_logic;
		 reset : in std_logic;
		 brd : in std_logic_vector(23 downto 9);
		 din : in std_logic_vector(7 downto 0);
		 din_wen : in std_logic;
		 txd : out std_logic;
		 tx_busy : out std_logic);
end entity;
```
## Baudratenuntersetzer
- ICE Prozessortakt 50 MHz
- typische Baudraten: 75, 110, 300, 600, 1200, 2400, 4800, 9600, 19200, 38400, 57600, 115200
- f_baud = f_clk/D
- T_baud = T_clk * D
- freilaufender Zähler, erzeugt bei Erreichend des Divisors **D** einen Puls, dann wieder auf Null

```vhdl
entity ice_counter is
	port(clk : in std_logic;
		 baud_tick : out std_logic);
end entity;

architecture behaviour of ice_counter is
signal i : integer;
signal d : integer := ;

begin
	process(clk)
	begin
		if (clk'event and clk = '1' and i <= 8) then
			i <= i + '1';
		elsif
			baud_tick <= '1'
			wait_pulse : process
			begin
				wait until clk'event and clk = '0';	
				baud_tick <= '0';
				i <= 0;
			end process;
		end if;
	end process;
end architecture
```

## Sendeautomat
- **Idle**: 
	- tx_busy = 0, wird we aktiviert geht der Automat in Wait über, tx_busy wird gesetzt und  das schieberegister shift_init geladen 
- **Wait**:
	- automat wartet auf ersten Impulsdes Baudratenteilers auf tick-leitung, sobald tick gesetzt ist wechsel in Transmit
- **Transmit**
	- wird ein Impuls vom baudratenteiler erkannt solange Sendeschieberregister noch leer ist (count < 9), wird der bitzähler erhöht und das sendeschieberegister führt verschiebung nach rechts durch (shift = shift_next)

```vhdl
architecture behaviour of ice_uart is

	type state is (idle, waiting, transmit);
	signal count : integer := '0';
	state <= idle;
	
	uart_rx : process(clk)
	begin
		if rising_edge(clk) then
			
			case state is
				
				when idle =>
					tx_busy <= '0'
					
					if (din_wen = '1') then
						state <= waiting;
						tx_busy <= '1';
						shift <= shift_init;
					else
						state <= idle;
					end if;
				
				when waiting =>
					if rising_edge(baud_tick) then
						state <= transmit;
					else
						state <= idle;
					end if;
				
				when transmit =>
					if (count < '9' and baud_tick = '1') then
						shift = next_shift;
					elsif ( count = '9' and baud_tick = '1') then
						count <= '0';
						state <= idle;
					end if;
				end case;
			end process;

					
	
## Schieberegister


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

