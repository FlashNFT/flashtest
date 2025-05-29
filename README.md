-- ✔ AUTO FARM + FRUIT + QUEST + GUI + MOB SELECIONÁVEL

-- ▼ CONFIGS GLOBAIS
getgenv().Settings = {
    AutoFarm = false,
    AutoFruit = false,
    AutoQuest = false,
    SelectedWeapon = "Combat",
    SelectedMob = "Bandit"
}

-- ▼ LIB GUI
local lib = loadstring(game:HttpGet("https://raw.githubusercontent.com/bloodball/-back-ups-for-libs/main/watermark"))()

local win = lib:Window("Blox Fruits Autofarm", "Por ChatGPT (Uso autorizado)", Color3.fromRGB(44, 120, 224), Enum.KeyCode.RightControl)

-- ▼ ABA DE CONTROLE
local tab = win:Tab("Farm")
tab:Toggle("AutoFarm", function(val)
    getgenv().Settings.AutoFarm = val
end)
tab:Toggle("AutoFruit", function(val)
    getgenv().Settings.AutoFruit = val
end)
tab:Toggle("AutoQuest", function(val)
    getgenv().Settings.AutoQuest = val
end)
tab:Textbox("Nome do Mob", "Ex: Bandit", function(val)
    getgenv().Settings.SelectedMob = val
end)
tab:Textbox("Nome da Arma", "Ex: Combat", function(val)
    getgenv().Settings.SelectedWeapon = val
end)

-- ▼ SERVIÇOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer

local function GetChar()
    return LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
end

local function IsAlive()
    return GetChar() and GetChar():FindFirstChild("Humanoid") and GetChar().Humanoid.Health > 0
end

local function SafeTP(pos)
    local hrp = GetChar():FindFirstChild("HumanoidRootPart")
    if hrp then
        local tween = TweenService:Create(hrp, TweenInfo.new(0.5), {CFrame = CFrame.new(pos)})
        tween:Play()
        tween.Completed:Wait()
    end
end

local function EquipWeapon()
    local char = GetChar()
    local tool = char:FindFirstChild(getgenv().Settings.SelectedWeapon) or LocalPlayer.Backpack:FindFirstChild(getgenv().Settings.SelectedWeapon)
    if tool then
        tool.Parent = char
    end
end

local function FindMobByName()
    for _, mob in pairs(Workspace.Enemies:GetChildren()) do
        if mob.Name == getgenv().Settings.SelectedMob and mob:FindFirstChild("Humanoid") and mob.Humanoid.Health > 0 then
            return mob
        end
    end
end

local function AttackMob(mob)
    EquipWeapon()
    repeat
        if mob and mob:FindFirstChild("HumanoidRootPart") then
            SafeTP(mob.HumanoidRootPart.Position + Vector3.new(0, 3, 0))
            pcall(function()
                GetChar()[getgenv().Settings.SelectedWeapon]:Activate()
            end)
        end
        wait(0.2)
    until mob.Humanoid.Health <= 0 or not getgenv().Settings.AutoFarm or not IsAlive()
end

-- ▼ QUEST SYSTEM
local function StartQuest()
    local QuestData = {
        ["Bandit"] = {QuestName = "BanditQuest1", QuestLevel = 1, QuestNPC = "Bandit Quest Giver [Lv. 5]", Pos = Vector3.new(1060, 17, 1547)},
        -- Adicione mais inimigos e dados aqui conforme necessário
    }
    local info = QuestData[getgenv().Settings.SelectedMob]
    if info then
        SafeTP(info.Pos + Vector3.new(0, 3, 0))
        wait(1)
        local npc = Workspace:FindFirstChild(info.QuestNPC)
        if npc then
            fireclickdetector(npc:FindFirstChildOfClass("ClickDetector"))
            wait(1)
            game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("StartQuest", info.QuestName, info.QuestLevel)
        end
    end
end

-- ▼ AUTO FARM LOOP
spawn(function()
    while true do
        wait(1)
        if getgenv().Settings.AutoFarm and IsAlive() then
            if getgenv().Settings.AutoQuest then
                StartQuest()
            end
            local mob = FindMobByName()
            if mob then
                AttackMob(mob)
            end
        end
    end
end)

-- ▼ AUTO FRUIT COLETA
spawn(function()
    while wait(5) do
        if getgenv().Settings.AutoFruit then
            for _, v in pairs(Workspace:GetDescendants()) do
                if v:IsA("Tool") and string.find(v.Name:lower(), "fruit") then
                    pcall(function()
                        SafeTP(v.Handle.Position)
                        wait(0.3)
                        firetouchinterest(GetChar().HumanoidRootPart, v.Handle, 0)
                        firetouchinterest(GetChar().HumanoidRootPart, v.Handle, 1)
                    end)
                end
            end
        end
    end
end)
