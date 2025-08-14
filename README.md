-- Script com menu aprimorado
-- Coloque em StarterPlayerScripts (LocalScript)

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

-- Variáveis de controle
local jumpEnabled = false
local defaultWalkSpeed = 16 -- Velocidade padrão do Roblox
local currentWalkSpeed = defaultWalkSpeed

-- Criando a GUI
local screenGui = Instance.new("ScreenGui", player.PlayerGui)
screenGui.Name = "ModMenu"

local menuFrame = Instance.new("Frame", screenGui)
menuFrame.Size = UDim2.new(0, 200, 0, 150)
menuFrame.Position = UDim2.new(0.5, -100, 0.5, -75)
menuFrame.AnchorPoint = Vector2.new(0.5, 0.5)
menuFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
menuFrame.BorderSizePixel = 0

-- Título do menu
local menuTitle = Instance.new("TextLabel", menuFrame)
menuTitle.Size = UDim2.new(1, 0, 0, 20)
menuTitle.Text = "Menu"
menuTitle.TextColor3 = Color3.new(1, 1, 1)
menuTitle.TextScaled = true
menuTitle.BackgroundTransparency = 1

-- Botão de pulo infinito
local jumpButton = Instance.new("TextButton", menuFrame)
jumpButton.Size = UDim2.new(0, 180, 0, 30)
jumpButton.Position = UDim2.new(0.5, -90, 0, 30)
jumpButton.Text = "Pulo Infinito: OFF"
jumpButton.TextScaled = true
jumpButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
jumpButton.TextColor3 = Color3.fromRGB(255, 255, 255)
jumpButton.BorderSizePixel = 0

jumpButton.MouseButton1Click:Connect(function()
    jumpEnabled = not jumpEnabled
    jumpButton.Text = "Pulo Infinito: " .. (jumpEnabled and "ON" or "OFF")
end)

-- Lógica para o pulo infinito
UserInputService.JumpRequest:Connect(function()
    if jumpEnabled and player and player.Character then
        local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        end
    end
end)

-- Label para o slider de velocidade
local speedLabel = Instance.new("TextLabel", menuFrame)
speedLabel.Size = UDim2.new(0, 180, 0, 20)
speedLabel.Position = UDim2.new(0.5, -90, 0, 70)
speedLabel.Text = "Velocidade: 16"
speedLabel.TextColor3 = Color3.new(1, 1, 1)
speedLabel.BackgroundTransparency = 1

-- Slider de velocidade
local speedSlider = Instance.new("Frame", menuFrame)
speedSlider.Size = UDim2.new(0, 180, 0, 10)
speedSlider.Position = UDim2.new(0.5, -90, 0, 95)
speedSlider.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
speedSlider.BorderSizePixel = 0

local speedHandle = Instance.new("Frame", speedSlider)
speedHandle.Size = UDim2.new(0, 10, 1, 0)
speedHandle.Position = UDim2.new(0, 0, 0, 0)
speedHandle.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
speedHandle.BorderSizePixel = 0

local draggingSlider = false
local function updateSpeed(input)
    if not draggingSlider then return end
    
    local pos = input.Position.X - speedSlider.AbsolutePosition.X
    local sliderWidth = speedSlider.AbsoluteSize.X
    local ratio = math.clamp(pos / sliderWidth, 0, 1)
    
    local newSpeed = math.floor(ratio * 100) + 16 -- De 16 a 116
    
    currentWalkSpeed = newSpeed
    speedHandle.Position = UDim2.new(ratio, 0, 0, 0)
    speedLabel.Text = "Velocidade: " .. tostring(newSpeed)
    
    if player and player.Character then
        local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = newSpeed
        end
    end
end

speedSlider.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingSlider = true
        updateSpeed(input)
    end
end)

speedSlider.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingSlider = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and draggingSlider then
        updateSpeed(input)
    end
end)

-- Lógica para manter a velocidade mesmo se o jogador morrer
player.CharacterAdded:Connect(function(character)
    character:WaitForChild("Humanoid").WalkSpeed = currentWalkSpeed
end)
