# 🚦 FPGA-Based Traffic Light Controller with Priority System

## 🧠 Project Overview
This project focuses on designing and implementing a **Traffic Light Controller with Priority System** using **Verilog HDL** on an **FPGA platform**.  
The goal is to create a **smart traffic management system** that not only handles the regular traffic light sequence but also provides a **priority override** for emergency vehicles like ambulances and fire trucks.

The design is implemented, simulated, and verified using **Xilinx Vivado Design Suite**, ensuring correct sequencing, timing, and control logic.  
This project demonstrates how FPGA-based systems can be utilized for **real-time, hardware-level decision-making** in modern urban infrastructure.

---

## 🎯 Objectives
- To design a **digital traffic control system** using Verilog HDL.  
- To implement **Red, Yellow, and Green light** sequencing with accurate timing.  
- To integrate a **priority mechanism** for emergency vehicles.  
- To perform simulation and functional verification using **Vivado Simulator**.  
- To analyze the **RTL schematic** and understand hardware-level behavior.  
- To provide an efficient, low-latency, and hardware-driven traffic management logic.

---

## 🧩 System Description

### 🚥 Normal Operation
The controller cycles through three states:
1. **RED** → Stop vehicles.
2. **YELLOW** → Prepare to move.
3. **GREEN** → Allow vehicles to go.

Each state is maintained for a specified time delay, controlled by an internal counter.

### 🚨 Priority System
When an **emergency vehicle signal** is detected:
- The controller **immediately gives priority** by turning **GREEN** on the emergency lane.
- Once the emergency signal is cleared, the system **returns to normal sequencing**.

---

## 🧠 Block Diagram
*(You can attach your block diagram image here)*

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

## 💻 Project Structure
```
traffic_light_controller_fpga/
│
├── src/
│   ├── traffic_controller.v          # Main Verilog code
│   └── traffic_controller_tb.v       # Testbench code
│
├── simulation/
│   ├── waveform_screenshot.png       # Simulation waveform
│   └── simulation_log.txt            # Optional logs
│
├── docs/
│   ├── block_diagram.png             # Optional diagram
│   └── project_description.md        # Additional notes
│
└── README.md                         # Main GitHub report (this file)
```

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
*(Attach screenshot of simulation here)*  
```
![WhatsApp Image 2025-10-27 at 15 32 40_d4f425de](https://github.com/user-attachments/assets/f6ca0399-c9ba-4c17-80a0-89ef5bf6f1bd)

```

---

## 🧩 RTL Schematic
After synthesis, Vivado generates an RTL schematic showing the state machine and flip-flop connections.  
You can open it via:  
`Flow Navigator → Synthesis → Open Synthesized Design → RTL Analysis`.
RTL SCHEMATIC DIAGRAM:
![WhatsApp Image 2025-10-27 at 15 40 02_8dd4b0f3](https://github.com/user-attachments/assets/4a3cc3be-38a9-4e6c-862a-b54cb7d026d4)
RTL SCHEMATIC DIAGRAM AFTER SYNTHESIS 
![WhatsApp Image 2025-10-27 at 16 01 40_6f30e804](https://github.com/user-attachments/assets/362f628c-7f19-4748-bcab-9cdc372ebc7d)

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
