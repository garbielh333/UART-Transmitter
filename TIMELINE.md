Project Timeline:

Phase 1: Setup & environment (2–4 hours)
- Install Quartus Prime Lite, verify it recognizes your Cyclone IV device
- Get your USB-Blaster clone working — load a trivial "blink an LED via button" design first, just to confirm your whole toolchain (write → compile → program) works end-to-end before touching UART logic
- Confirm what clock frequency your board's oscillator actually runs at (you mentioned "50M" silkscreen earlier — that's very likely 50 MHz, but verify against the board's schematic/listing)

Phase 2: Design on paper first (2–3 hours)

- Draw out the FSM state diagram: IDLE → START → 8x DATA bits → STOP → back to IDLE
- Decide your target baud rate (115200 is a good default, per our last conversation) and calculate the clock divider math: clock_freq / baud_rate = how many clock cycles = one bit period. At 50 MHz / 115200 baud, that's roughly 434 clock cycles per bit — you'll need a counter that counts to that value to know when to advance states.
- Sketch your module's inputs/outputs on paper: clock, reset, an 8-bit data input, a "send" trigger, and a single serial output pin, plus maybe a "busy" flag so you know when it's safe to load new data.

Phase 3: Write the Verilog/VHDL (4–8 hours)

- Write the FSM + bit counter + clock divider logic
- This is usually the fastest part once the paper design is solid — a UART TX is genuinely one of the more compact designs (well under 100 lines in most implementations)
- Expect a couple of iterations here as syntax errors and off-by-one state issues get shaken out

Phase 4: Simulation before hardware (3–6 hours)

- Before ever touching the FPGA, simulate the design (ModelSim, which ships with Quartus, or free alternatives like Icarus Verilog + GTKWave)
- Feed it a byte, watch the waveform: confirm start bit, all 8 data bits in the right order, stop bit, correct timing between transitions
- This step is where most bugs get caught — much faster to fix in simulation than by squinting at a logic analyzer on real hardware. Don't skip this even though it's tempting to go straight to hardware.

Phase 5: Hardware bring-up (3–5 hours)

- Compile for your actual board, assign pins (your serial output pin needs to go to wherever the CH340's RX line is, or just to a spare GPIO pin if you want to use an external USB-to-serial adapter instead for full flexibility)
- Program the FPGA, open a terminal program (PuTTY, or Arduino IDE's own Serial Monitor works fine) at your chosen baud rate
- Trigger a send (maybe wire a button to your "send" input) and confirm you see the expected character/byte appear in the terminal
- Debug any real-world issues — these are usually pin assignment mistakes or baud rate mismatches, both quick fixes once you find them

Phase 6: Push it further, tie into project 8 (2–4 hours, optional but valuable)

- Add a timing constraint (SDC file) specifying your clock period
- Run TimeQuest and look at your actual slack/critical path report — a design this small will have huge slack margins, but this is good practice for reading the report format
