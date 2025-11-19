-- Script para Roblox Studio - Bolinha Móvel com Categorias
-- Coloque este script em StarterGui ou StarterPlayerScripts

local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Criar ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FloatingOrbGui"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = playerGui

-- Criar a bolinha flutuante
local orb = Instance.new("TextButton")
orb.Name = "Orb"
orb.Size = UDim2.new(0, 60, 0, 60)
orb.Position = UDim2.new(0.05, 0, 0.3, 0)
orb.BackgroundColor3 = Color3.fromRGB(75, 150, 255)
orb.BorderSizePixel = 0
orb.Text = ""
orb.AutoButtonColor = false
orb.ZIndex = 10
orb.Parent = screenGui

-- Arredondar a bolinha
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(1, 0)
corner.Parent = orb

-- Adicionar borda preta
local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(0, 0, 0)
stroke.Thickness = 4
stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
stroke.Parent = orb

-- Adicionar gradiente
local gradient = Instance.new("UIGradient")
gradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(100, 180, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(50, 120, 200))
}
gradient.Rotation = 45
gradient.Parent = orb

-- Criar ícone de mira/alvo no centro (estilo caveira com mira)
local targetIcon = Instance.new("ImageLabel")
targetIcon.Name = "TargetIcon"
targetIcon.Size = UDim2.new(1, 0, 1, 0)
targetIcon.Position = UDim2.new(0.5, 0, 0.5, 0)
targetIcon.AnchorPoint = Vector2.new(0.5, 0.5)
targetIcon.BackgroundTransparency = 1
targetIcon.Image = "rbxassetid://8964489645"
targetIcon.ImageColor3 = Color3.fromRGB(255, 255, 255)
targetIcon.ScaleType = Enum.ScaleType.Fit
targetIcon.ZIndex = 11
targetIcon.Parent = orb

-- Arredondar a imagem também
local imageCorner = Instance.new("UICorner")
imageCorner.CornerRadius = UDim.new(1, 0)
imageCorner.Parent = targetIcon

-- Adicionar texto emoji como alternativa caso a imagem não carregue
local emojiLabel = Instance.new("TextLabel")
emojiLabel.Name = "EmojiIcon"
emojiLabel.Size = UDim2.new(1, 0, 1, 0)
emojiLabel.Position = UDim2.new(0.5, 0, 0.5, 0)
emojiLabel.AnchorPoint = Vector2.new(0.5, 0.5)
emojiLabel.BackgroundTransparency = 1
emojiLabel.Text = "🎯"
emojiLabel.TextSize = 35
emojiLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
emojiLabel.Font = Enum.Font.GothamBold
emojiLabel.ZIndex = 10
emojiLabel.Parent = orb

-- Criar botão de teleporte flutuante
local tpButton = Instance.new("TextButton")
tpButton.Name = "TeleportButton"
tpButton.Size = UDim2.new(0, 100, 0, 45)
tpButton.Position = UDim2.new(1, -120, 0, 80)
tpButton.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
tpButton.BorderSizePixel = 0
tpButton.Text = "TP"
tpButton.TextColor3 = Color3.fromRGB(255, 255, 255)
tpButton.TextSize = 16
tpButton.Font = Enum.Font.GothamBold
tpButton.Visible = false
tpButton.ZIndex = 200
tpButton.Parent = screenGui

local tpCorner = Instance.new("UICorner")
tpCorner.CornerRadius = UDim.new(0, 8)
tpCorner.Parent = tpButton

local tpStroke = Instance.new("UIStroke")
tpStroke.Color = Color3.fromRGB(255, 50, 50)
tpStroke.Thickness = 2
tpStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
tpStroke.Parent = tpButton

-- Criar botão de fly flutuante
local flyButton = Instance.new("TextButton")
flyButton.Name = "FlyButton"
flyButton.Size = UDim2.new(0, 100, 0, 45)
flyButton.Position = UDim2.new(1, -120, 0, 135)
flyButton.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
flyButton.BorderSizePixel = 0
flyButton.Text = "FLY"
flyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
flyButton.TextSize = 16
flyButton.Font = Enum.Font.GothamBold
flyButton.Visible = false
flyButton.ZIndex = 200
flyButton.Parent = screenGui

local flyCorner = Instance.new("UICorner")
flyCorner.CornerRadius = UDim.new(0, 8)
flyCorner.Parent = flyButton

local flyStroke = Instance.new("UIStroke")
flyStroke.Color = Color3.fromRGB(255, 50, 50)
flyStroke.Thickness = 2
flyStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
flyStroke.Parent = flyButton

-- Criar painel de controle principal
local panel = Instance.new("Frame")
panel.Name = "ControlPanel"
panel.Size = UDim2.new(0, 260, 0, 357)
panel.Position = UDim2.new(0.5, -130, 0.5, -178.5)
panel.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
panel.BackgroundTransparency = 0.2
panel.BorderSizePixel = 0
panel.Visible = false
panel.ZIndex = 100
panel.Parent = screenGui

local panelCorner = Instance.new("UICorner")
panelCorner.CornerRadius = UDim.new(0, 12)
panelCorner.Parent = panel

-- Título do painel
local title = Instance.new("TextLabel")
title.Name = "Title"
title.Size = UDim2.new(1, 0, 0, 38)
title.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
title.BackgroundTransparency = 0.3
title.BorderSizePixel = 0
title.Text = "Soute Hub"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 16
title.Font = Enum.Font.GothamBold
title.ZIndex = 101
title.Parent = panel

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 12)
titleCorner.Parent = title

-- Criar botões de categoria
local function createCategoryButton(name, text, position, icon)
    local button = Instance.new("TextButton")
    button.Name = name
    button.Size = UDim2.new(0.44, 0, 0, 35)
    button.Position = position
    button.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
    button.BorderSizePixel = 0
    button.Text = text
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 13
    button.Font = Enum.Font.GothamBold
    button.ZIndex = 102
    button.Parent = panel
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 8)
    btnCorner.Parent = button
    
    local btnStroke = Instance.new("UIStroke")
    btnStroke.Color = Color3.fromRGB(0, 0, 0)
    btnStroke.Thickness = 2
    btnStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    btnStroke.Parent = button
    
    return button
end

-- Criar categorias
local movementBtn = createCategoryButton("MovementButton", "Movement", UDim2.new(0.06, 0, 0, 45), "")
local settingsBtn = createCategoryButton("SettingsButton", "Settings", UDim2.new(0.5, 0, 0, 45), "")

-- Criar painéis de categorias
local function createCategoryPanel(name)
    local catPanel = Instance.new("Frame")
    catPanel.Name = name
    catPanel.Size = UDim2.new(0.92, 0, 0, 267)
    catPanel.Position = UDim2.new(0.04, 0, 0, 87)
    catPanel.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    catPanel.BackgroundTransparency = 0.3
    catPanel.BorderSizePixel = 0
    catPanel.Visible = false
    catPanel.ZIndex = 101
    catPanel.Parent = panel
    
    local catCorner = Instance.new("UICorner")
    catCorner.CornerRadius = UDim.new(0, 8)
    catCorner.Parent = catPanel
    
    return catPanel
end

local movementPanel = createCategoryPanel("MovementPanel")
local settingsPanel = createCategoryPanel("SettingsPanel")

-- Função para criar botões nas categorias
local function createButton(name, text, position, parent)
    local button = Instance.new("TextButton")
    button.Name = name
    button.Size = UDim2.new(0.88, 0, 0, 35)
    button.Position = position
    button.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
    button.BorderSizePixel = 0
    button.Text = text
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 13
    button.Font = Enum.Font.GothamBold
    button.ZIndex = 102
    button.Parent = parent
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 7)
    btnCorner.Parent = button
    
    local btnStroke = Instance.new("UIStroke")
    btnStroke.Color = Color3.fromRGB(0, 0, 0)
    btnStroke.Thickness = 2
    btnStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    btnStroke.Parent = button
    
    return button
end

-- Criar botões de Movement
local saveBtn = createButton("SaveButton", "Salvar Local", UDim2.new(0.06, 0, 0, 8), movementPanel)
local returnBtn = createButton("ReturnButton", "Desync Tp v2", UDim2.new(0.06, 0, 0, 50), movementPanel)

-- Botão Infinite Jump
local infJumpBtn = createButton("InfJumpButton", "Infinite Jump", UDim2.new(0.06, 0, 0, 137), movementPanel)

-- Botão Fly
local flyBtn = createButton("FlyButton", "Fly", UDim2.new(0.06, 0, 0, 179), movementPanel)

-- Criar controle de velocidade
local speedFrame = Instance.new("Frame")
speedFrame.Name = "SpeedFrame"
speedFrame.Size = UDim2.new(0.88, 0, 0, 42)
speedFrame.Position = UDim2.new(0.06, 0, 0, 92)
speedFrame.BackgroundColor3 = Color3.fromRGB(55, 55, 65)
speedFrame.BorderSizePixel = 0
speedFrame.ZIndex = 102
speedFrame.Parent = movementPanel

local speedCorner = Instance.new("UICorner")
speedCorner.CornerRadius = UDim.new(0, 7)
speedCorner.Parent = speedFrame

local speedLabel = Instance.new("TextLabel")
speedLabel.Name = "SpeedLabel"
speedLabel.Size = UDim2.new(0.5, 0, 1, 0)
speedLabel.Position = UDim2.new(0, 8, 0, 0)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "Velocidade: 16"
speedLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
speedLabel.TextSize = 12
speedLabel.Font = Enum.Font.GothamBold
speedLabel.TextXAlignment = Enum.TextXAlignment.Left
speedLabel.ZIndex = 103
speedLabel.Parent = speedFrame

local decreaseBtn = Instance.new("TextButton")
decreaseBtn.Name = "DecreaseButton"
decreaseBtn.Size = UDim2.new(0, 32, 0, 32)
decreaseBtn.Position = UDim2.new(1, -75, 0.5, -16)
decreaseBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
decreaseBtn.BorderSizePixel = 0
decreaseBtn.Text = "-"
decreaseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
decreaseBtn.TextSize = 18
decreaseBtn.Font = Enum.Font.GothamBold
decreaseBtn.ZIndex = 103
decreaseBtn.Parent = speedFrame

local decreaseCorner = Instance.new("UICorner")
decreaseCorner.CornerRadius = UDim.new(0, 5)
decreaseCorner.Parent = decreaseBtn

local decreaseStroke = Instance.new("UIStroke")
decreaseStroke.Color = Color3.fromRGB(0, 0, 0)
decreaseStroke.Thickness = 2
decreaseStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
decreaseStroke.Parent = decreaseBtn

local increaseBtn = Instance.new("TextButton")
increaseBtn.Name = "IncreaseButton"
increaseBtn.Size = UDim2.new(0, 32, 0, 32)
increaseBtn.Position = UDim2.new(1, -38, 0.5, -16)
increaseBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
increaseBtn.BorderSizePixel = 0
increaseBtn.Text = "+"
increaseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
increaseBtn.TextSize = 18
increaseBtn.Font = Enum.Font.GothamBold
increaseBtn.ZIndex = 103
increaseBtn.Parent = speedFrame

local increaseCorner = Instance.new("UICorner")
increaseCorner.CornerRadius = UDim.new(0, 5)
increaseCorner.Parent = increaseBtn

local increaseStroke = Instance.new("UIStroke")
increaseStroke.Color = Color3.fromRGB(0, 0, 0)
increaseStroke.Thickness = 2
increaseStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
increaseStroke.Parent = increaseBtn

-- Criar botões de Settings
local espBtn = createButton("ESPButton", "Wall Esp", UDim2.new(0.06, 0, 0, 8), settingsPanel)
local serverHopBtn = createButton("ServerHopButton", "Server Hop", UDim2.new(0.06, 0, 0, 50), settingsPanel)
local rejoinBtn = createButton("RejoinButton", "Server Rejoin", UDim2.new(0.06, 0, 0, 92), settingsPanel)

-- Variáveis de controle
local panelOpen = false
local currentCategory = nil
local savedPosition = nil
local espEnabled = false
local espHighlights = {}
local currentSpeed = 16
local tpButtonVisible = false
local infJumpEnabled = false
local infJumpConnection = nil
local flyEnabled = false
local flyConnection = nil
local flyBodyVelocity = nil
local flyBodyGyro = nil

-- Sistema de arrastar a bolinha
local dragging = false
local dragInput
local dragStart
local startPos

local function updateInput(input)
    local delta = input.Position - dragStart
    orb.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

orb.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = orb.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

orb.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        updateInput(input)
    end
end)

-- Sistema de arrastar o painel
local panelDragging = false
local panelDragInput
local panelDragStart
local panelStartPos

local function updatePanelInput(input)
    local delta = input.Position - panelDragStart
    panel.Position = UDim2.new(panelStartPos.X.Scale, panelStartPos.X.Offset + delta.X, panelStartPos.Y.Scale, panelStartPos.Y.Offset + delta.Y)
end

title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        panelDragging = true
        panelDragStart = input.Position
        panelStartPos = panel.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                panelDragging = false
            end
        end)
    end
end)

title.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        panelDragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == panelDragInput and panelDragging then
        updatePanelInput(input)
    end
end)

-- Função para mostrar categoria
local function showCategory(categoryName)
    movementPanel.Visible = false
    settingsPanel.Visible = false
    
    movementBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
    settingsBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
    
    if categoryName == "movement" then
        movementPanel.Visible = true
        movementBtn.BackgroundColor3 = Color3.fromRGB(150, 30, 30)
        title.Text = "Soute Hub - Movement"
    elseif categoryName == "settings" then
        settingsPanel.Visible = true
        settingsBtn.BackgroundColor3 = Color3.fromRGB(150, 30, 30)
        title.Text = "Soute Hub - Settings"
    end
    
    currentCategory = categoryName
end

-- Função para voltar ao menu principal
local function showMainMenu()
    movementPanel.Visible = false
    settingsPanel.Visible = false
    currentCategory = nil
    title.Text = "Soute Hub"
    
    movementBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
    settingsBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
end

-- Função para abrir/fechar painel
local function togglePanel()
    panelOpen = not panelOpen
    
    if panelOpen then
        panel.Visible = true
        panel.Position = UDim2.new(0.5, -130, 1, 0)
        showCategory("movement") -- Abre direto em Movement
        local tween = TweenService:Create(panel, TweenInfo.new(0.3, Enum.EasingStyle.Back), {
            Position = UDim2.new(0.5, -130, 0.5, -178.5)
        })
        tween:Play()
    else
        local tween = TweenService:Create(panel, TweenInfo.new(0.2, Enum.EasingStyle.Back), {
            Position = UDim2.new(0.5, -130, 1, 0)
        })
        tween:Play()
        wait(0.2)
        panel.Visible = false
        showMainMenu()
    end
end

-- Clique na bolinha
orb.MouseButton1Click:Connect(function()
    if not dragging then
        togglePanel()
    end
end)

-- Cliques nas categorias
movementBtn.MouseButton1Click:Connect(function()
    showCategory("movement")
end)

settingsBtn.MouseButton1Click:Connect(function()
    showCategory("settings")
end)

-- Função salvar posição
saveBtn.MouseButton1Click:Connect(function()
    local character = player.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        savedPosition = character.HumanoidRootPart.CFrame
        saveBtn.Text = "Local Salvo!"
        wait(1)
        saveBtn.Text = "Salvar Local"
    end
end)

-- Função voltar à posição
returnBtn.MouseButton1Click:Connect(function()
    if not savedPosition then
        returnBtn.Text = "Sem posição"
        wait(1)
        returnBtn.Text = "Desync Tp v2"
        return
    end
    
    -- Toggle do botão de teleporte
    tpButtonVisible = not tpButtonVisible
    
    if tpButtonVisible then
        tpButton.Visible = true
        returnBtn.Text = "Desativar TP"
        returnBtn.BackgroundColor3 = Color3.fromRGB(150, 30, 30)
    else
        tpButton.Visible = false
        returnBtn.Text = "Desync Tp v2"
        returnBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
    end
end)

-- Função do botão de teleporte flutuante
tpButton.MouseButton1Click:Connect(function()
    local character = player.Character
    if character and character:FindFirstChild("HumanoidRootPart") and savedPosition then
        character.HumanoidRootPart.CFrame = savedPosition
        tpButton.Text = "Teleportado!"
        wait(0.5)
        tpButton.Text = "TP"
    end
end)

-- Função para atualizar velocidade
local function updateSpeed()
    local character = player.Character
    if character and character:FindFirstChild("Humanoid") then
        character.Humanoid.WalkSpeed = currentSpeed
    end
end

-- Diminuir velocidade
decreaseBtn.MouseButton1Click:Connect(function()
    if currentSpeed > 16 then
        currentSpeed = currentSpeed - 1
        speedLabel.Text = "Velocidade: " .. currentSpeed
        updateSpeed()
    end
end)

-- Aumentar velocidade
increaseBtn.MouseButton1Click:Connect(function()
    if currentSpeed < 30 then
        currentSpeed = currentSpeed + 1
        speedLabel.Text = "Velocidade: " .. currentSpeed
        updateSpeed()
    end
end)

-- Manter velocidade ao respawnar
player.CharacterAdded:Connect(function(character)
    wait(0.5)
    if character:FindFirstChild("Humanoid") then
        character.Humanoid.WalkSpeed = currentSpeed
        
        -- Reativar Infinite Jump se estava ativo
        if infJumpEnabled then
            setupInfiniteJump(character)
        end
        
        -- Resetar fly se estava ativo
        if flyEnabled then
            flyEnabled = false
            flyButton.Visible = false
            flyBtn.Text = "Fly"
            flyBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
        end
    end
end)

-- Função para configurar Infinite Jump
local function setupInfiniteJump(character)
    if infJumpConnection then
        infJumpConnection:Disconnect()
    end
    
    local humanoid = character:FindFirstChild("Humanoid")
    if humanoid then
        infJumpConnection = UserInputService.JumpRequest:Connect(function()
            if infJumpEnabled then
                humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            end
        end)
    end
end

-- Toggle Infinite Jump
infJumpBtn.MouseButton1Click:Connect(function()
    infJumpEnabled = not infJumpEnabled
    
    if infJumpEnabled then
        infJumpBtn.Text = "Infinite Jump [ON]"
        infJumpBtn.BackgroundColor3 = Color3.fromRGB(150, 30, 30)
        
        local character = player.Character
        if character then
            setupInfiniteJump(character)
        end
    else
        infJumpBtn.Text = "Infinite Jump"
        infJumpBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
        
        if infJumpConnection then
            infJumpConnection:Disconnect()
            infJumpConnection = nil
        end
    end
end)

-- Função para controlar o fly
local function setupFly(character)
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    if not humanoidRootPart then return end
    
    -- Criar BodyVelocity
    flyBodyVelocity = Instance.new("BodyVelocity")
    flyBodyVelocity.Velocity = Vector3.new(0, 0, 0)
    flyBodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
    flyBodyVelocity.Parent = humanoidRootPart
    
    -- Criar BodyGyro
    flyBodyGyro = Instance.new("BodyGyro")
    flyBodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
    flyBodyGyro.P = 9e4
    flyBodyGyro.Parent = humanoidRootPart
    
    -- Controle de movimento
    if flyConnection then
        flyConnection:Disconnect()
    end
    
    flyConnection = game:GetService("RunService").RenderStepped:Connect(function()
        if not flyEnabled or not character or not humanoidRootPart then
            if flyConnection then
                flyConnection:Disconnect()
            end
            return
        end
        
        local camera = workspace.CurrentCamera
        flyBodyGyro.CFrame = camera.CFrame
        
        local moveDirection = Vector3.new(0, 0, 0)
        
        -- Detectar teclas (PC)
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then
            moveDirection = moveDirection + camera.CFrame.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then
            moveDirection = moveDirection - camera.CFrame.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then
            moveDirection = moveDirection - camera.CFrame.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then
            moveDirection = moveDirection + camera.CFrame.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
            moveDirection = moveDirection + Vector3.new(0, 1, 0)
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
            moveDirection = moveDirection - Vector3.new(0, 1, 0)
        end
        
        flyBodyVelocity.Velocity = moveDirection.Unit * 45
    end)
end

-- Função para desativar fly
local function disableFly()
    if flyConnection then
        flyConnection:Disconnect()
        flyConnection = nil
    end
    
    if flyBodyVelocity then
        flyBodyVelocity:Destroy()
        flyBodyVelocity = nil
    end
    
    if flyBodyGyro then
        flyBodyGyro:Destroy()
        flyBodyGyro = nil
    end
end

-- Toggle Fly no painel
flyBtn.MouseButton1Click:Connect(function()
    flyEnabled = not flyEnabled
    
    if flyEnabled then
        flyButton.Visible = true
        flyBtn.Text = "Desativar Fly"
        flyBtn.BackgroundColor3 = Color3.fromRGB(150, 30, 30)
        
        local character = player.Character
        if character then
            setupFly(character)
        end
    else
        flyButton.Visible = false
        flyBtn.Text = "Fly"
        flyBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
        disableFly()
    end
end)

-- Botão de fly flutuante
flyButton.MouseButton1Click:Connect(function()
    flyEnabled = not flyEnabled
    
    if flyEnabled then
        flyButton.Text = "FLY [ON]"
        flyButton.BackgroundColor3 = Color3.fromRGB(150, 30, 30)
        flyBtn.Text = "Desativar Fly"
        flyBtn.BackgroundColor3 = Color3.fromRGB(150, 30, 30)
        
        local character = player.Character
        if character then
            setupFly(character)
        end
    else
        flyButton.Text = "FLY"
        flyButton.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
        flyBtn.Text = "Fly"
        flyBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
        disableFly()
    end
end)

-- Função para criar ESP em um jogador
local function createESP(targetPlayer)
    if targetPlayer == player then return end
    
    local function setupESP()
        local character = targetPlayer.Character
        if not character then return end
        
        if espHighlights[targetPlayer] then
            espHighlights[targetPlayer]:Destroy()
        end
        
        local highlight = Instance.new("Highlight")
        highlight.Name = "ESP_Highlight"
        highlight.FillColor = Color3.fromRGB(255, 50, 50)
        highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
        highlight.FillTransparency = 0.5
        highlight.OutlineTransparency = 0
        highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        highlight.Parent = character
        
        espHighlights[targetPlayer] = highlight
    end
    
    setupESP()
    
    targetPlayer.CharacterAdded:Connect(function()
        if espEnabled then
            wait(0.5)
            setupESP()
        end
    end)
end

-- Função para remover ESP
local function removeESP(targetPlayer)
    if espHighlights[targetPlayer] then
        espHighlights[targetPlayer]:Destroy()
        espHighlights[targetPlayer] = nil
    end
end

-- Toggle ESP
espBtn.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    
    if espEnabled then
        espBtn.Text = "Wall Esp [ON]"
        espBtn.BackgroundColor3 = Color3.fromRGB(150, 30, 30)
        
        for _, targetPlayer in pairs(Players:GetPlayers()) do
            createESP(targetPlayer)
        end
        
        Players.PlayerAdded:Connect(function(targetPlayer)
            if espEnabled then
                targetPlayer.CharacterAdded:Wait()
                wait(0.5)
                createESP(targetPlayer)
            end
        end)
    else
        espBtn.Text = "Wall Esp"
        espBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
        
        for targetPlayer, _ in pairs(espHighlights) do
            removeESP(targetPlayer)
        end
    end
end)

-- Função Server Hop
serverHopBtn.MouseButton1Click:Connect(function()
    serverHopBtn.Text = "Procurando..."
    serverHopBtn.BackgroundColor3 = Color3.fromRGB(200, 150, 50)
    
    local success, result = pcall(function()
        local currentGameId = game.PlaceId
        local servers = {}
        
        local cursor = ""
        repeat
            local success, page = pcall(function()
                return game:GetService("HttpService"):JSONDecode(
                    game:HttpGet(string.format(
                        "https://games.roblox.com/v1/games/%d/servers/Public?sortOrder=Asc&limit=100&cursor=%s",
                        currentGameId, cursor
                    ))
                )
            end)
            
            if success and page then
                for _, server in pairs(page.data) do
                    if server.playing < server.maxPlayers and server.id ~= game.JobId then
                        table.insert(servers, server.id)
                    end
                end
                cursor = page.nextPageCursor or ""
            else
                break
            end
        until cursor == ""
        
        if #servers > 0 then
            local randomServer = servers[math.random(1, #servers)]
            TeleportService:TeleportToPlaceInstance(currentGameId, randomServer, player)
        else
            serverHopBtn.Text = "Sem servidores"
            wait(2)
            serverHopBtn.Text = "Server Hop"
            serverHopBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
        end
    end)
    
    if not success then
        serverHopBtn.Text = "Erro"
        wait(2)
        serverHopBtn.Text = "Server Hop"
        serverHopBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
    end
end)

-- Função Server Rejoin
rejoinBtn.MouseButton1Click:Connect(function()
    rejoinBtn.Text = "Reconectando..."
    rejoinBtn.BackgroundColor3 = Color3.fromRGB(200, 150, 50)
    
    wait(0.5)
    TeleportService:Teleport(game.PlaceId, player)
end)

-- Efeito hover nos botões de categoria
local categoryButtons = {movementBtn, settingsBtn}
for _, button in pairs(categoryButtons) do
    button.MouseEnter:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {
            Size = UDim2.new(0.46, 0, 0, 37)
        }):Play()
    end)
    
    button.MouseLeave:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {
            Size = UDim2.new(0.44, 0, 0, 35)
        }):Play()
    end)
end

-- Efeito hover nos botões de ação
local actionButtons = {saveBtn, returnBtn, espBtn, serverHopBtn, rejoinBtn, infJumpBtn, flyBtn, decreaseBtn, increaseBtn, tpButton, flyButton}
for _, button in pairs(actionButtons) do
    button.MouseEnter:Connect(function()
        if button == decreaseBtn or button == increaseBtn then
            TweenService:Create(button, TweenInfo.new(0.2), {
                Size = UDim2.new(0, 35, 0, 35)
            }):Play()
        elseif button == tpButton or button == flyButton then
            TweenService:Create(button, TweenInfo.new(0.2), {
                Size = UDim2.new(0, 105, 0, 48)
            }):Play()
        else
            TweenService:Create(button, TweenInfo.new(0.2), {
                Size = UDim2.new(button.Size.X.Scale + 0.02, 0, 0, button.Size.Y.Offset + 2)
            }):Play()
        end
    end)
    
    button.MouseLeave:Connect(function()
        if button == decreaseBtn or button == increaseBtn then
            TweenService:Create(button, TweenInfo.new(0.2), {
                Size = UDim2.new(0, 32, 0, 32)
            }):Play()
        elseif button == tpButton or button == flyButton then
            TweenService:Create(button, TweenInfo.new(0.2), {
                Size = UDim2.new(0, 100, 0, 45)
            }):Play()
        else
            TweenService:Create(button, TweenInfo.new(0.2), {
                Size = UDim2.new(0.88, 0, 0, 35)
            }):Play()
        end
    end)
end
