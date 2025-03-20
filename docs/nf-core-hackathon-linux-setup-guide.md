# Linux Setup Guide  

To ensure all participants have the necessary tools for the nf-core hackathon, please follow the steps below to set up your environment on a Linux system.

- ✅ Make sure to complete the [pre-hackathon checklist](https://nf-co.re/events/2025/hackathon-march-2025#pre-hackathon-checklist).

Below, you will find a guide to successfully install all required dependencies.


## 1. Install Miniconda  

Miniconda is a minimal installer for conda, a package manager that simplifies package installation and management.  

### **Steps:**  

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


## 2. Set Up Conda Environment with Nextflow and nf-core  

It's recommended to create a dedicated conda environment for Nextflow and nf-core to prevent version conflicts and maintain a clean workspace.  

### **Steps:**  

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


## 3. Choose a Containerization Tool: Docker or Singularity  

Nextflow pipelines utilize containerization tools to manage dependencies. You can choose between Docker or Singularity based on your system preferences and requirements.  

### **Option A: Install Docker**  

#### **Steps:**  

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

**Note:** After adding your user to the Docker group, it's advisable to log out and log back in to apply the changes.  

### **Option B: Install Singularity within the Conda Environment**  

#### **Steps:**  

1. **Activate the nf-core Environment:**  
   ```bash
   conda activate nf-core-env
   ```
2. **Install Singularity:**  
   ```bash
   conda install -c conda-forge singularity
   ```
3. **Verify Installation:**  
   ```bash
   singularity --version
   ```

**Note:** Installing Singularity within the conda environment ensures that it's isolated and doesn't interfere with system-wide settings.  

---

## 4. Run a Test Pipeline  

To ensure that everything is correctly installed and working, run the following test pipeline. Make sure you have your conda environment activated.

### **Steps:**  

1. **Execute the following command:**  
   ```bash
   nextflow run nf-core/testpipeline -profile test,docker --outdir results
   ```
   *(If using Singularity, replace `docker` with `singularity` in the command.)*  

2. **Expected Output:**  
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

If you see this output (or similar), your installation is complete and ready for the hackathon.  


## 5. Install Visual Studio Code (VSCode)  

To install VSCode is your linux system, please follow the vscode installation guide: [ the recommended code editor for the hackathon.  ](https://code.visualstudio.com/docs/setup/linux)


**Recommended VSCode Extensions:**  
Once VScode extension is installed supercharge it with the nf-core extensions pack: 

- [nf-core/vscode-extensionpack](https://marketplace.visualstudio.com/items?itemName=nf-core.nf-core-extensionpack)


**References:**  

- [nf-core Documentation](https://nf-co.re/)  
- [Nextflow Documentation](https://www.nextflow.io/docs/latest/index.html)  
- [Singularity Documentation](https://sylabs.io/guides/3.0/user-guide/installation.html)  
- [Docker Documentation](https://docs.docker.com/get-docker/)  
- [VSCode Documentation](https://code.visualstudio.com/docs)  
