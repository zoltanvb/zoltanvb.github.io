## ep128emu 2.0.12.x

Szia! Ez itt az ep128emu (egyik) fejlesztés alatti verziója, elsősorban a TVC256++ és 2DFX kártyák támogatásához. Jelenlegi verzió: 
| ep128emu verzió | TVC256++ firmware kompatibilitás | 2dfx firmware verzió |
|:--------:|:----:|:-----------:|
| 2.0.12.6 | 0.24 | (kísérleti) |

## Főbb TVC256++ emulációs hiányok
- csak 2 MB PSRAM van emulálva
- időzítés nem pontos (a műveletek rögtön végrehajtódnak, nincs várakozás)
- sprite ütközés a szélső keret alatt nincs jelezve
- sprite ütközés interrupt nem nagyon volt tesztelve, lehetnek gondok
- fájlkezelés különbségek:
  - az összes fájl művelet az ep128emu "work directory" alatt értelmezendő
  - a fájlhozzáférés nem érzékeny a kis/nagy betűre (semmilyen platformon)
  - a fájl/könyvtár írás, törlés, átnevezés mindig hibával tér vissza
  - a REG_DRIVE_SELECT nem implementált (mivel minden fájlművelet ugyanoda mutat)
  - a DSK fájl felcsatolás nem implementált (természetesen az ep128emu amúgy kezeli a formátumot)
- hiányzó REG_FUNCTION_EXECUTE általános rutin: copy_dir_to_psram (no-op)
- hiányzó REG_FUNCTION_EXECUTE PSRAM drive műveletek (az összes)
- SID emuláció az ep128emu-ba épített resid-del történik, ami nem pontosan a tvc256++-ba épített cucc emulálása
- devtool specifikus eltérés: az U2-U3 lapra a TVC256++ alapesetben a Fast RAM 4-5 lapját helyezi, de hogy a devtool memóriába áttöltő funkciója működjön indítás után, itt az alapérték az eredeti TVC memória (0xFF a vonatkozó regiszterekben).

## Főbb 2DFX emulációs hiányok
- csak 2 MB RRAM van emulálva
- még nagyon kezdeti állapotban van, egyedül TVC-n működik és csak a logo display és az upload

## Installálás
Segítség szükség esetén: [enterpriseforever wiki, ep128emu leírás](https://wiki.enterpriseforever.com/index.php?title=EP128Emu_le%C3%ADr%C3%A1s)
illetve [enterpriseforever wiki, más gépek emulálása](https://wiki.enterpriseforever.com/index.php?title=EP128Emu_-_m%C3%A1s_g%C3%A9pek_emul%C3%A1l%C3%A1sa#TVC64Emu)

TVC indítás: `ep128emu.exe -tvc`. A Machine/Configure ablakban a két utolsó fül az érdekes: a SID emuláció és a SpriteExt kapcsolható itt be.

## Ajánlott TVC konfiguráció
ROM szegmensek:
- 0x05: `tvc256++.rom` (a 16 kB-osra fordított változat, nem az eredeti 12 kB-os!)
- 0x10..0x13: 4x16 kB kezdeti TVC256++ fastRAM tartalom (tvc256_ram?_init... neveken)

A TVC256++ kártya a 2-es bővítő slotban érhető el, 0x30-as port tartomány. `EXT 2`-re elő kell jöjjön a menü, ami az írási és drive váltási műveletek kivételével működik. A portok működnek a tvc256++.rom nélkül is.

A 2DFX kártya a fix 248/249-es porton érhető el, TVC bővítő slot-ot nem foglal.

=== Egyéb TVC konfiguráció ===
További ROM szegmensek:
- 0x03: ha ide be van töltve a tvc_dos12d.rom, akkor lesz floppy emuláció 
  a 0-s bővítő slotban - ez korábbi funkció, nem most került bele
- 0x04: ha ide be van töltve a tvcfileio.rom, akkor lesz emulátor file io 
  az 1-es bővítő slotban - ez korábbi funkció, nem most került bele, és a tvc256++ extra
  funkciókat nem tudja, TVC256-hoz célszerű itt nem betölteni, a tvc256++ emuláció már megoldja magának
- 0x20..0x23: ide lehet betölteni a Gamecard 64kB-os ROM-ot.
  Ha ez ki van töltve (és nincs semmi a 0x06-os szegmensen) akkor a GameCard játékok is elérhetők.

Kerülő megoldás fájlbetöltéshez: rom-ok lehetnek tetszőlegesen, de indítás után
- poke 2821,128-cal működik a floppy (ha a floppyhoz tartozó EXT 2 nem is),
- poke 2821,129-cel működik a fileio (ha a 4-es szegmensen volt),
- poke 2821,130-cal működik a tvc256++ fájl kezelés,
- poke 2821,5-tel pedig a kazetta emuláció.

A `tvc12_fast2_sys` és a `tvc22_fast_sys` rom-ok rövidebb bootolásra vannak megpatkolva, használatuk teljesen opcionális (az egyik vagy másik eredeti helyett, a 0-s szegmensen)

Debuggerben (alt+B) látható a 2 MB-nyi slow ram (TVC256++) / resource RAM (2dfx), abszolút kezdőcím `1A0000`. A TVC256++ fast ram kezdőcíme `3A0000`.

## Demo programok
demomou.cas / demomou.bas: TVC256++ grafika + egér demo, az egér pozíciót és a sprite ütközésregisztert mutatja.
Billentyűk: 
- q-w-e: graphics 2-4-16 (a TVC eredeti képernyője)
- a-s-d-f: overlay nincs - txt 2 - txt 16 - bitmap
- z-g-h-j: az overlay scrollozása (átfordulás után az extra keret is megjelenik)
- o-p: aktív sprite 0..7 váltás (az aktív sprite-ot az egér mozgatja)
- u-i: a kétszínű sprite-ok tükrözése (vízszintes-függőleges)
- k-l: az aktív sprite fázisának váltása
- n-m: az aktív sprite színének váltása (2 színű sprite módban)
- b: az aktív sprite background/foreground váltása
- v: 2/16 színű sprite váltás (16 színű sprite-ból csak 1 van feltöltve)

demorst.cas / demorst.bas: TVC256++ RST $30 demo, az OS rendszer hívásokat tudja tesztelni
Betöltés után EXT 0-val hívható az RST 30:
  EXT 0,<funkció>,<DE értéke>,<BC értéke>
Visszatérés után a 6644..6640 címeken található az A..E regiszterek értéke
Pl. a GET_IOBASE $5D hívás, az eredmény a C regiszterben:
  EXT 0,93,0,0
  PRINT PEEK(6642)

crtcmax3.cas / crtcmax3.bas: 2DFX logo demo.
- wasd: logo mozgatása
- q: logo ki/be
- e: a TVC képernyő kiterjesztése (a basic képernyő szétesik tőle)
