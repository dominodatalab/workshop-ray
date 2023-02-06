# Ray Tutorial Project

## Contents

* Tutorial Notebooks:
  * 1-Beginner-Ray-Core.ipynb
  * 2-Intermediate-Ray-Core-And-Large-Data.ipynb
  * 3-Intermediate-XGBoost-Ray-Tune.ipynb
* Other Notebooks:
  * admin-notebooks/2-Intermediate-data-gen.ipynb (generates files needed for notebook 2)
  * troubleshooting/check_packages.ipynb (useful during compute environment setup)
  * troubleshooting/* (other miscellaneous troubleshooting notebooks)
* README

## Project and Workspace Setup

1. Fork or Copy this project, or otherwise get these files into a Domino project. Creating a Git-based project pointing to the original repo from Domino or your personal forked copy of the original repo are other options.
2. Mount the Datasets (not needed for Beginner notebook)
   * Navigate to Data in the project sidebar
   * Click on the "Domino Datasets" tab
   * Click "Mount Shared Dataset" and choose
       * `Points-For-Pi-Approximation` for use with `2-Intermediate-Ray-Core-And-Large-Data.ipynb`
       * `CA-Housing` for use with `3-Intermediate-XGBoost-Ray-Tune.ipynb`
       * Ask your admin if these are not available, or see the Prerequisites section of this readme
2. Start a new workspace
    * In Step 1
        * Ensure a suitable Ray Workspace environment (ask your admin, or see the Prerequisites section of this readme)
        * Jupyterlab is recommended for the IDE
        * The smallest available Hardware Tier is recommended
    * No changes in Step 2 are required
    * In Step 3
        * Choose Ray for the cluster
        * Ensure a suitable Ray Cluster environment (ask your admin, or see the Prerequisites section of this readme)
        * Choose 2 workers (no autoscaling needed)
        * The smallest available Hardware tier is recommended for both head and worker nodes.
        * No local storage needed


## Prerequisites

Below are instructions for configuring the required compute environments and datasets.
These prerequisites are intended to be setup once by a Domino admin; users can simply follow the section above.
Some general information can be found in Domino's [documentation](https://docs.dominodatalab.com/en/latest/user_guide/190175/configure-prerequisites/#creating_compute_ray_env).
This tutorial was developed and tested with the following specific compute environment builds.
Comparable environments are likely to work as well, but different package versions cannot be guaranteed to work smoothly.

### Workspace environment

Create an environment using a **custom base image** as follows:
```
quay.io/domino/compute-environment-images:ubuntu18-py3.8-r4.1-domino5.1-standard
```

Add the following to the **Dockerfile Instructions** (only `ray` is required for Beginner notebook):
```
# Install some packages to match the cluster image:
#  rayproject/ray-ml:1.9.2-py38
USER root
### Must install cmake if you install Ray RLlib (or "all", which includes it)!
RUN apt-get update -y && apt-get install -y cmake
RUN pip install \
    ray[all]==1.9.2 \
    modin==0.12.1 \
    pyarrow==7.0.0 \
    tblib==1.7.0 \
    xgboost_ray==0.1.8 \
    xgboost==1.6.2
```

Add the following **Pluggable Workspace Tools**:
```
jupyterlab:
  title: "JupyterLab"
  iconUrl: "/assets/images/workspace-logos/jupyterlab.svg"
  start: [  /opt/domino/workspaces/jupyterlab/start ]
  httpProxy:
    internalPath: "/{{ownerUsername}}/{{projectName}}/{{sessionPathComponent}}/{{runId}}/{{#if pathToOpen}}tree/{{pathToOpen}}{{/if}}"
    port: 8888
    rewrite: false
    requireSubdomain: false
vscode:
  title: "vscode"
  iconUrl: "/assets/images/workspace-logos/vscode.svg"
  start: [ "/opt/domino/workspaces/vscode/start" ]
  httpProxy:
    port: 8888
    requireSubdomain: false
```

### Cluster environment

Create an environment using a **custom base image** as follows:
```
rayproject/ray-ml:1.9.2-py38
```

Add the following to the **Dockerfile Instructions** (not required for Beginner notebook):
```
RUN pip install \
    pyarrow==7.0.0 \
    xgboost_ray==0.1.8 \
    xgboost==1.6.2

# Below is needed to avoid error (may only occur on longer Tune runs):
# KeyError: 'getpwuid(): uid not found: 12574'
USER root
RUN \
  groupadd -g 12574 ubuntu && \
  useradd -u 12574 -g 12574 -m -N -s /bin/bash ubuntu
USER ubuntu
```

No **Pluggable Workspace Tools** are necessary.

Be sure to select Ray for **Supported Cluster Settings**.

### Datasets

For `2-Intermediate-Ray-Core-And-Large-Data.ipynb`:
* If the data is not already available in a shared dataset in your Domino deployment you can create it as follows
* Create a Dataset in this project (suggested name `Points-For-Pi-Approximation`).
* Once created, launch a Workspace on Large hardware (no Ray cluster needed) with the Ray compute environment.
* Run the admin notebook `2-Intermediate-data-gen.ipynb` to generate the relevant files in the dataset.

For `3-Intermediate-XGBoost-Ray-Tune.ipynb`:
* If the data is not already available in a shared dataset in your Domino deployment you can create it as follows
* Create a Dataset in this project (suggested name `CA-Housing`).
* Once created, launch a Workspace on Large hardware (no Ray cluster needed) with the Ray compute environment.
* Run the first few cells in `3-Intermediate-XGBoost-Ray-Tune.ipynb` to generate the relevant files in the dataset.
