# Usage

## 1. Initial Configuration When Accessing the HPC for the First Time

When you access the HPC for the first time, there are a few essential configurations you need to apply to ensure that your environment is properly set up for running jobs and accessing resources efficiently. This section outlines the necessary steps, including modifications to your `.bashrc` file (at /home/user/), which will help optimize the environment and prevent common errors.

To get started, follow these instructions:

1. **Add Micromamba Initialization to `.bashrc`**  
    
    The first step is to ensure that **micromamba** is properly initialized when you log in. Add the following block to your `.bashrc` file:

    ```bash
    # >>> mamba initialize >>>
    # !! Contents within this block are managed by 'micromamba shell init' !!
    export MAMBA_EXE='/data/ucct/bi/pipelines/micromamba/bin/micromamba';
    export MAMBA_ROOT_PREFIX='/data/ucct/bi/pipelines/micromamba';
    __mamba_setup="$("$MAMBA_EXE" shell hook --shell bash --root-prefix "$MAMBA_ROOT_PREFIX" 2> /dev/null)"
    if [ $? -eq 0 ]; then
        eval "$__mamba_setup"
    else
        alias micromamba="$MAMBA_EXE"  # Fallback on help from micromamba activate
    fi
    unset __mamba_setup
    # <<< mamba initialize <<<
    ```
    This setup will automatically initialize micromamba when you open a new terminal session. If the initialization fails, it falls back to an alias for micromamba.

2. **Add Custom Environment Variables to `.bashrc`**
    
    Next, include additional exports to configure language settings, cache directories, and R library paths. Add the following block to your `.bashrc` file:

    ```bash
    ### EXPORTS ###
    export LC_ALL="en_US.UTF8"

    ## Added by me ##
    export NXF_SINGULARITY_CACHEDIR="/data/ucct/bi/pipelines/singularity-images"
    export R_LIBS_USER="/data/ucct/bi/pipelines/r-lib/"
    ```

3. **Add SLURM Aliases to `.bashrc`**
    
    To simplify the process of viewing SLURM job information, you can add the following aliases to your `.bashrc` file. These will allow you to easily check node and job statuses.

    ```bash
    ### SLURM ALIASES ###
    alias si="sinfo -o \"%20P %5D %14F %8z %10m %10d %11l %16f %N\""
    alias sq="squeue -o \"%8i %12j %4t %10u %20q %20P %10Q %5D %11l %11L %50R %10C %c\""
    alias allqueue="watch 'squeue -o \"%7i %75j %8T %10u %5a %10P %8Q %5D %11l %8M %7C %7m %R\" | grep \"RUNNING\"'"
    ```


4. **Create the `buisciii_config.yml` File**
    
    To use the buisciii-tools, you will need to create a configuration file called buisciii_config.yml in your home directory. This file stores the necessary API credentials. Create it with the following content:

    ```yml
    api_user: <user>
    api_password: <password>
    ```