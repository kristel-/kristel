+++
title = "Kaartide töötlus R-ga [Manipulating Maps with R]"
date = '2017-10-24'
author = "Kristel Uiboaed"
tags = [
    "kaardiandmed",
	"Shapefile",
	"geograafilised andmed",
	"maps",
	"geospatial data",
	"geospatial analysis",
	"R"]
+++

# Sissejuhatus

Analüüsides andmeid, mis on kuidagi seotud geograafiaga, tuleb üsna sageli ette, et andmete esitamiseks puudub sobiv kaart. Näiteks soovides kaardile kanda mingite väiksemate keelte uurimisandmeid, tuleb seda ette sageli. Keeltepiirkonnad ei pea kokku langema mõne riigi või riikide haldusterritoriaalse jaotusega, mille kaarti üldjuhul on üsna kerge leida. Järgnevalt näitan, kuidas oleks võimalik mõnda olemasolevat kaarti muutes tekitada uus töödeldav kaart, mida on hiljem võimalik andmete estiamiseks kasutada ja töödelda. Kasutan selleks programmi [R](https://www.r-project.org/) ning näiteandmetena [Eesti haldusjaotuse andmeid](http://geoportaal.maaamet.ee/est/Andmed-ja-kaardid/Haldus-ja-asustusjaotus-p119.html) ning minu eesmärgiks on tekitada uute omavalitsuste kaart, mis tekkis peale haldusreformi. See ülesanne tekkis praktilisest vajadusest, kui soovisine analüüsida kohalike omavalitsuste valimiste andmeid, kuid ei suutnud leida uute omavalitsuste kaarti töödeldaval kujul (näiteks Shapefile-formaadis). Et saada oma andmete jaoks sobiv kaart, võtsin aluseks olemasoleva, [asustusüksuste](http://geoportaal.maaamet.ee/est/Andmed-ja-kaardid/Haldus-ja-asustusjaotus-p119.html) kaardi, ühendasin liituvad piirkonnad ning nimetasin need uute nimedega. Järgnevalt esitan väikse juhendi, kuidas seda R-is teha.

Kõigepealt laadin tööks vajalikud paketid.

```{r, message=FALSE, warning=FALSE}
library(dplyr)
library(gridExtra)
library(maptools)
library(readr)
library(rgdal)
library(rgeos)
library(stringr)
library(tmap)
```

Muudetava kaardina kasutan [Maa-ameti Geoportaali](http://geoportaal.maaamet.ee/est/) Eesti asustusüksuste kaarti. Väiksemate üksuste kaarti kasutan sellepärast, et mõned omavalitsused jagunesid peale haldusreformi mitme uue omavalitsuste vahel ja ei liitunud alati üksteisega terviklikult. Asutustüksuste kaardi põhjal on võimalik näiteks erinevad külad või linnaosad liita erinevate omavalitsustega.

Kaardiandmed Shapefile-formaadis saab alla laadida [Maa-ameti Geoportaali](http://geoportaal.maaamet.ee/est/) kodulehelt automaatselt. Tegemist on kokkupakitud zip-failiga, mille saab lahti pakkida vastavat R-i käsku kasutades.

```{r, results='hide'}
download.file("http://geoportaal.maaamet.ee/docs/haldus_asustus/asustusyksus_shp.zip", dest = "asustusyksus_shp.zip", mode = "wb")
unzip("asustusyksus_shp.zip", exdir = "asustusyksus_shp")
```

Kui kaardifail on alla laetud ja lahti pakitud, saab selle töötlemiseks R-i sisse lugeda.

```{r, message=FALSE, results='hide'}
asustusyksused <- readOGR("asustusyksus_shp", "asustusyksus_20171001", encoding = "UTF-8")
```

Igaks juhuks kontrollin ka visuaalselt, kas kaart esitatakse nii nagu peab. Seda võib teha lihtsalt `plot`-käsuga, aga ka näiteks paketi `tm` võimalusi kasutades.

```{r, eval=FALSE}
plot(asustusyksused)
```

Edaspidi kasutan visualiseerimiseks teist, `tm` paketti. Viimast on väga mugav kasutada ning see jälgib `ggplot2` loogikat.

```{r}
tm_shape(asustusyksused) +
  tm_fill(col = "darkolivegreen3") +
  tm_borders()
```

![Asustusüksuste kaart](https://raw.githubusercontent.com/kristel-/kaardid-R/master/asustusyksused.png)

Peale objekti sisselugemist võib natuke täpsemalt uurida, milline see R-is välja näeb ja millist infot sisaldab. Järgneva käsuga saame objekti kohta hulgaliselt kokkuvõtvat infot, k.a geograafiliste andmetega seotud info (kasutatav koordinaatsüsteem jmt).

```{r}
summary(asustusyksused)
```

Kaardi töötlemiseks töötan edasi kaardifaili atribuuttabeliga.

```{r}
head(asustusyksused@data, 10)
```

Kuna ülesandeks on liita omavalitsused (`ONIMI`), aga liita tuleb ka üksikuid külasid või linnaosasid (`ANIMI`), siis ongi siin eelkõige olulised need kaks veergu. Probleemseks osutus korraliku tabeli leidmine, kus oleks kirjas kõik liituvad omavalitsused kohe töödeldaval kujul. Siin kombineerin kokku kas tabelit, mida tuleb natuke ka käsitsi töödelda. Kõigepealt pdf-formaadis tabel [haldusreformi kodulehelt](http://haldusreform.fin.ee/static/sites/3/2017/07/ulevaade_kovid_14.07.2017.pdf), mida on võimalik automaatselt töödeldavaks tabeliks teisendada. Kahjuks ainult sellest ei piisa ja abifailina kasutan ühte [Delfi artikli tabelit](http://www.delfi.ee/news/paevauudised/eesti/graafik-vaata-millised-vallad-said-5000-elanikku-kokku-ja-millised-mitte?id=76807124), mis on küll hästi vormistatud, kui natuke vananenud infoga, haldusreformi jaanuari seisuga.

```{r, results='hide'}
download.file("http://haldusreform.fin.ee/static/sites/3/2017/07/ulevaade_kovid_14.07.2017.pdf", dest = "ulevaade_kovid_14.07.2017.pdf", mode = "wb")
```

See tabel on esitatud kujul: vana omavalitsuse nimi, elanike arv (mis pole siin oluline info, kuid jätan selle tabelisse alles), uue omavalitsuse nimi (kui see jääb samaks, siis väärtus puudub) ning märkus (mis pole siin samuti oluline). 

```{r, message=FALSE}
haldusreform <- read_csv2("haldusreformi_tabel.csv")
head(haldusreform, 10)
```

Järgnevalt muudan tabelit nii, et uue omavalitsuse lahtrid saaksid täidetud ka juhtudel, kui need ei muutu. Ehk siis lisan sama väärtuse vana omavalitsuse veerust uue veergu.

```{r}
haldusreform <- mutate(haldusreform, Uus = ifelse(is.na(Uus), Vana, Uus)) # kui uue omavalitsuse nimi puudub, siis lisa vana, st jääb muutmata
head(haldusreform, 10)
```

Nüüd tuleks tekitada ka kaardifaili atribuuttabelis tulp uute omavalitsuste nimedega. Selleks tekitan kõigepealt uue tulba `UUE_ID`, kuhu alustuseks lihtsalt kopeerime vanade omavalitsuste nimed.

```{r}
asustusyksused@data$UUE_ID <- asustusyksused@data$ONIMI
head(asustusyksused@data)
```

Nüüd oleks vaja uues tekitatud veerus asendada vanade omavalitsuste nimed uutega. Selleks kasutan varem sisseloetud haldusreformi-tabelit. Selle tabeli veergude abil tekitan nn nimedega vektori (*named vector*). Selline vektor on sarnane Pythoni `dictionary` andmetüübiga, kus kõigile vanade omavalitsuste nimedele vastavad nende kuulumise järgi uute omavalitsuste nimed.

```{r}
vana <- haldusreform$Vana
uus <- haldusreform$Uus
names(uus) <- vana # named vector asendamiseks vanade omavalitsuste nimed uutega
```

Seda nimedega vektorit kasutades teen ühe käsuga kõik asendused. Muudan `UUE_ID` veeru väärtused nimedega vektori väärtustega, vanade omavalitsuste nimed uutega.

```{r}
asustusyksused@data$UUE_ID <- str_replace_all(asustusyksused@data$UUE_ID, uus)
head(asustusyksused@data)
```

Nagu eelpool mainitud, ei piisa ainult omavalitsuste liitmisest, vaid osad vanad omavalitsused jagunesid mitme uue vahel. Et ka seda arvesse võtta ja õiged külad liita õigete uute omavalitsustega, tekitasin abifaili, mis esitab asustusüksuse nime, selle omavalitsuse ning uue omavalitsuse nime, kuhu see asutustüksus (reeglina küla) peaks kuuluma. Ainult asustusüksuse infost ei piisaks, kuna sama küla nimi näiteks võib esineda mitmes ovalitsuses ning oleks kuidagi vaja teha kindalaks, et õigesse omavalitsusse kuuluv küla liidetakse õige uue üksusega.

```{r, results='hide', message='FALSE'}
haldusreform_lisa <- read_csv2("haldusreformi_tabel_yksikud_kylad.csv")
head(haldusreform_lisa)
```

Siin teen asendused natuke teistmoodi, kuna arvesse on vaja võtta rohkem kui ühte kriteeriumit (nt küla ja omavalitsus), mille põhjal asendused teha. Selleks käime läbi kogu lisatabeli ja teeme kaardifaili atribuuttabelis vastavad asendused.

```{r}
for (row in 1:nrow(haldusreform_lisa)) {
  A <- as.character(haldusreform_lisa[row, "ANIMI"])
  O <- as.character(haldusreform_lisa[row, "ONIMI"])
  U <- as.character(haldusreform_lisa[row, "UUS"])
  asustusyksused@data <- mutate(asustusyksused@data, UUE_ID = ifelse(ANIMI == A & ONIMI == O, U, UUE_ID))
}
```

Nüüd on kaardifaili atribuuttabelis igale vanale asutusüksusele liidetud tema uue omavalitsuse väärtus veerus (`UUE_ID`).

```{r}
head(asustusyksused@data)
```

Uute tekitatud väärtuste abil on võimalik ühendada ka kaardifailis kõik olemasolevad asustusüksused uuteks valdadeks ja linnadeks. Ning tulemust saab kohe näha ka visuaalselt ja võrrelda vanade omavalitsuste kaardiga.

```{r, message=FALSE, results='hide'}
uute_ov_kaart <- unionSpatialPolygons(asustusyksused, asustusyksused@data$UUE_ID)
```

```{r, eval=FALSE}
tm_shape(uute_ov_kaart) +
  tm_fill(col = "darkolivegreen3") +
  tm_borders()
```

```{r, results='hide'}
download.file("http://geoportaal.maaamet.ee/docs/haldus_asustus/omavalitsus_shp.zip", dest = "omavalitsus_shp.zip", mode = "wb")
unzip("omavalitsus_shp.zip", exdir = "omavalitsus_shp")

vanad_ov <- readOGR("omavalitsus_shp", "omavalitsus_20171001", encoding = "UTF-8")
```

```{r, echo=FALSE, message=FALSE}
vanade_kaart <- tm_shape(vanad_ov) +
  tm_fill(col = "darkolivegreen3") +
  tm_borders(alpha = 0.2)

uute_kaart <- tm_shape(uute_ov_kaart) +
  tm_fill(col = "darkolivegreen3") +
  tm_borders(alpha = 0.2)

current.mode <- tmap_mode("plot")
tmap_arrange(vanade_kaart, uute_kaart, ncol = 2, nrow = 1)
tmap_mode(current.mode)
```

![Vanad ja uued kohalikud omavalitsused](https://raw.githubusercontent.com/kristel-/kaardid-R/master/vanad-ja-uued-kovid.png)

Edasiseks kasutamiseks ja Shapefile-formaadis uue kaardifaili salvestamiseks teisendan uue tekitatud objekti `SpatialPolygonsDataFrame` andmetüübiks. 

```{r}
uute_ov_df <- SpatialPolygonsDataFrame(uute_ov_kaart, data.frame(UUS_ONIMI = getSpPPolygonsIDSlots(uute_ov_kaart), row.names = getSpPPolygonsIDSlots(uute_ov_kaart)))
```

Kaarti on võimalik vastavalt vajadustele muuta. Näiteks Võib lisada kaardile kõigi omavalitsuste nimed. Selleks võtan kõigist polügonidest (omavalitsuste piirkondadest) nende keskpunktid, kuhu soovin vastava sildi lisada. Lisaks muudan natuke olemasolevaid omavalitsuste nimesid, et nad kaardile paremini ära mahuks.

```{r, warning=FALSE, message=FALSE}
uute_centr <- gCentroid(uute_ov_df, byid=TRUE)

uute_ov_df@data$silt <- gsub(" vald", "", uute_ov_df@data$UUS_ONIMI)

uued_omavalitsused <- tm_shape(uute_ov_df) +
  tm_polygons("MAP_COLORS", palette="Pastel2") +
  tm_borders() +
  tm_borders(alpha = 0.1) +
  tm_text(text = "silt", size = 0.4)

save_tmap(uued_omavalitsused, "uued_omavalitsused.png", width=1920, height=1080)
```

```{r, echo=FALSE, warning=FALSE}
tmap_mode("plot")
tm_shape(uute_ov_df) +
  tm_polygons("MAP_COLORS", palette="Pastel2") +
  tm_borders() +
  tm_borders(alpha = 0.1) +
  tm_text(text = "silt", size = 0.6)
```
![Uute omavalitsuste kaart](https://raw.githubusercontent.com/kristel-/kaardid-R/master/uued-kovid-nimedega.png)

Lõpuks salvestan kaardi uue Shapefile-formaadis failina, mida on võimalik kasutada muude programmidega (nt mõne GIS tarkvaraga) või ka uuesti sisselugeda uute projetkide jaoks, kus sellist kaarti võiks vaja minna.

```{r, eval=FALSE}
writeOGR(obj=uute_ov_df, dsn="uued_omavalitsused", layer="omavalitsused", driver="ESRI Shapefile")
```

