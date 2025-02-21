# The Marine Organismal Body Size (MOBS) Database

The file *data\_all\_112224.csv* provides data in MOBS as of 11-22-24 with current taxonomic information and species names pulled from the World Register of Marine Species (WoRMS; [https://www.marinespecies.org/](https://www.marinespecies.org/)). You can downloadn this file, read it into your preferred software and begin analyses. This file has 181,531 rows and 14 columns. The columns are as follows:

|Column Name|Column Description|
|---|---|
|Order\_Added|order in which record was added to the database |
|valid\_aphiaID|AphiaID of the valid species name linking it to the taxonomy in WoRMS|
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

References are provided in both plain text (*Size\_Data\_Reference\_Plain.txt*) and RIS formats (*Size\_Data\_Reference\_RIS.txt*).

The current database contains 86,504 unique species (AphiaIDs). Most species are represented by a single row, however, some have more than one row, thus more than one size measurement. 

### Updating taxonomy from WoRMS 
Taxonomy is in constant flux and synonomies are continuously being added to WoRMS. If you would like to pull the most recent taxonomy from WoRMS, you may do so using the six files named *mobs.pt1.csv*, *mobs.pt2.csv*, etc. These files have the same columns as *data\_all\_112224.csv*, except they do not have those related taxonomy (e.g., scientificName, phylum).

The six seperate data files can be combined. Then package worrms [https://github.com/ropensci/worrms](https://github.com/ropensci/worrms) can be utilized to match AphiaIDs to WoRMS to gain taxonomic and functional information. Example R code is given below.

```r
# load reqired packages--these will need to be installed using install.packages() before loading the libraries
library(dplyr) # a tidyverse package for manipulating data frames
library(readr) # a tidyverse package for reading files
library(worrms) # package for interacting with WoRMS
library(reshape2) # package for transforming data frames

# read in individual files
data_size_temp1 <- readr::read_csv("mobs.pt1.csv") 
data_size_temp2 <- readr::read_csv("mobs.pt2.csv") 
data_size_temp3 <- readr::read_csv("mobs.pt3.csv") 
data_size_temp4 <- readr::read_csv("mobs.pt4.csv") 
data_size_temp5 <- readr::read_csv("mobs.pt5.csv") 
data_size_temp6 <- readr::read_csv("mobs.pt6.csv") 

# combine individual files into single data frame
data_size_all <-  bind_rows(data_size_temp1, data_size_temp2, data_size_temp3, data_size_temp4, data_size_temp5, data_size_temp6) 

# get unique AphiaIDs
aphia <- unique(data_size_all$valid_AphiaID)

# pull classification for all spcies--this is a slow process with >85K AphiaIDs. Just for convenience and to try and prevent timeout errors, we will use a loop to get the classificaiotn for groups of 500 species at a time. Each 500 species chunk will take longer than 1.5 minutes each. The total routine can take longer than 5 hours to run.

chunks <- seq(1,length(aphia), 500) # the starting index for the chunks
n.chunks <- length(chunks)
chunks <- c(chunks, length(aphia)) # adding the total number ids
# here we go with a simple for loop
taxonomy <- data.frame() # initalize the variable
for(i in 1:n.chunks) {
	these.taxa <- wm_classification_(id = aphia[chunks[i]:(chunks[i+1]-1)])
	taxonomy <- rbind(taxonomy, these.taxa)
}

# drop the non-canonical ranks
taxonomy <- subset(taxonomy, is.element(rank, c('Phylum','Class','Order','Family','Genus','Species')))

# convert worrms output to a dataframe with columns of AphiaID and each taxonomic rank
taxonomy.df <- dcast(taxonomy, id~rank, value.var="scientificname")
colnames(taxonomy.df)[1] <- "valid_AphiaID" # change column name to match data_size_all

# add taxonomy to original data frame
data_size_all <- merge(data_size_all, taxonomy.df, by="valid_AphiaID", all= T)

# finally, make column names match those of the combined data sets
colnames(data_size_all)[colnames(data_size_all) == "Species"] <- "scientificName"


```



