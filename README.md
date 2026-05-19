# Járókelő Chatbot Projekt

Ez a projekt a Járókelő hibabejelentő platform két chatbotos felületét tartalmazza: egy belsős ügyintézői modult (adminisztrációs asszisztens) és egy külsős bejelentői modult.

## Belső chatbot (Járókelő Adminisztrációs Asszisztens)

A belsős chatbot célja az adminisztratív és ügykezelési feladatok és a hibabejelentések háttérmunkájának megkönnyítése mesterséges intelligencia segítségével.

### A bot működése

1. **Geocode Node (Adatbázis koordinátáinak kiegészítése)**
   Amennyiben az adatbázisban (`jarokelo_extractTable`) szereplő ügyekhez tartozó `lat` (szélesség) és `lng` (hosszúság) oszlopok üresek, a bot végigfut ezeken a sorokon. A Nominatim API segítségével megpróbálja utólag geokódolni és frissíteni a rögzített esetek földrajzi koordinátáit.

2. **Ügy bekérése**
   A bot bekér egy URL-t a felhasználótól (az ügyintézőtől), amely a feldolgozni kívánt ügyre mutat.

3. **Adatkinyerés**
   A megadott URL alapján a bot feldolgozza az esetet, és kinyeri a szükséges információkat:
   - Város
   - Kerület
   - Cím
   - Kategória
   - Egy rövid összegzés az ügy leírásából.

4. **Tudásbázis kinyerés és szűrés**
   A rendszer az adatbázist leszűri azokra az esetekre, amelyek **ugyanabban a kerületben** találhatóak és **ugyanabba a kategóriába** tartoznak, mint a vizsgált ügy.
   Ennek alapján a mesterséges intelligencia megteszi a predikcióját, amelyben meghatározza, hogy az adott ügy **kihez tartozhat, ki a felelős és kinek kellene bejelenteni**.

5. **Lokáció vizsgálat (Térbeli közelség)**
   A korábban geokódolt adatok (pontos koordináták) alapján a bot megkeresi az 5 legközelebbi korábbi ügyet. Extra szűrésként a bot végül csak azokat az ügyeket adja vissza és jeleníti meg, amelyek a vizsgált ügytől **ténylegesen 50 méteren belül** helyezkednek el.

## Külső chatbot (Járókelő Bejelentő Asszisztens)

*A külső chatbot dokumentációjának helye. Folyamatban...*
