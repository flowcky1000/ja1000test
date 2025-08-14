-- Mod Menu Simples: ESP
-- Coloque em StarterPlayerScripts (LocalScript)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

-- Configurações
local ESPEnabled = false
local TeamCheck = true -- Ignorar time próprio

-- Criando GUI principal
local screenGui = Instance.new("ScreenGui", player.PlayerGui)
screenGui.Name = "ModMenu"

local menuFrame = Instance.new("Frame", screenGui)
menuFrame.Size = UDim2.new(0, 150, 0, 80)
menuFrame.Position = UDim2.new(0.5, -75, 0.5, -40)
menuFrame.AnchorPoint = Vector2.new(0.5, 0.5)
menuFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
menuFrame.BorderSizePixel = 0
menuFrame.Visible = true

-- Título do menu para arrastar
local menuTitle = Instance.new("TextLabel", menuFrame)
menuTitle.Size = UDim2.new(1, 0, 0, 20)
menuTitle.Text = "Menu"
menuTitle.TextColor3 = Color3.new(1, 1, 1)
menuTitle.TextScaled = true
menuTitle.BackgroundTransparency = 1
menuTitle.Position = UDim2.new(0, 0, 0, 0)

-- Lógica para arrastar o menu
local dragging
local dragInput
local dragStart
local startPos

menuTitle.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = menuFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

RunService.RenderStepped:Connect(function()
    if dragging and dragInput then
        local delta = dragInput.Position - dragStart
        menuFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- Botão para ativar/desativar o ESP
local espButton = Instance.new("TextButton", menuFrame)
espButton.Name = "ESPButton"
espButton.Size = UDim2.new(0, 130, 0, 30)
espButton.Position = UDim2.new(0, 10, 0, 30)
espButton.Text = "ESP: OFF"
espButton.TextScaled = true
espButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
espButton.TextColor3 = Color3.fromRGB(255, 255, 255)
espButton.BorderSizePixel = 0

espButton.MouseButton1Click:Connect(function()
    ESPEnabled = not ESPEnabled
    espButton.Text = ESPEnabled and "ESP: ON" or "ESP: OFF"
end)

-- Função para limpar o ESP de um jogador
local function cleanupESP(target)
    if target and target.Character then
        local espName = target.Character:FindFirstChild("ESP_Name")
        if espName then
            espName:Destroy()
        end
    end
end

-- Função de ESP
local function createESP(target)
    if not target or target == player or not target.Character or not target.Character:FindFirstChild("HumanoidRootPart") then
        cleanupESP(target)
        return
    end

    if TeamCheck and target.Team == player.Team then 
        cleanupESP(target)
        return
    end

    -- Lógica para criar ou remover o ESP
    if ESPEnabled then
        if not target.Character:FindFirstChild("ESP_Name") then
            local box = Instance.new("BillboardGui")
            box.Name = "ESP_Name"
            box.Size = UDim2.new(0, 100, 0, 50)
            box.AlwaysOnTop = true
            box.Parent = target.Character:WaitForChild("HumanoidRootPart")

            local label = Instance.new("TextLabel")
            label.Size = UDim2.new(1,0,1,0)
            label.BackgroundTransparency = 1
            label.TextColor3 = Color3.new(1,0,0)
            label.TextScaled = true
            label.Text = target.Name
            label.Parent = box
        end
    else
        cleanupESP(target)
    end
end

-- Loop de atualização do ESP
RunService.RenderStepped:Connect(function()
    for _, p in pairs(Players:GetPlayers()) do
        createESP(p)
    end
end)
