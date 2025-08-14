-- Script de pulo infinito com menu
-- Coloque em StarterPlayerScripts (LocalScript)

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

-- Variável de controle
local jumpEnabled = false

-- Criando a GUI
local screenGui = Instance.new("ScreenGui", player.PlayerGui)
screenGui.Name = "JumpModMenu"

local menuFrame = Instance.new("Frame", screenGui)
menuFrame.Size = UDim2.new(0, 150, 0, 50)
menuFrame.Position = UDim2.new(0.5, -75, 0.5, -25)
menuFrame.AnchorPoint = Vector2.new(0.5, 0.5)
menuFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
menuFrame.BorderSizePixel = 0

-- Botão de ativação/desativação
local jumpButton = Instance.new("TextButton", menuFrame)
jumpButton.Size = UDim2.new(0, 130, 0, 30)
jumpButton.Position = UDim2.new(0.5, -65, 0.5, -15)
jumpButton.AnchorPoint = Vector2.new(0.5, 0.5)
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
