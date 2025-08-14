-- Mod Menu Educativo: ESP + HitBox
-- Coloque em StarterPlayerScripts (LocalScript)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

-- Configurações
local ESPEnabled = false
local HitBoxEnabled = false
local TeamCheck = true -- Ignorar time próprio

-- Criando GUI principal
local screenGui = Instance.new("ScreenGui", player.PlayerGui)
screenGui.Name = "ModMenu"

local menuFrame = Instance.new("Frame", screenGui)
menuFrame.Size = UDim2.new(0, 250, 0, 150)
menuFrame.Position = UDim2.new(0, 20, 0, 20)
menuFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
menuFrame.BorderSizePixel = 0
menuFrame.Visible = true

-- Função para criar botões
local function createButton(text, yPos, callback)
    local btn = Instance.new("TextButton", menuFrame)
    btn.Size = UDim2.new(0, 230, 0, 40)
    btn.Position = UDim2.new(0, 10, 0, yPos)
    btn.Text = text
    btn.TextScaled = true
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.BorderSizePixel = 0
    btn.MouseButton1Click:Connect(callback)
    return btn
end

-- Toggle ESP
createButton("ESP: OFF", 10, function()
    ESPEnabled = not ESPEnabled
    menuFrame:FindFirstChild("ESP: OFF").Text = ESPEnabled and "ESP: ON" or "ESP: OFF"
end)

-- Toggle HitBox
createButton("HitBox: OFF", 60, function()
    HitBoxEnabled = not HitBoxEnabled
    menuFrame:FindFirstChild("HitBox: OFF").Text = HitBoxEnabled and "HitBox: ON" or "HitBox: OFF"
end)

-- Função de ESP
local function createESP(target)
    if target == player then return end
    if TeamCheck and target.Team == player.Team then return end
    if not target.Character then return end

    -- Nome (ESP)
    if ESPEnabled and not target.Character:FindFirstChild("ESP_Name") then
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

    -- HitBox / Esqueleto
    if HitBoxEnabled and not target.Character:FindFirstChild("HitBox") then
        local function drawLimb(part1, part2)
            local line = Instance.new("Part")
            line.Name = "HitBox"
            line.Anchored = true
            line.CanCollide = false
            line.Size = Vector3.new(0.2,0.2,(part1.Position - part2.Position).Magnitude)
            line.CFrame = CFrame.new(part1.Position, part2.Position) * CFrame.new(0,0,-line.Size.Z/2)
            line.BrickColor = BrickColor.new("Bright blue")
            line.Parent = workspace
            return line
        end

        local hum = target.Character:FindFirstChild("Humanoid")
        local root = target.Character:FindFirstChild("HumanoidRootPart")
        if hum and root then
            local limbs = {
                {root, target.Character:FindFirstChild("Head")},
                {root, target.Character:FindFirstChild("LeftUpperArm")},
                {root, target.Character:FindFirstChild("RightUpperArm")},
                {root, target.Character:FindFirstChild("LeftUpperLeg")},
                {root, target.Character:FindFirstChild("RightUpperLeg")},
            }
            for _, pair in ipairs(limbs) do
                if pair[1] and pair[2] then
                    drawLimb(pair[1], pair[2])
                end
            end
        end
    end
end

-- Atualização do ESP/HitBox
RunService.RenderStepped:Connect(function()
    for _, p in pairs(Players:GetPlayers()) do
        createESP(p)
    end
end)
