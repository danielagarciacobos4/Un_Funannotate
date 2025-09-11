# Un_Funannotate

### Installation: 

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
mkdir -p $CONDA_PREFIX/etc/conda/activate.d $CONDA_PREFIX/etc/conda/deactivate.d
echo 'export FUNANNOTATE_DB=/home/dgarcia/mendel-nas1/PacBio/Helicops_angulatus_Aug2024/funannotate/02_db/funannotate_db' > $CONDA_PREFIX/etc/conda/activate.d/funannotate.sh
echo 'unset FUNANNOTATE_DB' > $CONDA_PREFIX/etc/conda/deactivate.d/funannotate.sh
```
3) Make sure we downloaded everything okay, and run a test to make sure the installation worked okay

```
funannotate database --show

funannotate test -t all --cpus 8
```

If the test is successful it shows something like this: 
<img width="1596" height="857" alt="Screenshot 2025-08-31 at 5 03 31 PM" src="https://github.com/user-attachments/assets/14703f38-6b80-4c41-818d-01ebaf8eb6f6" />
<img width="839" height="215" alt="Screenshot 2025-08-31 at 5 03 59 PM" src="https://github.com/user-attachments/assets/9d6edd31-dd56-4aba-a3cd-4944350390e5" />

4) We need to install the Trinity package separately. After installing Trinity, we must link the path to our funnanotate environment

```
mamba create -n trinity215 -c bioconda -c conda-forge trinity=2.15.2 -y

conda deactivate

conda activate funannotate

# crea hooks para definir TRINITY_HOME al activar el env
mkdir -p "$CONDA_PREFIX/etc/conda/activate.d" "$CONDA_PREFIX/etc/conda/deactivate.d"

cat > "$CONDA_PREFIX/etc/conda/activate.d/trinity.sh" <<'EOF'
# Trinity instalado en un env aparte (bioconda lo pone en bin)
export TRINITY_HOME="/mendel-nas1/dgarcia/miniforge3/envs/trinity215/bin"
export TRINITYHOME="$TRINITY_HOME"
EOF

echo 'unset TRINITY_HOME; unset TRINITYHOME' > "$CONDA_PREFIX/etc/conda/deactivate.d/trinity.sh"

# recarga y verifica
conda deactivate
conda activate funannotate
echo "$TRINITY_HOME"
"$TRINITY_HOME"/Trinity --version
```

5) Problems with jellyfish?

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

6) 


