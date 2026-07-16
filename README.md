-- [[ DELTA MOBILE HUB - ORBIT & HITBOX ]] --

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- Configurações
_G.OrbitActive = false
_G.HitboxActive = false
local OrbitDistance = 5 -- 5 Studs de distância
local HitboxSize = 10   -- 10 Studs de hitbox
local angle = 0

-- Interface (GUI)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "DeltaMobileHub"
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Botão Hub (Abrir/Fechar)
local HubBtn = Instance.new("TextButton")
HubBtn.Name = "HubButton"
HubBtn.Parent = ScreenGui
HubBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
HubBtn.Position = UDim2.new(0, 10, 0.5, 0)
HubBtn.Size = UDim2.new(0, 60, 0, 60)
HubBtn.Text = "HUB"
HubBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
HubBtn.Font = Enum.Font.GothamBold
HubBtn.Active = true
HubBtn.Draggable = true

-- Painel Principal
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
MainFrame.Position = UDim2.new(0.2, 0, 0.3, 0)
MainFrame.Size = UDim2.new(0, 200, 0, 150)
MainFrame.Visible = false
MainFrame.Active = true

-- Lógica do Hub Abrir/Fechar
HubBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

-- Botões do Painel
local function CreateButton(name, pos, text)
    local btn = Instance.new("TextButton", MainFrame)
    btn.Name = name
    btn.Size = UDim2.new(0.8, 0, 0, 40)
    btn.Position = pos
    btn.BackgroundColor3 = Color3.fromRGB(180, 40, 40)
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.GothamSemibold
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
    return btn
end

local OrbitBtn = CreateButton("OrbitBtn", UDim2.new(0.1, 0, 0.15, 0), "ORBIT: OFF")
local HitboxBtn = CreateButton("HitboxBtn", UDim2.new(0.1, 0, 0.55, 0), "HITBOX: OFF")

-- Draggable Logic (Arrastar Painel)
local dragging, dragInput, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.Touch then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.Touch then dragging = false end end)

-- Funções de Lógica
OrbitBtn.MouseButton1Click:Connect(function()
    _G.OrbitActive = not _G.OrbitActive
    OrbitBtn.Text = _G.OrbitActive and "ORBIT: ON" or "ORBIT: OFF"
    OrbitBtn.BackgroundColor3 = _G.OrbitActive and Color3.fromRGB(40, 180, 40) or Color3.fromRGB(180, 40, 40)
end)

HitboxBtn.MouseButton1Click:Connect(function()
    _G.HitboxActive = not _G.HitboxActive
    HitboxBtn.Text = _G.HitboxActive and "HITBOX: ON" or "HITBOX: OFF"
    HitboxBtn.BackgroundColor3 = _G.HitboxActive and Color3.fromRGB(40, 180, 40) or Color3.fromRGB(180, 40, 40)
    
    if not _G.HitboxActive then
        -- Resetar Hitbox ao desligar
        for _, player in pairs(Players:GetPlayers()) do
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                player.Character.HumanoidRootPart.Size = Vector3.new(2, 2, 1)
            end
        end
    end
end)

-- Main Loop
RunService.Heartbeat:Connect(function(dt)
    -- Lógica do Orbit (5 studs)
    if _G.OrbitActive then
        local closest = nil
        local dist = 100
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                local d = (p.Character.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
                if d < dist then dist = d; closest = p end
            end
        end
        if closest and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            angle = angle + 3 * dt
            local offset = Vector3.new(math.cos(angle) * OrbitDistance, 0, math.sin(angle) * OrbitDistance)
            LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(closest.Character.HumanoidRootPart.Position + offset, closest.Character.HumanoidRootPart.Position)
        end
    end

    -- Lógica do Hitbox (10 studs)
    if _G.HitboxActive then
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                local hrp = p.Character.HumanoidRootPart
                hrp.Size = Vector3.new(HitboxSize, HitboxSize, HitboxSize)
                hrp.Transparency = 0.7
                hrp.CanCollide = false
            end
        end
    end
end)
