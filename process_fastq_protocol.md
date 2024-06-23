# scRNA Sequencing Data Analysis Tutorial
Authors: Ke, Jiho and Sophia


## Table of Contents
- [How to use SSH](#how-to-use-ssh)
- [How to download sequencing data from SRA NCBI](#how-to-download-sequencing-data-from-sra-ncbi)
- [Software Use Guide: Cellranger count](#software-use-guide-cellranger-count)
- [Software User Guide: Velocyto](#software-user-guide-velocyto)
- [Using Slurm](#using-slurm)
- [Using TMUX (under remote server using ssh)](#using-tmux-under-remote-server-using-ssh)

## How to use SSH

### What is SSH?
SSH or Secure Shell is a network protocol that allows users to securely access a computer over an unsecured network. It uses cryptography to encrypt connections between devices and authenticate them. SSH is often used by system administrators for tasks such as:
- Controlling servers remotely
- Managing infrastructure
- Transferring files
- Accessing cloud services
- Logging into remote computers
- Performing operations on remote computers

### Log in to SSH
1. Usually SSH is provided by the institution or university and one can access the SSH and use the software and download the files in the SSH.
2. Download the provided VPN for the SSH which is provided by the institution.
3. In the terminal login to the SSH server by `ssh <username>@cluster.csb.pitt.edu` (mine was `ssh jipark25@cluster.csb.pitt.edu`).
4. Sometimes step 3 doesn’t work due to configuration. So change the configuration by editing `.ssh/config` and change the whole thing into:
   ```
   Host *
   IgnoreUnknown UseKeychain
   ```
5. Then you can log in again and type in the password.
6. You will be in the SSH of the institution and you can check where you are at by typing in `pwd`.
7. Then you can navigate to the directory to wherever you want by typing `cd some_directory`.

### Use of SSH
- Remove file: `rm “file_name”`
- Remove directory: `rm -r “file_name”`
- `pwd`: gives you the current directory pathway
- `ls`: gives you the files in the current directory
- Move files: `mv example.txt /path/to/new_directory`
- Uncompress the fastq file:
  ```
  gunzip -k SRR11073244_1.fastq.gz
  ```
  (You can leave out `-k` so that you delete the `.gz` file and only get the fastq file:
  ```
  gunzip SRR11073244_1.fastq.gz
  ```

### Installing sratoolkit in the SSH
1. Navigate to your directory. (mine was `/net/capricorn/home/xing/jipark25`)
2. Download sratoolkit (It depends on the environment of the SSH and my SSH environment was Linux so I downloaded the Ubuntu version of sratoolkit):
   ```
   wget https://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/current/sratoolkit.current-ubuntu64.tar.gz
   ```
3. Extract the toolkit:
   ```
   tar -xvzf sratoolkit.current-ubuntu64.tar.gz
   ```
4. Then navigate to the bin directory of the sratoolkit directory so that you can download the sra file:
   ```
   prefetch -O /net/capricorn/home/xing/jipark25/sra SRR11073243
   ```
5. Convert the sra file to fastq file:
   ```
   ./fastq-dump --split-files --gzip /net/capricorn/home/xing/jipark25/sra/SRR11073243.sra
   ```

## How to download sequencing data from SRA NCBI
- [YouTube Tutorial](https://www.youtube.com/watch?v=zE851fWCYQg&ab_channel=Bioinformagician)
- NCBI SRA Toolkit (Free): [Download NCBI SRA Toolkit](https://github.com/ncbi/sra-tools/wiki/01.-Downloading-SRA-Toolkit)
- Google Cloud (gcloud) and Amazon S3 are alternative options to SRA NCBI. Ke used S3 and gcloud to download before. However, be careful that S3 and gcloud may require you (who download) to cover the cost.  

### Steps:
1. In terminal make an empty directory in Desktop:
   ```
   mkdir tmp
   ```
2. Uncompress the SRA Toolkit:
   ```
   tar -xyzf sratoolkit.3.1.1-mac-arm64.tar.gz
   ```
3. Configure (go to bin directory of uncompressed directory of SRA Toolkit since most executable files are in bin directory):
   ```
   cd /Users/jihopark/Desktop/sratoolkit.3.1.1-mac-arm64/bin/
   ./vdb-config -i
   ```
   Use Tab to move around and enter to go in. In the Cache tab change the location of user-repository to the tmp file that we made in step 2. Exit and save changes.
4. Download fasterq file from NCBI using tool called `fasterq-dump`:
   ```
   sratoolkit.3.1.1-mac-arm64/bin/fasterq-dump --split-files corresponding_SRR_ID (ex. SRR11073243)
   ```

## Software Use Guide: Cellranger count
(*Depending on what fastq data you have you need to choose which aligning software to use: https://www.10xgenomics.com/support/software/cell-ranger/latest/tutorials)

### Install Cellranger:
[Install Cellranger](https://www.10xgenomics.com/support/software/cell-ranger/latest/tutorials/cr-tutorial-in)

### Downloading reference genome:
[Reference Genome](https://www.10xgenomics.com/support/software/cell-ranger/downloads)

### Configure the path to the cellranger software:
```sh
export PATH=/net/capricorn/home/xing/jipark25/velocyto/cellranger-8.0.1:$PATH
```
(My cellranger software is stored in the velocyto directory so I am pointing to that direction)

### The fastq file name convention for cellranger:
```
SampleName_SampleNumber_LaneNumber_ReadNumber.fastq.gz
(Ex. GSM4307836_S1_L001_R1_001.fastq.gz)
```
** You must compress the fastq file to fastq.gz:
```
gzip name_of_the_file.fastq
```
Sample Name:
The Sample Name is typically derived from the Sample Name field or the Run field from the provided metadata. In this case we will use GSM4307836 as the sample name since it's provided as Sample Name.

Sample Number:
The Sample Number is usually indicated as S1 S2 etc. depending on the number of samples. In my case since there is no explicit sample number we can default to S1 if this is the only sample. If there are multiple samples you would increment this number accordingly.

Lane Number:
The Lane Number is part of the sequencing setup and is not explicitly provided in the metadata. For samples sequenced on Illumina platforms like the NovaSeq 6000 each flow cell typically contains multiple lanes. Without specific lane information we assume a single-lane setup or default to L001 for simplicity.

Read Number:
The Read Number is indicated by R1 for the first read (forward read) and R2 for the second read (reverse read).

### Run the Cellranger count (You need to use Slurm script for below):
(For sample SAMN14088153)
```sh
cellranger count --id=SAMN14088153 \
--transcriptome=/net/capricorn/home/xing/jipark25/velocyto/refdata-gex-GRCm39-2024-A \
--fastqs=/net/capricorn/home/xing/jipark25/kidney_data/SRR11073243/fastq \
--sample=GSM4307836 \
--create-bam=true \
--localcores=8 \
--localmem=128
```
- `cellranger count –id`: it is the directory where the data will be stored (I think)
- `transcriptome`: It’s where the reference file is stored (Download reference file: https://www.10xgenomics.com/support/software/cell-ranger/downloads)
- `fastqs`: where the fastq files that need to be processed are located
- `sample`: the prefix of the fastq files (since my fastq file is GSM4307836_S1_L001_R1_001.fastq.gz I put GSM4307836)
- `create-bam`: You need to create bam file of output so I can use it for next process. (Making it to the AnnData type for Dynamo)
- `localcores`: I don’t know what this is
- `localmem`: It’s local memory allocation that you assign. I assigned 128GB.

## Software User Guide: Velocyto
We will extract the pre-mature (unspliced) and mature (spliced) transcript information based on aligned RNA data (bam file) that is output of cellranger. We need this step since bam file only contains intron-intron intron-exon exon-exon junctions.

### Required software to run velocyto:
- [Velocyto](https://velocyto.org/velocyto.py/install/index.html)
- [Miniconda](https://docs.anaconda.com/miniconda/)
- Python3: built in Linux OS

### Run the Velocyto (You need to use Slurm script for below):
```sh
#!/bin/sh
#SBATCH --job-name=velocyto
#SBATCH --cpus-per-task=64
#SBATCH --mem=64G

#SBATCH --time=1-12
# partition (queue) declaration
#SBATCH --partition=any_cpu

# number of requested nodes
#SBATCH --nodes=1
#SBATCH --ntasks=1
# #SBATCH -w n001

# standard output & error
echo "SLURM NODE:"$SLURMD_NODENAME

# user=$(whoami)
# job_dir=${user}_${SLURM_JOB_NAME}_${SLURM_JOB_ID}.dcb.private.net
# mkdir /scr/$job_dir
# pwd
# cd /scr/$job_dir


# Companion file to the consensus.bam that serves as an external index.
# BAI_PATH=... # USE IF NECESSARY. For cell barcodes, velocyto only needs CB tag contained in BAM, so we do not need to use BAI.

TEMP_ENV_NAME=temp_velocyto_${SLURM_JOB_ID}

which conda
echo "conda tempEnvName:"$TEMP_ENV_NAME

whoami
module load anaconda
conda create -n $TEMP_ENV_NAME python=3.8 --no-default-packages
# fix needed due to cluster issue...
# https://stackoverflow.com/questions/61915607/commandnotfounderror-your-shell-has-not-been-properly-configured-to-use-conda
source /opt/anaconda3-cluster/etc/profile.d/conda.sh
# source activate $TEMP_ENV_NAME
conda activate $TEMP_ENV_NAME
PIP=/net/capricorn/home/xing/ken67/.conda/envs/$TEMP_ENV_NAME/bin/pip
VELOCYTO_EXEC=/net/capricorn/home/xing/ken67/.conda/envs/$TEMP_ENV_NAME/bin/velocyto
FILTERED_BARCODES=/net/capricorn/home/xing/ken67/external_collaborators/upmc/cellranger_runs/allo1/outs/filtered_feature_bc_matrix/barcodes.tsv.gz
OUT=./velocyto_output/allo1
BAM_PATH=/net/capricorn/home/xing/ken67/external_collaborators/upmc/cellranger_runs/allo1/outs/possorted_genome_bam.bam
echo BAM_PATH:$BAM_PATH
ls $BAM_PATH

# for mouse
# GENE_GTF_PATH=/net/capricorn/home/xing/ken67/cellranger_2022_may/refdata-gex-mm10-2020-A/genes/genes.gtf
# for human
GENE_GTF_PATH=/net/capricorn/home/xing/ken67/cellranger_2022_may/refdata-gex-mm10-2020-A/genes/genes.gtf


$PIP install numpy scipy cython numba matplotlib scikit-learn h5py click pandas
$PIP install velocyto

ls $VELOCYTO_EXEC
$VELOCYTO_EXEC run \
  -b $FILTERED_BARCODES\
  -o $OUT \
  $BAM_PATH  \
  $GENE_GTF_PATH \

conda env remove -n $TEMP_ENV_NAME
```

## Using Slurm
SLURM (Simple Linux Utility for Resource Management) is an open-source workload manager used to allocate resources on a cluster of computers. It's commonly used in high-performance computing (HPC) environments to schedule jobs manage resources and ensure efficient execution of computational tasks.

### Create a Slurm script:
```sh
nano name_of_script.sh
```

### Slurm Structure:
#### Shebang Line: This specifies the script interpreter.
```sh
#!/bin/bash
```

#### SLURM Directives: These are special comments that provide SLURM with instructions on how to schedule and run the job.
```sh
#SBATCH --job-name=my_job          # Name of the job 
#SBATCH --output=output_file.txt   # File to write the output to 
#SBATCH --error=error_file.txt     # File to write the error to 
#SBATCH --time=01:00:00            # Maximum runtime (hh:mm:ss) 
#SBATCH --ntasks=1                 # Number of tasks (processes) 
#SBATCH --cpus-per-task=4          # Number of CPU cores per task 
#SBATCH --mem=4G                   # Memory per node 
#SBATCH --partition=standard       # Partition to submit to
```

#### Job Commands: These are the actual commands to be executed by the script.
```sh
# Load necessary modules 
module load some_module 
# Run your program 
./my_program
```

### Submitting the SLURM Script
```sh
sbatch my_script.sh
```

### Checking Job Status
```sh
squeue -u your_username
```

### Cancelling a Job
```sh
scancel job_id
```

*There will be multiple datasets that need to be processed through cellranger, velocyto, or other software in the 10X genomics. So one might want to parallelly process the data using a terminal multiplexer (tmux).

## Using TMUX (under remote server using ssh)
[Tmux Guide](https://hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)
Tmux’s authors describe it as a terminal multiplexer. Behind this fancy term hides a simple concept: Within one terminal window you can open multiple windows and split-views (called “panes” in tmux lingo). Each pane will contain its own independently running shell instance (bash zsh whatever you’re using). This allows you to have multiple terminal commands and applications running side by side without the need to open multiple terminal emulator windows.
On top of that tmux keeps these windows and panes in a session. You can exit a session at any point. This is called “detaching”. tmux will keep this session alive until you kill the tmux server (e.g. when you reboot). This is incredibly useful because at any later point in time you can pick that session up exactly from where you left it by simply “attaching” to that session.
If you’ve ever worked with remote servers or a Raspberry Pi over ssh you can guess where this will be useful: When you lose your ssh connection the tmux session will simply be detached but will keep running on the server in the background including all the processes that run within your session. To continue your session simply ssh to the server again and attach to the running session.
tmux is helpful when working on a remote machine but it shines just as much when you’re working locally. Not only for its window management features but also for the session handling. Personally I find myself detaching from sessions when I’m switching context. I’ll just start a new session for my new task and attach to the old session whenever I want to continue with my old task.

### How to Install in remote server (ssh)
1. Create a source directory:
   ```sh
   mkdir -p /net/capricorn/home/xing/jipark25/src
   cd /net/capricorn/home/xing/jipark25/src
   ```
2. Download and compile ‘libevent’ ‘ncurses’ and ‘tmux’
   #### Downloading libevent
   ```sh
   wget https://github.com/libevent/libevent/releases/download/release-2.1.12-stable/libevent-2.1.12-stable.tar.gz
   tar -xzf libevent-2.1.12-stable.tar.gz
   cd libevent-2.1.12-stable
   ./configure --prefix=$HOME/local
   make
   make install
   ```
   #### Downloading ncurses
   ```sh
   cd ~/local/src
   wget https://ftp.gnu.org/pub/gnu/ncurses/ncurses-6.3.tar.gz
   tar -xzf ncurses-6.3.tar.gz
   cd ncurses-6.3
   ./configure --prefix=$HOME/local
   make
   make install
   ```
   #### Downloading tmux
   ```sh
   wget https://github.com/tmux/tmux/releases/download/3.3a/tmux-3.3a.tar.gz
   tar -xzf tmux-3.3a.tar.gz
   cd tmux-3.3a
   ./configure CFLAGS="-I$HOME/local/include -I$HOME/local/include/ncurses" LDFLAGS="-L$HOME/local/lib" --prefix=$HOME/local
   make
   make install
   ```

### Basic tmux usage
- Start a new session: `tmux`
- List sessions: `tmux ls`
- Detach from a session: `Ctrl+b then d`
- Attach to a specific session: `tmux attach -t mysession`
- Kill a session: `tmux kill-session -t mysession`
- Attach to a session: `tmux attach -t 0`
- Make a new window: `Ctrl+b then c`
- Switch between windows:
  - `Ctrl+b then n`  # Next window
  - `Ctrl+b then p`  # Previous window
  - `Ctrl+b then [0-9]`  # Specific window

### Example Workflow
```sh
tmux
Long running command
Detach from a session: Ctrl+b d
Later you can go back to working session
tmux ls
tmux attach -t detached_session
```

### How It Works
When you run a tmux session on a remote server the processes you start within that session are executed on the remote server itself not on your local machine. Here’s how this setup works:
#### Local Machine:
- Your local machine (laptop or desktop) is used to initiate an SSH connection to the remote server.
- Once connected you start a tmux session on the remote server.
- Any commands or processes you run within tmux are executed on the remote server.

#### Remote Server:
- The remote server is a computer (or a cluster of computers) managed by your university or organization.
- The server continues to run the tmux session and all associated processes even if your local machine disconnects.