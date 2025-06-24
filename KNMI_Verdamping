--[[ 
    KNMI Verdamping Script
    Haalt verdampingsgegevens op van KNMI voor de afgelopen dagen
    en slaat deze op in een virtuele sensor met IDX 290.
--]]

local webResponse = "knmiVerdampingData"

return {
    on = {
        timer = { 
            'at 08:00',
            'every 5 minutes' -- Voor testen, verwijder in productie
        },
        httpResponses = { webResponse },
        devices = { 'Verdamping Manual Update' }, -- Optioneel: maak een schakelaar om handmatig te updaten
    },

    logging = {
        level = domoticz.LOG_INFO,
        marker = 'KNMI Verdamping',
    },
    
    data = {
        lastUpdateDate = { initial = '' },
        currentDate = { initial = '' } -- Nieuwe variabele om de huidige datum bij te houden
    },
       
    execute = function(dz, item)
        -- Configuratie
        local sensorIdx = 290                  -- IDX van de verdampingssensor
        local knmiStation = 377                -- KNMI station (377 = Ell)
        local daysToFetch = 2                  -- Aantal dagen geleden om op te halen
        
        -- Functie om datum van N dagen geleden te krijgen in YYYYMMDD formaat
        local function getDateDaysAgo(daysAgo)
            local today = os.time()
            local targetDate = today - (daysAgo * 24 * 60 * 60)
            return os.date("%Y%m%d", targetDate)
        end
        
        -- Functie om KNMI data op te halen
        local function getKNMIData()
            -- Datum van X dagen geleden
            local date = getDateDaysAgo(daysToFetch)
            
            -- Controleer of we deze datum al verwerkt hebben
            if date == dz.data.lastUpdateDate then
                dz.log('Al verwerkt voor datum ' .. date .. ', overslaan', dz.LOG_INFO)
                return
            end
            
            -- Sla de huidige datum op in persistente data
            dz.data.currentDate = date
            
            -- KNMI API URL voor verdampingsgegevens (EV24 = 24-uurs verdamping)
            local url = "https://www.daggegevens.knmi.nl/klimatologie/daggegevens?stns=" .. 
                        knmiStation .. "&vars=EV24&start=" .. date .. "&end=" .. date
            
            dz.log('KNMI data ophalen voor datum ' .. date .. ': ' .. url, dz.LOG_INFO)
            
            -- HTTP verzoek maken zonder context (gebruiken we persistente data in plaats van)
            dz.openURL({
                url = url,
                method = 'GET',
                callback = webResponse
            })
        end
        
        -- Functie om KNMI CSV data te verwerken
        local function processKNMIData(response)
            local evaporation = nil
            
            -- Response in regels splitsen
            for line in string.gmatch(response, "[^\r\n]+") do
                -- Commentaarregels overslaan
                if not string.match(line, "^#") then
                    -- Controleren of deze regel station data bevat
                    if string.find(line, tostring(knmiStation)) then
                        dz.log('Data regel gevonden: ' .. line, dz.LOG_INFO)
                        
                        -- Splitsen op komma's
                        local parts = {}
                        for part in string.gmatch(line, "[^,]+") do
                            table.insert(parts, part)
                        end
                        
                        -- Verdampingswaarde uit derde deel halen (EV24)
                        if #parts >= 3 then
                            local valueStr = string.match(parts[3], "%s*(%d+)%s*")
                            if valueStr then
                                local numValue = tonumber(valueStr)
                                if numValue then
                                    -- KNMI geeft verdamping in 0.1 mm, dus delen door 10
                                    evaporation = numValue / 10
                                    dz.log('Verdampingswaarde: ' .. evaporation .. ' mm', dz.LOG_INFO)
                                    break
                                end
                            end
                        end
                    end
                end
            end
            
            return evaporation
        end
        
        -- Hoofdlogica
        if item.isTimer or (item.isDevice and item.name == 'Verdamping Manual Update' and item.active) then
            dz.log('KNMI verdamping update starten voor apparaat IDX ' .. sensorIdx, dz.LOG_INFO)
            getKNMIData()
            
        elseif item.isHTTPResponse then
            if item.ok then
                -- Gebruik de datum uit persistente data in plaats van context
                local date = dz.data.currentDate
                
                dz.log('KNMI response ontvangen voor datum ' .. date, dz.LOG_INFO)
                
                -- KNMI response verwerken
                local evaporation = processKNMIData(item.data)
                
                -- Apparaat bijwerken als we een geldige verdampingswaarde hebben
                if evaporation and evaporation >= 0 and evaporation < 100 then
                    dz.log('Apparaat IDX ' .. sensorIdx .. ' bijwerken met verdampingswaarde: ' .. evaporation .. ' mm', dz.LOG_INFO)
                    
                    -- Sensor bijwerken
                    dz.devices(sensorIdx).updateCustomSensor(evaporation)
                    
                    -- Datum opslaan die we net verwerkt hebben
                    dz.data.lastUpdateDate = date
                    dz.log('Laatste update datum nu ingesteld op ' .. date, dz.LOG_INFO)
                else
                    dz.log('Geen geldige verdampingswaarde gevonden of waarde buiten bereik', dz.LOG_ERROR)
                end
            else
                dz.log('HTTP verzoek naar KNMI mislukt: ' .. (item.statusText or "Onbekende fout"), dz.LOG_ERROR)
            end
        end
    end
}
