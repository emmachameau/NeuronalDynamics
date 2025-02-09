Leaky-integrate-and-fire model
==============================

Neuronal dynamics can be conceived as a summation process combined with an action potential triggering mechanism. This mechanism triggers an action potential, or spike, when the neuron reaches a critical voltage, $$\vartheta$$.
In this first exercise, I explored the dynamics of the leaky integrate and fire (LIF) model. This model reduces spikes to 'events'; action potentials have roughly the same form, and hence information is transferred in the presence or absence of a spike. No attempts will be made to describe the shape of the action potential. 

We can thus reduce the model to 
1) a linear differential equation which describes the evolution of the membrane potential
2) a threshold for spiking.

In order to obtain an equation which links the momentary voltage **_u<sub>i</sub>(t) - u<sub>rest</sub>_** of neuron **_i_** to the input current **_I(t)_**, we can regard the neuron as an electrical circuit. If a short current pulse I(t) is injected into the neuron, the additional electrical charge will charge the cell membrane. The cell membrane therefore acts like a capacitor of capacity **_C_**. Because the insulator is not perfect, the charge will, over time, slowly leak through the cell membrane. The cell membrane can therefore be characterized by a finite leak resistance **_R_**. 

![image](https://github.com/user-attachments/assets/11d4f186-73cd-4950-88ca-38c104d6dcee)

Eventually, this leads to the following equation: 

$$\tau_m \frac{du}{dt} = -\left[u(t) - u_{\text{rest}}\right] + R I(t)$$

Where $$tau_m$$ is the membrane time constant, u(t) is the membrane potential at time t, $$u_rest$$ is the membrane potential at rest, R is the membrane resistance and I(t) is the input current at time t.

Furthermore, the membrane potential is reset after each output spike: 

$$if u(t) = \vartheta then \lim_{\delta \to 0; \delta>0} u(t+\delta) = u_{rest}$$

1.1. Exercise: minimal current
==============================

```python
import brian2 as b2
import matplotlib.pyplot as plt
import numpy as np
np.set_printoptions(precision=30)
from neurodynex3.leaky_integrate_and_fire import LIF
from neurodynex3.tools import input_factory, plot_tools
```

In the absence of an input current, a LIF neuron has a constant membrane voltage V_REST. If an input current drives vm above the firing threshold, a spike is generated. Then, vm is reset to V_RESET and the neuron ignores any input during the refractory period.

**1.1.1. Question: minimal current (calculation)**

_For the default neuron parameters (see above), compute the minimal amplitude Imin of a step current to elicitate a spike._

**Input**

```python

print('Resting potential: {}'.format(LIF.V_REST))
print('Firing threshold: {}'.format(LIF.FIRING_THRESHOLD))
print('Membrane resistance: {}'.format(LIF.MEMBRANE_RESISTANCE))
I_min = (LIF.FIRING_THRESHOLD - LIF.V_REST) / LIF.MEMBRANE_RESISTANCE
print('The minimal current for firing is: {}'.format(I_min))
```
The code I have written above calculates the minimal current for firing. 

**Output**
```Resting potential: -70. mV
Firing threshold: -50. mV
Membrane resistance: 10. Mohm
The minimal current for firing is: 2.0000000000000004 nA
```

**1.1.2. Question: minimal current (simulation)**

_Use the value Imin you’ve computed and verify your result: inject a step current of amplitude Imin for 100ms into the LIF neuron and plot the membrane voltage. vm should approach the firing threshold but not fire._

**Input**
```python

# create a step current with amplitude = I_min
step_current = input_factory.get_step_current(
    t_start=5, t_end=100, unit_time=b2.ms,
    amplitude=I_min)

# run the LIF model.
(state_monitor,spike_monitor) = LIF.simulate_LIF_neuron(input_current=step_current, simulation_time = 100 * b2.ms)

# plot I and vm
plot_tools.plot_voltage_and_current_traces(
state_monitor, step_current, title="min input", firing_threshold=LIF.FIRING_THRESHOLD)
print("nr of spikes: {}".format(spike_monitor.count[0]))  # should be 0
```

**Output**
```
nr of spikes: 0
```
![image](https://github.com/user-attachments/assets/d7c11d87-a0aa-4c5c-a0ca-0ddd40271140)

Clearly, we see that vm approaches the firing threshold, but does not fire. 

1.2. Exercise: f-I Curve
==============================
For a constant input current I, an LIF neuron fires regularly with a firing frequency of f. If the current is to small (I<Imin) f is 0Hz; for larger I the rate increases. A neuron’s firing-rate versus input-amplitude relationship is visualized in an “f-I curve”.

**1.2.1. Question: f-I Curve and refractoriness**

We now study the f-I curve for a neuron with a refractory period of 3ms.

_Inject currents of different amplitudes (from 0nA to 100nA) into a LIF neuron. For each current, run the simulation for 500ms and determine the firing frequency in Hz. Then plot the f-I curve. Pay attention to the low input current._

**Input**

```python
current_amp_list = np.arange(0, 100, 10) 
firing_rate_record = list()
refractory_period = 3 * b2.ms

for current_amp in current_amp_list:
    step_current = input_factory.get_step_current(
               t_start=10, t_end=490, unit_time=b2.ms,
               amplitude = current_amp * b2.mA
               )
    
    (state_monitor, spike_monitor) = LIF.simulate_LIF_neuron(
                                 input_current=step_current,
                                 simulation_time=500 * b2.ms,
    abs_refractory_period = refractory_period
                                 )

    spike_count = spike_monitor.count[0]

    firing_rate = spike_count / (480 * 10**-3) # current injected for 95ms
    firing_rate_record.append(firing_rate)

plt.figure()
plt.plot(current_amp_list, firing_rate_record)
plt.xlabel('Current (mA)')
plt.ylabel('Firing rate (Hz)')
plt.show()
```

**Output**

![image](https://github.com/user-attachments/assets/ebd819ce-8759-443a-90a8-9c793f47d23e)

This f-I curve shows that firing frequency increases linearly with the input current amplitude. However, beyond a certain current the firing rate plateaus as the maximum firing rate is reached, limited by the refractory period. 

1.3. Exercise: “Experimentally” estimate the parameters of a LIF neuron
==============================

_A LIF neuron is determined by the following parameters: Resting potential, reset voltage, firing threshold, membrane resistance, membrane time-scale, absolute refractory period. By injecting a known test current into a LIF neuron (with unknown parameters), you can determine the neuron properties from the voltage response._

**1.3.1. Question: “Read” the LIF parameters out of the vm plot**
1. Get a random parameter set.
2. Create an input current of your choice.
3. Simulate the LIF neuron using the random parameters and your test-current. Note that the simulation runs for a fixed duration of 50ms.
4. Plot the membrane voltage and estimate the parameters. You do not have to write code to analyse the voltage data in the StateMonitor. Simply estimate the values from the plot. For the membrane resistance and the membrane time-scale you might have to change your current.
5. Compare your estimates with the true values.

**Input**
```python
# get a random parameter. provide a random seed to have a reproducible experiment
random_parameters = LIF.get_random_param_set(random_seed=433)

# define your test current
test_current = input_factory.get_step_current(
    t_start=10, t_end=30, unit_time=b2.ms, amplitude= 15 * b2.namp)

# probe the neuron. pass the test current AND the random params to the function
state_monitor, spike_monitor = LIF.simulate_random_neuron(test_current, random_parameters)

# plot
plot_tools.plot_voltage_and_current_traces(state_monitor, test_current, title="experiment")

# print the parameters to the console and compare with your estimates
LIF.print_obfuscated_parameters(random_parameters)
```

**Ouput**

![image](https://github.com/user-attachments/assets/31eee9bb-17cb-4a72-a62a-8a72d60a1be1)

I found the resting potential by taking the minimum of the graph: 

```python
print('Resting potential:', min(state_monitor.v[0]))
```

The reset potential is the membrane potential to which the neuron resets immediately after a spike. To obtain this, I took the membrane potential around 26 ms (during the reset potential): 

```python
state_monitor.t
indexOfInterest = np.where(state_monitor.t == 26.1 * b2.msecond)[0] # 30 ms
print('Reset potential:', state_monitor.v[0][indexOfInterest])
```

This gave me a correct reset potential of -64 mV. 

I estimated the refractory period with the graph. I increased the input current to 100 to obtain repetitive firing. 

![image](https://github.com/user-attachments/assets/be48703c-96f5-4c8e-9435-7dd1e221d10f)

The refractory period seemed to be around 5 ms. 

To approximate the firing threshold I took the maximum of the membrane voltage. 

```python
print('Firing threshold:', max(state_monitor.v[0]))
```
This gave me a nice approximation of -36.35 mV. 

Next, the membrane resistance can be obtained with Ohm's law:

```
\Delta V = I \cdot R_m,
```

However, there is a problem: the neuron will spike upon reaching its firing threshold. Because of this, the neuron will not reach a plateau. It is then possible that the membrane potential is reset before reaching the maximum voltage. One way to fix this is to saturate the neuron with a small current. This way, the maximum neuron potential is below the firing threshold. Hence, I set the input current to 1 nA and simulated for long enough to allow the neuron to saturate. 

**Input**

```python
# get random parameter
random_parameters = LIF.get_random_param_set(random_seed=433)

# define test current 
test_current = input_factory.get_step_current(
t_start=5, t_end=45, unit_time=b2.ms, amplitude=1*b2.namp)

# probe the neuron. pass the test current AND the random parames to the function 
state_monitor, spike_monitor = LIF.simulate_random_neuron(test_current, random_parameters)

# plot 
plot_tools.plot_voltage_and_current_traces(state_monitor, test_current, title='experiment')

```

**Output**

![image](https://github.com/user-attachments/assets/c6545430-5aea-4cf3-a8ff-3ae010d7425f)


Then, I calculated the membrane resistance as follows: 

```python
# Ohm's Law

max_v = np.max(state_monitor.v[0]) 
rest_v = np.min(state_monitor.v[0])

input_current = 1 * b2.nA 
delta_v = (max_v) - (reset_v)  
R = delta_v / input_current 

print("Membrane resistance: %.2f MΩ" % (R / b2.Mohm))
```

This gave me a membrane resistance of 1.82 MΩ, which I thought was a nice approximation. 

Next, I estimated the membrane time constant by substituting it into the equation of the passive membrane model. 

$$u(t) = v_{rest} + RI(1-e^{-(t-t_{0})/\tau})$$
$$u(t) = v_{rest} + RI(1-e^{-1})$$
$$u(t) \cong v_{rest} + 0.63RI$$

I obtained: v(t) = -69 mV + 0.63 * 1.82 MΩ * 1 nA = -67.85 mV

This gave me the target membrane voltage which I then used to find the time where state_monitor.v[0] == v_t _aka_ the time constant.

```python
rest_v = -69
max_v = -67.18
delta_v = max_v - rest_v
v_t = rest_v + 0.63 * delta_v
print('Target v_t: %.2f mV' % v_t)

v_t_index = np.where(np.round(state_monitor.v[0], 4) == np.round(v_t, 1) * b2.mV)

tau = state_monitor.t[v_t_index[0][0]] # pick the first match

print('Time constant:', tau)
```

This gave me a time constant of 17.7 ms.

Finally, to find the firing threshold I took the maximum of 




