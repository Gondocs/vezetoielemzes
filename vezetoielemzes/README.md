# Vezetői készségteszt elemzés (Notebook)

Ez a mappa tartalmazza a Python alapú elemző notebookot (`elemzes.ipynb`), amely az önértékelési és a mások által adott vezetői készségteszt válaszokból részletes összehasonlító és diagnosztikai ábrákat, mutatókat készít.

## 1. Cél
A vezetői készségteszt (kitölthető: **https://vezetoikeszsegekteszt.vercel.app**) eredményeinek:
- Összesítése (főcsoportok, alcsoportok, finomkategóriák).
- Összevetése az önértékelés ("sajat") és a mások által adott értékelés ("rolam") között.
- Különbségek, korrelációk, eloszlások feltárása.
- Vizualizációk létrehozása publikálható PNG / SVG formátumban.

## 2. Adatgyűjtés folyamata
1. Nyisd meg az oldalt: `vezetoikeszsegekteszt.vercel.app`.
2. Töltsd ki a tesztet (Likert skála 1–6 pontokkal).
3. Az oldal végén kattints az **adatok exportálása CSV-re** (vagy hasonló) gombra.
4. A rendszer két CSV fájlt tölt le:
   - `pams_export_YYYYMMDD_valaszok.csv` – kérdésszintű válaszok (id, text, answer).
   - `pams_export_YYYYMMDD_eredmenyek.csv` – aggregált pontszámok és százalékok (score, max, pct, csoportok).
5. Ha saját kitöltésed, tedd a `adatok/sajat/` mappába.
6. Ha más töltötte ki rólad, tedd a `adatok/rolam/` mappába.
7. Az elemzés mindig párokat használ: ugyanazon dátum két fájlja mindkét mappából (ön vs. mások). Ha csak az egyik oldal elérhető, a vonatkozó összehasonlítások részlegesen futnak.

## 3. Mappastruktúra
```
vezetoielemzes/
  elemzes.ipynb        # Fő notebook: betöltés + vizualizációk
  adatok/
    sajat/             # Saját kitöltések CSV fájljai
    rolam/             # Mások által rólam kitöltött CSV fájlok
  figures/             # Generált ábrák (PNG + SVG)
```

## 4. CSV fájlok szerkezete
`*_valaszok.csv` oszlopok:
- `id`: Kérdés azonosító (egész szám, összefügg a tartományokkal pl. 1–23).
- `text`: A kérdés szövege.
- `answer`: A Likert válasz (1–6), itt Pythonban `answer_val` névre átnevezve.

`*_eredmenyek.csv` oszlopok:
- `key`: Belső kulcs (pl. `I_total`, `II_hatalom_total`, `II_hatalomszerzes`).
- `label`: Megjelenítendő cím, gyakran tartományt tartalmaz pl. `Hatalomszerzés (33–37)`.
- `score`: Elért pontszám.
- `max`: Elérhető maximum pontszám.
- `pct`: Százalék (pl. `78%`). A notebookban normalizált numerikus: `pct_num` (0–100 float).
- `group`: Főcsoport neve.
- `subgroup`: Alcsoport (ha van). Üres, ha `key` teljes csoport-összegző vagy egy tartományos sor külön parent nélkül.

## 5. Fő feldolgozási lépések a notebookban
1. Könyvtárak importja (pandas, numpy, seaborn, matplotlib, scipy).
2. CSV fájlok betöltése mindkét mappából (ön / mások).
3. Százalékok (`pct`) tisztítása → `pct_num`.
4. Kérdés–kategória összerendelés: a `label` tartományok (pl. `(12–23)`) alapján minden kérdés azonosítása és legspecifikusabb kategóriájának kiválasztása.
5. Összeolvasztás: kérdésszintű válaszok + kategória mapping.
6. Derivált mutatók: `diff = others - self`, `abs_diff`.
7. Ábrák és táblák generálása.

## 6. Elkészülő ábrák és jelentésük
Számozás a notebookban (pontszám alapú sávdiagramok sávközépre írt értékekkel):

1. **Főcsoportok – pontszámok (Én vs. mások)**: Összpontszám összevetés a három fő készségterületen + TOTAL. Segít azonnal látni hol nagyobb / kisebb a külső értékelés.
2. **Főcsoportonként alcsoport-összegzések (pont)**: Minden főcsoport bontásban a hozzá tartozó alcsoportok (pl. Önismeret, Stressz kezelése, Problémamegoldás). Itt az alcsoportokat úgy gyűjtjük, hogy `_total` kulcs, vagy tartományos label parent nélkül is bekerül (pl. `Mások motiválása (41–49)`).
   - Radar változat: ugyanaz %-ban (0–100), gyors arány-összevetés.
3. **Finomkategóriák (pont)**: Alcsoport szint alatt (szülő `subgroup` alapján) a tartományos részek (pl. `Hatalomszerzés (33–37)`, `Befolyás gyakorlása (38–40)`). Részletes bontás, hol tér el leginkább.
4. **Top különbségek lollipop (összes kérdés)**: A legnagyobb abszolút eltérések kérdésszinten; a szóródás irányát szín jelzi.
5. **Top különbségek főcsoportonként**: Ugyanez csoportokra szűrve.
6. **Szórásdiagram (Én vs. mások)**: Kérdésszintű pontok regressziós illesztéssel; diagonálishoz képest torzulás vizsgálata. Pearson korreláció.
7. **Bland–Altman**: Átlag vs. különbség – torzítás és egyezési határok (LOA) megítélése.
8. **Eloszlások KDE**: Sűrűség a Likert skálán ön vs. mások. Tónus: rendszerszintű felfelé / lefelé tolódás.
9. **Különbségek box + pontok (Mások – Én)** főcsoportonként: Medián és szóródás; vizuális aszimmetriák.
10. **Táblázat – legnagyobb pozitív / negatív különbségek**: Konkrét kérdésszövegek felsorolása gyors áttekintéshez.

## 7. Futtatás
Ajánlott Python ≥ 3.10.

Telepítés (ha nincs virtuális környezet):
```bash
pip install pandas numpy matplotlib seaborn scipy
```

Notebook futtatása (VS Code vagy Jupyter):
1. Helyezd el a friss CSV-ket: `adatok/sajat/` és `adatok/rolam/` mappákba.
2. Nyisd meg: `vezetoielemzes/elemzes.ipynb`.
3. Futtasd sorban a cellákat (1 → végéig). A `figures/` mappába automatikusan menti a `.png` és `.svg` fájlokat.

## 8. Adatok frissítése
Új kitöltés esetén:
1. Töltsd le a két CSV-t (valaszok + eredmenyek).
2. Nevezés megtartható dátummal (nem kritikus, a notebook a pontos fájlneveket keresi jelenleg hard-code módon). Ha más dátumot használsz, módosítsd a notebook elején a fájlneveket.
3. Cseréld ki a régi fájlokat a megfelelő mappában.
4. Futtasd újra a notebookot → új ábrák.

## 9. Konfiguráció és testreszabás
- A sávdiagram címkék formátuma: `label_fmt="{:.0f}"` paraméter a `bar_compare` függvényben.
- Címkék kikapcsolása: `show_labels=False`.
- Sorrend fixálása: `order=[...]` átadás a `bar_compare` hívásnál.
- Pontszám vs. százalék: Sávdiagramok pontszámot (score), radar diagramok százalékot (`pct_num`) használnak.

## 10. Értelmezési megjegyzések
- A pozitív `diff` (Mások – Én) azt jelzi, hogy mások magasabbra értékelnek, mint az önértékelés.
- Nagy `abs_diff` fókuszterület lehet fejlődésre vagy visszajelzésre.
- Alacsony korreláció (Pearson r) esetén a belső és külső percepció kevésbé konzisztens.
- Bland–Altman határok szélessége mutatja az egyezés szóródását – minél szűkebb, annál egységesebb a megítélés.

## 11. Jövőbeli bővítési ötletek
- Automatikus dátumfelismerés a fájlnevekben.
- Több személy adatai batch feldolgozásban.
- Interaktív dashboard (Plotly / Streamlit / web átirat TypeScriptben).
- Normalizált pontszámok az alcsoportok kérdésszámának figyelembe vételével.

## 12. Hibakeresés
Ha egy ábra nem jelenik meg:
- Ellenőrizd, hogy mindkét (sajat / rolam) `*_eredmenyek.csv` fájl megvan.
- Ha új dátum, módosítsd a notebook elején a fájlneveket.
- Nézd meg a cella kimenetét: hiányzó könyvtár / import hiba esetén telepítsd a csomagot.

## 13. Licenc / Felhasználás
A belső elemző notebook, az eredményei és ábrái személyes / csapaton belüli visszajelzésre készültek. Publikálás előtt anonimizálás ajánlott.

---
Kérdések / módosítási igény esetén: frissítsd a notebookot vagy jelezd fejlesztési ötleteidet.
