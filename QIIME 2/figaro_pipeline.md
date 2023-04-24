# FIGARO tutorial

The following code is to 

NOTES: the article mentions trimming reads while the tutorial talks about truncation, which are not exactly the same parameters. Check why.

https://www.biorxiv.org/content/10.1101/610394v1  
https://github.com/Zymo-Research/figaro  
http://john-quensen.com/tutorials/figaro 

*NOTES: the article mentions trimming reads while the tutorial talks about truncation, which are not exactly the same parameters. Check why.*

## Install FIGARO 


```
git clone https://github.com/Zymo-Research/figaro.git
cd figaro
python3 setup.py bdist_wheel
```

## Run FIGARO 


### Decompress.
```
 gzip -dk *.fastq.gz
```
*NOTES: while our files are .gz, they are not compressed so no need to run the command above*

### Rename the files in Zymo format

```
mkdir renamed_figaro
cp renamed_figaro
mv forward.fastq sam1_16s_R1.fastq
mv reverse.fastq sam1_16s_R2.fastq
```
*NOTES: This names are arbitrary, but FIGARO doesn't recognize the original format, so this is supposed to be a Zymo format, an Illumina format can be used too.*

Now the parameters: 
- amplicon length of 16S bacteria V5-V6 is 295bp
- F primer is 20bp
- R primer is 17bp
- We changed the file names to the Zymo format  

*NOTES: I used the default Illumina format and it worked too somehow.*

```
cd ~/work/program/figaro/figaro
python3 figaro.py \
 -i /home/ECOFOG/finn.piatscheck/work/data/16S_bacteria/subsamples/renamed_figaro/ \
 -o /home/ECOFOG/finn.piatscheck/work/analysis/qiime2-pipeline_validation \
 -a 295 \q
 -f 20 \
 -r 17 \
 - F Zymo
 ```

The outputs are a log file, two expected/observed error plots (png) and a trimParameter.json file.

The results in trimParameters.json shows a list of "trimPosition" values with an "readRetentionPercent" metric, and a score. The truncation parameters to use are the ones given by the best score, normally at the top of the list. Explore the file with the command ```less```.
