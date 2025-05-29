-- AutoFarm + AutoFruit + AutoQuest + GUI simples (autônomo)

-- CONFIGURAÇÕES GLOBAIS
local Settings = {
    AutoFarm = false,
    AutoFruit = false,
    AutoQuest = false,
    SelectedMob = "Bandit",
    SelectedWeapon = "Combat"
}

-- SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer

-- FUNÇÃO PARA PEGAR O PERSONAGEM SEGURO
local function GetCharacter()
    return LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
end

-- FUNÇÃO PARA CHECAR SE ESTÁ VIVO
local function IsAlive()
    local char = GetCharacter()
    return char and char:FindFirstChild("Humanoid") and char.Humanoid.Health > 0
end

-- TELEPORTE SUAVE
local function SafeTP(pos)
    local hrp = GetCharacter():FindFirstChild("HumanoidRootPart")
    if hrp then
        local tween = TweenService:Create(hrp, TweenInfo.new(0.5), {CFrame = CFrame.new(pos)})
        tween:Play()
        tween.Completed:Wait()
    end
end

-- EQUIPAR ARMA
local function EquipWeapon()
    local char = GetCharacter()
    local tool = char:FindFirstChild(Settings.SelectedWeapon) or LocalPlayer.Backpack:FindFirstChild(Settings.SelectedWeapon)
    if tool and tool.Parent ~= char then
        tool.Parent = char
        wait(0.5)
    end
end

-- ENCONTRAR MOB PELO NOME
local function FindMobByName()
    for _, mob in pairs(Workspace.Enemies:GetChildren()) do
        if mob.Name == Settings.SelectedMob and mob:FindFirstChild("Humanoid") and mob.Humanoid.Health > 0 then
            return mob
        end
    end
    return nil
end

-- ATAQUE AUTOMÁTICO
local function AttackMob(mob)
    EquipWeapon()
    if not mob or not mob:FindFirstChild("HumanoidRootPart") then return end
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

-- INICIAR QUEST (exemplo com Bandit Quest)
local function StartQuest()
    local QuestNPCName = "Bandit Quest Giver [Lv. 5]"
    local QuestName = "BanditQuest1"
    local QuestLevel = 1
    local npc = Workspace:FindFirstChild(QuestNPCName)
    if not npc then
        -- Posição aproximada do NPC para ir pegar
        SafeTP(Vector3.new(1060, 17, 1547))
        wait(1)
        npc = Workspace:FindFirstChild(QuestNPCName)
    end
    if npc and npc:FindFirstChildOfClass("ClickDetector") then
        fireclickdetector(npc:FindFirstChildOfClass("ClickDetector"))
        wait(1)
        local success, err = pcall(function()
            ReplicatedStorage.Remotes.CommF_:InvokeServer("StartQuest", QuestName, QuestLevel)
        end)
        if not success then
            warn("Erro ao iniciar quest:", err)
        end
    end
end

-- AUTO COLETA DE FRUTAS
spawn(function()
    while true do
        wait(5)
        if Settings.AutoFruit and IsAlive() then
            for _, item in pairs(Workspace:GetDescendants()) do
                if item:IsA("Tool") and string.find(item.Name:lower(), "fruit") and item:FindFirstChild("Handle") then
                    SafeTP(item.Handle.Position)
                    wait(0.2)
                    local char = GetCharacter()
                    firetouchinterest(char.HumanoidRootPart, item.Handle, 0)
                    firetouchinterest(char.HumanoidRootPart, item.Handle, 1)
                    wait(1)
                end
            end
        else
            wait(1)
        end
    end
end)

-- AUTO FARM LOOP
spawn(function()
    while true do
        wait(1)
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
        else
            wait(1)
        end
    end
end)

-- GUI SIMPLES
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "BloxFruitsAutoFarmGui"
ScreenGui.Parent = game.CoreGui

local Frame = Instance.new("Frame", ScreenGui)
Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Frame.Size = UDim2.new(0, 280, 0, 250)
Frame.Position = UDim2.new(0, 20, 0, 20)
Frame.Active = true
Frame.Draggable = true

local function createLabel(text, posY)
    local label = Instance.new("TextLabel", Frame)
    label.Text = text
    label.TextColor3 = Color3.new(1, 1, 1)
    label.BackgroundTransparency = 1
    label.Size = UDim2.new(1, -20, 0, 20)
    label.Position = UDim2.new(0, 10, 0, posY)
    label.Font = Enum.Font.SourceSans
    label.TextSize = 18
    label.TextXAlignment = Enum.TextXAlignment.Left
    return label
end

local function createToggle(text, posY, callback)
    local button = Instance.new("TextButton", Frame)
    button.Text = text .. " [OFF]"
    button.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
    button.TextColor3 = Color3.new(1, 1, 1)
    button.Size = UDim2.new(1, -20, 0, 30)
    button.Position = UDim2.new(0, 10, 0, posY)
    button.Font = Enum.Font.SourceSansBold
    button.TextSize = 20
    local toggled = false
    button.MouseButton1Click:Connect(function()
        toggled = not toggled
        if toggled then
            button.Text = text .. " [ON]"
        else
            button.Text = text .. " [OFF]"
        end
        callback(toggled)
    end)
    return button
end

local function createTextbox(placeholder, posY, callback)
    local box = Instance.new("TextBox", Frame)
    box.PlaceholderText = placeholder
    box.Text = ""
    box.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
    box.TextColor3 = Color3.new(1, 1, 1)
    box.Size = UDim2.new(1, -20, 0, 30)
    box.Position = UDim2.new(0, 10, 0, posY)
    box.ClearTextOnFocus = false
    box.Font = Enum.Font.SourceSans
    box.TextSize = 18
    box.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            callback(box.Text)
        end
    end)
    return box
end

createLabel("Blox Fruits AutoFarm", 10)

createToggle("AutoFarm", 40, function(val) Settings.AutoFarm = val end)
createToggle("AutoFruit", 80, function(val) Settings.AutoFruit = val end)
createToggle("AutoQuest", 120, function(val) Settings.AutoQuest = val end)

createLabel("Nome do Mob:", 160)
createTextbox("Ex: Bandit", 185, function(val)
    if val ~= "" then
        Settings.SelectedMob = val
    end
end)

createLabel("Nome da Arma:", 220)
createTextbox("Ex: Combat", 245, function(val)
    if val ~= "" then
        Settings.SelectedWeapon = val
    end
end)
