+++
title = "Eestikeelsete stoppsõnade loend [Estonian stop words]"
date = '2018-04-18'
author = "Kristel Uiboaed"
tags = [
  "tekstianalüüs",
  "tekstikaeve",
  "loomuliku keele töötlus",
  "korpuslingvistika",
  "stoppsõnad",
  "text-mining",
  "NLP",
  "corpus linguistics",
  "stop words",
  "Python"]
+++

## Stoppsõnad
Stoppsõnad (*stop words*) on keeletehnoloogia ja tekstikaeve töös tavaliselt sõnad, mis teksti sisu analüüsimisel ütlevad selle sisu kohta vähe. Kindlasti pole olemas ühte lõplikku ja õiget loendit ja ebahuvitavad sõnad võivad olla eri ülesannete lahendamisel erinevad. Reeglina loetakse stoppsõnade hulka sidesõnad (*aga, et, kui, sest*), asesõnad (*see, tema, need, meie*), mõned rohkem grammatilise tähendusega tegusõnad (*olema, saama*), "sisutühjemad" määrsõnad (*nii, siis*) jne. Need on sõnad, mida esineb tekstides palju, kuid teksti sisu kohta ütlevad need sõnad reeglina vähe. Tekstikaeve ülesannetes kasutatakse stoppsõnade loendit enne põhjalikumat tekstianalüüsi nii, et tekstist eemaldatakse kõik loendis olevad sõnad ehk sõnad, mis meid teksti sisu analüüsimisel tõenäoliselt ei huvita.

## Eesti keele stoppsõnad
Mul endal on sageli ette tulnud olukordi, kus sellist loendit oleks väga vaja, kuid kohe sobivat pole kusagilt võtta. Nii olen teinud palju erinevaid loendeid kiiresti käigu pealt konkreetsete ülesannete lahendamiseks. Siit-sealt võib leida erinevaid loendeid, kuid sageli peab neidki oma vajaduste jaoks rohkem või vähem kohendama. Niisiis mõtlesin, et teen lõpuks ühe põhjalikuma ja universaalsema, mis oleks vajadusel kohe võtta.

Tegin kaks erinevat loendit. Üks neist on tekstisõnade ehk sõnavormide loend: loend sisaldab teatud sõnade kõiki vorme (*selline, sellise, sellist, sellisega*; *oli, on, oleme, olime* jne) ja lisaks ka hulga muutumatuid sõnu (kaassõnad, sidesõnad jmt). Teine loend sisaldab lisaks muutumatutele sõnadele ainult pöörd- ja käändsõnade algvorme (*selline, olema*). Niisiis võib rakendada ühte loendit juhul, kui meil on [lemmatiseerimata](https://kristel.gitbooks.io/sissejuhatus-tekstikaevesse/content/tekstikaeve-terminid.html) tekst ning teist loendit [lemmatiseeritud](https://kristel.gitbooks.io/sissejuhatus-tekstikaevesse/content/tekstikaeve-terminid.html) tekstidel. Tekstisõnade loendit võib kasutada ka lemmatiseeritud teksti jaoks, kuna loend sisaldab lisaks muudele vormidele ka loomulikult sõnade algvorme. Tegin kaks loendit selleks, et üks on märkimisväärselt lühem ning kui sellest piisab, ei pea pikemat loendit kasutama.

Mõlemad loendid leiab [Tartu Ülikooli andmerepositooriumist](). [GitHubi repositooriumisse](https://github.com/kristel-/estonian-stopwords)  lisasin abifailid ja ka [Pythoni töölehe](https://github.com/kristel-/estonian-stopwords/blob/master/stopword-list-generation.ipynb), mille abil loendi koostasin. Töölehel on esitatud näiteks sammud, kuidas sõnavorme automaatselt genereerida. Seega on töölehest abi, kui soovid koostada oma loendit või midagi olemasolevasse loendisse lisada või sealt eemaldada.

## Tähelepanu
Nagu eelpool juba mainitud, pole olemas ühte universaalset stoppsõnade loendit. Seega ei ole ka siin esitatud loend kindlasti sobiv kõigi ülesannete lahendamiseks. Kui on oluline teatud tegusõnade info, siis võib nee praegusest loendist eemaldada. Lisaks võivad ka muutumatud sõnad ja grammatilisemad sõnad olla teksti sisu seisukohalt olulised, näiteks võivad need sisalduda püsiühendites (*viga saama*), mis moodustavad tähenduse teiste sõnadega kombineerudes. Püsiühenditega toimetulek on aga juba laiem teema. Need on mõned asjad, mis tuleks enne tekstitöötlust selgeks mõelda.




