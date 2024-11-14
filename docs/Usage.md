# Usage

## 1. Initial Configuration When Accessing the HPC for the First Time

When you access the HPC for the first time, there are a few essential configurations you need to apply to ensure that your environment is properly set up for running jobs and accessing resources efficiently. This section outlines the necessary steps, including modifications to your `.bashrc` file (at /home/user/), which will help optimize the environment and prevent common errors.

To get started, follow these instructions:

1. **Add Micromamba Initialization to `.bashrc`**  
    The first step is to ensure that **micromamba** is properly initialized when you log in. Add the following block to your `.bashrc` file:

   ```bash
   # >>> mamba initialize >>>
   # !! Contents within this block are managed by 'micromamba shell init' !!
   export MAMBA_EXE='/data/bi/pipelines/micromamba/bin/micromamba';
   export MAMBA_ROOT_PREFIX='/data/bi/pipelines/micromamba';
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
    Next, include additional exports to configure language settings, cache directories, and R library paths. Add the following block to your `.bashrc file`:

    ```bash
    ### EXPORTS ###
    export LC_ALL="en_US.UTF8"

    ## Added by me ##
    export NXF_SINGULARITY_CACHEDIR="/data/bi/pipelines/singularity-images"
    export R_LIBS_USER="/data/bi/pipelines/r-lib/"
    ```