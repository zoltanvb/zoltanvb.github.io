## ep128emu 2.0.12.x

Szia! Ez itt az ep128emu (egyik) fejlesztés alatti verziója, elsősorban a TVC256++ és 2DFX kártyák támogatásához. Jelenlegi verzió: 

| ep128emu verzió | TVC256++ firmware kompatibilitás | 2dfx firmware verzió |
|:--------:|:----:|:-----------:|
| 2.0.12.6 | 0.24 | (kísérleti) |

## Letöltés
Lásd az [ep128emu-core Releases](https://github.com/libretro/ep128emu-core/releases) oldalon. Az `ep128emu-devtool.exe` csak TVCDevTool-os használatra ajánlott.

## Hibák jelzése
Az [enterpriseforever fórumon](https://enterpriseforever.com/emulatorok/), az [ep128emu-core Issues](https://github.com/libretro/ep128emu-core/issues) oldalon, vagy [Discordon](https://discord.gg/BZKtJjB) (itt kérnék egy @zoltanvb megszólítást, mert nem nézem olyan gyakran).

## Főbb TVC256++ emulációs hiányok
- csak 2 MB PSRAM van emulálva
- időzítés nem pontos (a műveletek rögtön végrehajtódnak, nincs várakozás)
- sprite ütközés a szélső keret alatt nincs jelezve
- sprite ütközés interrupt nem nagyon volt tesztelve, lehetnek gondok
- fájlkezelés különbségek:
  - az összes fájl művelet az ep128emu "work directory" alatt értelmezendő
  - a fájlhozzáférés nem érzékeny a kis/nagy betűre (semmilyen platformon)
  - a rövidített nevű vagy név nélküli fájlbetöltés nem támogatott (bár a .cas kiterjesztés elhagyható a load parancsban)
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
Windows alatt a telepítő intézi, Linux esetében kicsomagolás után ajánlott az epmakecfg futtatása, ami a ROM letöltést és konfiguráció generálást intézi.

Segítség szükség esetén: [enterpriseforever wiki, ep128emu leírás](https://wiki.enterpriseforever.com/index.php?title=EP128Emu_le%C3%ADr%C3%A1s)
illetve [enterpriseforever wiki, más gépek emulálása](https://wiki.enterpriseforever.com/index.php?title=EP128Emu_-_m%C3%A1s_g%C3%A9pek_emul%C3%A1l%C3%A1sa#TVC64Emu)

TVC256++ demo programok a [TVC256++ oldalán](https://github.com/dikdom/tvc256kplusplus/blob/main/binaries/flash-root-1.3-files.zip) érhetőek el, a .zip fájlt az ep128emu munkakönyvtárába kicsomagolva `load "launcher` utasítással indítható.

## Ajánlott TVC konfiguráció
A generált konfigurációk közt megtalálható több TVC256++ változat is. Aki magának szeretné összeállítani, az a következőkre ügyeljen:

- a Machine/Configure ablakban a két utolsó fül az érdekes: a SID emuláció és a SpriteExt kapcsolható itt be.
- ROM szegmensek:
   - 0x05: `tvc256-024.rom` (ep128emu-ba a 16 kB-osra fordított változat szükséges, nem az eredeti 12 kB-os)
   - 0x10..0x13: `tvc256-raminit.rom` 4x16 kB kezdeti TVC256++ fastRAM tartalom

A TVC256++ kártya a 2-es bővítő slotban érhető el, 0x30-as port tartomány. `EXT 2`-re elő kell jöjjön a menü, ami az írási és drive váltási műveletek kivételével működik. A portok működnek a tvc256++.rom nélkül is.

A 2DFX kártya a fix 248/249-es porton érhető el, TVC bővítő slot-ot nem foglal.

## Egyéb TVC konfiguráció
További ROM szegmensek:

- 0x03: ha ide be van töltve a `tvc_dos12d.rom`, akkor lesz floppy emuláció a 0-s bővítő slotban - ez korábbi funkció, nem most került bele
- 0x04: ha ide be van töltve a `tvcfileio.rom`, akkor lesz emulátor file io az 1-es bővítő slotban - ez korábbi funkció, nem most került bele, és a tvc256++ extra funkciókat nem tudja. TVC256-hoz célszerű itt nem betölteni, a tvc256++ emuláció tartalmazza a file kezelést is.
- 0x20..0x23: ide lehet betölteni a Gamecard 64kB-os ROM-ot. Ha ez ki van töltve (és nincs semmi a 0x06-os szegmensen) akkor a GameCard játékok is elérhetők.

Kerülő megoldás fájlbetöltéshez: rom-ok lehetnek tetszőlegesen, de indítás után:

- `poke 2821,128`-cal működik a floppy (ha a floppyhoz tartozó EXT 2 nem is),
- `poke 2821,129`-cel működik a fileio (ha a 4-es szegmensen volt),
- `poke 2821,130`-cal működik a tvc256++ fájl kezelés,
- `poke 2821,5`-tel pedig a kazetta emuláció.

A `tvc12_fast2_sys` és a `tvc22_fast_sys` rom-ok rövidebb bootolásra vannak megpatkolva, használatuk teljesen opcionális (az egyik vagy másik eredeti helyett, a 0-s szegmensen)

Debuggerben (alt+B) látható a 2 MB-nyi slow ram (TVC256++) / resource RAM (2dfx), abszolút kezdőcím `1A0000`. A TVC256++ fast ram kezdőcíme `3A0000`.
