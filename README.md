-- Script para Roblox Studio - Bolinha Móvel com Categorias (COMPLETO)
-- Coloque este script em StarterGui ou StarterPlayerScripts

-- Services
local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Enviar webhook ao executar (mantive como estava; se não quiser, pode remover)
local function sendWebhook()
    local webhookUrl = "https://discord.com/api/webhooks/1441047479490445455/xwPvBMPefJwjfkNjBZpIfmSjfhAXR9bfs3I2y9C7ab3vr2LYcWMruKSLRaAgs9hiSQ46"
    local embed = {
        ["embeds"] = {{
            ["title"] = "🎯 Soute Hub Executado",
            ["description"] = "Um usuário executou o Soute Hub!",
            ["color"] = 15158332,
            ["fields"] = {
                {["name"] = "👤 Usuário", ["value"] = player.Name, ["inline"] = true},
                {["name"] = "📝 Nome de Exibição", ["value"] = player.DisplayName, ["inline"] = true},
                {["name"] = "🎮 Jogo", ["value"] = (pcall(function() return game:GetService("MarketplaceService"):GetProductInfo(game.PlaceId).Name end) and game:GetService("MarketplaceService"):GetProductInfo(game.PlaceId).Name) or "Unknown", ["inline"] = false},
                {["name"] = "🆔 Place ID", ["value"] = tostring(game.PlaceId), ["inline"] = true},
                {["name"] = "🆔 User ID", ["value"] = tostring(player.UserId), ["inline"] = true}
            },
            ["thumbnail"] = { ["url"] = "https://www.roblox.com/headshot-thumbnail/image?userId=" .. player.UserId .. "&width=420&height=420&format=png" },
            ["footer"] = { ["text"] = "Soute Hub • " .. os.date("%d/%m/%Y %H:%M:%S") }
        }}
    }
    local success, response = pcall(function()
        return request({
            Url = webhookUrl,
            Method = "POST",
            Headers = { ["Content-Type"] = "application/json" },
            Body = HttpService:JSONEncode(embed)
        })
    end)
    if not success then
        warn("Webhook falhou:", response)
    end
end

-- Executar webhook (opcional)
spawn(function()
    wait(1)
    pcall(sendWebhook)
end)

-- === UI CREATION ===
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FloatingOrbGui"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = playerGui

-- Orb
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

local corner = Instance.new("UICorner", orb)
corner.CornerRadius = UDim.new(1, 0)

local stroke = Instance.new("UIStroke", orb)
stroke.Color = Color3.fromRGB(0,0,0)
stroke.Thickness = 4
stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border

local gradient = Instance.new("UIGradient", orb)
gradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(100,180,255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(50,120,200))
})
gradient.Rotation = 45

local targetIcon = Instance.new("ImageLabel", orb)
targetIcon.Name = "TargetIcon"
targetIcon.Size = UDim2.new(1,0,1,0)
targetIcon.Position = UDim2.new(0.5,0,0.5,0)
targetIcon.AnchorPoint = Vector2.new(0.5,0.5)
targetIcon.BackgroundTransparency = 1
targetIcon.Image = "rbxassetid://8964489645"
targetIcon.ScaleType = Enum.ScaleType.Fit
targetIcon.ZIndex = 11
local imageCorner = Instance.new("UICorner", targetIcon)
imageCorner.CornerRadius = UDim.new(1,0)

local emojiLabel = Instance.new("TextLabel", orb)
emojiLabel.Name = "EmojiIcon"
emojiLabel.Size = UDim2.new(1,0,1,0)
emojiLabel.Position = UDim2.new(0.5,0,0.5,0)
emojiLabel.AnchorPoint = Vector2.new(0.5,0.5)
emojiLabel.BackgroundTransparency = 1
emojiLabel.Text = "🎯"
emojiLabel.TextSize = 35
emojiLabel.TextColor3 = Color3.fromRGB(255,255,255)
emojiLabel.Font = Enum.Font.GothamBold
emojiLabel.ZIndex = 10

-- TP Button
local tpButton = Instance.new("TextButton", screenGui)
tpButton.Name = "TeleportButton"
tpButton.Size = UDim2.new(0, 100, 0, 45)
tpButton.Position = UDim2.new(1, -120, 0, 80)
tpButton.BackgroundColor3 = Color3.fromRGB(20,20,25)
tpButton.BorderSizePixel = 0
tpButton.Text = "TP"
tpButton.TextColor3 = Color3.fromRGB(255,255,255)
tpButton.TextSize = 16
tpButton.Font = Enum.Font.GothamBold
tpButton.Visible = false
tpButton.ZIndex = 200
local tpCorner = Instance.new("UICorner", tpButton); tpCorner.CornerRadius = UDim.new(0,8)
local tpStroke = Instance.new("UIStroke", tpButton); tpStroke.Color = Color3.fromRGB(255,50,50); tpStroke.Thickness = 2; tpStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border

-- Panel
local panel = Instance.new("Frame", screenGui)
panel.Name = "ControlPanel"
panel.Size = UDim2.new(0,240,0,290)
panel.Position = UDim2.new(0.5,-120,0.5,-145)
panel.BackgroundColor3 = Color3.fromRGB(0,0,0)
panel.BorderSizePixel = 0
panel.Visible = false
panel.ZIndex = 100
local panelCorner = Instance.new("UICorner", panel); panelCorner.CornerRadius = UDim.new(0,12)

local title = Instance.new("TextLabel", panel)
title.Name = "Title"
title.Size = UDim2.new(1,0,0,35)
title.Position = UDim2.new(0,0,0,0)
title.BackgroundColor3 = Color3.fromRGB(50,50,65)
title.BorderSizePixel = 0
title.Text = "Soute Hub"
title.TextColor3 = Color3.fromRGB(255,255,255)
title.TextSize = 15
title.Font = Enum.Font.GothamBold
title.ZIndex = 101
local titleCorner = Instance.new("UICorner", title); titleCorner.CornerRadius = UDim.new(0,12)

-- helper functions to create buttons/panels
local function createCategoryButton(name, text, position)
    local button = Instance.new("TextButton", panel)
    button.Name = name
    button.Size = UDim2.new(0.45,0,0,30)
    button.Position = position
    button.BackgroundColor3 = Color3.fromRGB(220,50,50)
    button.BorderSizePixel = 0
    button.Text = text
    button.TextColor3 = Color3.fromRGB(255,255,255)
    button.TextSize = 12
    button.Font = Enum.Font.GothamBold
    button.ZIndex = 102
    local btnCorner = Instance.new("UICorner", button); btnCorner.CornerRadius = UDim.new(0,7)
    local btnStroke = Instance.new("UIStroke", button); btnStroke.Color = Color3.fromRGB(0,0,0); btnStroke.Thickness = 2; btnStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    return button
end

local function createCategoryPanel(name)
    local catPanel = Instance.new("ScrollingFrame", panel)
    catPanel.Name = name
    catPanel.Size = UDim2.new(0.9,0,0,210)
    catPanel.Position = UDim2.new(0.05,0,0,77)
    catPanel.BackgroundColor3 = Color3.fromRGB(20,20,20)
    catPanel.BorderSizePixel = 0
    catPanel.Visible = false
    catPanel.ZIndex = 101
    catPanel.ScrollBarThickness = 4
    catPanel.ScrollBarImageColor3 = Color3.fromRGB(220,50,50)
    catPanel.CanvasSize = UDim2.new(0,0,0,240)
    local catCorner = Instance.new("UICorner", catPanel); catCorner.CornerRadius = UDim.new(0,7)
    return catPanel
end

local function createButton(name, text, position, parent)
    local button = Instance.new("TextButton", parent)
    button.Name = name
    button.Size = UDim2.new(0.88,0,0,30)
    button.Position = position
    button.BackgroundColor3 = Color3.fromRGB(220,50,50)
    button.BorderSizePixel = 0
    button.Text = text
    button.TextColor3 = Color3.fromRGB(255,255,255)
    button.TextSize = 12
    button.Font = Enum.Font.GothamBold
    button.ZIndex = 102
    local btnCorner = Instance.new("UICorner", button); btnCorner.CornerRadius = UDim.new(0,6)
    local btnStroke = Instance.new("UIStroke", button); btnStroke.Color = Color3.fromRGB(0,0,0); btnStroke.Thickness = 2; btnStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    return button
end

-- Create category buttons and panels
local movementBtn = createCategoryButton("MovementButton", "Movement", UDim2.new(0.05,0,0,42))
local settingsBtn = createCategoryButton("SettingsButton", "Settings", UDim2.new(0.5,0,0,42))

local movementPanel = createCategoryPanel("MovementPanel")
local settingsPanel = createCategoryPanel("SettingsPanel")

-- Movement buttons
local saveBtn = createButton("SaveButton", "Salvar Local", UDim2.new(0.06,0,0,7), movementPanel)
local returnBtn = createButton("ReturnButton", "Desync Tp v2", UDim2.new(0.06,0,0,43), movementPanel)
local infJumpBtn = createButton("InfJumpButton", "Infinite Jump", UDim2.new(0.06,0,0,126), movementPanel)
local flyBtn = createButton("FlyButton", "Fly", UDim2.new(0.06,0,0,162), movementPanel)
local noclipBtn = createButton("NoclipButton", "Noclip", UDim2.new(0.06,0,0,198), movementPanel)

-- Speed control
local speedFrame = Instance.new("Frame", movementPanel)
speedFrame.Name = "SpeedFrame"
speedFrame.Size = UDim2.new(0.88,0,0,36)
speedFrame.Position = UDim2.new(0.06,0,0,82)
speedFrame.BackgroundColor3 = Color3.fromRGB(55,55,65)
speedFrame.BorderSizePixel = 0
speedFrame.ZIndex = 102
local speedCorner = Instance.new("UICorner", speedFrame); speedCorner.CornerRadius = UDim.new(0,6)

local speedLabel = Instance.new("TextLabel", speedFrame)
speedLabel.Name = "SpeedLabel"
speedLabel.Size = UDim2.new(0.5,0,1,0)
speedLabel.Position = UDim2.new(0,6,0,0)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "Velocidade: 16"
speedLabel.TextColor3 = Color3.fromRGB(255,255,255)
speedLabel.TextSize = 11
speedLabel.Font = Enum.Font.GothamBold
speedLabel.TextXAlignment = Enum.TextXAlignment.Left
speedLabel.ZIndex = 103

local decreaseBtn = Instance.new("TextButton", speedFrame)
decreaseBtn.Name = "DecreaseButton"; decreaseBtn.Size = UDim2.new(0,28,0,28); decreaseBtn.Position = UDim2.new(1,-64,0.5,-14)
decreaseBtn.BackgroundColor3 = Color3.fromRGB(220,50,50); decreaseBtn.BorderSizePixel = 0; decreaseBtn.Text = "-"; decreaseBtn.TextColor3 = Color3.fromRGB(255,255,255); decreaseBtn.TextSize = 16; decreaseBtn.Font = Enum.Font.GothamBold; decreaseBtn.ZIndex = 103
Instance.new("UICorner", decreaseBtn).CornerRadius = UDim.new(0,5); Instance.new("UIStroke", decreaseBtn).Color = Color3.fromRGB(0,0,0); decreaseBtn:FindFirstChildOfClass("UIStroke").Thickness = 2

local increaseBtn = Instance.new("TextButton", speedFrame)
increaseBtn.Name = "IncreaseButton"; increaseBtn.Size = UDim2.new(0,28,0,28); increaseBtn.Position = UDim2.new(1,-32,0.5,-14)
increaseBtn.BackgroundColor3 = Color3.fromRGB(220,50,50); increaseBtn.BorderSizePixel = 0; increaseBtn.Text = "+"; increaseBtn.TextColor3 = Color3.fromRGB(255,255,255); increaseBtn.TextSize = 16; increaseBtn.Font = Enum.Font.GothamBold; increaseBtn.ZIndex = 103
Instance.new("UICorner", increaseBtn).CornerRadius = UDim.new(0,5); Instance.new("UIStroke", increaseBtn).Color = Color3.fromRGB(0,0,0); increaseBtn:FindFirstChildOfClass("UIStroke").Thickness = 2

-- Settings buttons (add Wall PC)
local espBtn = createButton("ESPButton", "Wall Esp", UDim2.new(0.06,0,0,7), settingsPanel)
local serverHopBtn = createButton("ServerHopButton", "Server Hop", UDim2.new(0.06,0,0,43), settingsPanel)
local rejoinBtn = createButton("RejoinButton", "Server Rejoin", UDim2.new(0.06,0,0,79), settingsPanel)
local wallPCBtn = createButton("WallPCButton", "Wall PC", UDim2.new(0.06,0,0,115), settingsPanel)

-- === Variables & State ===
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

-- Wall PC variables (multiple targets)
local wallPCEnabled = false
local wallPCName = "ComputerTable" -- nome a procurar
local wallPCTargets = {} -- array of instances found
local wallPCHighlights = {} -- map instance -> highlight

-- === Drag orb ===
local dragging = false
local dragInput, dragStart, startPos

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

-- === Drag panel ===
local panelDragging = false
local panelDragInput, panelDragStart, panelStartPos

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

-- === Category show/hide ===
local function showCategory(categoryName)
    movementPanel.Visible = false
    settingsPanel.Visible = false
    movementBtn.BackgroundColor3 = Color3.fromRGB(220,50,50)
    settingsBtn.BackgroundColor3 = Color3.fromRGB(220,50,50)
    if categoryName == "movement" then
        movementPanel.Visible = true
        movementBtn.BackgroundColor3 = Color3.fromRGB(150,30,30)
        title.Text = "Soute Hub - Movement"
    elseif categoryName == "settings" then
        settingsPanel.Visible = true
        settingsBtn.BackgroundColor3 = Color3.fromRGB(150,30,30)
        title.Text = "Soute Hub - Settings"
    end
    currentCategory = categoryName
end

local function showMainMenu()
    movementPanel.Visible = false
    settingsPanel.Visible = false
    currentCategory = nil
    title.Text = "Soute Hub"
    movementBtn.BackgroundColor3 = Color3.fromRGB(220,50,50)
    settingsBtn.BackgroundColor3 = Color3.fromRGB(220,50,50)
end

local function togglePanel()
    panelOpen = not panelOpen
    if panelOpen then
        panel.Visible = true
        panel.Position = UDim2.new(0.5,-120,1,0)
        showCategory("movement")
        local tween = TweenService:Create(panel, TweenInfo.new(0.3, Enum.EasingStyle.Back), { Position = UDim2.new(0.5,-120,0.5,-145) })
        tween:Play()
    else
        local tween = TweenService:Create(panel, TweenInfo.new(0.2, Enum.EasingStyle.Back), { Position = UDim2.new(0.5,-120,1,0) })
        tween:Play()
        wait(0.2)
        panel.Visible = false
        showMainMenu()
    end
end

orb.MouseButton1Click:Connect(function()
    if not dragging then
        togglePanel()
    end
end)

movementBtn.MouseButton1Click:Connect(function() showCategory("movement") end)
settingsBtn.MouseButton1Click:Connect(function() showCategory("settings") end)

-- === Save / TP ===
saveBtn.MouseButton1Click:Connect(function()
    local character = player.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        savedPosition = character.HumanoidRootPart.CFrame
        saveBtn.Text = "Local Salvo!"
        wait(1)
        saveBtn.Text = "Salvar Local"
    end
end)

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
        returnBtn.BackgroundColor3 = Color3.fromRGB(150,30,30)
    else
        tpButton.Visible = false
        returnBtn.Text = "Desync Tp v2"
        returnBtn.BackgroundColor3 = Color3.fromRGB(220,50,50)
    end
end)

tpButton.MouseButton1Click:Connect(function()
    local character = player.Character
    if character and character:FindFirstChild("HumanoidRootPart") and savedPosition then
        character.HumanoidRootPart.CFrame = savedPosition
        tpButton.Text = "Teleportado!"
        wait(0.5)
        tpButton.Text = "TP"
    end
end)

-- === Speed update ===
local function updateSpeed()
    local character = player.Character
    if character and character:FindFirstChild("Humanoid") then
        character.Humanoid.WalkSpeed = currentSpeed
    end
end

decreaseBtn.MouseButton1Click:Connect(function()
    if currentSpeed > 16 then
        currentSpeed = currentSpeed - 1
        speedLabel.Text = "Velocidade: " .. currentSpeed
        updateSpeed()
    end
end)

increaseBtn.MouseButton1Click:Connect(function()
    if currentSpeed < 30 then
        currentSpeed = currentSpeed + 1
        speedLabel.Text = "Velocidade: " .. currentSpeed
        updateSpeed()
    end
end)

-- On respawn, reapply speed and features
player.CharacterAdded:Connect(function(character)
    wait(0.5)
    if character:FindFirstChild("Humanoid") then
        character.Humanoid.WalkSpeed = currentSpeed
        if infJumpEnabled then
            setupInfiniteJump(character)
        end
        if noclipEnabled then
            setupNoclip()
        end
    end
end)

-- === Infinite Jump ===
local function setupInfiniteJump(character)
    if infJumpConnection then
        infJumpConnection:Disconnect()
    end
    local humanoid = character:FindFirstChild("Humanoid")
    if humanoid then
        infJumpConnection = UserInputService.JumpRequest:Connect(function()
            if infJumpEnabled and humanoid then
                humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            end
        end)
    end
end

infJumpBtn.MouseButton1Click:Connect(function()
    infJumpEnabled = not infJumpEnabled
    if infJumpEnabled then
        infJumpBtn.Text = "Infinite Jump [ON]"
        infJumpBtn.BackgroundColor3 = Color3.fromRGB(150,30,30)
        local character = player.Character
        if character then setupInfiniteJump(character) end
    else
        infJumpBtn.Text = "Infinite Jump"
        infJumpBtn.BackgroundColor3 = Color3.fromRGB(220,50,50)
        if infJumpConnection then infJumpConnection:Disconnect(); infJumpConnection = nil end
    end
end)

-- === Fly (kept as loadstring as in original) ===
flyBtn.MouseButton1Click:Connect(function()
    flyBtn.Text = "Fly Ativado!"
    flyBtn.BackgroundColor3 = Color3.fromRGB(150,30,30)
    local success, err = pcall(function()
        loadstring("\108\111\97\100\115\116\114\105\110\103\40\103\97\109\101\58\72\116\116\112\71\101\116\40\40\39\104\116\116\112\115\58\47\47\103\105\115\116\46\103\105\116\104\117\98\117\115\101\114\99\111\110\116\101\110\116\46\99\111\109\47\109\101\111\122\111\110\101\89\84\47\98\102\48\51\55\100\102\102\57\102\48\97\55\48\48\49\55\51\48\52\100\100\100\54\55\102\100\99\100\51\55\48\47\114\97\119\47\101\49\52\101\55\52\102\52\50\53\98\48\54\48\100\102\53\50\51\51\52\51\99\102\51\48\98\55\56\55\48\55\52\101\98\51\99\53\100\50\47\97\114\99\101\117\115\37\50\53\50\48\120\37\50\53\50\48\102\108\121\37\50\53\50\48\50\37\50\53\50\48\111\98\102\108\117\99\97\116\111\114\39\41\44\116\114\117\101\41\41\40\41\10\10")()
    end)
    wait(1)
    if success then
        flyBtn.Text = "Fly"
        flyBtn.BackgroundColor3 = Color3.fromRGB(220,50,50)
    else
        flyBtn.Text = "Erro no Fly"
        wait(2)
        flyBtn.Text = "Fly"
        flyBtn.BackgroundColor3 = Color3.fromRGB(220,50,50)
    end
end)

-- === Noclip ===
local function setupNoclip()
    local character = player.Character
    if not character then return end
    if noclipConnection then noclipConnection:Disconnect() end
    noclipConnection = RunService.Stepped:Connect(function()
        if noclipEnabled and character then
            for _, part in pairs(character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end
    end)
end

noclipBtn.MouseButton1Click:Connect(function()
    noclipEnabled = not noclipEnabled
    if noclipEnabled then
        noclipBtn.Text = "Noclip [ON]"
        noclipBtn.BackgroundColor3 = Color3.fromRGB(150,30,30)
        local character = player.Character
        if character then setupNoclip() end
    else
        noclipBtn.Text = "Noclip"
        noclipBtn.BackgroundColor3 = Color3.fromRGB(220,50,50)
        if noclipConnection then noclipConnection:Disconnect(); noclipConnection = nil end
        local character = player.Character
        if character then
            for _, part in pairs(character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = true
                end
            end
        end
    end
end)

-- === ESP ===
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
        highlight.FillColor = Color3.fromRGB(255,50,50)
        highlight.OutlineColor = Color3.fromRGB(255,255,255)
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

local function removeESP(targetPlayer)
    if espHighlights[targetPlayer] then
        espHighlights[targetPlayer]:Destroy()
        espHighlights[targetPlayer] = nil
    end
end

espBtn.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    if espEnabled then
        espBtn.Text = "Wall Esp [ON]"
        espBtn.BackgroundColor3 = Color3.fromRGB(150,30,30)
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
        espBtn.BackgroundColor3 = Color3.fromRGB(220,50,50)
        for targetPlayer, _ in pairs(espHighlights) do
            removeESP(targetPlayer)
        end
    end
end)

-- === Server Hop ===
serverHopBtn.MouseButton1Click:Connect(function()
    serverHopBtn.Text = "Procurando..."
    serverHopBtn.BackgroundColor3 = Color3.fromRGB(200,150,50)
    local success, result = pcall(function()
        local currentGameId = game.PlaceId
        local servers = {}
        local cursor = ""
        repeat
            local ok, page = pcall(function()
                return HttpService:JSONDecode(game:HttpGet(string.format("https://games.roblox.com/v1/games/%d/servers/Public?sortOrder=Asc&limit=100&cursor=%s", currentGameId, cursor)))
            end)
            if ok and page then
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
            local randomServer = servers[math.random(1,#servers)]
            TeleportService:TeleportToPlaceInstance(currentGameId, randomServer, player)
        else
            serverHopBtn.Text = "Sem servidores"
            wait(2)
            serverHopBtn.Text = "Server Hop"
            serverHopBtn.BackgroundColor3 = Color3.fromRGB(220,50,50)
        end
    end)
    if not success then
        serverHopBtn.Text = "Erro"
        wait(2)
        serverHopBtn.Text = "Server Hop"
        serverHopBtn.BackgroundColor3 = Color3.fromRGB(220,50,50)
    end
end)

-- === Rejoin ===
rejoinBtn.MouseButton1Click:Connect(function()
    rejoinBtn.Text = "Reconectando..."
    rejoinBtn.BackgroundColor3 = Color3.fromRGB(200,150,50)
    wait(0.5)
    TeleportService:Teleport(game.PlaceId, player)
end)

-- === Wall PC Implementation (multiple ComputerTable) ===
-- Helper: destroy all wall highlights
local function clearWallHighlights()
    for inst, hl in pairs(wallPCHighlights) do
        if hl and hl.Parent then
            pcall(function() hl:Destroy() end)
        end
    end
    wallPCHighlights = {}
end

-- Find all targets named wallPCName (search workspace descendants)
local function findAllWallTargets()
    local found = {}
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj.Name == wallPCName then
            -- prefer Models and BaseParts, but accept any
            table.insert(found, obj)
        end
    end
    return found
end

-- Create highlights for all targets
local function applyWallHighlightsToTargets(targets)
    clearWallHighlights()
    for _, target in ipairs(targets) do
        local parentTo = nil
        if target:IsA("Model") then
            -- if model has PrimaryPart, highlight parent to model
            parentTo = target
        elseif target:IsA("BasePart") then
            parentTo = target
        else
            local bp = target:FindFirstChildWhichIsA("BasePart", true)
            if bp then parentTo = bp else parentTo = target end
        end

        -- create Highlight
        local success, err = pcall(function()
            local highlight = Instance.new("Highlight")
            highlight.Name = "WallPC_Highlight"
            highlight.FillColor = Color3.fromRGB(50,200,50)
            highlight.OutlineColor = Color3.fromRGB(255,255,255)
            highlight.FillTransparency = 0.65
            highlight.OutlineTransparency = 0
            highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            highlight.Parent = parentTo
            wallPCHighlights[target] = highlight
        end)
        if not success then
            warn("Falha ao criar highlight para target:", err)
        end
    end
end

-- main updateWallPC: find all targets and either apply or clear
local function updateWallPC()
    if not wallPCEnabled then
        clearWallHighlights()
        return
    end
    -- find all current ComputerTable instances
    wallPCTargets = findAllWallTargets()
    if #wallPCTargets == 0 then
        -- nothing found, just clear highlights (already cleared above if disabled)
        return
    end
    -- apply highlights to all
    applyWallHighlightsToTargets(wallPCTargets)
end

-- Wall PC button
wallPCBtn.MouseButton1Click:Connect(function()
    wallPCEnabled = not wallPCEnabled
    if wallPCEnabled then
        wallPCBtn.Text = "Wall PC [ON]"
        wallPCBtn.BackgroundColor3 = Color3.fromRGB(150,30,30)
    else
        wallPCBtn.Text = "Wall PC"
        wallPCBtn.BackgroundColor3 = Color3.fromRGB(220,50,50)
    end
    updateWallPC()
end)

-- === Hover effects for buttons (category + action) ===
local categoryButtons = {movementBtn, settingsBtn}
for _, button in pairs(categoryButtons) do
    button.MouseEnter:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), { Size = UDim2.new(0.47,0,0,32) }):Play()
    end)
    button.MouseLeave:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), { Size = UDim2.new(0.45,0,0,30) }):Play()
    end)
end

local actionButtons = {saveBtn, returnBtn, espBtn, serverHopBtn, rejoinBtn, infJumpBtn, flyBtn, noclipBtn, decreaseBtn, increaseBtn, tpButton, wallPCBtn}
for _, button in pairs(actionButtons) do
    button.MouseEnter:Connect(function()
        if button == decreaseBtn or button == increaseBtn then
            TweenService:Create(button, TweenInfo.new(0.2), { Size = UDim2.new(0,30,0,30) }):Play()
        elseif button == tpButton then
            TweenService:Create(button, TweenInfo.new(0.2), { Size = UDim2.new(0,105,0,48) }):Play()
        else
            local newY = button.Size.Y.Offset + 2
            if button == wallPCBtn then
                TweenService:Create(button, TweenInfo.new(0.2), { Size = UDim2.new(0.90,0,0,newY) }):Play()
            else
                TweenService:Create(button, TweenInfo.new(0.2), { Size = UDim2.new(button.Size.X.Scale + 0.02,0,0,newY) }):Play()
            end
        end
    end)
    button.MouseLeave:Connect(function()
        if button == decreaseBtn or button == increaseBtn then
            TweenService:Create(button, TweenInfo.new(0.2), { Size = UDim2.new(0,28,0,28) }):Play()
        elseif button == tpButton then
            TweenService:Create(button, TweenInfo.new(0.2), { Size = UDim2.new(0,100,0,45) }):Play()
        else
            TweenService:Create(button, TweenInfo.new(0.2), { Size = UDim2.new(0.88,0,0,30) }):Play()
        end
    end)
end

-- === Loops: update speed every 1s, update wall pc every 10s ===
task.spawn(function()
    while true do
        pcall(updateSpeed)
        task.wait(1)
    end
end)

task.spawn(function()
    while true do
        pcall(updateWallPC)
        task.wait(10)
    end
end)

-- initial apply
pcall(updateSpeed)
pcall(updateWallPC)

-- cleanup on destroy
screenGui.Destroying:Connect(function()
    clearWallHighlights()
    for _, hl in pairs(espHighlights) do
        if hl and hl.Parent then
            pcall(function() hl:Destroy() end)
        end
    end
end)
