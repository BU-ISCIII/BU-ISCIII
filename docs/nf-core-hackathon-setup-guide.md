# nf-core Hackathon: Setup Guide

To ensure all participants have the necessary tools for the nf-core hackathon, please follow the steps below to set up your environment based on your operating system.

- ✅ Make sure to complete the [pre-hackathon checklist](https://nf-co.re/events/2025/hackathon-march-2025#pre-hackathon-checklist).

## Linux Setup Guide

### 1. Install Miniconda

Miniconda is a minimal installer for conda, a package manager that simplifies package installation and management.

#### **Steps:**

1. **Download Miniconda Installer:**  

   ```bash
   wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
   ```

2. **Run Installer:**

   ```bash
   bash Miniconda3-latest-Linux-x86_64.sh
   ```

   - Follow the on-screen instructions.  
3. **Initialize Conda:**  

   ```bash
   source ~/.bashrc
   ```

### 2. Set Up Conda Environment with Nextflow and nf-core

It's recommended to create a dedicated conda environment for Nextflow and nf-core to prevent version conflicts and maintain a clean workspace.

#### **Steps:**

1. **Add Bioconda and Conda-Forge Channels:**  

   ```bash
   conda config --add channels bioconda
   conda config --add channels conda-forge
   ```

2. **Create and Activate Environment:**  

   ```bash
   conda create --name nf-core-env nextflow nf-core
   conda activate nf-core-env
   ```

3. **Verify Installations:**  

   ```bash
   nextflow -version
   nf-core --version
   ```

### 3. Choose a Containerization Tool: Docker or Singularity

Nextflow pipelines utilize containerization tools to manage dependencies. You can choose between Docker or Singularity based on your system preferences and requirements.

#### **Option A: Install Docker**

**Steps:**

1. **Update Package List:**  

   ```bash
   sudo apt update
   ```

2. **Install Docker:**  

   ```bash
   sudo apt install -y docker.io
   ```

3. **Add User to Docker Group:**  

   ```bash
   sudo usermod -aG docker $USER
   ```

4. **Apply Group Changes:**  

   ```bash
   newgrp docker
   ```

5. **Verify Installation:**  

   ```bash
   docker --version
   ```

#### **Option B: Install Singularity within the Conda Environment**

**Steps:**

1. **Update Package List:**  

   ```bash
   sudo apt update
   ```

2. **Install Singularity:**  

   ```bash
   sudo apt install -y singularity-container
   ```

3. **Verify Installation:**  

   ```bash
   singularity --version
   ```

### 4. Run a Test Pipeline

To ensure that everything is correctly installed and working, run the following test pipeline. Make sure you have your conda environment activated.

**Steps:**

1. **Create a dir to test nextflow:**

   ```bash
   mkdir nf-core-tests ; cd nf-core-tests
   ```

2. **Execute the following command:**  

   ```bash
   nextflow run nf-core/testpipeline -profile test,docker --outdir results
   ```

   *(If using Singularity, replace `docker` with `singularity` in the command.)*  

3. **Expected Output:**  

   ```bash
   executor >  local (4)
   [5a/b1987b] NFCORE_TESTPIPELINE:TESTPIPELINE:FASTQC (SAMPLE1_PE) [100%] 3 of 3 ✔
   [1f/c3e18c] NFCORE_TESTPIPELINE:TESTPIPELINE:MULTIQC             [100%] 1 of 1 ✔
   -[nf-core/testpipeline] Pipeline completed successfully-
   Completed at: 20-Mar-2025 10:49:53
   Duration    : 1m 42s
   CPU hours   : (a few seconds)
   Succeeded   : 4
   ```

## 5. Install Visual Studio Code (VSCode)  

To install VSCode is your linux system, please follow the vscode installation guide: [the recommended code editor for the hackathon.](https://code.visualstudio.com/docs/setup/linux)

**Recommended VSCode Extensions:**  
Once VScode extension is installed you can supercharge it with the nf-core extensions pack:

- [nf-core/vscode-extensionpack](https://marketplace.visualstudio.com/items?itemName=nf-core.nf-core-extensionpack)

**References:**  

- [nf-core Documentation](https://nf-co.re/)  
- [Nextflow Documentation](https://www.nextflow.io/docs/latest/index.html)  
- [Singularity Documentation](https://sylabs.io/guides/3.0/user-guide/installation.html)  
- [Docker Documentation](https://docs.docker.com/get-docker/)  
- [VSCode Documentation](https://code.visualstudio.com/docs)  

## Windows Setup Guide

To run nf-core pipelines on Windows, it is **highly recommended** to use **Windows Subsystem for Linux (WSL2)** to enable a Linux-like environment.

### 1. Install Windows Subsystem for Linux (WSL2)

1. **Enable WSL2:**  
   Open PowerShell as Administrator and run:

   ```powershell
   wsl --install
   ```

2. **Restart your computer** when/if prompted.
3. **Verify Installation:**  

   ```powershell
   wsl --list --verbose
   ```

4. **Set WSL2 as Default:**  

   ```powershell
   wsl --set-default-version 2
   ```

5. **Install Ubuntu from the Microsoft Store.**

### 2. Install Miniconda, Nextflow, and nf-core inside WSL

Follow the [Linux Setup Guide](#linux-setup-guide) above within your WSL environment. DO NOT INSTALL singularity nor docker.

### 3. Configure Docker to Work with WSL2

1. Download and install **Docker Desktop** from:  
   [Docker Desktop for Windows](https://docs.docker.com/desktop/setup/install/windows-install/)
2. Enable the [WSL2 Backend](https://docs.docker.com/desktop/features/wsl/) in Docker settings.
3. Make WS2 and docker desktop to work together.

- Open Docker Desktop on Windows.
- Go to Settings → Resources → WSL2 Integration.
- You’ll see a list of your installed WSL2 distros (like Ubuntu, Ubuntu-18.04, etc).
- 👉 Turn ON integration for the distro you're using (probably Ubuntu).
- Click "Apply & Restart" at the bottom.

![WSL2-Docker](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/wsl2_integration_dockerdesktop.png)

4. Add user to docker group

```
sudo usermod -aG docker $USER
```

5. Close WSL2 terminal and open again. You are ready to go!

6. Verify installation by running inside WSL2:

   ```bash
   docker --version
   ```

> You also have this extended guide:
>
> - 📘 [Step-by-step guide: Installing Docker in Windows 11 with WSL2](https://www.linkedin.com/pulse/installing-docker-windows-11-using-wsl-2-step-by-step-ankit-lodaf/)

### 4. Alternative to Docker: Using Singularity in WSL2

If you prefer Singularity over Docker, you can install it **inside your WSL2 Ubuntu environment**. Follow the instructions provided in the [Linux Setup Guide – Option B](#option-b-install-singularity-system-wide).

> ⚠️ **Note:** This option requires additional configuration, and it's not tested. We recommend sticking to docker if you are using Windows!

### 5. Install Visual Studio Code

1. Download and install **VSCode** from:  
   [VSCode for Windows](https://code.visualstudio.com/download)
2. Install the **Remote - WSL** extension:  
   - [Remote - WSL Extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl)
3. Open a WSL session inside VSCode:

   ```powershell
   code .
   ```

### 6. Verify the Setup with a Test Pipeline

Inside WSL, follow the [**Run a Test Pipeline**](#4-run-a-test-pipeline) section from the Linux setup guide.

> Note: a minimum of 15gb is required by default for running the test pipeline. If you don't have so much RAM available you can create a test.config file like this:
```
process {
    resourceLimits = [
        cpus: 1,
        memory: '4.GB',
        time: '1.h'
    ]
}
```
> and run the test command like this 
```
nextflow run nf-core/testpipeline -c test_min.config -profile test,docker --outdir results
```

---

**References:**  

- [nf-core Documentation](https://nf-co.re/)  
- [Nextflow Documentation](https://www.nextflow.io/docs/latest/index.html)  
- [Singularity Documentation](https://sylabs.io/guides/3.0/user-guide/installation.html)  
- [Docker Documentation](https://docs.docker.com/get-docker/)  
- [VSCode Documentation](https://code.visualstudio.com/docs)
