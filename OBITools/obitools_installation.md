# OBITools installation

## Install OBITools 3

From: https://git.metabarcoding.org/obitools/obitools3/-/wikis/Installing-the-OBITools3

```
python3 -m venv obi3-env
source obi3-env/bin/activate
pip3 install --upgrade pip setuptools wheel Cython
pip3 install OBITools3
```


## Test OBITools

```
obi test
deactivate
```

## Create Conda Environment 

Not necessary I believe but script of this great website, they create an environment for conda environment for obitools and dada 2.
https://gitlab.mbb.univ-montp2.fr/edna

