# The Marine Organismal Body Size (MOBS) Database

The file *data\_all\_112224.csv* provides data in MOBS as of 11-22-24 with current taxonomic information and species names pulled from the World Register of Marine Species (WoRMS). You can downloadn this file, read it into your preferred software and begin analyses. This file has 181,532 rows and 14 columns. The columns are as follows:

|Column Name|Column Description|
|---|---|
|Order\_Added|order in which record was added to the database |
|valid\_aphiaID|AphiaID of the valid taxon name |
|phylum| phylum the species belongs to|
|class| class the species belongs to|
|order| order the species belongs to|
|family| family the species belongs to|
|genus| genus the species belongs to|
|scientificName| valid binomial species name|
|length\_cm| length of individual measured in centimeters|
|diameter\_width\_cm| diameter or width of individual measured in centimeters|
|height\_cm| height of individual measured in centimeters|
|Notes| notes on the measurement|
|Size\_Ref| reference for the sizes measurement(s)|
|Date\_Added| date record was added to database|

### Updating taxonomy from WoRMS 
Taxonomy is in constant flux and synonomies are continuusly being added to WoRMS. The most up to date version of MOBS is provided as six seperate data files.  
Each row of MOBS includes an AphiaID (linking it to the species name and taxonomy in WoRMS, [https://www.marinespecies.org/](https://www.marinespecies.org/)), 
length (cm), width/diameter (cm), height (cm), a note field to indicate other information about the measurement, and the reference for the size data. 

References are provided in both plain text (*Size\_Data\_Reference\_Plain.txt*) and RIS formats (*Size\_Data\_Reference\_RIS.txt*). The size seperate data files can be
combined by the following R code.

```r
library(readr) # load reqired package

# read in individual files
data_size_temp1 <- readr::read_csv("mobs.pt1.csv") 
data_size_temp2 <- readr::read_csv("mobs.pt2.csv") 
data_size_temp3 <- readr::read_csv("mobs.pt3.csv") 
data_size_temp4 <- readr::read_csv("mobs.pt4.csv") 
data_size_temp5 <- readr::read_csv("mobs.pt5.csv") 
data_size_temp6 <- readr::read_csv("mobs.pt6.csv") 

# combine individual files into single data frame
data_size_all <-  bind_rows(data_size_temp1, data_size_temp2, data_size_temp3, 
                        data_size_temp4, data_size_temp5, data_size_temp6) 
```

The package worrms (https://github.com/ropensci/worrms) can be utilized to match AphiaIDs to WoRMS to gain taxonomic and functional information. 

