-- ========================================
-- SERVICES CACHE
-- ========================================
local Services = setmetatable({}, {
    __index = function(self, serviceName)
        local service = game:GetService(serviceName)
        rawset(self, serviceName, service)
        return service
    end
})

local HttpService = Services.HttpService
local Players = Services.Players
local CollectionService = Services.CollectionService
local ReplicatedStorage = Services.ReplicatedStorage
local TweenService = Services.TweenService
local RunService = Services.RunService
local VirtualInputManager = Services.VirtualInputManager

local player = Services.Players.LocalPlayer

-- ========================================
-- SETTINGS SYSTEM
-- ========================================
local FolderName = "Zyn Hub | Farm"
local FileName = "Settings.json"
local FullPath = FolderName .. "/" .. FileName

if makefolder and not isfolder(FolderName) then 
    makefolder(FolderName) 
end

_G.SaveData = _G.SaveData or {}

function SaveSettings()
    if not writefile then return false end
    local success = pcall(function()
        writefile(FullPath, HttpService:JSONEncode(_G.SaveData))
    end)
    return success
end

function LoadSettings()
    if not (isfile and isfile(FullPath)) then return false end
    local success, result = pcall(function()
        return HttpService:JSONDecode(readfile(FullPath))
    end)
    if success and result then 
        _G.SaveData = result
        return true
    end
    return false
end

function GetSetting(name, default)
    return _G.SaveData[name] ~= nil and _G.SaveData[name] or default
end

LoadSettings()

-- ========================================
-- AUTO KEN
-- ========================================
local commE = ReplicatedStorage:WaitForChild("Remotes"):WaitForChild("CommE")

_G.AutoKen = _G.AutoKen ~= false

local function HasKen()
    local char = player.Character
    return char and CollectionService:HasTag(char, "Ken")
end

task.spawn(function()
    while _G.AutoKen do
        task.wait(0.2)
        if not HasKen() then
            pcall(function()
                commE:FireServer("Ken", true)
            end)
        end
    end
end)

-- ========================================
-- AUTO TEAM
-- ========================================
local desiredTeam = "Marines"
if not player.Team or player.Team.Name ~= desiredTeam then
    pcall(function()
        ReplicatedStorage.Remotes.CommF_:InvokeServer("SetTeam", desiredTeam)
    end)
end

-- ========================================
-- LIGHTING
-- ========================================
local Lighting = Services.Lighting
Lighting.Ambient = Color3.new(0.695, 0.695, 0.695)
Lighting.ColorShift_Bottom = Color3.new(0.695, 0.695, 0.695)
Lighting.ColorShift_Top = Color3.new(0.695, 0.695, 0.695)
Lighting.Brightness = 2
Lighting.FogEnd = 1e10

-- ========================================
-- GLOBAL VARIABLES
-- ========================================
_G = _G or {}
_B = false
PosMon = nil

_G.BringRange = _G.BringRange or 235
_G.MaxBringMobs = _G.MaxBringMobs or 3
_G.MobHeight = _G.MobHeight or 20
_G.SelectWeapon = _G.SelectWeapon or nil
_G.FruitSkills = _G.FruitSkills or { Z = false, X = false, C = false, V = false, F = false }

-- ========================================
-- WORLD DETECTION
-- ========================================
local placeId = game.PlaceId
local World1, World2, World3

if placeId == 2753915549 or placeId == 85211729168715 then
    World1 = true
elseif placeId == 4442272183 or placeId == 79091703265657 then
    World2 = true
elseif placeId == 7449423635 or placeId == 100117331123089 then
    World3 = true
else
    player:Kick("❌ Blox Fruits - World not recognized")
    return
end

Sea = World1 or World2 or World3

-- ========================================
-- WORLD DATA
-- ========================================
local Boss, MaterialList

if World1 then
    Boss = { "The Gorilla King", "Bobby", "The Saw", "Yeti", "Mob Leader", "Vice Admiral", "Saber Expert", "Warden", "Chief Warden", "Swan", "Magma Admiral", "Fishman Lord", "Wysper", "Thunder God", "Cyborg", "Ice Admiral", "Greybeard" }
    MaterialList = { "Leather + Scrap Metal", "Angel Wings", "Magma Ore", "Fish Tail" }
elseif World2 then
    Boss = { "Diamond", "Jeremy", "Fajita", "Don Swan", "Smoke Admiral", "Awakened Ice Admiral", "Tide Keeper", "Darkbeard", "Cursed Captain", "Order" }
    MaterialList = { "Leather + Scrap Metal", "Radioactive Material", "Ectoplasm", "Mystic Droplet", "Magma Ore", "Vampire Fang" }
elseif World3 then
    Boss = { "Stone", "Hydra Leader", "Kilo Admiral", "Captain Elephant", "Beautiful Pirate", "Cake Queen", "Longma", "Soul Reaper" }
    MaterialList = { "Scrap Metal", "Demonic Wisp", "Conjured Cocoa", "Dragon Scale", "Gunpowder", "Fish Tail", "Mini Tusk" }
end

-- ========================================
-- WAIT FOR CHARACTER (MELHORADO)
-- ========================================
local function WaitForCharacter()
    local char = player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then
        char = player.CharacterAdded:Wait()
        char:WaitForChild("HumanoidRootPart")
    end
    return char
end

local character = WaitForCharacter()
local Root = character:WaitForChild("HumanoidRootPart")

-- Atualiza Root quando o personagem respawnar
player.CharacterAdded:Connect(function(char)
    character = char
    Root = char:WaitForChild("HumanoidRootPart")
    plr = player
end)

ply = Players
plr = player
replicated = ReplicatedStorage
TeleportService = Services.TeleportService
TW = TweenService
Enemies = workspace:FindFirstChild("Enemies")
vim1 = VirtualInputManager
RunSer = RunService
Stats = Services.Stats

-- ========================================
-- REMOTE HELPER FUNCTIONS
-- ========================================
Marines = function()
    replicated.Remotes.CommF_:InvokeServer("SetTeam", "Marines")
end

Pirates = function()
    replicated.Remotes.CommF_:InvokeServer("SetTeam", "Pirates")
end

EquipWeapon = function(toolName)
    if not toolName then return end
    local backpack = player:FindFirstChild("Backpack")
    local char = player.Character
    if not backpack or not char then return end
    
    local tool = backpack:FindFirstChild(toolName)
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if tool and humanoid then
        humanoid:EquipTool(tool)
    end
end

weaponSc = function(tooltipName)
    local backpack = player:FindFirstChild("Backpack")
    if not backpack then return end
    
    for _, tool in pairs(backpack:GetChildren()) do
        if tool:IsA("Tool") and tool.ToolTip == tooltipName then
            EquipWeapon(tool.Name)
            return
        end
    end
end

-- ========================================
-- TELEPORT FUNCTION (CORRIGIDA)
-- ========================================
function _tp(cf)
    local char = player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    
    char.HumanoidRootPart.CFrame = cf
    char.HumanoidRootPart.Velocity = Vector3.new(0, 0, 0)
    char.HumanoidRootPart.RotVelocity = Vector3.new(0, 0, 0)
end

function notween(cf)
    local char = player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    
    char.HumanoidRootPart.CFrame = cf
    char.HumanoidRootPart.Velocity = Vector3.new(0, 0, 0)
    char.HumanoidRootPart.RotVelocity = Vector3.new(0, 0, 0)
end

-- ========================================
-- MOB FUNCTIONS
-- ========================================
local MobHelper = {}
MobHelper.__index = MobHelper

function MobHelper.Alive(mob)
    if not mob then return false end
    local hum = mob:FindFirstChildOfClass("Humanoid")
    return hum and hum.Health > 0
end

function MobHelper.Dist(mob, distance)
    if not mob or not Root then return false end
    local hrp = mob:FindFirstChild("HumanoidRootPart")
    if not hrp then return false end
    return (Root.Position - hrp.Position).Magnitude <= distance
end

-- ========================================
-- FRUIT SKILLS
-- ========================================
function UseFruitSkills()
    weaponSc("Blox Fruit")
    
    local skills = _G.FruitSkills
    if not skills then return end
    
    if skills.Z then Useskills("Blox Fruit", "Z") end
    if skills.X then Useskills("Blox Fruit", "X") end
    if skills.C then Useskills("Blox Fruit", "C") end
    if skills.V then Useskills("Blox Fruit", "V") end
    if skills.F then
        vim1:SendKeyEvent(true, "F", false, game)
        vim1:SendKeyEvent(false, "F", false, game)
    end
end

function Useskills(tooltipName, key)
    local char = player.Character
    if not char then return end
    
    local tool = char:FindFirstChildOfClass("Tool")
    if not tool or tool.ToolTip ~= tooltipName then
        weaponSc(tooltipName)
        tool = char:FindFirstChildOfClass("Tool")
    end
    
    if tool then
        vim1:SendKeyEvent(true, key, false, game)
        task.wait(0.05)
        vim1:SendKeyEvent(false, key, false, game)
    end
end

-- ========================================
-- FARM CHECK
-- ========================================
local farmFlags = {
    "FarmPriorityElf", "FarmMastery_S", "Level", "AutoFarm_Bone", "AutoFarm_Cake",
    "FarmMastery_Dev", "FarmMastery_G", "AutoMaterial", "AutoTyrant", "SailBoat_Hydra",
    "WardenBoss", "AutoFactory", "HighestMirage", "HCM", "PGB", "Leviathan1", "UPGDrago",
    "Complete_Trials", "TpDrago_Prehis", "BuyDrago", "AutoFireFlowers", "DT_Uzoth",
    "AutoBerry", "Prehis_Find", "Prehis_Skills", "Prehis_DB", "Prehis_DE", "FarmBlazeEM",
    "Dojoo", "CollectPresent", "AutoLawKak", "TpLab", "AutoPhoenixF", "AutoFarmChest",
    "AutoHytHallow", "LongsWord", "BlackSpikey", "AutoHolyTorch", "TrainDrago", "AutoSaber",
    "CitizenQuest", "AutoEctoplasm", "KeysRen", "Auto_Rainbow_Haki", "obsFarm", "AutoBigmom",
    "Doughv2", "AuraBoss", "Raiding", "Auto_Cavender", "TpPly", "Bartilo_Quest", "AutoZou",
    "CraftVM", "FrozenTP", "TPDoor", "AcientOne", "AutoFarmNear", "AutoRaidCastle",
    "DarkBladev3", "AutoFarmRaid", "Auto_Cake_Prince", "Addealer", "TPNpc", "TwinHook",
    "FindMirage", "FarmChestM", "Shark", "TerrorShark", "Piranha", "MobCrew", "SeaBeast1",
    "FishBoat", "Auto", "AutoPoleV2", "Auto_SuperHuman", "AutoDeathStep",
    "Auto_SharkMan_Karate", "Auto_Electric_Claw", "AutoDragonTalon", "Auto_Def_DarkCoat",
    "Auto_God_Human", "Auto_Tushita", "AutoMatSoul", "AutoKenVTWO", "AutoSerpentBow",
    "AutoFMon", "Auto_Soul_Guitar", "TPGEAR", "AutoSaw", "AutoTridentW2", "Auto_StartRaid",
    "AutoEvoRace", "AutoGetQuestBounty", "MarinesCoat", "TravelDres", "Defeating", "DummyMan",
    "Auto_Yama", "Auto_SwanGG", "SwanCoat", "AutoEcBoss", "Auto_Mink", "Auto_Human",
    "Auto_Skypiea", "Auto_Fish", "CDK_TS", "CDK_YM", "CDK", "AutoFarmGodChalice",
    "AutoFistDarkness", "AutoMiror", "Teleport", "AutoKilo", "AutoGetUsoap", "Praying",
    "TryLucky", "AutoColShad", "AutoUnHaki", "Auto_DonAcces", "AutoRipIngay", "DragoV3",
    "DragoV1", "SailBoats", "NextIs", "FarmGodChalice", "IceBossRen", "senth", "senth2",
    "Lvthan", "beasthunter", "DangerLV", "Relic123", "tweenKitsune", "Collect_Ember",
    "AutofindKitIs", "snaguine", "TwFruits", "tweenKitShrine", "Tp_LgS", "Tp_MasterA",
    "tweenShrine"
}

local function FarmAtivo()
    -- Prioridade absoluta
    if _G.FarmPriorityElf or _G.FarmElfLevelCustom then return true end
    if _G.FarmMastery_S then return true end
    
    -- Outros farms
    if not _G.StartFarm then return false end
    
    for _, flag in ipairs(farmFlags) do
        if _G[flag] or (getgenv and getgenv()[flag]) then
            return true
        end
    end
    
    return false
end

-- ========================================
-- IGNORE MOBS
-- ========================================
local function IsRaidMob(mob)
    if not mob then return true end
    
    local name = mob.Name:lower()
    if name:find("raid") or name:find("microchip") or name:find("island") then
        return true
    end
    
    if mob:GetAttribute("IsRaid") or mob:GetAttribute("RaidMob") or mob:GetAttribute("IsBoss") then
        return true
    end
    
    local hum = mob:FindFirstChildOfClass("Humanoid")
    if hum and hum.WalkSpeed == 0 then
        return true
    end
    
    if mob.Parent and tostring(mob.Parent):lower():find("_worldorigin") then
        return true
    end
    
    return false
end

-- ========================================
-- BRING ENEMY (CORRIGIDO)
-- ========================================
BringEnemy = function()
    if not FarmAtivo() or not _B then return end
    
    local char = player.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    pcall(function()
        sethiddenproperty(player, "SimulationRadius", math.huge)
    end)
    
    if not PosMon then return end
    
    local targetCFrame = CFrame.new(PosMon + Vector3.new(0, 30, 0))
    _tp(targetCFrame)
end

-- ========================================
-- KILL FUNCTIONS (CORRIGIDAS)
-- ========================================
G = {}
G.__index = G

G.Alive = function(mob)
    if not mob then return false end
    local hum = mob:FindFirstChildOfClass("Humanoid")
    return hum and hum.Health > 0
end

G.Kill = function(mob, distance)
    if not (mob and distance) then return end
    
    local hrp = mob:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    if not mob:GetAttribute("Locked") then
        mob:SetAttribute("Locked", hrp.CFrame)
    end
    
    PosMon = mob:GetAttribute("Locked").Position
    _B = true
    BringEnemy()
    
    EquipWeapon(_G.SelectWeapon)
    
    local tool = player.Character:FindFirstChildOfClass("Tool")
    if not tool then return end
    
    _tp(hrp.CFrame * CFrame.new(0, _G.MobHeight, 0))
end

G.Kill2 = function(mob, distance)
    if not (mob and distance) then return end
    
    local hrp = mob:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    if not mob:GetAttribute("Locked") then
        mob:SetAttribute("Locked", hrp.CFrame)
    end
    
    PosMon = mob:GetAttribute("Locked").Position
    BringEnemy()
    EquipWeapon(_G.SelectWeapon)
    
    local tool = player.Character:FindFirstChildOfClass("Tool")
    if not tool then return end
    
    local tooltip = tool.ToolTip
    if tooltip == "Blox Fruit" then
        _tp((hrp.CFrame * CFrame.new(0, 10, 0)) * CFrame.Angles(0, math.rad(90), 0))
    else
        _tp((hrp.CFrame * CFrame.new(0, 20, 8)) * CFrame.Angles(0, math.rad(180), 0))
    end
    
    if RandomCFrame then
        task.wait(0.1)
        _tp(hrp.CFrame * CFrame.new(0, 30, 25))
        task.wait(0.1)
        _tp(hrp.CFrame * CFrame.new(25, 30, 0))
        task.wait(0.1)
        _tp(hrp.CFrame * CFrame.new(-25, 30, 0))
        task.wait(0.1)
        _tp(hrp.CFrame * CFrame.new(0, 30, 25))
        task.wait(0.1)
        _tp(hrp.CFrame * CFrame.new(-25, 30, 0))
    end
end

G.KillSea = function(mob, distance)
    if not (mob and distance) then return end
    
    local hrp = mob:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    if not mob:GetAttribute("Locked") then
        mob:SetAttribute("Locked", hrp.CFrame)
    end
    
    PosMon = mob:GetAttribute("Locked").Position
    BringEnemy()
    EquipWeapon(_G.SelectWeapon)
    
    local tool = player.Character:FindFirstChildOfClass("Tool")
    if not tool then return end
    
    if tool.ToolTip == "Blox Fruit" then
        _tp((hrp.CFrame * CFrame.new(0, 10, 0)) * CFrame.Angles(0, math.rad(90), 0))
    else
        notween(hrp.CFrame * CFrame.new(0, 50, 8))
        task.wait(0.85)
        notween(hrp.CFrame * CFrame.new(0, 400, 0))
        task.wait(1)
    end
end

G.Sword = function(mob, distance)
    if not (mob and distance) then return end
    
    local hrp = mob:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    if not mob:GetAttribute("Locked") then
        mob:SetAttribute("Locked", hrp.CFrame)
    end
    
    PosMon = mob:GetAttribute("Locked").Position
    BringEnemy()
    weaponSc("Sword")
    _tp(hrp.CFrame * CFrame.new(0, 30, 0))
    
    if RandomCFrame then
        task.wait(0.1)
        _tp(hrp.CFrame * CFrame.new(0, 30, 25))
        task.wait(0.1)
        _tp(hrp.CFrame * CFrame.new(25, 30, 0))
        task.wait(0.1)
        _tp(hrp.CFrame * CFrame.new(-25, 30, 0))
        task.wait(0.1)
        _tp(hrp.CFrame * CFrame.new(0, 30, 25))
        task.wait(0.1)
        _tp(hrp.CFrame * CFrame.new(-25, 30, 0))
    end
end

G.Mas = function(mob, distance)
    if not (mob and distance) then return end
    
    local hrp = mob:FindFirstChild("HumanoidRootPart")
    local hum = mob:FindFirstChildOfClass("Humanoid")
    if not hrp or not hum then return end
    
    if not mob:GetAttribute("Locked") then
        mob:SetAttribute("Locked", hrp.CFrame)
    end
    
    PosMon = mob:GetAttribute("Locked").Position
    BringEnemy()
    
    if hum.Health <= HealthM then
        _tp(hrp.CFrame * CFrame.new(0, 20, 0))
        UseFruitSkills()
    else
        weaponSc("Melee")
        _tp(hrp.CFrame * CFrame.new(0, 30, 0))
    end
end

G.Masgun = function(mob, distance)
    if not (mob and distance) then return end
    
    local hrp = mob:FindFirstChild("HumanoidRootPart")
    local hum = mob:FindFirstChildOfClass("Humanoid")
    if not hrp or not hum then return end
    
    if not mob:GetAttribute("Locked") then
        mob:SetAttribute("Locked", hrp.CFrame)
    end
    
    PosMon = mob:GetAttribute("Locked").Position
    BringEnemy()
    
    if hum.Health <= HealthM then
        _tp(hrp.CFrame * CFrame.new(0, 35, 8))
        Useskills("Gun", "Z")
        Useskills("Gun", "X")
    else
        weaponSc("Melee")
        _tp(hrp.CFrame * CFrame.new(0, 30, 0))
    end
end

-- ========================================
-- STATS
-- ========================================
statsSetings = function(statName, amount)
    if player.Data.Points.Value == 0 then return end
    
    local statMap = {
        Melee = "Melee",
        Defense = "Defense",
        Sword = "Sword",
        Gun = "Gun",
        Devil = "Demon Fruit"
    }
    
    local mapped = statMap[statName]
    if mapped then
        replicated.Remotes.CommF_:InvokeServer("AddPoint", mapped, amount)
    end
end

-- ========================================
-- CLEANUP (protegido)
-- ========================================
pcall(function()
    local rocks = workspace:FindFirstChild("Rocks")
    if rocks then rocks:Destroy() end
end)

pcall(function()
    local worldOrigin = workspace:FindFirstChild("_WorldOrigin")
    if worldOrigin then
        local foam = worldOrigin:FindFirstChild("Foam;")
        if foam then foam:Destroy() end
    end
end)

-- ========================================
-- HOOKS (protegidos)
-- ========================================
pcall(function()
    local deathModule = ReplicatedStorage:WaitForChild("Effect"):WaitForChild("Container"):WaitForChild("Death")
    hookfunction(require(deathModule), function() end)
end)

pcall(function()
    local guideModule = ReplicatedStorage:WaitForChild("GuideModule")
    hookfunction(require(guideModule).ChangeDisplayedNPC, function() end)
end)

print("✅ Zyn Hub Farm - Carregado com sucesso!")
