# Inventory Discrete-Event Simulation

A discrete-event simulation of a warehouse inventory system, built with [SimPy](https://simpy.readthedocs.io/). The model simulates customer demand, an (s, S) reorder policy, order lead times, and tracks inventory levels, revenue, and holding/ordering costs over time.

## Project Overview

This project models a single-item inventory system under uncertainty:

- Customers arrive at random intervals (exponentially distributed interarrival times).
- Each customer generates a random demand, which is fulfilled from on-hand inventory (or partially fulfilled if stock runs out).
- When inventory drops below a reorder point (`order_cutoff`), a replenishment order is placed up to a target level (`order_target`).
- Orders arrive after a fixed lead time.
- The simulation tracks inventory level over time, revenue from sales, ordering costs, and holding costs, and produces a step plot of inventory level vs. time.

This is a classic (s, S) periodic-review inventory policy, implemented as an event-driven simulation rather than a fixed-time-step simulation, so computation only happens when something actually occurs (a customer arrival or an order delivery).

## Tech Stack

- **Python 3**
- **SimPy** — discrete-event simulation engine
- **NumPy** — random variate generation (demand, interarrival times)
- **Pandas** — data handling
- **Matplotlib** — visualization of inventory level over time
- **Jupyter Notebook** — development and experimentation environment

## Usage

Run the notebook cells in order. The core simulation is kicked off with:

```python
np.random.seed(15)
env = simpy.Environment()
env.process(warhouse_run(env, order_cutoff=5, order_target=30))
env.process(observe(env))
env.run(until=200)
```

- `order_cutoff` — inventory level that triggers a reorder
- `order_target` — inventory level to replenish up to
- `env.run(until=...)` — total simulated time horizon

After running, plot the inventory trajectory:

```python
import matplotlib.pyplot as plt
plt.figure(figsize=(10, 5))
plt.step(obs_time, inventory_level, where='post')
plt.xlabel('time')
plt.ylabel('inventory')
plt.show()
```

## Architecture

The simulation is built around SimPy's process-based event loop:

| Component | Responsibility |
|---|---|
| `warhouse_run(env, order_cutoff, order_target)` | Main process: handles customer arrivals, demand fulfillment, revenue/holding-cost accounting, and triggers reorders |
| `handle_order(env, order_target)` | Sub-process: places an order, waits for the lead time, then adds stock to inventory |
| `generate_interarrival_time()` | Draws exponential interarrival times between customers |
| `generate_demand()` | Draws random integer demand per customer |
| `observe(env)` | Periodically samples inventory level for later plotting |

State (`inventory`, `balance`, `num_ordered`) is currently tracked via module-level globals updated by each process. An earlier object-oriented design (a `Simulation` class using a manual event-clock loop) is included in the notebook as a commented-out reference implementation, showing the same logic expressed as an explicit next-event simulation.

## Examples

A sample run with `order_cutoff=5`, `order_target=30`, and a fixed random seed produces console output like:

```
0.31 sold 2
0.58 sold 1
1.02 sold 3
1.15 place order for 24
1.40 sold 1 (out of stock)
3.15 order received, 24 in inventory
```

...alongside a step plot showing inventory sawtoothing between the reorder point and the target level as demand depletes stock and periodic orders replenish it.
