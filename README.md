# ZorgMenu+

ZorgMenu+ is een digitale weekmenu- en boodschappenassistent voor zorginstellingen. Het platform combineert maaltijdplanning,
voorraadbeheer en leveranciersintegraties zodat teams meer tijd overhouden voor bewonerszorg en tegelijk grip houden op kosten
en diëten.

## Samenvatting
- **Probleem:** versnipperde Excel-lijsten, losse boodschappenapps en ontbrekend voorraadoverzicht leiden tot verspilling en
  stress bij zorgteams.
- **Oplossing:** één applicatie waarin menus, bewonersprofielen, voorraden en leveranciersbestellingen samenkomen.
- **Resultaat:** minder verspilling, minder regeldruk en betere borging van diëten en allergieën.

## Doelstellingen
1. Binnen 5 minuten een volledig weekmenu plannen per afdeling of woongroep.
2. Maximaal 2% voedselverspilling door realtime voorraad- en houdbaarheidsbewaking.
3. 100% naleving van dieet- en allergieafspraken via automatische controles.
4. Integratie met minimaal één groothandel (Sligro) binnen het MVP.

## Belangrijkste gebruikersrollen
| Rol | Behoefte | Belangrijkste schermen |
| --- | --- | --- |
| **Kok / Huiskamerteam** | Snel menu samenstellen, weten wat er besteld of aanwezig is. | Weekplanner, boodschappenlijst, voorraad.
| **Planner / Diëtist** | Controleren op diëten en allergieën, budgetbewaking. | Bewonersprofielen, dieetwaarschuwingen, budgetdashboards.
| **Inkoper / Facilitair** | Bestellingen consolideren, leveranciers aansturen. | Leveranciersbeheer, bestelstatus, integraties.
| **Teamleider / Manager** | Inzicht in kosten, naleving en planning. | KPI-dashboard, auditlog, rapportages.

## Kernfunctionaliteiten
1. **Weekmenuplanner**
   - Drag-and-drop recepten, automatische portieherberekening.
   - Alternatieven bij allergieën of dieetrestricties.
2. **Slimme boodschappenlijsten**
   - Consolidatie per leverancier met realtime prijsindicatie.
   - Statussen (concept, besteld, geleverd) en taaktoewijzing.
3. **Voorraadbeheer**
   - Barcode- of QR-scans (ZXing), houdbaarheidstracking en minimumvoorraden.
   - Automatische voorraadmutaties op basis van geplande maaltijden.
4. **Bewoners- en dieetbeheer**
   - Profielen met allergieën, voorkeuren, textuur, energie-inname.
   - Waarschuwingen bij menuconflicten en vervangingssuggesties.
5. **Budget- & rapportagemodule**
   - Budgetten per week/afdeling, afwijkingen en prognoses.
   - Export naar PDF/CSV of BI-connector.

## Niet-functionele eisen
- **Beschikbaarheid:** 99,5% uptime tijdens werkdagen.
- **Beveiliging:** AVG-conform, SSO via zorgbrede IdP (SAML/OIDC), auditlog.
- **Performance:** pagina’s laden < 2s bij 200 gelijktijdige gebruikers.
- **Toegankelijkheid:** WCAG 2.1 AA, inclusief schermlezer-ondersteuning.

## Architectuuroverzicht
```
┌────────────┐     ┌────────────────┐      ┌─────────────────┐
│  Frontend  │◀───▶│  BFF / API     │◀────▶│  Services        │
│  (React)   │     │  (Node/Nest)   │      │  (Supabase/Postg)│
└────────────┘     └────────────────┘      └─────────────────┘
        ▲                     ▲                      ▲
        │                     │                      │
   Browsers             Integratiehub           Leveranciers &
   Tablets (PWA)        (Message queue)         Externe API's
```
- **Frontend:** React + TypeScript, PWA voor tablets in keukens.
- **BFF/API-laag:** Node.js (NestJS) of Supabase Edge Functions voor auth, autorisatie en bedrijfslogica.
- **Services:** PostgreSQL met row level security, storage voor receptfoto’s, queue (e.g. Supabase Realtime) voor notificaties.
- **Integraties:** Webhooks/EDI via integratiehub (Cleo, EdiFabric) en CSV/REST fallback.

## Gegevensmodel (hoog niveau)
| Entiteit | Kernvelden | Relaties |
| --- | --- | --- |
| **Bewoner** | naam, dieetlabels, allergieën, energiebehoefte | n:m met Menu via MenuDeelnemer |
| **Menu** | datum, maaltijdtype, status, afdeling | m:n met Recepten, koppeling naar boodschappenlijsten |
| **Recept** | titel, instructies, porties, voedingswaarden | m:n met Ingrediënten |
| **Ingrediënt** | naam, categorie, eenheid, houdbaarheid | n:1 met LeverancierProduct |
| **VoorraadItem** | locatie, hoeveelheid, THT, minimum | 1:n met Ingrediënt |
| **Boodschappenregel** | hoeveelheid, prijs, status | 1:n met Leverancier, 1:n met Menu |
| **Leverancier** | naam, levertijden, EDI-config | n:1 met LeverancierProduct |

## Integraties en API's
| Kanaal | Ondersteuning | Opmerkingen |
| --- | --- | --- |
| **Sligro EDI** | Productie | ORDERS/ORDRSP/INVOIC via gateway, authenticatie met klantnummer & certificaat. |
| **Albert Heijn / Jumbo portaal** | MVP | Upload CSV (UTF-8, komma-gescheiden) met kolommen `sku, omschrijving, qty, unit, prijs`. |
| **Open Food Facts** | MVP | Barcode lookup voor productdetails. Rate limiting respecteren. |
| **Edamam / Spoonacular** | Optioneel | Recept-inspiratie en kostencalculatie. API-sleutels in secrets manager. |

## Operationele processen
1. **Menuplanning** – Planner stelt weekmenu op, controles op dieetlabels draaien realtime.
2. **Bestelrun** – Dagelijks om 14:00 genereert systeem ordervoorstellen; inkoper bevestigt en verstuurt.
3. **Ontvangst & voorraad** – Levering wordt gescand, verschillen worden geboekt en triggeren notificaties.
4. **Audit & rapportage** – Wekelijks overzicht van kosten per afdeling, compliance op allergenen.

## Implementatieroadmap
1. **Fase 1 – MVP (8 weken)**
   - Basis weekplanner, bewonersprofielen, CSV-export voor bestellingen.
   - Voorraadmutaties handmatig, notificaties via e-mail.
2. **Fase 2 – Automatisering (6 weken)**
   - Barcode-scans, minimale voorraadniveaus, realtime budgetbewaking.
   - Integratie met Sligro EDI via partner.
3. **Fase 3 – Optimalisatie (doorlopend)**
   - Machine learning voor verbruiksvoorspelling.
   - Multilocation support, API voor externe BI-tools.

## Governance & compliance
- Datalocatie binnen EU, encryptie in rust (AES-256) en tijdens transport (TLS 1.2+).
- Role Based Access Control met least-privilege-rollen.
- Jaarlijkse penetratietest en DPIA, bewaartermijnen conform zorgprotocol.

## Begrippenlijst
| Term | Definitie |
| --- | --- |
| **THT** | Ten minste houdbaar tot, bewaartermijn voor voedselproducten. |
| **EDI** | Electronic Data Interchange voor gestructureerde bestelberichten. |
| **BFF** | Backend-for-frontend; API-laag afgestemd op front-end behoeften. |
| **PWA** | Progressive Web App voor gebruik op tablets en offline scenario’s. |

## Volgende stappen
- Uitwerken UI-wireframes (planner, boodschappenlijst, voorraad).
- Technische spike naar Supabase Edge Functions voor webhook-afhandeling.
- Gesprek met Sligro-partner voor certificering en testomgeving.
