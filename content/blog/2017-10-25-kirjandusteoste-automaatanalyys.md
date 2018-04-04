+++
title = "Kirjandusteoste automaatanalüüs [Text-mining and Stylometric Analysis of Estonian Novels]"
date = '2017-10-25'
author = "Kristel Uiboaed"
tags = [
  "stilomeetria",
  "kirjandus",
  "tekstianalüüs",
  "tekstikaeve",
  "loomuliku keele töötlus",
  "korpuslingvistika",
  "andmeanalüüs",
  "stilometry",
  "text-mining",
  "NLP",
  "data analysis",
  "corpus linguistics",
  "R"]
+++


# Sissejuhatus

Humanitaar- ja sotsiaalteaduslikes uurimistöödes on üks peamisi andmetüüpe mingil kujul tekst, mida andmeteaduses nimetatakse sageli struktureerimata andmeteks. See võib olla sotsiaalmeedia postituste kogu, ajakirjandustekstid, arhiivimaterjalid, kirjandusteosed, intervjuud vmt. Eri tüüpi vabalt kättesaadavaid andmeid on meie kasutuses järjest rohkem, kuid selliste andmete kogumine ja analüüsitavale kujule viimine eeldab üsnagi mahukat eeltööd ja mõningase tehnilise töö oskust. Kust selliseid andmeid üldse leida, kuidas need kokku koguda, kuidas sealt kätte saada uurimisküsimuse seisukohalt oluline materjal ning kuidas see siis lõpuks viia kvantitatiivse analüüsi mingile ühtlustatud kujule, mida on võimalik töödelda, mõõta ja analüüsida? Järgnevalt esitatud näide illustreerib, kuidas neid küsimusi võiks praktiliselt lahendada.

Veeb on täis materjali, mida on võimalik automaatselt kokku koguda, kiirelt töödelda ja seda seejärel erinevatest vaatepunktidest analüüsida. Käesolev näide esitab kirjandusklassika automaatse ja kvantitatiivse analüüsi näitel sellise uurimistöö põhietapid. Iga uurimistöö algab loomulikult uurimisküsimusega, kuid kui see on selge, tuleb mõelda, millised on võimalikud andmed ja sobivaimad analüüsimeetodid, mis sellele küsimusele kõige paremini aitaksid vastata, kuid mis oleksid samas ka kättesaadavad või on neid võimalik mõistliku aja- ja töökuluga koguda ja töödelda. Kui sobivad andmeallikad on leitud, tuleks andmed kokku koguda ja kindlasti on möödapääsmatu struktureerimata andmete korrastamine ja ühtlustamine.

Näide põhineb e-raamatute analüüsil, mis on automaatselt alla laaditud [Tartu linnaraamatukogu](http://www.luts.ee/index.php/e-raamatud) veebilehelt. Raamatud on vabalt kasutatavad ning neid võib igaüks oma vajadustest lähtuvalt töödelda ja analüüsida. Järgnevalt esitan ühe võimaluse raamatute automaatseks allalaadimiseks veebilehelt, allalaaditud failide töötlemiseks ja salvestamiseks. Kirjandusteoste analüüsimiseks rakendan stilomeetriat. Stilomeetria on traditsiooniliselt olnud küll kirjandusliku teksti: žanri, autorluse jmt varieervuse analüüsimeetod, kuid selle abil on võimalik läheneda ka [ajaloolistele](http://www.sciencedirect.com/science/article/pii/S0957417416303116), [politoloogilistele](http://aliarsalankazmi.github.io/blog_DA/posts/r/2016/11/18/authorial_analysis_pm.html), [soouuringute](https://academic.oup.com/dsh/article/31/4/746/2748261/Vive-la-difference-Tracing-the-authorial-gender) ja paljudele teistele uurimisküsimustele, mille alusmaterjaliks on mingit tüüpi tekst. Analüüsi eesmärk on praegusel juhul tuvatada, kas eri autorite teosed üksteisest erinevad ning kas samade autorite teosed on uuritava tunnuse põhjal sarnased.

# Andmed ja tööriistad

Selles töös rakendan programmi [R](https://cran.r-project.org/bin/windows/base/) kogu töö jaoks: andmete allalaadimiseks, kogumiseks, töötlemiseks ja analüüsimiseks. R-i mugavamaks kasutamiseks soovitan kasutada [RStudiot](https://www.rstudio.com/products/rstudio/download/).

## Pakettide laadimine

Lisaks R-i põhipaketile kasutan käske, mille jaoks on vajalik mõne lisapaketi sissetoomine (eelnevalt tuleks need ka installeerida `install.packages("paketiNimi")`).

```{r}
# Load packages
library(rvest) # html processing
library(stringr) # string processing
library(stylo) # stylometric analysis
```

## Raamatute allalaadimine
### Veebilehe linkide tuvastamine

Järgnevad kolm koodirida tuvastavad veebilehe lingid, mille all e-raamatud asuvad. Põhi-URL on alati sama ning sellelt aadressilt loetakse sisse veebilehe [HTML](https://en.wikipedia.org/wiki/HTML)-kood, kust omakorda tuvastatakse kõik hüperlingid (osade külge neist on lingitud ka e-raamatute failid).

```{r}
URL <- "http://www.luts.ee/index.php/e-raamatud" # base URL of library e-books
page <- read_html(URL) # read in the whole webpage
urls <- html_attr(html_nodes(page, "a"), "href") # extract and list all the hyperlinks on the webpage
urls[c(80:95)] # output extracted hyperlinks in positions 80-95
```

### Raamatufailide allalaadimine

Eelnevalt kokkukogutud hüperlinkide abil saab nüüd tuvastada, milliste all neist on e-raamatu failid *epub*-formaadis. Selleks käiakse tsükliga läbi kõik raamatukogu veebilehelt tuvastatud hüperlingid ja kontrollitakse, millised neist lõpevad laiendiga *epub*. Kui link lõpeb *epub*-laiendiga, siis laaditakse see alla, lisaks eemaldatakse kogu hüperlingi info ning failinimena säilitatakse ainult lingi osa, kus on teose autor ja teose pealkiri. Veel asendatakse *epub* osa laiendiga *zip*, mis võimaldab need failid hiljem lahti pakkida. Selle tsükli töö tulemusena on töökataloogi alla laaditud kõik veebilehe *epub*-laiendiga failid, mis on töökataloogis teisendatud lahtipakitavateks *zip*-laiendiga failideks. 

```{r}
for(i in 1:length(urls)) {
  if (endsWith(urls[i], ".epub") == TRUE) { # extract only links with epub-extension
    download.file(paste0("http://www.luts.ee", urls[i]), paste0(gsub(".*\\/", "", urls[i]), ".zip"), mode="wb") # paste the main link with the epub-book link and download the file with the zip-extension, maintain only the author and the title of a book as a file name
  }
}
```

Vahepeal võib vaadata töökataloogi sisu ja näha on, et lisaks kõigile teistele failidele on seal ka e-raamatute *zip*-failid.

```{r, message=FALSE, warning=FALSE}
# List first files in the working directory
head(list.files())
```

### Raamatufailide lahtipakkimine

Nüüd koostatakse loend *zip*-laiendiga lõppevatest töökataloogi failinimedest. Järgnev tsükkel hakkab läbi käima ülesleitud *zip*-faile ja pakib igaühe neist lahti samanimelisse kausta.

```{r}
filenames <- list.files(pattern="*.zip", full.names = T, include.dirs=T) # read in zipped e-books for processing
for (fileName in filenames) {
        unzip(fileName, exdir = gsub("\\.epub\\.zip", "", fileName)) # extract zipped e-book files to corresponding folders
}
```

## Teksti koondamine ja puhastamine

Nüüd, kui failid on lahti pakitud, tuleks sealt üles leida teoste tekstiline osa, mida eelkõige oleks mõistlik analüüsida. Kataloogid sisaldavad ka muud kirjandusteoste info (nt kaanepildid jmt), mis oleks tekstilise analüüsi materjalist vaja välja jätta.

Iga kirjandusteose lahtipakitud kaustas on veel omakorda kaustad, millest tekstiline osa on ühes kaustas ja tekstiosa sisaldavate failide nimed on eraldi markeeritud. Et tekstiosa kätte saada, peame läbi käima iga teose kausta, leidma üles kataloogi, kus on teose põhitekst (`OEBPS/Text`) ja sealt üles leidma tekstiosaga failid. Praegu on need eri teoste puhul nimetatud erinevalt (`Section|page|split|text|Text`). Seega peame skriptis ära märkima kõik võimalikud variandid. Teoste tekstiosa on paigutatud veel omakorda eraldi falidesse nt peatükkide kaupa. Hilisema analüüsi seisukohalt oleks aga hea, kui ühe teose tekstiosa oleks ühes failis. Veel tuleb teha natukene teksti puhastamise tööd, näiteks ühtlustada jutumärgid, kustutada mõned ebasoovitavad sümbolid jmt. Kui see töö on tehtud liidetakse salvestatakse iga teos eraldi ühte tekstifailina.

```{r}
for (subdir in list.dirs(recursive=FALSE)) { # loop through book files
        print(subdir) # show process
        contentFiles <- list.files(paste0(subdir, "/OEBPS", "/Text")) # only process files in Text directory which contains the content of books
        oneNovel <- vector() # create an empty vector for one book and the content will be collected into this vector
        for (fileName in contentFiles) { # loop through content files
                if(grepl("^(Section|page|split|text|Text)", fileName) == TRUE) { # process only files that contain corresponding parts, because these are text content files
                        myFile <- paste0(getwd(), paste0(subdir, "/OEBPS", "/Text"), "/", fileName)
                        oneLineFile <- paste0(readLines(myFile, encoding = "UTF-8"),collapse=" ") # read in the content file do be processed
                        removedHeader <- gsub("<\\?xml.*<\\/head>", "", oneLineFile) # continue with some text processing
                        removedTags <- gsub("<[^>]*>", "", removedHeader)
                        editText <- gsub("\\&nbsp;", "", removedTags)
                        editText2 <- gsub("\\s+", " ", str_trim(editText))
                        editText3 <- gsub("\012", " ", editText2)
                        editText4 <- gsub("\015", " ", editText3)
                        editText5 <- gsub("[«»„”““]", "\"", editText4)
                        editText6 <- gsub("[\\*_]", "", editText5)
                        editText7 <- gsub("\\’\\’", "\\'", editText6)
                        oneNovel <- append(oneNovel, editText7) # aggregate one book into the one vector created above
                }
        }
        write(oneNovel, file = paste0(subdir, ".txt"), sep=" ") # output the result into the txt-file (one file contains one novel)
}

```

# Stilomeetriline analüüs

Kui uurimismaterjal^[Analüüsist on välja jäetud luulekogud ja mõned näidendid. Lisaks on siinsest töövoost välja jäetud kõigi failide utf-8 kodeeringule teisendamine. Selle jaoks on kasutatud [shelli skripti](https://github.com/kristel-/Corpus-Linguistics/blob/master/Unix/convertEcoding.sh).] on viidud tavalisele tekstikujule (*plain text*), on sellega edasi toimetada lihtne. Näiteks võime teha n-ö [stilomeetrilist analüüsi](https://sites.google.com/site/computationalstylistics/) ehk lihtsustatult öeldes "mõõta", millised kirjandusteosed on kvantitatiivselt stiili poolest üksteisele sarnasemad ja millised erinevamad. Järgnev analüüs on läbi viidud R-i paketiga `stylo` [(Eder, Rybicki & Kestemont 2016)](https://journal.r-project.org/archive/2016-1/RJ-2016-1.pdf).

```{r}
# stilometry
stylo(gui=T)
```

```{r}
corpus.format = "plain"
corpus.lang = "Other"
analyzed.features = "c"
ngram.size = 4
preserve.case = FALSE
encoding = "UTF-8"
mfw.min = 100
mfw.max = 100
mfw.incr = 100
start.at = 1
culling.min = 0
culling.max = 0
culling.incr = 20
mfw.list.cutoff = 5000
delete.pronouns = FALSE
use.existing.freq.tables = FALSE
use.existing.wordlist = FALSE
use.custom.list.of.files = FALSE
analysis.type = "PCV"
consensus.strength = 0.5
sampling = "no.sampling"
sample.size = 10000
number.of.samples = 1
display.on.screen = TRUE
write.pdf.file = FALSE
write.jpg.file = FALSE
write.svg.file = FALSE
write.png.file = TRUE
plot.custom.height = 7
plot.custom.width = 10
plot.font.size = 8
plot.line.thickness = 1
text.id.on.graphs = "labels"
colors.on.graphs = "colors"
titles.on.graphs = FALSE
label.offset = 0
add.to.margins = 2
dendrogram.layout.horizontal = TRUE
pca.visual.flavour = "classic"
save.distance.tables = FALSE
save.analyzed.features = FALSE
save.analyzed.freqs = FALSE
dump.samples = FALSE
```

Siinne analüüs on läbi viidud ülal esitatud parameetrite põhjal. Analüüsimaterjali koosnes kirjandusteoste lihttekstist ning analüüsitud on neljaliikmelisi tähekombinatsioon (neligramme). Neligrammid moodustatakse iga teose jaoks eraldi üle kogu ühe teose (*elas kord* --> *\"elas\", \"las \", \"as k\", \"s ko\", \" kor\", \"kord\"* jne). Seejärel moodustatakse neist sagedusloendid ja selle põhjal arvutatakse vastavalt valitud meetodile teostevahelised erinevused. Praegu on analüüsimeetodiks valitud peakomponentanalüüs (*principal component analysis* - PCA). Joonis esitab analüüsi tulemused graafiliselt.

![](neligrammid_pca.jpg)
![Neligrammide peakomponentanalüüs](https://raw.githubusercontent.com/kristel-/Kirjandusteoste-automaatanalyys/master/neligrammid_pca.jpg)

Kuidas neid tulemusi tõlgendada?^[PCA tulemuste ja tõlgendamise kohta vt täpsemalt nt Levshina 2015: 353--366] Lihtsustatult öeldes võib öelda, et teosed, mis on graafikul üksteisele lähemal, on neligrammide põhjal sarnasemad, ning kaugemal asetsevad teosed erinevamad. Näiteks näeme, et Jüri Parijõgi  mõned teosed on väga saransed, samas kui üks neist on oma stiililt hoopis sarnasem ühe Eduard Vilde teosega. Mida neligrammid meile teoste stiili kohta öelda võiks? Näiteks kombineeruvad sel viisil lisaks kõigele muule ka erinevad käände- ja pöördelõpud, nende sagedusloendid võiks muu hulgas osutada sellele, kas autor kasutab rohkem *mina*- või *tema*-vormi, millises ajavormis tegevust edasi antakse jne. Loomulikult võib teha sarnast analüüsi ka sõnade põhjal, mis intuitiivselt võiks olla arusaadavama stiilimarker. Selleks tuleb paketi käivitamisel valida lihtsal vastav parameeter (*words*). Eesti keelt analüüsides tekib siin sõnavormi ja sõna algvormi analüüsimise probleem. Kui soovime näiteks leida sõnu, mis just autoreid kõige rohkem eristaksid, siis sagedused hajuksid eri käände- ja pöördevormide vahel (*nägime, nähti, näinud, nägin* --> *nägema*), st et iga sõnavorm loetaks erinevaks sõnaks, kuigi tegemist on sama sõna eri vormidega. Teksti "algvormistamiseks" ehk lemmatiseerimiseks on vahendid olemas^[Eestikeelse teksti lemmatiseerimiseks võib rakendada näiteks Pythoni [Estnltk paketti](https://estnltk.github.io/estnltk/1.4.1/index.html), lemmatiseerimise jaoks on olemas lihtsustatud Estnltk-d rakendav [skript](https://github.com/kristel-/Tekstikoolitus-2017/blob/master/15-02-praktikum/lemmatizeFiles.py).] ning sõna algvormi taseme analüüsiks tuleks sisendtekstid eelnevalt lemmatiseerida ja seejärel teha stilomeetriline analüüs lemmatiseeritud tekstide põhjal. [Rybicki ja Eder (2011)](https://www.researchgate.net/publication/220675399_Deeper_Delta_across_genres_and_languages_Do_we_really_need_the_most_frequent_words) on natuke n-grammide ja algvormide valiku sobilikkust testinud nii eri keelte kui ka žanrite peal ja leidnud, et iga olukorra jaoks ühte õiget ja parimat valikut pole ning nagu igasuguse andmeanalüüsi puhul, tuleks sobivaima meetodi valimisel lähtuda materjalist ja uurimisküsimusest.

Selline automaatne andmetöötlus ja kvantitatiivne analüüs annab väga kiiresti üldpildi võimalikest autorite ja/või teoste vahelistest erinevustest, mida teoseid läbi lugedes oleks keeruline tuvastada. Kvantitatiivse analüüsi tulemuste tõlgendamiseks tuleks edasi minna kvalitatiivse analüüsiga ning selle põhjal tuvastada, millest tulevad sisulised erinevused. Stilomeetriline analüüs väljastab lisafailid, kus on esitatud loendid neligrammidest ja arvulistest väärtustest, mis olid kvantitatiivse analüüsi tulemuste aluseks. Seda informatsiooni saab edasi analüüsida, tuvastamaks sisulisi põhjusi, miks autorid või nende teosed üksteisest erinevad.

# Andmete ja uurimistulemuste avaldamine

Uurimisandmete hulka kuuluvad nii algandmed kui ka muud töö käigus loodud abifailid. Praegusel juhul kuulub andmete hulka kindlasti ka R-i skript, mille abil andmeid koguti, töödeldi ja analüüsiti. Kuna analüüsimiseks kasutati juba töödeldud ja muudetud kirjandusteoseid, siis algandmetena on esitatud siin need failid. Publikatsiooni koos andmete ja skritpidega võib üles laadida näiteks [Tartu Ülikooli andmerepositooriumisse](https://datadoi.ut.ee/), kus kogu tööle omistatakse automaatselt ka unikaalne digitaalse objekti tunnus (*Digital Object Identifier* - DOI), mis muudab kogu töö, k.a andmed ja skriptid ka teiste jaoks kasutatavaks ja viidatavaks. Niimoodi on kogu eelpool esitatav töö taaskasutatav ja täielikult reprodutseeritav.

# Kokkuvõte

Eelnevalt esitasin ülevaate, kuidas R-iga veebilehtedelt materjali koguda, seda töödelda ja seejärel analüüsida. Siin keskendusin stilomeetrilisele analüüsile, kuid teksti analüüsimiseks on võimalusi palju rohkem. Kindlasti aga ei lõpeks uurimistöö siin: selline kvantitatiivne analüüs on sisendiks järgnevale kvalitatiivsele osale, mis peaks pakkuma seletusi, millised ja millest on tingitud sisulised erinevused.

# Kirjandus
Eder, Maciej, Jan Rybicki, and Mike Kestemont (2016). *Stylometry with R: a package for computational text analysis.* R Journal, 8(1): 107-121.

Levshina, Natalia (2015). *How to Do Linguistics with R: Data Exploration and Statistical Analysis.* Amsterdam/Philadelphia: John Benjamins Publishing Company.
