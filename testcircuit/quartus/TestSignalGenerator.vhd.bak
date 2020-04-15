library ieee;
use ieee.numeric_std.all;
use ieee.std_logic_1164.all;

-- Test circuit to simulate the signals of a VIC in the C64 to drive
-- and test a C64 video enhancement board in PAL mode.
-- The program runs on an Delta Brd, which I got from another hobbyist, so
-- it is a super-special one-off. 

entity TestSignal is	
	port (
		-- reference clock
		CLK100:  in std_logic;
		
		-- generated signals
		DB:    out std_logic_vector(11 downto 0);
		A:     out std_logic_vector(5 downto 0);
		CS:    out std_logic;
		RW:    out std_logic; 
		BA:    out std_logic;
		AEC:   out std_logic;
		PHI0:  out std_logic
	);	
end entity;


architecture immediate of TestSignal is

	signal PIXELCLOCK : std_logic;

begin		
	-- generate a 7.88 Mhz clock from the 100 Mhz input.
	-- the duty cycle is very non-50%, but this makes no difference
	process (CLK100, PIXELCLOCK) 
		variable tick : integer range 0 to 15 := 0;
		variable needreset : boolean := false;
		variable out_clock : std_logic := '0';
		
		variable tmp_needreset : boolean;
		variable tmp_tick : std_logic_vector(3 downto 0);
		
		variable compensationsequence : integer range 0 to 255 := 0;
		variable longclock : boolean := false;
		 
	begin 
		if rising_edge(CLK100) then
			tmp_tick := std_logic_vector(to_unsigned(tick,4));
			out_clock := tmp_tick(3);
			
			tmp_needreset := (longclock and tick=11) or ((not longclock) and tick=10);
			if needreset then
				tick := 0;
			else
				tick := tick+1;
			end if;			
			needreset := tmp_needreset;
		end if;
		
		PIXELCLOCK <= out_clock;
		
		if falling_edge(PIXELCLOCK) then
			longclock := ((compensationsequence mod 2) = 0) 
					or ((compensationsequence mod 16) = 3)
					or ((compensationsequence mod 16) = 7)
					or ((compensationsequence mod 16) = 11);
			if compensationsequence/=251 then
				compensationsequence := compensationsequence+1;
			else
				compensationsequence := 0;
			end if;
		end if;
	end process;


	process (PIXELCLOCK)
		variable displayline : integer range 0 to 311 := 0;
		variable cycle : integer range 1 to 63 := 1;
		variable pixel: integer range 0 to 7 := 0;
	begin	

	
		if rising_edge(PIXELCLOCK) then
			-- idle levels of all signals
			DB <= "111111111111";
			A  <= "111111";
			CS <= '1';
			RW <= '1'; 
			BA <= '1';

			-- signals in first half of the c64 clock
			if pixel<4 then			
				AEC <= '0';
				PHI0 <= '0';
				
				-- dram refresh pattern (not the correct one, but the mod will detect it)
				if cycle>=11 and cycle<=15 then
					if displayline=311 then
						A(1 downto 0) <= "11";
					elsif cycle=11 then
						A(1 downto 0) <= "11";
					elsif cycle=12 then
						A(1 downto 0) <= "10";
					elsif cycle=13 then
						A(1 downto 0) <= "01";
					elsif cycle=14 then
						A(1 downto 0) <= "00";
					else
						A(1 downto 0) <= "11";
					end if;
				end if;			
				
			
			-- signals in second half of the c64 clock
			else	
				PHI0 <= '1';
				
				-- vic accessing the video matrix data (every line, but that does not bother me now)
				if displayline>=50 and displayline<250 and cycle>=15 and cycle<55 then
					AEC <= '0';
					DB(11 downto 8) <= "0110"; -- dark blue text color, character data is all pixel set by default
					-- add color stripes
					if cycle>=20 and cycle<30 and displayline>=80 and displayline<208 then
						DB(11 downto 8) <= std_logic_vector(to_unsigned((displayline-80) / 8, 4));
					end if;
	
				-- cpu may use the bus now
				else
					AEC <= '1';
					-- register writes
					if displayline=0 and cycle<=5 then
						CS <= '0';
						RW <= '0';
						case cycle is
						when 1 =>  
							A <= "010001";    -- control register 1
							DB(7 downto 0) <= "00011011";  -- enable display, text mode, 25 rows					
						when 2 =>  
							A <= "010110";    -- control register 2
							DB(7 downto 0) <= "00001000";  -- no multicolor, 40 columns					
						when 3 =>  
							A <= "100000";    --  Border color
							DB(7 downto 0) <= "00001110";  -- light blue					
						when 4 =>  
							A <= "100001";    --  Background color 0
							DB(7 downto 0) <= "00000001";  -- white
						when 5 =>  
							A <= "101001";    -- if A(3) were faulty, this writes to Background color 0
													-- otherwise it writes to sprite color 2, which is unused
							DB(7 downto 0) <= "00001000";  -- orange	
						when others =>
						end case;
						
					-- try to provoke register writes with CS=1 or RW=1
					elsif displayline=1 and cycle<=3 then
						case cycle is
						when 1 =>
							CS <= '1';
							RW <= '0';
							A <= "100001";    --  Background color 0
							DB(7 downto 0) <= "00000100";  -- purple	
						when 2 =>
							CS <= '0';
							RW <= '1';
							A <= "100001";    --  Background color 0
							DB(7 downto 0) <= "00000101";  -- green	
						when 3 =>
							CS <= '1';
							RW <= '1';
							A <= "100001";    --  Background color 0
							DB(7 downto 0) <= "00000111";  -- yellow	
						when others =>
						end case;
					end if;
				end if;				
			end if;
			
			-- text graphics pattern fetch response would belong to first half of cycle, 
			-- but because the response is expected so late, we need to provide this on both halves
			if displayline>=220 and displayline<229 then
				if cycle=45 then
					case displayline-220 is
					when 0  		=> DB(7 downto 0) <= "11111100";
					when 1 		=> DB(7 downto 0) <= "11111011";
					when 2 		=> DB(7 downto 0) <= "10001011";
					when 3 		=> DB(7 downto 0) <= "01111011";
					when 4		=> DB(7 downto 0) <= "01111011";
					when 5 		=> DB(7 downto 0) <= "01111011";
					when 6 		=> DB(7 downto 0) <= "10001100";
					when 7 		=> DB(7 downto 0) <= "11111111";
					when others => DB(7 downto 0) <= "11111111";
					end case;
				elsif cycle=46 then
					case displayline-220 is
					when 0 		=> DB(7 downto 0) <= "11111111";
					when 1 		=> DB(7 downto 0) <= "01111111";
					when 2 		=> DB(7 downto 0) <= "01000110";
					when 3 		=> DB(7 downto 0) <= "01011010";
					when 4 		=> DB(7 downto 0) <= "01011010";
					when 5 		=> DB(7 downto 0) <= "01011010";
					when 6 		=> DB(7 downto 0) <= "11000110";
					when 7 		=> DB(7 downto 0) <= "11011110";
					when others => DB(7 downto 0) <= "11011110";
					end case;						
				elsif cycle=47 then
					case displayline-220 is
					when 0 		=> DB(7 downto 0) <= "11111111";
					when 1 		=> DB(7 downto 0) <= "11111111";
					when 2 		=> DB(7 downto 0) <= "00111001";
					when 3 		=> DB(7 downto 0) <= "11010110";
					when 4 		=> DB(7 downto 0) <= "11010000";
					when 5 		=> DB(7 downto 0) <= "11010111";
					when 6 		=> DB(7 downto 0) <= "00111000";
					when 7 		=> DB(7 downto 0) <= "11111111";
					when others => DB(7 downto 0) <= "11111111";
					end case;						
				elsif cycle=48 then
					case displayline-220 is
					when 0 		=> DB(7 downto 0) <= "11111111";
					when 1 		=> DB(7 downto 0) <= "11111111";
					when 2 		=> DB(7 downto 0) <= "10000110";
					when 3 		=> DB(7 downto 0) <= "10110101";
					when 4 		=> DB(7 downto 0) <= "10111101";
					when 5 		=> DB(7 downto 0) <= "10111101";
					when 6 		=> DB(7 downto 0) <= "10111110";
					when 7 		=> DB(7 downto 0) <= "11111111";
					when others => DB(7 downto 0) <= "11111111";
					end case;						
				elsif cycle=49 then
					case displayline-220 is
					when 0 		=> DB(7 downto 0) <= "10111111";
					when 1 		=> DB(7 downto 0) <= "10111111";
					when 2 		=> DB(7 downto 0) <= "00100001";
					when 3 		=> DB(7 downto 0) <= "10101101";
					when 4 		=> DB(7 downto 0) <= "10101111";
					when 5 		=> DB(7 downto 0) <= "10101111";
					when 6 		=> DB(7 downto 0) <= "00101111";
					when 7 		=> DB(7 downto 0) <= "11111111";
					when others => DB(7 downto 0) <= "11111111";
					end case;						
				elsif cycle=50 then
					case displayline-220 is
					when 0 		=> DB(7 downto 0) <= "11111111";
					when 1 		=> DB(7 downto 0) <= "11111111";
					when 2 		=> DB(7 downto 0) <= "10011100";
					when 3 		=> DB(7 downto 0) <= "01101011";
					when 4 		=> DB(7 downto 0) <= "01101011";
					when 5 		=> DB(7 downto 0) <= "01101011";
					when 6 		=> DB(7 downto 0) <= "10001100";
					when 7 		=> DB(7 downto 0) <= "11111111";
					when others => DB(7 downto 0) <= "11111100";
					end case;						
				elsif cycle=51 then
					case displayline-220 is
					when 0 		=> DB(7 downto 0) <= "11111111";
					when 1 		=> DB(7 downto 0) <= "11111111";
					when 2 		=> DB(7 downto 0) <= "11100110";
					when 3 		=> DB(7 downto 0) <= "01011010";
					when 4 		=> DB(7 downto 0) <= "01011010";
					when 5 		=> DB(7 downto 0) <= "01011010";
					when 6 		=> DB(7 downto 0) <= "01100110";
					when 7 		=> DB(7 downto 0) <= "01111111";
					when others => DB(7 downto 0) <= "01111111";
					end case;						
				elsif cycle=52 then
					case displayline-220 is
					when 0 		=> DB(7 downto 0) <= "11111111";
					when 1 		=> DB(7 downto 0) <= "11111111";
					when 2 		=> DB(7 downto 0) <= "00111111";
					when 3 		=> DB(7 downto 0) <= "11011111";
					when 4 		=> DB(7 downto 0) <= "11011111";
					when 5 		=> DB(7 downto 0) <= "11011111";
					when 6 		=> DB(7 downto 0) <= "11011111";
					when 7 		=> DB(7 downto 0) <= "11111111";
					when others => DB(7 downto 0) <= "11111111";
					end case;						
				end if;
			end if;	
		
			-- progress the counters 
			if pixel/=7 then 
				pixel := pixel+1;
			else
				pixel := 0;
				if cycle/=63 then
					cycle := cycle+1;
				else
					cycle := 1;
					if displayline/=311 then
						displayline := displayline +1;
					else
						displayline := 0;
					end if;
				end if;
			end if;
		end if;
	end process;		
end immediate;
