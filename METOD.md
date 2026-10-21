# AI-testets Metod 1.0

**Version:** Metod 1.0 (`method_version` i `method.yaml`)
**Datum:** 2026-10-20
**Metodägare:** Satori ML AB (avsändare, se avsnitt 15)
**Låsning:** låsfilens hash 20049643e10befd569ef56548986b25790f6f54b4847d32cda7412752024ba8f, publicerad 2026-10-21 16:00 UTC (18.00 svensk tid), före huvudkörningen (avsnitt 13)
**Synpunkter:** tas emot löpande, se avsnitt 18 och `CONTRIBUTING.md`

Det här dokumentet är fristående. Det beskriver hur **AI-testet** mäter AI-modeller på svenska, och vad vi gör med resultaten. Varje upplaga har ett namn efter tiden den körs, till exempel "AI-testet, hösten 2026". Testet består av **delar** efter vad som mäts: **Detaljer** ("Märker AI:n när något är fel i en text? Mätt på svensk lagtext"), **Sverige** ("Vet AI:n eller hittar den på?") och **Kostnad** (vad vanliga arbetsuppgifter kostar per modell; uppgiftsmätningen, beskriven separat). Där inget annat sägs gäller en regel alla delar. Det finns **inget sammanvägt totalresultat** förrän en viktningsregel är förregistrerad; fram till dess redovisas resultat per del.

Metod 1.0 låses innan huvudkörningen. Låsfilens hash publiceras före körningen, och resultaten publiceras tillsammans med metoden. Det har alltså inte funnits någon granskningsperiod före första körningen. Synpunkter tas emot löpande och prövas inför nästa version (1.1). Efter låsning ändras metoden bara med ett nytt versionsnummer och en dateringsmärkt post i ändringsloggen.

Gör gärna invändningar. Avsnitt 17 pekar ut de punkter där vi själva är mest osäkra.

---

## 1. Syfte

AI-testet ska ge företag, journalister, beslutsfattare och andra som väljer eller använder AI-modeller ett underlag som går att kontrollera. Tanken är att varje tal på sajten ska kunna följas tillbaka till en fråga, ett svar, en rättningsregel och en statistisk regel som är offentliga och som bestämdes innan svaren var kända.

Metoden är skriven så att den kan ha fel på ett sätt som syns. Därför publiceras reglerna, ändringarna och synpunkterna.

## 2. Vad som mäts, och vad som inte mäts

**Mäts:**
- Modellen via API, i standardläge, utan webbsökning och utan verktyg.
- **Delen Sverige:** svensk faktakunskap och benägenheten att avstå eller hitta på när modellen inte vet (vilken kommun eller vilket län en ort hör till, och påhittade kommunnamn).
- **Delen Detaljer:** om modellen känner igen gällande svensk lag. Modellen får en paragraf ur en gällande författning där kod antingen inte har ändrat något eller har ändrat en sak så att innebörden blir en annan (ett tal, en paragrafhänvisning, ska/får, en negation, en myndighet eller aktör, senast/tidigast), och ska säga om texten stämmer och i så fall vad som är ändrat. I den kombinerade varianten, som är vald för huvudkörningen, finns också språkfel i en del av styckena; innehållet är huvudmåttet och språket ett beskrivande mått. Paragrafer som ändrats de senaste tre åren används inte, så att modellernas kunskapsgräns inte avgör. Kända lagar för företag och den långa svansen av mindre kända författningar redovisas var för sig. I piloten (24 stycken) låg ingen modell över taket på 90 %. Pilotens tre konsensusflaggor stämde med gällande lydelse när de kontrollerades mot en andra källa; det var modellerna som hade fel, i ett fall genom att återge en äldre lydelse.
- **Språkunderlaget:** korrekt svensk skrift, med två beskrivande nivåer. Den ena är **korrekturläsning**: modellen får ett stycke löpande text ur svensk författningstext där kod har satt in noll till tre fel (böjning, genus, stavning, sär- och sammanskrivning, de/dem, var/vart), och ska ange exakt de felen. Stycken utan fel finns med, så att det syns vilka modeller som hittar på fel. Den andra är enkla frågor med en regel per fråga (till exempel preteritum av ett starkt verb). Bara uppgifter som kod kan rätta utan språkmodell ingår. Fri text, stil och ton mäts inte.
- Alla tabeller märks "API, standardläge, utan sökning".

**Mäts inte:**
- Konsumentappar. De har egna systemprompter och sökning, och kan bete sig annorlunda än API:et.
- Lika beräkningsbudget mellan modeller. I standardläge får modeller olika mycket resonemang. Det är verklig användning men inte lika villkor. Antal tokens redovisas så att skillnaden syns.
- Allmän förmåga. Ett prov om svenska orters kommuntillhörighet säger inget om hur bra en modell är på att räkna eller programmera, och språkunderlaget säger inget om hur bra en modell skriver egna texter, bara om den hittar och rättar fel av bestämda slag.

**Namn** (beslut 2026-10-21). Testet heter AI-testet, och upplagan anges med säsong och år ("AI-testet, hösten 2026"). Delarna heter Detaljer, Sverige och Kostnad. Detaljer är det som under utvecklingen kallades Lagprovet (kombinerad variant, där språkmåttet är beskrivande), och Sverige är det tidigare Sverigeprovet. Språkunderlaget (det tidigare Språkprovet: korrekturläsning och enkla frågor) är ingen egen del; det är beskrivande underlag till Detaljer eller utvecklingsdata. I `method.yaml` står visningsnamnen under `parts`; de interna id:na (`lagprovet`, `sverigeprovet`, `sprakprovet`) är oförändrade. Beteckningarna "delprov A" och "delprov B" används inte, eftersom A är namnet på ett körläge (avsnitt 9). Instruktionsföljning ingår inte i version 1.0.

## 3. Åtkomstväg

- **Alla modeller nås via OpenRouter.** Leverantören låses till modellens förstapartsleverantör med `provider.order = [leverantör]`, `allow_fallbacks = false` och `data_collection = "deny"`.
- **Kontroll per anrop.** Faktisk leverantör (`provider_actual`) och det modell-id som svaret anger loggas för varje anrop. Om ett anrop har gått till en annan leverantör än den låsta stoppas publiceringen tills avvikelsen utretts (grinden kallas G6, utfall HOLD).
- **Undantag för öppna modeller.** Om förstapartens eget API inte godtar `data_collection = "deny"` används en namngiven tredjepartsleverantör i full precision, med `allow_fallbacks = false` och `data_collection = "deny"`. Samma väg används för modeller från leverantörer som står på en förregistrerad försiktighetslista (`training_on_inputs_vendors` i `method.yaml`): leverantörer som vi i förväg behandlar som om de kan träna på inskickade data. I version 1.0 står DeepSeek på listan. Listan är ett försiktighetsval och inget påstående om vad leverantörens villkor säger, och någon källa för villkoren redovisas inte här. För dessa modeller körs leverantörskänslighet på minst två leverantörer, och skillnaden redovisas.
- **Valfri jämförelse.** När det finns egna API-nycklar kan en engångskontroll göras mellan OpenRouter och leverantörens direkta API för högst två leverantörer, med TOST på ±5 procentenheter. Bara offentliga frågor används. Privata frågor skickas aldrig till leverantörens eget API. Kontrollen är inte ett krav.
- **Parametrar.** Följande skickas och loggas: temperatur, `top_p`, `seed`, `max_tokens`, resonemangsnivå eller tokenbudget, systemprompt och modell-id med datum. Temperatur, `top_p`, `seed` och resonemangsnivå är leverantörens standard. En parameter som en modell inte stöder skickas inte och loggas som "ej stödd".
- **Reproducerbarhet.** 10 % av frågorna körs också vid temperatur 0 för att visa hur mycket svaren varierar.
- **Tokenbudget:** `max_tokens = 16000`, lika för alla modeller och alla delar (beslut 2026-10-21; utvecklingskörningarna före delen Detaljer hade 8 000; se avsnitt 8 och ändringsloggen).
- **Modellurval (förregistrerad regel, beslut 2026-10-20).** Regeln tillämpas på OpenRouters modellkatalog och leverantörernas endpoints den dag som står i `method.yaml` (`model_selection.catalog_date`), och underlaget sparas rått med hash.
  1. *Anthropic, OpenAI och Google:* alla aktuella allmänna textmodeller i leverantörens huvudlinje, en per nivå (snabb, mellan, topp, och en högsta nivå när leverantören har en sådan), i senaste versionen, plus den mest kapabla modellen i den sista versionen av förra generationen.
  2. *Övriga leverantörer:* leverantörens flaggskepp, alltså den mest kapabla allmänna textmodellen i senaste versionen.
  3. *Varianter räknas inte som egna modeller:* samma modell med mer resonemang (pro-läge), batch, snabbare eller billigare tjänstenivåer (flex, fast, priority), bild- och kodvarianter, och dyrare SKU:er av samma modell.
  4. *Tillägg utanför regeln* redovisas öppet, med skäl. De körs och jämförs som alla andra.
  Listan ändras bara med en ny metodversion. Modellerna och skälen:

| Modell (OpenRouter-id) | Leverantör | Nivå enligt regeln | Låst leverantör |
|---|---|---|---|
| `anthropic/claude-fable-5.1` | Anthropic | högsta nivån (Fable) | Anthropic |
| `anthropic/claude-opus-5.5` | Anthropic | topp (Opus) | Anthropic |
| `anthropic/claude-sonnet-5.5` | Anthropic | mellan (Sonnet) | Anthropic |
| `anthropic/claude-haiku-5.5` | Anthropic | snabb (Haiku) | Anthropic |
| `anthropic/claude-opus-4.8` | Anthropic | förra generationen, sista versionen | Anthropic |
| `anthropic/claude-opus-4.5` | Anthropic | **tillägg utanför regeln:** diskuteras mycket i forum, inte vald av regeln | Anthropic |
| `openai/gpt-6-astra` | OpenAI | topp (Astra) | OpenAI |
| `openai/gpt-6.1-sol` | OpenAI | mellan (Sol) | OpenAI |
| `openai/gpt-6-luna` | OpenAI | snabb (Luna) | OpenAI |
| `openai/gpt-5.6-sol` | OpenAI | förra generationen, sista versionen (GPT-5.6, flaggskeppet Sol) | OpenAI |
| `google/gemini-4-argon` | Google | topp | Google AI Studio |
| `google/gemini-3.8-flash` | Google | snabb (Flash, senaste versionen; generation 4 har ännu ingen Flash) | Google AI Studio |
| `google/gemini-3.1-pro-preview` | Google | förra generationen, sista versionen (finns bara som preview) | Google AI Studio |
| `x-ai/grok-4.7` | xAI | flaggskepp | xAI |
| `mistralai/mistral-large-4-0` | Mistral | flaggskepp (europeisk) | Mistral |
| `deepseek/deepseek-v4-pro-0813` | DeepSeek | flaggskepp (öppna vikter) | Parasail, fp8 (försiktighetslistan, se ovan) |
| `qwen/qwen3.8-max-0902` | Alibaba (Qwen) | flaggskepp | Alibaba |
| `moonshotai/kimi-k3` | Moonshot (Kimi) | flaggskepp (öppna vikter) | Moonshot AI |

  Två tolkningar bekräftades av metodägaren 2026-10-21: regeln för OpenAI pekar på GPT-5.6 Sol (katalogen visar GPT-5.6 som en senare version av generation 5 än GPT-5.5), och Googles Flash är den snabba nivån, så Gemini 3.5 Flash-Lite ingår inte.
- **Begränsning med mellanhand.** En mellanhand kan översätta vissa parametrar, till exempel resonemangsnivå. Det är därför loggningen finns och därför leverantörskänsligheten körs.

## 4. Frågekällor och licenser

Frågor och facit skapas av kod ur öppna källdata. Språkmodeller skriver inte frågor, facit eller påhittade namn.

| Källa | Licens eller villkor |
|---|---|
| SCB (kodlistor, tätorts- och småortstabeller, digitala gränser) | CC0. Får inte framställas som officiellt samarbete med SCB. |
| Wikidata | CC0 |
| Lantmäteriets öppna geodata | CC0. För Ortnamn är licensen inte fastställd, och vi använder det inte som enda källa. |
| Isof, Ortnamnsregistret | CC BY. Används för existenskontroll av påhittade namn när registret går att nå. |
| Riksdagen och SFS (för senare frågetyper) | Källa anges ("Källa: Sveriges riksdag"). Sajten framställs inte som riksdagens. Ingen logotyp. |
| SALDO-morfologi (Språkbanken Text, Göteborgs universitet), för språkunderlaget | CC BY 4.0. Källa anges: "SALDO, Språkbanken Text, Göteborgs universitet". Ordformer, genus och ordlistan som felstavningar prövas mot. |
| Wikidatas svenska lexem, för språkunderlaget | CC0. Andra källa för ordformer och genus, och kontroll av att ett ord finns. |
| Egna meningar och listor, för språkunderlaget | Egen text (Satori ML AB), skriven för provet: mallmeningar för de/dem och var/vart, och efterled för sammansättningar med siffra. |
| Svensk författningstext (SFS), för språkunderlagets korrekturläsning | Fri enligt 1 kap. 9 § upphovsrättslagen. Hämtad ur Riksdagens öppna data ("Källa: Sveriges riksdag"); källa (SFS-nummer och rubrik), URL och hämtningstid anges per stycke. Sajten framställs inte som riksdagens. |

**Spärrlista mot olämpliga ord** (beslut 2026-10-21). Alla generatorer, både språkunderlagets och Sverigedelens påhittade namn, prövar varje ord, lemma, felstavning, påhittat namn och stycke mot en egen spärrlista (`method/sparrlista.py`): svordomar, sexuella och vulgära ord, ord om kroppsfunktioner, nedsättande ord om grupper eller personer, och ämnen som är olämpliga i ett test som visas för företag (till exempel sexualbrott, våldsbrott, självmord och droger). För ord och namn räcker det att en post finns någonstans i ordet; i löpande text spärrar korta poster bara som hela ord. Listan är bred med avsikt, och den utesluter också en del oskyldiga ord. Riktiga ortnamn spärras inte.

**Skyddade ordlistor används inte.** Svenska Akademiens ordlista (SAOL), Svensk ordbok och andra ordböcker vars innehåll är skyddat används inte som källa och kopieras inte. Grammatiska regler (till exempel att en sammansättning med en förkortning skrivs med bindestreck) är fakta och anges utan kopierad text.

**Krav på varje fråga:**
- **Två oberoende källor** ska ge samma facit. Facit lagras tillsammans med båda källornas värden. Småortens namn finns bara i Wikidata. SCB:s småortsdata saknar namn. För nivå 3 används därför SCB:s kommun- och folkmängdsdata, och en koordinatkontroll av Wikidatas punkt mot SCB:s kommungränser, som andra källa. Det är en svagare kontroll än för övriga nivåer och redovisas som det.
- **Ursprung per fråga** sparas: källa, källversion eller dumpdatum, "as of"-datum, mall, nivå, entitet och slumpfrö.
- **Svarsformatet står i frågan**, till exempel "Svara med kommunens namn". Frågan anger också enhet och precision när det behövs.
- **Inga färska fakta.** Svaret ska ha varit offentligt före den äldsta kunskapsgränsen bland de testade modellerna, enligt leverantörernas dokumentation på låsdatumet, och vara oförändrat vid körningen.
- **Mångfald:** högst en fråga per entitet, mall och period.
- **Revision:** en tvetydig fråga revideras först (datum, län eller precision läggs till). Går den inte att göra entydig stryks den. Andelen reviderade och strukna frågor redovisas.
- **Facitrevision efter körningen:** en människa granskar frågan före publicering om minst hälften av modellfamiljerna (en röst per familj) svarar samma sak mot facit, eller om en enskild modell ger en motivering med källa som motsäger facit. För de fall där koordinatkontrollen finns bifogas den som underlag. Underlaget är inget beslut.

**Offentligt, ankare och privat:**
- 20 % av frågorna är offentliga (stratifierat urval). De poängsätts inte. De finns för granskning och märks med en canary-sträng.
- 25 % är fasta ankarfrågor som alltid är privata och återkommer varje period. De publiceras först när de pensioneras.
- Resten är privata med rotation. Ett hashvärde publiceras före körningen och frågorna avslöjas när de roteras ut.
- Den offentliga delens likhet med hela mängden prövas med TOST, marginal ±5 procentenheter. Visas ingen ekvivalens märks delen "ej visat representativ".
- En avvikelse mellan ankarfrågor och nya frågor är en varningssignal, inte en diagnos. Kontaminering, ändrad svårighet, ämnesspecifika förändringar och slump redovisas som möjliga förklaringar.
- Privata testdelar har inte visats skydda mot mättnad. De är en försiktighetsåtgärd, ingen garanti.

**Antal i huvudkörningen.** *Delen Sverige:* 40 frågor på nivå 1, 100 på nivå 2, 400 på nivå 3 (huvudnivån) och 60 påhittade, sammanlagt 600, genererade 2026-10-20 (inga pilotfrågor, inga entiteter eller namn från piloterna). Av dem är 20 % offentliga och poängsätts inte, så poängsatta blir 432 riktiga (320 på huvudnivån) och 48 påhittade. *Språkunderlaget:* 400 stycken på huvudnivån (korrekturläsning, lika många med 0, 1, 2 och 3 fel) och 170 enkla frågor på grundnivån (10 per regeltyp); de genereras först när den andra piloten är bedömd (avsnitt 5). *Detaljer:* 200 nya stycken, 50 per kategori (inga fel, bara innehållsfel, bara språkfel, båda) och inom varje kategori 25 ur kända lagar och 25 ur övriga författningar, inget stycke från piloten. Varje stycke märks med rättsområde (HR/arbetsrätt, ekonomi/skatt/redovisning, bolags- och handelsrätt, offentlig sektor/förvaltning, övrigt), så att resultaten kan redovisas per målgrupp. Dessutom 50 stycken ur paragrafer som ändrats för cirka 1–3 år sedan (ändringsnotis 2023, 2024 eller 2025). De redovisas separat och beskrivande, inte i huvudtalet. Facit är gällande lydelse, och den tidigare lydelsen sparas per stycke, så att det syns när en modell "rättar" tillbaka till den gamla texten. Alla stycken i Detaljer är privata före körningen, och deras hash publiceras. **Tvåkällskontroll:** varje stycke ur Riksdagens konsoliderade text ska finnas ordagrant i lagen.nu:s konsoliderade text för samma författning. Kostnadstaket för huvudkörningen är 140 USD. Frågornas hash per del och per uppdelning (offentlig, ankare, roterande) står i låsfilen och publiceras före körningen, frågorna i de privata delarna gör det inte.

**Språkunderlagets facit.** Ingen språkmodell skriver frågor eller facit. Varje fråga har två källor eller en källa och en regel:
- *Böjning:* SALDO:s former för den efterfrågade kategorin, och Wikidatas form för samma kategori måste finnas bland dem. Alla former som någon av källorna anger godkänns (till exempel både "simmade" och "sam"). Ord som finns i flera poster (homografer, till exempel "bok" som träd och som skrift) används inte, och inte heller sammansättningar vars form följer av ett vanligare grundord.
- *Genus:* SALDO och Wikidata ska ange samma genus, och ord med växlande genus används inte.
- *Stavning:* det rätta ordet finns i SALDO och i Wikidata. De tre felstavningarna skapas av kod med en regel (dubbelteckning, sj-, j- och tj-ljudens stavningar, e/ä och o/å) och får inte finnas som ordform någonstans i SALDO.
- *Sär- och sammanskrivning:* sammansättningar och förkortningsord ur SALDO (med Wikidata som andra källa för vanliga sammansättningar), sammansättningar med siffra ur en egen lista; regeln är skrivregeln om bindestreck.
- *Ordval:* egna mallmeningar där facit följer av en förregistrerad regel (de som subjekt, dem som objekt och efter preposition; var om plats, vart om riktning).
- Högst en fråga per ord eller mening i hela uppsättningen. Regeltypen (17 förregistrerade typer) lagras på varje fråga.
- *Korrekturläsning:* ett stycke per författning (SFS från 2010), 60–120 ord. Varje ord i originalstycket ska finnas i SALDO, vara en sammansättning av SALDO-ord eller vara ett namn med versal, så att äldre eller ovanlig stavning inte ger falska fel. Kod sätter in 0–3 fel i olika ord som inte står intill varandra och aldrig i ett ord med versal. Felen är av sex slag: böjning (starka verb och oregelbundna ord får regelbunden böjning, "skrev" blir "skrivde"), genus (en/ett före ett substantiv med fast genus), stavning (samma regler som grundnivån), särskrivning (en sammansättning delas vid sin enda ledgräns, inte när förleden också är en preposition, som "för ordning"), de/dem och var/vart. Utom för genus, de/dem och var/vart är det felaktiga ordet aldrig en ordform i SALDO. Samma ord rättas högst en gång i hela uppsättningen. **Facit är exakt de insatta felen**, med originalets form som rätt.

## 5. Nivåer

Nivåerna är fasta under en metodversion. De anpassas inte efter hur nya modeller presterar.

| Nivå | Fråga | Roll |
|---|---|---|
| 1 | Kommun, i vilket län? | Beskrivande grundnivå. Redovisas med konfidensintervall. Inget tak- eller golvkrav. Ingår inte i huvudtalet. |
| 2 | Tätort under 1 500 invånare, i vilken kommun? Länet anges i frågan. | Beskrivande grundnivå. Redovisas med konfidensintervall. Inget tak- eller golvkrav. Ingår inte i huvudtalet. |
| 3 | Småort, i vilken kommun? Länet anges i frågan. | **Huvudnivå.** Ingår i huvudtalet. Tak och golv gäller. |
| F | Påhittade kommunnamn, i vilket län? | Egen trappa (avsnitt 7). Ingår inte i träffsäkerhet eller index. |
| L | delen Detaljer: en paragraf ur gällande författning, oförändrad eller med en ändring av innebörden (i den kombinerade varianten också 0–2 språkfel). Rätt = rätt besked, och vid ändring rätt ställe och rätt lydelse. | **Huvudnivå för delen Detaljer** (innehållet). Tak och golv gäller. Språket redovisas beskrivande. |
| K | språkunderlaget: korrekturläsning av ett stycke ur SFS med 0–3 insatta fel. Rätt = exakt rätt mängd rättelser. | Beskrivande (sedan 2026-10-21). Redovisas med konfidensintervall. Inget tak- eller golvkrav. |
| G | språkunderlaget: enkla frågor, en regel per fråga (17 regeltyper: böjning, genus, stavning, sär- och sammanskrivning, ordval) | Beskrivande grundnivå. Redovisas med konfidensintervall. Inget tak- eller golvkrav. |

Nivåerna 1, 2, 3 och F hör till delen Sverige, K och G till språkunderlaget och L till delen Detaljer. Språkunderlagets nivåer K och G är beskrivande sedan 2026-10-21, när delen Detaljer tog över rollen som prov med svensk text på huvudnivå (beslut av metodägaren: "ingen mer pilot där"). Regeln om en pilot och en omdefiniering gäller varje del för sig efter lansering.

**Tak och golv gäller bara huvudnivån:** ingen modell får ligga över 90 % träffsäkerhet (punktskattning, intervallet visas bredvid), och den bästa modellen ska nå minst 30 %. Syftet är att provet varken ska vara mättat eller så svårt att det inte skiljer något åt.

**Regel för kommande prov:** en pilot och högst en omdefiniering av nivåerna per prov före låsning. Blir provet ändå för lätt publiceras det som mättat, med dokumentation. Det konstrueras aldrig vidare. Skälet är att varje justering mot dagens modeller formar provet efter dem.

**Urvalseffekt som redovisas:** nivåindelningen har formats med pilotmodellerna som referens (avsnitt 16).

I utvecklingskörningarna var grundnivåerna mättade: på nivå 1 svarade alla modeller rätt på samtliga frågor, och på nivå 2 låg flera modeller över 90 %. Det redovisas som ett fynd, inte som ett fel i provet. Modellerna kan tätorter men gissar oftare på småorter.

**Språkunderlagets första pilot (2026-10-20) var mättad.** Alla 18 modeller låg över taket på 90 % (lägsta 91,3 %, högsta 99,3 %, 150 enkla frågor). Metodägaren beslutade 2026-10-21 att omdefiniera språkunderlaget: korrekturläsning blev huvudnivå och de enkla frågorna blev grundnivå, prövat i en andra pilot. Regeln om en pilot per prov tillämpades inte här, eftersom metoden ännu inte var publicerad (uppbyggnadsfasen); den gäller efter lansering. Det redovisas som en pilotdriven ändring i ändringsloggen.

**Språkunderlagets andra pilot (2026-10-21, korrekturläsning)** skilde modellerna åt betydligt bättre (träffsäkerhet 62–97 % på 110 stycken), men de 8 starkaste modellerna låg fortfarande över taket på 90 %. Stavfel var tre av fyra insatta fel och hittades nästan alltid. Hur huvudnivån slutligen definieras beslutas av metodägaren före låsning (förslag i pilotrapporten) och skrivs in här.

## 6. Påhittade frågor

Påhittade frågor mäter om en modell hittar på när frågan gäller något som inte finns.

- **Bara entitetstyper med ett fullständigt officiellt register.** I den här versionen är det kommunnamn: Sveriges 290 kommuner är en sluten lista, så "det finns ingen sådan kommun" är verifierbart rätt.
- **Namnen genereras av kod, inte av språkmodell:** en stam och en ändelse hämtade ur riktiga ortnamn.
- **Spärr mot riktiga namn.** Ett namn förkastas om något av följande gäller:
  - namnet, eller dess normalform med genitiv-s struket, finns som etikett eller alias (svenska eller engelska) på något Wikidata-objekt med land Sverige
  - namnet finns i Isofs ortnamnsregister, om registret går att nå
  - namnet finns i de listor över kommuner, tätorter, småorter, historiska kommuner, socknar, församlingar och platser som hämtats
  - namnet ligger närmare än tre redigeringssteg (Levenshtein) från något riktigt namn, har en teckentrigram som är för ovanlig i riktiga ortnamn, eller börjar eller slutar på ett riktigt kommun- eller ortnamn
- **Kontrollen är automatisk.** Varje kontroll loggas per namn med källa, resultat och tidsstämpel. Ingen människa granskar varje namn. Det är en avvikelse från vår tidigare plan (se ändringsloggen och avsnitt 16).
- **Pilotspärr.** Ett påhittat namn markeras om minst två olika modeller påstår samma enda län om det. Namnet behålls i huvudanalysen och räknas som påhitt, eftersom kommunregistret är komplett. Resultatet redovisas också utan de markerade namnen (avsnitt 11).
- **Rätt beteende är både "vet ej" och "finns inte".** Det som räknas som fel är att namnge ett län, det vill säga att påstå att kommunen finns.

## 7. Beslutstrappa

Trappan är förregistrerad. Reglerna prövas i ordning, och ett svar får exakt ett utfall. Regel-id loggas per svar. Text, kod och `method.yaml` ska stämma överens, och en avvikelse ska anmälas, inte tystas.

**Svarets form.** Alla modeller instrueras att avsluta med en sista rad på formen `Svar: <svar>`. **Bara den raden rättas.** Systemprompten i läge A är:

> Svara kort. Avsluta alltid med en sista rad på formen 'Svar: <ditt svar>'. Om du inte vet svaret, skriv 'Svar: VET EJ'.

Ordalydelsen är godkänd av metodägaren (2026-10-15 för läge A, 2026-10-16 för A2 och B), fastställd i låsfilen och kan ändras först i en senare version. En giltig markörrad är den sista icke-tomma raden, på formen `Svar: <text>` med minst ett ord efter kolonet (versaler och gemener spelar ingen roll, och markdown runt raden godtas). Saknas en giltig markörrad rättas hela svarstexten bokstavligt enligt trappan nedan, och svaret räknas dessutom som **formatbrott**. Formatbrott redovisas för alla modeller. Känslighetsanalysen "bokstavlig rättning av hela texten" visar om slutsatserna beror på markörregeln (avsnitt 11).

**Mätfel är inget utfall.** Ett tekniskt fel (API-fel, timeout, 5xx, eller att tokenbudgeten tar slut utan svar) körs om högst två gånger. Kvarstår felet räknas svaret som Ogiltigt och redovisas som tekniskt bortfall per modell. Budgeten höjs aldrig för en enskild modell, eftersom det gynnar modeller som använder många tokens.

**Riktiga frågor.** Fem utfall: Rätt, Delvis, Fel, Avstår, Ogiltigt.

| Regel | Villkor | Utfall |
|---|---|---|
| R0 | Svaret är tomt | Ogiltigt |
| R1 | Tekniskt fel som kvarstår efter två omkörningar | Ogiltigt |
| R2a | Exakt en kandidat efter normalisering och borttagning av aliasdubbletter, och den matchar facit med efterfrågad precision (gäller även "vet ej, men troligen X" när X är rätt) | Rätt |
| R2b | En annan kandidat, flera kandidater där någon är fel, eller en annan entitetstyp än den efterfrågade (till exempel län när kommun efterfrågades) | Fel |
| R2c | Siffra med grövre precision än efterfrågad (rätt tal avrundat till tusental när heltal efterfrågades). 0 poäng, räknas i nämnaren. | Delvis |
| R2d | Bara när garderingsregeln AA används: gardering med kandidat räknas som ej försökt | Avstår |
| R3a | Ingen kandidat, och uttryckligt "vet ej" eller osäkerhet | Avstår |
| R3b | Ingen kandidat, men en nekad kandidat ("det är inte X"), eller "vet ej" tillsammans med en nekad kandidat | Fel |
| R3c | Ingen kandidat, högst fem ord och ingen vägran: kort fritt svar utan känt namn | Fel |
| R3d | Ingen kandidat, vägran utan osäkerhetsskäl, eller längre irrelevant text | Ogiltigt |

**Garderingsregel.** Huvudregeln följer SimpleQA: "vet ej, men troligen X" rättas på X. Garderingsmarkörer är *troligen, kanske, förmodligen, antagligen, jag tror, tror att, vet ej, vet inte, osäker*. Regeln enligt AA:s rättningsmall, där gardering med kandidat räknas som ej försökt, används bara som känslighetsanalys.

**Påhittade frågor har en egen trappa** som redovisas separat och inte ingår i index eller träffsäkerhet.

| Regel | Villkor | Utfall |
|---|---|---|
| F0 | Svaret är tomt | Ogiltigt |
| F1 | Svaret avvisar men påstår också något (namnger län eller kommun utan förslagsformulering). Blandat svar räknas som påhitt. | Påhitt |
| F2 | Säger att entiteten inte finns, utan att påstå något om den | Korrekt avvisande |
| F3 | Föreslår en riktig entitet ("du kanske menar …") utan påstående | Korrekt avvisande |
| F4 | Namnger ett län eller en kommun, alltså påstår att entiteten finns | Påhitt |
| F5 | "Vet ej" | Avstår |
| F6 | Högst fyra ord och ingen vägran: kort fritt svar behandlas som påstående | Påhitt |
| F7 | Övrigt | Ogiltigt |

**Språkunderlaget har en egen trappa** (`subtests.sprakprovet.decision_ladder` i `method.yaml`, koden i `method/sprakprovet.py`). Samma systemprompt och samma markörrad som i delen Sverige. Fyra utfall: Rätt, Fel, Avstår och Ogiltigt (Delvis finns inte).

| Regel | Villkor | Utfall |
|---|---|---|
| S0 | Svaret är tomt efter normalisering | Ogiltigt |
| S1 | Tekniskt fel som kvarstår efter två omkörningar, eller att tokenbudgeten tog slut | Ogiltigt |
| S2a | Exakt en kandidat, och den är en godkänd ordform, rätt bokstav eller alternativets text, eller rätt ord ur listan (en/ett, de/dem, var/vart) | Rätt |
| S2b | En annan ordform, bokstav eller ett annat ord ur listan, flera kandidater, eller bokstav och text som pekar på olika alternativ | Fel |
| S3a | Ingen kandidat, och uttryckligt "vet ej" eller osäkerhet | Avstår |
| S3b | Ingen kandidat, högst fem ord | Fel |
| S3c | Ingen kandidat, längre text | Ogiltigt |

**Korrekturläsningen (nivå K) har egen systemprompt och trappa** (`subtests.sprakprovet.proofreading` i `method.yaml`). Systemprompten, ett förslag som metodägaren ska godkänna:

> Du är korrekturläsare. Texten nedan kan innehålla noll, ett eller flera språkfel: stavfel, fel böjning, fel genus (en/ett), särskrivning av sammansatta ord eller fel ordval (de/dem, var/vart). Rätta bara sådana fel. Ändra inte stil, ordval, ordföljd, skiljetecken eller stor och liten bokstav. Skriv varje fel på en egen rad på formen: fel → rätt, där fel är ordet eller orden exakt som de står i texten. Avsluta med en sista rad 'Svar: KLART'. Om texten inte innehåller något fel, skriv bara raden 'Svar: INGA FEL'.

Användarmeddelandet är "Text:" följt av stycket. *Motivering:* (1) prompten räknar upp vilka slags fel som kan finnas, så att uppgiften är definierad och inte ett gissningsspel om vad som räknas som fel; (2) den förbjuder ändringar av stil, ordval, ordföljd, skiljetecken och versaler, eftersom sådana ändringar är smakfrågor som kod inte kan rätta rättvist och som annars skulle räknas som falsklarm; (3) det fasta formatet "fel → rätt" och markörraden gör rättningen deterministisk, på samma sätt som "Svar: <svar>" i delen Sverige; (4) "Svar: INGA FEL" är ett uttryckligt sätt att säga att texten är felfri, motsvarigheten till "VET EJ": en modell som hittar på fel i en felfri text gör samma sak som en modell som namnger ett län för en påhittad kommun.

| Regel | Villkor | Utfall |
|---|---|---|
| K0 | Svaret är tomt | Ogiltigt |
| K1 | Tekniskt fel som kvarstår efter två omkörningar, eller att tokenbudgeten tog slut | Ogiltigt |
| K2a | Modellens rättelsepar är exakt facits par (för stycken utan fel: inga par) | Rätt |
| K2b | Minst ett par stämmer, men inte exakt mängden (missade fel eller extra par) | Delvis |
| K3 | Inget par stämmer: alla fel missade, eller bara falsklarm | Fel |
| K4 | Ingen markörrad, inga rättelserader och ingen fras "inga fel", längre fri text | Ogiltigt |

Ett rättelsepar är de ord som skiljer vänster och höger sida åt på en rad med pil ("en beslut → ett beslut" ger paret en/ett); gemener, och markdown, numrering och citattecken tas bort. En rad som bara upprepar formatet ("fel → rätt") räknas inte. Markörraden avgör bara formatbrott. *Mått för nivå K:* träffsäkerhet (Rätt), andel delvis rätt, **andel falsklarm på stycken utan fel** (andelen felfria stycken där modellen rapporterade minst ett fel), falsklarm per stycke, andel funna fel, formatbrott och tekniskt bortfall.

**Delen Detaljer har egen systemprompt och trappa** (`lagprovet` i `method.yaml`, koden i `method/lagprovet.py`). Användarmeddelandet är författningens rubrik, bestämmelsen (kapitel och paragraf) och stycket. Systemprompten för den kombinerade varianten, godkänd av metodägaren 2026-10-21:

> Du får ett stycke ur en svensk författning med författningens rubrik och paragraf. Stycket kan avvika från gällande lydelse på två sätt. Innehåll: högst en sak kan vara ändrad så att innebörden blir en annan, till exempel ett tal, en paragrafhänvisning, ska eller får, en negation, en myndighet eller aktör, eller senast och tidigast. Språk: det kan finnas upp till två språkfel, till exempel stavfel, fel böjning, särskrivning eller de/dem. Skriv varje avvikelse på en egen rad, antingen 'innehåll: <texten som den står i stycket> → <gällande lydelse>' eller 'språk: <fel> → <rätt>'. Ändra inte stil, ordföljd, skiljetecken eller stor och liten bokstav. Om du inte vet om innehållet stämmer, skriv raden 'innehåll: VET EJ'. Avsluta med en sista rad 'Svar: KLART'. Om stycket stämmer med gällande lydelse och inte har några språkfel, skriv bara 'Svar: STÄMMER'.

Varianten med bara innehåll, som inte valdes, har en kortare prompt med "ändrat: ... → ...", "Svar: ÄNDRAT", "Svar: STÄMMER" och "Svar: VET EJ" (`lagprovet.system_prompt`). *Motivering:* (1) prompten säger att högst en sak är ändrad och vilka slags ändringar det kan vara, så att uppgiften är definierad och inte blir en gissning om stil; (2) det fasta formatet med prefixen "innehåll:" och "språk:" gör att innehåll och språk kan rättas var för sig av kod; (3) "Svar: STÄMMER" är ett uttryckligt sätt att säga att lagtexten är korrekt återgiven, och en modell som påstår en ändring som inte finns gör samma sak som en modell som hittar på ett län för en påhittad kommun; (4) "innehåll: VET EJ" ger samma möjlighet att avstå som i delen Sverige.

**Rättning i delen Detaljer.** Ett par är de ord som skiljer vänster och höger sida åt, med normalisering av tal (räkneord blir siffror, "1 000" blir "1000", "%" blir "procent") och paragrafbeteckningar ("§" och "kap" behålls som ord). En avslutande förklaring i parentes på höger sida tas bort.

| Regel | Villkor | Utfall (innehåll) |
|---|---|---|
| L0 | Svaret är tomt | Ogiltigt |
| L1 | Tekniskt fel som kvarstår efter två omkörningar, eller att tokenbudgeten tog slut | Ogiltigt |
| L2a | Oförändrat stycke: inget innehållspar och inte VET EJ. Ändrat stycke: exakt facits par och inget annat | Rätt |
| L2b | Ändrat stycke: facits par plus andra par, eller rätt ställe men fel lydelse | Delvis |
| L3 | Ändrat stycke: STÄMMER, eller bara andra par | Fel |
| L3b | Oförändrat stycke: modellen påstår en ändring (falsklarm) | Fel |
| L4 | VET EJ utan innehållspar | Avstår |
| L5 | Ingen markörrad, inga par och ingen av fraserna "stämmer" eller "vet ej" | Ogiltigt |

I den kombinerade varianten rättas raderna med "språk:" som språkunderlagets korrekturläsning (avsnittet ovan), som ett eget, beskrivande mått. Falsklarm räknas per typ. Formatbrott är: ingen markörrad (KLART eller STÄMMER), STÄMMER tillsammans med rader, KLART utan rader, eller rader med pil som saknar prefix (de rättas inte). *Mått för delen Detaljer:* träffsäkerhet för innehållet (huvudmått), andel delvis rätt, andel falsklarm på oförändrade stycken, andel upptäckta ändringar (med och utan språkfel i samma stycke), andel avstår, språkets träffsäkerhet och falsklarm, formatbrott, tekniskt bortfall, och antal stycken där modellerna i stor enighet anger en ändring som facit saknar (möjliga källfel, som konsensusflaggan i delen Sverige).

**Normalisering i språkunderlaget:** gemener; markdown, citattecken och omgivande skiljetecken tas bort; å, ä, ö, é och bindestreck behålls (bindestrecket är det som rättas i sär- och sammanskrivning). För ordformer tas också en avslutande parentes bort ("gned (preteritum)") och ett inledande hjälpord ur en sluten lista (till exempel "har gnidit" rättas som "gnidit"). För flerval godtas bokstaven eller alternativets exakta text, och i ett kort svar en enda fristående bokstav ("jag tror att det är B", som garderingsregeln). "dom" i stället för "de" eller "dem" är Fel.

**Talregler** (för framtida sifferfrågor, används inte i nuvarande frågor). Rätt om svaret avrundat till angiven precision matchar facit och enheten stämmer. Punkt tolkas som decimaltecken, mellanslag och hårt mellanslag som tusentalsavgränsare. Motstridiga enheter (till exempel "tusen" eller "procent" när invånare efterfrågas) ger Fel.

**Rättare och validering.** Rättningen görs av kod: normaliserad matchning mot facit och alias. Svar som matchningen inte kan avgöra går till en jury av språkmodeller som ser facit och ett svar i taget, inga par och inga modellnamn. Juryn innehåller aldrig modellfamiljen som skrivit svaret.

Rättaren valideras före låsning på ett sätt som avviker från vår ursprungliga plan (två personer, minst 200 svar):
- Två oberoende rättare, A (huvudrättaren) och B (referensrättare), rättar alla svar.
- En person med svenska som modersmål märker alla svar där A och B skiljer sig, plus ett stratifierat slumpurval om 30 svar ur enigheterna (per modellfamilj och nivå). Högst 50 poster totalt. Slumpfrö: 20261014.
- En blind jury av tre språkmodeller från icke-Anthropic-familjer (OpenAI, Google, Mistral) bedömer samma poster.
- Det som går förlorat: det finns bara en människa, så kappa mellan människor kan inte beräknas. Vi redovisar kappa mellan människan och A, B och juryn. Gemensamma fel där både A och B har fel på samma sätt fångas bara av de 30 enigt rättade posterna. Noll fel i 30 ger en övre 95-procentig gräns på cirka 9,5 % för felfrekvensen.

## 8. Mått

Alla mått redovisas per modell, läge och nivå, med 95 % konfidensintervall, n och antal kluster.

**Nollnämnare ger "ej definierat", aldrig 0.** Det gäller varje mått och alla jämförelser. Skälet lagras bredvid.

| Mått | Definition | När nämnaren är 0 |
|---|---|---|
| Träffsäkerhet | Rätt / riktiga | Ej definierat om det saknas riktiga frågor |
| Andel delvis, fel, avstår, ogiltigt | Respektive utfall / riktiga. Summan av de fem andelarna är 100 %. | Som ovan |
| Felandel | (Fel + ogiltigt) / riktiga. Ogiltiga räknas som fel så att skräpsvar inte sänker felandelen. | Som ovan |
| Precision bland besvarade | Rätt / (rätt + delvis + fel) | Ej definierat om modellen aldrig svarar. Aldrig 0 %. |
| Index | 100 · (rätt − fel) / riktiga. Bara för jämförbarhet med andra tester. Ingen titel delas ut på index. | Som träffsäkerhet |
| AA-kvot | Fel / (fel + delvis + avstår + ogiltigt). Bara i rådatan, visas inte på sajten. Kallas inte "hallucinationsgrad": 1 fel av 100 svar ger 100 %, vilket vilseleder som rubrik. Ingen titel. | Ej definierat vid 100 % rätt |
| Felavdragspoäng | 100 · (rätt − p · fel) / riktiga för p ∈ {0,5, 1, 2}. Känslighetsanalys. | Som träffsäkerhet |
| Påhittade: korrekt avvisande, avstår, påhitt, ogiltigt | Respektive utfall / påhittade | Ej definierat om det saknas påhittade frågor |
| Andel korrekt beteende | (korrekt avvisande + avstår) / påhittade | Som ovan |
| Påhitt och korrekt beteende utan markerade namn | Som ovan, utan pilotspärrade påhittade namn. Redovisas bredvid huvudmåttet. | Som ovan |
| Formatbrott | Andel svar utan giltig markörrad, av svar utan tekniskt bortfall. Redovisas för alla modeller. | Ej definierat om alla svar är tekniskt bortfall |
| Tekniskt bortfall | Svar med tekniskt fel / alla svar. Redovisas per modell. | – |
| Pris per rätt svar | Summa kostnad / antal rätt. Valuta enligt Riksbankens kurs. | Ej definierat vid 0 rätt. Aldrig "billigast". |
| Tokens ut | Median och 90:e percentil per svar | – |
| Tokens ut per svar, medel | Summa tokens ut / antal svar. Resonemangstokens räknas in. | Ej definierat utan svar |
| Kostnad per 100 frågor | Summa kostnad / antal svar · 100, i kronor med den fasta kursen i `method.yaml` (`presentation.cost`) | Ej definierat utan svar |
| Kostnad per rätt svar, i kronor | Summa kostnad / antal rätt, med samma kurs | Ej definierat vid 0 rätt. Aldrig "billigast". |

Kostnaden redovisas per del, och kostnaden för påhittade frågor redovisas separat.

**Faktisk kostnad i testet.** Listpriset per token säger inte allt: en modell med högre pris per token kan bli billigare i praktiken om den använder färre tokens, till exempel för resonemang. Därför redovisar vi också tokens ut per svar, kostnad per 100 frågor och kostnad per rätt svar på huvudnivån, räknade av kod ur körningens loggade tokens och kostnad. Kronbeloppen räknas med en fast kurs från Riksbanken som står i `method.yaml`, så att de går att räkna om. Måtten påverkar inte rättning, jämförelser eller titlar.

**Tekniskt bortfall och budget.** Tokenbudgeten är densamma för alla. Om en modells bortfall överstiger 5 % stoppas publiceringen för den modellens siffror tills bortfallet redovisats och bedömts. Avvägning: samma budget gynnar inte modeller som använder många tokens, men en modell vars resonemang tar slut på svåra frågor får osäkrare siffror där. Det syns i tekniskt bortfall och i hur ofta bortfallet sker per nivå.

## 9. Körlägen

Varje fråga ställs fyra gånger: tre gånger i läge A och en gång i läge B. Dessutom körs en omformulering (A2) en gång på ett förregistrerat urval om cirka 20 % av frågorna. Läge A+ ingår inte (se nedan). Beslutet fattades 2026-10-16 och står i `method.yaml` under `modes`.

- **Läge A (huvudläge, likt AA), formulering A1, tre körningar per fråga.** "Om du inte vet svaret, svara VET EJ", med markörraden enligt avsnitt 7. Ingen information om poäng. Läget bär huvudtalet (avsnitt 10), och alla titlar och placeringar bygger på det. *Varför tre körningar:* en modell svarar inte alltid likadant på samma fråga, och ett enda svar säger då för lite. Tre körningar medelvärdesbildas per fråga (avsnitt 12), vilket ger ett stabilare värde per fråga och visar hur ofta svaret varierar. Systemprompten i avsnitt 7 är den som användes i utvecklingskörningarna, och ordalydelsen är fastställd i låsfilen.
- **Läge B (vanlig fråga), en körning per fråga.** Ingen instruktion om avstående: frågan ställs som en vanlig användare skulle ställa den, utan att modellen får höra att den får svara "vet ej". *Varför:* skillnaden mellan A och B visar hur mycket en modells beteende styrs av instruktionen att få säga "vet ej". Läge B är en känslighetsanalys och inte ett huvudresultat, så en körning räcker. Systemprompten i läge B är: "Avsluta med en sista rad på formen 'Svar: <ditt svar>'."
- **Läge A2 (omformulering), en körning, bara för ett urval om cirka 20 % av frågorna.** Samma uppgift som läge A, formulerad med andra ord. *Varför:* för att se om resultatet beror på just vår formulering. En omformulering på ett urval räcker för att upptäcka en sådan effekt, men är för liten för att skilja modeller åt, och A2 används därför bara för att jämföra mot A1 och inte för några egna rangordningar eller titlar. Urvalet är deterministiskt och förregistrerat: en fråga ingår om `sha256(salt + ":" + fråge-id)`, de första 15 hextecknen läsna som tal delat med 16^15, är mindre än 0,20. Det beror alltså bara på fråge-id och ett salt som står i `method.yaml`, aldrig på svar, modell eller frågetext, och gäller riktiga och påhittade frågor lika. Andelen blir cirka 20 %, inte exakt. Stabiliteten mellan A1 och A2 redovisas på samma frågor, det vill säga urvalet. Systemprompten i A2 är: "Håll svaret kort. Den sista raden ska alltid vara 'Svar: ' följt av ditt svar. Vet du inte svaret skriver du 'Svar: VET EJ'."
- **Läge A+ (poängläge efter Kalai m.fl.) ingår inte.** *Varför:* med ett tredje utfall (delvis rätt) blir tröskeln på 50 % bara ungefärlig, och läget skulle kosta en omgång till utan att visa något som läge A, läge B och felavdragspoängen i avsnitt 11 inte redan visar.

> **Varför ställer vi varje fråga fyra gånger?**
>
> En AI svarar inte alltid likadant. Ställer du samma fråga två gånger kan du få två olika svar, och då säger ett enda svar för lite. Därför ställer vi varje fråga tre gånger på exakt samma sätt (läge A) och räknar hur ofta modellen svarar rätt. Den fjärde gången ställer vi frågan så som en vanlig användare gör, utan att be modellen säga "vet ej" när den inte vet (läge B). Då syns det om modellen beter sig annorlunda när den inte får den chansen. På ungefär var femte fråga ställer vi dessutom samma fråga med andra ord (läge A2), för att se att resultatet inte hänger på just våra formuleringar.
>
> Det är alltså inte fyra olika frågor, utan samma fråga fyra gånger. Antalet frågor är detsamma, och osäkerheten räknas på frågorna och inte på antalet svar. Tre körningar ger därför inte tre gånger så säkra siffror: de gör varje frågas värde stabilare, men antalet olika frågor och län sätter ändå gränsen för hur små skillnader vi kan påvisa (avsnitt 12).

- Läge B och omformuleringen A2 visar hur resultaten ändras med instruktionen och med formuleringen, och är därför känslighetsanalyser genom nya körningar.

**Körlägen i delen Detaljer.** Läge A en gång per stycke, med systemprompten för Detaljer (beslut 2026-10-21). Ett förregistrerat, deterministiskt urval om 20 % av huvudstyckena (de 40 av 200 vars sha256(salt + ":" + id) är lägst, saltet i `method.yaml`) körs en gång till, och stabiliteten redovisas som andelen lika utfall mellan de två körningarna per modell. Inget läge B och ingen A2.

**Körlägen i språkunderlaget.** Språkunderlaget körs i läge A, tre körningar per stycke och fråga (beslut 2026-10-21), med korrekturläsningens systemprompt på nivå K och läge A:s systemprompt på grundnivån. Läge B och A2 körs inte för språkunderlaget i version 1.0 (`subtests.sprakprovet.modes` i `method.yaml`). *Varför:* läge B mäter hur mycket instruktionen att få säga "vet ej" styr beteendet, och det är Sverigedelens fråga, inte språkunderlagets: i språkunderlagets pilot var avstående ovanligt. A2 skulle pröva en annan formulering av frågan, men språkunderlagets frågor är redan korta och formaliserade (en uppmaning och en bokstav eller ordform), så en omformulering ändrar mindre än i ett faktaprov. Metodägaren beslutade så 2026-10-21, med förbehållet att den andra piloten inte visar skäl till något annat.

## 10. Huvudtal och titlar

**Huvudtalet** är fördelningen över alla fem utfall (Rätt, Delvis, Fel, Avstår, Ogiltigt) som stapel, plus träffsäkerhet och felandel med konfidensintervall, för läge A på huvudnivån.

**Ordning och delad placering.** Resultatöversikten visar modellerna efter träffsäkerhet i läge A på huvudnivån, högst först, med felandelen bredvid. Varje modell får en placering som intervall: den lägsta siffran är 1 plus antalet modeller som är statistiskt säkert bättre, den högsta är antalet modeller minus antalet som är statistiskt säkert sämre. "Statistiskt säkert" betyder en stödd jämförelse enligt avsnitt 12. Modeller vars skillnad inte är statistiskt säkerställd får därför delad placering, till exempel "1–4". En modell utan någon stödd jämförelse får hela intervallet och redovisas som att den inte går att placera. En modell som kontrollen av körningens kvalitet (grind G6) har satt i väntan jämförs inte och får ingen placering, och antalet modeller räknar då bara de jämförda. Samma regel används för felandelen på modellsidorna. Placeringen är ingen titel. Regeln står i `method.yaml` under `presentation`.

**Journalistinstruktion** (så får resultatet återges): *"I det svenska faktaprovet svarade modell X rätt på A %, fel eller ogiltigt på B % och avstod från C %. Övriga D % var delvis rätta."*

**Titlar.** Titlar delas ut av kod ur de förregistrerade reglerna, inte av en redaktörs bedömning.
- **"Bäst i faktaprovet (läge A)"** får bara en modell som har statistiskt säkert *högre träffsäkerhet och lägre felandel* än varje annan modell (båda måtten, mot varje annan). Det finns ingen titel för hela portföljen, eftersom ett prov inte säger vad som är bäst överlag.
- **Annars** får "högst träffsäkerhet" och "lägst felandel" bara nämnas tillsammans, med den symmetriska meningen: **"Träffsäkerhet gynnar modeller som gissar, felandel gynnar modeller som avstår."**
- **Ingen titel** delas ut på index eller AA-kvot.
- Skälet: rubrikerna blir mindre slagkraftiga, men regeln är symmetrisk mot de båda beteendeprofilerna.

**Vad som får påstås om en jämförelse** avgörs av avsnitt 12. "Ingen skillnad" får aldrig skrivas utifrån att en skillnad inte blev signifikant.

## 11. Känslighetsanalyser

Förregistrerade, av två slag.

**Omräkning av samma svar.** En jämförelse håller bara om *samma riktning och ett oklustrat 95 %-intervall utan 0* gäller i varje variant:
- felavdrag p ∈ {0,5, 1, 2}
- garderingsregel enligt AA i stället för SimpleQA
- utan de frågor som granskningen efter körningen har flaggat (alla flaggor, oavsett beslut)
- för riktiga frågor: bokstavlig rättning av hela svarstexten i stället för markörraden
- för påhittade namn dessutom: utan de pilotspärrade namnen (avsnitt 6)

Leverantörskänslighet (samma frågor via minst två leverantörer för modeller som körs via en tredjepartsleverantör, avsnitt 3) redovisas bredvid. Den ingår inte i villkoret ovan.

**Beteende genom nya körningar:** läge B och omformuleringen A2 (avsnitt 9).

## 12. Statistik

Detaljerna och formlerna finns i `STATISTIK.md`, som är en exakt kopia av den fil som låses. Sammanfattning:

- **Enhet:** frågan. Upprepade körningar av samma fråga medelvärdesbildas inom fråga och modell. Tre körningar av samma fråga är inte tre frågor. Läge A2 körs bara på urvalet (avsnitt 9). Jämförelser i A2 gäller därför bara urvalet, och samma krav på antal entiteter och länskluster gäller där.
- **Standardfel och intervall är klustrade på län.** Klusternyckel är det län frågan gäller (21 län ger högst 21 kluster). För påhittade namn finns inget län, så varje påhittat namn är ett eget kluster. Variansen är en enkelvägs klusterrobust varians med småprovsfaktor G/(G−1), och kritiskt värde är t med df = antal län − 1. Formuleringsmall och entitet sparas som attribut men klustrar inte. Skälet var att ρ för mall var nära noll i båda utvecklingskörningarna medan fyra mallar gav en minsta påvisbara skillnad på 12–41 procentenheter.
- **Två egna val som ska granskas:** (1) variansen får aldrig vara lägre än den oklustrade variansen (golv), så att klustring inte ger falsk precision med få kluster, och (2) när alla utfall är 0 eller alla är 1 används Wilson-intervall på antalet frågor, eftersom klustervariansen då inte är definierad. Wilson-intervallet ignorerar klustring och kan vara för smalt om ρ > 0.
- **Jämförelser:** parvisa differenser på frågenivå, med samma klustrade formel. Familjen är alla parvisa jämförelser inom läge × nivå × mått (8 modeller ger 28 par). **Holm-korrektion** (step-down) görs över hela familjen. Okorrigerade intervall redovisas också, märkta som sådana.
- **Minsta påvisbara skillnad (MDE):** (t_{1−α/(2m)} + t_{effekt}) · SE, med 80 % effekt, skattad ur parets egen standardavvikelse. Den redovisas för varje jämförelse. När den är större än den skillnad som observerats skriver vi "för få frågor för att skilja åt".
- **Ekvivalens (TOST):** 90 %-intervallet för differensen ligger inom ±5 procentenheter.
- **En jämförelse är "stödd" först när alla fem villkor är uppfyllda:** Holm-justerat p < 0,05, skillnad på minst 5 procentenheter, minst 30 olika entiteter, minst 15 länskluster, och att känslighetsanalyserna håller. Annars är den inte stödd, eller "otillräckligt underlag" när klusterkraven inte nås.
- Med klustring på län kan antalet kluster aldrig bli större än 21, hur många frågor som än ställs. Fler frågor hjälper bara om ρ för län är liten.
- Data per fråga sparas och publiceras enligt avsnitt 13. Metoden bygger inte på IRT.
- **Delen Detaljer har inga län.** I huvudkörningen högst åtta stycken per känd lag och ett per övrig författning (i piloten tre), och stycket är enheten (oklustrad varians, df = n − 1); en känslighetsanalys klustrar på författning. Att flera stycken ur samma kända lag kan hänga ihop redovisas som en begränsning; kända lagar och den långa svansen redovisas var för sig.
- **Språkunderlaget har inga län.** På huvudnivån K kommer varje stycke från en egen författning, så styckena är oberoende och stycket är enheten (ingen klustring, df = n − 1). På grundnivån G är kandidatnyckeln regeltypen (17 typer). Regeln, förregistrerad: klustra på regeltyp om ρ för regeltyp skattad i språkunderlagets pilot är större än 0, annars är frågan enheten. Golvet vid oklustrad varians gäller i båda fallen. Beslutet fattas en gång, på pilotens svar, och står i `method.yaml` (`subtests.sprakprovet.clustering`). Kravet för en stödd jämförelse är då minst 15 regeltypskluster i stället för 15 länskluster, och minst 30 frågor. Detaljer och motivering i `STATISTIK.md`.

## 13. Förregistrering, låsfil och publicering

### Förregistrering

Metod 1.0 är förregistrerad på följande sätt. Före huvudkörningen skrevs en låsfil, och dess hash, **20049643e10befd569ef56548986b25790f6f54b4847d32cda7412752024ba8f**, publicerades **2026-10-21 16:00 UTC (18.00 svensk tid)**, alltså innan körningen gjordes och innan några svar fanns. Låsfilen innehåller en hash per låst fil. De filer som ingår i det här paketet (`method.yaml` och `STATISTIK.md`) kan kontrolleras mot den med hjälp av `CHECKSUMS.md`. Körningen stoppas automatiskt om något som ingår i låsfilen har ändrats efter låsningen. Resultaten publiceras tillsammans med metoden, och det var ingen granskningsperiod före första körningen. Synpunkter som kommer in efter låsningen kan därför inte ändra Metod 1.0 eller dess resultat. De prövas inför Metod 1.1 och loggas öppet (avsnitt 18).

### Låsfil och publicering

- **Låsfil.** Före första körningen skrivs en låsfil med hashvärden för frågor, facit, regler, rättningskod, poängformel, prompter, modellista, åtkomstvägar, parametrar och analysplan. Körningen stoppas om något har ändrats. Hashvärdena för de två filer som publiceras här (`method.yaml` och `STATISTIK.md`) står i `CHECKSUMS.md` och ska stämma med den publicerade låsfilen.
- **Allt under metodens katalog ingår i låset.** Det som ändras efter låsning kräver ett nytt metodversionsnummer.
- **Alla förregistrerade körningar publiceras.** Offentliga frågor: råsvaren publiceras direkt. Privata frågor: statistik per fråga och hashvärden publiceras direkt, råsvaren när frågorna pensioneras. Alltid med leverantörs-id, kostnad och ändringar.
- **Utvecklingsförsök** (piloterna i oktober 2026) publiceras efter lanseringen som en märkt utvecklingslogg, utan rangordning och aldrig som resultat. De är inte en del av det publicerade paketet.
- **"Preliminärt"** gäller bara snabbtest av nya modeller, tills en fullständig körning med tre omgångar är gjord. Lanseringens resultat är inte preliminära.
- **Ändringslogg.** Varje ändring dateras och motiveras. Se `CHANGELOG.md`.

## 14. Rättelseprotokoll och svarsrätt

**Rättelser.**
- Sakfel i facit eller kod rättas öppet, loggas med datum och orsak, och både gammalt och nytt resultat visas. Rättelserna samlas i den offentliga rättelseloggen (`/rattelser/`), och en rättelse markeras också på de sidor den gäller.
- Regeländringar inom en period görs aldrig. En ny regel gäller först från nästa period och nästa metodversion.
- Ingenting tas bort i tysthet. Det krävs också för att inte vilseleda läsaren.
- Den som hittar ett sakfel kan anmäla det till hej@aitestet.se när som helst, eller enligt avsnitt 18 för synpunkter på metoden. Anmälan och vårt svar loggas offentligt. Ett sakfel rättas direkt enligt reglerna ovan, utan att vänta på nästa version.

**Inget förhandsvarsel, och svarsrätt.**
- Vi skickar inget förhandsvarsel till leverantörerna före publicering. Alla leverantörer behandlas lika.
- Sakfel kan anmälas till hej@aitestet.se när som helst. Rättelser görs öppet enligt rättelsereglerna ovan.
- Ett svar (genmäle) från en leverantör publiceras i sin helhet, i rättelseloggen och med länk från de sidor det gäller.
- Ett svar ger ingen rätt att påverka frågor, regler eller resultat. Det ger rätt att anmäla sakfel och att få sitt svar publicerat.

## 15. Avsändare, intäkter och märkning

**Avsändare.** AI-testet drivs av Satori ML AB, som hjälper företag att välja och införa AI. Vi arbetar främst med Claude och ChatGPT. Alla modeller testas på samma sätt, oavsett vilka vi själva arbetar med, och metoden är öppen för granskning. Hör gärna av er om ni har frågor eller vill ha hjälp.

**Så här hålls testet rakt:**
- Ingen betald placering, inga sponsrade resultat och ingen affiliate.
- Inga skräddarsydda tester säljs till en leverantör som rankas, utan offentlig notering.
- Ingen kund påverkar frågor, vikter eller tidpunkt.
- Försäljning av licenser och tjänster ligger på en annan del av sajten, skild från resultaten.

**Märkning.** Text på sajten som är AI-assisterad märks som det. Publicerade råsvar från modellerna märks som AI-genererade.

**Granskning.** Metoden är öppen, och det är öppenheten som ska göra den trovärdig. Forskare och leverantörerna själva är välkomna att granska den (avsnitt 18).

## 16. Användning av AI och mänskligt godkännande

- **Mätningen** görs av kod. Frågor, facit och påhittade namn genereras av kod ur källdata. Rättningen är en deterministisk funktion av svarstexten, med jury av språkmodeller bara för svar som matchningen inte kan avgöra.
- **Texter** på sajten (rapporter, förklaringar, nyhetsanalyser) skrivs med hjälp av AI-agenter. Varje tal i en text hämtas ur resultatfilerna med ett id och skrivs aldrig av en språkmodell. Ett påstående om en jämförelse kräver att jämförelsen är "stödd" enligt avsnitt 12.
- **Kontroll före publicering.** Två modeller från olika familjer kontrollerar varje text mot en förregistrerad checklista (avsändare synlig, inget vilseledande, jämförelser sakliga och kontrollerbara, märkning av AI-text, källa och datum för påståenden om leverantörer, inga personuppgifter, inget som kan läsas som juridisk rådgivning). Resultatet är en lista med flaggor. Den ersätter inte en jurist.
- **En människa godkänner varje publicering.** En namngiven person (Satori ML AB) har redaktionellt ansvar. Ingen text och ingen titel publiceras utan det godkännandet. Beslut om flaggade facitfrågor fattas av en människa.

## 17. Begränsningar och öppna frågor

**Begränsningar (de ska stå på sajten):**
- **Vad provet täcker.** Svensk faktakunskap, avstående och påhitt, i standardläge via API. Inte allmän förmåga. Variationen mellan modeller i ett faktaprov kan till stor del förklaras av en enda gemensam faktor.
- **Provets form.** Träffsäkerheten gynnar modeller som gissar. Felandelen gynnar modeller som avstår. Båda visas och titelregeln är symmetrisk, men ingen regel tar bort snedheten helt.
- **Nivåerna har formats med pilotmodellerna som referens.** Två utvecklingskörningar med åtta modeller användes för att välja och justera nivåerna (avsnitt 5 och `CHANGELOG.md`). Det betyder att provets svårighet inte är oberoende av just de modellerna. Nivåerna är låsta för den här metodversionen så att de inte justeras vidare mot nya modeller, men urvalseffekten finns kvar. Eftersom metoden låstes utan en granskningsperiod före körningen har ingen utomstående hunnit pröva nivåerna före första resultaten. Utvecklingsdata används bara som utvecklingslogg och är aldrig resultat.
- **Standardläget** ger modellerna olika beräkningsbudget (avsnitt 2).
- **Tokenbudgeten** på 16 000 är lika för alla men drabbar resonemangsmodeller mer på de svåraste frågorna. Det syns i tekniskt bortfall.
- **Svensk träningsdata** ger fördel åt modeller som tränats på mycket svensk text. Det är en del av det som mäts.
- **Rättarens fel** ingår inte i konfidensintervallen. Valideringen bygger på en människa och ett stickprov, inte på två människor (avsnitt 7).
- **Påhittade namn.** Existenskontrollen är automatisk. Vi kan inte utesluta att något genererat namn motsvarar ett verkligt men dåligt dokumenterat ställe. Pilotspärren och känslighetsanalysen finns för det.
- **Instruktionen om avstående.** Systemprompten ber modellen skriva "VET EJ" när den inte vet. Den uppmanar därför inte till "finns inte", och kategorin "korrekt avvisande" kan bli nästan tom. Måttet som skiljer är då andelen påhitt, vilket är en egenskap hos prompten.
- **Antalet kluster är högst 21.** Det begränsar hur små skillnader som kan påvisas.
- **Utvecklingskörningarna hade en körning per fråga.** Deras intervall säger mer om metoden än om modellerna.
- **Inget förhandsvarsel.** Leverantörerna ser sina resultat först när de publiceras. Ett sakfel kan därför vara publicerat en tid innan det anmäls och rättas (avsnitt 14).
- **Källsvaghet på nivå 3.** Småortens namn finns bara i Wikidata. Kontrollen mot SCB:s data är oberoende på geometrin men inte på själva punkten.
- **Språkunderlaget mäter korrekturläsning av bestämda fel, inte skrivande.** Det säger inget om hur väl en modell skriver ett mejl eller en rapport. De enkla frågorna var mättade i den första piloten (avsnitt 5).
- **Delen Detaljer mäter minne av lagtext, inte juridisk förmåga.** En modell som kan tillämpa lagen men inte minns den exakta lydelsen kan missa en ändring. Urvalet utesluter paragrafer som ändrats de senaste tre åren, men en modell kan ändå minnas en äldre lydelse. Ändringarna är konstruerade av kod och är inte alltid lika trovärdiga.
- **Korrekturläsningens text är lagtext.** Författningsspråk är formellt och tätt. En modell kan uppfatta en ålderdomlig men korrekt konstruktion som fel; det räknas som falsklarm. Det är en del av det som mäts, men det gynnar inte nödvändigtvis den modell som är bäst på vardagstext.
- **Felen är konstruerade av kod** och fördelas inte som i verkliga texter. Stavfel dominerar, eftersom de går att sätta in i nästan alla stycken.
- **Språkunderlagets facit kan vara för snävt.** Facit är de former som SALDO och Wikidata anger. En form som är godtagbar men saknas i båda källorna rättas som fel. Piloten hittade ett sådant fall och ett fall där facit troligen var fel (en sammansättning med ett initialord som uttalas som ett ord). Granskningen efter körningen (avsnitt 4) gäller också språkunderlaget.
- **SALDO är från 2017.** Nyare rekommendationer kan saknas, och äldre varianter finns med och godkänns.

**Där vi särskilt vill ha synpunkter (de prövas inför version 1.1):**
1. Är **länsklustringen** med golv vid oklustrad varians och Wilson-intervall vid gränsen en rimlig behandling när antalet kluster är högst 21?
2. Är kravet på **minst 15 länskluster** (vårt egna val) för "stödd" lagom?
3. Är **markörraden** `Svar: <svar>` en rättvis regel för modeller med olika svarsstil, och är bokstavlig rättning den rätta känslighetsanalysen?
4. Är **tokenbudgeten** på 16 000, lika för alla, ett rimligt sätt att behandla resonemangsmodeller?
5. Är **rättarvalideringen** (en människa, 30 enigt rättade svar plus alla oenigheter, jury av tre) tillräcklig, och vad bör läggas till?
6. Är **nivåindelningen** ett försvarbart sätt att hantera mättnad, med tanke på att den formats med pilotmodellerna?
7. Räknas **"vet ej" och "finns inte"** rimligt som lika korrekt beteende på påhittade namn?
8. Är **titelreglerna** och den symmetriska meningen rättvisa mot båda beteendeprofilerna?
9. Är **avsändartexten och intäktsreglerna** i avsnitt 15 tillräckliga, och vad saknas?

## 18. Så lämnar du synpunkter

- Skapa ett ärende (issue) i det offentliga arkivet: https://github.com/SatoriML/aitestet-metod. Instruktioner och en mall finns i `CONTRIBUTING.md`. Du kan också skriva till hej@aitestet.se med synpunkter på metoden eller om fel du hittat.
- Ange vilket avsnitt synpunkten gäller, vad du föreslår och varför.
- **Synpunkter tas emot löpande och prövas inför nästa version. Alla synpunkter och svar loggas öppet.**
- Vi anger för varje ärende om det lett till ändring, delvis ändring eller ingen ändring, med skäl. Sakfel i facit eller kod hanteras enligt rättelseprotokollet (avsnitt 14) och inte först i nästa version.
- Ändringar som följer av synpunkter skrivs in i ändringsloggen som en del av Metod 1.1, med datum och motivering. De ändrar inte redan publicerade resultat i version 1.0.
- Du får gärna ange om du vill namnges. Skriv inga personuppgifter i ärendena, de är offentliga.

## 19. Version och datum

| | |
|---|---|
| Metodversion | Metod 1.0 |
| Status | Låst före huvudkörningen. Hash och tidpunkt: 20049643e10befd569ef56548986b25790f6f54b4847d32cda7412752024ba8f, 2026-10-21 16:00 UTC (18.00 svensk tid) |
| Datum för texten | 2026-10-20 |
| Maskinläsbar form | `method.yaml` (exakt kopia) |
| Statistikbeskrivning | `STATISTIK.md` (exakt kopia) |
| Kontrollsummor | `CHECKSUMS.md` |
| Ändringshistorik | `CHANGELOG.md` |
| Nästa version | 1.1, med synpunkter som kommit in löpande och eventuellt en del om instruktionsföljning |
