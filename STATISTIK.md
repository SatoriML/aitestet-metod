# Statistik i Metod 1.0

Koden finns i `stats.py` och `metrics.py`. Allt är deterministiskt (ingen slump, ingen bootstrap).

## Enhet och skattare
- Enheten är **frågan**. Upprepade körningar (`repeat`) medelvärdesbildas inom fråga och modell: tre körningar av samma
  fråga är inte tre frågor. Fråga *i* får värdet y_i = andel av dess körningar med utfallet (0/1 vid en körning).
- Andelsmått (träffsäkerhet, felandel, andel avstår …) är medelvärdet ȳ. Index och felavdragspoäng är 100·medelvärdet
  av (rätt − p·fel). Precision bland besvarade och AA-kvoten är kvoter Σa/Σb och linjäriseras (u_i = (a_i − R·b_i)/Σb).
- Nollnämnare ger `value = null` med skäl i `undefined`, aldrig 0.

## Standardfel: klustring på län (beslut 2026-10-14)
Klusternyckel = **länet frågan gäller** (`metrics.county_key`): det angivna länet i frågan (nivå 2 och 3), facit-länet (nivå 1) eller,
för påhittade namn som inte har något län, namnet självt (varje påhittat namn är ett eget kluster, alltså ingen klustring där).
Det ger 17–21 kluster per nivå i pilot 2 (nivå 1: 17, nivå 2: 20, nivå 3: 21, påhittade: 40). Med s_i = (y_i − ȳ)/n:

    V = G/(G−1) · Σ_g (Σ_{i∈g} s_i)²,        df = G − 1

(enkelvägs klusterrobust varians med småprovsfaktor G/(G−1)). **Mall** (formuleringsvariant) och **entitet** är fortsatt lagrade
attribut på varje fråga och redovisas (`clusters_template`, `entities`), men de klustrar inte. Skäl: i båda piloterna var
ρ för mall ≈ 0 (−0,009 och 0,005 i pilot 2; −0,002 och −0,036 i pilot 1), medan fyra mallar gav df = 3 och därmed en MDE på 12–41
procentenheter som inte speglade något verkligt beroende. Entiteten är redan en fråga per nivå, så den kan inte skilja sig från
den oklustrade enheten; länet fångar däremot det beroende som faktiskt finns (ρ för län 0,20 på nivå 3 i pilot 2). Regeln är
förregistrerad före låsning, och ingen körning har gjorts med den innan beslutet.

Två tillägg är **egna val** och ska granskas:
1. **Golv vid oklustrad varians.** V = max(V_kluster, V_iid). Klustring ska aldrig ge falsk precision med få kluster.
2. **Wilson vid gränsen.** När alla y_i är 0 (eller alla 1) är klustervariansen odefinierad (0). Då används Wilson-intervallet på
   n frågor (`ci_method = "wilson_boundary"`). Det ignorerar klustring och kan vara för smalt om ρ > 0; det är känt och redovisas i `ci_method`.

Om antalet län är under 2 faller skattningen tillbaka på oklustrad varians (`ci_method = "iid_wald"`). `ci_method` är annars `cluster_floor_iid`.

## Parvisa jämförelser (G9)
- Differensen d_i = y_a,i − y_b,i på samma frågor; samma länsklustrade formel på d. p-värde ur t(df), tvåsidigt.
  Om SE = 0: p = 1 vid diff 0, annars 0 (degenererat, ingen variation). df = antal län − 1 (20 på nivå 3 i pilot 2).
- **Familj** = alla parvisa jämförelser inom läge × nivå × mått. m = k(k−1)/2 oordnade par (8 modeller: 28).
  **Holm** (step-down, monoton) över hela familjen. Varje par skrivs i båda ordningarna (a,b) och (b,a) med
  negerad diff så att texter kan slå upp i valfri riktning; `family_size` är m.
- **Exakt McNemar** (`p_mcnemar`) är en oklustrad kontroll och visas bara när alla utfall är binära.
- **MDE** (minsta påvisbara skillnad): (t_{1−α/(2m)} + t_{power})·SE, med α/m = Holms första steg och effekt 80 %.
  Skattas ur parets egen SE, alltså ur data. Med få frågor eller få län blir MDE stor; då skriver G9 "för få frågor för att skilja åt".
- **Ekvivalens** (TOST): `equivalent_within_margin` = 90 %-intervallet för diff ligger inom ±min_effect.
  "Ingen skillnad" får aldrig skrivas utifrån icke-signifikans (METOD.md avsnitt 12).
- **supported** kräver alla fem: Holm-justerat p < 0,05, |diff| ≥ 5 procentenheter, minst 30 entiteter (oberoende frågor),
  minst 15 länskluster (`min_county_clusters`, eget val 2026-10-14, ersätter kravet 2 mallkluster; gränsen bekräftad av metodägaren 2026-10-14), och
  `sensitivity_ok`. Annars `not_supported`, eller `insufficient_n` när klusterkraven inte nås.

## Känslighetsanalyser (förregistrerade, `sensitivity_ok`)
En jämförelse håller bara om **samma riktning och oklustrat 95 %-intervall utan 0** gäller i varje variant:
- felavdrag p ∈ {0,5, 1, 2}: poäng 100·(rätt − p·fel); riktningen jämförs med huvudmåttet (för felandel omvänd);
- garderingsregel AA (gardering med kandidat räknas som ej försökt) i stället för SimpleQA;
- utan frågor som G8 flaggat (alla flaggor, oavsett beslut);
- för påhittade namn dessutom **utan pilotspärrade påhittade namn** (se nedan).

## Påhittade namn: korrekt beteende och pilotspärr (beslut 2026-10-14)
- **Korrekt beteende** = "VET EJ"/avstår **och** "finns inte" (korrekt avvisande). Båda räknas som rätt beteende (`andel_korrekt_beteende`
  = (korrekt avvisande + avstår) / påhittade); fel beteende är **påhitt** (ett län namnges) och ogiltigt (tekniskt bortfall eller oklassat).
  Prompten ändras inte. Huvudmåttet för jämförelser är oförändrat `andel_pahitt`.
- **Alla påhittade namn behålls** i huvudanalysen och räknas som påhitt när ett län namnges, även de som minst två modeller svarade
  samma län på i den andra utvecklingskörningen (fyra namn; de kan vara riktiga ställen som existenskontrollen inte hittade).
- **Känslighetsanalys "utan pilotspärrade påhittade namn"** (förregistrerad): ett namn är spärrat om minst två olika modeller påstår samma
  enda län om det (`metrics.fake_pilot_blocked_ids`, deterministisk ur rådata). `andel_pahitt_utan_sparrade` och
  `andel_korrekt_beteende_utan_sparrade` redovisas bredvid huvudmåtten i results.json (nivå F), och en G9-variant (`excl_pilot_blocked_fakes`)
  ingår i `sensitivity_ok` för jämförelser av påhitt.

## Körlägen, upprepningar och urvalet till A2 (beslut 2026-10-16; `method.yaml: modes`)
- **Läge A** körs tre gånger per fråga, **läge B** en gång, och **A2** (omformulering) en gång på ett förregistrerat urval om cirka 20 % av frågorna.
  Läge A+ finns inte. Enheten är fortfarande frågan: de tre körningarna i A medelvärdesbildas till y_i ∈ {0, 1/3, 2/3, 1}, och antalet frågor n,
  antalet län G och därmed df = G − 1 ändras inte.
- **Vad tre körningar ger och inte ger.** Variansen för ȳ har en del mellan frågor och en del inom frågor (svar som varierar mellan körningar).
  Tre körningar minskar bara den andra delen, med en faktor tre. Den första delen, och beroendet inom län, ändras inte. Standardfel och MDE blir
  därför mindre, men inte tre gånger mindre, och de kan aldrig bli mindre än vad G ≤ 21 kluster tillåter. Det är skälet till att en körning
  till i B och A2 inte behövs.
- **Urvalet till A2** är en deterministisk funktion av fråge-id och ett salt: en fråga ingår om sha256(salt + ":" + fråge-id), de första 15
  hextecknen som heltal delat med 16^15, är mindre än 0,20 (`method.config.in_mode_sample`). Det gäller riktiga och påhittade frågor lika, och
  beror aldrig på svar eller modell. Andelen är cirka 20 %, inte exakt, och urvalets storlek redovisas.
- **Statistik i A2.** Samma formler som i övriga lägen, räknade på urvalet. Kraven på minst 30 entiteter och minst 15 länskluster gäller även
  där, och en jämförelse som inte når dem är "otillräckligt underlag". Stabiliteten mellan A och A2 är en parad differens på samma frågor
  (urvalet) med samma länsklustrade formel. Titlar och placering bygger bara på läge A, och A2 används inte för egna rangordningar.

## Begränsningar
- Rättarens utfall är en deterministisk funktion av texten; mätfelet i rättaren ingår inte i intervallen (G5, G7 b).
- Piloten har en körning per fråga: intervallen där säger mer om metoden än om modellerna.
- Med klustring på län är antalet kluster högst 21 (Sveriges län), oavsett hur många frågor som ställs. MDE kan därför aldrig bli
  mindre än vad 21 kluster tillåter; fler frågor per län hjälper bara om ρ för län är liten.

## Nivåer och krav (beslut 2026-10-14; `method.yaml: levels, level_policy`)
- Nivå 1 (kommun → län) och nivå 2 (tätort < 1 500 → kommun) är **beskrivande grundnivåer** (`role: descriptive`): redovisas med
  konfidensintervall, har inget tak- eller golvkrav och ingår inte i huvudtalet. Nivå 3 (småort → kommun) är **huvudnivån**
  (`role: headline`); tak (ingen modell över 90 %) och golv (bästa modellen minst 30 %) gäller den. `alla` i results/comparisons = huvudnivån.
- Kravet "tak på nivå 2–3" ersätts av "tak/golv gäller huvudnivån".
- **Regel för kommande prov:** en pilot och högst en omdefiniering per prov före låsning. Mättas ett prov ändå publiceras det som mättat,
  med dokumentation; det konstrueras aldrig vidare.


## Delar utan län: språkunderlaget (beslut 2026-10-20; `method.yaml: subtests.sprakprovet.clustering`)
Allt ovan gäller delen **Sverige**. Språkunderlaget har inga län, så länsklustringen kan inte användas. Regeln nedan är förregistrerad
före huvudkörningen. Formlerna (enkelvägs klusterrobust varians med G/(G−1), golv vid oklustrad varians, Wilson vid gränsen, t med
df = G − 1, Holm, MDE, TOST) är desamma; bara klusternyckeln och klusterkravet skiljer sig.

- **Kandidatnyckel = regeltyp** (`rule_type`, 17 förregistrerade typer, till exempel preteritum av starka verb, dubbelteckning,
  sammansättning med förkortning, de/dem). Frågor av samma regeltyp prövar samma regel, så de kan hänga ihop på samma sätt som
  småorter i samma län: en modell som inte kan regeln missar flera av dem.
- **Beslutsregel:** klustra på regeltyp om ρ för regeltyp skattad i **piloten** är större än 0 (ICC(1), ANOVA-skattning, värdet per fråga
  = andelen modeller som svarade rätt i läge A, grupper = regeltyp). Annars är **frågan enheten** (oklustrad varians, df = n − 1).
  Golvet vid oklustrad varians gäller i båda fallen, så klustring kan bara göra intervallen bredare, aldrig smalare.
- **Beslutet fattas en gång**, på pilotens svar, och skrivs in i `method.yaml` (`clustering.decision` och `clustering.pilot_rho`) före
  huvudkörningen. Det ändras inte efter att huvudkörningens svar finns.
- **Varför inte alltid klustra:** finns inget mätbart beroende inom regeltyp kostar klustringen frihetsgrader (17 typer ger df = 16 i
  stället för n − 1) utan att skydda mot något. **Varför inte bara frågan:** mallklustringen i delen Sverige visade att ett grovt kluster
  (fyra mallar) ger orimligt stora intervall, men länet visade att ett verkligt beroende finns när frågor delar något; regeltypen är
  Språkunderlagets motsvarighet till länet, med ungefär lika många kluster (17 mot 21).
- **Krav för "stödd" jämförelse i språkunderlaget:** samma fem villkor som i avsnittet om parvisa jämförelser, men "minst 15 länskluster"
  ersätts av **minst 15 regeltypskluster** när regeltypen klustrar (`min_clusters`), och kravet på minst 30 entiteter gäller som minst
  30 frågor (`min_questions`). När frågan är enheten är varje fråga ett kluster och kravet på 15 kluster uppfylls av kravet på 30 frågor.
- **Skattningen av ρ är osäker.** Med 150 frågor och 8–9 frågor per regeltyp har ρ ett brett intervall. Regeln använder punktskattningen
  och väljer klustring vid varje positivt värde, vilket är det försiktiga valet.
- **Känslighetsanalyser:** felavdrag p ∈ {0,5, 1, 2}, bokstavlig rättning av hela svarstexten och "utan frågor som granskningen
  flaggat" gäller som i delen Sverige. Garderingsregeln AA ingår inte i version 1.0 för språkunderlaget: rättaren har bara huvudregeln
  (simpleqa, "jag tror att det är B" rättas på B), och det redovisas som en begränsning. Språkunderlaget har inga Delvis-utfall och inga
  påhittade frågor; "vet ej" är Avstår.

### Språkunderlagets nivå K, korrekturläsning (beslut 2026-10-21; beskrivande)
Varje stycke kommer från en egen författning (högst ett stycke per SFS-nummer), så styckena är oberoende av konstruktion. **Stycket är
enheten**: oklustrad varians, df = n − 1, golvet och Wilson vid gränsen som ovan. Klusterregeln för regeltyp gäller grundnivån G. Ett
stycke kan innehålla flera feltyper, så regeltypen kan inte vara klusternyckel på nivå K. Kravet för en stödd jämförelse är minst 30
stycken. Huvudmåttet är träffsäkerhet (exakt rätt mängd rättelser); andel delvis rätt, andel falsklarm på stycken utan fel och andel
funna fel redovisas bredvid med samma formler (andelar per stycke; falsklarm på stycken utan fel räknas bara på de felfria styckena).

## Delen Detaljer (beslut 2026-10-21; `method.yaml: lagprovet`; under utvecklingen kallad Lagprovet)
Inga län. Högst tre stycken per känd lag och ett per övrig författning; **stycket är enheten** (oklustrad varians, df = n − 1, golv och
Wilson som ovan). Kända lagar ("kand") och den långa svansen ("svans") redovisas var för sig med samma formler. Huvudmåttet är träffsäkerhet
för innehållet; andel delvis, falsklarm på oförändrade stycken och upptäckt av ändringar (med och utan språkfel i samma stycke) redovisas
bredvid. I den kombinerade varianten är språkets träffsäkerhet och falsklarm beskrivande mått. Kravet för en stödd jämförelse är minst 30
stycken. Att stycken ur samma kända lag kan hänga ihop ingår inte i intervallen och redovisas som en begränsning; skattas ρ för lag > 0 i
huvudkörningen redovisas en känslighetsanalys med klustring på författning.

**Huvudkörningen i Detaljer** (beslut 2026-10-21). Högst åtta stycken per känd lag; därför redovisas en känslighetsanalys med klustring på
författning bredvid huvudmåtten. **Upprepningsurvalet** (40 av 200, förregistrerat) körs en gång till; stabilitet = andelen stycken där de
två körningarna ger samma utfall för innehållet, per modell, med Wilson-intervall. Huvudtalet bygger bara på den första körningen. **Nyligen
ändrade** (50 stycken) redovisas med samma formler men separat och beskrivande; där redovisas också andelen svar som anger den tidigare
lydelsen som rätt (`previous_pair`). Det finns inget sammanvägt totalresultat över delarna förrän en viktningsregel är förregistrerad.
