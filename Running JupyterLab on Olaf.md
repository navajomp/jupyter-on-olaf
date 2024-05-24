As Python <= 3.10 is a necessity for `esm_tools`, it might be better to use a separate environment for JupyterLab where newer versions of Python can be used.

Step 1: On Olaf, add the `conda-forge` channel for packages. (Tip: Always use the conda-forge channel which will be essential for proper running of dask later)

```bash
conda config --add channels conda-forge
conda config --set channel_priority strict
```

Step 2: Create an environment.

```bash
conda create -n myenv jupyterlab xarray dask dask-mpi cartopy scipy 
```

Replace `myenv` with any other preferred name. This is a short list of packages to begin with. Additional packages can be installed using:

```bash
conda install -n myenv PACKAGENAME
```

Activate/deactivate the environment with:

```bash
conda activate myenv
conda deactivate
```

Step 3: Add the following line to the `.bashrc` on Olaf.

```bash
alias jlab="jupyter lab --no-browser --port 8889"
```

Step 4: Add the following lines to the `.bashrc` or `.zshrc`  (or any other shell) your local machine.

```bash
alias jcon="ssh -N -L 8889:localhost:8889"
alias jcondask="ssh -N -L 8889:localhost:8889 -L 8787:localhost:8787"
```

You may choose different port numbers in step 3 and 4.

Step 5: Set up the `.ssh/config` on your local machine. 

```bash
Host olaf
    Hostname olaf.ibs.re.kr
    User <your Olaf user name>
    Port 4022
```

Steps 1-5 are done only once.

Step 6: Actually running JupyterLab on Olaf.

1. Log in to Olaf
2. Activate the environment
```bash
   conda activate myenv
```
3. Initiate Jupyterlab
```bash
   jlab
```

Copy the URL that starts with `http://localhost:`

4. On your local machine, open up a terminal establish the tunnel.
```bash
   jcon olaf
```

`olaf` is the name used in `.ssh/config`

5. Paste the URL in a web browser to open the JupyterLab.

Voila!