# FPGA-MPC

## Introduction
Model Predictive Control (MPC) is a feedback control strategy that has seen great success in many
robotic applications. MPC often leverages trajectory optimization (TO) on an objective
function, subject to system dynamics and other constraints. These TO problems have
hundreds/thousands of variables and must be solved in milliseconds to achieve real-time
performance. As such, careful approximations/simplifications of underlying numerical methods and
the optimal control problem are often used to enable real-time deployments on a variety of hardware
platforms.
This project, FPGA-MPC, is a fast quadratic programming solver for MPC, which is inspired by
recent works that leverage acceleration on GPUs and FPGAs for online trajectory optimization. In particular, TinyMPC [8] exploits the closed-form solution of LQR through Riccati
recursion and compresses the ADMM algorithm to enable high frequency control on
resource-constrained platforms. In this work we aim to accelerate TinyMPC’s algorithm in hardware
on an FPGA, unlocking better performance and capabilities for future MPC research.


## 2) System Design
<img width="2296" height="1186" alt="image" src="https://github.com/user-attachments/assets/8c12a9fc-0ca2-4530-aa8a-c5560c7537e1" />
Figure 1: Dataflow Diagram of FPGA-MPC

The quadrotor knows its current position but relies on external instructions on where to go next. It
sends its current position and orientation data as a 16-bit struct via ethernet to an external CPU.
The CPU stores the incoming parameters in memory. The CPU processes the drone’s position and
orientation data to compute the drone’s linear velocity, angular velocity, and other important
parameters. It converts this raw parameter data into a format that is friendly for the FPGA to read.
The CPU writes this formatted parameter data via an Avalon Bus interface into the FPGA’s Block
Random Access Memory (BRAM).

The FPGA’s job is to calculate the drone’s next position and orientation given the drone’s current
position and orientation. It does so using the Alternating Direction Method of Multipliers (ADMM)
algorithm, which will be described further in the following section. In order to reduce the number of
computations the FPGA’s memory. Helper functions simplify future calculations. After the ADMM
Algorithm finishes, the FPGA outputs a new set of controls in a 16-bit struct to its memory. This
data is written back to the CPU memory through the Avalon Bus. The CPU forwards the new
control actions via ethernet to the Quadrotor simulator.
