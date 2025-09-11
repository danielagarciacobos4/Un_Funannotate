# Un_Funannotate

### 1) Installation: 

1) Create a conda environment named funannotate. I installed with mamba because it is faster
```
mamba create -n funannotate -c conda-forge -c bioconda funannotate

```
Keep in mind: 

- <img width="1074" height="690" alt="Screenshot 2025-08-31 at 3 30 15 PM" src="https://github.com/user-attachments/assets/0a631334-81ea-42eb-81a3-57ed656db891" />
- Path of folder: /home/dgarcia/mendel-nas1/PacBio/Helicops_angulatus_Aug2024/funannotate/
- Structure of folder: 

<img width="730" height="591" alt="Screenshot 2025-08-31 at 4 00 56 PM" src="https://github.com/user-attachments/assets/39e001b3-7298-4ea4-890e-1d97a2b6e000" />

2) Download the Funannotate databases:
```
# downloading the funannotate database
funannotate setup -d /home/dgarcia/mendel-nas1/PacBio/Helicops_angulatus_Aug2024/funannotate/02_db/funannotate_db -i all

# This command just makes FUNANNOTATE_DB persistent in the environment. In orther words this step ensures that the environment variable FUNANNOTATE_DB (which stores the path to my Funannotate database) is automatically set every time I activate this conda environment — so I don’t need to manually export it in every new terminal session:

mkdir -p $CONDA_PREFIX/etc/conda/activate.d $CONDA_PREFIX/etc/conda/deactivate.d
echo 'export FUNANNOTATE_DB=/home/dgarcia/mendel-nas1/PacBio/Helicops_angulatus_Aug2024/funannotate/02_db/funannotate_db' > $CONDA_PREFIX/etc/conda/activate.d/funannotate.sh
echo 'unset FUNANNOTATE_DB' > $CONDA_PREFIX/etc/conda/deactivate.d/funannotate.sh
```
3) Make sure we downloaded everything okay, and run a test to make sure the installation worked okay

```
funannotate database --show

funannotate test -t all --cpus 8
```

If the test is successful, it shows something like this (I eliminated these files after running the test: 
<img width="1596" height="857" alt="Screenshot 2025-08-31 at 5 03 31 PM" src="https://github.com/user-attachments/assets/14703f38-6b80-4c41-818d-01ebaf8eb6f6" />
<img width="839" height="215" alt="Screenshot 2025-08-31 at 5 03 59 PM" src="https://github.com/user-attachments/assets/9d6edd31-dd56-4aba-a3cd-4944350390e5" />


4) We need to install the Trinity package. After installing Trinity, we must link the path to our funannotate environment (in this case, I installed Trinity in a separate Conda environment, but it can also be installed within the Funannotate environment). This step is just about making sure that your funannotate environment always knows where to find Trinity — no matter which terminal session you are in. By writing these small activation/deactivation scripts, you make the setup automatic and avoid having to type export TRINITY_HOME=... manually every time.

- I later had to change the path of Perl from the Trinity environment into the Funannotate environments because Perl was being called from Trinity for other packages and dependencies installed within funannotate (keep this in mind Garcia- this was a lot of troubleshooting for you).

```
mamba create -n trinity215 -c bioconda -c conda-forge trinity=2.15.2 -y

conda deactivate

conda activate funannotate

#Creates the special activate.d and deactivate.d folders (if they don’t already exist):

mkdir -p "$CONDA_PREFIX/etc/conda/activate.d" "$CONDA_PREFIX/etc/conda/deactivate.d"

#Create an activation script that sets TRINITY_HOME:

cat > "$CONDA_PREFIX/etc/conda/activate.d/trinity.sh" <<'EOF'
export TRINITY_HOME="/mendel-nas1/dgarcia/miniforge3/envs/trinity215/bin"
export TRINITYHOME="$TRINITY_HOME"
EOF

echo 'unset TRINITY_HOME; unset TRINITYHOME' > "$CONDA_PREFIX/etc/conda/deactivate.d/trinity.sh"

# Reload the environment and verify:
conda deactivate
conda activate funannotate
echo "$TRINITY_HOME"
"$TRINITY_HOME"/Trinity --version
```
5) Download GeneMark software in this [link](http://topaz.gatech.edu/GeneMark/license_download.cgi). I downloaded the software GeneMark-ES/ET/EP+ ver 4.72_lic and the operating system LINUX 64 kernel 3.10 - 5. Also, it is important to download the key and then configure permissions in the cluster. As seen in the scripts of the second section of the pipeline (fun-predict) I used Jon's Hoffmans path for genemark (not sure why mine was not working). However, I did have to download the key license and give it access. 

6) I had problems with jellyfish because my job was trying to read jellyfish from the funannotate environment instead of the trinity environment. The following commands are just adjusting the paths so that jellyfish is read from the trinity environment.

```
# activa funannotate
source /mendel-nas1/dgarcia/miniforge3/etc/profile.d/conda.sh
conda activate funannotate

# añade Trinity + Jellyfish del env trinity215 al PATH al activar funannotate
mkdir -p "$CONDA_PREFIX/etc/conda/activate.d" "$CONDA_PREFIX/etc/conda/deactivate.d"

cat > "$CONDA_PREFIX/etc/conda/activate.d/trinity_path.sh" <<'EOF'
export TRINITY_HOME="/mendel-nas1/dgarcia/miniforge3/envs/trinity215/bin"
export TRINITYHOME="$TRINITY_HOME"
export PATH="$TRINITY_HOME:$PATH"
EOF

echo 'unset TRINITY_HOME; unset TRINITYHOME' > "$CONDA_PREFIX/etc/conda/deactivate.d/trinity_path.sh"

# recarga y verifica
conda deactivate && conda activate funannotate
which Trinity
which jellyfish
jellyfish --version
```

### 2) Training step: 

After making sure that we have configured all the paths and all the packages needed are installed, we run the first step of training in funannotate. Make sure that when you write "funannotate check --show-versions" you see that "All 11 python packages installed", "All 27 Perl modules installed", and "All 6 environmental variables are set". 

Now we can run the first step training which is processing the RNA-seq reads so they can be used for genome-guided transcriptome assembly (clean and normalize your RNA-seq reads):

```
#!/bin/sh
#SBATCH --job-name funtrain
#SBATCH --nodes=1
#SBATCH --tasks-per-node=10
#SBATCH --mem=100gb
#SBATCH --time=100:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=dgarcia@amnh.org


source ~/.bash_profile
conda activate funannotate

cd /home/dgarcia/mendel-nas1/PacBio/Helicops_angulatus_Aug2024/funannotate/03_train

export FUNANNOTATE_DB=/home/dgarcia/mendel-nas1/PacBio/Helicops_angulatus_Aug2024/funannotate/02_db/funannotate_db
export GENEMARK_PATH=/home/jhoffman1/mendel-nas1/fasciatus_genome/funannotate/gmes_linux_64_4

#Train to align RNA-seq data, run Trinity, and then run PASA

# Rutas (ajusta GENOME_FA si el nombre difiere)
GENOME_FA="/home/dgarcia/mendel-nas1/PacBio/Helicops_angulatus_Aug2024/funannotate/01_inputData/Helicops_angulatus_18Oct2024.softmasked.fasta"
OUT_DIR="/home/dgarcia/mendel-nas1/PacBio/Helicops_angulatus_Aug2024/funannotate/03_train/fun_train_out"

# Ejecuta funannotate train con la lista explícita (separada por comas)
funannotate train \
  -i "${GENOME_FA}" \
  -o "${OUT_DIR}" \
    --left IAvH-CT-36861-HBackup_R1_001.fastq.gz IAvH-CT-36861-KBackup_R1_001.fastq.gz IAvH-CT-36861-LiBackup_R1_001.fastq.gz IAvH-CT-36861-LuBackup_R1_001.fastq.gz IAvH-CT-36861-RTBac$
    --right IAvH-CT-36861-HBackup_R2_001.fastq.gz IAvH-CT-36861-KBackup_R2_001.fastq.gz IAvH-CT-36861-LiBackup_R2_001.fastq.gz IAvH-CT-36861-LuBackup_R2_001.fastq.gz IAvH-CT-36861-RTBa$
    --species "Helicops angulatus" \

```

- Since my script failed after trinity (I had to change the path of perl to the funannotate environment), I had to relaunch the script. However, since I already had the trinity results, I did not have to repeat this step. Here is the script I used so that the pipeline would proceed from the already existing assembled transcripts:

```
#!/bin/sh
#SBATCH --job-name funtrain
#SBATCH --nodes=1
#SBATCH --tasks-per-node=15
#SBATCH --mem=100gb
#SBATCH --time=100:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=dgarcia@amnh.org


source ~/.bash_profile
conda activate funannotate

cd /home/dgarcia/mendel-nas1/PacBio/Helicops_angulatus_Aug2024/funannotate/03_train

export FUNANNOTATE_DB=/home/dgarcia/mendel-nas1/PacBio/Helicops_angulatus_Aug2024/funannotate/02_db/funannotate_db
export GENEMARK_PATH=/home/dgarcia/mendel-nas1/PacBio/Helicops_angulatus_Aug2024/funannotate/gmes_linux_64_4

#Train to align RNA-seq data, run Trinity, and then run PASA

# Rutas
GENOME_FA="/home/dgarcia/mendel-nas1/PacBio/Helicops_angulatus_Aug2024/funannotate/01_inputData/Helicops_angulatus_18Oct2024.softmasked.fasta"
OUT_DIR="/home/dgarcia/mendel-nas1/PacBio/Helicops_angulatus_Aug2024/funannotate/03_train/fun_train_out"
TRINITY_MERGED="/home/dgarcia/mendel-nas1/PacBio/Helicops_angulatus_Aug2024/funannotate/03_train/fun_train_out/training/trinity.fasta"

# Ejecuta funannotate train con la lista explícita (separada por comas)
funannotate train \
  -i "${GENOME_FA}" \
  -o "${OUT_DIR}" \
  --trinity "$TRINITY_MERGED" \
  --no_normalize_reads --no_trimmomatic \
  --species "Helicops angulatus" \
  --max_intronlen 6000 --stranded RF \
  --cpus $SLURM_NTASKS_PER_NODE

```
- I obtained an error in the transdecoder, which I was able to fix by adjusting the path:

```
# 1) Verifica dónde está el script
echo "$CONDA_PREFIX"
ls -l "$CONDA_PREFIX/opt/transdecoder/util/cdna_alignment_orf_to_genome_orf.pl"

# 2) Añade TransDecoder util (y scripts de PASA) al PATH para esta sesión
export PASAHOME="$CONDA_PREFIX/opt/pasa-2.5.3"
export PATH="$CONDA_PREFIX/opt/transdecoder/util:$PASAHOME/scripts:$PATH"

# 3) Comprueba que ahora se ve:
which cdna_alignment_orf_to_genome_orf.pl
```




