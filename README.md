# The Marine Organismal Body Size (MOBS) Database

The file data_all_112224.csv provides data in MOBS as of 11-22-24 with current taxonomic information and species names pulled from WoRMS 

The most up to date version of MOBS is provided as six seperate data files.  
Each row of MOBS includes an AphiaID (linking it to the species name and taxonomy in WoRMS, https://www.marinespecies.org/index.php), 
length (cm), width/diameter (cm), height (cm), a note field to indicate other information about the measurement, and the reference for the size data. 
References are provided in both plain text (Size_Data_Reference_Plain.txt) and RIS formats (Size_Data_Reference_RIS.txt).  The size seperate data files can be
combined by the following R code.  

```{r, eval=FALSE}
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

