# Guideline for Creating a New Example in ChemPlasKin

This guide explains how to add a new example case to the ChemPlasKin project, based on the structure of existing examples like `H2O2He` and `Air`.

## 1. Directory Structure
Navigate to the `examples` directory. Each example is a self-contained folder.
e.g., `examples/MyNewExample`

A typical example folder contains:
- **`CMakeLists.txt`**: Build configuration for the example.
- **`testMyExample.cpp`** (or main source file): The C++ code that sets up the reactor and runs the simulation.
- **`chemPlasProperties`** (Optional): A configuration file used by some examples to define conditions without recompiling.
- **`controlDict`** (Optional): A configuration file for time-stepping and output control.
- **`local_kinetics.yaml`** (Recommended): A local copy of the chemistry mechanism file.

## 2. Connection to Data Files
ChemPlasKin relies on two main types of data located in the `data` directory at the project root:

### A. Cross-Section Data (`data/LXCat/`)
- Contains `.dat` files (e.g., `bolsigdb_N2.dat`) with electron scattering cross-sections.
- **Usage**: Loaded by the Boltzmann solver in your C++ code or `chemPlasProperties`.
- **Format**: LXCat format. Used to calculate electron energy distribution functions (EEDF).

### B. Chemical Kinetics (`data/PAC_kinetics/`)
- Contains `.yaml` (or `.inp`) files defining species, thermodynamics, and reactions.
- **Usage**: Loaded by Cantera to define the gas phase chemistry.
- **Recommendation**: Do **not** modify files in `data/` directly for specific examples. Instead, **copy** the relevant YAML file to your example folder and modify the local copy.

## 3. Step-by-Step Guide to Create a New Example

### Step 1: Create Folder
Copy an existing, similar example folder to a new name.
```bash
cp -r examples/N2 examples/MyNewTransport
```

### Step 2: Clean and Prepare
Enter the new folder and remove old build artifacts.
```bash
cd examples/MyNewTransport
rm -rf build output*.csv
```

### Step 3: Create Local Kinetics File
Copy the base mechanism you want to use (e.g., `plasmaN2.yaml`) into your folder.
```bash
cp ../../data/PAC_kinetics/plasmaN2.yaml my_mechanism.yaml
```
*Why?* This allows you to add specific reactions (like wall losses or auxiliary species) without affecting other simulations.

### Step 4: Update Configuration
1.  **Edit `CMakeLists.txt`**:
    - Update `project(...)` name if desired.
    - Update `add_executable(...)` if you rename the `.cpp` file.
    - Ensure include paths are correct (relative paths `../../../` might need adjustment if depth changes, but usually it's fine).

2.  **Edit Source Code (`.cpp`)**:
    - Update the path to load your **local** kinetics file:
      ```cpp
      auto sol = newSolution("../my_mechanism.yaml");
      ```
    - Update the species list for the Boltzmann solver (if needed).
    - Set initial parameters ($T_{gas}$, $P$, $E/N$).

### Step 5: Implement Specific Physics (e.g., Wall Transport)
To simulate transport losses in a global model (0D), add "effective" reactions to your local YAML file.

**Example: Wall Loss Reaction**
For a species $X$ diffusing to the wall:
$$ X \xrightarrow{k_{wall}} \text{Products} $$
Rate $k_{wall} \approx \frac{D_{eff}}{\Lambda^2}$ or $\frac{\gamma v_{th}}{2L}$ (for surface-limited regimes).

**YAML Implementation:**
```yaml
- equation: N => 0.5 N2
  type: PlasmaCustomExpr
  rateExpr:
    A: 1.0
    # Example expression for rectangular gap L=0.1cm, gamma=1e-3
    Expr: "(1.0e-3 * sqrt(8 * 8.314e7 * Tgas / (3.14159 * 14.0))) / (2 * 0.1)"
```

### Step 6: Build and Run
```bash
mkdir build
cd build
cmake ..
make -j4
./ChemPlasKin
```

## 4. Troubleshooting
- **Zero Rates**: If a wall reaction has a rate of 0, check if the reactant species is actually being produced. You may need to enable specific production channels (e.g., Boltzmann electron impact excitation).
- **Paths**: Ensure file paths in `.cpp` relative to the *executable* (which is in `build/`) are correct (usually `../filename`).
