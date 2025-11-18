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
targetIcon.ScaleType = Enum.ScaleType.Crop
targetIcon.ZIndex = 11
targetIcon.Parent = orb

-- Arredondar a imagem também
local imageCorner = Instance.new("UICorner")
imageCorner.CornerRadius = UDim.new(1, 0)
imageCorner.Parent = targetIcon

-- Criar painel de controle
local panel = Instance.new("Frame")
panel.Name = "ControlPanel"
panel.Size = UDim2.new(0, 250, 0, 270)
panel.Position = UDim2.new(0.5, -125, 0.5, -135)
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
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
title.BackgroundTransparency = 0.3
title.BorderSizePixel = 0
title.Text = "Controles"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 18
title.Font = Enum.Font.GothamBold
title.ZIndex = 101
title.Parent = panel

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 12)
titleCorner.Parent = title

-- Função para criar botões
local function createButton(name, text, position)
    local button = Instance.new("TextButton")
    button.Name = name
    button.Size = UDim2.new(0.85, 0, 0, 40)
    button.Position = position
    button.BackgroundColor3 = Color3.fromRGB(60, 120, 220)
    button.BorderSizePixel = 0
    button.Text = text
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 14
    button.Font = Enum.Font.GothamBold
    button.ZIndex = 102
    button.Parent = panel
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 8)
    btnCorner.Parent = button
    
    return button
end

-- Criar botões
local saveBtn = createButton("SaveButton", "Salvar Local", UDim2.new(0.075, 0, 0, 55))
local returnBtn = createButton("ReturnButton", "Desync Tp v2", UDim2.new(0.075, 0, 0, 105))
local espBtn = createButton("ESPButton", "Wall Esp", UDim2.new(0.075, 0, 0, 155))
local serverHopBtn = createButton("ServerHopButton", "Server Hop", UDim2.new(0.075, 0, 0, 205))

-- Variáveis de controle
local panelOpen = false
local savedPosition = nil
local espEnabled = false
local espHighlights = {}

-- Sistema de arrastar a bolinha (mobile e PC)
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

-- Função para abrir/fechar painel
local function togglePanel()
    panelOpen = not panelOpen
    
    if panelOpen then
        panel.Visible = true
        panel.Position = UDim2.new(0.5, -125, 1, 0)
        local tween = TweenService:Create(panel, TweenInfo.new(0.3, Enum.EasingStyle.Back), {
            Position = UDim2.new(0.5, -125, 0.5, -135)
        })
        tween:Play()
    else
        local tween = TweenService:Create(panel, TweenInfo.new(0.2, Enum.EasingStyle.Back), {
            Position = UDim2.new(0.5, -125, 1, 0)
        })
        tween:Play()
        wait(0.2)
        panel.Visible = false
    end
end

-- Clique na bolinha para abrir/fechar painel
orb.MouseButton1Click:Connect(function()
    if not dragging then
        togglePanel()
    end
end)

-- Função salvar posição
saveBtn.MouseButton1Click:Connect(function()
    local character = player.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        savedPosition = character.HumanoidRootPart.CFrame
        saveBtn.Text = "✅ Local Salvo!"
        wait(1)
        saveBtn.Text = "Salvar Local"
    end
end)

-- Função voltar à posição
returnBtn.MouseButton1Click:Connect(function()
    local character = player.Character
    if character and character:FindFirstChild("HumanoidRootPart") and savedPosition then
        character.HumanoidRootPart.CFrame = savedPosition
        returnBtn.Text = "✅ Teleportado!"
        wait(1)
        returnBtn.Text = "Desync Tp v2"
    else
        returnBtn.Text = "❌ Sem posição"
        wait(1)
        returnBtn.Text = "Desync Tp v2"
    end
end)

-- Função para criar ESP em um jogador
local function createESP(targetPlayer)
    if targetPlayer == player then return end
    
    local function setupESP()
        local character = targetPlayer.Character
        if not character then return end
        
        -- Remover ESP antigo se existir
        if espHighlights[targetPlayer] then
            espHighlights[targetPlayer]:Destroy()
        end
        
        -- Criar Highlight
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
    
    -- Reconectar quando o personagem respawnar
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
        espBtn.BackgroundColor3 = Color3.fromRGB(50, 200, 100)
        
        -- Ativar ESP para todos os jogadores
        for _, targetPlayer in pairs(Players:GetPlayers()) do
            createESP(targetPlayer)
        end
        
        -- Ativar para novos jogadores
        Players.PlayerAdded:Connect(function(targetPlayer)
            if espEnabled then
                targetPlayer.CharacterAdded:Wait()
                wait(0.5)
                createESP(targetPlayer)
            end
        end)
    else
        espBtn.Text = "Wall Esp"
        espBtn.BackgroundColor3 = Color3.fromRGB(60, 120, 220)
        
        -- Remover ESP de todos
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
        
        -- Buscar servidores disponíveis
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
        
        -- Teleportar para servidor aleatório
        if #servers > 0 then
            local randomServer = servers[math.random(1, #servers)]
            TeleportService:TeleportToPlaceInstance(currentGameId, randomServer, player)
        else
            serverHopBtn.Text = "❌ Sem servidores"
            wait(2)
            serverHopBtn.Text = "Server Hop"
            serverHopBtn.BackgroundColor3 = Color3.fromRGB(60, 120, 220)
        end
    end)
    
    if not success then
        serverHopBtn.Text = "❌ Erro"
        wait(2)
        serverHopBtn.Text = "Server Hop"
        serverHopBtn.BackgroundColor3 = Color3.fromRGB(60, 120, 220)
    end
end)

-- Efeito hover nos botões
local buttons = {saveBtn, returnBtn, espBtn, serverHopBtn}
for _, button in pairs(buttons) do
    button.MouseEnter:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {
            Size = UDim2.new(0.88, 0, 0, 42)
        }):Play()
    end)
    
    button.MouseLeave:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {
            Size = UDim2.new(0.85, 0, 0, 40)
        }):Play()
    end)
end
