# Domoticz-KNMI-verdamping
# KNMI Verdamping Script voor Domoticz

## Beschrijving
Dit dzVents script haalt automatisch verdampingsgegevens op van het KNMI en slaat deze op in een virtuele sensor in Domoticz. Het script is ontworpen om dagelijks te draaien en de verdampingsgegevens van twee dagen geleden op te halen (omdat KNMI de gegevens met vertraging publiceert).

## Functionaliteiten
- Automatisch ophalen van verdampingsgegevens van KNMI
- Opslaan in een Domoticz Custom Sensor
- Voorkomt dubbele updates door bij te houden welke datum al is verwerkt
- Kan handmatig worden geactiveerd via een optionele schakelaar
- Uitgebreide logging voor probleemoplossing

## Vereisten
- Domoticz met dzVents ondersteuning
- Een Custom Sensor in Domoticz (standaard IDX 290)
- Optioneel: Een schakelaar genaamd "Verdamping Manual Update" voor handmatige updates

## Installatie
1. Ga naar Domoticz > Instellingen > Meer opties > Gebeurtenissen
2. Klik op "+" om een nieuw script toe te voegen
3. Geef het script een naam (bijv. "KNMI_Verdamping")
4. Selecteer "dzVents" als scripttype
5. Kopieer en plak de scriptcode
6. Klik op "Opslaan"

## Configuratie
De volgende instellingen kunnen worden aangepast in het script:

```lua
-- Configuratie
local sensorIdx = 290                  -- IDX van de verdampingssensor
local knmiStation = 377                -- KNMI station (377 = Ell)
local daysToFetch = 2                  -- Aantal dagen geleden om op te halen
```

### KNMI Stations
Enkele veelgebruikte KNMI stations:
- 260: De Bilt
- 377: Ell
- 240: Schiphol
- 370: Eindhoven
- 280: Eelde

Een volledige lijst is beschikbaar op de KNMI website.

## Gebruik
Het script draait automatisch elke dag om 8:00 uur. Als je een "Verdamping Manual Update" schakelaar hebt aangemaakt, kun je deze activeren om het script handmatig te laten draaien.

## Probleemoplossing
- Controleer de Domoticz logs voor berichten met de marker "KNMI Verdamping"
- Als er geen gegevens worden opgehaald, controleer of het KNMI station correct is ingesteld
- Als het script meerdere keren per dag draait maar geen updates uitvoert, is dit normaal gedrag - het script controleert of de datum al is verwerkt

## Persistente Data
Het script gebruikt persistente data om bij te houden:
- `lastUpdateDate`: De laatst verwerkte datum (in formaat YYYYMMDD)
- `currentDate`: De datum waarvoor momenteel gegevens worden opgehaald

Om de persistente data te resetten:
1. Ga naar Domoticz > Instellingen > Meer opties > Gebeurtenissen
2. Zoek het script in de lijst
3. Klik op "Reset" naast het script

## Aanpassingen
- Om de uitvoeringstijd te wijzigen, pas de timer instelling aan in de `on` sectie
- Voor meer of minder debug informatie, pas het logging niveau aan
- Om gegevens van een andere dag op te halen, pas de `daysToFetch` variabele aan
