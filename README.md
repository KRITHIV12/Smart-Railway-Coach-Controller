# Smart-Railway-Coach-Controller
# PROBLEM STATEMENT
"A silent energy thief on every train"

Every railway coach wastes precious electricity by keeping lights and fans running long after passengers leave. Despite Indian Railways’ push for energy-efficient LEDs and BLDC fans, this wasted consumption still creeps into the non-traction power bill. Yet, turning things off the moment a coach empties can feel abrupt or unsafe. The solution? A smart, presence-sensitive Verilog controller that switches systems on when people are aboard and turns them off only after a thoughtful delay — saving energy without compromising comfort.

# INTRODUCTION
In today’s railway systems, energy efficiency is not just about the wheels. Non-traction power, like lighting and ventilation in passenger coaches, accounts for a large share of electricity consumption. Indian Railways, for instance, spends billions annually to run lights and fans even when coaches sit empty. 
Occupancy sensors, such as PIR or ultrasonic devices, offer a powerful way to tackle this problem by detecting when people are present and switching systems on or off accordingly. While such sensors are common in buildings, their adoption inside train coaches is limited, leading to missed opportunities for energy savings.

This project proposes a smart, Verilog-based controller that brings presence-driven automation into railway coaches. It turns lights and fans ON when passengers are detected, and OFF only after a configurable delay once they leave a balancing energy savings with passenger comfort. By designing and simulating this controller using an FSM + timer logic, we aim to demonstrate a low-cost, efficient solution that could make rail travel greener and smarter.

# SOFTWARE/HARDWARE COMPONENTS
1. xilinx Vivado
2. FPGA Development Board / FPGA Chip
3. Peripheral Components (LEDS/FANS)
4. Sensors (PIR, ultrasonic)

# PROCEDURE
**1)Define Requirements & States**
Identify the inputs (presence, clock, reset), outputs (light_on, fan_on, display_state), and FSM states (e.g., IDLE, ACTIVE, WAIT_OFF).

**2)Draw State Diagram**
Create a visual state-transition diagram showing how you move between IDLE, ACTIVE, and WAIT_OFF based on the presence signal and timer events.

**3)Write Verilog Modules**
o	FSM module (state register + next-state logic + output logic) 
o	Timer/counter module for delay when switching off
o	Top-level module to integrate FSM and timer

**4)Develop Testbench**
Simulate scenarios: passenger enters, stays, leaves, and re-enters. Check FSM transitions, timer behavior, and outputs.

**5)Run Functional Simulation**
Use a Verilog simulator (like Icarus or ModelSim) and inspect waveforms. Verify that the FSM and timer behave correctly. 

**6)Synthesize / (Optional) Deploy**
If targeting FPGA, synthesize the design using a tool like Vivado. Verify resource usage and timing. 

**7)Test on Hardware (If Available)**
Load the design on an FPGA board and test using real or simulated input (e.g., a sensor) and output (LEDs, fans).

**8)Refine & Document**
Based on test results, tweak delay, state logic, or inputs. Document the design (state diagram, module architecture, testing procedure).

# PROGRAM
```
module coach_controller(
  input clk,
  input reset,
  input sensor_in,
  output reg light,
  output reg fan,
  output reg [1:0] display 
);
  parameter IDLE=2'b00, ACTIVE=2'b01, TIMER=2'b10;
  parameter DELAY_CONST=10; 

  reg [1:0] state, next_state;
  reg [15:0] timer;

  always @(posedge clk or posedge reset) begin
    if (reset) begin
      state <= IDLE;
      timer <= 0;
    end else begin
      state <= next_state;
      if(state == TIMER && timer != 0)
        timer <= timer - 1;
      else if (state == ACTIVE && !sensor_in)
        timer <= DELAY_CONST;
    end
  end
  always @(*) begin
    next_state = state;
    case(state)
      IDLE:   next_state = (sensor_in) ? ACTIVE : IDLE;
      ACTIVE: next_state = (!sensor_in) ? TIMER : ACTIVE;
      TIMER:  next_state = (sensor_in) ? ACTIVE :
                          (timer == 0) ? IDLE : TIMER;
     default: next_state = IDLE;
    endcase

    if (state == ACTIVE || state == TIMER) begin
      light = 1'b1;
      fan = 1'b1;
      display = 2'b01; 
    end else begin
      light = 1'b0;
      fan = 1'b0;
      display = 2'b00; 
    end
  end
endmodule

module testbench();
  reg clk, reset, sensor_in;
  wire light, fan;
  wire [1:0] display;

  coach_controller dut (
    .clk(clk),
    .reset(reset),
    .sensor_in(sensor_in),
    .light(light),
    .fan(fan),
    .display(display)
  );

  initial clk = 0;
  always #5 clk = ~clk;

  initial begin
    $monitor("Time=%0t sensor_in=%b light=%b fan=%b display=%b", $time, sensor_in, light, fan, display);
    reset = 1; sensor_in = 0; #12;
    reset = 0; #10;

    sensor_in = 1; #20;

    sensor_in = 1; #30;

    sensor_in = 0; #50;

    sensor_in = 1; #20;

    sensor_in = 0; #100;
    #50;
    $finish;
  end
endmodule
```
# OUTPUT
<img width="1039" height="964" alt="Screenshot 2025-11-18 090818" src="https://github.com/user-attachments/assets/1f45bc8d-cc22-484d-89fb-9f770195ef77" />

# EXPECTED RESULTS 
The FSM switches correctly: IDLE → ACTIVE on passenger presence, then ACTIVE → WAIT_OFF when they leave, and finally back to IDLE once the timer expires.

The timer module reliably counts the delay during WAIT_OFF, then signals “timer done.”

During ACTIVE and WAIT_OFF states, lights and fans remain ON; in IDLE, they turn OFF.

If a passenger returns before the timer ends, the system goes back to ACTIVE without turning off.

Simulation waveforms clearly show state transitions, the timer counter, and output behavior.

# CONCLUSION
This project demonstrates a practical and efficient energy-saving solution for railway coaches by using a Verilog-based controller that switches lights and fans ON only when passengers are detected, and OFF after a configurable delay. The design—built as a finite state machine (FSM) with timer logic—maintains comfort by preventing abrupt shutoffs, and avoids unnecessary power consumption. Simulation results verify correct state transitions, delay behavior, and output control, proving the viability of the approach. In the broader context, such a controller aligns well with Indian Railways’ non-traction energy-conservation initiatives.With further tuning and real-sensor integration, this system could be deployed on FPGA or embedded platforms, contributing to smarter and greener rail transportation.
