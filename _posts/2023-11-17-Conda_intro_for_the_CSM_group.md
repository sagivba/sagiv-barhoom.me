---
layout: post
title:  "Conda intro for the CSM research group"
author: "Sagiv Barhoom"
date:   2023-11-17
categories: Python
background: '/img/posts/python.jpg'
---


# Conda intro for the CSM research group
[comment]: <>  (built with the help of chatgpt)

Welcome to the exciting journey of learning and growing together as Python developers! 
Here's a guide to some essential topics that will help us navigate through our development tasks seamlessly using conda environments.
We encourage  you to *Read and try while learning from this document*.

## 1.  Assumptions
1. You possess a basic understanding of Python.
2. You are familiar with Anaconda/Miniconda, or at least have heard of it.
3. You have a foundational knowledge of the Linux command line.
4. All commands in this document are after loading our anaconda version
   ```bash
   module load anaconda/2023.7
   ```

## 2.  Some terminology 
### 2.1 What is `conda`?
Conda is a powerful package and environment management system widely used in data science and software development. 
It simplifies the process of installing, managing, and updating libraries, ensuring consistent and reproducible environments for projects. 
Conda's flexibility and ease of use make it a valuable tool for streamlining the development workflow and enhancing collaboration among team members.

### 2.2 What is conda environment
A Conda environment is a self-contained workspace where you can manage and isolate dependencies for specific projects, ensuring consistency and reproducibility across different computing environments. It allows users to create encapsulated environments with their own set of packages and configurations, minimizing conflicts and simplifying the management of project-specific requirements.

### 2.3 What is the `.condarc` file?
The `.condarc` file is a configuration file used by Conda to customize its behavior. 
It allows us to define preferences for package installations, manage channel priorities, and configure various aspects of Conda's functionality, providing a tailored environment for efficient development and collaboration.

The `.condarc` is written in YML format, but don't worry about that right now, keep reading and if you want to learn YAML basics you can [start here](https://www.youtube.com/embed/BEki_rsWu4E?si=s8PNleZ5RnKDjXYh).

### 2.4 How do I find the `.condarc` location?
In our team, it is most likely that it is located in your `$HOME`

You can easily locate the `.condarc` file by running the following command:
```bash
conda config --show-sources
 ```
 Additionally, if you wish to set a custom location for the `.condarc` file, you can do so by defining the `CONDARC` environment variable:
```bash
 export CONDARC=/path/to/your/custom/location/.condarc
 ```
### 2.5 How to edit  `.condarc`
Two options
- use a text editor like vim
- use  `conda config` as seen below
  
### 2.6. Our  `.condarc` Configuration
Here's an example `.condarc` configuration:

```yml
# our basic .condarc Configuration File

# Define proxy servers for Conda package downloads
proxy_servers:
    http: http://proxy.cslab.openu.ac.il:80
    https: http://proxy.cslab.openu.ac.il:80

# Disable SSL verification for package downloads (Note: Use cautiously in secure environments)
ssl_verify: False

# Specify the directory for storing Conda packages
pkgs_dirs:
    - ~/.conda/pkgs

# Specify the directory for storing Conda environments
envs_dirs:
    - $HOME/conda/envs
```


### 3. So let's start with the conda command...
Make your interaction with Conda smooth and efficient with these commands. 

#### 3.1 Helpful Commands
 **Getting Help:**
```bash 
conda --help
conda create --help
```
#### 3.2 Using conda to edit .condarc
We can also `conda config` command to edit  our `.condarc` file
For example,  **changing prompt after activation**:
To display only the environment name and not the path after activating, use the following command:
Take a look at your `.condarc` file before and after
```bash
conda config --set changeps1 False
```

#### 3.3 Create conda enviroment
Here are some options that provide flexibility in tailoring environments to a  project needs.
```bash
conda create --name myenv python=3.10            #  --> Specify Python Version
conda create --name myenv numpy pandas           # --> Install Specific Packages:
conda create --name myenv --file another_env.yml # --> Create from Environment File for clonning
conda create --name newenv --clone existingenv   # --> Clone an Existing Environment
```

#### 3.4 Installing Conda Packages
Installing Conda packages is a breeze. Just use the following command:
```bash
conda install <package-name>
```    

#### 3.5 conda init
Most likely you do not need this on our team.

 **<span style="color: red;">Please! don't run it  unless you understand what your are doing.</span>**
 
Conda init is a command that facilitates the initialization of Conda for seamless integration with your shell environment. 
By running 'conda init', you enable Conda to set up the necessary configurations within your chosen shell (e.g., bash, zsh) so that Conda commands can be conveniently accessed and utilized directly from the command line. 
This initialization process enhances the user experience by ensuring a smooth and integrated workflow when working with Conda-managed environments and packages.
```bash 
conda init --dry-run        # Display what would have been done.
conda init <shell-name>     # Initialize Conda for shell interaction.
conda init --all            # Initialize all currently available shells.
conda init --reverse        # Undo the effects of the last conda init.
```   

#### 3.6 Exporting and Cloning Environments
##### 3.6.1 Our preferred method
- Export Environment Configuration
- Create a New Environment
- Recreate Environment from YAML File
- Clone an Existing Environment
``` bash
conda env export > environment.yml 
conda create --name <env-name>
conda create --name <env-name> --file environment.yaml
conda create --name cloned_env --clone original_env
```

##### 3.6.2 Copying Directories - a less desirable method
Copying directories directly is not advised, but if necessary:
``` bash 
conda info --envs
cp -r path/of/original/env location/of/new/env
```
    
##### 3.6.3 Cloning for self usage 
If you want to clone the environment for self-use (and not for another member of the team):
``` bash 
conda create --name cloned_env --clone original_env
```

### 4. F.A.Q
##### 4..1 Is a central `.condarc` possible?
Yes, indeed!  but most likely you don't need that in our team.
For a centralized configuration, follow these steps:
4.1.1.  Create a Base Configuration File, `base_condarc.yml`:
```yml
channels:
    - defaults
```      

4.1.2.  Create Environment-Specific Configuration Files, e.g., `development_condarc.yml` and `production_condarc.yml`:
```yml
# development_condarc.yaml
!include base_condarc.yaml
channels:
    - conda-forge
```
```yml
# production_condarc.yaml
!include base_condarc.yaml
    channels:
        - my-internal-channel
```      

4.1.3.  Choose your configuration:
```bash
conda config --file development_condarc.yaml
conda config --file production_condarc.yaml
```
The `!include` directive relies on the YAML processor used by Conda, 
so please check the Conda documentation for the latest updates.


## Additional Resources
*   [Adding Conda Environments to Jupyter](https://janakiev.com/blog/jupyter-virtual-envs/)
*   [YAML Tutorial](https://www.cloudbees.com/blog/yaml-tutorial-everything-you-need-get-started)
