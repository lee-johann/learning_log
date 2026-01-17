https://neuron.eng.wayne.edu/ECE330/Practical_Electronics_for_Inventors.pdf

Topics:
- Theory: voltage, current, resistance, capacitance, inductance 
- Discrete passive circuits (current limiting network, voltage divider, filter circuit, attenuators)
- Discrete active devices (diode, transistor, thyristor)
- Discrete circuits (amps, oscillator, modulators)
- Integrated Circuits
- IO devices
- Constructing circuits

Theory
- Current $I=\frac{dQ}{dt}$ represents the amount of electrical charge dQ (unit is Coulomb) crossing a cross-sectional area per unit time
	- unit is ampere A = C/s
	- "flows" from pos to neg, although electrons move from neg to pos
- When charge moves between charge distributions (2 points), its PE changes. This PE change is equivalent to the work done by the charge over a distance (remember W=Fd) 
	- Voltage is amount of energy required to move a unit of charge (PE / unit of charge) from 1 point to another
	- unit is volt V = J/C
- Resistance reduce current flow, $I=\frac{V}{R}$ 
	- unit is ohms = V/A
- when calculating voltage between two endpoints, visualize how the electrons flow (do they have multiple paths through batteries (higher current), or do they go through multiple batteries (higher voltage) )
- Components like LEDs "use up" voltage (work is done, converts away PE)
	- current is the same across all points in a single path circuit
	- voltage is the same across all branches of two connected nodes
		- so if there are 2 branches between A and B, voltage across both branches are same (I can differ if R differs)
		- treat sequential resistors as sums in circuit reduction
- Kirchoff's laws
	- $\Delta V = 0$ , the sum of voltage changes around a closed path is 0 (conservation of energy)
	- $I_{in}=I_{out}$ , the sum of currents that enter a junction equal the sum of currents that leave a junction (conservation of charge flow through a circuit)
- Thevenin equivalent circuit: in any linear dc circuit, you can take any two terminals and treat the rest of the circuit as a black box with some V and R
- 