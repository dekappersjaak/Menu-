 (cd "$(git rev-parse --show-toplevel)" && git apply --3way <<'EOF' 
diff --git a/README.md b/README.md
new file mode 100644
index 0000000000000000000000000000000000000000..eb920e91059fd067f0576f48c8099b3279053457
--- /dev/null
+++ b/README.md
@@ -0,0 +1,85 @@
+# ZorgMenu+
+
+ZorgMenu+ is een slimme weekmenu- en boodschappenapp voor zorginstellingen. Het platform helpt teams om maaltijden, boodschappen en voorraadbeheer in één systeem te organiseren, rekening houdend met bewoners, diëten, allergieën en budgetten.
+
+## Doelstellingen
+- Tijdswinst en overzicht bij het plannen van maaltijden en het doen van boodschappen.
+- Minder verspilling door automatisch voorraadbeheer en bestellijsten.
+- Heldere taakverdeling binnen zorgteams (koken, boodschappen, mee-eten).
+
+## Kernfuncties
+- Weekmenu samenstellen per dag en maaltijd.
+- Automatisch boodschappenlijstje op basis van menu, aantal eters en aanwezige voorraad.
+- Voorraadbeheer met waarschuwingen voor bijna lege producten of naderen van THT-datum.
+- Duidelijke rolverdeling (wie kookt, wie doet boodschappen, wie eet mee).
+- Budgetcontrole en prijsinschatting per leverancier.
+- Exporteren of bestellen via CSV, PDF of koppeling met leveranciers.
+- Dashboard met overzicht van kook- en besteldagen.
+- Notificaties en herinneringen voor bestellingen of voorraadcontroles.
+
+## Belangrijke Datavelden
+| Domein | Datavelden |
+| --- | --- |
+| **Eters** | naam, dieet, allergieën, aanwezig/niet-aanwezig |
+| **Recepten** | titel, ingrediënten, porties, instructies |
+| **Ingrediënten** | naam, hoeveelheid, eenheid, houdbaarheid, barcode |
+| **Menu's** | dag, maaltijd, gekoppelde recepten |
+| **Boodschappenlijst** | item, hoeveelheid, leverancier, prijs, status |
+| **Voorraad** | product, hoeveelheid, locatie, THT-datum |
+| **Leveranciers** | naam, contact, type (AH, Jumbo, Sligro) |
+
+## Automatisch Voorraad- en Bestelproces
+1. **Datalaag**
+   - Productcatalogus per leverancier met vaste SKU-mapping.
+   - Voorraadregistratie via barcode-scan (ZXing) of OCR op etiket (Tesseract).
+   - Verbruik voorspellen op basis van geplande menu’s.
+2. **Logica**
+   - Replenishmentregel: \(\text{bestelhoeveelheid} = \max(0, \text{behoefte tot volgende levering} - (\text{voorraad} - \text{veiligheidsvoorraad}))\).
+   - Substitutie bij allergenen of uitverkoop.
+   - Budgetcontrole per week of afdeling.
+3. **Output**
+   - Complete bestellijst per leverancier met prijzen en leverdag.
+   - Kanalen: EDI-koppeling, CSV-export of e-mail.
+
+## Leveranciers-API's en Integratie
+- **Sligro** (aanbevolen)
+  - Ondersteunt EDI (ORDERS/EDIFACT) via partner of gateway.
+  - Retourinformatie via ORDRSP/INVOIC.
+  - Vereist klantnummer en contract.
+  - Productie-klaar en officieel ondersteund.
+- **Albert Heijn & Jumbo**
+  - Geen publieke bestel-API.
+  - Export via CSV- of PDF-upload in portaal.
+  - Ongeautoriseerde community-API’s zijn instabiel en niet geschikt voor productie.
+- **CSV-route (voor MVP)**
+  - Exporteer velden: `{item, sku, hoeveelheid, eenheid, leverancier, prijs, THT, status}`.
+  - Formaat: UTF-8, komma-gescheiden, één leverancier per bestand.
+
+## Stackvoorstel
+- **Frontend:** React of Flutter.
+- **Backend:** Supabase of Firebase + Cloud Functions.
+- **Database:** PostgreSQL met JSON-kolommen voor recepten en voorraad.
+- **Scanner/OCR:** ZXing, Tesseract.
+- **EDI-gateway (Sligro):** EdiFabric, Cleo of partnerkoppeling (Apicbase).
+
+## Externe API's voor Recepten en Producten
+| API | Gebruik |
+| --- | --- |
+| **Edamam** | Zoeken naar recepten met diëten, allergenen, voedingswaarden. Endpoint: `GET /api/recipes/v2?type=public&q={query}&health={label}&diet={label}` |
+| **Spoonacular** | Kostencalculatie per recept en ingrediënt. Endpoints: `/recipes/{id}/information`, `/recipes/priceBreakdownByID/{id}` |
+| **TheMealDB** | Gratis, eenvoudige JSON-API. Endpoints: `lookup.php?i={id}`, `search.php?s={name}` |
+| **Open Food Facts** | Product- en barcode-informatie. Endpoint: `/api/v2/product/{barcode}.json` |
+| **Tandoor Recipes (self-hosted)** | Eigen veilige receptenbank met API. |
+
+## Aanbevolen API-combinaties
+- Edamam voor diëten en allergieën.
+- Spoonacular voor prijsberekening.
+- Open Food Facts voor verpakte producten en barcodes.
+- Tandoor voor interne zorgrecepten.
+
+## Datastructuur-mapping
+- Koppel leveranciers-SKU's aan interne ingrediënten via een mappingtabel met prioriteit per leverancier.
+- Koppel menu's aan recepten en recepten aan ingrediënten met portie-grootte conversies.
+- Houd voorraadmutaties bij via barcode-scans en menuverbruik.
+- Bewaak budgetten per afdeling met weeklimieten en koppeling aan boodschappenlijsten.
+ 
EOF
)
