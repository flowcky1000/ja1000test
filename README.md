-- Script de ESP Educativo para Testes
-- Coloque em LocalScript dentro de StarterPlayerScripts

-- Configurações
local ESPEnabled = false
local TeamCheck = true -- Ignora membros do mesmo time

-- Função para criar a GUI
local player = game.Players.LocalPlayer
local screenGui = Instance.new("ScreenGui", player.PlayerGui)
screenGui.Name = "ESP_GUI"

local toggleButton = Instance.new("TextButton", screenGui)
toggleButton.Size = UDim2.new(0, 150, 0, 50)
toggleButton.Position = UDim2.new(0, 20, 0, 20)
toggleButton.Text = "ESP: OFF"

toggleButton.MouseButton1Click:Connect(function()
    ESPEnabled = not ESPEnabled
    toggleButton.Text = ESPEnabled and "ESP: ON" or "ESP: OFF"
end)

-- Função de ESP
local function createESP(playerTarget)
    if playerTarget == player then return end -- ignora você mesmo
    if TeamCheck and playerTarget.Team == player.Team then return end

    local box = Instance.new("BillboardGui")
    box.Size = UDim2.new(0, 100, 0, 50)
    box.AlwaysOnTop = true
    box.Parent = playerTarget.Character:WaitForChild("HumanoidRootPart")

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1,0,1,0)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.new(1,0,0)
    label.TextScaled = true
    label.Text = playerTarget.Name
    label.Parent = box
end

-- Loop de atualização do ESP
game:GetService("RunService").RenderStepped:Connect(function()
    if ESPEnabled then
        for _, p in pairs(game.Players:GetPlayers()) do
            if p.Character and not p.Character:FindFirstChild("ESP_Box") then
                createESP(p)
            end
        end
    else
        -- Remove ESP se desligado
        for _, p in pairs(game.Players:GetPlayers()) do
            if p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                local esp = p.Character.HumanoidRootPart:FindFirstChildOfClass("BillboardGui")
                if esp then esp:Destroy() end
            end
        end
    end
end)
