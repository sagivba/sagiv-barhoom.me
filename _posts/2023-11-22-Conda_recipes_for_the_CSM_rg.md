---
layout: post
title:  "Conda recipes for the CSM research group"
author: "Sagiv Barhoom"
date:   2023-11-22
categories: Python
background: '/img/posts/anaconda.jpg'
---

# Anaconda Environment Setup Guide

## Recipe 1. Configure Conda

Before starting, ensure that your `.condarc` is properly configured. 
Run the following commands:

```bash
module load anaconda/2023.7

mkdir -p $HOME/conda/pkgs
mkdir -p $HOME/conda/envs/

cp $HOME/.condarc $HOME/.condarc.orig
cp /RG/theochem/software/ours/our_condarc $HOME/.condarc
```
## Recipe 2. Create Your First Python Environment
our first personal environment name is `py11`
If an existing environment named py11 exists, remove it using the instructions in Recipe 6.
Ensure that Recipe 1 has been executed. 

Then, proceed with the following steps:

```bash
module load anaconda/2023.7
cp /RG/theochem/software/ours/conda_env_files/p11.yml $HOME/conda/envs/py11.yml
conda env create --file $HOME/conda/envs/py11.yml --name py11

# Activate the py11 environment
source activate py11

# Verify the environment
which python
python --version
conda list
```

## Recipe 3. Create a Copy of py11 for Customization

Employ this Recipe if you intend to experiment with upgrading existing Python modules or installing new ones for testing purposes.

Create a cloned environment for your personal use and customization:
```bash
module load anaconda/2023.7
conda env create --name cloned_env --clone py11
# cloned_env - is the name of the new environment, choose your name wisely ;-) 
```

## Recipe 4. Export Environment Configuration
To export the environment configuration for cloning on another user usage, run:

```bash
module load anaconda/2023.7
source activate py11
conda env export > p11.2023.11.22.yml
```

## Recipe 5. Clone Environment from Another User
This Recipe will help you to clone the environment based on the exported YAML file, for  cloning on another user account:

```bash
module load anaconda/2023.7
cp /somewhere/env_file.yml $HOME/conda/envs/
conda env create --file $HOME/conda/envs/env_file.yml --name env_name
```

## Recipe 6. Remove Conda Environment
To remove the py11 environment, use the following commands:

```bash
module load anaconda/2023.7
conda info --env
conda env remove --name py11
```





