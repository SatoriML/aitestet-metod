# Ändringslogg för AI-testets metod

Varje post har datum, beslut och skäl. Poster märkta **pilotdriven** är ändringar som gjordes efter att vi sett resultat från våra två utvecklingskörningar (piloterna). Det betyder att de formades med kännedom om hur åtta modeller betedde sig i just de körningarna. Det säger vi uttryckligen eftersom det är en urvalseffekt som läsaren ska kunna väga in.

Utvecklingskörningarna publiceras inte som resultat. Siffror som nämns här är sammanfattningar av det som låg bakom ett beslut, utan modellnamn och utan frågor.

Senaste versionen står överst.

---

## Metod 1.0, låsning

**Ingen granskningsperiod före första körningen (beslut av metodägaren).** Metod 1.0 låses innan huvudkörningen. Låsfilens hash (20049643e10befd569ef56548986b25790f6f54b4847d32cda7412752024ba8f) publiceras före körningen (2026-10-21 16:00 UTC (18.00 svensk tid)), och resultaten publiceras tillsammans med metoden. Synpunkter tas emot löpande och prövas inför Metod 1.1. Detta ersatte en tidigare plan om att publicera metodutkastet och vänta minst två veckor på synpunkter före låsning. Beslutet fattades av metodägaren. Följden är att metoden låses utan att utomstående hunnit granska den, och det redovisas som en begränsning i `METOD.md` (avsnitt 17).

**Instruktionsföljning ingår inte i version 1.0.** Instruktionsföljning och strukturerad utmatning kan komma som ett eget delprov i en senare version. Version 1.0 har två delprov: Sverigeprovet och Språkprovet (se posten 2026-10-20).

---

## 2026-10-21, metoden låst lokalt

- **Systemprompten för Detaljer** (kombinerad variant) är godkänd av metodägaren 2026-10-21 (`METOD.md` avsnitt 7).
- **Systemprompterna för delen Sverige** är förregistrerade: läge A (godkänd 2026-10-15), A2 och B (godkända 2026-10-16). Texterna står i `METOD.md` avsnitt 7 och 9 och i `method.yaml`.
- **Interna spår** (namn i beslutsrader, TODO-markeringar, interna rapporthänvisningar, pilot-id och interna sökvägar) är omskrivna till neutral text i `method.yaml` och `STATISTIK.md`, så att de exakta kopiorna kan publiceras.
- **Status:** `method.yaml` har status `locked`. Låsfilen skrivs om, och dess hash publiceras i det publika arkivet före huvudkörningen av Detaljer. Därefter ändras metoden bara med ett nytt versionsnummer.
- **Beslut:** Joel, 2026-10-21.

---

## 2026-10-21, namn och delar, huvudkörningen för Detaljer

- **Namn:** testet heter AI-testet, och upplagorna heter till exempel "AI-testet, hösten 2026". Testet består av delar: **Detaljer** ("Märker AI:n när något är fel i en text? Mätt på svensk lagtext", tidigare kallad Lagprovet), **Sverige** ("Vet AI:n eller hittar den på?", tidigare Sverigeprovet) och **Kostnad** (uppgiftsmätningen). Språkunderlaget, tidigare kallat Språkprovet, är ingen egen del. Det finns inget sammanvägt totalresultat förrän en viktningsregel är förregistrerad.
- **Detaljer, huvudkörningen:** 200 nya stycken, balanserade över de fyra kategorierna och mellan kända lagar och övriga författningar, med rättsområde per stycke. 20 % körs en gång till för att mäta stabilitet. Dessutom 50 stycken ur nyligen ändrade bestämmelser, som redovisas separat och med den tidigare lydelsen sparad. Ny tvåkällskontroll mot lagen.nu. Läge A en gång per stycke, och tokenbudgeten 16 000.
- **Pilotens tre konsensusflaggor** stämde med gällande lydelse; det var modellerna som hade fel (i ett fall med en äldre lydelse).
- **Beslut:** Joel, 2026-10-21.

---

## 2026-10-21, Lagprovet (nu delen Detaljer), 16 000 tokens

- **Nytt delprov, Lagprovet:** en paragraf ur gällande svensk författning, antingen oförändrad eller med en ändring av innebörden som kod har gjort (ett tal, en hänvisning, ska/får, en negation, en myndighet eller aktör, senast/tidigast). Modellen ska säga om texten stämmer med gällande lydelse och vad som i så fall är ändrat. I den valda kombinerade varianten finns också 0–2 språkfel i hälften av styckena; innehållet är huvudmåttet och språket ett beskrivande mått. Paragrafer som ändrats de senaste tre åren används inte. Kända lagar för företag och övriga författningar redovisas var för sig.
- **Språkprovet** blir beskrivande (korrekturläsningen och de enkla frågorna); Lagprovet är det prov med svensk text som har huvudnivå.
- **Tokenbudget:** 16 000 för alla modeller i alla prov (tidigare 8 000).
- **Korrekturläsningens systemprompt** är godkänd. Lagprovets systemprompt är ett förslag.
- **Beslut:** Joel, 2026-10-21.

---

## 2026-10-21, Språkprovet blir korrekturläsning (pilotdriven), spärrlista

- **Vad:** Språkprovets huvudnivå är nu korrekturläsning: ett stycke ur svensk författningstext där kod har satt in 0–3 fel, och modellen ska ange exakt de felen i ett fast format. Stycken utan fel ingår, och andelen falsklarm redovisas. De enkla frågorna från den första piloten är kvar som beskrivande grundnivå. En ny systemprompt för korrekturläsningen är föreslagen (`METOD.md` avsnitt 7).
- **Varför (pilotdriven):** i den första piloten låg alla 18 modeller över taket på 90 % på de enkla frågorna. Att hitta fel i löpande text utan att hitta på fel är svårare och närmare hur språkmodeller används i företag.
- **Regeln om en pilot per prov** tillämpades inte, eftersom metoden ännu inte var publicerad. Den gäller efter lansering.
- **Spärrlista:** alla generatorer prövar ord, namn och stycken mot en egen lista över olämpliga ord (`METOD.md` avsnitt 4). Två facitfel från den första piloten är rättade i generatorn (förkortningar bara med versaler som uttalas bokstav för bokstav; ett olämpligt verb spärras).
- **Modellistan** är bekräftad: 18 modeller, med GPT-5.6 Sol som OpenAI:s förra generation och utan Gemini 3.5 Flash-Lite.
- **Körlägen för Språkprovet:** läge A tre gånger; inget läge B och ingen A2.
- **Beslut:** Joel, 2026-10-21.

---

## 2026-10-20, modellurval, Sverigeprovets frågor och Språkprovet

- **Två delprov:** Sverigeprovet (svensk faktakunskap) och Språkprovet (korrekt svensk skrift: böjning, genus, stavning, sär- och sammanskrivning, ordval). Beteckningarna "delprov A/B" används inte längre, eftersom A är ett körläge. Instruktionsföljning ingår inte i version 1.0.
- **Modellurvalet är en förregistrerad regel** (`METOD.md` avsnitt 3, `model_selection` i `method.yaml`): för Anthropic, OpenAI och Google alla aktuella modeller i huvudlinjen per nivå plus den mest kapabla modellen i förra generationens sista version, för övriga leverantörer flaggskeppet, och varianter av samma modell räknas inte. Claude Opus 4.5 är ett öppet redovisat tillägg utanför regeln. 18 modeller. Regeln ersätter den tidigare formuleringen om "mest kapabla modell, standardmodellen i företagsprodukten, minst en europeisk och två öppna". Underlaget är OpenRouters katalog 2026-10-20, sparad med hash.
- **Sverigeprovets frågor till huvudkörningen** är genererade (40, 100, 400 och 60 per nivå), med samma generator och kontroller som i utvecklingskörningarna, utan någon fråga, entitet eller påhittat namn från dem. 20 % offentliga, 25 % ankare och resten roterande. Planen på 1 000 riktiga och 200 påhittade frågor är ersatt av dessa antal.
- **Språkprovet** har egen beslutstrappa, egna källor med licens (SALDO, CC BY 4.0; Wikidata, CC0; egen text) och 17 regeltyper. Skyddade ordlistor används inte. Rättningen görs av kod.
- **Statistik utan län (pilotdriven):** för Språkprovet klustras standardfelen på regeltyp om ρ för regeltyp i piloten är större än 0, annars är frågan enheten. Piloten gav ρ ≈ −0,007, så frågan är enheten. Golvet vid oklustrad varians gäller som tidigare.
- **Språkprovets pilot (pilotdriven):** 150 frågor, 18 modeller. Provet var mättat (alla modeller över 90 %). Piloten hittade också ett troligt facitfel och ett facit som kan vara för snävt. Vad som görs med mättnaden (en omdefiniering enligt regeln, eller publicering som mättat) beslutas före låsning.
- **Körlägen för Språkprovet:** läge A tre gånger per fråga; B och A2 körs inte (förslag, ska bekräftas).
- **Beslut:** Joel, 2026-10-20 ("Yes vi kör"), för delproven, modellistan och frågorna. Tolkningarna i modellurvalet, Språkprovets körlägen och vad som görs med mättnaden är öppna tills metodägaren bekräftat dem.

---

## 2026-10-16, körlägen, sajtens namn och avsändartext

- **Vad, körlägen:** läge A (huvudläget, formulering A1) körs tre gånger per fråga och läge B (vanlig fråga) en gång. Omformuleringen A2 körs en gång på ett förregistrerat, deterministiskt urval om cirka 20 % av frågorna. Läge A+ (poängläge efter Kalai m.fl.) stryks. Tidigare var planen A1 tre gånger, A2 en gång på alla frågor, A+ en gång och B tre gånger. Se `METOD.md`, avsnitt 9, med motivering per läge och en kort ruta, "Varför ställer vi varje fråga fyra gånger?".
- **Varför:** en körning per fråga säger för lite när modeller kan svara olika på samma fråga, och tre körningar i huvudläget ger ett stabilare värde per fråga. Läge B och A2 är känslighetsanalyser och behöver inte köras lika många gånger. A2 på ett urval räcker för att se om resultatet hänger på formuleringen. A+ visar inget som läge A, läge B och felavdragspoängen inte redan visar, och hade kostat en omgång till.
- **Urvalet till A2:** en fråga ingår om `sha256(salt + ":" + fråge-id)`, de första 15 hextecknen läsna som tal delat med 16^15, är mindre än 0,20. Saltet och andelen står i `method.yaml` (`modes`, läge A2, `sample`). Urvalet beror bara på fråge-id, aldrig på svar eller modell.
- **Vad som inte ändras:** inga mått, trösklar, jämförelseregler eller titlar. Titlar och placering bygger fortfarande bara på läge A. A2 används för att jämföra mot A1 och inte för egna rangordningar. Id:t för huvudläget är fortfarande `A` i resultatfilerna (det är formulering A1).
- **Sajtens namn:** texterna i metodpaketet talar nu om AI-testet i stället för AI-granskaren. Sverigeprovet behåller sitt namn som delprov. Rättelseadressen är nu hej@aitestet.se (i stället för adressen på den tidigare domänen) och står i `publication.corrections_email` i `method.yaml`. Den gäller också för frågor och synpunkter, vid sidan av ärenden i arkivet. Äldre poster nedan nämner den gamla adressen utan att ange den, och står kvar som historik.
- **Avsändartext:** avsnitt 15 i `METOD.md` har ersatt jävsdeklarationen med en avsändartext: "AI-testet drivs av Satori ML AB, som hjälper företag att välja och införa AI. Vi arbetar främst med Claude och ChatGPT. Alla modeller testas på samma sätt, oavsett vilka vi själva arbetar med, och metoden är öppen för granskning. Hör gärna av er om ni har frågor eller vill ha hjälp." Intäktsreglerna är kvar. Avsnittet om en framtida extern underskrift är struket.
- **Beslut:** Joel, 2026-10-16.

---

## 2026-10-15, presentationsregel för resultatöversikten

- **Vad:** resultatöversikten visar modellerna efter träffsäkerhet (läge A, huvudnivån), högst först, med felandelen bredvid, i stället för i bokstavsordning. Varje modell får en placering som intervall. Den lägsta siffran är 1 plus antalet modeller som är statistiskt säkert bättre, den högsta är antalet modeller minus antalet som är statistiskt säkert sämre. Modeller vars skillnad inte är statistiskt säkerställd får därför delad placering, till exempel "1–4". Se `METOD.md`, avsnitt 10.
- **Varför:** beslut av redaktionellt ansvarig. En läsare ska kunna se ordningen direkt, utan att osäkerheten döljs.
- **Vad som inte ändras:** inga mått, jämförelser, trösklar eller titlar. "Bäst i faktaprovet (läge A)" delas fortfarande bara ut enligt titelregeln. Placeringen bygger på samma förregistrerade jämförelser som allt annat.
- **I maskinläsbar form:** `presentation` i `method.yaml` (`sort_by: traffsakerhet`, `tie_rule: rank_range_supported`).
- **Faktisk kostnad:** tre nya mått som visas bredvid listpriset: tokens ut per svar (resonemangstokens inräknade), kostnad per 100 frågor och kostnad per rätt svar på huvudnivån, i kronor. De räknas av kod ur körningens loggade tokens och kostnad, med Riksbankens kurs från 2026-10-14 som står fast i `method.yaml` (`presentation.cost`). Skälet är att en modell med högre pris per token kan bli billigare om den använder färre tokens. Måtten påverkar inte rättning, jämförelser eller titlar.

---

## 2026-10-14, rättelser i efterhand i stället för förhandsvarsel

- **Vad:** vi skickar inget förhandsvarsel till leverantörerna före publicering. I stället gäller rättelser i efterhand: en rättelseadress (den dåvarande, på den tidigare domänen), en offentlig rättelselogg och rätt till genmäle. Ett svar från en leverantör publiceras i sin helhet. Alla leverantörer behandlas lika. Ett svar ger ingen rätt att påverka frågor, regler eller resultat. Se `METOD.md`, avsnitt 14.
- **Varför:** beslut av redaktionellt ansvarig. Ett varsel gav redan tidigare ingen rätt att påverka frågor, regler eller resultat. Det det gav leverantören var rätten att anmäla sakfel och få ett svar publicerat, och den rätten finns kvar, men efter publiceringen.
- **Vad detta betyder, uttryckt rakt:** en leverantör ser sina resultat först när de publiceras. Ett sakfel kan därför vara publicerat en tid innan det anmäls och rättas. Därför är rättelser öppna och daterade, med gammalt och nytt värde.
- **I maskinläsbar form:** `publication` i `method.yaml` (`notice_policy: corrections_after`). Grind G13 kontrollerar att varje sida med mätresultat eller leverantörsnamn visar rättelseadressen och länkar till rättelseloggen, och att rättelseloggen beskriver rätten till genmäle.
- **Övrigt:** gränsen på minst 15 länskluster (`min_county_clusters`) är bekräftad av metodägaren. Beskrivningen i avsnitt 3 av vilka modeller som körs via tredjepartsleverantör är omformulerad så att den inte påstår mer än som är belagt om en leverantörs villkor.

---

## 2026-10-14, efter pilot 2 (pilotdriven)

### Nivåroller: stopp för omdefiniering

- **Vad:** nivå 1 (kommun till län) och nivå 2 (tätort under 1 500 invånare till kommun) blev *beskrivande grundnivåer*. De redovisas med konfidensintervall, har inget tak- eller golvkrav och ingår inte i huvudtalet. Nivå 3 (småort till kommun) är *huvudnivån*. Tak (ingen modell över 90 %) och golv (bästa modellen minst 30 %) gäller bara den. Kravet "tak på nivå 2 och 3" ersattes av "tak och golv gäller huvudnivån".
- **Varför, i ordning:**
  1. Efter pilot 1 var den dåvarande nivå 2 (tätorter 1 500 till 10 000 invånare) för lätt: 7 av 8 modeller låg över 90 % och den högsta på 100 %. Nivå 3 var en blandning av små tätorter och småorter. Vi omdefinierade nivåerna till kommun till län (kontroll), tätort under 1 500 till kommun, och småort till kommun.
  2. Pilot 2 prövade de nya nivåerna med nya frågor. Nivå 1 var mättad (alla modeller 100 %). Nivå 2 (tätort under 1 500) klarade fortfarande inte taket: 5 av 8 modeller över 90 %, högsta 99,3 %. Att sänka folkmängdsgränsen hjälpte inte, eftersom även de minsta tätorterna var kända av de starkaste modellerna. Nivå 3 klarade både tak och golv (högsta modell cirka 77 %, lägsta cirka 32 %).
  3. Den tidigare regeln tillät en andra omdefiniering. Vi valde i stället att stoppa. Varje ytterligare justering mot dagens modeller formar provet efter dem. Därför blev grundnivåerna beskrivande, och mättnaden redovisas som ett fynd: modellerna kan tätorter men gissar oftare på småorter.
- **Vad detta betyder, uttryckt rakt:** ett krav som nivå 2 inte klarade ersattes av ett krav som gäller den nivå som klarade det. Beslutet fattades efter att vi sett utfallet, och nivåindelningen har därför formats med pilotmodellerna som referens. Det är en urvalseffekt som står i begränsningarna.
- **Regel framåt (`level_policy`):** högst en pilot och högst en omdefiniering per prov före låsning. Mäts ett prov ändå som mättat publiceras det som mättat, med dokumentation, och konstrueras aldrig vidare.

### Slutligt svar: markörraden

- **Vad:** alla modeller instrueras att avsluta med en sista rad på formen `Svar: <svar>`, och bara den raden rättas (`final_answer_rule: marker_line`). Saknas en giltig markörrad rättas hela svaret bokstavligt, och svaret räknas som formatbrott. Bokstavlig rättning av hela svaret är kvar som känslighetsanalys.
- **Varför:** i pilot 1 hade systemprompten ingen sådan instruktion, och i praktiken saknade 98 till 100 % av svaren en tydlig slutrad. Då beror utfallet på hur modellen råkar formulera sig, och rättningsregeln (bokstavlig text eller sista raden) kunde ändra vilka modeller som gynnades. Frågan "bokstavlig text eller sista raden" lämnades öppen efter pilot 1. Markörraden löste formatproblemet i pilot 2: 2 av 3 012 svar saknade giltig markörrad.
- **Förbehåll:** formuleringen av instruktionen är ett utkast och fastställs i låsfilen. Markörregeln är ett val som kan gynna modeller som följer formatinstruktioner. Formatbrott redovisas därför för alla modeller.
- **Rättelse:** ordet "pa" i systemprompten rättades till "på". Pilot 2 kördes med den gamla texten, och inga svar kördes om.

### Tokenbudget

- **Vad:** `max_tokens = 8000`, lika för alla modeller. Tar budgeten slut utan svar räknas svaret som tekniskt bortfall (Ogiltigt) och redovisas per modell. Budgeten ändrades inte efter pilot 2.
- **Varför samma budget:** en större budget för enskilda modeller gynnar modeller som använder många tokens, och gör jämförelsen svårare att försvara.
- **Vad piloten visade, uttryckt rakt:** hos en öppen resonemangsmodell tog budgeten slut i 6,3 % av svaren, mer än gränsen på 5 % per modell. Bortfallet föll på de svåraste frågorna (småorter och påhittade namn), alltså där det skevar mest. En andra modell hade 1,1 %, övriga 0. Samma budget är ett val, och effekten är att den modellens siffror på nivå 3 och påhittade namn är osäkrare än de övrigas. Det redovisas per modell och nivå.
- **Diagnostik utanför mätningen:** en engångskörning med 16 000 tokens för den modellen görs bara för att förstå bortfallet. Den ändrar inte budgeten eller några regler.

### Statistik: klustring på län

- **Vad:** standardfel och minsta påvisbara skillnad (MDE) klustras på län (högst 21 kluster, df = antal län − 1) i stället för på formuleringsmall och entitet. Mall och entitet finns kvar som sparade attribut. Kravet på minst två mallkluster byttes mot ett krav på minst 15 länskluster (eget val, bekräftat av metodägaren 2026-10-14, öppet för synpunkter). Kravet på minst 30 entiteter räknas nu som antal olika entiteter.
- **Varför:** i båda piloterna var korrelationen inom mall nära noll (−0,009 och 0,005 i pilot 2; −0,002 och −0,036 i pilot 1), medan fyra mallar gav df = 3 och därmed en MDE på 12 till 41 procentenheter som inte speglade något verkligt beroende. Entiteten är redan en fråga per nivå och kan inte skilja sig från den oklustrade enheten. Länet fångar däremot ett beroende som faktiskt finns (korrelation 0,20 på nivå 3 i pilot 2).
- **Två skydd, egna val:** variansen får aldrig vara lägre än den oklustrade variansen, och när alla utfall är 0 eller alla är 1 används Wilson-intervall.
- **Hur valet ska läsas:** regeln valdes efter att vi sett att mallklustring gav mycket breda intervall. Det kan misstänkas ha valts för att göra intervallen smalare. Skydden ovan och kravet på 15 län finns för att motverka det. Ingen körning har gjorts med den nya regeln före beslutet, och den är förregistrerad före låsning. Se `STATISTIK.md`.

### Påhittade namn

- **Korrekt beteende:** både "vet ej" och "finns inte" räknas som korrekt beteende (`andel_korrekt_beteende`). Prompten ändrades inte. I båda piloterna var kategorin "korrekt avvisande" tom, eftersom prompten ber modellerna skriva "VET EJ". Måttet som skiljer är därför andelen påhitt, och det är en egenskap hos prompten.
- **Pilotspärr:** alla påhittade namn behålls i huvudanalysen. Ett namn där minst två modeller påstår samma enda län markeras, och resultatet redovisas också utan de markerade namnen som förregistrerad känslighetsanalys. I pilot 2 gällde det 4 av 40 namn. De kan vara riktiga men dåligt dokumenterade ställen som existenskontrollen inte hittade, eller så gissar modellerna län ur namnets ändelse. Vi vet inte vilket.
- **Automatisk existenskontroll i stället för mänsklig:** den ursprungliga planen var att en människa granskar varje påhittat namn. Det ersattes av en automatisk kontroll som loggar källa, resultat och tidsstämpel per namn (beslut av metodägaren). Det är en försvagning, och den står bland begränsningarna.

---

## 2026-10-14, före pilot 2

### Åtkomstväg

- **Vad:** alla modeller körs via OpenRouter, låsta till förstapartsleverantören (`allow_fallbacks: false`, `data_collection: deny`). Faktisk leverantör och returnerat modell-id loggas per anrop, och en avvikelse stoppar publiceringen. Direktanrop mot leverantörernas API är inte ett krav. De är en valfri engångskontroll på offentliga frågor.
- **Varför:** ersatte ett tidigare krav på direktanrop. En nyckel och ett gränssnitt för alla modeller, och leverantörslåsningen fungerade i pilotens logg. Nackdelen är att en mellanhand kan översätta vissa parametrar, och att "leverantörens eget API" lättare går att försvara vid en tvist.
- **Följd:** öppna modeller från leverantörer på den förregistrerade försiktighetslistan (avsnitt 3 i `METOD.md`) körs via en namngiven tredjepartsleverantör, med leverantörskänslighet på minst två leverantörer. Listan är ett försiktighetsval och inget påstående om vad leverantörernas villkor säger.

### Rättarvalidering

- **Vad:** valideringen av rättaren ändrades från "två människor märker minst 200 svar" till: två oberoende rättare (A och B) rättar alla svar, en människa märker alla svar där A och B skiljer sig plus 30 slumpade ur enigheterna (högst 50 poster), och en blind jury av tre språkmodeller från icke-Anthropic-familjer bedömer samma poster.
- **Varför:** det fanns bara en människa att tillgå.
- **Följd:** kappa mellan människor kan inte beräknas. Gemensamma fel hos A och B fångas bara av de 30 enigt rättade posterna. Noll fel i 30 ger en övre 95-procentig gräns på cirka 9,5 %. Det redovisas som en avvikelse från den ursprungliga planen.

### Nivåer efter pilot 1 (pilotdriven)

Se "Nivåroller" ovan, punkt 1: nivåerna omdefinierades en första gång efter pilot 1 eftersom nivå 2 var för lätt.

---

## 2026-10, före första pilot

### Första utkastet

Metod 1.0 skrevs som en specifikation för delprov A: svensk faktakunskap och påhittade namn, med beslutstrappa, mått, titelregler, känslighetsanalyser, statistik, förregistrering och rättelseprotokoll. Den bygger på öppen metod och publicerade förebilder för den här typen av prov, bland annat SimpleQA (garderingsregeln), Artificial Analysis (öppna "svara VET EJ"-läget och kontroll av den offentliga delmängden) och Kalai m.fl. (läge A+).
