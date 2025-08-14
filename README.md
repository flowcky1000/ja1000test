-- Mod Menu Educativo: ESP + HitBox
-- Coloque em StarterPlayerScripts (LocalScript)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local workspace = game:GetService("Workspace")

-- Configurações
local ESPEnabled = false
local HitBoxEnabled = false
local TeamCheck = true -- Ignorar time próprio

-- Criando GUI principal
local screenGui = Instance.new("ScreenGui", player.PlayerGui)
screenGui.Name = "ModMenu"

local menuFrame = Instance.new("Frame", screenGui)
menuFrame.Size = UDim2.new(0, 250, 0, 150)
menuFrame.Position = UDim2.new(0.5, -125, 0.5, -75) -- Centraliza o menu
menuFrame.AnchorPoint = Vector2.new(0.5, 0.5)
menuFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
menuFrame.BorderSizePixel = 0
menuFrame.Visible = true

-- Título do menu
local menuTitle = Instance.new("TextLabel", menuFrame)
menuTitle.Size = UDim2.new(1, 0, 0, 20)
menuTitle.Text = "Mod Menu Educativo"
menuTitle.TextColor3 = Color3.new(1, 1, 1)
menuTitle.TextScaled = true
menuTitle.BackgroundTransparency = 1
menuTitle.Position = UDim2.new(0, 0, 0, 0)

-- Permite arrastar o menu
local dragging
local dragInput
local dragStart
local startPos

local function startDrag(input)
    dragging = true
    dragStart = input.Position
    startPos = menuFrame.Position
    input.InputChanged:Connect(function(input2)
        if input2 == dragInput and dragging then
            local delta = input2.Position - dragStart
            menuFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

local function stopDrag()
    dragging = false
end

menuTitle.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
        startDrag(input)
    end
end)

menuTitle.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        stopDrag()
    end
end)

-- Dicionário para armazenar os botões e facilitar o acesso
local buttons = {}

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
    buttons[text] = btn
    return btn
end

-- Toggle ESP
createButton("ESP: OFF", 30, function()
    ESPEnabled = not ESPEnabled
    buttons["ESP: OFF"].Text = ESPEnabled and "ESP: ON" or "ESP: OFF"
end)

-- Toggle HitBox
createButton("HitBox: OFF", 80, function()
    HitBoxEnabled = not HitBoxEnabled
    buttons["HitBox: OFF"].Text = HitBoxEnabled and "HitBox: ON" or "HitBox: OFF"
end)

-- Removendo elementos antigos
local function cleanup(target)
    if target.Character then
        local espName = target.Character:FindFirstChild("ESP_Name")
        if espName then
            espName:Destroy()
        end
        local hitBoxes = target.Character:GetChildren()
        for _, part in ipairs(hitBoxes) do
            if part.Name == "HitBox" then
                part:Destroy()
            end
        end
    end
end

-- Dicionário para armazenar as hitboxes de cada jogador
local hitboxes = {}

-- Função de ESP
local function createESP(target)
    if not target or target == player or not target.Character or not target.Character:FindFirstChild("HumanoidRootPart") then
        cleanup(target)
        return
    end

    -- Limpeza de ESPs anteriores, caso o jogador não esteja mais na lista
    if not ESPEnabled and target.Character:FindFirstChild("ESP_Name") then
        target.Character:FindFirstChild("ESP_Name"):Destroy()
    end

    if not HitBoxEnabled then
        for _, part in pairs(hitboxes[target] or {}) do
            part:Destroy()
        end
        hitboxes[target] = nil
    end

    if TeamCheck and target.Team == player.Team then 
        cleanup(target)
        return
    end

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
    if HitBoxEnabled and not hitboxes[target] then
        local function drawLimb(part1, part2)
            if not part1 or not part2 then return end
            local line = Instance.new("Part")
            line.Name = "HitBox"
            line.Anchored = true
            line.CanCollide = false
            line.Size = Vector3.new(0.2, 0.2, (part1.Position - part2.Position).Magnitude)
            line.CFrame = CFrame.new(part1.Position, part2.Position) * CFrame.new(0,0,-line.Size.Z/2)
            line.BrickColor = BrickColor.new("Bright blue")
            line.Material = Enum.Material.Neon
            line.Transparency = 0.5
            line.Parent = workspace
            return line
        end

        local limbsToDraw = {}
        local character = target.Character
        if character then
            local root = character:FindFirstChild("HumanoidRootPart")
            local head = character:FindFirstChild("Head")
            local leftUpperArm = character:FindFirstChild("LeftUpperArm")
            local rightUpperArm = character:FindFirstChild("RightUpperArm")
            local leftUpperLeg = character:FindFirstChild("LeftUpperLeg")
            local rightUpperLeg = character:FindFirstChild("RightUpperLeg")

            if root and head and leftUpperArm and rightUpperArm and leftUpperLeg and rightUpperLeg then
                limbsToDraw = {
                    {root, head},
                    {root, leftUpperArm},
                    {root, rightUpperArm},
                    {root, leftUpperLeg},
                    {root, rightUpperLeg},
                    {leftUpperArm, character:FindFirstChild("LeftLowerArm")},
                    {rightUpperArm, character:FindFirstChild("RightLowerArm")},
                    {leftUpperLeg, character:FindFirstChild("LeftLowerLeg")},
                    {rightUpperLeg, character:FindFirstChild("RightLowerLeg")},
                }
            end
        end

        local currentHitboxes = {}
        for _, pair in ipairs(limbsToDraw) do
            local line = drawLimb(pair[1], pair[2])
            if line then
                table.insert(currentHitboxes, line)
            end
        end
        hitboxes[target] = currentHitboxes
    end
end

-- Atualização do ESP/HitBox
RunService.RenderStepped:Connect(function()
    -- Garante que os jogadores que saíram do jogo não tenham elementos na tela
    local currentPlayers = {}
    for _, p in pairs(Players:GetPlayers()) do
        currentPlayers[p] = true
        createESP(p)
    end
    for p, _ in pairs(hitboxes) do
        if not currentPlayers[p] then
            cleanup(p)
            hitboxes[p] = nil
        end
    end

    -- Atualiza as hitboxes existentes
    if HitBoxEnabled then
        for p, boxes in pairs(hitboxes) do
            local character = p.Character
            if character then
                local root = character:FindFirstChild("HumanoidRootPart")
                if root then
                    for i, box in ipairs(boxes) do
                        local part1 = box.CFrame.p -- A hitbox original não tem a referência do Part, então precisamos recriá-la
                        local part2 = root -- Usaremos a raiz como base para o cálculo

                        -- Atualiza a hitbox
                        if i == 1 then -- Exemplo: da cabeça ao torso
                           local head = character:FindFirstChild("Head")
                           if head then
                               box.CFrame = CFrame.new(root.Position, head.Position) * CFrame.new(0, 0, -(root.Position - head.Position).Magnitude/2)
                               box.Size = Vector3.new(0.2, 0.2, (root.Position - head.Position).Magnitude)
                           end
                        end
                        -- Lógica similar para outras partes do corpo
                    end
                end
            end
        end
    end
end)
