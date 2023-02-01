# MEMS 1.x ECUs - 1.2, 1.3, 1.6, 1.9

These ECUs have a common diagnostic protocol

The 1.9 ECU is slightly different in how this is initialised, but the rest are all the same

Still need to format this properly for markdown


Main portion of the PC application code is here: https://github.com/james-portman/rover-mems-agent/
This may mean absolutely nothing to most people.
I am trying to catalogue information about how to communicate with these ECUs for anyone who wants to know more, or join in with the development work.
There is a lot of very good information out there already but it is all a bit disjointed or only applies to certain ECU versions.
Wake up/initialize ECU
MEMS 1.3 and 1.6
Send bytes 0xCA, 0x75, 0xD0
ECU should reply 0xCA, 0x75, 0xD0, then 4 more bytes which is the ECU ID
5V TTL level must be used or the ECU will not respond

MEMS 1.2, 1.3, 1.6, 1.9 "Data packets"
These ECUs respond to the commands 0x80 and 0x7D with data packets, which have multiple data points in
The (single plug?) 1.2 ECUs do not respond with a 0x7D packet at all
Each version replies with a slightly different amount of data
The first byte of the packet (ignoring the echo/command reply) is the length, so that should be used to wait for the rest of the data packet to be received

Information is a bit vague/mixed but here is the best explanation I can get so far for the data points:

Data packet 0x7D
0x00 Size of data packet, including this byte
0x01 Ignition switch - 0 for off (does comms even work with ignition off?)
0x02 Throttle angle - divide by 2 for degrees throttle open
0x03 Unknown
0x04 Sometimes documented to be air/fuel ratio, but often observed to never change from 0xFF
0x05 DTC byte
   bit 3 - lambda heater (relay)
   bit 4 - crank shaft sync
   bit 5 - fan 1 control
   bit 7 - fan 2 control
0x06 Lambda sensor voltage, 0.5mv per LSB
0x07 Lambda sensor - 0=off, greater=on
0x08 Lambda sensor duty cycle?
0x09 Lambda sensor status? 0x01 for good, any other value for no good
0x0A Loop indicator, 0 for open loop and nonzero for closed loop
0x0B Long term trim? Injector dead time (-1024 to 1016)?
0x0C Fuel correction, 1% per LSB, 100 is normal, could be 100-[this value]?
0x0D Carbon canister purge valve duty cycle?
0x0E DTC
   bit 1 - cam shaft sync
0x0F Idle position - 0 to 255 steps
0x10 Unknown
0x11 DTC
   bit 6 - RPM gauge circuit
0x12 Unknown
0x13 Unknown
0x14 Idle error?
0x15 Unknown
0x16 DTC
   bit 1 - injector 1/4 driver
   bit 2 - injector 2/3 driver
   bit 3 - engine bay ventilator warning light
   bit 4 - engine bay ventilator relay
   bit 5 - hill assist
   bit 6 - cruise control
0x17
0x18
0x19
0x1A
0x1B
0x1C
0x1D
0x1E
0x1F - Crank counter?


Data packet 0x80
0x00 Size of data packet, including this byte
0x01-2 Engine speed in RPM (16 bits)
0x03 Coolant temperature in degrees C with +55 offset and 8-bit wrap
0x04 Computed ambient temperature in degrees C with +55 offset and 8-bit wrap
0x05 Intake air temperature in degrees C with +55 offset and 8-bit wrap
0x06 Fuel rail temperature in degrees C with +55 offset and 8-bit wrap. This is not supported on the Mini SPi, and always appears as 0xFF.
0x07 MAP sensor value in kilopascals
0x08 Battery voltage, 0.1V per LSB (e.g. 0x7B == 12.3V)
0x09 Throttle pot voltage, 0.02V per LSB. WOT should probably be close to 0xFA or 5.0V.
0x0A Idle switch. Bit 4 will be set if the throttle is closed, and it will be clear otherwise.
0x0B Unknown. Probably a bitfield. Observed as 0x24 with engine off, and 0x20 with engine running. A single sample during a fifteen minute test drive showed a value of 0x30.
0x0C Park/neutral switch. Zero is closed, nonzero is open. Air con switch?
0x0D Fault codes. On the Mini SPi, only two bits in this location are checked:
   Bit 0: Coolant temp sensor fault (Code 1)
   Bit 1: Inlet air temp sensor fault (Code 2)
   bit 3: turbo overboosted
   bit 4: ambient temp sensor
   bit 5: fuel rail temp sensor
   bit 6: knock detected
0x0E Fault codes. On the Mini SPi, only two bits in this location are checked:
   bit 0: (coolant?) temperature gauge
   Bit 1: Fuel pump circuit fault (Code 10)
   bit 3: air con clutch
   bit 4: purge valve
   bit 5: map sensor
   bit 6: boost valve
   Bit 7: Throttle pot circuit fault (Code 16)
0x0F Idle setpoint - 0 to 1556 RPM (6.1x this value)
0x10 Idle HotBDPos, steps?
0x11 Unknown
0x12 Idle air control motor position/steps. On the Mini SPi's A-series engine, 0 is closed, and 180 is wide open.
0x13-14 Idle speed deviation (16 bits)
0x15 Ignition Advance offset - 0 to 159.38 degrees
0x16 Ignition advance, 0.5 degrees per LSB with range of -24 deg (0x00) to 103.5 deg (0xFF), -30 to 129.38 degrees?
0x17-18 Coil charge/dwell time, 0.002 milliseconds per LSB (16 bits) - 0 to 130.56 microseconds

MEMS 1.3, 1.6, 1.9 "Commands"
Still being tidied/updated
Command	Info	Response
Stop/Disable commands (match the Start/enable commands but swap 0/1 at start)	
0x00	Coolant gauge	0x00, 0x00
0x01	Fuel pump relay	0x01, 0x00
0x02	PTC (inlet manifold heater) relay	0x02, 0x00
0x03	Air Conditioning relay	0x03, 0x00
0x04	Idle solenoid	
0x05	ORFCO solenoid	
0x06	Pulse air valve	
0x07	EGR valve	
0x08	Purge valve	0x08, 0x00
0x09	O2 heater relay	0x09, 0x00
0x0A	Emissions fail lamp	
0x0B	Wastegate	
0x0C	Fuel used	
0x0D	Fan 1	
0x0E	Fan 2	
0x0F	Variable valve timing	
Start/Enable commands (match the open commands but swap 0/1 at start)	
0x10	Coolant gauge	0x10, 0x00
0x11	Fuel pump relay	0x11, 0x00
0x12	PTC (inlet manifold heater) relay	0x12, 0x00
0x13	Air Conditioning relay	0x13, 0x00
0x14	Idle solenoid	
0x15	ORFCO solenoid	
0x16	Pulse air valve	
0x17	EGR valve	
0x18	Purge valve	0x18, 0x00
0x19	O2 heater relay	0x19, 0x00
0x1A	Emissions fail lamp	
0x1B	Wastegate	
0x1C	Fuel used	
0x1D	Fan 1 relay	0x1D
0x1E	Fan 2 relay	0x1E
0x1F	Variable valve timing	
End of Start/Stop grouped commands (there are some more later)	
0x20	Engine bay temperature warning light off	20 00
0x21	Cruise control disable relay	21 00
0x30	Engine bay temperature warning light on	30 00
0x31	Cruise control disable relay off	31 00
0x60	Test RPM gauge stop / Exhaust backpressur valve test	60 00
0x61	Variable intake test	61 00
0x63	Test RPM gauge	63 00
0x64	Test Boost gauge	64 00
0x65	S/W Throt S.W	65 00
0x67	Fan 3 (engine bay) off	67 00
0x6B	Test RPM gauge start	6B 00
0x6D	?	6D 00
0x6F	Fan 3 (engine bay) on	6F 00
0x79	Increments fuel trim setting and returns the new value	0x79, [new value]
0x7A	Decrements fuel trim setting and returns the new value	0x7A, [new value]
0x7B	Increments second fuel trim setting and returns the new value	0x7B, [new value]
0x7C	Decrements second fuel trim setting and returns the new value	0x7C, [new value]
0x7D	Request data packet 0x7D	0x7D, [data packet]	
0x7E	? Used as part of (auto?) idle adjustment	7E 08
0x7F	? Used as part of (auto?) ignition adjustment	7F 05
0x80	Request data frame 80, followed by 28-byte data frame	
0x81	? Used at end of idle/ignition/clearing (auto?) adjustments	0x81, 0x00
0x82	?	82 09 9E 1D 00 00 60 05 FF FF
0x89	Increments idle decay setting and returns the new value	0x89, [new value]
0x8A	Decrements idle decay setting and returns the new value	0x8A, [new value]
0x91	Increments idle speed setting and returns the new value	0x91, [new value]
0x92	Decrements idle speed setting and returns the new value	0x92, [new value]
0x93	Increments ignition advance offset and returns the new value	0x93, [new value]
0x94	Decrements ignition advance offset and returns the new value	0x94, [new value]
0x9E	Alternate first byte of init (different diag mode/security level?)	9E
0xC4	Swap to diagnostic mode 4 (only from mode 3)	C4 xx
0xCA	First byte of "normal" init	CA
0xCB	?	CB 00
0xCC	Clear fault codes	CC 00
0xCD	Debug? Read RAM?	CD 01
0xCE	Alternate first byte of init (different diag mode/security level?)	CE
0xCF	Alternate first byte of init (different diag mode/security level?)	CF
0xD0	ECU/Software ID	D0 99 00 03 03
0xD1	ECU/Software IDs 1x integer, 1x Ascii	D1 41 42 4E 4D 50 30 30 33 99 00 03 03 41 42 4E 4D 50 30 30 33 99 00 03 03 41 42 4E 4D 50 30 30 33 99 00 03 03
e.g. integer: 99 00 03 03
e.g. string/Ascii: 41 42 4E 4D 50 30 30 33 = ABNMP003
0xD2	Read security status	D2, followed by 02 01, 00 01, or 01 01
0xD3	Recode ECU	D3, followed by 02 01, 00 02, or 01 01 (reply needs checking)
0xDA	Test injector 1 (mems 1.9)	DA, 01?
0xDB	Test injector 2 (mems 1.9)	DB, 01?
0xDE	Alternate first byte of init (different diag mode/security level?)	DE
0xE0	Alternate first byte of init (different diag mode/security level?)	E0
0xE5	Alternate first byte of init (different diag mode/security level?)	E5
0xE7	?	E7 02
0xE8	?	E8 05 26 01 00 01
0xE9	Clear faults 2nd? needs ignition cycle?	
0xEA		
0xEB		
0xEC		
0xED	?	ED 00
0xEE	?	EE 00
0xEF	Actuate fuel injectors? (MPi?)	EF 03
0xF0	Check current diagnostic mode	F0 14 - mode 3 (default), 1E - mode 4, 50 - mode 5 or 6 (you should know which)
0xF1		
0xF2	Swap to diagnostic mode 6 (only from mode 4)	F2 xx
0xF3	Swap to diagnostic mode 4 (from mode 5,6)	F3 xx
0xF4	Swap to diagnostic mode 5 (from mode 3)	F4 xx
0xF5	Swap to diagnostic mode 3 (from mode 4,5,6)	F5 xx
0xF6	Disconnect/Reset diagnostic session	
0xF7	Actuate fuel injector (SPi?)	F7 03
0xF8	Fire ignition coil	F8 02
0xF9	Adjust main map? 2 bytes input also?	0x00 on success
0xFA	Clear all adaptations	0xFA, 0x00
0xFB	Request current IAC position	0xFB [IAC position XX]
0xFC	?	FC 00
0xFD	Open IAC by one step and report current position	FD, [IAC position]
0xFE	Close IAC by one step and report current position	FE, [IAC position]
0xFF	Request current IAC position?	


Diag mode 4 commands - factory programming

Some commands will only work if the ECU is in development mode,
for the 2J ECUs this means they need to have additional RAM installed.
The byte/word reads only work in certain locations on stock ECU.
Command	Info	Response
0x00-0x7F	Read a byte/word from RAM/ROM, command is offset (block set by 0xDC command)	
0x80-0x9F	Increase value by 1 in selected cal (0x80 == 0x00 offset) (block set by 0xC1,0xC2,0xC3 commands)	
0xA0-0xBF	Decrease value by 1 in selected cal (0xA0 == 0x00 offset) (block set by 0xC1,0xC2,0xC3 commands)	
0xC1	Clears the block number for writes (blocks of 0x20 bytes)	
0xC2	Increase the block number for writes (blocks of 0x20 bytes)	
0xC3	Decrease the block number for writes (blocks of 0x20 bytes)	
0xC5	Choose RAM bank 1 for data loads	0xC5
0xC6	Choose RAM bank 2 for data loads	0xC6
0xC7	Set cal pointer to normal ROM	
0xC8	Set cal pointer to RAM bank 1	
0xC9	Set cal pointer to RAM bank 2	
0xD1	Write both RAM cal banks to ROM banks	
0xD3	Write single cal from RAM -> ROM (use C5/C6 to choose bank)	
0xD4	Write single cal from ROM -> RAM (or ROM -> ROM if bank not set) (use C5/C6 to choose bank)	
0xDC, 0x__	Sets the block number for reads (blocks of 0x80 bytes)	0xDC, 0x__
0xF7	Reads out the running calibration in full	0xF7, ...full calibration is streamed out
0xF8	Write full calibration (only where ECU is in development mode) - follow with full cal data	0xF8
