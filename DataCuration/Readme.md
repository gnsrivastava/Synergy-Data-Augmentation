# DataCuration

## Pre-requisites
* Python3
* Pandas *(v1.1.3)*
* IHP-PING PPI *(https://github.com/gkm-software-dev/post-analysis-tools.git)*
* openbabel *(v2.3.1, https://sourceforge.net/projects/openbabel/)*

## Step 1: Convert ENSP-ID in the chemical-protein links file to UniProt
<p>
Chemical-Protein Links file represents interaction of drugs with proteins. The score in the file represents how well a drug interacts with proteins.<br>
idmapping.dat file contains ID conversions between UniProt ID and other ID types<br>
convert_to_uniprot.py uses idmapping.dat file to convert ENSEMBL ID to UniProt ID </br></br>
</p>

* Download chemical-protein links (9606.protein_chemical.links.v5.0.1.tsv) file from STITCH dataset from http://stitch.embl.de
* seprate **STRING** and **Ensembl_PRO** from the total datasets
* ID mapping file (idmapping.dat) can be downloaded from https://ftp.uniprot.org/pub/databases/uniprot
```
grep STRING idmapping.dat > string.data
grep Ensembl_PRO idmapping.dat > ensp.data
cat string.data ensp.data > idmapping.data
python convert_to_uniprot.py
```

## Step 2: Extract all the targets for CIDs in chemical-protein links if target in IHP-PING
This step requires users to have IHP-PING PPI in a csv file downloaded in the same folder as the `make_nodes_dict.py`<br>
`python make_nodes_dict.py`

## Step 3: Similarity search of CIDs from AZ-DREAM challenge against STITCH
Get all SMILES files from AZ-DREAM synergy data

1. Tanimoto Similarity Search (TSS)
codes mention below can be either copied into a python script or run on terminal
The script below uses `SMILES` from stitch.smi
```
import pandas as pd
import os, sys
cids = [l for l in os.listdir() if "CIDs" in l]
for cid in cids:
  os.system("openbabel-2.3.1/bin/babel "+cid+" stitch.smi -ofpt > "+cid+".out")
 ```
 2. Converting TSS outputs to CSV

```
out_list = [l for l in os.listdir() if "CIDs" in l]
for out in out_list:
  data = pd.read_csv(out, sep=" ")
  index = [l[0].split(" ")[0].replace(">", "") for l in list(data.index)]
  data.index = index
  data.reset_index(inplace=True)
  data.columns = ["SIMILAR_CID", "TC"]
  data = data[data.SIMILAR_CID.str.contains("CIDs")]
  data.to_csv(out+".csv", index=False)
 ```
## Step 4: Finding drug action and chemical similarity score (DACS) for original drug in a pair
#### Requirements 
* multiprocessing, math, pickle python modules
* target dictionary generated at **step 2**
* IHP-PING PPI CSV file
* __Input__: CSV file generated at **step 3**

Replace "filename" with the CSV input file

`python dacs.py`

## Step 5: Concat all dacs data into one file
Get header from the first dacs file and save it in a `header.txt`

```
for f in dacs_*; do sed -i 1d $f; done;
```
Concat all the dacs files into a larger file

```
cat path_to_dacs_files/dacs_* > dacs_score_between_original_drug_and_similar_drugs.csv
```
## Step 6: get augemnted data
Use the `augmentation.py` to get augmented data
```
python augmented.py
```


