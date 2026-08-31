# An Administration Handbook of AI Hub's HPC Cluster

## Login to the cluster

### Apply for an account

Before logging in for the first time, submit the appropriate account application form:

- **Faculty and teaching/research staff:** [Application form](https://forms.cloud.microsoft/r/E6EC3JqP8G) (sign in with your `@cuhk.edu.hk` account).
- **Research postgraduate (RPG) students:** [Application form](https://forms.cloud.microsoft/r/iczFc8p4di) (sign in with your `@link.cuhk.edu.hk` account).

### Direct login using SSH command

You can use the `ssh` command to log in to the cluster's management node (137.189.75.110). The login command is: 

```
    ssh <username>@<hostname>
```

On the first login, the system will prompt you to change your password. Please follow the instructions to modify your it.

### Remote login using VS Code

You can also use plugins provided by VS Code (or Cursor, etc.) to log in remotely. In VS Code, you can use Remote-SSH plugin:

1. Search and install the `Remote-SSH` plugin.

2. Add a remote host. 

   In the Remote Explorer on the left, click the "+" icon, select "Add New SSH Host", then enter the corresponding ssh command. 
   
   After entering the command, you will be asked to select the SSH configuration path. Choose according to your actual situation.
![](./pics/vscode_login_0.png)
![](./pics/vscode_login_2.png)

3. After adding the remote host, you can select the corresponding host from the list on the left and start the connection.
![](./pics/vscode_login_1.png)

   If asked to choose the platform type of the remote host, select `linux`.
![](./pics/vscode_login_3.png)

    Enter the password when prompted.
![](./pics/vscode_login_4.png)

4. The first connection usually takes from tens of seconds to a few minutes. Once the connection is successfully established, you can choose to open a directory on the server and begin your subsequent work.
![](./pics/vscode_login_5.png)

### Upload data

For small amounts of data, you can upload directly by dragging and dropping in VS Code.

For large amounts of data, you can use the `scp` command to upload. Additionally, there are plugins in VS Code that support `SFTP`; you can search for them in the plugin marketplace.

## Prepare your environment

### Load modules

For some modules in the system, users can load the corresponding items according to their actual needs, such as anaconda, cuda, etc. 

The following are introductions to some commonly used commands.

- `module avail`: List all available modules.
![](./pics/module_avail.png)

- `module list`: List currently loaded modules.
![](./pics/module_list.png)

- `module load <module_name>`: Load the specified module. 

  Generally, `anaconda`, `cuda`, `nvhpc`, etc. are commonly used modules. If you cannot remember the exact module name, you can use the `Tab` key for auto-completion.
![](./pics/module_load.png)

- `module unload <module_name>`: Unload the specified module.

### Prepare Conda environment

After loading the anaconda module using the `module` command, you can normally perform all anaconda operations:

- `conda create -n <env_name> python=<version>`: Create a virtual environment.

- `conda activate/deactivate`: Activate/deactivate a virtual environment.

After activating the virtual environment, users can install required packages via conda or pip.

## Prepare your scripts and data

Besides writing your Python scripts, generally you also need to prepare an sbatch script. 

In this script, users need to perform some basic configurations, such as job name, resource consumption, etc., and call the entry Python script for the business. 

Specific writing methods can refer to the examples provided.
![](./pics/sbatch.png)

## Using slurm to submit jobs

### Debug your code

When you need to debug code rather than running long tasks, do not submit scripts directly. You can use 

```
srun --gres=gpu:1 --mem=64G --pty bash
``` 

to request an interactive terminal with GPU, and then operate like a normal server inside.
![](./pics/srun.png)

> ⚠️ Note: Since compute nodes have no internet connection, users must install packages and components requiring internet access in advance on the management node. During debugging, if missing dependencies are discovered, you need to exit back to the management node for installation, then re-enter the compute node for debugging.

> 💡 A tip: If you need to use models from huggingface, you can first try running the Python script on the management node. After the model download completes (you can manually kill the process), modify the model loading part in the code, adding `local_files_only=True`. Specific script content can refer to `examples/infer_with_qwen/infer_with_qwen.py`.

### Submit your job
![](./pics/sbatch_submit.png)

If you find that the job configuration is wrong (refer to [Resource Quota](#resource-quota)), immediately usec`scancel <JOBID>` to cancel it, releasing resources for others.

If a job remains in PD (Pending) status for a long time, use `squeue -j <JOBID>` or `scontrol show job <JOBID>` to check the Reason field. Common reasons include Resources (insufficient resources), Priority (low priority), or QOSMaxNodePerUserLimit (exceeding user quota).

After the job ends, you can check the content of files specified by `#SBATCH --output` and `#SBATCH --error` in the sbatch script to view program output.

### Some common slurm commands

| Command | Description | Common Parameter Examples |
| :--- | :--- | :--- |
| sinfo | View cluster node and partition status | `sinfo -p gpu` (view GPU partition)<br>`sinfo -N` (display by node) |
| squeue | View job queue (queued/running) | `squeue -u username` (view your own jobs)<br>`squeue -t R` (only view running jobs) |
| sbatch | Submit batch job script | `sbatch job.sh`<br>`sbatch -p cpu job.sh` |
| scancel | Cancel/delete job | `scancel <JOBID>`<br>`scancel -u username` (cancel all jobs of a user) |
| sacct | View historical records of finished jobs | `sacct -j <JOBID>` (check specific job)<br>`sacct -u username` |
| scontrol | View detailed information of jobs or nodes | `scontrol show job <JOBID>`<br>`scontrol show node <NODELIST>` |
| srun | Interactive job submission (for debugging) | `srun -p Interactive --pty bash` (request interactive terminal) |

# Resource quota

Computing resources, especially GPUs, are highly limited. Please request only the resources you need. The associated resource quotas are allocated on an individual basis and are provided below for reference.

### Default Resource Allocation

If a resource is not specified in your Slurm job, the following default allocation applies:

| Resource | Default allocation |
| :--- | :--- |
| CPU | 1 core |
| GPU | 0 |
| RAM | 16 GB |

### Resource Limits per User

| User tier | Logical CPUs | Default / max memory (GB) | Max concurrent GPUs / user | Max GPUs / job | Default / max wall time | Preemption role |
| :--- | ---: | :--- | ---: | ---: | :--- | :--- |
| Faculty (Professoriate, Teaching & Research Academic) | 120 | 128 / 480 | 8 | 8 | 24 / 48 hours | Preemptor (bumps lower tiers instantly) |
| Full-time Research Staff (RA, Post-doc) | 120 | 128 / 480 | 8 | 8 | 24 / 36 hours | Preemptable by Faculty |
| Research Postgraduate Students (MPhil-PhD) | 60 | 64 / 240 | 4 | 4 | 12 / 24 hours | Preemptable by all |

If you need a temporary allocation above your quota for a particular resource, submit a request to AI Hub. Approval depends on the actual circumstances.

### Storage Quota

| Location | Quota |
| :--- | ---: |
| `/hpchome/<user>` | 100 GB |
| `/hpclarge/<user>` | 1000 GB （**Note: This folder will be deleted in 15 September 2026, you must move your data to /hpchome before the date.**） |

The personal storage quota is 1000G per user.

## Additional Information

### Fairshare Policy

Job priority is dynamically adjusted based on resource usage. The more resources you have occupied in the last 7 days, the lower your job's priority will be in the queue.

### Local Scratch Space on GPU Nodes

The local scratch space for a job is located at `/mnt/slurm_scratch/job_<JOB_ID>`.

When your job starts, Slurm assigns it a job ID and creates a temporary folder on the GPU node's flash drives at `/mnt/slurm_scratch/job_<JOB_ID>`. Only you can access this folder, and it has a disk quota of 1000GB. Processing data in this folder is much faster than reading from or writing to your home directory at `/hpchome/<user>`.

The scratch folder is deleted immediately when your job finishes or when you exit the job. Before exiting a running job, make sure to copy all important data back to `/hpchome/<user>`. It is better to synchronize important data incrementally and regularly while the job is running.

### Resource Usage Policy

Please release computing resources as soon as you have finished using them. We conduct regular checks and will forcibly terminate jobs that have requested resources but remain idle for more than 2 hours, as well as jobs that have requested GPUs but are only using CPUs.
