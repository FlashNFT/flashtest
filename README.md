--[[
    Script de Controle de Velocidade (v5 - Correção de Sintaxe e Diagnóstico)
    - Corrigido erro de digitação ('UDIm2' -> 'UDim2') que impedia o script de rodar.
    - Adicionados prints de diagnóstico para o console (aperte F9 no jogo para ver).
]]

--================================================================================--
--[ SERVIÇOS E VARIÁVEIS PRINCIPAIS ]
--================================================================================--
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local localPlayer = Players.LocalPlayer
local playerGui = localPlayer:WaitForChild("PlayerGui")

--================================================================================--
--[ INTERFACE GRÁFICA (GUI) ]
--================================================================================--
print("Iniciando criação da GUI...")

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "CustomSpeedGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = playerGui

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 200, 0, 100)
MainFrame.Position = UDim2.new(0, 10, 0, 10)
MainFrame.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
MainFrame.BorderSizePixel = 2
MainFrame.BorderColor3 = Color3.new(0.5, 0.5, 0.5)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Name = "Title"
TitleLabel.Size = UDim2.new(1, 0, 0, 20)
TitleLabel.BackgroundColor3 = Color3.new(0.3, 0.3, 0.3)
TitleLabel.TextColor3 = Color3.new(1, 1, 1)
TitleLabel.Text = "Controle de Velocidade"
TitleLabel.Font = "SourceSansBold"
TitleLabel.TextSize = 14
TitleLabel.Parent = MainFrame

local SpeedValueLabel = Instance.new("TextLabel")
SpeedValueLabel.Name = "SpeedValue"
SpeedValueLabel.Size = UDim2.new(0, 50, 0, 20)
SpeedValueLabel.Position = UDim2.new(1, -55, 0, 0)
SpeedValueLabel.BackgroundColor3 = Color3.fromRGB(51, 51, 51)
SpeedValueLabel.TextColor3 = Color3.new(1, 1, 0)
SpeedValueLabel.Text = "16x"
SpeedValueLabel.Font = "SourceSansBold"
SpeedValueLabel.Parent = TitleLabel

local SliderTrack = Instance.new("Frame")
SliderTrack.Name = "SliderTrack"
SliderTrack.Size = UDim2.new(0, 180, 0, 10)
SliderTrack.Position = UDim2.new(0, 10, 0, 35)
SliderTrack.BackgroundColor3 = Color3.new(0.4, 0.4, 0.4)
SliderTrack.Parent = MainFrame

local SliderHandle = Instance.new("TextButton")
SliderHandle.Name = "SliderHandle"
SliderHandle.Size = UDim2.new(0, 10, 0, 20)
SliderHandle.Position = UDim2.new(0, 0, 0, -5)
SliderHandle.BackgroundColor3 = Color3.new(0.8, 0.8, 0.8)
SliderHandle.Parent = SliderTrack

local ResetSpeedButton = Instance.new("TextButton")
ResetSpeedButton.Name = "ResetSpeedButton"
ResetSpeedButton.Size = UDim2.new(0, 180, 0, 25)
-- AQUI ESTAVA O ERRO -> 'UDIm2'. AGORA CORRIGIDO PARA 'UDim2'.
ResetSpeedButton.Position = UDim2.new(0, 10, 0, 60)
ResetSpeedButton.BackgroundColor3 = Color3.new(0.8, 0.5, 0.2)
ResetSpeedButton.TextColor3 = Color3.new(1, 1, 1)
ResetSpeedButton.Text = "Resetar Velocidade"
ResetSpeedButton.Font = "SourceSansBold"
ResetSpeedButton.Parent = MainFrame

print("GUI criada com sucesso.")

--================================================================================--
--[ LÓGICA DE VELOCIDADE - MÉTODO DE FORÇA FÍSICA ]
--================================================================================--

local MIN_SPEED = 16
local MAX_SPEED = 500

local originalSpeed = 16
local currentSpeed = 16
local draggingSlider = false
local speedEnabled = false

local physicsAttachment = nil
local linearVelocity = nil

local function setupPhysics(character)
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    print("Configurando física para o personagem...")
    
    if linearVelocity then linearVelocity:Destroy() end
    if physicsAttachment then physicsAttachment:Destroy() end

    local rootPart = character.HumanoidRootPart
    
    physicsAttachment = Instance.new("Attachment", rootPart)
    linearVelocity = Instance.new("LinearVelocity", physicsAttachment)
    
    linearVelocity.Attachment0 = physicsAttachment
    linearVelocity.MaxForce = math.huge
    linearVelocity.RelativeTo = "World"
    linearVelocity.VectorVelocity = Vector3.new(0, 0, 0)
    linearVelocity.Enabled = false
    print("Física configurada.")
end

local function onHeartbeat()
    if not speedEnabled or not localPlayer.Character or not linearVelocity then return end
    
    local humanoid = localPlayer.Character:FindFirstChildOfClass("Humanoid")
    if not humanoid or humanoid:GetState() == Enum.HumanoidStateType.Dead then
        linearVelocity.VectorVelocity = Vector3.new(0,0,0) -- Para a força se o jogador morrer
        return
    end
    
    local moveDirection = humanoid.MoveDirection
    local targetVelocity = Vector3.new(moveDirection.X * currentSpeed, 0, moveDirection.Z * currentSpeed)
    linearVelocity.VectorVelocity = targetVelocity
end

local function updateSpeedFromSlider()
    local handlePos = SliderHandle.Position.X.Offset
    local trackWidth = SliderTrack.AbsoluteSize.X
    local handleWidth = SliderHandle.AbsoluteSize.X
    if trackWidth <= handleWidth then return end
    
    local percentage = handlePos / (trackWidth - handleWidth)
    currentSpeed = MIN_SPEED + (MAX_SPEED - MIN_SPEED) * percentage
    SpeedValueLabel.Text = string.format("%.0fx", currentSpeed)
    
    if not speedEnabled then
        speedEnabled = true
        if linearVelocity then linearVelocity.Enabled = true end
        print("Força de velocidade ATIVADA.")
    end
end

ResetSpeedButton.MouseButton1Click:Connect(function()
    speedEnabled = false
    if linearVelocity then linearVelocity.Enabled = false end
    
    if localPlayer.Character and localPlayer.Character:FindFirstChild("Humanoid") then
        localPlayer.Character.Humanoid.WalkSpeed = originalSpeed
    end
    
    currentSpeed = originalSpeed
    SpeedValueLabel.Text = string.format("%.0fx", originalSpeed)
    local percentage = math.max(0, (originalSpeed - MIN_SPEED) / (MAX_SPEED - MIN_SPEED))
    local trackWidth = SliderTrack.AbsoluteSize.X
    local handleWidth = SliderHandle.AbsoluteSize.X
    local newX = percentage * (trackWidth - handleWidth)
    SliderHandle.Position = UDim2.new(0, newX, 0, -5)
    print("Força de velocidade DESATIVADA. Velocidade resetada.")
end)

SliderHandle.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then draggingSlider = true end end)
SliderHandle.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then draggingSlider = false end end)
UserInputService.InputChanged:Connect(function(input)
    if draggingSlider and input.UserInputType == Enum.UserInputType.MouseMovement then
        local mouseX = input.Position.X
        local trackStart = SliderTrack.AbsolutePosition.X
        local handleSizeX = SliderHandle.AbsoluteSize.X
        local trackSizeX = SliderTrack.AbsoluteSize.X
        local newX = math.clamp(mouseX - trackStart - (handleSizeX / 2), 0, trackSizeX - handleSizeX)
        SliderHandle.Position = UDim2.new(0, newX, 0, -5)
        updateSpeedFromSlider()
    end
end)

--================================================================================--
--[ INICIALIZAÇÃO ]
--================================================================================--
local function initialize()
    local character = localPlayer.Character or localPlayer.CharacterAdded:Wait()
    setupPhysics(character)
    local humanoid = character:WaitForChild("Humanoid")
    originalSpeed = humanoid.WalkSpeed
    currentSpeed = originalSpeed
    SpeedValueLabel.Text = string.format("%.0fx", originalSpeed)
end

localPlayer.CharacterAdded:Connect(setupPhysics)

initialize()
RunService.Heartbeat:Connect(onHeartbeat)
print("Script de Velocidade (Método de Força Física) carregado e pronto.")
