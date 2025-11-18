-- Script para Roblox Studio - Bolinha Flutuante com ESP
-- Coloque este script em StarterGui ou StarterPlayerScripts

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
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

-- Criar painel de controle
local panel = Instance.new("Frame")
panel.Name = "ControlPanel"
panel.Size = UDim2.new(0, 250, 0, 220)
panel.Position = UDim2.new(0.5, -125, 0.5, -110)
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

-- Variáveis de controle
local panelOpen = false
local savedPosition = nil
local espEnabled = false
local espConnections = {}
local espHighlights = {}
local espTracers = {}

-- Animação da bolinha flutuante
local floatTween
local function animateOrb()
    local goal = {}
    goal.Position = UDim2.new(orb.Position.X.Scale, 0, orb.Position.Y.Scale + 0.02, 0)
    local tweenInfo = TweenInfo.new(1.5, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true)
    floatTween = TweenService:Create(orb, tweenInfo, goal)
    floatTween:Play()
end

animateOrb()

-- Função para abrir/fechar painel
local function togglePanel()
    panelOpen = not panelOpen
    
    print("Painel toggled:", panelOpen) -- Debug
    
    if panelOpen then
        panel.Visible = true
        panel.Position = UDim2.new(0.5, -125, 1, 0)
        local tween = TweenService:Create(panel, TweenInfo.new(0.3, Enum.EasingStyle.Back), {
            Position = UDim2.new(0.5, -125, 0.5, -110)
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
    togglePanel()
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
        
        if espTracers[targetPlayer] then
            espTracers[targetPlayer]:Destroy()
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
        
        -- Criar linha traçadora (tracer)
        local tracerGui = Instance.new("ScreenGui")
        tracerGui.Name = "ESP_Tracer"
        tracerGui.ResetOnSpawn = false
        tracerGui.IgnoreGuiInset = true
        tracerGui.Parent = playerGui
        
        local tracerFrame = Instance.new("Frame")
        tracerFrame.Name = "TracerLine"
        tracerFrame.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        tracerFrame.BorderSizePixel = 0
        tracerFrame.AnchorPoint = Vector2.new(0.5, 0)
        tracerFrame.Parent = tracerGui
        
        espTracers[targetPlayer] = tracerGui
        
        -- Atualizar posição da linha em tempo real
        local connection = RunService.RenderStepped:Connect(function()
            if not character or not character:FindFirstChild("HumanoidRootPart") then return end
            
            local camera = workspace.CurrentCamera
            local characterPos = character.HumanoidRootPart.Position
            
            -- Converter posição 3D para 2D
            local screenPos, onScreen = camera:WorldToViewportPoint(characterPos)
            
            if onScreen then
                -- Ponto inicial (topo central da tela)
                local startX = camera.ViewportSize.X / 2
                local startY = 0
                
                -- Ponto final (posição do jogador na tela) - ajustado para ir até os pés
                local feetPos = character.HumanoidRootPart.Position - Vector3.new(0, 3, 0)
                local feetScreen = camera:WorldToViewportPoint(feetPos)
                local endX = feetScreen.X
                local endY = feetScreen.Y
                
                -- Calcular distância e ângulo
                local distance = math.sqrt((endX - startX)^2 + (endY - startY)^2)
                local angle = math.deg(math.atan2(endY - startY, endX - startX))
                
                -- Atualizar a linha
                tracerFrame.Size = UDim2.new(0, distance, 0, 2)
                tracerFrame.Position = UDim2.new(0, startX, 0, startY)
                tracerFrame.Rotation = angle
                tracerFrame.Visible = true
            else
                tracerFrame.Visible = false
            end
        end)
        
        espConnections[targetPlayer] = connection
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
    
    if espTracers[targetPlayer] then
        espTracers[targetPlayer]:Destroy()
        espTracers[targetPlayer] = nil
    end
    
    if espConnections[targetPlayer] then
        espConnections[targetPlayer]:Disconnect()
        espConnections[targetPlayer] = nil
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

-- Efeito hover nos botões
local buttons = {saveBtn, returnBtn, espBtn}
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
