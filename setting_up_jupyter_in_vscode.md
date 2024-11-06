# Setting Up a Conda Environment for Jupyter Notebook/Lab on NCI + Integrating with Visual Studio Code

This guide will walk you through the steps to create a Conda environment, install Jupyter Notebook/Lab, and ensure that your environment is accessible within Jupyter. This will allow you to work within an isolated environment with specific dependencies for your R/Python projects.

## Prerequisites

- **Anaconda** or **Miniforge** installed on your machine.
  - *I prefer miniforge and recommend running the command below, otherwise install mamba, micromamba, anaconda however you like.*
- Visual Studio Code
  - Extensions: remote-ssh, jupyter, copilot (if you want AI)


```bash
wget "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
```

## Steps to Create the Conda Environment and Install Jupyter

### 1. Create a New Conda Environment

Open a terminal or Anaconda Prompt and run the following command to create a new Conda environment. Replace `myenv` with your preferred environment name and specify the Python version (e.g., `python=3.9`):

```bash
conda create --name myenv python=3.9
```

You will see a list of packages that will be installed in the environment, including the Python version. Type `y` to proceed.

### 2. Activate the Conda Environment

Once the environment is created, activate it with the following command:

```bash
conda activate myenv
```

Your terminal prompt should now reflect that you're working in the `myenv` environment.

### 3. Install Jupyter Notebook

With your environment activated, you can install Jupyter Notebook or JupyterLab. You can choose either depending on your preference:

- To install **Jupyter Notebook**:

```bash
conda install jupyter
```

### 4. Install the IPython Kernel for Jupyter + what other packages you need for your analysis

In order for your Conda environment to be accessible as a kernel in Jupyter, you need to install the `ipykernel` package in your environment:

```bash
conda install ipykernel
```

### 5. Add the Conda Environment as a Jupyter Kernel

After installing `ipykernel`, you can register your Conda environment as a Jupyter kernel so that you can use it within Jupyter Notebook/Lab. Run the following command:

```bash
python -m ipykernel install --prefix=./venv/ --name '<ENV_NAME>' #venv
ipython kernel install --user --name=env_name_kern #conda (conda env must be activated first)
```

- `--name=myenv`: The name of the Conda environment you want to register.
- `--display-name "Python (myenv)"`: The name that will appear in the Jupyter interface.

### 6. Launch an interactive job -- can be non-interactive if you want but it'll require some log additions

example:
```bash
qsub -I -q normalbw -l ncpus=6,mem=24gb,jobfs=20gb,walltime=04:00:00,storage=scratch/ei56+gdata/ei56
```

### 7. Launch Jupyter Notebook or JupyterLab

After setting everything up, you can now launch Jupyter Notebook or JupyterLab:

- For Jupyter Notebook:

  ```bash
  jupyter notebook --no-browser --port=8089 --ip=0.0.0.0
  ```

Copy the urls into a text editor -- you'll need these later.
e.g 
```bash
http://gadi-cpu-bdw-0001.gadi.nci.org.au:8089/lab?token=c00a48e952c4733e94c3c0710ec4f549897bd318939c0802
http://127.0.0.1:8089/lab?token=c00a48e952c4733e94c3c0710ec4f549897bd318939c0802
```
Top link, give you the compute node that you'll need in step 10.
Bottom link is the link you'll use (modified) to give the vscode-jupyter in step 11. 

### 8. Start-up Remote-ssh in VScode (can be done before step 1 if you prefer the terminal in vscode)
*Note: When you use remote-ssh in vscode, you are on the login (head) node.*

Click on the bottom left `><` button and select `connect current window to host` or CMD+shift+P, type `remote-ssh` then `connect current window to host`.

### 9. Start another terminal screen
I like to just split the terminal so I can see both the running jupyter instance on the compute node and the another terminal.

### 10. On the non-jupyter terminal session (on the head/login node), start a ssh-tunnel connecting to the jupyter session on the compute node
Use the following command to create the tunnel to the compute node. Replace `<COMPUTE-NODE>` with the compute node e.g. `gadi-cpu-bdw-0001`

```bash
ssh -N -L 8888:<COMPUTE-NODE>:8089 <COMPUTE-NODE>.gadi.nci.org.au
```
so using this example:

```bash
ssh -N -L 8888:gadi-cpu-bdw-0001:8089 gadi-cpu-bdw-0001.gadi.nci.org.au
```

At this point, the ssh tunnel command will be running. If you need to use bash or anything, start up another terminal session.

### 11. Connect the VScode-Jupyter kernel to the compute node
In vscode, enter the command pallete using CMD+shift+p and type `jupyter` then select `Create interactive window`.

If a window that says you should install ipykernel pops up, just close it. In the jupyter window, click the top right kernel (it may have `Python version-id`). Then select another kernel. Select existing jupyter kernel and paste the url from step 7 **but replace the port with the tunneled port**


E.g.
```bash
http://127.0.0.1:8089/lab?token=c00a48e952c4733e94c3c0710ec4f549897bd318939c0802
```
to
```bash
http://127.0.0.1:8888/lab?token=c00a48e952c4733e94c3c0710ec4f549897bd318939c0802
```

## 12. Quick python test
In the jupyter interactive window. Copy and run this little script. This assumes you have matplotlib and numpy in your environment. 

```python
import matplotlib.pyplot as plt
import numpy as np

# Create figure and axis
fig, ax = plt.subplots()

face = plt.Circle((0.5, 0.5), 0.4, color='yellow', ec='black')
ax.add_patch(face)

left_eye = plt.Circle((0.35, 0.65), 0.05, color='black')
right_eye = plt.Circle((0.65, 0.65), 0.05, color='black')
ax.add_patch(left_eye)
ax.add_patch(right_eye)

mouth_x = np.linspace(0.3, 0.7, 100)
mouth_y = 0.4 - 0.1 * np.sin(np.pi * (mouth_x - 0.3) / 0.4)
ax.plot(mouth_x, mouth_y, color='black')

# Set limits and aspect
ax.set_xlim(0, 1)
ax.set_ylim(0, 1)
ax.set_aspect('equal')

# Remove axes
ax.axis('off')

# Show plot
plt.title("You did it!")
plt.show()
```


## Troubleshooting

- **Jupyter not showing the environment kernel**: Ensure you have installed `ipykernel` and run the `python -m ipykernel install` command correctly. Restart Jupyter after doing this.
- **Missing packages**: If you need additional packages in your environment, you can install them via Conda or `pip`:

  ```bash
  conda install numpy pandas matplotlib
  ```

  Or:

  ```bash
  pip install <package-name>
  ```
