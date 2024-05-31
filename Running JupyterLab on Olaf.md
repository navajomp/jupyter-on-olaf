## Running JupyterLab on Olaf

As Python <= 3.10 is a necessity for `esm_tools`, it might be better to use a separate environment for JupyterLab where newer versions of Python can be used.

Step 1: On Olaf, add the `conda-forge` channel for packages.

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

Step 4: Add the following lines to the `.bashrc` or `.zshrc`  (or any other shell) of your local machine.

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

6.1. Log in to Olaf
6.2. Activate the environment
```bash
   conda activate myenv
```
6.3. Initiate Jupyterlab
```bash
   jlab
```

Copy the URL that starts with `http://localhost:`

6.4. On your local machine, open up a terminal and establish the tunnel. This does not produce any output.
```bash
   jcon olaf
```

`olaf` is the name used in `.ssh/config`

6.5. Paste the URL in a web browser to open the JupyterLab!

---

### Taking care of the environment

Be wary of installing new packages or tampering with the environment as it can cause troubles with `dask`.  Environments can be rolled back to previous versions in two ways.

#### Method 1 - a shareable file

If you want to make substantial changes to your environment, backup the environment by exporting it to a `yml` file as following:

```bash
conda activate myenv
conda env export > environment.yml
```
The name of the `yml` file is arbitrary.

The file can be used to restore the environment.

```bash
conda env create -f environment.yml
```

#### Method 2 - the builtin --revisions method

Conda tracks the revisions made to an environment. You can list the revisions using:

```bash
conda list --revisions
```

To go back to a version 'when things were fine':

```bash
conda install --revision=REVNUM
```

where `REVNUM` refers to the revision number you want restored.

---

### How to parallelize your computations?

Step 1: Setting up mpirun directories.

Create a dedicated directory for our mpirun needs. 

```bash
mkdir /proj/internal_group/iccp/$USER/mpi
```

Create the following two files, to start and reset mpi, inside the folder.

File 1 `/proj/internal_group/iccp/$USER/mpi/start.sh`:

```bash
#!/bin/sh
mpirun --np 6 dask-mpi --worker-class distributed.Worker --scheduler-file /proj/internal_group/iccp/$USER/mpi/scheduler.json --dashboard-address :8787 --memory-limit=90e9 --local-directory /proj/internal_group/iccp/$USER/mpi/tmp/
```
Here we use 6 nodes in a session. Ideally, you can use more nodes when the cluster is not busy -- at nights or on the weekends. 

File 2 `/proj/internal_group/iccp/$USER/mpi/reset.sh`:

```bash
#!/bin/sh

rm -rf worker*
rm *.lock
rm /proj/internal_group/iccp/$USER/mpi/scheduler.json
rm -rf /proj/internal_group/iccp/$USER/mpi/tmp/
rm -rf dask-worker-space
```

You may add aliases to make things easier. Add the lines to `.bashrc`.

```bash
alias start='bash /proj/internal_group/iccp/$USER/mpi/start.sh'
alias reset='bash /proj/internal_group/iccp/$USER/mpi/reset.sh'
```

Step 2: Running dask on notebooks

2.1. Log into Olaf. Initiate JupyterLab.
```bash
conda activate myenv
jlab
```
Copy the URL that starts with `https://localhost.`

2.2. On a new tab, log into Olaf again and initiate the mpirun command
```bash
conda activate myenv
start
```
You should see the cluster information as output.

2.3. Open a new terminal window. On your local machine, initiate the tunnel. This doesn't produce any output.
```bash
jcondask olaf
```
Paste the URL in a browser to open JupyterLab running on Olaf with dask enabled.

2.4. In your python notebook, run the following cell before anything else:
```python
from dask.distributed import Client
client = Client(scheduler_file='<path to your mpi folder>/scheduler.json')
```

Once you complete your analysis, run the `reset` script to clear the workers.

---
#### (OPTIONAL) Using the `dask-labextension`
Monitoring your dask processes can be cumbersome, but it gets easier with the `dask-dashboard` which provies a fancy realtime graphical view of the jobs, their progress, and other details.

Once you initiate JupyterLab with dask, open up a terminal from within the browser (File -> New -> Terminal).

To install the `dask-labextension`, 

```bash
conda activate myenv
conda install dask-labextension
```

You may have to refresh the webpage. You should now see a yellow-orange-red icon of the extension on the left taskbar.

The extension works only after the Step 2.4 cell for client initilization is executed. Once the client is live, click on the extension icon, and type in `https://localhost/8889/proxy/8787` in the address bar. It should produce several monitoring options.

More details can be found [here](https://github.com/dask/dask-labextension).
