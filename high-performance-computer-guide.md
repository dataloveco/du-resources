# HPC Quick Guide (DU Environment · Mac + JupyterLab)

## 1. Connect to the HPC (Terminal on Mac)

```bash
# from your local Mac terminal
ssh [username]@login.rc.du.edu
```

- Replace `login.rc.du.edu` with the actual login node your professor provided (sometimes `hpc.du.edu` or similar).
- First login may ask to accept a fingerprint → type `yes`.

## 2. File Management (move notebooks / scripts)

From your **Mac local terminal**:

```bash
# upload file to HPC
scp my_notebook.ipynb [username]@login.rc.du.edu:/path/to/workdir/

# download file back
scp [username]@login.rc.du.edu:/path/to/workdir/results.csv ~/Downloads/
```

**Tip:** Use a project/work directory in your HPC home or scratch folder (`/scratch/[username]/` if available).

## 3. Loading Software (modules)

HPCs use *environment modules*:

```bash
module avail             # see available modules
module load anaconda     # load Python/conda environment
module list              # check what's loaded
```

## 4. Starting an Interactive Session

Login nodes are **not** for running big jobs. Start an interactive compute session:

```bash
srun --partition=short --nodes=1 --ntasks=1 --cpus-per-task=4 --mem=16G --time=02:00:00 --pty bash
```

- `--partition=short` → pick queue/partition (short/long/GPU, depending on HPC policy)
- `--time=02:00:00` → job time limit
- Inside the session, load modules and run your code interactively.

## 5. Batch Jobs with SLURM

Create a job script `job.slurm`:

```bash
#!/bin/bash
#SBATCH --job-name=test_job
#SBATCH --partition=short
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --time=02:00:00
#SBATCH --output=out_%j.txt

module load anaconda
python my_script.py
```

Submit it:

```bash
sbatch job.slurm      # submit
squeue -u [username]  # check your jobs
scancel <jobid>       # cancel job
```

## 6. Running Jupyter Lab on HPC

### Step A – Start Jupyter on HPC (in interactive session)

```bash
module load anaconda
jupyter lab --no-browser --port=8888
```

Copy the long tokenized URL shown in the output.

### Step B – On your Mac, forward the port

Open a new local terminal:

```bash
ssh -N -L 8888:localhost:8888 [username]@login.rc.du.edu
```

### Step C – Open browser

Go to http://localhost:8888 on your Mac, paste the token.  
Now you're using Jupyter Lab with HPC compute.

## 7. Good Practices

- Always work in `scratch` or project directories, not home (to avoid quota issues).
- Use **batch jobs** for training large models. Interactive mode is for debugging.

Monitor usage:

```bash
squeue -u [username]
seff <jobid>
```

- Clean up outputs & checkpoints before syncing back to GitHub.