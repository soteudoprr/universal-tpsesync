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

-- Criar painel de controle principal
local panel = Instance.new("Frame")
panel.Name = "ControlPanel"
panel.Size = UDim2.new(0, 280, 0, 320)
panel.Position = UDim2.new(0.5, -140, 0.5, -160)
panel.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
panel.BackgroundTransparency = 0.3
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
title.Size = UDim2.new(1, 0, 0, 45)
title.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
title.BackgroundTransparency = 0.3
title.BorderSizePixel = 0
title.Text = "Soute Hub"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 18
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
    button.Size = UDim2.new(0.45, 0, 0, 60)
    button.Position = position
    button.BackgroundColor3 = Color3.fromRGB(60, 120, 220)
    button.BorderSizePixel = 0
    button.Text = text
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 15
    button.Font = Enum.Font.GothamBold
    button.ZIndex = 102
    button.Parent = panel
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 10)
    btnCorner.Parent = button
    
    return button
end

-- Criar categorias
local movementBtn = createCategoryButton("MovementButton", "Movement", UDim2.new(0.05, 0, 0, 60), "")
local settingsBtn = createCategoryButton("SettingsButton", "Settings", UDim2.new(0.5, 0, 0, 60), "")

-- Criar painéis de categorias
local function createCategoryPanel(name)
    local catPanel = Instance.new("Frame")
    catPanel.Name = name
    catPanel.Size = UDim2.new(0.95, 0, 0, 180)
    catPanel.Position = UDim2.new(0.025, 0, 0, 130)
    catPanel.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
    catPanel.BackgroundTransparency = 0.5
    catPanel.BorderSizePixel = 0
    catPanel.Visible = false
    catPanel.ZIndex = 101
    catPanel.Parent = panel
    
    local catCorner = Instance.new("UICorner")
    catCorner.CornerRadius = UDim.new(0, 10)
    catCorner.Parent = catPanel
    
    return catPanel
end

local movementPanel = createCategoryPanel("MovementPanel")
local settingsPanel = createCategoryPanel("SettingsPanel")

-- Função para criar botões nas categorias
local function createButton(name, text, position, parent)
    local button = Instance.new("TextButton")
    button.Name = name
    button.Size = UDim2.new(0.9, 0, 0, 40)
    button.Position = position
    button.BackgroundColor3 = Color3.fromRGB(70, 130, 230)
    button.BorderSizePixel = 0
    button.Text = text
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 14
    button.Font = Enum.Font.GothamBold
    button.ZIndex = 102
    button.Parent = parent
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 8)
    btnCorner.Parent = button
    
    return button
end

-- Criar botões de Movement
local saveBtn = createButton("SaveButton", "Salvar Local", UDim2.new(0.05, 0, 0, 10), movementPanel)
local returnBtn = createButton("ReturnButton", "Desync Tp v2", UDim2.new(0.05, 0, 0, 60), movementPanel)

-- Criar controle de velocidade
local speedFrame = Instance.new("Frame")
speedFrame.Name = "SpeedFrame"
speedFrame.Size = UDim2.new(0.9, 0, 0, 50)
speedFrame.Position = UDim2.new(0.05, 0, 0, 115)
speedFrame.BackgroundColor3 = Color3.fromRGB(55, 55, 65)
speedFrame.BorderSizePixel = 0
speedFrame.ZIndex = 102
speedFrame.Parent = movementPanel

local speedCorner = Instance.new("UICorner")
speedCorner.CornerRadius = UDim.new(0, 8)
speedCorner.Parent = speedFrame

local speedLabel = Instance.new("TextLabel")
speedLabel.Name = "SpeedLabel"
speedLabel.Size = UDim2.new(0.5, 0, 1, 0)
speedLabel.Position = UDim2.new(0, 10, 0, 0)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "Velocidade: 16"
speedLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
speedLabel.TextSize = 13
speedLabel.Font = Enum.Font.GothamBold
speedLabel.TextXAlignment = Enum.TextXAlignment.Left
speedLabel.ZIndex = 103
speedLabel.Parent = speedFrame

local decreaseBtn = Instance.new("TextButton")
decreaseBtn.Name = "DecreaseButton"
decreaseBtn.Size = UDim2.new(0, 35, 0, 35)
decreaseBtn.Position = UDim2.new(1, -85, 0.5, -17.5)
decreaseBtn.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
decreaseBtn.BorderSizePixel = 0
decreaseBtn.Text = "-"
decreaseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
decreaseBtn.TextSize = 20
decreaseBtn.Font = Enum.Font.GothamBold
decreaseBtn.ZIndex = 103
decreaseBtn.Parent = speedFrame

local decreaseCorner = Instance.new("UICorner")
decreaseCorner.CornerRadius = UDim.new(0, 6)
decreaseCorner.Parent = decreaseBtn

local increaseBtn = Instance.new("TextButton")
increaseBtn.Name = "IncreaseButton"
increaseBtn.Size = UDim2.new(0, 35, 0, 35)
increaseBtn.Position = UDim2.new(1, -45, 0.5, -17.5)
increaseBtn.BackgroundColor3 = Color3.fromRGB(50, 200, 100)
increaseBtn.BorderSizePixel = 0
increaseBtn.Text = "+"
increaseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
increaseBtn.TextSize = 20
increaseBtn.Font = Enum.Font.GothamBold
increaseBtn.ZIndex = 103
increaseBtn.Parent = speedFrame

local increaseCorner = Instance.new("UICorner")
increaseCorner.CornerRadius = UDim.new(0, 6)
increaseCorner.Parent = increaseBtn

-- Criar botões de Settings
local espBtn = createButton("ESPButton", "Wall Esp", UDim2.new(0.05, 0, 0, 10), settingsPanel)
local serverHopBtn = createButton("ServerHopButton", "Server Hop", UDim2.new(0.05, 0, 0, 60), settingsPanel)

-- Botão para voltar
local backBtn = Instance.new("TextButton")
backBtn.Name = "BackButton"
backBtn.Size = UDim2.new(0.3, 0, 0, 35)
backBtn.Position = UDim2.new(0.35, 0, 1, -45)
backBtn.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
backBtn.BorderSizePixel = 0
backBtn.Text = "Voltar"
backBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
backBtn.TextSize = 13
backBtn.Font = Enum.Font.GothamBold
backBtn.Visible = false
backBtn.ZIndex = 102
backBtn.Parent = panel

local backCorner = Instance.new("UICorner")
backCorner.CornerRadius = UDim.new(0, 8)
backCorner.Parent = backBtn

-- Variáveis de controle
local panelOpen = false
local currentCategory = nil
local savedPosition = nil
local espEnabled = false
local espHighlights = {}
local currentSpeed = 16

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
    backBtn.Visible = true
    
    movementBtn.BackgroundColor3 = Color3.fromRGB(60, 120, 220)
    settingsBtn.BackgroundColor3 = Color3.fromRGB(60, 120, 220)
    
    if categoryName == "movement" then
        movementPanel.Visible = true
        movementBtn.BackgroundColor3 = Color3.fromRGB(50, 200, 100)
        title.Text = "Soute Hub - Movement"
    elseif categoryName == "settings" then
        settingsPanel.Visible = true
        settingsBtn.BackgroundColor3 = Color3.fromRGB(50, 200, 100)
        title.Text = "Soute Hub - Settings"
    end
    
    currentCategory = categoryName
end

-- Função para voltar ao menu principal
local function showMainMenu()
    movementPanel.Visible = false
    settingsPanel.Visible = false
    backBtn.Visible = false
    currentCategory = nil
    title.Text = "Soute Hub"
    
    movementBtn.BackgroundColor3 = Color3.fromRGB(60, 120, 220)
    settingsBtn.BackgroundColor3 = Color3.fromRGB(60, 120, 220)
end

-- Função para abrir/fechar painel
local function togglePanel()
    panelOpen = not panelOpen
    
    if panelOpen then
        panel.Visible = true
        panel.Position = UDim2.new(0.5, -140, 1, 0)
        showCategory("movement") -- Abre direto em Movement
        local tween = TweenService:Create(panel, TweenInfo.new(0.3, Enum.EasingStyle.Back), {
            Position = UDim2.new(0.5, -140, 0.5, -160)
        })
        tween:Play()
    else
        local tween = TweenService:Create(panel, TweenInfo.new(0.2, Enum.EasingStyle.Back), {
            Position = UDim2.new(0.5, -140, 1, 0)
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

backBtn.MouseButton1Click:Connect(function()
    showMainMenu()
end)

-- Função salvar posição
saveBtn.MouseButton1Click:Connect(function()
    local character = player.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        savedPosition = character.HumanoidRootPart.CFrame
        saveBtn.Text = "✅ Local Salvo!"
        wait(1)
        saveBtn.Text = "💾 Salvar Local"
    end
end)

-- Função voltar à posição
returnBtn.MouseButton1Click:Connect(function()
    local character = player.Character
    if character and character:FindFirstChild("HumanoidRootPart") and savedPosition then
        character.HumanoidRootPart.CFrame = savedPosition
        returnBtn.Text = "✅ Teleportado!"
        wait(1)
        returnBtn.Text = "⚡ Desync Tp v2"
    else
        returnBtn.Text = "❌ Sem posição"
        wait(1)
        returnBtn.Text = "⚡ Desync Tp v2"
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
        speedLabel.Text = "🚀 Velocidade: " .. currentSpeed
        updateSpeed()
    end
end)

-- Aumentar velocidade
increaseBtn.MouseButton1Click:Connect(function()
    if currentSpeed < 30 then
        currentSpeed = currentSpeed + 1
        speedLabel.Text = "🚀 Velocidade: " .. currentSpeed
        updateSpeed()
    end
end)

-- Manter velocidade ao respawnar
player.CharacterAdded:Connect(function(character)
    wait(0.5)
    if character:FindFirstChild("Humanoid") then
        character.Humanoid.WalkSpeed = currentSpeed
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
        espBtn.Text = "👁️ Wall Esp [ON]"
        espBtn.BackgroundColor3 = Color3.fromRGB(50, 200, 100)
        
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
        espBtn.Text = "👁️ Wall Esp"
        espBtn.BackgroundColor3 = Color3.fromRGB(70, 130, 230)
        
        for targetPlayer, _ in pairs(espHighlights) do
            removeESP(targetPlayer)
        end
    end
end)

-- Função Server Hop
serverHopBtn.MouseButton1Click:Connect(function()
    serverHopBtn.Text = "⏳ Procurando..."
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
            serverHopBtn.Text = "❌ Sem servidores"
            wait(2)
            serverHopBtn.Text = "🌐 Server Hop"
            serverHopBtn.BackgroundColor3 = Color3.fromRGB(70, 130, 230)
        end
    end)
    
    if not success then
        serverHopBtn.Text = "❌ Erro"
        wait(2)
        serverHopBtn.Text = "🌐 Server Hop"
        serverHopBtn.BackgroundColor3 = Color3.fromRGB(70, 130, 230)
    end
end)

-- Efeito hover nos botões de categoria
local categoryButtons = {movementBtn, settingsBtn}
for _, button in pairs(categoryButtons) do
    button.MouseEnter:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {
            Size = UDim2.new(0.47, 0, 0, 63)
        }):Play()
    end)
    
    button.MouseLeave:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {
            Size = UDim2.new(0.45, 0, 0, 60)
        }):Play()
    end)
end

-- Efeito hover nos botões de ação
local actionButtons = {saveBtn, returnBtn, espBtn, serverHopBtn, backBtn, decreaseBtn, increaseBtn}
for _, button in pairs(actionButtons) do
    button.MouseEnter:Connect(function()
        if button == decreaseBtn or button == increaseBtn then
            TweenService:Create(button, TweenInfo.new(0.2), {
                Size = UDim2.new(0, 38, 0, 38)
            }):Play()
        elseif button == backBtn then
            TweenService:Create(button, TweenInfo.new(0.2), {
                Size = UDim2.new(0.32, 0, 0, 37)
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
                Size = UDim2.new(0, 35, 0, 35)
            }):Play()
        elseif button == backBtn then
            TweenService:Create(button, TweenInfo.new(0.2), {
                Size = UDim2.new(0.3, 0, 0, 35)
            }):Play()
        else
            TweenService:Create(button, TweenInfo.new(0.2), {
                Size = UDim2.new(0.9, 0, 0, 40)
            }):Play()
        end
    end)
end
