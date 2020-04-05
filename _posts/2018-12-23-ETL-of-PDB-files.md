---
layout: post
title:  "ETL of PDB files"
author: "Sagiv Barhoom"
date:   2018-12-23
categories: Chemistry 
---
In computing, extract, transform, load (ETL) is the general procedure of copying data from one or more sources into a destination system.

## Extract PDB files

The main source of PDB files is the [RCSB PDB](http:/rscb.org) site.

We use several methods to download PDB files from this great data source:
* Directly from the site
* Using the "[pdb-tools/pdb_fetch.py](https://github.com/JoaoRodrigues/pdb-tools.git)" - an open source that project we found in github.

### Explore the data
As part of our [pdb_prep](http://sagivba.github.io/pdb_prep) program, we have implemented a program that called pdb_info.
This program gives us brief information about the PDB file which we downloaded.

### Extract&Explore Example
```
[sagiv@The_CSM_RG ETL-demo]$ /pdb_fetch.py 5a95 > 5a95.pdb
[sagiv@The_CSM_RG ETL-demo]$ ls -l 5a95.pdb
-rw-rw-r--. 1 sagiv sagiv 1627128 Dec 26 13:53 5a95.pdb

[sagiv@The_CSM_RG ETL-demo]$ pdb_info brief -i 5a95.pdb
+---------------------------------------------------
| 5a95.pdb
+---------------------------------------------------
file_name: 5a95.pdb
        number_fo_models: 1             number_fo_remarks: 17

        remark 2:               Resolution = 1.35        Resolution_Grade = EXCELLENT
        remark 3:               R_value = 0.129         R_free  = 0.156         R_free_grade = MUCH BETTER THAN AVERAGE at this resolution
        remark 350:             bio_struct_identical_to_the_asymmetric_unit = True

        models_info:
                model_number: 1         number_of_chains: 3

            model No   chain id   residues      atoms
                   1          A        337       2819
                   1          B        335       2800
                   1          C        336       2819

```

##  Transform  PDB files
We are processing the PDB files by bisecting the protein into subunits.
The program implements the following methods of subunits bisecting:
Amino acid - includes all the atoms of a given residue
Ramachandran subunit –for amino acid #i includes the atoms: C(i-1)-N(i)-CA(i)-C(i)-N(i+1)
User defined subunit - with an external atoms.txt file (see format below)
Amino acid backbone – includes the atoms N-CA-C-O
Ramachandran with side chain - for amino acid #i includes the atoms: C(i-1)-N(i)-CA(i)-C(i)-N(i+1), the backbone atoms and all the atoms of the side chain
Ramachandran with side chain and without the backbone oxygen – similar to subunit 5, without the backbone oxygen
We provide the [pdbslicer](https://github.com/continuous-symmetry/pdbslicer) code as an open source.

### An example of pdbslicer usage 
```
[sagiv@The_CSM_RG ETL-demo]$ pdbslicer -i 5a95.pdb -C 1 -J
+------------------------------------------------------------------
| pdbslicer
| Version     : 0.9.5.3
| input_dir   : .
| output_dir  : ./out_5a95
+------------------------------------------------------------------
Wed Dec 26 13:59:54 IST 2018
parsing file progress: 100.00%
finish parsing ./5a95.pdb
Wed Dec 26 14:00:05 IST 2018
setting model range...finish setting model range


- - - - - - - - - - - - - - - - - - -
 output_dir
./out_5a95
-rw-rw-r--. 1 sagiv sagiv 70727 Dec 26 14:00 ./out_5a95/001.m001.A-m001.A.pdb
-rw-rw-r--. 1 sagiv sagiv 71438 Dec 26 14:00 ./out_5a95/002.m001.A-m001.A.pdb
-rw-rw-r--. 1 sagiv sagiv 69542 Dec 26 14:00 ./out_5a95/003.m001.A-m001.A.pdb
-rw-rw-r--. 1 sagiv sagiv 71991 Dec 26 14:00 ./out_5a95/004.m001.A-m001.B.pdb
-rw-rw-r--. 1 sagiv sagiv 72228 Dec 26 14:00 ./out_5a95/005.m001.B-m001.B.pdb
-rw-rw-r--. 1 sagiv sagiv 69542 Dec 26 14:00 ./out_5a95/006.m001.B-m001.B.pdb
-rw-rw-r--. 1 sagiv sagiv 70016 Dec 26 14:00 ./out_5a95/007.m001.B-m001.C.pdb
-rw-rw-r--. 1 sagiv sagiv 70648 Dec 26 14:00 ./out_5a95/008.m001.C-m001.C.pdb
-rw-rw-r--. 1 sagiv sagiv 71122 Dec 26 14:00 ./out_5a95/009.m001.C-m001.C.pdb
-rw-rw-r--. 1 sagiv sagiv 69937 Dec 26 14:00 ./out_5a95/010.m001.C-m001.C.pdb
-rw-rw-r--. 1 sagiv sagiv  1518 Dec 26 14:00 ./out_5a95/011.m001.C-m001.C.pdb
-rw-rw-r--. 1 sagiv sagiv    76 Dec 26 13:59 ./out_5a95/cmd_string.txt
procesesd 1 molecules.

```
##  Load the PDB file to our [web tools site](http://csm.ouproj.org.il) 
Our group has opened a website [http://csm.ouproj.org.il](http://csm.ouproj.org.il).
This site allows measuring the degree of symmetry, of chirality, and of polyhedral shapes.
