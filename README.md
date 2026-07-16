# 🚦FPGA-Based Traffic Light Controller (Highway / Country Road Intersection)

## 🧠 Project Overview
This project implements a **vehicle-triggered traffic light controller** for a highway/country-road intersection using Verilog HDL on a Xilinx Spartan-7 FPGA (Arty S7-50).

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
[Sensor Inputs] → [Control Logic Unit] → [Timing Controller] → [LED Outputs]
```

| Signal | Description |
|---------|--------------|
| `clk` | System clock for timing control |
| `reset` | Asynchronous reset for restarting sequence |
| `emergency` | Input signal from emergency vehicle detector |
| `red`, `yellow`, `green` | Output signals driving the LEDs |

---

## ⚙️ Tools & Technologies Used
| Tool | Purpose |
|------|----------|
| **Xilinx Vivado 2025.1** | Design, simulation, and verification |
| **Verilog HDL** | Hardware description and logic design |
| **Vivado Simulator** | Functional simulation and waveform analysis |
| **FPGA Target Board (Optional)** | Xilinx Spartan-6 / Artix-7 |
| **GitHub** | Version control and documentation |

---

---

## 🧾 Verilog Design Description

### 🔹 Main Module (`traffic_controller.v`)
This module defines the control logic for the traffic lights and includes:
- A state machine with 3 states: `RED`, `YELLOW`, and `GREEN`.  
- Counter logic for timing control.  
- Priority condition for emergency override.

```verilog
module traffic_controller (
    input clk,
    input reset,
    input emergency,
    output reg red,
    output reg yellow,
    output reg green
);

    reg [1:0] state;
    reg [31:0] counter;

    parameter RED_STATE    = 2'b00;
    parameter YELLOW_STATE = 2'b01;
    parameter GREEN_STATE  = 2'b10;

    always @(posedge clk or posedge reset) begin
        if (reset) begin
            state <= RED_STATE;
            counter <= 0;
        end else if (emergency) begin
            state <= GREEN_STATE;
        end else begin
            counter <= counter + 1;
            case (state)
                RED_STATE: if (counter == 100000000) begin
                    state <= GREEN_STATE;
                    counter <= 0;
                end
                GREEN_STATE: if (counter == 100000000) begin
                    state <= YELLOW_STATE;
                    counter <= 0;
                end
                YELLOW_STATE: if (counter == 50000000) begin
                    state <= RED_STATE;
                    counter <= 0;
                end
            endcase
        end
    end

    always @(*) begin
        red    = (state == RED_STATE);
        yellow = (state == YELLOW_STATE);
        green  = (state == GREEN_STATE);
    end

endmodule
```

---

### 🔹 Testbench (`traffic_controller_tb.v`)
This file simulates different input scenarios — normal operation and emergency overrides.

```verilog
`timescale 1ns/1ps
module traffic_controller_tb();

    reg clk, reset, emergency;
    wire red, yellow, green;

    traffic_controller uut (
        .clk(clk),
        .reset(reset),
        .emergency(emergency),
        .red(red),
        .yellow(yellow),
        .green(green)
    );

    always #5 clk = ~clk;  // 10ns clock

    initial begin
        $display("Simulation started");
        clk = 0; reset = 1; emergency = 0;
        #10 reset = 0;

        // Normal cycle
        #200;

        // Emergency override
        emergency = 1;
        #50 emergency = 0;

        #200;
        $display("Simulation completed");
        $stop;
    end

endmodule
```

---

## 📈 Simulation Results

### ✅ Observed Behavior
| Mode | Expected Output |
|------|-----------------|
| Normal | RED → GREEN → YELLOW → RED |
| Emergency | GREEN overrides immediately |
| Post-Emergency | Returns to normal cycle |

### 📸 Example Waveform Output
![WhatsApp Image 2025-10-27 at 15 32 40_2ee2595c](https://github.com/user-attachments/assets/c5df64f6-a161-4d06-acb6-343ddde9ffa7)


---

## 🧩 RTL Schematic
After synthesis, Vivado generates an RTL schematic showing the state machine and flip-flop connections.  
You can open it via:  
`Flow Navigator → Synthesis → Open Synthesized Design → RTL Analysis`.

RTL SCHEMATIC DIAGRAM:
![WhatsApp Image 2025-10-27 at 15 40 02_8dd4b0f3](https://github.com/user-attachments/assets/4a3cc3be-38a9-4e6c-862a-b54cb7d026d4)

RTL SCHEMATIC DIAGRAM AFTER SYNTHESIS:
![WhatsApp Image 2025-10-27 at 16 11 21_2a17c467](https://github.com/user-attachments/assets/143ae31d-1ad1-4eb0-a9df-b30ace76a77d)

DEVICE LAYOUT AFTER IMPLEMENTATION
![WhatsApp Image 2025-10-27 at 16 01 40_6f30e804](https://github.com/user-attachments/assets/362f628c-7f19-4748-bcab-9cdc372ebc7d)

ZOOM IN VIEW:
![WhatsApp Image 2025-10-27 at 15 59 28_20202a8e](https://github.com/user-attachments/assets/6a752554-bd3e-4681-88c2-b541d169e830)


---

## 🔍 Key Learnings
- Gained hands-on experience with **FPGA design flow**.  
- Understood **behavioral modeling** in Verilog.  
- Learned to **simulate and debug digital circuits** using Vivado.  
- Explored how **priority-based control systems** work in real-world traffic management.

---

## 🧾 Conclusion
The **FPGA-Based Traffic Light Controller with Priority System** was successfully designed and simulated using **Xilinx Vivado**.  
The system correctly performed sequential light transitions and responded to emergency override conditions.  
This design can be extended to include **multiple intersections**, **sensor integration**, and **adaptive traffic flow control** for smart city applications.

---

## 🧑‍💻 Author
**Arunachalam P**  
Department of Electronics and Communication Engineering  
Saveetha Engineering College, Chennai  

📧 **Email:** [arunachalam862005@gmail.com](mailto:arunachalam862005@gmail.com)  
📅 **Batch:** 2023–2027  
⭐ *Project developed and simulated using Xilinx Vivado 2025.1*
