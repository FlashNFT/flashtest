-- CONFIGURAÇÕES GLOBAIS
local Settings = {
    AutoFarm = false,
    AutoFruit = false,
    AutoQuest = false,
    SelectedMob = "Bandit",
    SelectedWeapon = "Combat"
}

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer

-- Espera personagem carregar
local function GetCharacter()
    local char = LocalPlayer.Character
    if not char or not char.Parent then
        char = LocalPlayer.CharacterAdded:Wait()
    end
    return char
end

local function IsAlive()
    local char = GetCharacter()
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    return hum and hum.Health > 0
end

local function SafeTP(pos)
    local char = GetCharacter()
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if hrp then
        local tween = TweenService:Create(hrp, TweenInfo.new(0.5), {CFrame = CFrame.new(pos)})
        tween:Play()
        tween.Completed:Wait()
    end
end

local function EquipWeapon()
    local char = GetCharacter()
    local tool = char:FindFirstChild(Settings.SelectedWeapon) or LocalPlayer.Backpack:FindFirstChild(Settings.SelectedWeapon)
    if tool and tool.Parent ~= char then
        tool.Parent = char
        wait(0.5)
    end
end

local function FindMobByName()
    local enemies = Workspace:FindFirstChild("Enemies")
    if not enemies then return nil end
    for _, mob in pairs(enemies:GetChildren()) do
        if mob.Name == Settings.SelectedMob and mob:FindFirstChildOfClass("Humanoid") and mob.Humanoid.Health > 0 then
            return mob
        end
    end
    return nil
end

local function AttackMob(mob)
    if not mob or not mob:FindFirstChild("HumanoidRootPart") then return end
    EquipWeapon()
    repeat
        if not IsAlive() or not Settings.AutoFarm or mob.Humanoid.Health <= 0 then break end
        SafeTP(mob.HumanoidRootPart.Position + Vector3.new(0, 3, 0))
        local char = GetCharacter()
        local weapon = char:FindFirstChild(Settings.SelectedWeapon)
        if weapon then
            pcall(function()
                weapon:Activate()
            end)
        end
        wait(0.3)
    until mob.Humanoid.Health <= 0 or not Settings.AutoFarm or not IsAlive()
end

local function StartQuest()
    local QuestNPCName = "Bandit Quest Giver [Lv. 5]"
    local QuestName = "BanditQuest1"
    local QuestLevel = 1

    local npc = Workspace:FindFirstChild(QuestNPCName)
    if not npc then
        SafeTP(Vector3.new(1060, 17, 1547))
        wait(1)
        npc = Workspace:FindFirstChild(QuestNPCName)
    end

    if npc and npc:FindFirstChildOfClass("ClickDetector") then
        pcall(function()
            fireclickdetector(npc:FindFirstChildOfClass("ClickDetector"))
        end)
        wait(1)
        if ReplicatedStorage:FindFirstChild("Remotes") and ReplicatedStorage.Remotes:FindFirstChild("CommF_") then
            local success, err = pcall(function()
                ReplicatedStorage.Remotes.CommF_:InvokeServer("StartQuest", QuestName, QuestLevel)
            end)
            if not success then
                warn("Erro ao iniciar quest:", err)
            end
        else
            warn("Remotes.CommF_ não encontrado!")
        end
    else
        warn("NPC ou ClickDetector da Quest não encontrado!")
    end
end

-- Auto Fruit coleta
spawn(function()
    while wait(5) do
        if Settings.AutoFruit and IsAlive() then
            for _, item in pairs(Workspace:GetDescendants()) do
                if item:IsA("Tool") and string.find(item.Name:lower(), "fruit") and item:FindFirstChild("Handle") then
                    SafeTP(item.Handle.Position)
                    wait(0.2)
                    local char = GetCharacter()
                    pcall(function()
                        firetouchinterest(char.HumanoidRootPart, item.Handle, 0)
                        firetouchinterest(char.HumanoidRootPart, item.Handle, 1)
                    end)
                    wait(1)
                end
            end
        end
    end
end)

-- AutoFarm loop
spawn(function()
    while wait(1) do
        if Settings.AutoFarm and IsAlive() then
            if Settings.AutoQuest then
                StartQuest()
                wait(5)
            end
            local mob = FindMobByName()
            if mob then
                AttackMob(mob)
            else
                wait(2)
            end
        end
    end
end)
