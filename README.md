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

## Külső chatbot (Járókelő Bejelentő Asszisztens)

*A külső chatbot dokumentációjának helye. Folyamatban...*
