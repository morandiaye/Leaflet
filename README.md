# Leaflet
Utlisons les donnees issus  de la base de données SOTER pour visualiser les echantillons preleves au niveau du Senegal et representons en arriere plan les departement  du Senegal.
Ainsi nous pouvons voire la repartition spatiale  des echantillons de sol en fonction des differents departements.

Appelons les libraries necessaires pour la realisations des differents qui seront effectueé au cours de ce travail
`
```{r , results="hide"}
library(rgdal)
library(sp)
library(raster)
library(leaflet)

```

# Importons les donnees necessaires

```{r,cache=TRUE}
data_sol<-read.csv2("C:\\Users\\pc\\Downloads\\Modelisation_Soter_data_Senegal-master\\Modelisation_Soter_data_Senegal-master\\donnees soter.csv")

```

Regardons quelques ligne sur les donnees importees

```{r}
head(data_sol,3)
```
# Creation Dataset 

Assignons un nouveau dataset apres elimination des NA sur les variable LONG et LAT
```{r}
sol<-data_sol[!is.na(data_sol$LNGI),]
```

Verfions effectivement les NA sont extraite dans LAT et LONG
```{r}
sum(is.na(sol$LNGI))

sum(is.na(sol$LATI))
```

# Transformation en spatial dataframe 
le data  set en spatial dataframe
```{r}
coordinates(sol)<-~LNGI+LATI
```

# Verifions la transformation operé
```{r}
str(sol)
```

Etablissons le systeme de projection du nouveau spatial dataframe concu
```{r}
proj4string(sol)<-crs("+init=epsg:4326")
str(sol)
```


# Raster (Miscellous)
```{r}
dem<-raster("C:\\Users\\pc\\Desktop\\DEM\\Elevation.tif")

```

jetons un coup d'oeil sur la nature du raster et affichons la carte
```{r}
print(dem)
plot(dem)
```

# Application avec usage du package leaflet

la creation d'une carte interactive avec les données issues de soter 
D'abord creons des  pop up pour les points echantillonnés et la carte du senegal avec les 45 departement en arriere plan.
 - Importons le fichier shapefile
```{r}
library(sp)
```
 
```{r eval=TRUE}
dep<-readOGR("D:\\carto\\limite 2014 ANSD\\LIMITE_DEPT_2014_.shp")
```
 - Corrigeons le systeme de projection
```{r}
departement<-spTransform(dep,crs("+init=epsg:4326"))
```
 Creation des pop up  pour le shapefile des departements du senegal et les echantions
```{r}
my_pops <- paste0("<strong>ID: </strong>", na.omit(sol$PRID),"<br>\n <strong> Organic Carbon (%): </strong>",round(na.omit(sol$total_carbone),3), "<br>\n <strong>soilpH:</strong>",round(na.omit(sol$PH),2),"<br>\n <strong> CEC (meq/100g): </strong>",round(na.omit(sol$CECS),2),"<br>\n <strong>Nitogen(%): </strong>",round(na.omit(sol$total_azote),2));
pops <- paste0("<strong>Departement: </strong>", departement$DEPT)

```



# Carte interactive portant sur les departement du Senegal et les echantillons de sols prelevées par SOTER 
```{r}
leaflet(departement)%>%addPolygons(layerId = departement$DEPT,weight = 2,fill = TRUE,fillOpacity = 0,popup =pops,group = "polygons",fillColor = "red")%>%
  addTiles(group = "OSM (default)") %>%
  addProviderTiles("Esri.WorldImagery") %>%
  addMarkers(data = sol,  popup = my_pops,group = "points")%>%
  addLayersControl(baseGroups = c("OSM (default)",  "Imagery"),overlayGroups = c("points", "polygons"))
```


               
