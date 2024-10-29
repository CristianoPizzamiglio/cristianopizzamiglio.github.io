---
layout: post
title: "PyChrono: Multi-Physics Simulation in Python"
date: 2024-10-12 00:00:00 -0400
---

[PyChrono](https://projectchrono.org/pychrono/) is a Python library that wraps the [Chrono](https://projectchrono.org/) C++ multi-physics simulation library. PyChrono can be used together with popular Python libraries, such as NumPy for postprocessing, and TensorFlow for developing systems powered by neural networks.

[Prof. Alessandro Tasora](https://www.linkedin.com/in/alessandrotasora/), from Università di Parma, started the development of the Chrone engine in 1998 when he was a student at Politecnico di Milano. Other key developers include [Prof. Dan Negrut](https://sbel.wisc.edu/negrut-dan/), and [Distinguished Scientist Radu Serban](https://sbel.wisc.edu/radu-serban/), both from the University of Wisconsin-Madison, and [Prof. Mihai Anitescu](https://web.cels.anl.gov/~anitescu/) from University of Chicago. The development efforts have also been supported by funds from various sources. For instance, in 2014 the US Army invested $1.8 million over a two-year period, and $1.0 million were invested by the National Science Foundation from 2019 to 2023.

Chrono was released as [open-source](https://github.com/projectchrono) in 2013 under a [BSD-3 license](https://choosealicense.com/licenses/bsd-3-clause/), and the name Project Chrono was adopted. For simplicity, the term Chrono will be used throughout this post.

As stated in its website, Chrono allows to
> simulate, for instance, wheeled and tracked vehicles operating on deformable terrains, robots, mechatronic systems, compliant mechanisms, and fluid solid interaction phenomena. Systems can be made of rigid and flexible/compliant parts with constraints, motors and contacts; parts can have three-dimensional shapes for collision detection.

A [rich gallery of videos](https://projectchrono.org/gallery/) showcases the amazing projects developed with Chrono. Three videos are embedded below.
* Three lunar rovers interacting with granular material under Moon gravity.
<div style="text-align: center;">
<iframe width="640" height="360" src="https://www.youtube.com/embed/Oi9gBdyUirg" title="Bulldozing simulation of Moon VIPER rovers on a crater with a straight blade under Earth gravity" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</div>

* Autonomous vehicle which uses simulated LIDAR data to navigate a closed course without prior knowledge of the track.
<div style="text-align: center;">
<iframe width="640" height="360" src="https://www.youtube.com/embed/BykI_evlxO4" title="1/6th Scale Autonomous Car Simulation in Chrono" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</div>

* Tibiofemoral joint simulation during the gait motion. The articular surfaces are modeled with nonlinear shell elements.
<div style="text-align: center;">
<iframe width="640" height="360" src="https://www.youtube.com/embed/z-1H5csiAKc" title="126 Tibiofemoral Joint Simulation" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</div>

Creating from scratch a multibody model with PyChrono could be a little intimidating at first. With this post I would like to guide you through the development of a simple four-wheeled vehicle, where each wheel operates independently and is powered by its own motor. The code repository is available [here](https://github.com/CristianoPizzamiglio/multibody-dynamics-pychrono) and can serve as a template for developing more complex models. The codebase is well-structured into several modules and the object-oriented programming paradigm (OOP) is adopted. Moreover, type hints are provided to improve code readability and maintainability. Therefore, basic knowledge of the Python programming language and OOP is required.

The content of this post has also been transformed into an AI-generated podcast, powered by the amazing [Google NotebookLM](https://notebooklm.google/). The podcast is available [here](https://notebooklm.google.com/notebook/e259b315-8516-4a23-8603-d56bf3c81d5c/audio).

To get started, PyChrono can be easily installed using [conda](https://docs.conda.io/projects/conda/en/latest/user-guide/getting-started.html), and all the installation details are available [here](https://api.projectchrono.org/pychrono_installation.html). Moreover, the repository includes the `environment.yml` file for creating the conda environment.

The structure of the repository is as follows:

```plaintext
repo/
├── config/
│   ├── simulation_params.yaml
│   └── vehicle_params.yaml
├── images/
│   └── motion.gif
├── src/
│   ├── main.py
│   ├── plotter.py
│   ├── simulation.py
│   ├── simulation_scenarios.py
│   ├── utils.py
│   └── vehicle.py
├── .gitignore
├── environment.yml
├── LICENSE
└── README.md
```

The `config` directory stores [yaml](https://en.wikipedia.org/wiki/YAML) configuration files. These files allow to set the values of simulation parameters (e.g. total simulation time) and vehicle parameters (e.g. wheel dimensions).

The source code (`src`) folder contains the following modules:
* `main.py`: the entry point. Once the conda environment is set up, the simulation can be run from this file.
* `plotter.py`: for plotting torque values using the [matplotlib](https://matplotlib.org/) library.
* `simulation.py`: for setting up the simulation and collecting simulation results.
* `simulation_scenarios.py`: for modelling different simulation scenarios, such as moving on a straight line at constant speed.
* `utils.py`: a collection of utility functions used by the other modules.
* `vehicle.py`: for modelling the components of the vehicle, i.e., chassis, wheels, etc.

Let's dive deeper into the details of the modules listed above.

The `vehicle.py` contains the `Vehicle` class, which includes five computed attributes: one representing the chassis and the four for modelling the mobility subsystems. As shown in the code below, the chassis is modelled as a box by using the `chrono.ChBodyEasyBox` class and its dimension and mass can be set in the `vehicle_params.yaml` file. For further details about the Chrono modules, classes, and other components, please refer to the [reference manual](https://api.projectchrono.org/development/manual_root.html).

```python
@cached_property
def chassis(self) -> chrono.ChBodyEasyBox:
    density = utils.compute_box_density(
        self._params.chassis.length,
        self._params.chassis.width,
        self._params.chassis.height,
        self._params.chassis.mass,
    )
    chassis = chrono.ChBodyEasyBox(
        self._params.chassis.length,
        self._params.chassis.width,
        self._params.chassis.height,
        density,
        True,
    )
    chassis.SetPos(
        chrono.ChVectorD(
            self._params.chassis.x_cg,
            self._params.chassis.y_cg,
            self._params.chassis.z_cg,
        )
    )
    chassis.GetVisualShape(0).SetColor(chrono.ChColor(0.8, 0.8, 0.8))
    chassis.SetName("Chassis")
    return chassis
```
The inertia properties of the `chrono.ChBodyEasyBox` are automatically set based on the density value provided as input. The density is computed by the `compute_box_density` utility function. The box is created at the center of mass, which can be translated using the `SetPos` method. The translation is defined by the `ChVectorD` vector, and its components can be modified in the `vehicle_params.yaml` file. Additionally, the appearance properties of the box can be modified as well with the `SetColor` method.

The wheel-hub-motor assembly is modelled by the `MobilitySubsystem` class. Both the wheel and hub are instances of the `chrono.ChBodyEasyCylinder` class.

```python
@cached_property
def wheel(self) -> chrono.ChBodyEasyCylinder:
    """
    Wheel.

    Returns
    -------
    chrono.ChBodyEasyCylinder

    """
    density = utils.compute_cylinder_density(
        self._params.wheel.radius,
        self._params.wheel.width,
        self._params.wheel.mass,
    )
    wheel = chrono.ChBodyEasyCylinder(
        chrono.ChAxis_Y,
        self._params.wheel.radius,
        self._params.wheel.width,
        density,
        True,
        True,
        self._material,
    )
    wheel.GetVisualShape(0).SetColor(chrono.ChColor(0.0, 0.0, 0.0))
    wheel.GetVisualShape(0).SetOpacity(0.8)
    wheel.SetPos(
        chrono.ChVectorD(
            self._wheel_x_cg, self._wheel_y_cg, self._params.wheel.z_cg
        )
    )
    wheel.SetName("Wheel")
    return wheel
```
Once again, the dimensions, position, and mass of the wheel and hub can be set in the `vehicle_params.yaml` file. The `material` parameter is an instance of the `chrono.ChMaterialSurface`, which allows to set up the wheel-ground contact forces. Two main types of contact can be defined:
* `ChMaterialSurfaceNSC`:  non-smooth (complementarity) contact method.
* `ChMaterialSurfaceSMC `:  smooth (penalty) contact method.

The `chrono.ChLinkLockRevolute` class allows to define the wheel-hub revolute joint. The revolute joint coordinate system is an instance of the `chrono.ChCoordsysD` and its position and orientation are defined by a translation vector and a [quaternion](https://en.wikipedia.org/wiki/Quaternions_and_spatial_rotation) (`chrono.Q_from_AngAxis`) respectively. The joint must be initialized by specifying the two bodies to be connected, i.e. wheel and hub, and the joint coordinate system.
```python
@cached_property
def wheel_to_hub_revolute_joint(self) -> chrono.ChLinkLockRevolute:
    """
    Wheel-hub revolute joint.

    Returns
    -------
    chrono.ChLinkLockRevolute

    """
    y_cg = self._wheel_y_cg - self._params.wheel.width / 2
    revolute_csys = chrono.ChCoordsysD(
        chrono.ChVectorD(self._wheel_x_cg, y_cg, self._params.wheel.radius),
        chrono.Q_from_AngAxis(chrono.CH_C_PI_2, chrono.VECT_X),
    )
    revolute_joint = chrono.ChLinkLockRevolute()
    revolute_joint.Initialize(self.wheel, self.hub, revolute_csys)
    return revolute_joint
```

The wheel motor is modelled by leveraging the `chrono.ChLinkMotorRotationSpeed` class, which enforces an angular speed \\(\omega(t)\\) between the wheel and hub. The angular speed is defined as a function using the `chrono.ChFunction` class.
```python
@cached_property
def driving_motor(self) -> chrono.ChLinkMotorRotationSpeed:
    """
    Driving motor.

    Returns
    -------
    chrono.ChLinkMotorRotationSpeed

    """
    y_cg = self._wheel_y_cg - self._params.wheel.width / 2
    revolute_csys = chrono.ChFrameD(
        chrono.ChVectorD(self._wheel_x_cg, y_cg, self._params.wheel.radius),
        chrono.Q_from_AngAxis(chrono.CH_C_PI_2, chrono.VECT_X),
    )
    motor = chrono.ChLinkMotorRotationSpeed()
    motor.Initialize(self.wheel, self.hub, revolute_csys)
    motor.SetSpeedFunction(self._angular_velocity_function)
    return motor
```

Each mobility subsystem has an additional independent motor, whose axis is perpendicular to the ground, allowing the steering of the vehicle. In this case, the motor is an instance of the `chrono.ChLinkMotorRotationAngle` class that enforces a rotation angle \\(\alpha(t)\\) between the chassis and the mobility subsystem. The motion law is again defined as an instance of the `chrono.ChFunction` class.
```python
@cached_property
def steering_motor(self) -> chrono.ChLinkMotorRotationAngle:
    """
    Steering motor.

    Returns
    -------
    chrono.ChLinkMotorRotationAngle

    """
    chassis_length = self._params.chassis.length
    chassis_width = self._params.chassis.width
    x_cg = chassis_length / 2 if self._is_front else -chassis_length / 2
    y_cg = chassis_width / 2 if self._is_left else -chassis_width / 2
    revolute_csys = chrono.ChFrameD(
        chrono.ChVectorD(x_cg, y_cg, self._params.chassis.z_cg),
        chrono.ChQuaternionD(1, 0, 0, 0),
    )
    motor = chrono.ChLinkMotorRotationAngle()
    motor.Initialize(self.hub, self._chassis, revolute_csys)
    motor.SetAngleFunction(self._steering_angle_function)
    return motor
```

Both the `Vehicle` and `MobilitySubsystem` classes have a `add_to_system` method, which adds all vehicle components to an instance of the `chrono.ChSystem`. The latter is the main simulation object used to collect:
* the simulation model itself, through the `ChAssembly` object;
* the solver and the timestepper;
* the collision system;
* system-wise parameters (time, gravitational acceleration, etc.)
* counters and timers; as well as other secondary features.

The `system` instance is defined in the `main` function of the `main.py` module, along with the `material`, `vehicle`, and `ground` objects. The gravity vector is specified using the `Set_G_acc` method of the system instance.
```python
def main(vehicle_params: SimpleNamespace, simulation_params: SimpleNamespace) -> None:
    """
    Entry point.

    Parameters
    ----------
    vehicle_params : SimpleNamespace
    simulation_params : SimpleNamespace

    """
    simulation_scenario = get_simulation_scenario(vehicle_params, simulation_params)
    system = chrono.ChSystemNSC()
    material = chrono.ChMaterialSurfaceNSC()
    material.SetFriction(simulation_scenario.friction)

    ground = utils.create_ground(material, z_offset=0.0)
    vehicle = Vehicle(vehicle_params, material, simulation_scenario)

    system.Set_G_acc(simulation_scenario.gravity_vector)
    system.Add(ground)
    vehicle.add_to_system(system)

    visual_system = create_visual_system(system)

    simulation = Simulation(system, visual_system, vehicle, simulation_scenario)
    if simulation_params.mode == 0:
        view_model(visual_system)
    else:
        simulation.run()
        plotter = Plotter(simulation.results)
        plotter.plot_driving_motor_torques()
        plotter.plot_driving_motor_torques_smoothed()
        plotter.plot_steering_motor_torques()
        plotter.plot_steering_motor_torques_smoothed()
```

The `ground` object is modelled as a `chrono.ChBodyEasyBox` by the `create_ground` utility function.
```python
def create_ground(
    material: chrono.ChMaterialSurface, z_offset: float = 0.0
) -> chrono.ChBodyEasyBox:
    """
    Create ground.

    Parameters
    ----------
    material : chrono.ChMaterialSurface
    z_offset : float
        The default is 0.0.

    Returns
    -------
    chrono.ChBodyEasyBox

    """
    ground = chrono.ChBodyEasyBox(50.0, 2.0, 0.001, 1000, True, True, material)
    ground.SetPos(chrono.ChVectorD(0.0, 0.0, z_offset))
    ground.SetBodyFixed(True)
    ground.SetName("Ground")
    ground.GetVisualShape(0).SetTexture(
        chrono.GetChronoDataFile("textures/concrete.jpg")
    )
    return ground
```

Various simulation scenarios are provided by the `SimulationScenarios` class, such as straight line motion at constant speed. Each scenario is an instance of the `SimulationScenario` dataclass, whose attributes are, for instance, solver settings and motion law functions.
```python
@dataclass
class SimulationScenario:
    """
    Simulation scenario.

    Notes
    -----
    angle : deg
    angular_velocity : rad/s
    gravity : m/s²

    """

    angular_velocity_left_front: chrono.ChFunction
    angular_velocity_right_front: chrono.ChFunction
    angular_velocity_left_rear: chrono.ChFunction
    angular_velocity_right_rear: chrono.ChFunction
    steering_angle_left_front: chrono.ChFunction = chrono.ChFunction_Const(0.0)
    steering_angle_right_front: chrono.ChFunction = chrono.ChFunction_Const(0.0)
    steering_angle_left_rear: chrono.ChFunction = chrono.ChFunction_Const(0.0)
    steering_angle_right_rear: chrono.ChFunction = chrono.ChFunction_Const(0.0)
    are_towing_wheel: bool = False
    slope: float = 0.0
    friction: float = 0.7
    damping: float = 10.0
    total_time: float = 10.0
    time_step: float = 5.0e-3
    gravity_vector: chrono.ChVectorD = field(init=False)
    gravity: float = 9.81

    def __post_init__(self) -> None:
        self.gravity_vector = self._compute_gravity_vector()

    def _compute_gravity_vector(self) -> chrono.ChVectorD:
        """
        Compute gravity vector (constant-slope ramp).

        Returns
        -------
        chrono.ChVectorD

        """
        slope = np.deg2rad(self.slope)
        x = self.gravity * np.sin(slope)
        z = self.gravity * np.cos(slope)
        return chrono.ChVectorD(-x, 0.0, -z)
```

The `main.py` module incudes also the `create_visual_system` function, which allows to play an animation using the [Irrlicht realtime 3D engine](https://irrlicht.sourceforge.io/).

```
def create_visual_system(system: chrono.ChSystem) -> irr.ChVisualSystemIrrlicht:
    """
    Create visual system.

    Parameters
    ----------
    system : chrono.ChSystem

    Returns
    -------
    irr.ChVisualSystemIrrlicht

    """
    visual_system = irr.ChVisualSystemIrrlicht()
    visual_system.AttachSystem(system)
    visual_system.SetWindowSize(user32.GetSystemMetrics(0), user32.GetSystemMetrics(1))
    visual_system.Initialize()
    visual_system.AddSkyBox()
    visual_system.AddCamera(chrono.ChVectorD(0, 0, 1))
    visual_system.AddTypicalLights()
    visual_system.ShowExplorer(True)
    visual_system.EnableBodyFrameDrawing(True)
    return visual_system
```
A gif of the vehicle moving at constant speed on a straight line is showed below:

<center><img src="/img/posts/2024-10-12-mbd-pychrono/motion.gif" width="700"/></center>

Finally, configuring the `mode` parameter to 0 in the `simulation_params.yaml` file allows the model to be displayed during construction without starting the simulation.

I hope this post was helpful and encourages you to develop your own amazing multibody models! For further assistance on the repository, feel free to contact me.
