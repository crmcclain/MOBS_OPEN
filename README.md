![MOBS Logo](https://github.com/user-attachments/assets/07e838d1-ab64-4180-9670-41db95380241)


# The Marine Organismal Body Size (MOBS) Database

The file *data\_all\_112224.csv* provides data in MOBS as of 11-22-24 (MOBS 1.0) with current taxonomic information and species names pulled from the World Register of Marine Species (WoRMS; [https://www.marinespecies.org/](https://www.marinespecies.org/)). You can download this file, read it into your preferred software and begin analyses. This file has 181,531 rows (size measuremments) and 14 columns for 85,204 unique species (AphiaIDs).

**NOTE:** The file *data\_all\_112224.csv* will only contain size data as of 11-22-24. The separate data files named *mobs.pt1.csv*, *mobs.pt2.csv*, etc. will contain the most up-to-date size data and with additonal species and sizes (MOBS 1.x).  If you would prefer to use the most recent data follow the section below titled Updating taxonomy from WoRMS.

### Citation Requirement
If you use the MOBS database in any form—whether in publications, presentations, or derivative datasets—you are required to cite the following reference:

McClain, C. R., Heim, N. A., Knope, M. L., Monarrez, P. M., Payne, J. L., Santos, I. T., & Webb, T. J. (2025). MOBS 1.0: A database of interspecific variation in marine organismal body sizes. *Global Ecology and Biogeography*, 34: e70062. https://doi.org/10.1111/geb.70062 

Proper citation ensures credit to the creators and supports the continued development and maintenance of the database.

### Papers Utilizing MOBS
McClain, C.R., Webb, T.J., Heim, N.A., Knope, M.L., Monarrez, P.M. and Payne, J.L. (2025), Size bias in the documentation of marine biodiversity. Oikos, 2025: e10828. https://doi.org/10.1111/oik.10828

McClain, C. R., Webb, T. J., Heim, N. A., Knope, M. L., Monarrez, P. M., & Payne, J. L. (2024). Navigating uncertainty in maximum body size in marine metazoans. Ecology and Evolution, 14, e11506. https://doi.org/10.1002/ece3.11506

### Press on MOBS

Kaplan, S. (2025, June 22). *A massive new database captures the true scale of life in the ocean*. The Washington Post. https://www.washingtonpost.com/science/2025/06/22/ocean-life-database-animal-size/

Nipponese News. (2025, June 24). *海洋生命体のサイズデータベース「海洋生物多様性の鍵」、国際研究チームが公開* [Marine life size database, “Key to Marine Biodiversity,” released by international research team].

Al Jazeera. (2025, June 15). *قاعدة بيانات جديدة لحجم جسم 85 ألف نوع من الكائنات البحرية* [New database reveals body size of 85,000 marine species]. https://www.aljazeera.net/climate/2025/6/

The Spokesman-Review. (2025, June 22). *New marine life database touted as tool for ocean conservation*. https://www.spokesman.com/stories/2025/jun/22/new-marine-life-database-touted-as-tool-for-ocean-/

Ouellette, J. (2025, June 22). *New body size database for marine animals is a “library of life”*. Ars Technica. https://arstechnica.com/science/2025/06/new-body-size-database-for-marine-animals-is-a-library-of-life/

ECO Magazine. (2025, June 24). *New body size database holds the key to marine life protections*. https://ecomagazine.com/news/research/new-body-size-database-holds-the-key-to-marine-life-protections/

University of Louisiana at Lafayette. (2025, June 24). *Scientists launch largest database of marine animal sizes* [Press release]. EurekAlert!. https://www.eurekalert.org/news-releases/1084479

Indian Defence Review. (2025, June 24). *Scientists document body sizes of 85,000 marine species*. https://indiandefencereview.com/scientists-body-sizes-85000-marine-species/


### Dataset Details
The columns are as follows:

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
|Biological\_Unit| whether the measurement is for a zooid, polyp, colony, or solitary|

References are provided in both plain text (*Size\_Data\_Reference\_Plain.txt*) and RIS formats (*Size\_Data\_Reference\_RIS.txt*).

Most species are represented by a single row, however, some have more than one row, thus more than one size measurement. 

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

# read in data files and combine into a single data frame
mobs_files <- list(
	"https://github.com/crmcclain/MOBS_OPEN/raw/refs/heads/main/mobs.pt1.csv",
	"https://github.com/crmcclain/MOBS_OPEN/raw/refs/heads/main/mobs.pt2.csv",
	"https://github.com/crmcclain/MOBS_OPEN/raw/refs/heads/main/mobs.pt3.csv",
	"https://github.com/crmcclain/MOBS_OPEN/raw/refs/heads/main/mobs.pt4.csv",
	"https://github.com/crmcclain/MOBS_OPEN/raw/refs/heads/main/mobs.pt5.csv",
	"https://github.com/crmcclain/MOBS_OPEN/raw/refs/heads/main/mobs.pt6.csv"
) # the file locations on GitHub

data_size_all <- do.call("rbind", lapply(mobs_files, read.csv, strip.white=TRUE))

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



