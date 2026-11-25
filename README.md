# 2D Laminar Channel Flow in OpenFOAM 14

A simple 2D channel flow case that I set up and ran in OpenFOAM 14 on Ubuntu. The channel is 4 m long and 2 m high, water-like fluid enters from the left at a constant velocity, and the walls on the top and bottom are no-slip. It's a basic case, but it's a good way to get comfortable with the full OpenFOAM workflow: building a mesh with `blockMesh`, setting boundary conditions, running the solver and looking at the results in ParaView.

The geometry comes from the "2D Channel Flow" slide in the FOSSEE / Spoken Tutorial OpenFOAM series. I followed the same layout but ran it as a laminar case using the OpenFOAM 14 solver setup.

## What the case looks like

| Item | Value |
|---|---|
| Domain | 4 m (x) by 2 m (y), 0.1 m thick in z |
| Mesh | 80 x 40 x 1 cells (one cell in z, so it's 2D) |
| Inlet | fixed velocity, U = (1 0 0) m/s, zero-gradient pressure |
| Outlet | fixed pressure, p = 0, zero-gradient velocity |
| Walls | no-slip, zero-gradient pressure |
| Front and back | `empty` (this is what makes it 2D) |
| Kinematic viscosity | nu = 0.01 m²/s |
| Flow regime | laminar |
| Reynolds number | about 200 (U x H / nu = 1 x 2 / 0.01) |
| Solver | `foamRun` with the `incompressibleFluid` module |
| Time | 0 to 20 s, deltaT = 0.01 s, written every 1 s |

The time step gives a Courant number of roughly 0.2 to 0.26, so the run is stable.

## A note about OpenFOAM 14

If you've seen older tutorials, a few things are different in version 14, and they tripped me up at first:

- `icoFoam` is gone. You run `foamRun` and set `solver incompressibleFluid;` in `controlDict`.
- `transportProperties` is now `physicalProperties`.
- The turbulence file is `momentumTransport`, and for this case it says `simulationType laminar;`.
- The cavity tutorial lives at `$FOAM_TUTORIALS/incompressibleFluid/cavity`. There is no extra `cavity/cavity` folder inside it.

## Repository structure

```
openfoam_2D_channel_flow/
├── README.md
├── mesh.txt                        # system/blockMeshDict
├── velocity.txt                    # 0/U
├── pressure.txt                    # 0/p
├── viscosity.txt                   # constant/physicalProperties
├── laminar flow.txt                # constant/momentumTransport
├── solver settings.txt             # system/controlDict
├── run commands.txt                # commands used, in order
├── pressure_contour.png
├── velocity_magnitude_contour.png
└── velocity_vector.png
```

The `.txt` files hold the contents of the OpenFOAM dictionaries, saved with simple names so they're easy to read on GitHub. The comment next to each one shows which real file it belongs to.

## How to run it

Make sure OpenFOAM 14 is sourced first. You can check with `echo $WM_PROJECT_VERSION`.

**1. Copy the tutorial as a starting point**

```bash
mkdir -p $FOAM_RUN && cd $FOAM_RUN
cp -r $FOAM_TUTORIALS/incompressibleFluid/cavity channel
cd channel
```

**2. Replace the files**

Paste the contents of each `.txt` file from this repo into the matching OpenFOAM file:

| Repo file | Goes into |
|---|---|
| `mesh.txt` | `system/blockMeshDict` |
| `velocity.txt` | `0/U` |
| `pressure.txt` | `0/p` |
| `viscosity.txt` | `constant/physicalProperties` |
| `laminar flow.txt` | `constant/momentumTransport` |
| `solver settings.txt` | `system/controlDict` |

The cavity tutorial also leaves `k`, `epsilon`, `omega`, `nut` and `nuTilda` in the `0` folder. You can ignore them, since laminar mode doesn't read them.

**3. Mesh and run**

```bash
blockMesh
checkMesh
foamRun
```

**4. Open in ParaView**

```bash
touch case.foam
paraview case.foam
```

In ParaView, click the last-frame button before taking screenshots. It opens at t = 1 s by default, and the flow is still developing at that point.

## Results

**Pressure** (`pressure_contour.png`)
Pressure is highest at the inlet and drops towards zero at the outlet, which is what you'd expect when the outlet is fixed at p = 0 and the flow is pushed through the channel. There are small high-pressure spots at the two inlet corners where the incoming uniform flow hits the walls and has to slow to zero.

**Velocity magnitude** (`velocity_magnitude_contour.png`)
Velocity starts out uniform at the inlet. Moving downstream, the no-slip walls slow the fluid next to them, and the boundary layers slowly grow in towards the centreline, which speeds up to make up for it.

**Velocity vectors** (`velocity_vector.png`)
Vectors show the flow going left to right along the channel, getting shorter close to the walls.

**Convergence**
Continuity errors in the log came down to around 1e-11 by the end of the run, and the solver needed only one or two iterations per step for pressure, so the solution is well converged for this setup.

## Things to keep in mind

- The channel is only 4 m long. At Re of about 200 the entrance length is far longer than that (on the order of 20 m), so the profile at the outlet is **not** fully developed parabolic flow. It's still evolving, and that's expected.
- Mesh is 80 x 40, which is fine for a first run. I haven't done a grid independence study, so I wouldn't read the numbers too closely.
- This is a laminar case only. For turbulent flow you'd need to switch to a RAS model such as k-epsilon and set up `k`, `epsilon` and `nut` properly.

## Ideas for next steps

- Plot velocity profiles at x = 0.5, 1, 2 and 3.9 m with Plot Over Line to see the profile develop.
- Plot pressure along the centreline and check the pressure drop.
- Try a longer channel so the flow can fully develop and compare with the analytical parabolic profile.
- Refine the mesh and see how much the results change.

## Tools used

- OpenFOAM 14
- Ubuntu
- ParaView 5.11.2

## Author

**Pratyush Dash**

B.Tech Chemical Engineering, KIIT University, Bhubaneswar
