# Ryan Mitchell's R Portfolio
[Home](index.md)    [Python Portfolio](python.md)   [R Portfolio](R.md)   [Resume](resume.md)

Below is a snippet of code used to generate how it would look like to cover %20 of Queensland in nature reserves based on data relating to environmental factors

```R
  pack.to.install <-
  c(
    "rgdal",
    "rgeos",
    "raster",
    "sf",
    "classInt",
    "velox",
    "cleangeo",
    "maptools"
  ) #defines the packages in a list

install.packages(pack.to.install) # install the packages (to do once only)
install.packages("zoom") # install the packages (to do once only)

#runs the libraries 
library(rgdal)
library(rgeos)
library(raster)
library(sf)
library(classInt)
library(velox)
library(cleangeo)
library(zoom)

#reads the shapefiles and defines them so they can be read easily in the future
QLD_PUs <- shapefile("Queensland_PUs") 
QLD_Amphibia <- shapefile("QLD_Amphibia")
QLD_Mammals <- shapefile("QLD_Mammals")
QLD_Reptilia <- shapefile("QLD_Reptilia")
QLD_Birds <- shapefile("QLD_Birds")

#calculate the area & area^2 of each planning unit
QLD_PUs$area<-area(QLD_PUs)
QLD_PUs$area_km<-QLD_PUs$area/1000000

#summarising all of the critters by PU using an intersect
all_critters_input <- rbind(QLD_Amphibia, QLD_Mammals, QLD_Reptilia, QLD_Birds)
#section off only the binomial coloumn
all_critters <- all_critters<-all_critters_input[,(3)]

#creating an SFpoly so the intersect is easier to achieve 
QLD_PUs_SFPoly <- st_as_sf(QLD_PUs)
all_critters_SFPoly <- st_as_sf(all_critters)

#creating an intersect of both the all_critters and of the QLD_PUs
all_critters_vs_PUs <-st_intersects(all_critters_SFPoly, QLD_PUs_SFPoly)
all_critters_and_PUs_table<-data.frame(all_critters_vs_PUs)

#reattaching the column names
colnames(all_critters_and_PUs_table)<-(c("species","GridID"))

#creating a species list
species_list<- data.frame(all_critters)
species_list$species <- row(species_list)

# creating a final critters file merged with the species list
final_all_critters_merge <- merge(all_critters_and_PUs_table,species_list, by="species")

#creating the foundation for the final_env_data
env_class_files<-subset(QLD_PUs,select=c("GridID","area_km","tmp_dc_syr",
                                                         "pre_mm_syr", "swc_pc_syr", "gwt_mt_sav"))

#adding temperature columns to the Env_class_files
env_class_files$tmp_dc_syr_class_num <-  findCols(classIntervals(env_class_files$tmp_dc_syr,n=6, style="kmeans"))
env_class_files$tmp_dc_syr_class <- paste("temp",env_class_files$tmp_dc_syr_class_num, sep="_")

#adding precipitation per mm to the Env_class_files
env_class_files$pre_mm_syr_class_num <- findCols(classIntervals(env_class_files$pre_mm_syr,n=6, style="kmeans"))
env_class_files$pre_mm_syr_class <- paste("precipitation",env_class_files$pre_mm_syr_class_num, sep="_")

#adding swc to the _env_class_files
env_class_files$swc_pc_syr_class_num <- findCols(classIntervals(env_class_files$swc_pc_syr,n=6, style="kmeans"))
env_class_files$swc_pc_syr_class <- paste("swc",env_class_files$swc_pc_syr_class_num, sep="_")

#adding gwt to the env_class_files
env_class_files$gwt_mt_sav_class_num <- findCols(classIntervals(env_class_files$gwt_mt_sav,n=6, style="kmeans"))
env_class_files$gwt_mt_sav_class <- paste("gwt",env_class_files$gwt_mt_sav_class_num, sep="_")

#defining the built env_class_files as a data frame as well as creating the variable final_env_data
final_env_data <-data.frame(env_class_files)

#creating the PUVSPR file
temp_by_PU<-final_env_data[c("GridID", "area_km", "tmp_dc_syr_class")]
names(temp_by_PU)[3]<-"species"   #Rename the altitude_class column to "species" - that's what Marxan needs

prec_by_PU<-final_env_data[c("GridID", "area_km", "pre_mm_syr_class")]
names(prec_by_PU)[3]<-"species"   #Rename the altitude_class column to "species" - that's what Marxan needs

swc_by_PU<-final_env_data[c("GridID", "area_km", "swc_pc_syr_class")]
names(swc_by_PU)[3]<-"species"   #Rename the altitude_class column to "species" - that's what Marxan needs

gwt_by_PU<-final_env_data[c("GridID", "area_km", "gwt_mt_sav_class")]
names(gwt_by_PU)[3]<-"species"   #Rename the altitude_class column to "species" - that's what Marxan needs



#binding the above files together
all_env_relational<- rbind(temp_by_PU, prec_by_PU,swc_by_PU,gwt_by_PU)

#creating the specfile
specfile_env<-as.data.frame(unique(all_env_relational$species))
colnames(specfile_env)[1]<-"name"
specfile_env$id<- 1:nrow(specfile_env)


#inputting the area into the final_all_critters_merge
pu_area<-as.data.frame(QLD_PUs@data)[c("GridID", "area_km")]

all_critters_relational<-merge(final_all_critters_merge,pu_area, by="GridID")

#clearing the species column repeat
all_critters_relational$species<-NULL
#all_critters_relational<-all_critters_relational[,-(2)]
colnames(all_critters_relational)[2]<-"species"

#working on the specfile more
specfile_critters<-species_list
specfile_critters$id<- (1:nrow(specfile_critters)) + 2000 
specfile_critters$species<-NULL

#changing column name in critters specfile to name
colnames(specfile_critters)[1]<- "name"




#creating the final files
puvspr_all<-rbind(all_env_relational, all_critters_relational)

spec_all<-rbind(specfile_env, specfile_critters)

puvspr_incl_codes <- merge(puvspr_all, spec_all, by.y="name", by.x="species")

puvspr_incl_codes[1]<-NULL
final_puvspr<-puvspr_incl_codes[c(3,1,2)]  
colnames(final_puvspr)<-c("species","pu","amount")
final_puvspr_sorted<-final_puvspr[order(final_puvspr$pu,final_puvspr$species),]

#reordering the spec_all and creating a Final_spec file
spec_all$target<-50100    #Stick with 10 sqkm for now
spec_all$prop<-0  
spec_all$spf<-100000 #This should always be high if you want to  preserve everything
final_spec<-spec_all[c("id","prop","target","spf","name")]   #correct_order
write.csv(final_spec,"F:/3418ENV Spatial Modelling/Assignment/Assigment 1 for students(1)/Assigment 1 for students/MARXAN/input/spec_QLD.csv", row.names=FALSE)


#writing the PUVSPR and spec csv files
dir.create("input")
write.csv(final_puvspr_sorted,"F:/3418ENV Spatial Modelling/Assignment/Assigment 1 for students(1)/Assigment 1 for students/MARXAN/input/puvspr_QLD.csv", row.names=FALSE)

#preparing the planning unit to write into a csv
pu_file<-pu_area
pu_file$status<-0
colnames(pu_file)<-c("id","cost","status")

#writing the planning unit csv file
write.csv(pu_file,"F:/3418ENV Spatial Modelling/Assignment/Assigment 1 for students(1)/Assigment 1 for students/MARXAN/input/pu_QLD.csv", row.names=FALSE)

#preparing the connectivity file
borders <- st_intersects(QLD_PUs_SFPoly,QLD_PUs_SFPoly)
connectivity <-data.frame(borders)
colnames(connectivity)<-(c("id1","id2"))
connectivity$boundary<-1

#writing the connectivity csv
write.csv(connectivity,"F:/3418ENV Spatial Modelling/Assignment/Assigment 1 for students(1)/Assigment 1 for students/MARXAN/input/connectivity_QLD.csv", row.names=FALSE)

pu_QLD.csv
puvspr_QLD.csv
spec_QLD.csv
```

Below is the map of reserves that it generated
![Map of reserves](.png)
