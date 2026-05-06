# Interface Analysis: MCSampler

## Overview
The `MCSampler` class implements an adaptive Monte Carlo integrator based on the VARAHA algorithm. It is used to integrate likelihood functions in parameter spaces.

## Class: MCSampler

### Constructor: `__init__(self, n_chunk=400000, **kwargs)`
- **n_chunk**: Number of samples to draw in each iteration (default 400000)
- **kwargs**: Additional keyword arguments (not explicitly used in __init__ but may be set as attributes)
- **Initializes**:
  - `ntotal`: total number of samples drawn (starts at 0)
  - `params`: set of parameter names
  - `params_ordered`: list to maintain parameter order
  - `params_pinned_vals`: dictionary for pinned parameter values
  - `_rvs`: cache for sampling points
  - `llim`, `rlim`: dictionaries for lower and upper limits
  - `n_chunk`: chunk size
  - `nbins`, `ninbin`, `adaptive`: initialized to None or empty
  - `pdf`, `prior_pdf`: dictionaries for likelihood and prior functions
  - `xpy`: numpy (or cupy if available)
  - `identity_convert`: function to convert to numpy
  - `V`, `delta_V`: volume and delta volume
  - `setup()`: called to initialize ranges and bin widths

### Key Methods

#### `setup(self, **kwargs)`
- Initializes binning and ranges based on current parameters.
- Sets `nbins`, `ninbin`, `binunique`, `my_ranges`, `dx`, `dx0`, `cycle`, `V`, `V_s`, `lnL_thresh`, `enc_prob`, `is_varaha`.

#### `clear(self)`
- Resets parameters and clears cache: `params`, `params_ordered`, `pdf`, `_pdf_norm`, `_rvs`, `llim`, `rlim`, `adaptive`.

#### `add_parameter(self, params, pdf, cdf_inv=None, left_limit=None, right_limit=None, prior_pdf=None, adaptive_sampling=False)`
- Adds a parameter (or tuple of parameters) to be sampled.
- Updates `params`, `params_ordered`, `pdf`, `prior_pdf`, `llim`, `rlim`.
- If `adaptive_sampling` is True, adds parameter to `adaptive` list.

#### `prior_prod(self, x)`
- Computes the product of prior PDFs for all parameters at points `x`.
- Returns: array of prior values.

#### `draw_simplified(self, n_to_get, *args, **kwargs)`
- Draws samples and returns sampling prior, likelihood, and parameter values.
- Returns: `(ps, p, rv)` where:
  - `ps`: sampling prior
  - `p`: likelihood values
  - `rv`: parameter values (transposed)

#### `draw_simple(self)`
- Draws samples from the current sampling distribution.
- Returns: `(rv, log_p)` where:
  - `rv`: parameter values
  - `log_p`: log of prior probabilities

#### `update_sampling_prior_selfish(self, lnF, *args, xpy=xpy_default, no_protect_names=True, **kwargs)`
- Updates the sampling prior using the VARAHA algorithm based on likelihood evaluations.
- Updates internal state: `V`, `delta_V`, `nbins`, `ninbin`, `binunique`, `cycle`.

#### `integrate_log(self, lnF, *args, xpy=xpy_default, **kwargs)`
- Integrates `exp(lnF)` using the sampling algorithm.
- **Parameters**:
  - `lnF`: function returning log-likelihood (or log of integrand)
  - `kwargs`: 
    - `nmax`: max samples (default inf)
    - `neff`: effective samples to stop (default inf)
    - `n`: batch size (default min(100000, nmax))
    - `history_mult`: batches for adaptive history (default 2)
    - `tempering_exp`: exponent for tempering (default 0.0)
    - `n_adapt`: chunks for adaptation (default 1000)
    - `floor_level`: floor probability (default 0)
    - `temper_log`: temper log flag (default False)
    - `tempering_adapt`: tempering adapt flag (default False)
    - `save_intg`: save intermediate (default False)
    - `igrand_threshold_deltalnL`: threshold for log-likelihood difference (default inf)
    - `igrand_threshold_p`: threshold for probability (default 0)
    - `igrand_fairdraw_samples`: fair draw flag (default False)
    - `igrand_fairdraw_samples_max`: max fair draw samples (default None)
    - `verbose`: verbose flag (default False)
    - `super_verbose`: super verbose flag (default False)
    - `dict_return`: return dict flag (default False)
    - `enforce_bounds`: enforce integer bins (default False)
- **Returns**: `(log_int, log_var, eff_samp, dict_return)` where:
  - `log_int`: log of integral
  - `log_var`: log of variance
  - `eff_samp`: effective sample size
  - `dict_return`: additional debug info if requested

#### `integrate(self, func, *args, **kwargs)`
- Integrates a function (not log) by converting to log internally.
- **Parameters**: same as `integrate_log` plus:
  - `use_lnL`: if True, uses `lnF` directly (default False)
- **Returns**: same as `integrate_log`: `(log_int_val, log_var, eff_samp, dict_return)`

### Attributes (set during setup and sampling)
- `params`: set of parameter names
- `params_ordered`: ordered list of parameter names
- `params_pinned_vals`: dict of fixed parameter values
- `llim`, `rlim`: dicts of lower/upper limits per parameter
- `nbins`: array of bin counts per dimension
- `ninbin`: array of samples per bin
- `adaptive`: list of parameters using adaptive sampling
- `pdf`, `prior_pdf`: dicts of likelihood and prior functions
- `V`: current volume estimate
- `delta_V`: uncertainty in volume
- `ntotal`: total samples drawn
- `xpy`: numpy or cupy module
- `identity_convert`: conversion function to numpy

### Usage Pattern
1. Create instance: `sampler = MCSampler(n_chunk=...)`
2. Add parameters: `sampler.add_parameter(param_name, pdf_func, prior_pdf=prior_func, left_limit=low, right_limit=high, adaptive_sampling=True/False)`
3. (Optional) Set pinned values via kwargs in integrate call.
4. Call integrate: `log_integral, log_variance, eff_size, info = sampler.integrate_log(my_log_likelihood_function, nmax=1e6, neff=1000)`
5. Access samples: `sampler._rvs` contains arrays of sampled parameters and derived quantities.

## Notes
- The algorithm adapts the sampling density based on likelihood values.
- Supports GPU via cupy if available.
- Parameters can be grouped as tuples for joint sampling.
- The integrator returns the log of the integral and its variance.