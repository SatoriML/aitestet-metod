# Kontrollsummor för de publicerade metodfilerna

Filerna `method.yaml` och `STATISTIK.md` i den här katalogen är **exakta kopior** av filerna med samma namn i metodens källkatalog vid tidpunkten för paketet (2026-10-21). De är inte omskrivna eller förkortade. Kontrollsummorna nedan är SHA-256 och är desamma som de som står i låsfilen `method.lock` under nycklarna `method/method.yaml` och `method/STATISTIK.md`.

| Fil | SHA-256 |
|---|---|
| `method.yaml` | `dfbef1302782414902e90bbec464bc40e941d5279294d07a9692a7fde95e43a6` |
| `STATISTIK.md` | `10c49b913f797f6d623493c198f95893c956d3c3e043392882a7b50e36b3d582` |

Det finns två olika hashvärden för låsningen, och de ska inte förväxlas:

1. **Låsfilens hash** är SHA-256 över själva filen `method.lock`. Det är den som står i `METOD.md` och på sajten. Den publiceras före huvudkörningen: 20049643e10befd569ef56548986b25790f6f54b4847d32cda7412752024ba8f (2026-10-21 16:00 UTC (18.00 svensk tid)). Den kan kontrolleras med `shasum -a 256 method.lock` i arkivets rot.
2. **`method_hash`** är ett fält inuti låsfilen: en samlad hash över alla låsta metodfiler, inklusive kod som inte publiceras i det här paketet. Den står i varje körnings `manifest.json` och i kontrollrapporterna, och körningen stoppas om den inte stämmer med låsfilen. Värdet för den här versionen: `f194755a4781136c4626d0529e14aa12ae5c328e611da80557c5483f0bc296e4`

**Kontrollera själv** (i den här katalogen):

```
shasum -a 256 method.yaml STATISTIK.md
```

Summorna ska vara identiska med tabellen och med den publicerade låsfilen. Om de skiljer sig har filerna ändrats efter låsningen. Ändringar gäller bara om de står i `CHANGELOG.md` med datum och ny metodversion.

**Obs, innan publicering:** tabellen ovan gäller de filer som fanns när paketet sammanställdes om 2026-10-21 (vid låsningen 2026-10-21). Skrivs låsfilen om (till exempel när `status` i `method.yaml` ändras vid låsning) ska filerna kopieras om och tabellen uppdateras så att den stämmer med den publicerade låsfilen. Den här anmärkningen tas bort då.
