# Apscale installation

## First install VSEARCH

```
cd ~/work/programs/
```

Download Vsearch:
```
wget https://github.com/torognes/vsearch/archive/v2.22.1.tar.gz
tar xzf v2.22.1.tar.gz
cd vsearch-2.22.1
./autogen.sh
./configure CFLAGS="-O3" CXXFLAGS="-O3"
make
make install  # as root or sudo make install
```

Check if the VSEARCH works:
```
vsearch --version
```

## Install APSALE's command-based version 

From: 
https://github.com/DominikBuchner/apscale

```
pip install apscale
pip install --upgrade apscale
```
Check for a proper installation:
```
apscale --help
cutadapt --version
```



## Install APSCALE's graphical interface version

APSCALE's GUI version allows more analyses, including raw data summarizes, demultiplexing and post-MOTU/ESV table analyses. It can also perform local alignments, NCBI Blast alignments and integrates BOLDigger, a

From: https://github.com/DominikBuchner/demultiplexer

```
pip install apscale_gui
pip install --upgrade apscale_gui
```
Similarly to the command-based version, VSEARCH needs to be installed.

Run the interface with:
```
python -m apscale_gui
```
or 
```
apscale_gui
```

## Install APSCALE's Demultiplexer

From: https://github.com/DominikBuchner/demultiplexer

```
pip install demultiplexer
pip install --upgrade demultiplexer
```
Open the demultiplexer from ```apscale_gui```.

*NOTES: I'm having several issues with Demultiplexer. First, when installed and opened from the APSCALE GUI, after loading files and entering tag's information, the program crashes, indicating that Demultiplexer needs to be installed first. Then, the "tagging scheme" parameters are not really built for forward and reverse read conbinatorial tagged individuals. With only two (R1 and R2) FASTQ files and a large combinaison of tags, the tagging scheme file will be two rows and up to 400 columns in our cases. Not very convinient. Three colums files with a sample list and forward and reverse tags would be preferable. I also found the tutorial on the Github page quite unintuitive, so maybe I'm missing something.*

## Further modules

Apscale's outputs (ESV and MOTU tables) can be used for taxonomic assignment with BOLDigger
https://github.com/DominikBuchner/BOLDigger and can be used for quick summary statics with TaxonTableTools downloadable at https://github.com/TillMacher/TaxonTableTools.

*NOTES: I am not sure if I use these yet, if yes, install them and give them a try.*