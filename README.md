# Science Skills

Collection of scientific and engineering skills. As examples, you can run it through `mistralrs_skill_cli.py`. The CLI is intentionally simple:

- it zips a skill directory,
- uploads the skill to `POST /v1/skills`,
- uploads each `--file` to `POST /v1/files` with `purpose=user_data`,
- sends a Responses request with `input_file.file_id`,
- enables shell, code execution, and web search by default,
- writes a local results package for every run.

<img width="1046" height="533" alt="image" src="https://github.com/user-attachments/assets/720b8e0d-f77a-4327-9396-9c2acf1fb7cd" />


## Install mistral.rs

**Linux/macOS:**
```bash
curl --proto '=https' --tlsv1.2 -sSf https://raw.githubusercontent.com/EricLBuehler/mistral.rs/master/install.sh | sh
```

**Windows (PowerShell):**
```powershell
irm https://raw.githubusercontent.com/EricLBuehler/mistral.rs/master/install.ps1 | iex
```

This downloads a self-contained prebuilt binary for your platform (Metal on Apple Silicon; per-GPU CUDA or CPU on Linux; CPU on Windows), falling back to a source build if none matches. No Rust or CUDA toolkit needed for the prebuilt path.

[Manual installation, accelerator details & other platforms](https://ericlbuehler.github.io/mistral.rs/quickstart/)

![Uploading mistralrs-quickstart.gif…]()


## Start mistral.rs

The Skills runtime requires the shell executor. The easiest local mode is `--agent`, which also enables code execution and web search.

A minimal single-model server is:

```bash
mistralrs from-config -f models.toml
```
For direct serving without TOML:

```bash
mkdir -p /tmp/mistralrs-shell-workdir /tmp/mistralrs-skills

mistralrs serve --agent \
  --host 0.0.0.0 \
  --port 1234 \
  --shell-workdir /tmp/mistralrs-shell-workdir \
  --skills-dir /tmp/mistralrs-skills \
  -m google/gemma-4-E4B-it
```

`--shell-workdir` is optional but useful. Without it, mistral.rs uses a
per-session temp directory for shell outputs. With it, generated files are also
visible directly on disk under the chosen path.

Check the server:

```bash
curl http://localhost:1234/v1/models
```

## Examples

### Fracture Mechanics Movie And Stress-Strain Curve

Use this when you want a serious simulation artifact that a smaller local model
can produce reliably by running a bundled script. It simulates a pre-cracked 2D
triangular lattice under Mode I or Mode II loading and saves a movie plus
stress-strain data.

```bash
./mistralrs_skill_cli.py ../skills/fracture-mechanics \
  --max-tool-rounds 8 \
  --response-timeout 2400 \
  --require-tool \
  --query "Use fracture-mechanics to simulate a pre-cracked 2D lattice titled 'Brittle Mode I Fracture'. Use potential morse, mode I, orientation 90, nx 96, ny 36, max-atoms 5000, crack-length 0.34, temperature 0.003, strain-rate 0.0035, damping 0.30, dt 0.005, steps 10000, frames 48, dpi 120, movie-fps 16, bond-cutoff 1.35, break-stretch 1.65, color-by stress, and morse-a 7.0. Use fixed slab axes, fixed stress-strain axes, and fixed color scale across frames to avoid movie flicker. Save fracture_movie.gif, fracture_movie.html, stress_strain.png, final_lattice.png, stress_strain.csv, summary.json, parameters.json, and README. Verify files with find and report peak stress, peak strain, and dynamically broken bonds from summary.json."
```

Expected outputs:

- `fracture_movie.gif`,
- `fracture_movie.html`,
- `frames/frame_*.png`,
- `stress_strain.png`,
- `final_lattice.png`,
- `stress_strain.csv`,
- `summary.json`,
- `parameters.json`,
- `README.md`.

<img width="1260" height="648" alt="fracture_movie 6" src="https://github.com/user-attachments/assets/39d74a26-f228-42c7-8db0-966f114d9be3" />

### Beam Mechanics Plots And Deformation GIFs

Use this for a compact, reliable mechanics artifact. The skill is dimensionless: no units, no YAML, no JSON input schema.  

Baseline simply supported beam:

```bash
./mistralrs_skill_cli.py ../skills/beam-mechanics \
  --max-tool-rounds 8 \
  --response-timeout 1200 \
  --query "Use beam-mechanics to analyze a dimensionless simply supported beam of length 10 with a downward point load of magnitude 1 at midspan. Use the bundled simple_beam_lab.py script. Save all plots, the deformation GIF, field data CSV, reactions JSON, summary JSON, manifest JSON, and README. Verify the output directory with find before answering. Final answer must report support reactions, maximum absolute deflection, maximum absolute bending moment, maximum absolute shear, vertical equilibrium residual, and exact artifact paths."
```

Expected outputs:

- `structure.png`,
- `deformed_shape.png`,
- `deflection.png`,
- `moment.png`,
- `shear.png`,
- `dashboard.png`,
- `deformation.gif`,
- `frames/frame_*.png`,
- `field_data.csv`,
- `summary.json`,
- `reactions.json`,
- `manifest.json`,
- `README.md`.

<img width="960" height="384" alt="deformation 4" src="https://github.com/user-attachments/assets/1f1089ac-010c-45fb-aed6-881ee6af248b" />


### Hierarchical Topology Optimization Examples

Use this when you want a more design-like artifact: density fields, resolved
boundary conditions, convergence history, and optional STL geometry. The skill
is intentionally lightweight and avoids gyroids, hole lattices, marching cubes,
and heavy boolean geometry.

Cantilever with midpoint load, profile STL:

```bash
./mistralrs_skill_cli.py ../skills/hierarchical-topopt \
  --max-tool-rounds 10 \
  --response-timeout 1800 \
  --query "Use hierarchical-topopt to optimize a 2D cantilever. Use nelx 100, nely 35, volfrac 0.50, penal 4.0, rmin 4.0, density filter, maxiter 220, and bc-preset cantilever-mid-down. Use mesh-mode profile-stl, dens-cut 0.30, max-height 10, and profile-origin bottom. Save density.png, density_resized.png, density.npy, density.csv, optimization_history.csv, boundary_conditions.json, boundary_conditions_resolved.json, bc_preview.png, convergence.png, result_profile.stl, summary.json, parameters.json, and README. Verify files with find and report compliance, final volume fraction, fixed DOFs, loaded DOFs, total force, and exact artifact paths."
```

Bridge with pin/roller supports and middle-20-percent top load:

```bash
./mistralrs_skill_cli.py ../skills/hierarchical-topopt \
  --max-tool-rounds 12 \
  --response-timeout 2400 \
  --query "Use hierarchical-topopt to optimize a bridge-like 2D structure. Use nelx 120, nely 40, volfrac 0.45, penal 4.0, rmin 4.2, density filter, and 250 iterations. Define custom boundary conditions with a pin support at the lower-left corner, a roller support at the lower-right corner, and a downward distributed total load across the middle 20 percent of the top edge. Write boundary_conditions.json using edge_fraction start 0.4 end 0.6 for the top-edge load, preview it as bc_preview.png, then run the optimization with mesh-mode profile-stl, dens-cut 0.30, max-height 10, and profile-origin center. Save density images/data, optimization history, boundary_conditions.json, boundary_conditions_resolved.json, bc_preview.png, result_profile.stl, summary.json, parameters.json, and README. Verify files with find and report compliance, final volume fraction, fixed DOFs, loaded DOFs, total force, and exact artifact paths."
``` 

Expected outputs:

- `density.png`,
- `density_resized.png`,
- `density.npy`,
- `density_resized.npy`,
- `density.csv`,
- `optimization_history.csv`,
- `boundary_conditions.json`,
- `boundary_conditions_resolved.json`,
- `bc_preview.png`,
- `convergence.png`,
- `summary.json`,
- `parameters.json`,
- `README.md`,
- optional `result_profile.stl`, `result_flat.stl`, or multimaterial STL files depending on `--mesh-mode`.
- optional STL render directory with `turntable.gif`, `preview.png`, `frames/frame_*.png`, `render_manifest.json`, and `README.md`.

<img width="1080" height="480" alt="density" src="https://github.com/user-attachments/assets/de443b0a-3010-49ed-93f0-d071744696d7" />

### Chladni Plate Resonance Examples

Use this when you want visually compelling physics artifacts: nodal-line
patterns, sound-wave resonance demonstrations, text-free high-resolution PNGs,
and GIF animations of vibrating plates.

Classic square Chladni figure:

```bash
./mistralrs_skill_cli.py ../skills/chladni-plates \
  --max-tool-rounds 8 \
  --response-timeout 1200 \
  --require-tool \
  --query "Use chladni-plates to render a classic square Chladni plate resonance. Use preset classic-square, palette blue-sand, size 1080, 48 frames, and 18 fps. Keep all rendered images and GIF frames text-free. Save chladni_pattern.png, chladni_preview.png, chladni_animation.gif, frames, field_data.npz, parameters.json, caption.txt, and README. Verify files with find and report exact artifact paths plus preset, shape, modes, palette, frames, and fps from parameters.json."
```
<img width="4615" height="1439" alt="small-chald" src="https://github.com/user-attachments/assets/1bb59a75-a803-4b85-96d0-b44e1368c777" />



