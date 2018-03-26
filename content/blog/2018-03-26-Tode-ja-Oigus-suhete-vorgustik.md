+++
title = "Tõe ja Õiguse suhete võrgustik: eeltöötlus [Network of Novel Characters: Preprocessing]"
date = '2018-03-25'
author = "Kristel Uiboaed"
tags = [
  "kirjandus",
  "tekstianalüüs",
  "tekstikaeve",
  "loomuliku keele töötlus",
  "korpuslingvistika",
  "andmeanalüüs",
  "ilukirjandus",
  "võrgustikuanalüüs",
  "Gephi",
  "Python",
  "text-mining",
  "NLP",
  "data analysis",
  "corpus linguistics",
  "network analysis",
  "Gephi",
  "Python"]
+++

See Pythoni tööleht esitab sammud, kuidas ühte kirjandusteost analüüsiks natuke puhastada, ühtlustada ning seejärel võrgustikuanalüüsiks ette valmistada.

Esmalt tekst tükeldatakse 200 sõna suurusteks osadeks. Seejärel moodustatakse iga väikse tekstilõigu sees nimede paarid. Võib eeldada, et tegelased, keda mainitakse koos lühemas tekstilõigus ja sageli, on teoses omavahel rohkem seotud. Selliste nimepaaride põhjal saame joonistada ühe romaani suhete võrgustiku.

Kuna analüüsi aluseks tekst on automaatselt töödeldud ja veebist tõmmatud esineb seal vigu. Näiteks enne teksti alvormistamist ehk lemmatiseerimist parandan ära mõned trükivead. Kuna selles analüüsis keskendun ainult isikunimedele, siis pööran tähelepanu ainult vigadele, mis esinevad pärisnimedes. Nimede asendused on failis `name-replacments.txt`. Puhastatud ja parandatud fail on lemmatiseerimise sisendiks. See samm on praegu töölehel vahele jäetud.

Kuna lemmatiseerimine viiakse läbi automaatselt, siis on loomulik, et mõned pärisnimed jäävad tundmatuks või tulemuses on muud vead. Et sellest tulenevaid vigu väljundis natuke vähendada, on mõned vigased nimed nimepaaride loendist välja visatud. Samuti on välja visatud muud pärisnimed (näiteks kohanimed), kuna hetkel oleme huvitatud ainult isikunimedest. Eemaldatud sõnade loend on failis `excluded-names.txt`.

Lõpuks korrastatakse väljundit nii, et sellest tuleks tabel. Eemaldatakse madala sagedusega paarid ning korrastatakse tabelit selleks, et seda oleks hiljem lihtsam võrgustikuanalüüsi sisendina kasutada.

