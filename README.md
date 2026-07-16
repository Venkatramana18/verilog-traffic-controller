# 🚦FPGA-Based Traffic Light Controller (Highway / Country Road Intersection)

## 🧠 Project Overview
This project implements a **vehicle-triggered traffic light controller** for a highway/country-road intersection using Verilog HDL on a Xilinx Spartan-7 FPGA.

Unlike a simple fixed-timer traffic light, this design uses a vehicle sensor input to decide when the country road gets a green light, so the highway stays green by default and only yields when a vehicle is actually waiting. The design is implemented, simulated, and verified in Xilinx Vivado.

---

## 🎯 Objectives
- Design a digital traffic control FSM using Verilog HDL.
- Implement Red, Yellow, and Green sequencing with accurate timing.
- Use a vehicle sensor input to trigger the country road's turn, instead of a fixed round-robin timer.
- Drive both intersection lights from on-board RGB LEDs.
- Perform simulation and functional verification using the Vivado Simulator.


---
## 🚥 System Description

### Normal Operation
The highway light stays **green** by default. The controller only advances when the vehicle sensor (`x`) detects a car waiting at the country road.
 
| State | Highway Light | Country Road Light | Condition to Advance |
|-------|---------------|---------------------|------------------------|
| S0 | Green | Red | Vehicle detected (`x == 1`) |
| S1 | Yellow | Red | 1 second elapsed |
| S2 | Red | Red | 1 second elapsed |
| S3 | Red | Green | Vehicle leaves (`x == 0`) |
| S4 | Red | Yellow | 1 second elapsed |
 
After S4, the FSM returns to S0 and the highway goes green again.

---

## 🧠 Block Diagram
<img width="1024" height="1024" alt="Gemini_Generated_Image_qq5gaeqq5gaeqq5g" src="https://github.com/user-attachments/assets/3b3bad74-e556-42be-9006-f3455cf6ff1a" />

```
[Vehicle Sensor] → [FSM Control Logic] → [Timing Counter] → [RGB LED Outputs]
```
 
| Signal | Description |
|--------|-------------|
| `clk` | 100 MHz system clock |
| `reset` | Asynchronous reset, restarts sequence at S0 |
| `x` | Vehicle sensor input (country road) |
| `rgb1[2:0]` | Highway light — on-board RGB LED 0 (bit0=R, bit1=G, bit2=B) |
| `rgb2[2:0]` | Country road light — on-board RGB LED 1 (bit0=R, bit1=G, bit2=B) |

---

## ⚙️ Tools & Technologies Used
 
| Tool | Purpose |
|------|---------|
| Xilinx Vivado | Design, simulation, synthesis, and implementation |
| Verilog HDL | Hardware description and FSM logic |
| Vivado Simulator | Functional simulation and waveform analysis |
| Arty S7-50 (Spartan-7 XC7S50-CSGA324) | Target FPGA board |
| GitHub | Version control and documentation |
---

---

## 🧾 Verilog Design Description

### Main Module (`traffic_controller.v`)
A 5-state Moore machine (`S0`–`S4`) with:
- An asynchronous-reset synchronous state register.
- A free-running counter used to time the yellow/red hold states (1 second at 100 MHz).
- Combinational output logic mapping each state to an `rgb1`/`rgb2` color code.

```verilog
module traffic_controller (
    input  wire clk,
    input  wire reset,
    input  wire x,
    output reg [2:0] rgb1,   // highway light: [0]=R, [1]=G, [2]=B
    output reg [2:0] rgb2    // country road light: [0]=R, [1]=G, [2]=B
);
    reg [2:0] state;
    reg [31:0] counter;
    parameter S0 = 3'b000; // Highway Green, Country Red
    parameter S1 = 3'b001; // Highway Yellow, Country Red
    parameter S2 = 3'b010; // Highway Red, Country Red
    parameter S3 = 3'b011; // Highway Red, Country Green
    parameter S4 = 3'b100; // Highway Red, Country Yellow
    parameter HOLD = 100000000; // 1 second at 100 MHz
 
    always @(posedge clk or posedge reset) begin
        if (reset) begin
            state   <= S0;
            counter <= 0;
        end else begin
            case (state)
                S0: if (x) begin
                    state   <= S1;
                    counter <= 0;
                end
                S1: begin
                    counter <= counter + 1;
                    if (counter == HOLD) begin
                        state   <= S2;
                        counter <= 0;
                    end
                end
                S2: begin
                    counter <= counter + 1;
                    if (counter == HOLD) begin
                        state   <= S3;
                        counter <= 0;
                    end
                end
                S3: if (!x) begin
                    state   <= S4;
                    counter <= 0;
                end
                S4: begin
                    counter <= counter + 1;
                    if (counter == HOLD) begin
                        state   <= S0;
                        counter <= 0;
                    end
                end
                default: state <= S0;
            endcase
        end
    end
 
    // Output logic: rgb1/rgb2 bit order = R(bit0), G(bit1), B(bit2)
    always @(*) begin
        rgb1 = 3'b000;
        rgb2 = 3'b000;
        case (state)
            S0: begin rgb1 = 3'b010; rgb2 = 3'b001; end // hwy green, ctr red
            S1: begin rgb1 = 3'b011; rgb2 = 3'b001; end // hwy yellow, ctr red
            S2: begin rgb1 = 3'b001; rgb2 = 3'b001; end // hwy red, ctr red
            S3: begin rgb1 = 3'b001; rgb2 = 3'b010; end // hwy red, ctr green
            S4: begin rgb1 = 3'b001; rgb2 = 3'b011; end // hwy red, ctr yellow
        endcase
    end
endmodule

```

---

### Testbench (`traffic_controller_tb.v`)
Simulates reset, a vehicle arriving and triggering the full S0→S4 cycle, and a second cycle to confirm repeatability.
 
```verilog
`timescale 1ns/1ps
module traffic_controller_tb();
    reg clk, reset, x;
    wire [2:0] rgb1, rgb2;
 
    traffic_controller uut (
        .clk(clk), .reset(reset), .x(x),
        .rgb1(rgb1), .rgb2(rgb2)
    );
 
    always #5 clk = ~clk;
    initial begin
        clk = 0; reset = 1; x = 0;
        #20 reset = 0;
        #100;
        x = 1;
        #2_000_000_100;
        x = 0;
        #1_000_000_100;
        x = 1;
        #2_000_000_100;
        x = 0;
        #1_000_000_100;
        $display("Simulation completed");
        $stop;
    end
endmodule
```

---

## 📈 Simulation Results
 
| Phase | Expected Behavior |
|-------|--------------------|
| Reset | Highway Green, Country Red (S0) |
| Vehicle arrives | S0 → S1 (Yellow) → S2 (Red) after two 1-second holds |
| Country road active | S3: Highway Red, Country Green until vehicle clears |
| Vehicle leaves | S3 → S4 (Country Yellow) → back to S0 |

### 📸 Example Waveform Output
<img width="1918" height="1066" alt="image" src="https://github.com/user-attachments/assets/499d690b-9aa3-4b89-afbb-7fb61127f378" />



---

## 🧩 RTL Schematic
After synthesis, Vivado generates an RTL schematic showing the state machine and flip-flop connections.  
You can open it via:  
`Flow Navigator → Synthesis → Open Synthesized Design → RTL Analysis`.

RTL SCHEMATIC DIAGRAM:
<img width="1630" height="992" alt="image" src="https://github.com/user-attachments/assets/42951caa-5038-40a9-930a-67af9885cfbf" />


RTL SCHEMATIC DIAGRAM AFTER SYNTHESIS:
<img width="1630" height="997" alt="image" src="https://github.com/user-attachments/assets/084cc7e2-be5e-4f43-9232-0a150462ff07" />


DEVICE LAYOUT AFTER IMPLEMENTATION
<img width="1631" height="987" alt="image" src="https://github.com/user-attachments/assets/5f06ff12-a46d-40b9-b6a2-07cb670de9a0" />



---

## 🔍 Key Learnings
- Hands-on experience with the FPGA design flow: RTL → simulation → constraints → bitstream.
- Debugged switch bounce, clock-speed-related invisible intermediate states, and active-high vs. active-low LED logic on real hardware.
- Learned to combine three discrete LED signals into a single RGB output bus and update both the constraints file and the module port list in tandem.

---

## 🧾 Conclusion
The **FPGA-Based Traffic Light Controller** was successfully designed and simulated using **Xilinx Vivado**.
The system correctly performed sequential light transitions and responded to the vehicle sensor input, holding the highway green until a vehicle was detected and returning to it once the country road cleared.
This design can be extended to include **pedestrian crossing states**, **multiple intersections**, and **adaptive timing based on traffic density** for smart city applications.


## 🧑‍💻 Author
**Venkat Ramana S B**  
Department of Electronics and Communication Engineering  
Saveetha Engineering College, Chennai  

📧 **Email:** [sbvenkatramana@gmail.com](mailto:sbvenkatramana@gmail.com)  
📅 **Batch:** 2024–2028
⭐ *Project developed and simulated using Xilinx Vivado 2025.1*
