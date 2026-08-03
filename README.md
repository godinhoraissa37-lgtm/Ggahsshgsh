-- ========================================================
-- PAINEL MOBILE COM BOTÃO FLUTUANTE E LOOP DE TELEPORTE (EXATO)
-- ========================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- Garante que o painel antigo seja removido se você executar de novo
if game:GetService("CoreGui"):FindFirstChild("MobileTPPanel") then
    game:GetService("CoreGui").MobileTPPanel:Destroy()
end

-- ScreenGui Principal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MobileTPPanel"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game:GetService("CoreGui")

-----------------------------------------------------------
-- 1. BOTÃO FLUTUANTE (BOLINHA ARRASTÁVEL)
-----------------------------------------------------------
local OpenButton = Instance.new("TextButton")
OpenButton.Name = "OpenButton"
OpenButton.Size = UDim2.new(0, 50, 0, 50)
OpenButton.Position = UDim2.new(0, 20, 0.4, 0)
OpenButton.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
OpenButton.Text = "TP"
OpenButton.TextColor3 = Color3.fromRGB(255, 255, 255)
OpenButton.TextSize = 18
OpenButton.Font = Enum.Font.SourceSansBold
OpenButton.Active = true
OpenButton.Parent = ScreenGui

-- Bordas arredondadas na bolinha
local OpenCorner = Instance.new("UICorner")
OpenCorner.CornerRadius = UDim.new(1, 0)
OpenCorner.Parent = OpenButton

local OpenStroke = Instance.new("UIStroke")
OpenStroke.Color = Color3.fromRGB(0, 170, 255)
OpenStroke.Thickness = 2
OpenStroke.Parent = OpenButton

-----------------------------------------------------------
-- 2. PAINEL PRINCIPAL (JANELA ARRASTÁVEL)
-----------------------------------------------------------
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 260, 0, 170)
MainFrame.Position = UDim2.new(0.5, -130, 0.4, -85)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Visible = false -- Inicia escondido
MainFrame.Parent = ScreenGui

-- Bordas arredondadas no painel
local FrameCorner = Instance.new("UICorner")
FrameCorner.CornerRadius = UDim.new(0, 12)
FrameCorner.Parent = MainFrame

local FrameStroke = Instance.new("UIStroke")
FrameStroke.Color = Color3.fromRGB(50, 50, 60)
FrameStroke.Thickness = 2
FrameStroke.Parent = MainFrame

-- Barra de Título (Barra Superior para Arrastar)
local TopBar = Instance.new("Frame")
TopBar.Name = "TopBar"
TopBar.Size = UDim2.new(1, 0, 0, 35)
TopBar.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
TopBar.BorderSizePixel = 0
TopBar.Parent = MainFrame

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, -40, 1, 0)
TitleLabel.Position = UDim2.new(0, 10, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "Painel Teleporte"
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleLabel.TextSize = 16
TitleLabel.Font = Enum.Font.SourceSansBold
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.Parent = TopBar

-- Botão Fechar (X)
local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0, 30, 0, 30)
CloseButton.Position = UDim2.new(1, -32, 0, 2)
CloseButton.BackgroundTransparency = 1
CloseButton.Text = "X"
CloseButton.TextColor3 = Color3.fromRGB(200, 50, 50)
CloseButton.TextSize = 18
CloseButton.Font = Enum.Font.SourceSansBold
CloseButton.Parent = TopBar

-- Botão Toggle (Liga / Desliga)
local ToggleButton = Instance.new("TextButton")
ToggleButton.Name = "ToggleButton"
ToggleButton.Size = UDim2.new(0.85, 0, 0, 45)
ToggleButton.Position = UDim2.new(0.075, 0, 0.5, -10)
ToggleButton.BackgroundColor3 = Color3.fromRGB(180, 40, 40)
ToggleButton.Text = "TP Loop: DESATIVADO"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.TextSize = 15
ToggleButton.Font = Enum.Font.SourceSansBold
ToggleButton.Parent = MainFrame

local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(0, 8)
ToggleCorner.Parent = ToggleButton

-----------------------------------------------------------
-- 3. FUNÇÃO DE ARRASTAR COM O DEDO (TOUCH / MOUSE)
-----------------------------------------------------------
local function makeDraggable(guiObject)
    local dragging = false
    local dragInput, dragStart, startPos

    local function update(input)
        local delta = input.Position - dragStart
        guiObject.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )
    end

    guiObject.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = guiObject.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    guiObject.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            update(input)
        end
    end)
end

makeDraggable(OpenButton)
makeDraggable(MainFrame)

-----------------------------------------------------------
-- 4. LÓGICA DO BOTÃO ABRIR / FECHAR
-----------------------------------------------------------
OpenButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

CloseButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
end)

-----------------------------------------------------------
-- 5. LÓGICA DE TELEPORTE DENTRO DOS PLAYERS (0.2s)
-----------------------------------------------------------
local isTpActive = false

local function startTpLoop()
    task.spawn(function()
        while isTpActive do
            local allPlayers = Players:GetPlayers()
            
            for _, targetPlayer in ipairs(allPlayers) do
                if not isTpActive then break end

                if targetPlayer ~= LocalPlayer then
                    local myChar = LocalPlayer.Character
                    local targetChar = targetPlayer.Character

                    if myChar and myChar:FindFirstChild("HumanoidRootPart") and targetChar and targetChar:FindFirstChild("HumanoidRootPart") then
                        -- Teleporta EXATAMENTE na posição/orientação do jogador alvo (dentro dele)
                        myChar.HumanoidRootPart.CFrame = targetChar.HumanoidRootPart.CFrame
                    end

                    task.wait(0.2)
                end
            end

            task.wait(0.1)
        end
    end)
end

ToggleButton.MouseButton1Click:Connect(function()
    isTpActive = not isTpActive

    if isTpActive then
        ToggleButton.BackgroundColor3 = Color3.fromRGB(40, 180, 40)
        ToggleButton.Text = "TP Loop: ATIVADO"
        startTpLoop()
    else
        ToggleButton.BackgroundColor3 = Color3.fromRGB(180, 40, 40)
        ToggleButton.Text = "TP Loop: DESATIVADO"
    end
end)
