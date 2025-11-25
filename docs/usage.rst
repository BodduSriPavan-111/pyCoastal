Usage
=====

Quick start
-----------

After installation

.. code-block:: bash

   pip install pyCoastal

you can run one of the shipped examples from the command line

.. code-block:: bash

   pycoastal run examples/shallow_water_1d.yml

This command reads the YAML configuration, integrates the shallow–water equations, and writes the model state to a NetCDF file in the output directory defined in the config.


Basic Python example
--------------------

The same example can be run programmatically from Python

.. code-block:: python

   import pyCoastal as pc

   # 1. Load configuration
   cfg = pc.load_config("examples/shallow_water_1d.yml")

   # 2. Build grid and initial condition
   grid = pc.Grid1D.from_config(cfg)
   state0 = pc.initial_conditions.sine_wave(grid, cfg)

   # 3. Create time integrator and register RHS and boundary handlers
   rhs = pc.rhs.shallow_water_1d(cfg)
   bc  = pc.boundary.periodic_1d(grid)

   integrator = pc.TimeIntegrator(
       grid=grid,
       rhs=rhs,
       boundary_handler=bc,
       dt=cfg["time"]["dt"],
       t_end=cfg["time"]["t_end"],
   )

   # 4. Run the simulation
   history = integrator.run(state0)

   # 5. Export results to xarray for analysis and plotting
   ds = history.to_xarray()
   print(ds)

This example integrates the one–dimensional shallow–water equations on a uniform grid using explicit time stepping. The YAML file documents the physical parameters (gravity, depth), numerical settings (grid spacing, time step, CFL target), and output frequency.
