# QIIME 2 Installation

QIIME 2 installation tutorials are widespread on the internet, starting with the QUIIME 2 official documentation: 
https://docs.qiime2.org/2023.2/. 
Following is the code I used for QIIME 2 version 2023.2 installation.

---

## Installing Miniconda
### First, Download and install Miniconda.

```
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
```
```
bash Miniconda3-latest-Linux-x86_64.sh
```
### Update Miniconda.
```
conda update conda
```
## Installing QIIME2
### Download and install QIIME2.
```
wget https://data.qiime2.org/distro/core/qiime2-2023.2-py38-linux-conda.yml
```
```
 conda env create -n qiime2-2023.2 --file ./work/programs/qiime2-2023.2-py38-linux-conda.yml
 ```
### Test the installation.
#### Activate QIIME 2.
```
conda activate qiime2-2023.2
```
#### Run a simple command.

```
quiime --help
```
If the command works, then QUIIME 2 is installed.