# Ion 1D Model

The one-dimensional ion model includes formation, recombination, diffusion, convection, and electrical mobility in a one-dimensional domain with absorbing surfaces at either end of the domain.  The detailed development of the model and its solution are presented in [the 2020 Combustion Theory and Modelling paper](2020_ctm.pdf)[1], the preprint for which is included in this directory. The model data are divided into individual cases.  Each case is intended to represent a different physical operating condition for the system, at which many applied electrical potentials are applied to the system.

Each case contains a whitespace delimited file, `contents.txt` that summarizes the conditions for each of the data files in the case.  The `source` directory contains the code for the case, written in Python.  Appart from the native Python libraries, all custom code needed for each case is contained in `source`.  The `data` directory contains the results of the model as tarballs, and `export` contains plots.

## The ion1d module

`ion1d.py` is the module that contains the algorithm for the model.  The classes are designed to make the model easy to manipulate and run from the command line, so debugging convergence problems for a new parameter set is intended to be easy.  The classes are documented in-line using Python help.

As of version 1.7, there are two model classes built from the base `Ion1D` class.  The `FiniteIon1D` class defines a model for uniform ion generation between two locations in space; z1 and z2.  The `AnchoredFiniteIon1D` class is a minor variation that eliminates the upstream region, so that ion formation begin propmtly at z=0 and ends at z1.

An example session that produces a converged solution might appear
```python
>>> import ion1d
>>> M = ion1d.FiniteIon1D()                 # Create the model
>>> M.init_param(z1=.01, z2=.21, R=1000)    # Initialize model params
>>> M.init_grid(d=.001, r=(.02, .5, 1, .02))# Define the grid
>>> M.init_mat()                            # Initialize the solution tensors
>>> M.init_solution()                       # Form an initial guess
>>> while not M.test_solution():            # Test for convergence
...     M.step_solution()                   # Iterate
...
>>> M.show_solution()                       # Plot the solution+residuals
>>> P = ion1d.PostIon1D(M)                  # Generate a post object
>>> P.save('/my/filename/here')             # Record the results in a tarball
```
The parameters are used to build the grid and construct the solution matrices, so they must not be altered after these steps.

There is too much to say about how all this works to say it here.  Instead, use the in-line help to learn the ion1d API.  You can get started by typing
```python
>>> help(ion1d)
```
Plus, each of the classes and methods is fully documented.
```python
>>> help(ion1d.FiniteIon1D)
...
>>> help(M.init_param)
...
```

## The SOURCE directory

There is a `source` directory in each case and in the model root, each of which is a self-contained collection of code.  Changes to one case do not affect the other cases, and there is no requirement that the cases be run with the same versions of the `ion1d` module.  The other codes are scripts responsible for managing the case: 

`runbaseline.py` creates a tarball called `baseline.tar.bz2` in the case directory that is the result of a model run that is used as an initial guess for the case model runs.  The baseline data set is also the athoritative source for the case's parameters, so grid or parameter changes must be made there.  

The `runcase.py` runs many parallel models simultaneously to establish the case's voltage characteristics.  The parameters and grid are borrowed explicitly from the baseline model.  The results of each individual model run are stored in the case's `data` directory as tarballs; just like the baseline data.

`plotcase.py` loads the case's data and builds plots showing the solution for each case and summary plots.  These are stored as .png files in the `export` directory.  As part of this process, `plotcase.py` is responsible for running the `expand_post()` method for each model, and the tarballs in `data` are then modified to include these results.

`appendcase.py` is just like `runcase.py` except that it is designed to add data to an existing case run without disturbing the existing data.  This is useful if the user sees that there are additional `phia` values that would be useful.

## The DATA directory

The data generated by the `ion1d` model are contained in the model object as NumPy arrays and as subordinate objects.  Rather than attempt to serialize them into individual files, the results are split into separate files in a single tarball, which may nor may not be compressed.  The individual models cannot save themselves, nor are they configured to do post-processing of any kind.  That behavior is relegated to the `PostIon1D` class, which can be created (see above) from any child of the `Ion1D` base class.

Each tarball contains a `post.json` file (a JavaScript Object Notation file) that defines a dictionary with the attributes of the `PostIon1D` object that created it.  This will include user-created attributes that were part of the post-processing.  NumPy arrays are stored as .npy files and their values are replaced by the filename in the dictionary.  This and a few other minor steps to serialize the attributes are reversed to re-build the `PostIon1D` on load.


## The EXPORT directory

The `export` directory contains plots for each case: `XXX.png` shows the ion and electron concentrations over the entire domain.  `XXX_tsheath.png` zooms in on the sheath at z=0.  `XXX_wsheath.png` zooms in on the sheath at z=1.  `XXX_charge.png` shows the charge imbalance over the entire domain.  `XXX_phi.png` shows the electric potential.  `XXX_1.png` and `XXX_phi1.png` show the perturbed solution over the entire domain.

There are also plots that summarize the entire case: `jphi.png` is a current-voltage characteristic plot over the entire dataset.  `jphi1.png` shows the local slope of the `jphi.png` produced using the perturbation method.

## References

[1] C. Martin, A. Untaroiu, K. Xu, “A one dimensional model for ion transport in flame with two absorbing surfaces,”  Combustion Theory and Modelling,  2020. [in press]
