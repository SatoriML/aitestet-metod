# Så lämnar du synpunkter / How to give feedback

## Svenska

Vi tar emot synpunkter på AI-testets metod **löpande**. Metod 1.0 är låst, så synpunkter ändrar inte version 1.0 eller dess resultat. De prövas inför nästa version (1.1) och förs in i `CHANGELOG.md`. Det enda undantaget är sakfel i facit eller kod, som rättas direkt enligt rättelseprotokollet i `METOD.md` (avsnitt 14).

**Alla synpunkter och svar loggas öppet.** Det gäller också de vi inte tar till oss. Vi anger för varje ärende om det lett till ändring, delvis ändring eller ingen ändring, med skäl.

### Så gör du

1. Öppna ett ärende (issue) i det offentliga arkivet: https://github.com/SatoriML/aitestet-metod.
2. Du kan också skriva till hej@aitestet.se, till exempel om du inte använder GitHub eller har hittat ett sakfel. Svaret och ärendet loggas på samma sätt (utan personuppgifter).
3. Rubrik: `[avsnitt N] kort beskrivning`, till exempel `[avsnitt 12] Golv vid oklustrad varians`.
4. Skriv:
   - **Avsnitt eller fil** synpunkten gäller (`METOD.md`, `method.yaml`, `STATISTIK.md`)
   - **Vad** du tycker är fel, oklart eller saknas
   - **Varför** (gärna med källa, till exempel en artikel, en formel eller ett exempel)
   - **Förslag** på hur det kan lösas, om du har ett
   - Om du vill **namnges** i ändringsloggen när synpunkten leder till ändring
5. Skriv **inga personuppgifter** i ärendet. Det är offentligt och loggas.

### Vilken sorts synpunkter vi söker

- Sakfel i regler, formler, källhänvisningar eller licenser
- Statistik: klustring på län, golv och Wilson-intervall, Holm, minsta påvisbara skillnad, kravet på 15 länskluster
- Rättvisa mellan modeller: markörraden `Svar: <svar>`, tokenbudgeten, garderingsregeln, hur "vet ej" och "finns inte" räknas
- Frågekällor: facit, tvåkällskravet, urval av nivåer, påhittade namn
- Rättarvalideringen och juryn
- Titelregler och formuleringar som kan vilseleda en läsare
- Avsändartext, intäktsregler och märkning
- Oklara eller motsägande formuleringar mellan `METOD.md`, `method.yaml` och `STATISTIK.md`

### Vad vi inte kan göra

- Vi kan inte dela privata frågor, facit eller råsvar på privata frågor före deras pensionering. De skyddas för att inte kunna tränas in. Du kan däremot kontrollera hashvärdena som publiceras före varje körning.
- Vi tar inte emot önskemål om rangordning eller placering för en viss modell. Leverantörer har en egen väg via rättelseadressen och rätten till genmäle (`METOD.md`, avsnitt 14). Vi skickar inget förhandsvarsel. Samma regler gäller för alla.
- Vi svarar inte på frågor som inte rör metoden.

### Vad som händer sedan

- Ärendet får ett svar i ärendet. Svaret anger status: *ändras i 1.1*, *ändras delvis*, *ändras inte* (med skäl), *utanför metoden* eller *sakfel, rättas nu*.
- Ändringar förs in i `CHANGELOG.md` med datum och motivering.
- Stängda ärenden raderas aldrig.

## English

We accept feedback on the AI-testet method **on a rolling basis**. Method 1.0 is locked, so feedback does not change version 1.0 or its results. It is considered for the next version (1.1) and recorded in `CHANGELOG.md`. The one exception is factual errors in answer keys or code, which are corrected immediately under the corrections protocol in `METOD.md` (section 14, in Swedish).

**All feedback and all responses are logged publicly**, including feedback we decline. For each issue we state whether it led to a change, a partial change, or no change, with reasons.

### How

1. Open an issue in the public repository: https://github.com/SatoriML/aitestet-metod.
2. You can also write to hej@aitestet.se, for example if you do not use GitHub or have found a factual error. The reply is logged in the same way (without personal data).
3. Title: `[section N] short description`.
4. Include: the section or file concerned, what you think is wrong, unclear or missing, why (with a source if possible), a suggested fix if you have one, and whether you want to be credited in the changelog.
5. Do **not** include personal data. Issues are public.

Issues may be written in Swedish or English.

### Useful kinds of feedback

Factual errors in rules, formulas, citations or licences; statistics (county clustering, variance floor, Wilson interval, Holm, minimum detectable effect, the 15-cluster requirement); fairness between models (the `Svar:` marker line, the token budget, the hedging rule, treatment of "don't know" versus "does not exist"); question sources and answer keys; grader validation and the jury; title rules and wording that could mislead; the sender statement and revenue rules; contradictions between the three method files.

### What we cannot do

We cannot share private questions, answer keys or raw answers to private questions before they are retired; they are withheld to prevent training on them. You can verify the hashes published before each run. We do not take requests for a particular ranking or placement of any model; vendors have a separate route through notice and right of reply, with the same rules for everyone.
