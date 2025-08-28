# Snipper

*Developed using Python by Heider Jeffer*

Educational Python projectile simulator with air drag, wind, and terrain slope. Run single simulations, sweep angles for max range, or search for target hits. Supports CSV export &amp; plotting. Great for sports balls, water rockets, and classroom physics demos.


---

## Introduction

This project is an **educational Python simulator** for projectile motion.
It models the flight of objects (like soccer balls, tennis balls, water rockets, or ping-pong balls) under the influence of **gravity, air drag, and wind**.

Unlike the simple textbook parabola, this simulator accounts for **real-world effects**:

* Air resistance that slows the object down,
* Wind that alters the path,
* Terrain slopes (uphill or downhill landings).

You can run single simulations, sweep through angles to find the **maximum range**, or search for an angle that makes the projectile land near a **target point**. The program can export full trajectories to CSV and plot them for visualization.

It’s designed for **students, hobbyists, and teachers** who want to explore the physics of motion safely and interactively.

---

## Motivation

Projectile motion is one of the first physics topics we learn, but the neat parabolas in textbooks ignore real-world effects like drag and wind. I built this simulator to **bridge the gap between theory and reality** — giving students, hobbyists, and teachers a tool to experiment with motion in a way that’s both **interactive and accurate**.

Whether you’re exploring how far a soccer ball can really travel, testing a water rocket design, or teaching physics concepts in class, this project makes it easy to visualize and understand how forces shape trajectories.

---

## High-Level Idea

This program simulates how an object (like a ball, water balloon, rocket, etc.) moves when launched at an angle.
It accounts for:

* **Gravity** pulling the object downward,
* **Air resistance (drag)** that slows it down,
* **Wind** that shifts the effective velocity,
* **Terrain slope** (flat ground, uphill, downhill).

It lets you:

1. Run one simulation,
2. Sweep through many launch angles to find the one that gives the longest range,
3. Search for an angle that makes the projectile land near a given target.

It can also:

* Save the trajectory into a CSV file,
* Plot the trajectory (if `matplotlib` is installed).

---

## Structure of the Program

The program is divided into **data classes**, **physics functions**, and **user interaction**.

---

### 1. Data Classes

```python
@dataclass
class ProjectileParams:
    speed: float
    angle_deg: float
    height: float = 0.0
    mass: float = 0.145
    area: float = 0.0042
    cd: float = 0.47
    air_density: float = 1.225
    wind_speed: float = 0.0
    dt: float = 0.002
    max_time: float = 60.0
    ground_slope_deg: float = 0.0
    ground_offset: float = 0.0
```

This class holds all **input parameters** (speed, angle, mass, drag coefficient, etc.).

```python
@dataclass
class Result:
    time: float
    range: float
    max_height: float
    impact_x: float
    impact_y: float
    trace: list
```

This stores the **output** of a simulation: flight time, distance, peak height, impact point, and the full trajectory.

---

### 2. Terrain

```python
def terrain_height_at(x, params):
    m = math.tan(math.radians(params.ground_slope_deg))
    return m * x + params.ground_offset
```

Ground is modeled as a **sloped line**: `y = mx + b`.

* `m = tan(slope)` → slope of terrain.
* `b = ground_offset` → vertical shift.

If slope = 0, ground is flat at `y = 0`.

---

### 3. Physics: Drag + Gravity

```python
def drag_accel(vx, vy, params):
    vrel_x = vx - params.wind_speed
    vrel_y = vy
    vrel = math.hypot(vrel_x, vrel_y)  # relative speed to air

    D = 0.5 * rho * Cd * A * v^2   # drag force
    ax = -D * (vrel_x / vrel) / m
    ay = -D * (vrel_y / vrel) / m
    return ax, ay
```

This calculates the **air drag acceleration** in both x and y directions.

---

### 4. Numerical Integration (RK4 method)

```python
def rk4_step(x, y, vx, vy, dt, params):
    ...
```

The motion equations are solved with the **Runge–Kutta 4th-order method (RK4)**.
This is much more accurate than the simple Euler method, especially when drag is involved.

Each step updates:

* Position `(x, y)`
* Velocity `(vx, vy)`

---

### 5. Simulation Loop

```python
def simulate(params, ...):
    while t < params.max_time:
        ...
        if crossed ground:
            interpolate to find exact impact point
            return results
        step forward in time
```

The loop:

1. Saves the current position,
2. Steps forward in time (`dt`),
3. Checks if the projectile hit the ground (crossed below terrain line),
4. If yes → interpolates the exact impact point,
5. If no → keeps going until `max_time`.

At the end, it returns a `Result` object with trajectory info.

---

### 6. Extra Features

* **Angle sweep** (`sweep_max_range`) → tries many angles, picks the one with the largest range.
* **Target finder** (`find_angle_to_hit_target`) → brute-force search for an angle that lands near a given (x,y).
* **CSV export** → saves trajectory to file.
* **Plotting** → optional, shows curve with terrain + target.

---

### 7. User Interaction

```python
def run_interactive():
    print("=== Projectile Simulator ===")
    # Prompts for inputs (speed, angle, mass, etc.)
    # Offers 3 choices:
    # 1) Single simulation
    # 2) Sweep angles
    # 3) Hit a target
```

* **Choice 1:** Run one simulation and print results.
* **Choice 2:** Sweep angles and find the one with max range.
* **Choice 3:** Search for an angle to hit a given target.

---

## Example Run

```
=== Projectile Simulator (Educational) ===
Launch speed (m/s) [30.0]:
Launch angle (degrees) [35.0]:
Launch height (m) [1.8]:
...
Choose action:
1) Single simulation
2) Sweep angles to find maximum range
3) Find angle to hit a target (search)
Select 1/2/3 [1]:
```

If you choose **1**, you’ll see:

```
Flight time:  3.82 s
Impact (x,y): (92.13 m, 0.00 m)
Horizontal range: 92.13 m
Max height: 16.40 m
```

---

## Why This Is Useful

* Teaches you how physics of projectile motion works,
* Lets you compare with/without drag and wind,
* Helps design safe experiments (water rockets, basketball arcs, etc.),
* Demonstrates numerical methods (RK4) in a real-world problem.

---
