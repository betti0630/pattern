# Blúz teljes szerkesztése — a könyv menete + a nagy mellre történő saját módosításaink

Ez a leírás a könyv ("Női blúz alapszerkesztése és modellezése mellformázóval", 7.2. fejezet, 86–89. oldal) szerkesztési lépéseit követi, **pontról pontra**, ugyanazokkal a betűjelekkel, amiket a Seamly2D fájlod is használ (`A`, `B`, `C`... `M`, `N`, `J`, `K` stb.). Minden lépésnél jelölöm, hogy **megtartottuk-e változatlanul**, vagy **módosítottuk-e** — és ha igen, miért és hogyan.

> A képletekben szereplő rövidítések: `mb` = mellbőség fele, `db` = derékbőség fele, `csb` = csípőbőség fele, `hm` = hónaljmélység, `hdh` = hátaderékhossz, `cm` = csípőmélység, `hsz`/`hósz`/`esz` = hát-/hónalj-/elejeszélesség, `nysz` = nyakszélesség, `hnyt` = háta-nyaktőmagasság, `edh` = elejederékhossz, `vsz` = vállszélesség.

---

## 0. Alapméretek

A könyv a szerkesztést egy átlagos alkatra adja meg, majd a saját méreteinkkel helyettesítjük be. A Betti-féle méretek:

| Mérték | Érték |
|---|---|
| Mellbőség (teljes) | 102 cm → `mb` = 51 |
| Derékbőség (teljes) | 85 cm → `db` = 42,5 |
| Csípőbőség (teljes) | 100 cm → `csb` = 50 |
| Mellbőség elöl, legszélesebb ponton (`bust_arc_f`) | 60 cm |
| Felsőmell-bőség (`highbust_circ`) | 90 cm |
| Mellponttól a nyak oldaláig (`bustpoint_to_neck_side`) | mért érték |
| Mell mélysége a hónaljvonal alatt (saját mérés) | **4,5 cm** *(a próba után korrigálva, eredetileg 12 cm-nek gondoltuk)* |

**Ez az a pont, ahol a mi utunk elágazik a könyvtől:** a könyv a mellcsúcsot mindig a hónaljvonal magasságában feltételezi, és a mellkivét mélységét egyetlen, `mb`-ből számolt képlettel adja meg. Nálad ez a feltételezés nem állta meg a helyét — ezért vezettük be a saját mérésen alapuló korrekciókat, amik lentebb, a megfelelő lépéseknél jelennek meg.

---

## 1. Blúzalapvonalak szerkesztése

A csomagolópapír jobb szélén kijelöljük a hátközépvonalat (`A` pont), és lefelé haladva:

| Lépés | Könyv szerinti képlet | **MEGTARTVA / MÓDOSÍTVA** |
|---|---|---|
| `A–B` (hónaljmélység) | `hm = tm/10+mb/10*2,5-8,4` + öltöztetési bőség | **MEGTARTVA** — nem nyúltunk hozzá |
| `A–C` (hátaderékhossz) | `hdh = tm/10*2+mb/10+2` | **MEGTARTVA** |
| `C–D` (csípőmélység) | `cm = tm/10+csb/10-2` | **MEGTARTVA** |
| `C–C1` (hátközépvonal beállítása derékban) | 1,5 cm | **MEGTARTVA** |
| `B1–E` (hátszélesség) | `hsz = mb/10*2,5+5,5` + öb | **MEGTARTVA** (nálad már korábban is mért értékből jött) |
| `E–F` (hónaljszélesség) | `hósz = mb/10*2,5-1,5` + öb | **MEGTARTVA** |
| `F–G` (elejeszélesség) | `esz = mb/10*5-4` + öb | **MÓDOSÍTVA — ez a legnagyobb beavatkozásunk, lásd 3. pont** |
| `F–H` (oldalvonal-elhelyezés) | `F–E/2-1` | **MEGTARTVA** (élő képlet, automatikusan követi, ha `E`/`F` mozdul) |
| `A–I` (hátanyakszélesség) | `mb/10+2` | **MEGTARTVA** |
| `G–J` (melltávolság az elejeközépvonaltól) | `mb/10*2+0,5` | **MEGTARTVA** |

---

## 2. A blúz háta- és elejerészének szerkesztése

### 2.1 Hátrész (nyak, váll, hátkivét)

| Lépés | Könyv szerint | **Nálunk** |
|---|---|---|
| `N–N1` (háta váll-lejtés) | 0,5 cm | **MEGTARTVA** |
| `I–N2` (hátavállszélesség) | testméret+szélesítés+betartás | **MEGTARTVA** |
| `E–E1` (karöltősegédpont magassága) | 5 cm | **MEGTARTVA** |
| Hátkivét (a derékvonalon, hátközépnél) | fix 2,5 cm-es kivét | **MEGTARTVA, de a végén hozzákötve egy változóhoz** (`#hat_kivet`), hogy a könyveléssel konzisztens legyen |

### 2.2 A valódi mellcsúcs pozicionálása — **ÚJ LÉPÉS, ami nincs a könyvben**

A könyv a mellcsúcsot a hónaljvonal magasságában kezeli. Mi bevezettünk egy **külön pontot (`MP`)**, ami a te ténylegesen mért mellmélységed alapján a hónaljvonaltól lejjebb helyezkedik el:

```
#mellmelyseg = 4,5   (saját mérés, a próba után korrigálva)
MP: J ponttól függőlegesen lefelé, #mellmelyseg távolságra
```

Ez a pont lett az **igazi célpontja** mindennek, ami a mell 3D-s formázásáról szól (lásd 2.3 pont) — de **nem** lett minden egyes, csak referenciaként használt pont (pl. a derékkivét) alapja, mert azt feleslegesen bonyolította volna.

**Fontos kiegészítés a próba után:** a mellkivét **látható vége** nem érhet egészen `MP`-ig (klasszikus szabás-szabály, hogy ne legyen kúpos a varrás a mellbimbón). Ezért egy második pontot (`MP_vege`) is bevezettünk, `MP`-től 2,5 cm-rel feljebb, és a kivét látható vonalai eddig futnak, nem `MP`-ig.

### 2.3 A mellkivét (mellformázó varrás)

| Lépés | Könyv szerint | **Nálunk** |
|---|---|---|
| Mellformázó mélysége köríven (`L1–L2`) | `mb/10*3-7` | **MÓDOSÍTVA**: `#mfm = #mfm_alap + #fba`, ahol `#fba` a te tényleges `bustpoint_to_neck_side` mértéked és a rajzon jelenleg meglévő nyakoldal–mellcsúcs (`M1`–`MP`) távolság különbsége. Így a kivét mélysége a te valós alkatodhoz igazodik, nem egy átlagos arányhoz. |
| Mellformázóhossz átmérése (`J–L1=J–L2`) | segédívvel | **MEGTARTVA**, de a kört meghosszabbítottuk `#mellmelyseg`-gel, mert a célpont (`MP`) lejjebb került |
| Vállszélesség (`M1–L1+L2–N4`) | mért érték | **MEGTARTVA**, a szög/hossz számítás automatikusan követi a fenti változásokat |
| `F–F1` (elejekaröltő-segédpont) | 5 cm | **MEGTARTVA** |
| `F–F2` (illesztési pont magassága) | `mb/10-1` | **MEGTARTVA** |

---

## 3. Az elejeszélesség — a legnagyobb korrekció

Ez volt a legtöbb próbálkozást igénylő pont, ezért külön részletezem, hogyan jutottunk el a végleges megoldásig.

1. **Első próbálkozás (tévút):** a könyv `esz` képlete helyett a mell legszélesebb pontján mért, feszes körméretet (`bust_arc_f/2`) használtuk közvetlenül. Ez **túl sok** lett — a próbadarab összességében túl bőre sikeredett, mert a mellkivét már önmagában is helyet ad a domborulatnak, nem kell ráadásul a teljes feszes körméretet is a sík anyagba beleszámolni.
2. **A working megoldás:** a próbadarabon **ténylegesen összefogtuk kézzel** a felesleges anyagot a mell legszélesebb pontján, és megmértük — ez adta a pontos korrekciót:
   ```
   #esz = bust_arc_f/2 - 8,5
   ```
   Érdekesség: ez a végeredmény nagyon közel esett az eredeti, "mell fölötti" méréshez — vagyis a kiindulási mérésünk nem is volt olyan rossz, csak a `bust_arc_f`-fel való helyettesítés vitte túlzásba a bővítést.

---

## 4. A derékbőség tényleges elérése — **ÚJ LÉPÉS, ami nincs a könyvben**

A könyv a derékkivéteket fix centiméterekben adja meg (átlagos alkatra). Mi bevezettünk egy **célzott, mérésalapú rendszert**:

```
#derek_ob = 5                                        (öltöztetési bőség)
#derek_cel = #db + #derek_ob                         (a te derékbőséged + bőség)
#mellvonal_szelesseg = hsz+hsz_öb+hósz+hósz_öb+esz+esz_öb
#derek_kivet_osszesen = #mellvonal_szelesseg - #derek_cel
```

Ez adja meg, **összesen mennyi kivétre van szükség** a derékvonalon, hogy a végeredmény tényleg a te derékbőségedre jöjjön ki. Ezt osztottuk szét:

```
#oldal_kivet = 2                                     (oldalvarrás — visszaállítva a könyv szerinti értékre)
#hat_kivet = 2,5                                      (hátkivét, változatlan)
#eleje_kivet_osszesen = #derek_kivet_osszesen - #oldal_kivet - #hat_kivet
#eleje_szar = #eleje_kivet_osszesen / 2               (egyetlen elejei kivét, két szár)
```

**Fontos fejlődéstörténet:** útközben **két** elejei kivétet is kipróbáltunk (mert a korábbi, túl nagy `#esz` miatt egy kivétbe nem fért volna bele ésszerűen a szükséges mennyiség). Miután az `#esz`-t és a `#mellmelyseg`-et is pontosítottuk a próbák alapján, a szükséges kivét lecsökkent annyira, hogy **ismét elég egyetlen elejei kivét** — ezért végül visszatértünk az eredeti, egykivétes elrendezéshez, csak a méretét igazítottuk a valós derékbőségedhez.

**A kivét formája:** a kivét teteje (`A15`) nem a derékvonal-referenciaponthoz (`K`), hanem külön, a `K` alatt elhelyezkedő ponthoz (`K_kozep`) van kötve — ez adja a klasszikus, rombusz-szerű formát, ahelyett hogy a csúcs túl meredek lenne.

**A kivét alja:** ahelyett, hogy a kivét egy pontban záródna a derék alatt, majd a szabásvonal visszaugrana a teljes szélességre, a kivét alja **egyetlen pontba fut össze, kb. 4 cm-rel a csípővonal fölött** — ugyanolyan mélyen, mint a hátkivét. Ez biztosítja, hogy a derék és a csípő között ne legyen indokolatlan visszaugrás a szélességben.

---

## 5. Az ujj szerkesztése (7.2.4. alapján)

| Lépés | Könyv szerint | **Nálunk** |
|---|---|---|
| Karöltőméretek (`A–D`, `A–E` stb.) | a blúz karöltő-méreteiből számolva | **MEGTARTVA, élő hivatkozással** — az ujjív két fő szegmense (`Spl_eip_H_1` és `Spl_H_E2+0,5`) automatikusan az aktuális, módosított karkivágás-görbe hosszát veszi át, nem egy rögzített számot |
| `B–B1`, `D–D1` (illesztési pontok) | `mb/10-1`, `mb/10-1,5` | **MEGTARTVA, generikus képlettel** — ez a rész még mindig a könyv szerinti, átlagos alkatra szabott arányt használja. Ezt egyelőre nem korrigáltuk, mert a hossz már jól illeszkedik; ha a bevarrt ujjnál gyűrődés vagy húzás jelentkezik, ez az a pont, amit utólag célzottan finomítani érdemes. |

**A `Piece` (végleges szabásvonal) az ujjnál még nincs összeállítva** — ez a következő lépés, miután a próbaujjat megvarrtad és ellenőrizted.

---

## 6. Blúzalapminta készítése — a végleges szabásvonalak

A könyv szerint az alapmintát a szerkesztési vonalak mentén másoljuk ki, és bejelöljük az illesztési pontokat. Nálunk ez Seamly2D-ben a **Piece (darab) eszközzel** történt: `Eleje` és `Hátulja` darabok, mindegyikben a megfelelő szerkesztési pontokra/görbékre hivatkozva. A kivétek (`Szűkítő 1`, hátkivét) külön `path`-ként vannak berajzolva a darabokon belül.

---

## Összefoglaló táblázat — minden változónk egy helyen

| Változó | Jelentés | Miért vezettük be |
|---|---|---|
| `#mellmelyseg` | Mellcsúcs mélysége a hónaljvonal alatt | A könyv feltételezésével ellentétben a mellcsúcs nem a hónaljvonalban van |
| `#dart_vege_tavolsag` | A mellkivét vége ennyivel áll meg `MP` előtt | Klasszikus szabás-szabály: a kivét ne érjen a mellbimbóig |
| `#mfm_alap`, `#fba`, `#mfm` | Mellformázó mélysége, mérésalapú korrekcióval | A könyv generikus `mb`-képlete nem vette figyelembe az egyéni mell-kiugrást |
| `#esz` | Elejeszélesség | A helyes, legszélesebb ponton mért + próbán korrigált érték |
| `#derek_ob`, `#derek_cel`, `#derek_kivet_osszesen` | Derékbőség célzott elérése | A könyv fix kivétei nem igazodtak a valós derékbőséghez |
| `#oldal_kivet`, `#hat_kivet`, `#eleje_kivet_osszesen`, `#eleje_szar` | A derékkivét szétosztása | Hogy egyetlen kivét ne legyen irreálisan nagy |
| `#delta_kozep` | A kivét legszélesebb pontjának mélysége `K` alatt | Klasszikus, nem túl meredek rombusz-forma eléréséhez |

---

*Ez a leírás a projektedhez csatolt könyv (7.2. fejezet) szerkesztési menetét követi; a képletek és lépések sorrendje onnan származik, a fenti táblázatokban jelölt módosítások a mi, a próbadarabok alapján végzett korrekcióink.*
