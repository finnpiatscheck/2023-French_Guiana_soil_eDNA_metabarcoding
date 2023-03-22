# FIGARO tutorial

The following code is to 

NOTES: the article mentions trimming reads while the tutorial talks about truncation, which are not exactly the same parameters. Check why.

https://www.biorxiv.org/content/10.1101/610394v1  
http://john-quensen.com/tutorials/figaro/ 

*NOTES: the article mentions trimming reads while the tutorial talks about truncation, which are not exactly the same parameters. Check why.*

## Install FIGARO

### Create FIGARO environment.

```
wget http://john-quensen.com/wp-content/uploads/2020/03/figaro.yml
conda env create -n figaro -f figaro.yml
```
### Download and install FIGARO.
```
wget https://github.com/Zymo-Research/figaro/archive/master.zip
unzip master.zip
rm master.zip
cd figaro-master/figaro
chmod 755 *.py
```

*NOTES: if there is an error message, check the tutorial listed above to fix it.*

## Run FIGARO



### Decompress.
```
 gzip -d *.fastq.gz
```
### Rename the files in Zymo format
```
 mv forward.fastq sam1_16s_R1.fastq
 mv reverse.fastq sam1_16s_R2.fastq
 ```
 
### Run FIGARO.
```
 cd to installation folder
 cd ~/figaro-master/figaro
 python figaro.py -i ~/destination -o ~/destination/ -f 10 -r 10 -a 253 -F zymo
```

Several files are written to the output directory:

- trimParameters.json
- forwardExpectedError.png
- reverseExpectedError.png

To view the recommended truncation parameters:

```
less trimParametersjson
```