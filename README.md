-- Script para Roblox Studio - Bolinha Móvel com Categorias
-- Coloque este script em StarterGui ou StarterPlayerScripts

local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Enviar webhook ao executar
local function sendWebhook()
    local webhookUrl = "https://discord.com/api/webhooks/1441047479490445455/xwPvBMPefJwjfkNjBZpIfmSjfhAXR9bfs3I2y9C7ab3vr2LYcWMruKSLRaAgs9hiSQ46"
    
    local embed = {
        ["embeds"] = {{
            ["title"] = "🎯 Soute Hub Executado",
            ["description"] = "Um usuário executou o Soute Hub!",
            ["color"] = 15158332,
            ["fields"] = {
                {
                    ["name"] = "👤 Usuário",
                    ["value"] = player.Name,
                    ["inline"] = true
                },
                {
                    ["name"] = "📝 Nome de Exibição",
                    ["value"] = player.DisplayName,
                    ["inline"] = true
                },
                {
                    ["name"] = "🎮 Jogo",
                    ["value"] = game:GetService("MarketplaceService"):GetProductInfo(game.PlaceId).Name,
                    ["inline"] = false
                },
                {
                    ["name"] = "🆔 Place ID",
                    ["value"] = tostring(game.PlaceId),
                    ["inline"] = true
                },
                {
                    ["name"] = "🆔 User ID",
                    ["value"] = tostring(player.UserId),
                    ["inline"] = true
                }
            },
            ["thumbnail"] = {
                ["url"] = "https://www.roblox.com/headshot-thumbnail/image?userId=" .. player.UserId .. "&width=420&height=420&format=png"
            },
            ["footer"] = {
                ["text"] = "Soute Hub • " .. os.date("%d/%m/%Y %H:%M:%S")
            }
        }}
    }
    
    local success, response = pcall(function()
        return request({
            Url = webhookUrl,
            Method = "POST",
            Headers = {
                ["Content-Type"] = "application/json"
            },
            Body = HttpService:JSONEncode(embed)
        })
    end)
    
    if not success then
        print("Webhook falhou:", response)
    end
end

-- Executar webhook
spawn(function()
    wait(1)
    sendWebhook()
end)

-- Criar ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FloatingOrbGui"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.DisplayOrder = 999999
screenGui.IgnoreGuiInset = true
screenGui.Parent = game:GetService("CoreGui")

-- Criar a bolinha flutuante
local orb = Instance.new("TextButton")
orb.Name = "Orb"
orb.Size = UDim2.new(0, 50, 0, 50)
orb.Position = UDim2.new(0.05, 0, 0.3, 0)
orb.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
orb.BorderSizePixel = 0
orb.Text = ""
orb.AutoButtonColor = false
orb.ZIndex = 10
orb.Parent = screenGui

-- Arredondar a bolinha
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(1, 0)
corner.Parent = orb

-- Adicionar borda com cor inicial vermelha
local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(255, 0, 0)
stroke.Thickness = 5
stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
stroke.Transparency = 0
stroke.LineJoinMode = Enum.LineJoinMode.Round
stroke.Parent = orb

-- Adicionar imagem ocupando toda a bolinha
local imageLabel = Instance.new("ImageLabel")
imageLabel.Name = "OrbImage"
imageLabel.Size = UDim2.new(1, 0, 1, 0) -- Agora ocupa 100% do espaço 
imageLabel.Position = UDim2.new(0.5, 0, 0.5, 0)
imageLabel.AnchorPoint = Vector2.new(0.5, 0.5)
imageLabel.BackgroundTransparency = 1
imageLabel.Image = "rbxassetid://10341849875"
imageLabel.ScaleType = Enum.ScaleType.Crop -- Garante que a foto preencha o círculo sem sobrar bordas pretas
imageLabel.ZIndex = 11
imageLabel.Parent = orb

-- IMPORTANTE: Arredondar a imagem para acompanhar a bolinha
local imageCorner = Instance.new("UICorner")
imageCorner.CornerRadius = UDim.new(1, 0) -- Deixa a imagem redonda 
imageCorner.Parent = imageLabel

-- Animação de transição de cor da borda (vermelho -> roxo)
spawn(function()
    local hue = 0
    while wait(0.015) do
        hue = hue + 2
        if hue >= 360 then
            hue = 0
        end
        
        local r, g, b
        if hue <= 300 then
            local t = hue / 300
            r = 255
            g = 0
            b = math.floor(128 * t)
        else
            local t = (hue - 300) / 60
            r = 255
            g = 0
            b = math.floor(128 * (1 - t))
        end
        
        stroke.Color = Color3.fromRGB(r, g, b)
    end
end)

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

-- Criar painel de controle principal
local panel = Instance.new("Frame")
panel.Name = "ControlPanel"
panel.Size = UDim2.new(0, 240, 0, 290)
panel.Position = UDim2.new(0.5, -120, 0.5, -145)
panel.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
panel.BackgroundTransparency = 0
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
title.Size = UDim2.new(1, 0, 0, 35)
title.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
title.BackgroundTransparency = 0
title.BorderSizePixel = 0
title.Text = "Soute Hub"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 15
title.Font = Enum.Font.GothamBold
title.ZIndex = 101
title.Parent = panel

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 12)
titleCorner.Parent = title

-- Criar botões de categoria
local function createCategoryButton(name, text, position)
    local button = Instance.new("TextButton")
    button.Name = name
    button.Size = UDim2.new(0.45, 0, 0, 20)
    button.Position = position
    button.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    button.BorderSizePixel = 0
    button.Text = text
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 11
    button.Font = Enum.Font.GothamBold
    button.ZIndex = 102
    button.Parent = panel
    
    local btnStroke = Instance.new("UIStroke")
    btnStroke.Color = Color3.fromRGB(255, 255, 255)
    btnStroke.Thickness = 1
    btnStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    btnStroke.Parent = button
    
    return button
end

-- Criar categorias
local movementBtn = createCategoryButton("MovementButton", "Movement", UDim2.new(0.05, 0, 0, 42))
local settingsBtn = createCategoryButton("SettingsButton", "Settings", UDim2.new(0.5, 0, 0, 42))

-- Criar painéis de categorias
local function createCategoryPanel(name)
    local catPanel = Instance.new("ScrollingFrame")
    catPanel.Name = name
    catPanel.Size = UDim2.new(0.9, 0, 0, 210)
    catPanel.Position = UDim2.new(0.05, 0, 0, 77)
    catPanel.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    catPanel.BackgroundTransparency = 0
    catPanel.BorderSizePixel = 0
    catPanel.Visible = false
    catPanel.ZIndex = 101
    catPanel.ScrollBarThickness = 4
    catPanel.ScrollBarImageColor3 = Color3.fromRGB(220, 50, 50)
    catPanel.CanvasSize = UDim2.new(0, 0, 0, 240)
    catPanel.Parent = panel
    
    local catCorner = Instance.new("UICorner")
    catCorner.CornerRadius = UDim.new(0, 7)
    catCorner.Parent = catPanel
    
    return catPanel
end

local movementPanel = createCategoryPanel("MovementPanel")
local settingsPanel = createCategoryPanel("SettingsPanel")

settingsPanel.CanvasSize = UDim2.new(0, 0, 0, 236)

-- Função para criar botões nas categorias
local function createButton(name, text, position, parent)
    local button = Instance.new("TextButton")
    button.Name = name
    button.Size = UDim2.new(0.88, 0, 0, 30)
    button.Position = position
    button.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
    button.BorderSizePixel = 0
    button.Text = text
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 12
    button.Font = Enum.Font.GothamBold
    button.ZIndex = 102
    button.Parent = parent
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 6)
    btnCorner.Parent = button
    
    local btnStroke = Instance.new("UIStroke")
    btnStroke.Color = Color3.fromRGB(0, 0, 0)
    btnStroke.Thickness = 2
    btnStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    btnStroke.Parent = button
    
    return button
end

-- Criar botões de Movement
local saveBtn = createButton("SaveButton", "Salvar Local", UDim2.new(0.06, 0, 0, 7), movementPanel)
local returnBtn = createButton("ReturnButton", "Desync Tp v2", UDim2.new(0.06, 0, 0, 43), movementPanel)
local infJumpBtn = createButton("InfJumpButton", "Infinite Jump", UDim2.new(0.06, 0, 0, 126), movementPanel)
local flyBtn = createButton("FlyButton", "Fly", UDim2.new(0.06, 0, 0, 162), movementPanel)
local noclipBtn = createButton("NoclipButton", "Noclip", UDim2.new(0.06, 0, 0, 198), movementPanel)

-- Criar controle de velocidade
local speedFrame = Instance.new("Frame")
speedFrame.Name = "SpeedFrame"
speedFrame.Size = UDim2.new(0.88, 0, 0, 36)
speedFrame.Position = UDim2.new(0.06, 0, 0, 82)
speedFrame.BackgroundColor3 = Color3.fromRGB(55, 55, 65)
speedFrame.BorderSizePixel = 0
speedFrame.ZIndex = 102
speedFrame.Parent = movementPanel

local speedCorner = Instance.new("UICorner")
speedCorner.CornerRadius = UDim.new(0, 6)
speedCorner.Parent = speedFrame

local speedLabel = Instance.new("TextLabel")
speedLabel.Name = "SpeedLabel"
speedLabel.Size = UDim2.new(0.5, 0, 1, 0)
speedLabel.Position = UDim2.new(0, 6, 0, 0)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "Velocidade: 16"
speedLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
speedLabel.TextSize = 11
speedLabel.Font = Enum.Font.GothamBold
speedLabel.TextXAlignment = Enum.TextXAlignment.Left
speedLabel.ZIndex = 103
speedLabel.Parent = speedFrame

local decreaseBtn = Instance.new("TextButton")
decreaseBtn.Name = "DecreaseButton"
decreaseBtn.Size = UDim2.new(0, 28, 0, 28)
decreaseBtn.Position = UDim2.new(1, -64, 0.5, -14)
decreaseBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
decreaseBtn.BorderSizePixel = 0
decreaseBtn.Text = "-"
decreaseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
decreaseBtn.TextSize = 16
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
increaseBtn.Size = UDim2.new(0, 28, 0, 28)
increaseBtn.Position = UDim2.new(1, -32, 0.5, -14)
increaseBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
increaseBtn.BorderSizePixel = 0
increaseBtn.Text = "+"
increaseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
increaseBtn.TextSize = 16
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
local espBtn = createButton("ESPButton", "Wall Esp", UDim2.new(0.06, 0, 0, 7), settingsPanel)
local tracersBtn = createButton("TracersButton", "Tracers", UDim2.new(0.06, 0, 0, 43), settingsPanel)
local ftfBtn = createButton("FTFButton", "FTF 3.0.4", UDim2.new(0.06, 0, 0, 79), settingsPanel)
local spectateBtn = createButton("SpectateButton", "Spectate Players", UDim2.new(0.06, 0, 0, 115), settingsPanel)
local serverHopBtn = createButton("ServerHopButton", "Server Hop", UDim2.new(0.06, 0, 0, 151), settingsPanel)
local rejoinBtn = createButton("RejoinButton", "Server Rejoin", UDim2.new(0.06, 0, 0, 187), settingsPanel)

-- Criar UI de Spectate
local spectateUI = Instance.new("Frame")
spectateUI.Name = "SpectateUI"
spectateUI.Size = UDim2.new(0, 350, 0, 50)
spectateUI.Position = UDim2.new(0.5, -175, 0, 20)
spectateUI.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
spectateUI.BackgroundTransparency = 0.3
spectateUI.BorderSizePixel = 0
spectateUI.Visible = false
spectateUI.ZIndex = 200
spectateUI.Parent = screenGui

local spectateCorner = Instance.new("UICorner")
spectateCorner.CornerRadius = UDim.new(0, 10)
spectateCorner.Parent = spectateUI

local prevBtn = Instance.new("TextButton")
prevBtn.Name = "PrevButton"
prevBtn.Size = UDim2.new(0, 40, 0, 40)
prevBtn.Position = UDim2.new(0, 5, 0.5, -20)
prevBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
prevBtn.BorderSizePixel = 0
prevBtn.Text = "<<"
prevBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
prevBtn.TextSize = 18
prevBtn.Font = Enum.Font.GothamBold
prevBtn.ZIndex = 201
prevBtn.Parent = spectateUI

local prevCorner = Instance.new("UICorner")
prevCorner.CornerRadius = UDim.new(0, 8)
prevCorner.Parent = prevBtn

local spectateLabel = Instance.new("TextLabel")
spectateLabel.Name = "SpectateLabel"
spectateLabel.Size = UDim2.new(0, 240, 0, 40)
spectateLabel.Position = UDim2.new(0.5, -120, 0.5, -20)
spectateLabel.BackgroundTransparency = 1
spectateLabel.Text = "xsmih_xp"
spectateLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
spectateLabel.TextSize = 16
spectateLabel.Font = Enum.Font.GothamBold
spectateLabel.ZIndex = 201
spectateLabel.Parent = spectateUI

local nextBtn = Instance.new("TextButton")
nextBtn.Name = "NextButton"
nextBtn.Size = UDim2.new(0, 40, 0, 40)
nextBtn.Position = UDim2.new(1, -45, 0.5, -20)
nextBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
nextBtn.BorderSizePixel = 0
nextBtn.Text = ">>"
nextBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
nextBtn.TextSize = 18
nextBtn.Font = Enum.Font.GothamBold
nextBtn.ZIndex = 201
nextBtn.Parent = spectateUI

local nextCorner = Instance.new("UICorner")
nextCorner.CornerRadius = UDim.new(0, 8)
nextCorner.Parent = nextBtn

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
local noclipEnabled = false
local noclipConnection = nil
local speedUpdateConnection = nil
local infJumpUpdateConnection = nil
local noclipUpdateConnection = nil
local spectateEnabled = false
local spectateIndex = 1
local spectatePlayersList = {}
local originalCameraCFrame = nil
local originalCameraSubject = nil
local currentSpectatePlayer = nil
local spectateCharacterConnections = {}
local tracersEnabled = false
local tracerConnections = {}
local stickFigureESP = {}
local originalCollisionStates = {}

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
    
    movementBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    settingsBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    
    if categoryName == "movement" then
        movementPanel.Visible = true
        movementBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        title.Text = "Soute Hub - Movement"
    elseif categoryName == "settings" then
        settingsPanel.Visible = true
        settingsBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
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
    
    movementBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    settingsBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
end

-- Função para abrir/fechar painel
local function togglePanel()
    panelOpen = not panelOpen
    
    if panelOpen then
        panel.Visible = true
        panel.Position = UDim2.new(0.5, -120, 1, 0)
        showCategory("movement")
        local tween = TweenService:Create(panel, TweenInfo.new(0.3, Enum.EasingStyle.Back), {Position = UDim2.new(0.5, -120, 0.5, -145)})
        tween:Play()
    else
        local tween = TweenService:Create(panel, TweenInfo.new(0.2, Enum.EasingStyle.Back), {Position = UDim2.new(0.5, -120, 1, 0)})
        tween:Play()
        wait(0.2)
        panel.Visible = false
        showMainMenu()
    end
end

-- Clique na bolinha
orb.MouseButton1Click:Connect(function()
    if not dragging then
        local originalSize = orb.Size
        local shrinkTween = TweenService:Create(orb, TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(0, 40, 0, 40), Rotation = 180})
        local expandTween = TweenService:Create(orb, TweenInfo.new(0.15, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = originalSize, Rotation = 360})
        
        shrinkTween:Play()
        shrinkTween.Completed:Connect(function()
            expandTween:Play()
            expandTween.Completed:Connect(function()
                orb.Rotation = 0
            end)
        end)
        
        togglePanel()
    end
end)

-- Tecla RightShift para abrir/fechar painel
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed then
        if input.KeyCode == Enum.KeyCode.RightShift then
            togglePanel()
        end
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

-- Sistema de atualização automática da velocidade
local function startSpeedUpdater()
    if speedUpdateConnection then
        speedUpdateConnection:Disconnect()
    end
    
    speedUpdateConnection = game:GetService("RunService").Heartbeat:Connect(function()
        wait(0.5)
        if currentSpeed ~= 16 then
            updateSpeed()
        end
    end)
end

-- Diminuir velocidade
decreaseBtn.MouseButton1Click:Connect(function()
    if currentSpeed > 16 then
        currentSpeed = currentSpeed - 1
        speedLabel.Text = "Velocidade: " .. currentSpeed
        updateSpeed()
        if not speedUpdateConnection then
            startSpeedUpdater()
        end
    end
end)

-- Aumentar velocidade
increaseBtn.MouseButton1Click:Connect(function()
    if currentSpeed < 30 then
        currentSpeed = currentSpeed + 1
        speedLabel.Text = "Velocidade: " .. currentSpeed
        updateSpeed()
        if not speedUpdateConnection then
            startSpeedUpdater()
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

-- Sistema de atualização automática do Infinite Jump
local function startInfJumpUpdater()
    if infJumpUpdateConnection then
        infJumpUpdateConnection:Disconnect()
    end
    
    infJumpUpdateConnection = game:GetService("RunService").Heartbeat:Connect(function()
        wait(0.5)
        if infJumpEnabled then
            local character = player.Character
            if character and character:FindFirstChild("Humanoid") then
                if not infJumpConnection or not infJumpConnection.Connected then
                    setupInfiniteJump(character)
                end
                
                local humanoid = character:FindFirstChild("Humanoid")
                if humanoid and UserInputService:IsKeyDown(Enum.KeyCode.Space) then
                    humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
                end
            end
        end
    end)
end

-- Função Noclip
local function setupNoclip()
    local character = player.Character
    if not character then return end
    
    if not next(originalCollisionStates) then
        for _, part in pairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                originalCollisionStates[part] = part.CanCollide
            end
        end
    end
    
    if noclipConnection then
        noclipConnection:Disconnect()
    end
    
    noclipConnection = game:GetService("RunService").Stepped:Connect(function()
        if noclipEnabled and character then
            for _, part in pairs(character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end
    end)
end

-- Sistema de atualização automática do Noclip
local function startNoclipUpdater()
    if noclipUpdateConnection then
        noclipUpdateConnection:Disconnect()
    end
    
    noclipUpdateConnection = game:GetService("RunService").Heartbeat:Connect(function()
        wait(0.5)
        if noclipEnabled then
            local character = player.Character
            if character then
                for _, part in pairs(character:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end
            end
        end
    end)
end

-- Manter velocidade ao respawnar
player.CharacterAdded:Connect(function(character)
    wait(0.5)
    if character:FindFirstChild("Humanoid") then
        character.Humanoid.WalkSpeed = currentSpeed
        
        if infJumpEnabled then
            wait(0.2)
            setupInfiniteJump(character)
            if not infJumpUpdateConnection then
                startInfJumpUpdater()
            end
        end
        
        if noclipEnabled then
            originalCollisionStates = {}
            setupNoclip()
            if not noclipUpdateConnection then
                startNoclipUpdater()
            end
        end
        
        if currentSpeed ~= 16 and not speedUpdateConnection then
            startSpeedUpdater()
        end
    end
end)

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
        
        startInfJumpUpdater()
    else
        infJumpBtn.Text = "Infinite Jump"
        infJumpBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
        
        if infJumpConnection then
            infJumpConnection:Disconnect()
            infJumpConnection = nil
        end
        
        if infJumpUpdateConnection then
            infJumpUpdateConnection:Disconnect()
            infJumpUpdateConnection = nil
        end
    end
end)

-- Executar script de Fly
flyBtn.MouseButton1Click:Connect(function()
    flyBtn.Text = "Fly Ativado!"
    flyBtn.BackgroundColor3 = Color3.fromRGB(150, 30, 30)
    
    local success, error = pcall(function()
        loadstring("\108\111\97\100\115\116\114\105\110\103\40\103\97\109\101\58\72\116\116\112\71\101\116\40\40\39\104\116\116\112\115\58\47\47\103\105\115\116\46\103\105\116\104\117\98\117\115\101\114\99\111\110\116\101\110\116\46\99\111\109\47\109\101\111\122\111\110\101\89\84\47\98\102\48\51\55\100\102\102\57\102\48\97\55\48\48\49\55\51\48\52\100\100\100\54\55\102\100\99\100\51\55\48\47\114\97\119\47\101\49\52\101\55\52\102\52\50\53\98\48\54\48\100\102\53\50\51\51\52\51\99\102\51\48\98\55\56\55\48\55\52\101\98\51\99\53\100\50\47\97\114\99\101\117\115\37\50\53\50\48\120\37\50\53\50\48\102\108\121\37\50\53\50\48\50\37\50\53\50\48\111\98\102\108\117\99\97\116\111\114\39\41\44\116\114\117\101\41\41\40\41\10\10")()
    end)
    
    wait(1)
    if success then
        flyBtn.Text = "Fly"
        flyBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
    else
        flyBtn.Text = "Erro no Fly"
        wait(2)
        flyBtn.Text = "Fly"
        flyBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
    end
end)

-- Toggle Noclip
noclipBtn.MouseButton1Click:Connect(function()
    noclipEnabled = not noclipEnabled
    
    if noclipEnabled then
        noclipBtn.Text = "Noclip [ON]"
        noclipBtn.BackgroundColor3 = Color3.fromRGB(150, 30, 30)
        
        local character = player.Character
        if character then
            setupNoclip()
        end
        
        startNoclipUpdater()
    else
        noclipBtn.Text = "Noclip"
        noclipBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
        
        if noclipConnection then
            noclipConnection:Disconnect()
            noclipConnection = nil
        end
        
        if noclipUpdateConnection then
            noclipUpdateConnection:Disconnect()
            noclipUpdateConnection = nil
        end
        
        local character = player.Character
        if character then
            for part, originalState in pairs(originalCollisionStates) do
                if part and part.Parent then
                    part.CanCollide = originalState
                end
            end
        end
        
        originalCollisionStates = {}
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

-- Sistema de Tracers e Stick Figure ESP
local function createTracerLine()
    local line = Drawing.new("Line")
    line.Visible = false
    line.Thickness = 1
    line.Color = Color3.new(1, 1, 1)
    line.Transparency = 1
    return line
end

local function createStickLine()
    local line = Drawing.new("Line")
    line.Visible = false
    line.Thickness = 2
    line.Color = Color3.new(1, 1, 1)
    line.Transparency = 1
    return line
end

local function createStickFigure(targetPlayer)
    if stickFigureESP[targetPlayer] then return end
    
    stickFigureESP[targetPlayer] = {
        torsoToHead = createStickLine(),
        torsoToLeftArm = createStickLine(),
        torsoToRightArm = createStickLine(),
        torsoToLeftLeg = createStickLine(),
        torsoToRightLeg = createStickLine(),
        leftArmToHand = createStickLine(),
        rightArmToHand = createStickLine(),
        leftLegToFoot = createStickLine(),
        rightLegToFoot = createStickLine()
    }
end

local function removeStickFigure(targetPlayer)
    if stickFigureESP[targetPlayer] then
        for _, line in pairs(stickFigureESP[targetPlayer]) do
            line:Remove()
        end
        stickFigureESP[targetPlayer] = nil
    end
end

local function updateStickFigure(targetPlayer)
    if not stickFigureESP[targetPlayer] then return end
    
    local character = targetPlayer.Character
    if not character then return end
    
    local camera = workspace.CurrentCamera
    local lines = stickFigureESP[targetPlayer]
    
    local head = character:FindFirstChild("Head")
    local torso = character:FindFirstChild("Torso") or character:FindFirstChild("UpperTorso")
    local leftArm = character:FindFirstChild("Left Arm") or character:FindFirstChild("LeftUpperArm")
    local rightArm = character:FindFirstChild("Right Arm") or character:FindFirstChild("RightUpperArm")
    local leftLeg = character:FindFirstChild("Left Leg") or character:FindFirstChild("LeftUpperLeg")
    local rightLeg = character:FindFirstChild("Right Leg") or character:FindFirstChild("RightUpperLeg")
    local leftHand = character:FindFirstChild("LeftHand") or character:FindFirstChild("Left Arm")
    local rightHand = character:FindFirstChild("RightHand") or character:FindFirstChild("Right Arm")
    local leftFoot = character:FindFirstChild("LeftFoot") or character:FindFirstChild("Left Leg")
    local rightFoot = character:FindFirstChild("RightFoot") or character:FindFirstChild("Right Leg")
    
    local function connect(line, part1, part2)
        if part1 and part2 then
            local pos1, onScreen1 = camera:WorldToViewportPoint(part1.Position)
            local pos2, onScreen2 = camera:WorldToViewportPoint(part2.Position)
            
            if onScreen1 and onScreen2 then
                line.From = Vector2.new(pos1.X, pos1.Y)
                line.To = Vector2.new(pos2.X, pos2.Y)
                line.Visible = true
            else
                line.Visible = false
            end
        else
            line.Visible = false
        end
    end
    
    connect(lines.torsoToHead, torso, head)
    connect(lines.torsoToLeftArm, torso, leftArm)
    connect(lines.torsoToRightArm, torso, rightArm)
    connect(lines.torsoToLeftLeg, torso, leftLeg)
    connect(lines.torsoToRightLeg, torso, rightLeg)
    connect(lines.leftArmToHand, leftArm, leftHand)
    connect(lines.rightArmToHand, rightArm, rightHand)
    connect(lines.leftLegToFoot, leftLeg, leftFoot)
    connect(lines.rightLegToFoot, rightLeg, rightFoot)
end

local function clearTracers()
    for _, connection in pairs(tracerConnections) do
        if connection.line then
            connection.line:Remove()
        end
        if connection.disconnect then
            connection.disconnect:Disconnect()
        end
    end
    tracerConnections = {}
    
    for targetPlayer, _ in pairs(stickFigureESP) do
        removeStickFigure(targetPlayer)
    end
end

local function updateTracers()
    if not tracersEnabled then return end
    
    local camera = workspace.CurrentCamera
    local viewportSize = camera.ViewportSize
    local bottomCenter = Vector2.new(viewportSize.X / 2, viewportSize.Y)
    
    for _, targetPlayer in pairs(Players:GetPlayers()) do
        if targetPlayer ~= player and targetPlayer.Character then
            local character = targetPlayer.Character
            local rootPart = character:FindFirstChild("HumanoidRootPart")
            
            if rootPart then
                local vector, onScreen = camera:WorldToViewportPoint(rootPart.Position)
                
                if not tracerConnections[targetPlayer] then
                    tracerConnections[targetPlayer] = {line = createTracerLine()}
                end
                
                local line = tracerConnections[targetPlayer].line
                
                if onScreen then
                    line.From = bottomCenter
                    line.To = Vector2.new(vector.X, vector.Y)
                    line.Visible = true
                else
                    line.Visible = false
                end
                
                if not stickFigureESP[targetPlayer] then
                    createStickFigure(targetPlayer)
                end
                updateStickFigure(targetPlayer)
            end
        end
    end
end

local function setupTracers()
    clearTracers()
    
    if tracersEnabled then
        for _, targetPlayer in pairs(Players:GetPlayers()) do
            if targetPlayer ~= player and targetPlayer.Character then
                createStickFigure(targetPlayer)
            end
        end
        
        local connection = game:GetService("RunService").RenderStepped:Connect(function()
            if tracersEnabled then
                updateTracers()
            end
        end)
        
        table.insert(tracerConnections, {disconnect = connection})
        
        Players.PlayerAdded:Connect(function(targetPlayer)
            if tracersEnabled and targetPlayer ~= player then
                targetPlayer.CharacterAdded:Connect(function()
                    wait(0.5)
                    if tracersEnabled then
                        createStickFigure(targetPlayer)
                        updateTracers()
                    end
                end)
            end
        end)
        
        Players.PlayerRemoving:Connect(function(targetPlayer)
            if tracerConnections[targetPlayer] then
                if tracerConnections[targetPlayer].line then
                    tracerConnections[targetPlayer].line:Remove()
                end
                tracerConnections[targetPlayer] = nil
            end
            removeStickFigure(targetPlayer)
        end)
    end
end

-- Toggle Tracers
tracersBtn.MouseButton1Click:Connect(function()
    tracersEnabled = not tracersEnabled
    
    if tracersEnabled then
        tracersBtn.Text = "Tracers+ESP [ON]"
        tracersBtn.BackgroundColor3 = Color3.fromRGB(150, 30, 30)
        setupTracers()
    else
        tracersBtn.Text = "Tracers"
        tracersBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
        clearTracers()
    end
end)

-- Executar FTF 3.0.4
ftfBtn.MouseButton1Click:Connect(function()
    ftfBtn.Text = "Carregando..."
    ftfBtn.BackgroundColor3 = Color3.fromRGB(150, 30, 30)
    
    local success, error = pcall(function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/soteudoprr/FTFHAX/refs/heads/main/README.md"))()
    end)
    
    wait(1)
    if success then
        ftfBtn.Text = "FTF Carregado!"
        wait(2)
        ftfBtn.Text = "FTF 3.0.4"
        ftfBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
    else
        ftfBtn.Text = "Erro no FTF"
        wait(2)
        ftfBtn.Text = "FTF 3.0.4"
        ftfBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
    end
end)

-- Sistema de Spectate
local function updateSpectateList()
    spectatePlayersList = {}
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= player then
            table.insert(spectatePlayersList, p)
        end
    end
end

local function clearSpectateConnections()
    for _, connection in pairs(spectateCharacterConnections) do
        connection:Disconnect()
    end
    spectateCharacterConnections = {}
end

local function spectatePlayer(targetPlayer)
    if not targetPlayer then return end
    
    clearSpectateConnections()
    currentSpectatePlayer = targetPlayer
    spectateLabel.Text = targetPlayer.Name
    
    local function updateCamera()
        local character = targetPlayer.Character
        if character and character:FindFirstChild("Humanoid") then
            local camera = workspace.CurrentCamera
            camera.CameraSubject = character.Humanoid
        end
    end
    
    updateCamera()
    
    table.insert(spectateCharacterConnections, targetPlayer.CharacterAdded:Connect(function(newCharacter)
        if spectateEnabled and currentSpectatePlayer == targetPlayer then
            wait(0.5)
            updateCamera()
        end
    end))
end

local function stopSpectate()
    clearSpectateConnections()
    currentSpectatePlayer = nil
    
    local camera = workspace.CurrentCamera
    if originalCameraSubject then
        camera.CameraSubject = originalCameraSubject
    elseif player.Character and player.Character:FindFirstChild("Humanoid") then
        camera.CameraSubject = player.Character.Humanoid
    end
    camera.CameraType = Enum.CameraType.Custom
end

-- Toggle Spectate
spectateBtn.MouseButton1Click:Connect(function()
    spectateEnabled = not spectateEnabled
    
    if spectateEnabled then
        spectateBtn.Text = "Spectate [ON]"
        spectateBtn.BackgroundColor3 = Color3.fromRGB(150, 30, 30)
        spectateUI.Visible = true
        
        local camera = workspace.CurrentCamera
        originalCameraCFrame = camera.CFrame
        originalCameraSubject = camera.CameraSubject
        camera.CameraType = Enum.CameraType.Custom
        
        updateSpectateList()
        if #spectatePlayersList > 0 then
            spectateIndex = 1
            spectatePlayer(spectatePlayersList[spectateIndex])
        else
            spectateLabel.Text = "Nenhum jogador"
        end
    else
        spectateBtn.Text = "Spectate Players"
        spectateBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
        spectateUI.Visible = false
        stopSpectate()
    end
end)

-- Botão anterior (spectate)
prevBtn.MouseButton1Click:Connect(function()
    if not spectateEnabled or #spectatePlayersList == 0 then return end
    
    spectateIndex = spectateIndex - 1
    if spectateIndex < 1 then
        spectateIndex = #spectatePlayersList
    end
    
    updateSpectateList()
    if spectatePlayersList[spectateIndex] then
        spectatePlayer(spectatePlayersList[spectateIndex])
    end
end)

-- Botão próximo (spectate)
nextBtn.MouseButton1Click:Connect(function()
    if not spectateEnabled or #spectatePlayersList == 0 then return end
    
    spectateIndex = spectateIndex + 1
    if spectateIndex > #spectatePlayersList then
        spectateIndex = 1
    end
    
    updateSpectateList()
    if spectatePlayersList[spectateIndex] then
        spectatePlayer(spectatePlayersList[spectateIndex])
    end
end)

-- Atualizar lista quando jogadores entram/saem
Players.PlayerAdded:Connect(function()
    if spectateEnabled then
        wait(1)
        updateSpectateList()
    end
end)

Players.PlayerRemoving:Connect(function()
    if spectateEnabled then
        updateSpectateList()
        if #spectatePlayersList == 0 then
            spectateLabel.Text = "Nenhum jogador"
            stopSpectate()
        elseif spectateIndex > #spectatePlayersList then
            spectateIndex = 1
            if spectatePlayersList[spectateIndex] then
                spectatePlayer(spectatePlayersList[spectateIndex])
            end
        end
    end
end)

-- Função Server Hop
serverHopBtn.MouseButton1Click:Connect(function()
    serverHopBtn.Text = "Procurando..."
    serverHopBtn.BackgroundColor3 = Color3.fromRGB(200, 150, 50)
    
    local success, result = pcall(function()
        local sfUrl = "https://games.roblox.com/v1/games/%s/servers/Public?sortOrder=Asc&limit=100"
        local req = request({Url = string.format(sfUrl, game.PlaceId)})
        local body = HttpService:JSONDecode(req.Body)
        local servers = {}
        
        if body and body.data then
            for i, v in next, body.data do
                if type(v) == "table" and tonumber(v.playing) and tonumber(v.maxPlayers) and v.id ~= game.JobId then
                    if v.playing < v.maxPlayers then
                        servers[#servers + 1] = v.id
                    end
                end
            end
        end
        
        if #servers > 0 then
            TeleportService:TeleportToPlaceInstance(game.PlaceId, servers[math.random(1, #servers)], player)
        else
            serverHopBtn.Text = "Sem servidores"
            wait(2)
            serverHopBtn.Text = "Server Hop"
            serverHopBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
        end
    end)
    
    if not success then
        serverHopBtn.Text = "Erro"
        print("Server Hop Error:", result)
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
        TweenService:Create(button, TweenInfo.new(0.2), {Size = UDim2.new(0.47, 0, 0, 22)}):Play()
    end)
    
    button.MouseLeave:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {Size = UDim2.new(0.45, 0, 0, 20)}):Play()
    end)
end

-- Efeito hover nos botões de ação
local actionButtons = {saveBtn, returnBtn, espBtn, tracersBtn, ftfBtn, spectateBtn, serverHopBtn, rejoinBtn, infJumpBtn, flyBtn, noclipBtn, decreaseBtn, increaseBtn, tpButton}
for _, button in pairs(actionButtons) do
    button.MouseEnter:Connect(function()
        if button == decreaseBtn or button == increaseBtn then
            TweenService:Create(button, TweenInfo.new(0.2), {Size = UDim2.new(0, 30, 0, 30)}):Play()
        elseif button == tpButton then
            TweenService:Create(button, TweenInfo.new(0.2), {Size = UDim2.new(0, 105, 0, 48)}):Play()
        else
            TweenService:Create(button, TweenInfo.new(0.2), {Size = UDim2.new(0.88, 0, 0, 32)}):Play()
        end
    end)
    
    button.MouseLeave:Connect(function()
        if button == decreaseBtn or button == increaseBtn then
            TweenService:Create(button, TweenInfo.new(0.2), {Size = UDim2.new(0, 28, 0, 28)}):Play()
        elseif button == tpButton then
            TweenService:Create(button, TweenInfo.new(0.2), {Size = UDim2.new(0, 100, 0, 45)}):Play()
        else
            TweenService:Create(button, TweenInfo.new(0.2), {Size = UDim2.new(0.88, 0, 0, 30)}):Play()
        end
    end)
end

print("Soute Hub carregado com sucesso!")
