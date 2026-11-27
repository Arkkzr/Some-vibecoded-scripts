loadstring(game:HttpGet("https://scripts.wabisabi.mom/wabi-sabi-ui-lib.lua"))()
local Library = WabiSabi

local Window = Library:CreateWindow({
    Title = "Pony",
    SubTitle = "",
    Size = Vector2.new(400, 650),
    Resize = true,
    Theme = "Dark",
    Translucent = false,
    MinimizeKey = "End",
})

local Visuals = Window:AddTab({ Title = "Visuals", Icon = "eye" })
local Raid = Window:AddTab({ Title = "Raid", Icon = "bomb" })
local Settings = Window:AddTab({ Title = "Settings", Icon = "cog" })

local settings = {
    enabled = false,
    maxDistance = 1000,
    showMilitaryCrate = false,
    showLoot = false,
    showResources = false,
    showSleepers = false,
    showSleepingBags = false,
    showAnimals = false,
    showSoldiers = false,
    showMonsters = false,
    showChestSmall = false,
    showChestLarge = false,
    showChestPrivate = false,
    showChestStrong = false,
    showProtectionSigns = false,
    showRaidLogger = false,
    showTripBomb = false,
    showDoors = false,
    showDoorHP = true,
    doorMaxDistance = 500,
    updateDelay = 3,
    resourceFilter = {},
    animalFilter = {},
}


local ANIMAL_COLORS = {
    ["Pig"] = Color3.fromRGB(255, 182, 193),
    ["Wolf"] = Color3.fromRGB(200, 200, 200),
    ["Bear"] = Color3.fromRGB(139, 69, 19),
    ["Alligator"] = Color3.fromRGB(0, 150, 0),
    ["Chicken"] = Color3.fromRGB(255, 200, 100),
}

local MONSTER_COLORS = {
    ["Zombie"] = Color3.fromRGB(144, 238, 144),
    ["Ghost"] = Color3.fromRGB(200, 200, 200),
}

local RESOURCE_COLORS = {
    ["StoneOre"] = Color3.fromRGB(200, 200, 200),
    ["IronOre"] = Color3.fromRGB(200, 150, 50),
    ["SulfurOre"] = Color3.fromRGB(255, 255, 0),
    ["Barrel"] = Color3.fromRGB(255, 140, 0),
    ["Grass"] = Color3.fromRGB(34, 139, 34),
}

local USEFUL_ITEMS = {
    ["ChestStrong"] = Color3.fromRGB(255, 0, 0),
    ["ChestTrade"] = Color3.fromRGB(255, 0, 0),
    ["ChestLarge"] = Color3.fromRGB(255, 165, 0),
    ["Medkit"] = Color3.fromRGB(255, 165, 0),
    ["RepairHammer"] = Color3.fromRGB(255, 165, 0),
    ["SmgAmmo"] = Color3.fromRGB(255, 165, 0),
    ["MilitaryPants"] = Color3.fromRGB(255, 165, 0),
    ["WaterBottle"] = Color3.fromRGB(0, 150, 255),
    ["IronAxe"] = Color3.fromRGB(255, 0, 0),
    ["Steak"] = Color3.fromRGB(200, 100, 50),
    ["IronPickAxe"] = Color3.fromRGB(255, 0, 0),
    ["Sulfur"] = Color3.fromRGB(255, 165, 0),
    ["Ectoplasm"] = Color3.fromRGB(0, 200, 255),
    ["CrateKey"] = Color3.fromRGB(255, 0, 0),
    ["PistolAmmo"] = Color3.fromRGB(255, 165, 0),
    ["MetalDoor"] = Color3.fromRGB(255, 0, 0),
    ["PajamaShirt"] = Color3.fromRGB(255, 0, 0),
    ["PajamaPants"] = Color3.fromRGB(255, 0, 0),
    ["KevlarShirt"] = Color3.fromRGB(255, 0, 0),
    ["Revolver"] = Color3.fromRGB(255, 0, 0),
    ["Shotgun"] = Color3.fromRGB(255, 0, 0),
    ["Dynamite"] = Color3.fromRGB(255, 0, 0),
    ["ShotgunAmmo"] = Color3.fromRGB(255, 165, 0),
    ["MilitaryShirt"] = Color3.fromRGB(255, 165, 0),
}

local CHEST_COLORS = {
    ["ChestSmall"] = Color3.fromRGB(255, 215, 0),
    ["ChestLarge"] = Color3.fromRGB(255, 200, 0),
    ["ChestPrivate"] = Color3.fromRGB(255, 180, 0),
    ["ChestStrong"] = Color3.fromRGB(255, 160, 0),
}

local DOOR_COLORS = {
    ["Door"] = Color3.fromRGB(255, 200, 100),
    ["MetalDoor"] = Color3.fromRGB(200, 200, 255),
}

local IGNORED_ITEMS = {
    ["Rock"] = true,
    ["Torch"] = true,
}

local espData = {}
local targets = {}
local dirty = true
local lastScan = 0
local lastWarn = 0
local player = game.Players.LocalPlayer

local LABEL_Y_OFFSET = -12
local SOLDIER_Y_OFFSET = -12
local frameCounter = 0
local doorCache = {}
local lastDoorScan = 0

local function isPart(inst)
    if not inst then
        return false
    end
    local ok, pos = pcall(function()
        return inst.Position
    end)
    return ok and typeof(pos) == "Vector3"
end

local function childrenOf(inst)
    local ok, kids = pcall(function()
        return inst:GetChildren()
    end)
    if ok and kids then
        return kids
    end
    return {}
end

local function findFirst(parent, name)
    if not parent then
        return nil
    end
    local ok, child = pcall(function()
        return parent:FindFirstChild(name)
    end)
    if ok then
        return child
    end
    return nil
end

local function nameOf(inst)
    local ok, name = pcall(function()
        return inst.Name
    end)
    if ok then
        return name
    end
    return nil
end

local function partFrom(model, preferredName)
    if not model then
        return nil
    end
    if preferredName then
        local preferred = findFirst(model, preferredName)
        if isPart(preferred) then
            return preferred
        end
    end
    local okPrimary, primary = pcall(function()
        return model.PrimaryPart
    end)
    if okPrimary and isPart(primary) then
        return primary
    end
    local kids = childrenOf(model)
    for i = 1, #kids do
        if isPart(kids[i]) then
            return kids[i]
        end
    end
    return nil
end

local function addTarget(part, text, color, dynamic, maxDist, yOffset)
    if not part then
        return
    end
    if #targets >= 500 and dynamic ~= true then
        return
    end
    local entry = {
        part = part,
        text = text,
        color = color,
        colorKey = color.R .. "," .. color.G .. "," .. color.B,
        dynamic = dynamic == true,
        maxDist = maxDist,
        yOffset = yOffset or LABEL_Y_OFFSET,
    }
    if not entry.dynamic then
        local ok, pos = pcall(function()
            return part.Position
        end)
        entry.pos = ok and pos or nil
    end
    targets[#targets + 1] = entry
end

local function getAnimalType(model)
    local scriptNames = { "WolfAI", "PigAI", "ChickenAI", "BearAI", "AlligatorAI" }
    for i = 1, #scriptNames do
        if findFirst(model, scriptNames[i]) then
            return (string.gsub(scriptNames[i], "AI", ""))
        end
    end
    return "Animal"
end

local function getItemColor(itemName)
    return USEFUL_ITEMS[itemName] or Color3.fromRGB(255, 255, 255)
end

local function getResourceColor(resourceName)
    for key, color in pairs(RESOURCE_COLORS) do
        if string.find(resourceName, key) then
            return color
        end
    end
    return Color3.fromRGB(255, 255, 255)
end

local function matchesFilter(name, filters)
    if #filters == 0 then
        return true
    end
    for i = 1, #filters do
        if string.find(name, filters[i]) then
            return true
        end
    end
    return false
end

local function rebuildTargets()
    targets = {}
    if not settings.enabled then
        return
    end

    local ws = game.Workspace

    if settings.showMilitaryCrate then
        local folder = findFirst(ws, "Soldiers")
        if folder then
            local kids = childrenOf(folder)
            for i = 1, #kids do
                local child = kids[i]
                if nameOf(child) == "MilitaryCrate" and isPart(child) then
                    addTarget(child, "Military Crate", Color3.fromRGB(85, 107, 47), false)
                end
            end
        end
    end

    if settings.showLoot then
        local folder = findFirst(ws, "Loot")
        if folder then
            local kids = childrenOf(folder)
            for i = 1, #kids do
                local item = kids[i]
                local itemName = nameOf(item)
                if itemName and not IGNORED_ITEMS[itemName] then
                    local part = partFrom(item, "Handle")
                    if part then
                        addTarget(part, itemName, getItemColor(itemName), false)
                    end
                end
            end
        end
    end

    if settings.showResources then
        local folder = findFirst(ws, "Suroviny")
        if folder then
            local kids = childrenOf(folder)
            for i = 1, #kids do
                local obj = kids[i]
                if isPart(obj) then
                    local objName = nameOf(obj)
                    if objName and matchesFilter(objName, settings.resourceFilter) then
                        addTarget(obj, objName, getResourceColor(objName), false)
                    end
                end
            end
        end
    end

    if settings.showSleepers then
        local folder = findFirst(ws, "Sleepers")
        if folder then
            local kids = childrenOf(folder)
            for i = 1, #kids do
                local part = partFrom(kids[i], "HumanoidRootPart")
                if part then
                    addTarget(part, "Sleeper", Color3.fromRGB(245, 222, 179), true)
                end
            end
        end
    end

    if settings.showSleepingBags then
        local folder = findFirst(ws, "SleepingBags")
        if folder then
            local kids = childrenOf(folder)
            for i = 1, #kids do
                local part = partFrom(kids[i], nil)
                if part then
                    addTarget(part, "Sleeping Bag", Color3.fromRGB(0, 255, 0), false)
                end
            end
        end
    end

    if settings.showAnimals then
        local folder = findFirst(ws, "Animals")
        if folder then
            local kids = childrenOf(folder)
            for i = 1, #kids do
                local animal = kids[i]
                local animalType = getAnimalType(animal)
                if matchesFilter(animalType, settings.animalFilter) then
                    local part = partFrom(animal, "AnimalMesh")
                    if part then
                        addTarget(part, animalType, ANIMAL_COLORS[animalType] or Color3.fromRGB(255, 200, 100), true)
                    end
                end
            end
        end
    end

    if settings.showSoldiers then
        local root = findFirst(ws, "Soldiers")
        local folder = root and findFirst(root, "Soldiers")
        if not folder then
            folder = root
        end
        if folder then
            local kids = childrenOf(folder)
            for i = 1, #kids do
                local part = partFrom(kids[i], "HumanoidRootPart")
                if part then
                    addTarget(part, "Soldier", Color3.fromRGB(255, 0, 0), true, nil, SOLDIER_Y_OFFSET)
                end
            end
        end
    end

    if settings.showMonsters then
        local folder = findFirst(ws, "Monsters")
        if folder then
            local kids = childrenOf(folder)
            for i = 1, #kids do
                local monster = kids[i]
                local monsterName = nameOf(monster)
                local part = partFrom(monster, "HumanoidRootPart")
                if part and monsterName then
                    addTarget(part, monsterName, MONSTER_COLORS[monsterName] or Color3.fromRGB(255, 255, 255), true)
                end
            end
        end
    end

    if settings.showProtectionSigns then
        local folder = findFirst(ws, "ProtectingSigns")
        if folder then
            local kids = childrenOf(folder)
            for i = 1, #kids do
                local sign = kids[i]
                if isPart(sign) then
                    local names = {}
                    local gui = findFirst(sign, "SurfaceGui")
                    if gui then
                        for n = 1, 4 do
                            local label = findFirst(gui, "jmeno" .. n)
                            if label then
                                local okText, text = pcall(function()
                                    return label.Text
                                end)
                                if okText and text and text ~= "" and not string.find(text, "Failed") and not string.find(text, "failed") and tonumber(text) == nil then
                                    names[#names + 1] = text
                                end
                            end
                        end
                    end
                    if #names > 0 then
                        addTarget(sign, table.concat(names, "\n"), Color3.fromRGB(255, 50, 50), false)
                    end
                end
            end
        end
    end

    if settings.showRaidLogger then
        local folder = findFirst(ws, "Items")
        if folder then
            local kids = childrenOf(folder)
            for i = 1, #kids do
                local child = kids[i]
                local childName = nameOf(child)
                if childName and string.find(childName, "RaidLogger") then
                    local part = partFrom(child, "Main1")
                    if part then
                        addTarget(part, "Raid Logger", Color3.fromRGB(220, 240, 255), false)
                    end
                end
            end
        end
    end

    if settings.showTripBomb then
        local folder = findFirst(ws, "Items")
        local tripBomb = folder and findFirst(folder, "TripBomb")
        if tripBomb then
            local part = nil
            if isPart(tripBomb) then
                part = tripBomb
            else
                part = partFrom(tripBomb, nil)
            end
            if part then
                addTarget(part, "Trip Bomb", Color3.fromRGB(200, 0, 0), false)
            end
        end
    end

    if settings.showDoors then
        if tick() - lastDoorScan > 10 or #doorCache == 0 then
            doorCache = {}
            local buildings = findFirst(ws, "Buildings")
            if buildings then
                local bKids = childrenOf(buildings)
                for i = 1, #bKids do
                    if #doorCache >= 150 then
                        break
                    end
                    local cKids = childrenOf(bKids[i])
                    for j = 1, #cKids do
                        if #doorCache >= 150 then
                            break
                        end
                        local child = cKids[j]
                        local childName = nameOf(child)
                        if childName == "Doorway" or childName == "StoneDoorway" then
                            local pKids = childrenOf(child)
                            for k = 1, #pKids do
                                if #doorCache >= 150 then
                                    break
                                end
                                local part = pKids[k]
                                if isPart(part) then
                                    local doorName = nameOf(part)
                                    if doorName == "Door" or doorName == "MetalDoor" then
                                        doorCache[#doorCache + 1] = { part = part, name = doorName }
                                    end
                                end
                            end
                        end
                    end
                end
            end
            lastDoorScan = tick()
        end

        for i = 1, #doorCache do
            local door = doorCache[i]
            local hp = 0
            local hpValue = findFirst(door.part, "HP")
            if hpValue then
                local okHp, value = pcall(function()
                    return hpValue.Value
                end)
                if okHp and typeof(value) == "number" then
                    hp = value
                end
            end
            local text = door.name
            if settings.showDoorHP and hp > 0 then
                text = door.name .. "\nHP: " .. hp
            end
            addTarget(door.part, text, DOOR_COLORS[door.name] or Color3.fromRGB(255, 255, 255), false, settings.doorMaxDistance)
        end
    end

    if settings.showChestSmall or settings.showChestLarge or settings.showChestPrivate or settings.showChestStrong then
        local items = findFirst(ws, "Items")
        if items then
            local kids = childrenOf(items)
            for i = 1, #kids do
                local obj = kids[i]
                local objName = nameOf(obj)
                if objName and CHEST_COLORS[objName] then
                    local wanted = (settings.showChestSmall and objName == "ChestSmall")
                        or (settings.showChestLarge and objName == "ChestLarge")
                        or (settings.showChestPrivate and objName == "ChestPrivate")
                        or (settings.showChestStrong and objName == "ChestStrong")
                    if wanted then
                        local part = partFrom(obj, "ChestBody")
                        if part then
                            addTarget(part, objName, CHEST_COLORS[objName], false)
                        end
                    end
                end
            end
        end
    end
end

local function getOrCreateLabel(index)
    local data = espData[index]
    if not data then
        local label = Drawing.new("Text")
        label.Outline = true
        label.Font = Drawing.Fonts.SystemBold
        label.Size = 11
        label.Center = true
        label.Visible = false
        data = {
            label = label,
            text = nil,
            colorKey = nil,
            x = nil,
            y = nil,
            visible = false,
        }
        espData[index] = data
    end
    return data
end

local function hideAll()
    for i = 1, #espData do
        local data = espData[i]
        if data.visible then
            data.label.Visible = false
            data.visible = false
        end
    end
end

local function getPlayerPosition()
    if not player then
        return nil
    end
    local okChar, char = pcall(function()
        return player.Character
    end)
    if not okChar or not char then
        return nil
    end
    local hrp = findFirst(char, "HumanoidRootPart")
    if not hrp then
        return nil
    end
    local okPos, pos = pcall(function()
        return hrp.Position
    end)
    if okPos then
        return pos
    end
    return nil
end

local function renderESP()
    frameCounter = frameCounter + 1
    local used = 0
    local playerPos = getPlayerPosition()

    for i = 1, #targets do
        local target = targets[i]
        local pos = target.pos
        if target.dynamic or not pos then
            local okPos, livePos = pcall(function()
                return target.part.Position
            end)
            if okPos then
                pos = livePos
            elseif target.dynamic then
                pos = nil
            end
        end

        if pos then
            local limit = target.maxDist or settings.maxDistance
            local show = true
            if playerPos then
                show = (playerPos - pos).Magnitude <= limit
            end

            if show then
                local screenPos, onScreen = WorldToScreen(pos)
                if onScreen then
                    used = used + 1
                    local data = getOrCreateLabel(used)

                    if data.text ~= target.text then
                        data.label.Text = target.text
                        data.text = target.text
                    end

                    local colorKey = target.colorKey
                    if data.colorKey ~= colorKey then
                        data.label.Color = target.color
                        data.colorKey = colorKey
                    end

                    local sx = screenPos.X
                    local sy = screenPos.Y + (target.yOffset or LABEL_Y_OFFSET)
                    local canWrite = target.dynamic or (i % 2) == (frameCounter % 2)
                    if canWrite and (data.x ~= sx or data.y ~= sy) then
                        data.label.Position = Vector2.new(sx, sy)
                        data.x = sx
                        data.y = sy
                    end

                    if not data.visible then
                        data.label.Visible = true
                        data.visible = true
                    end
                end
            end
        end
    end

    for i = used + 1, #espData do
        local data = espData[i]
        if data.visible then
            data.label.Visible = false
            data.visible = false
        end
    end
end

local function clearESP()
    hideAll()
end

local function updateESP()
    dirty = true
end

local RunService = game:GetService("RunService")

RunService.RenderStepped:Connect(function()
    if Library.Unloaded then
        return
    end
    local ok, err = pcall(function()
        local now = tick()
        local delay = settings.updateDelay
        if delay < 0.25 then
            delay = 0.25
        end

        if dirty or now - lastScan >= delay then
            dirty = false
            lastScan = now
            rebuildTargets()
        end

        if settings.enabled then
            renderESP()
        else
            hideAll()
        end
    end)

    if not ok and tick() - lastWarn > 1 then
        lastWarn = tick()
        warn("ESP: " .. tostring(err))
    end
end)

Visuals:AddToggle({
    Id = "Toggle",
    Title = "Master Enable",
    Default = false,
    Callback = function(value)
        settings.enabled = value
        if not value then clearESP() else updateESP() end
    end,
})

local FarmSection = Visuals:AddSection("Farm")
FarmSection:AddToggle({
    Id = "MilitaryCrate",
    Title = "Military Crate",
    Default = false,
    Callback = function(value)
        settings.showMilitaryCrate = value
        updateESP()
    end,
})

FarmSection:AddToggle({
    Id = "Loot",
    Title = "Loot",
    Default = false,
    Callback = function(value)
        settings.showLoot = value
        updateESP()
    end,
})

FarmSection:AddToggle({
    Id = "Resources",
    Title = "Resources",
    Default = false,
    Callback = function(value)
        settings.showResources = value
        updateESP()
    end,
})

FarmSection:AddDropdown({
    Id = "ResourceFilter",
    Title = "Resource Types",
    Values = {"StoneOre", "IronOre", "SulfurOre", "Barrel", "Grass"},
    Multi = true,
    Default = {},
    Callback = function(value)
        local filters = {}
        for option, enabled in pairs(value) do
            if enabled then
                table.insert(filters, option)
            end
        end
        settings.resourceFilter = filters
        updateESP()
    end,
})

local MobsSection = Visuals:AddSection("Mobs")
MobsSection:AddToggle({
    Id = "Soldiers",
    Title = "Soldiers",
    Default = false,
    Callback = function(value)
        settings.showSoldiers = value
        updateESP()
    end,
})

MobsSection:AddToggle({
    Id = "Animals",
    Title = "Animals",
    Default = false,
    Callback = function(value)
        settings.showAnimals = value
        updateESP()
    end,
})

MobsSection:AddDropdown({
    Id = "AnimalFilter",
    Title = "Animal Types",
    Values = {"Pig", "Wolf", "Bear", "Alligator", "Chicken"},
    Multi = true,
    Default = {},
    Callback = function(value)
        local filters = {}
        for option, enabled in pairs(value) do
            if enabled then
                table.insert(filters, option)
            end
        end
        settings.animalFilter = filters
        updateESP()
    end,
})

MobsSection:AddToggle({
    Id = "Monsters",
    Title = "Monsters",
    Default = false,
    Callback = function(value)
        settings.showMonsters = value
        updateESP()
    end,
})

local ChestsSection = Raid:AddSection("Chests")
ChestsSection:AddToggle({
    Id = "ChestSmall",
    Title = "Chest Small",
    Default = false,
    Callback = function(value)
        settings.showChestSmall = value
        clearESP()
        updateESP()
    end,
})

ChestsSection:AddToggle({
    Id = "ChestLarge",
    Title = "Chest Large",
    Default = false,
    Callback = function(value)
        settings.showChestLarge = value
        clearESP()
        updateESP()
    end,
})

ChestsSection:AddToggle({
    Id = "ChestPrivate",
    Title = "Chest Private",
    Default = false,
    Callback = function(value)
        settings.showChestPrivate = value
        clearESP()
        updateESP()
    end,
})

ChestsSection:AddToggle({
    Id = "ChestStrong",
    Title = "Chest Strong",
    Default = false,
    Callback = function(value)
        settings.showChestStrong = value
        clearESP()
        updateESP()
    end,
})

local BaseSection = Raid:AddSection("Base")
BaseSection:AddToggle({
    Id = "Doors",
    Title = "Doors (Laggy)",
    Default = false,
    Callback = function(value)
        settings.showDoors = value
        updateESP()
    end,
})

BaseSection:AddToggle({
    Id = "DoorHP",
    Title = "Show Door HP",
    Default = true,
    Callback = function(value)
        settings.showDoorHP = value
        updateESP()
    end,
})

BaseSection:AddSlider({
    Id = "DoorDistance",
    Title = "Door Max Distance",
    Min = 100,
    Max = 3000,
    Default = 1500,
    Callback = function(value)
        settings.doorMaxDistance = value
    end,
})

BaseSection:AddToggle({
    Id = "ProtectionSigns",
    Title = "Protection Signs",
    Default = false,
    Callback = function(value)
        settings.showProtectionSigns = value
        updateESP()
    end,
})

BaseSection:AddToggle({
    Id = "RaidLogger",
    Title = "Raid Logger",
    Default = false,
    Callback = function(value)
        settings.showRaidLogger = value
        updateESP()
    end,
})

BaseSection:AddToggle({
    Id = "TripBomb",
    Title = "Trip Bomb",
    Default = false,
    Callback = function(value)
        settings.showTripBomb = value
        updateESP()
    end,
})

local PlayersSection = Raid:AddSection("Players")
PlayersSection:AddToggle({
    Id = "Sleepers",
    Title = "Sleepers",
    Default = false,
    Callback = function(value)
        settings.showSleepers = value
        updateESP()
    end,
})

PlayersSection:AddToggle({
    Id = "SleepingBags",
    Title = "Sleeping Bags",
    Default = false,
    Callback = function(value)
        settings.showSleepingBags = value
        updateESP()
    end,
})

Settings:AddSlider({
    Id = "Distance",
    Title = "Max Distance",
    Min = 100,
    Max = 3000,
    Default = 1500,
    Callback = function(value)
        settings.maxDistance = value
    end,
})

Settings:AddSlider({
    Id = "UpdateDelay",
    Title = "Update Delay (seconds)",
    Min = 0,
    Max = 5,
    Default = 3,
    Callback = function(value)
        settings.updateDelay = value
    end,
})

Library:Notify({
    Title = "Loaded",
    Content = "Hello user!",
    Duration = 4,
})

task.wait(999999)
clearESP()
