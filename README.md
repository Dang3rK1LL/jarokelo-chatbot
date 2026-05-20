# Járókelő Chatbot Projekt

Ez a projekt a Járókelő hibabejelentő platform két chatbotos felületét tartalmazza: egy belsős ügyintézői modult (adminisztrációs asszisztens) és egy külsős bejelentői modult.

## Belső chatbot (Járókelő Adminisztrációs Asszisztens)

A belsős chatbot célja az adminisztratív és ügykezelési feladatok és a hibabejelentések háttérmunkájának megkönnyítése mesterséges intelligencia segítségével.

### A bot működése (Munkafolyamatok és Csomópontok)

1. **Geocode / Standard hívás (Háttérfolyamat)**
   Már a belépési ponton (Entry) lefut egy folyamat (`Batch Geocoding for Hungari...` kártya), ami automatikusan kiegészíti az adatbázis hiányos koordinátáit. Amennyiben az adatbázisban (`jarokelo_extractTable`) szereplő ügyekhez tartozó `lat` (szélesség) és `lng` (hosszúság) oszlopok üresek, a bot végigfut ezeken a sorokon, és a **LocationIQ API** segítségével frissíti a koordinátákat. Kereséskor a rendszer először egy pontosított, strukturált lekérdezést használ, majd ha az nem hoz eredményt, automatikusan visszavált egy általános (free-form) keresési módra.

2. **Ügy_bekérése node**
   A főfolyamat első lépéseként a bot bekér egy hivatkozást az ügyintézőtől (*"Kérlek adj meg egy ügyet, amivel s..."*), ez lesz a feldolgozni kívánt URL. Ezt a bot az `incoming_issue` változóba menti el.

3. **Adatkinyerés node**
   A bot egy `Execute code` kártya segítségével feldolgozza az `incoming_issue` változóban kapott url-t, és kinyeri belőle az ügy érdemi részleteit:
   - Város
   - Kerület
   - Cím
   - Kategória
   - Ügy leírásának összegzése (summary).
   Az adatokat a rendszer az `extracted_data` (kinyert adatok) változóba gyűjti össze.

4. **Tudásbázis_kinyerés_és_döntés node**
   A `Filter and Limit Database Res...` kód alapján a rendszer leszűri az adatbázist azokra az esetekre, amelyek **ugyanabban a kerületben** találhatóak és **ugyanabba a kategóriába** tartoznak, mint ami az `extracted_data` változóban van.
   A leszűrt esetekből az AI predikciót hajt végre, amely meghatározza az illetékes hatóságot, az ügy felelősét.
   Ennek kimenetét (indoklással) a `final_decision` változó tárolja és mutatja be (`{{workflow.final_decision.reasonin...}}` formájában).

5. **Lokáció_vizsgálat node**
   Végül a `Geocoding and Nearby Case...` szkript megvizsgálja a geokódolt esetek távolságát az aktuális ügyhöz képest. A bot kikeresi az 5 legközelebbi egyezést, amelyből szűrve csak a **ténylegesen 50 méteren belüli** ügyeket listázza ki. A megfelelő találatok leírását a `{{workflow.kozeli_ugyek_text}}` változón keresztül írja ki a képernyőre, majd a folyamat lezárul (End - Go to next workflow).

# Külső chatbot – Járókelő Bejelentő Asszisztens

AI-alapú chatbot rendszer közterületi hibabejelentések automatizált feldolgozására.

---

# Projekt célja

A workflow célja egy intelligens chatbot megvalósítása a Járókelő platform számára, amely:

- támogatja a közterületi hibabejelentéseket,
- AI segítségével elemzi a feltöltött képeket,
- automatikusan felismeri a nem megfelelő tartalmakat,
- képes hajléktalansággal kapcsolatos esetek szűrésére,
- valamint adatvédelmi anonimizálást is végez.

---

# Fő funkciók

## A rendszer képes:

- képfeltöltések fogadására,
- AI Vision alapú képelemzésre,
- hajléktalan detektálásra,
- automatikus blur / censor vizsgálatra,
- hibabejelentési adatok begyűjtésére,
- valamint a bejelentés véglegesítésére.

---

# Workflow áttekintés

## Fő lépések

1. Üdvözlés és adatvédelem
2. Képfeltöltés
3. AI képelemzés
4. Hajléktalan detektálás
5. Döntési logika
6. Leírás megadása
7. Bejelentés véglegesítése
8. Sikeres visszajelzés

---

# 1. Üdvözlés és adatvédelem

## Node
`Üdvözlés_és_adatvédelem`

## Funkció

A chatbot:

- üdvözli a felhasználót,
- ismerteti az adatkezelési szabályokat,
- tájékoztatást ad az AI használatáról,
- megerősítést kér a folytatáshoz.

## Adatvédelmi tartalom

A tájékoztató kitér:

- AI alapú képfeldolgozásra,
- automatikus anonimizálásra,
- emberi felügyeletre,
- adatkezelési célokra.

## Kimenetek

| Ág | Funkció |
|---|---|
| Accept | Workflow folytatása |
| Decline | Folyamat vége |

---

# 2. Képfeltöltés

## Node
`Képfeltöltés`

## Funkció

A felhasználó:

- feltölti a hibáról készült képet,
- opcionálisan megadhat helyszínt.

## Mentett változók

| Változó | Leírás |
|---|---|
| `user_photo_url` | Feltöltött kép URL |
| `user_photo_location` | Helyszín |

---

# 3. AI képelemzés

## Node
`Extract Content from Image`

## Funkció

A rendszer AI Vision modellt használ:

- objektumfelismerésre,
- közterületi hibák azonosítására,
- vizuális moderációra.

## Példák detektált elemekre

- kátyú,
- illegális hulladék,
- sérült infrastruktúra,
- utcai táborozás,
- hajléktalan személy,
- közterületi probléma.

---

# 4. Hajléktalan detektálás

## Node
`workflowhajlektalan`

## Funkció

A workflow elemzi:

- található-e hajléktalan személy,
- utcai élethelyzet,
- közterületi alvás,
- takaró,
- karton,
- ideiglenes fekhely.

---

# AI Elemzés eredménye

A workflow több AI folyamatot futtat párhuzamosan:

- hajléktalan detektálás,
- automatikus anonimizálás,
- censoring / blurring detection.

## Strukturált AI válasz

```json
{
  "homeless_detected": boolean,
  "confidence": 0-100,
  "reason": "Short factual explanation.",

  "censor_required": boolean,
  "censor_confidence": 0-100,
  "censor_reason": "Short factual explanation.",

  "detected_censor_elements": [
    "face",
    "license_plate"
  ]
}
```

---

# Hajléktalan detektálás

## A rendszer pozitív találatként kezelheti:

- utcán alvó személy,
- kartonon fekvő ember,
- kültéri fekhely,
- takaróval fedett személy,
- ideiglenes táborozás,
- életvitelszerű közterületi tartózkodás.

---

# Blurring / Censor Detection

## A rendszer automatikusan felismerheti:

- emberi arcokat,
- rendszámokat,
- érzékeny vizuális elemeket.

## Cél

- GDPR kompatibilitás,
- személyes adatok védelme,
- moderáció támogatása,
- biztonságos publikáció.

---

# 5. Döntési logika

## Node
`hajlektalan`

## Funkció

A chatbot eldönti:

- normál hibabejelentés történt-e,
- vagy szociális problémát tartalmaz a kép.

---

# 5.1 Pozitív hajléktalan találat

## Node
`Valószínűleg hajléktalan van a képen`

## Funkció

A chatbot figyelmezteti a felhasználót, hogy:

- a platform nem hajléktalan-bejelentő rendszer,
- a feltöltött kép szociális problémát tartalmazhat.

## Kimenetek

| Ág | Funkció |
|---|---|
| Accept | Tovább a bejelentéshez |
| Decline | Workflow vége |

---

# 5.2 Negatív hajléktalan találat

## Node
`Hajléktalan detektálása sikertelen`

Normál hibabejelentési workflow folytatása.

---

# 6. Leírás megadása

## Node
`Leírás_megadása`

## Funkció

A felhasználó szövegesen megadja:

- a probléma leírását,
- a hiba részleteit.

## Mentett változó

| Változó | Leírás |
|---|---|
| `user_description` | Felhasználói leírás |

---

# 7. Bejelentés véglegesítése

## Node
`Standard6`

## Funkció

A chatbot:

- összegzi a bejelentést,
- megjeleníti:
  - helyszínt,
  - képet,
  - leírást,
- megerősítést kér.

## Kimenetek

| Ág | Funkció |
|---|---|
| Accept | Beküldés |
| Decline | Módosítás |

---

# 8. Sikeres beküldés

## Node
`Standard7`

## Funkció

A rendszer visszajelzést ad:

```text
Sikeres beküldés!
```

---

# Workflow változók

| Változó | Típus | Funkció |
|---|---|---|
| `user_photo_url` | string | Feltöltött kép URL |
| `user_photo_location` | string | Helyszín |
| `workflowhajlektalan` | JSON | AI elemzés eredménye |
| `user_description` | string | Felhasználói leírás |

---

# Technológiai stack

| Technológia | Funkció |
|---|---|
| Botpress | Workflow engine |
| OpenAI Vision | Képelemzés |
| HTML/CSS | Frontend |
| GitHub Pages | Hosting |
| JavaScript | Embed logika |

---

# Összefoglalás

A rendszer egy modern, AI-alapú közterületi hibabejelentő chatbot, amely:

- automatizált képfeldolgozást használ,
- támogatja az intelligens moderációt,
- képes érzékeny tartalmak szűrésére,
- automatikus anonimizálást végez,
- és felhasználóbarát módon támogatja a hibabejelentési folyamatot.

Sikeres beküldés!

A workflow itt lezárul.


