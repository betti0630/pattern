# Nagy mellre alakítás Seamly2D-ben — lépésről lépésre

Ez az útmutató azt a folyamatot foglalja össze, amit a Blúz_alap_Betti_init.sm2d mintán végigcsináltunk. A lépések sorrendben követik egymást, mert mindegyik az előzőre épül. A képletekben szereplő pontneveket (`J`, `F`, `K` stb.) minden mintában a saját rajzod szerint kell azonosítani — a *logika* az, ami újrafelhasználható, nem a pontos nevek.

---

## 0. Miért kell egyáltalán hozzányúlni az alapmintához?

A legtöbb magyar szabás-könyv (Cser Ferencné, és a hozzá hasonló rendszerek) a mellformázást **kizárólag a mellbőségből (mb)** számolt, arányos képlettel adja meg (pl. `mb/10*3-7`). Ez egy "átlagos" alkatra jó közelítés, de ha:

- a melled aránytalanul nagy a többi méretedhez képest,
- vagy a mellcsúcsod nem a hónaljvonal magasságában van (ahogy a könyv feltételezi),

...akkor ez a képlet **nem elég**, és a mintát egyénileg kell korrigálni. Az alábbi lépések pontosan ezt csinálják, mérésekre alapozva, nem találgatással.

---

## 1. Szükséges mérések (a `.smis` fájlban legyenek meg)

| Mérték neve (Seamly2D) | Mit jelent |
|---|---|
| `bust_circ` | Mellbőség (teljes körméret, a legdomborúbb ponton) |
| `highbust_circ` | Felsőmell-bőség (a mell fölött mérve) |
| `bust_arc_f` | Mellbőség **elölről**, a legszélesebb ponton (oldaltól oldalig, elöl) |
| `waist_circ`, `hip_circ` | Derék- és csípőbőség |
| `bustpoint_to_bustpoint` | A két mellcsúcs közti távolság |
| `bustpoint_to_neck_side` | Mellcsúcstól a nyak oldaláig |
| **Mellmélység hónalj alatt** | Saját mérés: mennyivel van a mellcsúcsod lejjebb a hónaljvonaladnál (nem szabvány Seamly2D mérték, ezt neked kell megmérned és felvenned egy változóba) |

**Fontos csapda:** ellenőrizd, hogy az `armfold_to_armfold_f` (karhajtástól karhajtásig, elöl) méretedet **hol** vetted fel. Ha a mell *fölött* mérted, az nem tükrözi a valós mellbőséget — ilyenkor helyette a `bust_arc_f`-et (a legszélesebb ponton mért érték) kell használni az elejeszélességhez.

---

## 2. A valódi mellcsúcs pozicionálása (`MP` pont)

A legtöbb rendszer a mellkivétet a **hónaljvonal magasságában** szerkeszti, feltételezve, hogy ott van a mellcsúcs. Ha ez nálad nem így van:

1. Vedd fel egy változóba a hónaljvonal és a valódi mellcsúcs közti **függőleges** távolságot:
   ```
   #mellmelyseg = 12   (a te mért értéked)
   ```
2. Hozz létre egy új pontot, pontosan ennyivel lejjebb a mellcsúcs-oszlopon (a mellponttól — pl. `J` — függőlegesen lefelé):
   ```
   MP: basePoint=J, angle=270, length=#mellmelyseg, type=endLine
   ```
3. **Irányíts át rá mindent, ami ténylegesen a mell 3D-s formázásáról szól** — de csak azt:
   - a mellkivét célpontját kereső ív/vonal (amivel a vállkivét vagy dart-szár hosszát számolod)
   - a mellkivét szögét/hosszát meghatározó képletek
   - **ne** irányítsd át rá az összes egyéb, csak referenciaként használt pontot (pl. a derékkivét egyszerű helyzeti pontjait) — ott elég, ha a forma *arányosan* lejjebb kerül, nem kell szó szerint a valódi mellponthoz kötni. (Ez volt az egyik leggyakoribb hibaforrásunk: túl sok mindent kötöttünk `MP`-hez, ami feleslegesen bonyolította és el is rontotta a formát.)

---

## 3. A mellkivét mélységének korrekciója

Ha van a mintában egy pont (pl. a nyak oldalán), ahonnan a mellkivét szára indul, és van egy másik pont, ami a mellcsúcs felé mutat:

```
#fba = bustpoint_to_neck_side - Line_<nyakoldali_pont>_MP
#mfm = #mfm_alap + #fba
```

Vagyis: a rajzon **jelenleg meglévő** nyakoldal–mellcsúcs távolságot kivonjuk a **ténylegesen mért** értékből, és a különbséget hozzáadjuk a könyv szerinti alapmélységhez. Ez azt jelenti, hogy Seamly2D **magától** kiszámolja a hiányzó/felesleges mennyiséget, valahányszor megnyitod a fájlt — nem kell újra és újra kézzel mérned.

**Ha a próba még mindig nem elég/túl sok:** adj hozzá (vagy vonj le) egy kézzel hangolt állandót:
```
#mfm = #mfm_alap + #fba + 8   (a próbán bevált plusz érték)
```

---

## 3/b. A kivét vége ne érjen egészen a mellcsúcsig!

Ez egy klasszikus szabás-alapszabály, amit könnyű elfelejteni, miután pontosan pozicionáltuk a valódi mellpontot (`MP`): **a mellkivét hegye SOHA nem érhet egészen a mellbimbóig/mellcsúcsig.** Ha odáig varrod, a kivét ott mindig kúpos, kellemetlen, "hegyes" hatást ad — ez teljesen független az alkattól, ez alapvető szabásvarrási elv.

**A hiba, amibe mi is belefutottunk:** miután `MP`-t helyesen pozicionáltuk, a kivét látható vonalait (a két "szárat", amik a vállról/oldalról a mellponthoz futnak) egyenesen `MP`-ig húztuk. Ezt próbadarabon látni lehet: a varrás pont a mellbimbóra megy rá.

**A helyes megoldás — FONTOS: ne magát `MP`-t told el!** `MP` a valódi mellcsúcs, és minden *méretezési* számításnak (mennyi anyagra van szükség a kivétben, mekkora szöget zárjon be) továbbra is erre kell épülnie. Csak azt a pontot told el, ahol a **látható varrásvonal ténylegesen véget ér**:

```
#dart_vege_tavolsag = 2.5   (mennyivel álljon meg MP előtt — 2-3 cm szokásos)
MP_vege: basePoint=MP, angle=90, length=#dart_vege_tavolsag, type=endLine
```

Majd a kivét látható vonalait (amik korábban `MP`-ig futottak) irányítsd át `MP_vege`-re — de a méretezést végző képletek (pl. az `#fba`, vagy bármi, ami `Line_valami_MP`-t használ a szükséges bőség kiszámításához) **maradjanak `MP`-n**, ne `MP_vege`-n.

**Miért fontos ez a kettéválasztás?** Mert két különböző dolgot kell kiszolgálni:
- **"Mennyi anyagra van szükség?"** → ehhez a *valódi* mellcsúcs távolsága kell (`MP`)
- **"Hol érjen véget ténylegesen a varrás?"** → ez mindig egy kicsit *rövidebb*, mint a valós távolság

Ha összekevered a kettőt (pl. `MP`-t magát tolod el "biztonságból" feljebb), akkor a méretezés lesz pontatlan — a kivét vagy túl sekély, vagy túl mély lesz máshol.

---

## 4. Az elejeszélesség korrekciója — és amit a próba tanított

**Első próbálkozás (tévút, ne csináld ezt):** ha az elejeszélesség egy "mell fölötti" mértékből (pl. `armfold_to_armfold_f`) számolódik, csábító azt gondolni, hogy elég lecserélni a legszélesebb ponton mért körméretre:

```
#esz = bust_arc_f/2     ← EZ TÚL SOK LESZ, ne így csináld
```

Ez **logikusan jó ötletnek tűnik, de a gyakorlatban túl sokat ad hozzá.** Az ok: a `bust_arc_f` egy **feszes** körméret a legdomborúbb ponton — de a mellkivét (dart) **már önmagában is helyet csinál** a mell domborulatának, mert zárásakor a sík anyag 3D-sen kidomborodik. Ha a sík szélességet IS a feszes körméretre állítod, ÉS a kivét is megvan, a kettő összeadva **túl sok anyagot ad** — pont ez történt nálunk is: az első próbadarab összességében túl bő lett, nem feszült sehol.

**Ami tényleg működött: a próbadarabon végzett "összefogás-teszt".**

Vedd fel a próbadarabot, és **fogd össze kézzel a felesleges anyagot** pontosan a mell legszélesebb pontján, elöl. Mérd meg, mennyi anyagot tudsz így "kivenni" (ez lesz a felesleg, amit a mintából le kell vágni). Ez a szám **sokkal megbízhatóbb**, mint bármelyik mérésből számolt becslés, mert:
- a valós testre, a valós anyagra, a már meglévő kivétekkel együtt méred — nem elméleti számítást csinálsz
- nem kell találgatni, mennyit "vesz el" a kivét a feszes körméretből

**Hogyan építsd be a mintába:** ha a próbán elöl összesen X cm-t tudtál összefogni (a teljes eleje szélességén, mindkét oldalon együtt), és a mintád egy félmintát rajzol (csak az egyik oldalt), akkor a felére van szükséged korrekcióként:

```
#esz = <az eredeti, első becslésed> - X/2
```

Nálunk ez végül **majdnem pontosan visszaadta az eredeti, "mell fölötti" mérésből számolt értéket** — vagyis kiderült, hogy az az érték, amit az elején "rossz helyen mértnek" gondoltunk, nem is állt olyan messze a valóságtól. **A tanulság:** egy mérésből számolt becslés mindig csak egy kiindulópont — a próbadarabon végzett közvetlen mérés az, ami ténylegesen dönt.

---

## 5. A derékbőség tényleges elérése

Ez a legfontosabb, gyakran kihagyott lépés: **a legtöbb alapminta a derékkivéteket fix, könyv szerinti centiméterekben adja meg, függetlenül attól, mekkora a te valós derékbőséged.** Ha az elejeszélesség nő, a kivétek mérete nem nő automatikusan vele — a derék egyszerűen szélesebb lesz.

```
#derek_ob = 5                                  (öltöztetési bőség, ízlés szerint)
#derek_cel = #db + #derek_ob                   (db = derékbőség fele)
#mellvonal_szelesseg = #hsz+#hsz_öb+#hósz+#hósz_öb+#esz+#esz_öb
#derek_kivet_osszesen = #mellvonal_szelesseg - #derek_cel
```

Ez adja meg, **összesen mennyi kivétre van szükség** a derékvonalon ahhoz, hogy a végeredmény tényleg a te derékbőségedre jöjjön ki — nem többre, nem kevesebbre.

**Oszd szét ezt a mennyiséget több helyre**, ha egy kivétben túl sok lenne (egy kivét ökölszabályként ne legyen 4 cm-nél szélesebb, mert varráskor csúnya, éles törést ad):

```
#oldal_kivet = 4                               (oldalvarrás)
#hat_kivet = 2.5                                (hátkivét, változatlan)
#eleje_kivet_osszesen = #derek_kivet_osszesen - #oldal_kivet - #hat_kivet
#eleje_szar = #eleje_kivet_osszesen / 4         (2 elejei kivét x 2 szár)
```

Ha egy kivét mérete így is irreálisan nagy (pl. 10+ cm), az annak a jele, hogy **valamelyik alapmérték hibás** (pl. rosszul mért `bust_arc_f`), nem annak, hogy több/más elosztás kell — érdemes visszamenni és ellenőrizni a méréseket.

---

## 6. A csípő (opcionális, de érdemes átgondolni)

Ugyanez a probléma jelentkezik csípőnél is, ha a kivétek egy pontban összefutnak a derék alatt, majd onnantól a szabásvonal visszaugrik a teljes (csípőnél túl nagy) szélességre.

**Megoldás: ne engedd, hogy a kivét szárai egy pontban záródjanak.** Ehelyett:

1. Keresd meg a csípővonal-magasságú pontot a kivét oszlopában (pl. a derékponttól induló, csípő-magasságú metszéspont).
2. Ebből a pontból indítva, **tisztán vízszintesen** (ne "átlós" `alongLine`-nal egy másik, más magasságú ponthoz!) helyezz el két pontot, a derékszárnál kisebb távolságra:
   ```
   A12a: basePoint=<csípő-közép>, angle=180, length=#eleje_szar/2, type=endLine
   A12b: basePoint=<csípő-közép>, angle=0,   length=#eleje_szar/2, type=endLine
   ```
3. Kösd össze a derékszárat (`A13`) ezzel az új ponttal — így a kivét fokozatosan, nem hirtelen keskenyedik el csípőig.

**Csapda, amibe mi is belefutottunk:** ha a "vízszintes" pontot `alongLine`-nal, egy MÁSIK (nem azonos magasságú) pont felé méred, a pont **nem marad pontosan a csípővonalon** — enyhén elcsúszik felfelé/lefelé is. Mindig `angle=0`/`180` + `endLine` típust használj, ha garantáltan egy adott vízszintesen akarsz maradni.

---

## 7. A kivét formájának finomítása (esztétikai lépés)

Ha a kivét "háza" túl meredek/hegyes a tetején, vagy két kivét csúcsa nem egy magasságban van:

- **Ne kösd a kivét minden egyes pontját a valódi mellponthoz (`MP`)** — csak a csúcsot (a kivét legfelső pontját), egy kis állandó távolsággal (pl. 3 cm).
- **A kivét legszélesebb pontja (ahol a derékszárak vannak) lehet egy külön pont**, nem feltétlenül ugyanaz, mint a derékvonal referenciapontja:
  ```
  K_kozep: basePoint=K, angle=270, length=#delta_kozep, type=endLine
  ```
  Így a `K` pont megmaradhat "tiszta" referenciapontnak (pl. igazodva a hát derékvonalához), miközben a kivét saját formája ettől függetlenül, lejjebb, szabadon alakítható.
- **Ha két kivét csúcsát egy magasságba akarod hozni**, és a kivétek különböző oszlopokban vannak: hozz létre mindkét oszlopban egy-egy "azonos magasságú" segédpontot (pl. mindkettőt a mellponttól `#mellmelyseg`-nyire lefelé vetítve), és **ne** kösd mindkettőt szó szerint ugyanahhoz az egy ponthoz, ha azok különböző oszlopban vannak — az átlós vonalakat és kereszteződést okoz.

---

## 8. Ellenőrzés és karbantartás

- **Minden szerkesztés után nyisd meg Seamly2D-ben**, és nézd meg, nem dob-e hibát ("Can't find intersection point"). Ha igen, valamelyik ív/kör sugara nem ér össze — általában elég azt az egy értéket módosítani.
- **Kerüld a `Line_A_B` hivatkozást olyan pontnál, ami saját maga A vagy B** — ez körkörös hivatkozást okozhat, és hibát dob. Ilyenkor helyette egyszerű, már bevált változókkal számolj.
- **Időnként nézd át, nincs-e a fájlban használaton kívüli pont vagy változó** (amit egy korábbi próbálkozás során hoztál létre, aztán máshogy oldottad meg, de elfelejtetted törölni). Ez nem hibás, csak felesleges — kereshető úgy, hogy minden pont/változó nevére rákeresel, hivatkozik-e rá bármi más.
- **A próbadarab az igazi teszt.** A papíron/képernyőn jól kinéző arányok nem helyettesítik a testen való ellenőrzést — a képletekben szereplő "kézzel hangolt" számok (pl. a mellkivét +8 cm-es korrekciója, vagy a `#delta_kozep` értéke) pontosan azért vannak külön változóban, hogy próba után könnyen finomíthatók legyenek.
- **Bármelyik szélesség-korrekciónál (nem csak a mellnél) a próbán végzett "összefogás-teszt" megbízhatóbb, mint a mérésekből számolt becslés.** Ha egy rész túl bőnek vagy túl szűknek tűnik, ne találgass újabb képletet — vedd fel a próbadarabot, fogd össze/engedd ki kézzel a problémás részt, mérd meg a különbséget, és ezt az egy számot építsd be a mintába. Ez gyorsabb és pontosabb is, mint bármennyi elméleti számolás.

---

## Gyors összefoglaló – a lépések sorrendje

1. Mérd fel pontosan: `bust_circ`, `bust_arc_f`, `highbust_circ`, a saját mellmélységedet a hónaljvonal alatt
2. Hozd létre a valódi mellcsúcs-pontot (`MP`), és **csak** a mell 3D-formázásáért felelős elemeket kösd hozzá
3. Korrigáld a mellkivét mélységét mérésalapú képlettel
4. **A kivét látható vége álljon meg 2-3 cm-rel `MP` előtt** — ne magát `MP`-t told el, csak a varrásvonal végpontját
5. Cseréld le az elejeszélességet a helyesen (legszélesebb ponton) mért értékre
6. Számold ki a ténylegesen szükséges derékkivétet, és oszd szét több helyre
7. Ha kell, oldd meg a csípő fokozatos elkeskenyedését is (ne engedd, hogy egy pontban záródjon)
8. Finomítsd a kivétek formáját (csúcs, legszélesebb pont) esztétikai szempontból
9. Próbadarab, majd finomhangolás a kézzel beállítható változókon keresztül
