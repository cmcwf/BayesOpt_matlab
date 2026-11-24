# bayesopt_matlab

`bayesopt_matlab` is a comprehensive MATLAB suite for **Single-Objective** and **Multi-Objective Bayesian Optimization**. The repository provides robust implementations designed for high-dimensional, noisy, or computationally expensive black-box optimization problems, with specialized applications in continuous flow reactors, chemical synthesis, and electrochemical process optimization.

## 🌟 Key Features

### 1. BOAEI: Bayesian Optimization with Augmented Expected Improvement

* **Sequential & Batch Sequential BO:** Handles single-objective Bayesian optimization under noisy evaluation conditions using Augmented Expected Improvement ($\text{AEI}$) as the acquisition function, developed in collaboration with Dr. Adam Clayton for reaction self-optimization.

* **Batch Processing:** Support for parallelized evaluation of multiple candidate design points per iteration via `BayesianOptimiser_BatchSequential.m`.

* **Integrated Global Optimizer (`GODLIKE`):** Includes the hybrid global optimization toolbox `GODLIKE` (combining Genetic Algorithms, Particle Swarm Optimization, Adaptive Simulated Annealing, and Differential Evolution) for optimizing acquisition functions.

### 2. TSEMO: Thompson Sampling Efficient Multi-Objective Optimization

* **Based on Eric Bradford's TS-EMO:** Incorporates the [TS-EMO MATLAB implementation](https://github.com/Eric-Bradford/TS-EMO) by Eric Bradford for efficient multi-objective optimization.

* **Multi-Objective Optimization:** Efficient Pareto frontier identification across competing objectives ($f_1(\mathbf{x}), f_2(\mathbf{x}), \dots, f_m(\mathbf{x})$ ).

* **Surrogate Modeling:** Utilizes Gaussian Process ($\mathcal{GP}$) regression coupled with spectral sampling for scalable Thompson sampling.

* **High-Performance Acceleration:** Optimized MEX functions (`hypervolume2D`, `hypervolume3D`, `invChol`, `paretofront`) for rapid hypervolume calculation and non-dominated sorting.

* **NSGA-II Integration:** Embedded `NGPM_v1.4` (Next Generation Pareto Master) solver for multi-objective sub-problems.

### 3. Real-World Datasets & Case Studies

* Benchmark test functions ($DTLZ1$, $DTLZ2$, $DTLZ4$, $DTLZ7$, $KNO1$, $OKA1$, $OKA2$, $VLMOP2$, $VLMOP3$).

* Process reaction datasets (`XEC_data` series, DIPMeA optimization, ReLU/LReLU activation surrogate datasets) for chemical and electrochemical process self-optimization.

## 📁 Repository Structure

```
bayesopt_matlab/
├── BOAEI/                                 # Bayesian Optimization w/ Augmented Expected Improvement
│   ├── BayesianOptimiser.m                # Sequential BOAEI implementation
│   ├── BayesianOptimiser_BatchSequential.m# Batch-sequential BOAEI algorithm
│   ├── example.m                          # BOAEI usage example script
│   └── GODLIKE/                           # Hybrid metaheuristic global optimizer toolbox
│
├── TSEMO/                                 # Thompson Sampling Efficient Multi-Objective Optimization
│   ├── TSEMO_V1.m / TSEMO_V2.m            # TSEMO core algorithm routines (from Eric-Bradford/TS-EMO)
│   ├── TSEMO_V2_SOFR.m                    # TSEMO variant for Self-Optimizing Flow Reactors
│   ├── TSEMO_Example.m                    # Multi-objective optimization example script
│   ├── TSEMO_options.m                    # Configuration options structure
│   ├── Direct/                            # DIRECT optimization algorithm
│   ├── NGPM_v1.4/                         # NSGA-II MATLAB toolbox
│   ├── Mex_files/                         # Compiled C/MEX binaries (Hypervolume, Cholesky, Pareto)
│   └── Test_functions/                    # Standard multi-objective benchmark functions
│
├── matlab_file/                           # Process optimization datasets & MAT files
├── XEC_data_processed.xlsx                # Processed experimental data
└── model.mat                              # Saved surrogate models and optimization states
```

## ⚙️ Installation & Setup

1. **Clone the repository:**

   ```bash
   git clone https://github.com/cmcwf/bayesopt_matlab.git
   cd bayesopt_matlab
   ```

2. **Open MATLAB:**
   Ensure MATLAB R2018b or newer is installed with the *Statistics and Machine Learning Toolbox*.

3. **Add Paths:**
   Run the following in your MATLAB Command Window to add all necessary subdirectories to your path:

   ```matlab
   addpath(genpath(pwd));
   savepath;
   ```

4. **(Optional) Recompile MEX Binaries:**
   Pre-compiled 64-bit Windows MEX binaries (`.mexw64`) are provided in `TSEMO/Mex_files/`. If running on macOS or Linux, recompile the source files using:

   ```matlab
   mex TSEMO/Mex_files/hypervolume/hypervolume2D.c
   mex TSEMO/Mex_files/hypervolume/hypervolume3D.c
   ```

## 🚀 Quick Start Examples

### Running Single-Objective BOAEI

Navigate to the `BOAEI/` folder and execute `example.m`:

```matlab
% Define objective function with noisy evaluations
objFcn = @(x) sin(x(:,1)) + cos(x(:,2)) + 0.1 * randn(size(x,1), 1);

% Define parameter bounds [lower_bounds; upper_bounds]
lb = [-3, -3];
ub = [ 3,  3];

% Execute BOAEI
[bestX, bestY] = BayesianOptimiser(objFcn, lb, ub);
```

### Running Multi-Objective TSEMO

Navigate to the `TSEMO/` folder and execute `TSEMO_Example.m`:

```matlab
% Set up optimization options
options = TSEMO_options();
options.max_evals = 50;
options.n_initial = 10;

% Define benchmark objective (e.g., VLMOP2)
fun = @vlmop2; 

% Execute TSEMO algorithm
[X_sample, Y_sample, ParetoFront] = TSEMO_V2(fun, options);
```

## 📐 Mathematical Formulation

### Augmented Expected Improvement ($\text{AEI}$)

For noisy function evaluations $y = f(\mathbf{x}) + \epsilon$ where $\epsilon \sim \mathcal{N}(0, \sigma_n^2)$, the Augmented Expected Improvement acquisition function is defined as:

$$
\text{AEI}(\mathbf{x}) = \mathbb{E}\left[ \max\left(0, \mu(\mathbf{x}^*) - f(\mathbf{x})\right) \right] \times \left( 1 - \frac{\sigma_n}{\sqrt{\sigma^2(\mathbf{x}) + \sigma_n^2}} \right)
$$

where $\mu(\mathbf{x}^*)$ represents the estimated mean value at the current effective best location $\mathbf{x}^*$, and $\sigma^2(\mathbf{x})$ is the Gaussian Process predictive variance.

### Thompson Sampling in Multi-Objective Space

TSEMO constructs independent Gaussian Process surrogates for each target objective function $f_m(\mathbf{x})$:

$$
f_m(\mathbf{x}) \sim \mathcal{GP}\left(m_m(\mathbf{x}), k_m(\mathbf{x}, \mathbf{x}')\right), \quad m = 1, \dots, M
$$

In each iteration, function realizations $\tilde{f}_m(\mathbf{x})$ are sampled from the posterior GP distributions using Fourier spectral sampling, and the resulting multi-objective sub-problem is solved via NSGA-II to select next-eval candidate points on the predicted Pareto frontier.

## 📜 License

This project is open-source. Please consult individual headers and license files in subdirectories (`GODLIKE/license.txt`, `TSEMO/LICENSE`, `TSEMO/Mex_files/...`) for specific third-party module terms.

## 🤝 Acknowledgements

* **Dr. Adam Clayton:** Special thanks to Dr. Adam Clayton for collaboration on the MATLAB implementation of Bayesian Optimization using Augmented Expected Improvement ($\text{AEI}$) for reaction self-optimization (`BOAEI/`).

* **Eric Bradford:** The TSEMO implementation included in this repository is adapted from the original [TS-EMO MATLAB Code](https://github.com/Eric-Bradford/TS-EMO) developed by Eric Bradford.

## 📚 References & Citation

If you use **BOAEI** for reaction self-optimization, please cite:

> A. D. Clayton et al., "Bayesian Self-Optimization for Telescoped Continuous Flow Synthesis," *Angewandte Chemie International Edition*, vol. 62, no. 3, p. e202214511, 2023. DOI: [10.1002/anie.202214511](https://doi.org/10.1002/anie.202214511)

If you use the **TSEMO** algorithm in your research, please cite:

> E. Bradford, A. M. Schweidtmann, and A. A. Lapkin, "Efficient multiobjective optimization employing Gaussian processes, spectral sampling and a genetic algorithm," *Journal of Global Optimization*, vol. 71, no. 2, pp. 407–438, 2018. DOI: [10.1007/s10898-018-0609-2](https://doi.org/10.1007/s10898-018-0609-2)

For queries or bug reports regarding this repository, please open an issue on GitHub.
