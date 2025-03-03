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

**NOTE:** this code will take a long time to run. I have set it up to run in chunks to keep the WoRMS data service from returning a run time error. Depending on the speed of your computer and the traffic on WoRMS, this code will take 5 to 9 hours to run. You could set this up to run in prarallel, though I doubt it will save much time since the major bottle neck is WoRMS. Though it may be worth a try.

```r
# check to see if requried packages are installed, if not install them
list.of.packages <- c("worrms", "reshape2")
new.packages <- list.of.packages[!(list.of.packages %in% installed.packages()[,"Package"])]
if(length(new.packages)) install.packages(new.packages, repos='https://repo.miserver.it.umich.edu/cran/') 

# load reqired packages--these will need to be installed using install.packages() before loading the libraries
library(worrms) # package for interacting with WoRMS
library(reshape2) # package for transforming data frames

# read in individual files
data_size_temp1 <- read.csv("mobs.pt1.csv") 
data_size_temp2 <- read.csv("mobs.pt2.csv") 
data_size_temp3 <- read.csv("mobs.pt3.csv") 
data_size_temp4 <- read.csv("mobs.pt4.csv") 
data_size_temp5 <- read.csv("mobs.pt5.csv") 
data_size_temp6 <- read.csv("mobs.pt6.csv") 

# combine individual files into single data frame
data_size_all <-  rbind(data_size_temp1, data_size_temp2, data_size_temp3, data_size_temp4, data_size_temp5, data_size_temp6) 

# get unique AphiaIDs
aphia <- unique(data_size_all$valid_AphiaID)

# pull classification for all spcies--this is a slow process with >85K AphiaIDs. Just for convenience and to try and prevent timeout errors, we will use a loop to get the classificaiotn for groups of 500 species at a time. Each 500 species chunk will take longer than 1.5 minutes each. The total routine can take longer than 5 hours to run.

chunks <- seq(1,length(aphia), 50) # the starting index for the chunks
n.chunks <- length(chunks) # number of chunks to iterate over
chunks <- c(chunks, length(aphia)) # adding the total number of ids
# here we go with a simple for loop--alternatively you could set this code up to run in parallel
taxonomy <- data.frame() # initialize the variable
t0 <- Sys.time(); print(t0) # lets keep track of time
for(i in 1:n.chunks) {
	these.taxa <- wm_classification_(id = aphia[chunks[i]:(chunks[i+1]-1)])
	# drop the non-canonical ranks
	these.taxa <- subset(these.taxa, is.element(rank, c('Kingdom','Phylum','Class','Order','Family','Genus','Species')))
	
	# bind new dataframe to existing--add the new rows
	taxonomy <- rbind(taxonomy, these.taxa)
	if(i %% 50 == 0) {print(chunks[i+1])}
}
t1 <- Sys.time(); print(t1); print(t1 - t0) # how long did it take? [about 5 hours]

# convert worrms output to a dataframe with columns of AphiaID and each taxonomic rank
taxonomy.df <- dcast(taxonomy, id~rank, value.var="scientificname")
colnames(taxonomy.df)[1] <- "valid_AphiaID" # change column name to match data_size_all

# add taxonomy to original data frame
data_size_all <- merge(data_size_all, taxonomy.df, by="valid_AphiaID", all= T)

# finally, make column names match those of the combined data sets
colnames(data_size_all)[colnames(data_size_all) == "Species"] <- "scientificName"

# save the data frame as an R Object for later use
save(data_size_all, file="MOBS.RData")
```



