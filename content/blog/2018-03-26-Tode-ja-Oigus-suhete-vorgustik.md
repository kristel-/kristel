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

# Sissejuhatus

See Pythoni tööleht esitab sammud, kuidas ühe kirjandusteose teksti tekstianalüüsiks, korrigeerida, ühtlustada ning seejärel võrgustikuanalüüsiks ette valmistada. E-raamatu faili eeltöötluse kohta saad täpsemalt lugeda [ühest eelnevast postitusest](http://www.tekstikaeve.ee/blog/2017-10-25-kirjandusteoste-automaatanalyys/). Siin keskendun juba eelnevalt puhastatud teksti korrigeerimisele ja ettevalmistamisele võrgustikuanalüüsiks. Töös kasutatud abifailid ja Pythoni tööleht on kättesaadav ka [GitHubis](https://github.com/kristel-/Preprocessing-of-a-novel-for-network-analysis).

# Töö etapid

Järgnevalt esitan lühidalt töö etapid, mis võrgustikuanalüüsiks oli vaja läbida. Allpool on lühikeste kommentaaridega esitatud ka Pythoni töövoog.

- Esmalt tekst tükeldatakse 200 sõna suurusteks osadeks. Seejärel moodustatakse iga väikse tekstilõigu sees nimede paarid. Võib eeldada, et tegelased, keda mainitakse sageli koos lühemas tekstilõigus, on teoses omavahel rohkem seotud. Selliste nimepaaride põhjal saame joonistada ühe romaani suhete võrgustiku.

- Kuna analüüsi aluseks olev tekst on automaatselt töödeldud ja veebist tõmmatud esineb seal vigu. Näiteks enne teksti alvormistamist ehk lemmatiseerimist parandan ära mõned trükivead. Kuna selles analüüsis keskendun ainult isikunimedele, siis pööran tähelepanu ainult vigadele, mis esinevad pärisnimedes. Nimede asendused on failis `name-replacments.txt`. Puhastatud ja parandatud fail on lemmatiseerimise sisendiks. See samm on praegu töölehel vahele jäetud.

- Kuna lemmatiseerimine viiakse läbi automaatselt, siis on loomulik, et mõned pärisnimed jäävad tundmatuks või tulemuses on muud vead. Et sellest tulenevaid vigu väljundis natuke vähendada, on mõned vigased nimed nimepaaride loendist välja visatud. Samuti on välja visatud muud pärisnimed (näiteks kohanimed), kuna hetkel oleme huvitatud ainult isikunimedest. Eemaldatud sõnade loend on failis `excluded-names.txt`.

Lõpuks korrastatakse väljundit nii, et sellest tuleks tabel. Eemaldatakse madala sagedusega paarid ning korrastatakse tabelit selleks, et seda oleks hiljem lihtsam võrgustikuanalüüsi sisendina kasutada.

Võrgustikuanalüüsi tegemiseks võib kasutada eri vahendeid. Esitan siin ühe näite R-i ja [Gephiga](https://gephi.org/) tehtud võrgustikust.

<figure><img src='/img/Tode-ja-Oigus-vorgustik-R.png'><figcaption>Joonis 1. Võrgustik R-ga</figcaption></figure>
<figure><img src='/img/Tode-ja-Oigus-vorgustik-Gephi-2.png'><figcaption>Joonis 2. Võrgustik Gephiga</figcaption></figure>



# Pythoni tööleht (*notebook*) kirjandusteose töötlemiseks ja võrgustiku analüüsiks ettevalmistamiseks

*Kõik abifailid ja Pythoni töölehe leiad [GitHubist](https://github.com/kristel-/Preprocessing-of-a-novel-for-network-analysis).*

Võta kasutusele tööks vajalikud paketid.


```python
import re
from itertools import combinations
from itertools import permutations
from collections import Counter
import pandas as pd
```

Teksti tükeldamise [funktsioon](https://github.com/PacktPublishing/Python-Machine-Learning-Cookbook/blob/master/Chapter06/chunking.py) `splitter`, mille sisendiks on tekst ja soovitud tekstiosade pikkus sõnade arvuna. Tekst tükeldatakse tühikute kohalt.


```python
# Split a text into chunks 
def splitter(data, num_words):
    words = data.split(' ')
    output = []

    cur_count = 0
    cur_words = []
    for word in words:
        cur_words.append(word)
        cur_count += 1
        if cur_count == num_words:
            output.append(' '.join(cur_words))
            cur_words = []
            cur_count = 0

    output.append(' '.join(cur_words) )

    return output 
```

Loo failist asenduste leksikon. Algfailis olevaid isikunimed ühtlustatakse. Parandatakse mõned vead ning teisendatakse hüüdnimed ja eri nimekujud ühtsele kujule (Krõet --> Krõõt jne), kuna tegemist on sama isikuga ja oleme huvitatud tegelaskujude paaridest.


```python
name_replacements = {}
with open("name-replacements.txt", 'r', encoding='utf-8') as f:
    for line in f:
       (key, val) = line.split(";")
       name_replacements[key.strip()] = val.strip()
```

Algfailis tehakse nimede asendused ning väljund kirjutatakse uude faili.


```python
with open("Anton_Hansen_Tammsaare_Tode_ja_oigus_I.utf8", 'r', encoding="utf-8") as f:
    content = f.read()
    content = re.sub(r'\b' + '|'.join(name_replacements.keys()) + r'\b', lambda m: name_replacements[m.group(0)], content)

with open("Anton_Hansen_Tammsaare_Tode_ja_oigus_I_asendustega.utf8", "w", encoding = "utf-8") as outp:
    outp.write(content)
```

Loetakse sisse lemmatiseeritud fail, lemmad on moodustatud asendustega failist. Lemmatiseerimise etapp on koodis praegu vahele jäetud.


```python
with open("Anton_Hansen_Tammsaare_Tode_ja_oigus_I_lemmad.txt", 'r', encoding="utf-8") as f:
    content = f.read()
```

Teksti tükeldamine 200-sõnalisteks juppideks.


```python
text_chunks = splitter(content, 200)
```

Kogu tekstist moodustatakse kaheliikmelised kombinatsioonid.


```python
combs = []
for chunk in text_chunks:
    combs.append(list(combinations(chunk.split(" "), 2)))
```

Tekstist võetakse välja ainult need kombinatsioonid, mille mõlema liikmes on mõni suurtäht. Kuna sisendiks oli lemmatiseeritud ja teisendatud fail, siis lausealgulised suurtähed jmt on muudetud väikesteks, seega on potentsiaalselt nimed ainult need, kus esineb suurtäht. Samuti jäetkase välja kombinatsioonid, kus mõlemad liikmed on samad ning kõik kaksikud sorteeritakse enne lõplikku listi lisamist tähestikuliselt.


```python
unique_combs_in_chunks = []
for i in combs:
    for j in i:
        if any(x.isupper() for x in j[0]) and any(x.isupper() for x in j[1]) and j[0] != j[1]:
            unique_combs_in_chunks.append(tuple(sorted(j)))
```

Kuna ka lemmatiseeritud fail pole täiesti puhas, siis oleks vaja kombinatsioone veel natuke puhastada. Välja visatakse kõik kohanimed, kuna oleme huvitatud ainult isikunimede paaridest. Lisaks visatakse välja veel mõned valesti nimeks märgendatud sõnad. Selleks on eelnevalt tekitatud fail loendiga ebasoovitavatest sõnadest.


```python
exclusion = []
with open("excluded-names.txt", 'r', encoding='utf-8') as f:
    for line in f:
        exclusion.append(line.strip())
        
unique_combs_in_chunks = [e for e in unique_combs_in_chunks if e[0] not in exclusion and e[1] not in exclusion]
```

Nimekombinatsioonidest moodustatakse sagedusloend.


```python
combination_freqs = Counter(unique_combs_in_chunks)
```

Nimepaaride sagedustest moodustatakse tabel.


```python
name_df = pd.DataFrame.from_dict(combination_freqs, orient='index').reset_index()
name_df = name_df.rename(columns={'index':'pair', 0:'Count'})
name_df['pair'] = name_df['pair'].apply(','.join)
name_df.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>pair</th>
      <th>Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Maasik,Mari</td>
      <td>4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Eedi,Madis</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Madis,Nonäh</td>
      <td>5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Karja-Eedi,Madis</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Madis,Mart</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



Nimepaaride sagedustega tabelist tekitatakse tabel, kus mõlemad nimed ning sagedus kuuluvad eraldi veergudesse. Paaridega veerg eemaldatakse ning veergude järjekorda muudetakse.


```python
name_df['Source'], name_df['Target'] = zip(*name_df['pair'].map(lambda x: x.split(',')))
name_df = name_df.drop('pair', axis=1)
name_df = name_df.reindex_axis(['Source', 'Target', 'Count'], axis=1)
name_df.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Source</th>
      <th>Target</th>
      <th>Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Maasik</td>
      <td>Mari</td>
      <td>4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Eedi</td>
      <td>Madis</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Madis</td>
      <td>Nonäh</td>
      <td>5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Karja-Eedi</td>
      <td>Madis</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Madis</td>
      <td>Mart</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



Lõpuks võetakse välja ainult paarid, mille esinemissagedus on kümnest suurem. Väljund kirjutatakse eraldi faili.


```python
name_df = name_df.query('Count > 9')
name_df.to_csv("name_pairs.csv", sep = ";", index=False, encoding = "utf-8")
```
