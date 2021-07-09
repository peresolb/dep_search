# Strukturen til originale Depsearch

## Innledning
Dette dokumentet inneholder en gjennomgang av strukturen i den originale Depsearch-koden og refleksjoner om hvordan koden kan refaktoreres.

## Indeksering
Fila `build_index.py` lager en sqlite-database av en conllu-fil.
`cat ../no_bokmaal-ud-dev.conllu | python build_index.py --max 100000 -d ../no` lager en database 
med 100 000 setninger i `../no/trees_00000.db`. Den lager også en fil `symbols.json` i samme mappe,
som ser ut til å inneholde pos-tagger, feats og deprels med statistikk.

### Indekseringsprosedyre
1. `scr_data` tilordnes outputet av `read_conll()` på input fra kommandolinja. `read_conll` er en generator
som gir fra seg (`yields`) én og én setning av gangen som en liste av lister av kolonner. Den gir også fra seg
en liste med kommentarer (`#...` i conll-fila).
2. Objektet `SymbolStats()`, definert i fila `tree.py` initialiseres i variabelen `stats`. `symbols.json`.
ser ut til å bli generert herfra.
3. Det opprettes en fil som per default heter `trees_00000.db`. Det sjekkes om en fil ved det navnet fins fra
før, slettes den. En sqlite3-tilkobling `conn` opprettes i denne fila.
4. funksjonen `prepare_tables(conn)` kjøres. Denne kjører `CREATE TABLE`-statements for tabellene `graph`, `token_index`,
`lemma_index`, `tag_index`, `rel`.
5. Funksjonen `fill_db()` kjøres. Denne oppretter objektet `Tree` fra `tree.py` og `scr_data` leses inn i dette objektet via
klassemetoden `from_conll()`. Denne ser ut til å kjøre noen valideringer og noen normaliseringer.  Dessuten ser den ut til å 
lagre informasjon om rekursiv dominans (veldig nyttig!). Har ikke gått gjennom koden i detalj.
i detalj. Den returnerer informasjonen fra `scr_data` som objektsattributter, og i `fill_db()` blir dette lagt til i databasen
via `INSERT`-statements.
6. `build_indices()` kjører en del `CREATE UNIQUE INDEX`-statements på databasen.
7. Databasen lukkes.
8. Det skrives statistikk.

#### Tanker om ny organisering
Jeg ser for meg en mappestruktur som er omtrent slik der koden for applikasjonen er i en mappe som heter `dep_search/` med en undermappe
`indexer/`. I denne undermappa ligger koden for indeksering. Alle sql-statements bør flyttes til en egen `constants`-fil. Håndtering av
databasetilkobling bør også skilles ut i en egen fil, og det bør være et objekt for dette. `build_index.py` bør kun inneholde selve koden
for riktig innmating av trebankdataene. Lesing av conll-bør foregå i en fil med et mer forståelig filnavn enn `tree.py`. Genereringen
av statistikk bør gjøres mer transparent. Alle funksjoner bør dokumenteres. Conll-filer bør leses fra fil, ikke fra kommandolinje. Det
bør være tester av all koden.

## Søk
TODO