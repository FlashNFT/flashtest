--[[ 
  Auto Farm Avançado Blox Fruits (Roblox)
  GUI completa e detecção automática de mobs/quests
  Uso autorizado para testes/monitoramento!
]]

-- Baixe OrionLib: https://raw.githubusercontent.com/shlexware/Orion/main/source
local OrionLib = loadstring(game:HttpGet("https://raw.githubusercontent.com/shlexware/Orion/main/source"))()
local mobsList = {}
local selectedMob = nil
local autofarm = false

-- Função para detectar todos os mobs disponíveis
function UpdateMobsList()
    mobsList = {}
    for _, mob in pairs(workspace.Enemies:GetChildren()) do
        if mob:FindFirstChild("HumanoidRootPart") and mob:FindFirstChild("Humanoid") and mob.Humanoid.Health > 0 then
            if not table.find(mobsList, mob.Name) then
                table.insert(mobsList, mob.Name)
            end
        end
    end
    table.sort(mobsList)
end

-- Função para encontrar a quest relacionada ao mob
function FindQuestForMob(mobName)
    local questData = {
        -- Adicione/edite aqui para mobs/quests específicos se precisar precisão máxima
        ["Bandit"] = {"BanditQuest1", "Bandit"},
        ["Monkey"] = {"MonkeyQuest", "Monkey"},
        ["Gorilla"] = {"GorillaQuest", "Gorilla"},
        ["Brute"] = {"BruteQuest", "Brute"},
        -- Autodetecta no padrão se não estiver listado
    }
    if questData[mobName] then
        return unpack(questData[mobName])
    end
    -- Tenta adivinhar nomes padronizados
    local questName = mobName.."Quest"
    return questName, mobName
end

-- Função para pegar quest automaticamente
function TakeQuest(questName)
    local player = game.Players.LocalPlayer
    local npcFolder = workspace:FindFirstChild("NPCs") or workspace:FindFirstChild("QuestNPCs") or workspace:FindFirstChild("Quests")
    if npcFolder then
        for _, npc in pairs(npcFolder:GetChildren()) do
            if npc.Name:find(questName) and npc:FindFirstChild("Head") then
                player.Character.HumanoidRootPart.CFrame = npc.Head.CFrame + Vector3.new(0,2,0)
                wait(0.6)
                fireclickdetector(npc:FindFirstChildWhichIsA("ClickDetector"))
                wait(1)
                break
            end
        end
    end
end

-- Função para atacar mob
function FarmMob(mobName)
    local player = game.Players.LocalPlayer
    local mobs = workspace.Enemies:GetChildren()
    local closest, dist = nil, math.huge
    for _, mob in pairs(mobs) do
        if mob.Name == mobName and mob:FindFirstChild("HumanoidRootPart") and mob.Humanoid.Health > 0 then
            local d = (player.Character.HumanoidRootPart.Position - mob.HumanoidRootPart.Position).Magnitude
            if d < dist then
                dist = d
                closest = mob
            end
        end
    end
    if closest then
        player.Character.HumanoidRootPart.CFrame = closest.HumanoidRootPart.CFrame + Vector3.new(0,4,0)
        wait(0.15)
        -- Simula ataques rápidos
        for i = 1,5 do
            game.VirtualInputManager:SendMouseButtonEvent(0,0,0,true,game,0)
            wait(0.03)
            game.VirtualInputManager:SendMouseButtonEvent(0,0,0,false,game,0)
        end
    end
end

-- Loop principal do autofarm
spawn(function()
    while true do
        wait(0.4)
        if autofarm and selectedMob ~= nil then
            -- Garante atualização da lista de mobs
            UpdateMobsList()
            -- Quest
            local questName, mobName = FindQuestForMob(selectedMob)
            local gui = game.Players.LocalPlayer.PlayerGui:FindFirstChild("QuestGUI")
            if not gui or not gui.Enabled then
                TakeQuest(questName)
            else
                FarmMob(mobName)
            end
        end
    end
end)

-- Interface avançada com OrionLib
local Window = OrionLib:MakeWindow({Name = "Blox Fruits AutoFarm Avançado", HidePremium = false, SaveConfig = true, ConfigFolder = "BFARMADV"})
local Tab = Window:MakeTab({Name = "AutoFarm", Icon = "rbxassetid://4483345998", PremiumOnly = false})

Tab:AddDropdown({
    Name = "Selecionar Mob para Farmar",
    Default = "",
    Options = mobsList,
    Callback = function(Value)
        selectedMob = Value
    end
})

Tab:AddButton({
    Name = "Atualizar Lista de Mobs",
    Callback = function()
        UpdateMobsList()
        OrionLib:MakeNotification({Name="Mobs Atualizados!", Content="Lista de mobs recarregada.", Image="rbxassetid://4483345998", Time=2})
    end
})

Tab:AddToggle({
    Name = "Ativar AutoFarm",
    Default = false,
    Callback = function(Value)
        autofarm = Value
        OrionLib:MakeNotification({Name="AutoFarm", Content=Value and "Ativado!" or "Desativado!", Image="rbxassetid://4483345998", Time=2})
    end
})

OrionLib:Init()

-- Carrega mobs ao iniciar o script
UpdateMobsList()
