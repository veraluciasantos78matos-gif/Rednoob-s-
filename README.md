local loadstring = loadstring or load

local Script = [[
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local TeleportService = game:GetService("TeleportService")

local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

-- Configurações
local Settings = {
    ESP = {
        Players = true,
        Brainrot = true,
        Distance = true,
        Health = true,
        Boxes = true,
        Tracers = true,
        Names = true,
        Color = Color3.fromRGB(0, 255, 0)
    },
    XRay = {
        Enabled = false,
        BaseWalls = true,
        Transparency = 0.3
    },
    Teleport = {
        SemiInsta = true,
        Speed = 100,
        BrainrotTP = true,
        CheckBaseTime = true
    },
    Protection = {
        AntiGrab = true,
        AntiStun = true,
        AntiVoid = true,
        BaseProtector = true
    },
    Spam = {
        AP = false,
        APDelay = 0.1,
        APMessage = "STEAL A BRAINROT HACKS ACTIVATED"
    },
    BrainrotSteal = {
        AutoSteal = false,
        StealDistance = 10,
        StealCooldown = 1
    }
}

-- Variáveis globais
local ESPFolder = Instance.new("Folder")
ESPFolder.Name = "BrainrotSteal_ESP"
ESPFolder.Parent = CoreGui

local XRayFolder = Instance.new("Folder")
XRayFolder.Name = "BrainrotSteal_XRay"
XRayFolder.Parent = CoreGui

local Teleporting = false
local Connections = {}
local LastStealTime = 0

-- Função de bypass
local function BypassTeleport(destination)
    if Settings.Teleport.SemiInsta then
        local character = LocalPlayer.Character
        if character and character:FindFirstChild("HumanoidRootPart") then
            local tween = TweenService:Create(character.HumanoidRootPart, 
                TweenInfo.new((destination - character.HumanoidRootPart.Position).Magnitude/Settings.Teleport.Speed, 
                Enum.EasingStyle.Linear), 
                {CFrame = CFrame.new(destination)})
            tween:Play()
            tween.Completed:Wait()
        end
    else
        LocalPlayer.Character:SetPrimaryPartCFrame(CFrame.new(destination))
    end
end

-- Sistema ESP
local ESPCache = {}
local function CreateESP(player)
    if not player.Character then return end
    
    if ESPCache[player] then
        for _, obj in pairs(ESPCache[player]) do
            if obj then
                obj:Destroy()
            end
        end
    end
    
    local esp = {}
    
    -- Box ESP
    if Settings.ESP.Boxes then
        esp.Box = Instance.new("BoxHandleAdornment")
        esp.Box.Name = player.Name .. "_Box"
        esp.Box.Adornee = player.Character:WaitForChild("HumanoidRootPart", 5)
        if esp.Box.Adornee then
            esp.Box.AlwaysOnTop = true
            esp.Box.ZIndex = 5
            esp.Box.Size = Vector3.new(4, 6, 1)
            esp.Box.Transparency = 0.3
            esp.Box.Color3 = Settings.ESP.Color
            esp.Box.Parent = ESPFolder
        end
    end
    
    -- Name Tag
    if Settings.ESP.Names then
        esp.NameTag = Instance.new("BillboardGui")
        esp.NameTag.Name = player.Name .. "_Name"
        esp.NameTag.Adornee = player.Character:WaitForChild("HumanoidRootPart", 5)
        if esp.NameTag.Adornee then
            esp.NameTag.Size = UDim2.new(0, 200, 0, 50)
            esp.NameTag.StudsOffset = Vector3.new(0, 3.5, 0)
            esp.NameTag.AlwaysOnTop = true
            
            local nameLabel = Instance.new("TextLabel")
            nameLabel.Text = player.Name
            nameLabel.TextColor3 = Settings.ESP.Color
            nameLabel.TextStrokeTransparency = 0
            nameLabel.TextSize = 14
            nameLabel.Font = Enum.Font.GothamBold
            nameLabel.BackgroundTransparency = 1
            nameLabel.Size = UDim2.new(1, 0, 1, 0)
            nameLabel.Parent = esp.NameTag
            
            esp.NameTag.Parent = ESPFolder
        end
    end
    
    -- Health Bar
    if Settings.ESP.Health and player.Character:FindFirstChild("Humanoid") then
        esp.HealthBar = Instance.new("BillboardGui")
        esp.HealthBar.Name = player.Name .. "_Health"
        esp.HealthBar.Adornee = player.Character:WaitForChild("HumanoidRootPart", 5)
        if esp.HealthBar.Adornee then
            esp.HealthBar.Size = UDim2.new(2, 0, 0.2, 0)
            esp.HealthBar.StudsOffset = Vector3.new(0, 2.8, 0)
            esp.HealthBar.AlwaysOnTop = true
            
            local healthFrame = Instance.new("Frame")
            healthFrame.Size = UDim2.new(1, 0, 1, 0)
            healthFrame.BackgroundColor3 = Color3.new(0, 0, 0)
            healthFrame.BorderSizePixel = 1
            
            local healthFill = Instance.new("Frame")
            healthFill.Size = UDim2.new(player.Character.Humanoid.Health/100, 0, 1, 0)
            healthFill.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
            healthFill.BorderSizePixel = 0
            healthFill.Parent = healthFrame
            
            healthFrame.Parent = esp.HealthBar
            esp.HealthBar.Parent = ESPFolder
        end
    end
    
    ESPCache[player] = esp
    return esp
end

-- ESP Brainrot
local function BrainrotESP()
    for _, obj in pairs(Workspace:GetDescendants()) do
        if (obj.Name:find("Brainrot") or obj.Name:find("Monster") or obj.Name:find("Zombie")) and not obj:FindFirstChild("BrainrotESP") then
            local highlight = Instance.new("Highlight")
            highlight.Name = "BrainrotESP"
            highlight.FillColor = Color3.fromRGB(255, 0, 255)
            highlight.OutlineColor = Color3.fromRGB(255, 255, 0)
            highlight.FillTransparency = 0.5
            highlight.OutlineTransparency = 0
            highlight.Adornee = obj
            highlight.Parent = obj
            
            -- Label para Brainrot
            local billboard = Instance.new("BillboardGui")
            billboard.Name = "BrainrotLabel"
            billboard.Size = UDim2.new(0, 100, 0, 30)
            billboard.StudsOffset = Vector3.new(0, 3, 0)
            billboard.AlwaysOnTop = true
            billboard.Adornee = obj.PrimaryPart or obj:FindFirstChild("HumanoidRootPart") or obj:FindFirstChild("Head")
            
            local label = Instance.new("TextLabel")
            label.Text = "BRAINROT"
            label.TextColor3 = Color3.fromRGB(255, 0, 255)
            label.TextStrokeTransparency = 0
            label.TextSize = 14
            label.Font = Enum.Font.GothamBold
            label.BackgroundTransparency = 1
            label.Size = UDim2.new(1, 0, 1, 0)
            label.Parent = billboard
            
            if billboard.Adornee then
                billboard.Parent = obj
            end
        end
    end
end

-- X-Ray para bases e paredes
local function XRayBases()
    for _, part in pairs(Workspace:GetDescendants()) do
        if part:IsA("BasePart") and (part.Name:find("Wall") or part.Name:find("Base") or part.Name:find("Building") or part.Name:find("Fence")) then
            part.LocalTransparencyModifier = Settings.XRay.Enabled and Settings.XRay.Transparency or 0
        end
    end
end

-- Encontrar Brainrot mais próximo
local function FindClosestBrainrot()
    local closestBrainrot = nil
    local closestDistance = math.huge
    
    for _, obj in pairs(Workspace:GetDescendants()) do
        if (obj.Name:find("Brainrot") or obj.Name:find("Monster") or obj.Name:find("Zombie")) and obj:IsA("Model") then
            local root = obj:FindFirstChild("HumanoidRootPart") or obj:FindFirstChild("Head") or obj.PrimaryPart
            if root and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                local distance = (LocalPlayer.Character.HumanoidRootPart.Position - root.Position).Magnitude
                if distance < closestDistance then
                    closestDistance = distance
                    closestBrainrot = {Model = obj, Root = root}
                end
            end
        end
    end
    
    return closestBrainrot, closestDistance
end

-- Teleport para Brainrot
local function TeleportToBrainrot()
    local brainrotData, distance = FindClosestBrainrot()
    if brainrotData then
        BypassTeleport(brainrotData.Root.Position)
        return brainrotData.Model
    end
    return nil
end

-- Sistema de roubo automático de Brainrot
local function AutoStealBrainrot()
    if not Settings.BrainrotSteal.AutoSteal then return end
    if tick() - LastStealTime < Settings.BrainrotSteal.StealCooldown then return end
    
    local brainrotData, distance = FindClosestBrainrot()
    if brainrotData and distance <= Settings.BrainrotSteal.StealDistance then
        -- Simular ação de roubo (ajuste conforme o jogo)
        local args = {
            [1] = brainrotData.Model,
            [2] = "Steal"
        }
        
        -- Tentar encontrar o RemoteEvent correto para roubar
        local remotes = game:GetService("ReplicatedStorage"):FindFirstChild("Remotes") or 
                       game:GetService("ReplicatedStorage"):FindFirstChild("Events") or
                       game:GetService("ReplicatedStorage"):FindFirstChild("RemoteEvents")
        
        if remotes then
            for _, remote in pairs(remotes:GetChildren()) do
                if remote:IsA("RemoteEvent") and (remote.Name:find("Steal") or remote.Name:find("Rob") or remote.Name:find("Take")) then
                    remote:FireServer(unpack(args))
                    LastStealTime = tick()
                    print("[AUTO-STEAL] Tentativa de roubo no Brainrot")
                    break
                end
            end
        end
        
        -- Tentativa alternativa via __namecall
        game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = brainrotData.Root.CFrame
        wait(0.1)
        fireclickdetector(brainrotData.Model:FindFirstChildOfClass("ClickDetector"))
        LastStealTime = tick()
    end
end

-- Base Protector
local function BaseProtector()
    local character = LocalPlayer.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        local baseParts = Workspace:FindFirstChild("Base") or Workspace:FindFirstChild("SafeZone") or Workspace:FindFirstChild("Safe")
        if baseParts then
            local basePosition = baseParts:IsA("Model") and (baseParts.PrimaryPart and baseParts.PrimaryPart.Position or baseParts:GetPivot().Position) or baseParts.Position
            local distance = (character.HumanoidRootPart.Position - basePosition).Magnitude
            
            if distance > 50 and Settings.Protection.BaseProtector then
                BypassTeleport(basePosition)
                print("[PROTECTION] Retornando para base")
            end
        end
    end
end

-- Spam AP
local function SpamAP()
    while Settings.Spam.AP do
        pcall(function()
            game:GetService("ReplicatedStorage"):WaitForChild("DefaultChatSystemChatEvents"):WaitForChild("SayMessageRequest"):FireServer(
                Settings.Spam.APMessage, "All"
            )
        end)
        wait(Settings.Spam.APDelay)
    end
end

-- GUI inspirado no Fun Hub
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "BrainrotStealHub"
ScreenGui.Parent = CoreGui

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 450, 0, 550)
MainFrame.Position = UDim2.new(0.5, -225, 0.5, -275)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 12)
UICorner.Parent = MainFrame

local TopBar = Instance.new("Frame")
TopBar.Size = UDim2.new(1, 0, 0, 40)
TopBar.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
TopBar.BorderSizePixel = 0
TopBar.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Text = "🧠 STEAL A BRAINROT HUB 🧠"
Title.TextColor3 = Color3.fromRGB(255, 105, 180)
Title.TextSize = 22
Title.Font = Enum.Font.GothamBold
Title.BackgroundTransparency = 1
Title.Size = UDim2.new(1, 0, 1, 0)
Title.Parent = TopBar

local CloseButton = Instance.new("TextButton")
CloseButton.Text = "X"
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.TextSize = 18
CloseButton.Font = Enum.Font.GothamBold
CloseButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
CloseButton.Size = UDim2.new(0, 30, 0, 30)
CloseButton.Position = UDim2.new(1, -35, 0, 5)
CloseButton.Parent = TopBar

CloseButton.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
    ESPFolder:Destroy()
    XRayFolder:Destroy()
    for _, conn in pairs(Connections) do
        conn:Disconnect()
    end
end)

local TabButtonsFrame = Instance.new("Frame")
TabButtonsFrame.Size = UDim2.new(1, -20, 0, 40)
TabButtonsFrame.Position = UDim2.new(0, 10, 0, 50)
TabButtonsFrame.BackgroundTransparency = 1
TabButtonsFrame.Parent = MainFrame

local TabButtons = {}
local TabFrames = {}

local Tabs = {"ESP", "TELEPORT", "PROTECTION", "SPAM", "BRAINROT", "VISUALS"}

for i, tabName in ipairs(Tabs) do
    local tabButton = Instance.new("TextButton")
    tabButton.Text = tabName
    tabButton.TextColor3 = Color3.fromRGB(200, 200, 200)
    tabButton.TextSize = 12
    tabButton.Font = Enum.Font.GothamBold
    tabButton.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
    tabButton.Size = UDim2.new(1/#Tabs - 0.02, 0, 1, 0)
    tabButton.Position = UDim2.new((i-1)/#Tabs + 0.01, 0, 0, 0)
    tabButton.Parent = TabButtonsFrame
    
    local tabFrame = Instance.new("ScrollingFrame")
    tabFrame.Size = UDim2.new(1, -20, 1, -120)
    tabFrame.Position = UDim2.new(0, 10, 0, 100)
    tabFrame.BackgroundTransparency = 1
    tabFrame.Visible = i == 1
    tabFrame.ScrollBarThickness = 5
    tabFrame.CanvasSize = UDim2.new(0, 0, 0, 600)
    tabFrame.Parent = MainFrame
    
    TabButtons[tabName] = tabButton
    TabFrames[tabName] = tabFrame
    
    tabButton.MouseButton1Click:Connect(function()
        for _, frame in pairs(TabFrames) do
            frame.Visible = false
        end
        tabFrame.Visible = true
        
        -- Atualizar cor dos botões
        for _, btn in pairs(TabButtons) do
            btn.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
        end
        tabButton.BackgroundColor3 = Color3.fromRGB(255, 105, 180)
    end)
end

-- Função para criar toggles
local yOffset = 10
local function CreateToggle(parent, text, settingTable, settingKey)
    local toggleFrame = Instance.new("Frame")
    toggleFrame.Size = UDim2.new(1, -20, 0, 35)
    toggleFrame.Position = UDim2.new(0, 10, 0, yOffset)
    toggleFrame.BackgroundTransparency = 1
    
    local toggleText = Instance.new("TextLabel")
    toggleText.Text = "   " .. text
    toggleText.TextColor3 = Color3.fromRGB(220, 220, 220)
    toggleText.TextSize = 14
    toggleText.Font = Enum.Font.GothamSemibold
    toggleText.TextXAlignment = Enum.TextXAlignment.Left
    toggleText.BackgroundTransparency = 1
    toggleText.Size = UDim2.new(0.7, 0, 1, 0)
    toggleText.Parent = toggleFrame
    
    local toggleButton = Instance.new("TextButton")
    toggleButton.Size = UDim2.new(0.2, 0, 0.7, 0)
    toggleButton.Position = UDim2.new(0.75, 0, 0.15, 0)
    toggleButton.Text = settingTable[settingKey] and "ON" or "OFF"
    toggleButton.TextColor3 = settingTable[settingKey] and Color3.fromRGB(0, 255, 150) or Color3.fromRGB(255, 50, 50)
    toggleButton.BackgroundColor3 = settingTable[settingKey] and Color3.fromRGB(30, 80, 50) or Color3.fromRGB(80, 30, 30)
    toggleButton.Font = Enum.Font.GothamBold
    toggleButton.Parent = toggleFrame
    
    toggleButton.MouseButton1Click:Connect(function()
        settingTable[settingKey] = not settingTable[settingKey]
        toggleButton.Text = settingTable[settingKey] and "ON" or "OFF"
        toggleButton.TextColor3 = settingTable[settingKey] and Color3.fromRGB(0, 255, 150) or Color3.fromRGB(255, 50, 50)
        toggleButton.BackgroundColor3 = settingTable[settingKey] and Color3.fromRGB(30, 80, 50) or Color3.fromRGB(80, 30, 30)
    end)
    
    toggleFrame.Parent = parent
    yOffset = yOffset + 40
end

-- Criar controles ESP
yOffset = 10
CreateToggle(TabFrames["ESP"], "Player ESP", Settings.ESP, "Players")
CreateToggle(TabFrames["ESP"], "Brainrot ESP", Settings.ESP, "Brainrot")
CreateToggle(TabFrames["ESP"], "Show Distance", Settings.ESP, "Distance")
CreateToggle(TabFrames["ESP"], "Show Health", Settings.ESP, "Health")
CreateToggle(TabFrames["ESP"], "Box ESP", Settings.ESP, "Boxes")
CreateToggle(TabFrames["ESP"], "Tracers", Settings.ESP, "Tracers")
CreateToggle(TabFrames["ESP"], "Show Names", Settings.ESP, "Names")

-- Criar controles TELEPORT
yOffset = 10
CreateToggle(TabFrames["TELEPORT"], "Semi-Insta TP", Settings.Teleport, "SemiInsta")
CreateToggle(TabFrames["TELEPORT"], "Brainrot TP", Settings.Teleport, "BrainrotTP")

-- Slider de velocidade
local speedSliderFrame = Instance.new("Frame")
speedSliderFrame.Size = UDim2.new(1, -20, 0, 50)
speedSliderFrame.Position = UDim2.new(0, 10, 0, yOffset)
speedSliderFrame.BackgroundTransparency = 1

local speedText = Instance.new("TextLabel")
speedText.Text = "   TP Speed: " .. Settings.Teleport.Speed
speedText.TextColor3 = Color3.fromRGB(220, 220, 220)
speedText.TextSize = 14
speedText.Font = Enum.Font.GothamSemibold
speedText.TextXAlignment = Enum.TextXAlignment.Left
speedText.BackgroundTransparency = 1
speedText.Size = UDim2.new(1, 0, 0.5, 0)
speedText.Parent = speedSliderFrame

local speedValueBox = Instance.new("TextBox")
speedValueBox.Text = tostring(Settings.Teleport.Speed)
speedValueBox.Size = UDim2.new(0.3, 0, 0.5, 0)
speedValueBox.Position = UDim2.new(0.65, 0, 0.5, 0)
speedValueBox.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
speedValueBox.TextColor3 = Color3.fromRGB(255, 255, 255)
speedValueBox.PlaceholderColor3 = Color3.fromRGB(150, 150, 150)
speedValueBox.PlaceholderText = "Speed"
speedValueBox.Font = Enum.Font.Gotham
speedValueBox.Parent = speedSliderFrame

speedValueBox.FocusLost:Connect(function()
    local newSpeed = tonumber(speedValueBox.Text)
    if newSpeed and newSpeed > 0 and newSpeed <= 500 then
        Settings.Teleport.Speed = newSpeed
        speedText.Text = "   TP Speed: " .. Settings.Teleport.Speed
    else
        speedValueBox.Text = tostring(Settings.Teleport.Speed)
    end
end)

speedSliderFrame.Parent = TabFrames["TELEPORT"]
yOffset = yOffset + 60

-- Botão de TP para Brainrot
local brainrotButtonFrame = Instance.new("Frame")
brainrotButtonFrame.Size = UDim2.new(1, -20, 0, 60)
brainrotButtonFrame.Position = UDim2.new(0, 10, 0, yOffset)
brainrotButtonFrame.BackgroundTransparency = 1

local brainrotButton = Instance.new("TextButton")
brainrotButton.Text = "🧠 TELEPORT TO BRAINROT 🧠"
brainrotButton.Size = UDim2.new(1, 0, 1, 0)
brainrotButton.BackgroundColor3 = Color3.fromRGB(255, 105, 180)
brainrotButton.TextColor3 = Color3.fromRGB(255, 255, 255)
brainrotButton.Font = Enum.Font.GothamBold
brainrotButton.TextSize = 16
brainrotButton.Parent = brainrotButtonFrame

brainrotButton.MouseButton1Click:Connect(function()
    local brainrotModel = TeleportToBrainrot()
    if brainrotModel then
        brainrotButton.Text = "✅ TELEPORTED!"
        wait(1)
        brainrotButton.Text = "🧠 TELEPORT TO BRAINROT 🧠"
    else
        brainrotButton.Text = "❌ NO BRAINROT FOUND!"
        wait(1)
        brainrotButton.Text = "🧠 TELEPORT TO BRAINROT 🧠"
    end
end)

brainrotButtonFrame.Parent = TabFrames["TELEPORT"]

-- Criar controles PROTECTION
yOffset = 10
CreateToggle(TabFrames["PROTECTION"], "Anti-Grab", Settings.Protection, "AntiGrab")
CreateToggle(TabFrames["PROTECTION"], "Anti-Stun", Settings.Protection, "AntiStun")
CreateToggle(TabFrames["PROTECTION"], "Anti-Void", Settings.Protection, "AntiVoid")
CreateToggle(TabFrames["PROTECTION"], "Base Protector", Settings.Protection, "BaseProtector")

-- Criar controles SPAM
yOffset = 10
CreateToggle(TabFrames["SPAM"], "Spam AP", Settings.Spam, "AP")

local apMessageFrame = Instance.new("Frame")
apMessageFrame.Size = UDim2.new(1, -20, 0, 40)
apMessageFrame.Position = UDim2.new(0, 10, 0, yOffset + 40)
apMessageFrame.BackgroundTransparency = 1

local apMessageLabel = Instance.new("TextLabel")
apMessageLabel.Text = "   AP Message:"
apMessageLabel.TextColor3 = Color3.fromRGB(220, 220, 220)
apMessageLabel.TextSize = 14
apMessageLabel.Font = Enum.Font.GothamSemibold
apMessageLabel.TextXAlignment = Enum.TextXAlignment.Left
apMessageLabel.BackgroundTransparency = 1
apMessageLabel.Size = UDim2.new(1, 0, 0.5, 0)
apMessageLabel.Parent = apMessageFrame

local apMessageBox = Instance.new("TextBox")
apMessageBox.Text = Settings.Spam.APMessage
apMessageBox.Size = UDim2.new(1, 0, 0.5, 0)
apMessageBox.Position = UDim2.new(0, 0, 0.5, 5)
apMessageBox.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
apMessageBox.TextColor3 = Color3.fromRGB(255, 255, 255)
apMessageBox.Font = Enum.Font.Gotham
apMessageBox.Parent = apMessageFrame

apMessageBox.FocusLost:Connect(function()
    Settings.Spam.APMessage = apMessageBox.Text
end)

apMessageFrame.Parent = TabFrames["SPAM"]

-- Criar controles BRAINROT (novo!)
yOffset = 10
CreateToggle(TabFrames["BRAINROT"], "Auto-Steal Brainrot", Settings.BrainrotSteal, "AutoSteal")

-- Slider de distância de roubo
local stealDistanceFrame = Instance.new("Frame")
stealDistanceFrame.Size = UDim2.new(1, -20, 0, 50)
stealDistanceFrame.Position = UDim2.new(0, 10, 0, yOffset + 40)
stealDistanceFrame.BackgroundTransparency = 1

local stealDistanceText = Instance.new("TextLabel")
stealDistanceText.Text = "   Steal Distance: " .. Settings.BrainrotSteal.StealDistance .. " studs"
stealDistanceText.TextColor3 = Color3.fromRGB(220, 220, 220)
stealDistanceText.TextSize = 14
stealDistanceText.Font = Enum.Font.GothamSemibold
stealDistanceText.TextXAlignment = Enum.TextXAlignment.Left
stealDistanceText.BackgroundTransparency = 1
stealDistanceText.Size = UDim2.new(1, 0, 0.5, 0)
stealDistanceText.Parent = stealDistanceFrame

local stealDistanceBox = Instance.new("TextBox")
stealDistanceBox.Text = tostring(Settings.BrainrotSteal.StealDistance)
stealDistanceBox.Size = UDim2.new(0.3, 0, 0.5, 0)
stealDistanceBox.Position = UDim2.new(0.65, 0, 0.5, 0)
stealDistanceBox.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
stealDistanceBox.TextColor3 = Color3.fromRGB(255, 255, 255)
stealDistanceBox.Font = Enum.Font.Gotham
stealDistanceBox.Parent = stealDistanceFrame

stealDistanceBox.FocusLost:Connect(function()
    local newDist = tonumber(stealDistanceBox.Text)
    if newDist and newDist > 0 and newDist <= 50 then
        Settings.BrainrotSteal.StealDistance = newDist
        stealDistanceText.Text = "   Steal Distance: " .. Settings.BrainrotSteal.StealDistance .. " studs"
    else
        stealDistanceBox.Text = tostring(Settings.BrainrotSteal.StealDistance)
    end
end)

stealDistanceFrame.Parent = TabFrames["BRAINROT"]

-- Criar controles VISUALS
yOffset = 10
CreateToggle(TabFrames["VISUALS"], "X-Ray Bases", Settings.XRay, "Enabled")

-- Slider de transparência X-Ray
local xrayTransparencyFrame = Instance.new("Frame")
xrayTransparencyFrame.Size = UDim2.new(1, -20, 0, 50)
xrayTransparencyFrame.Position = UDim2.new(0, 10, 0, yOffset + 40)
xrayTransparencyFrame.BackgroundTransparency = 1

local xrayTransparencyText = Instance.new("TextLabel")
xrayTransparencyText.Text = "   X-Ray Transparency: " .. Settings.XRay.Transparency
xrayTransparencyText.TextColor3 = Color3.fromRGB(220, 220, 220)
xrayTransparencyText.TextSize = 14
xrayTransparencyText.Font = Enum.Font.GothamSemibold
xrayTransparencyText.TextXAlignment = Enum.TextXAlignment.Left
xrayTransparencyText.BackgroundTransparency = 1
xrayTransparencyText.Size = UDim2.new(1, 0, 0.5, 0)
xrayTransparencyText.Parent = xrayTransparencyFrame

local xrayTransparencyBox = Instance.new("TextBox")
xrayTransparencyBox.Text = tostring(Settings.XRay.Transparency)
xrayTransparencyBox.Size = UDim2.new(0.3, 0, 0.5, 0)
xrayTransparencyBox.Position = UDim2.new(0.65, 0, 0.5, 0)
xrayTransparencyBox.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
xrayTransparencyBox.TextColor3 = Color3.fromRGB(255, 255, 255)
xrayTransparencyBox.Font = Enum.Font.Gotham
xrayTransparencyBox.Parent = xrayTransparencyFrame

xrayTransparencyBox.FocusLost:Connect(function()
    local newTransparency = tonumber(xrayTransparencyBox.Text)
    if newTransparency and newTransparency >= 0 and newTransparency <= 1 then
        Settings.XRay.Transparency = newTransparency
        xrayTransparencyText.Text = "   X-Ray Transparency: " .. Settings.XRay.Transparency
    else
        xrayTransparencyBox.Text = tostring(Settings.XRay.Transparency)
    end
end)

xrayTransparencyFrame.Parent = TabFrames["VISUALS"]

-- Loop principal de atualização
table.insert(Connections, RunService.RenderStepped:Connect(function(deltaTime)
    -- Atualizar ESP de jogadores
    if Settings.ESP.Players then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                CreateESP(player)
            end
        end
        
        -- Limpar ESP de jogadores que saíram
        for playerName, espData in pairs(ESPCache) do
            if not Players:FindFirstChild(playerName) then
                for _, obj in pairs(espData) do
                    if obj then obj:Destroy() end
                end
                ESPCache[playerName] = nil
            end
        end
    else
        -- Limpar todo o ESP se desativado
        for _, espData in pairs(ESPCache) do
            for _, obj in pairs(espData) do
                if obj then obj:Destroy() end
            end
        end
        ESPCache = {}
    end
    
    -- Atualizar ESP Brainrot
    if Settings.ESP.Brainrot then
        BrainrotESP()
    else
        for _, obj in pairs(Workspace:GetDescendants()) do
            if obj:FindFirstChild("BrainrotESP") then
                obj.BrainrotESP:Destroy()
            end
            if obj:FindFirstChild("BrainrotLabel") then
                obj.BrainrotLabel:Destroy()
            end
        end
    end
    
    -- Atualizar X-Ray
    if Settings.XRay.Enabled then
        XRayBases()
    else
        for _, part in pairs(Workspace:GetDescendants()) do
            if part:IsA("BasePart") then
                part.LocalTransparencyModifier = 0
            end
        end
    end
    
    -- Base Protector
    if Settings.Protection.BaseProtector then
        BaseProtector()
    end
    
    -- Auto-Steal Brainrot
    if Settings.BrainrotSteal.AutoSteal then
        AutoStealBrainrot()
    end
    
    -- Spam AP (executado em thread separada se ativado)
    if Settings.Spam.AP then
        spawn(SpamAP)
    end
    
    -- Atualizar traçadores (tracers)
    if Settings.ESP.Tracers then
        for playerName, espData in pairs(ESPCache) do
            local player = Players:FindFirstChild(playerName)
            if player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local rootPart = player.Character.HumanoidRootPart
                local screenPoint, onScreen = Workspace.CurrentCamera:WorldToViewportPoint(rootPart.Position)
                
                if onScreen and espData.Tracer then
                    espData.Tracer.Visible = true
                    espData.Tracer.Position = UDim2.new(0, screenPoint.X, 0, screenPoint.Y)
                elseif espData.Tracer then
                    espData.Tracer.Visible = false
                end
            end
        end
    end
    
    -- Atualizar barras de vida
    if Settings.ESP.Health then
        for playerName, espData in pairs(ESPCache) do
            local player = Players:FindFirstChild(playerName)
            if player and player.Character and player.Character:FindFirstChild("Humanoid") and espData.HealthBar then
                local healthFill = espData.HealthBar:FindFirstChild("Frame") and espData.HealthBar.Frame:FindFirstChild("Frame")
                if healthFill then
                    healthFill.Size = UDim2.new(player.Character.Humanoid.Health/player.Character.Humanoid.MaxHealth, 0, 1, 0)
                    healthFill.BackgroundColor3 = Color3.fromRGB(
                        255 * (1 - player.Character.Humanoid.Health/player.Character.Humanoid.MaxHealth),
                        255 * (player.Character.Humanoid.Health/player.Character.Humanoid.MaxHealth),
                        0
                    )
                end
            end
        end
    end
end))

-- Bypass de anti-cheat avançado para Steal a Brainrot
local function AntiCheatBypass()
    local oldNamecall
    oldNamecall = hookmetamethod(game, "__namecall", function(self, ...)
        local method = getnamecallmethod()
        local args = {...}
        
        -- Prevenir kicks e bans
        if method == "Kick" or method == "kick" then
            return nil
        end
        
        -- Prevenir teleports indesejados (anti-afk)
        if method == "Teleport" and Settings.Teleport.CheckBaseTime then
            return wait(9e9)
        end
        
        -- Bypass de detecção de speed hack
        if method == "FindFirstChild" and tostring(self) == "Humanoid" then
            local arg1 = args[1]
            if arg1 == "WalkSpeed" or arg1 == "JumpPower" then
                return nil -- Retorna nil para evitar detecção de modificação de velocidade
            end
        end
        
        return oldNamecall(self, ...)
    end)
    
    -- Proteção contra anti-cheat que verifica propriedades do character
    local characterAddedConn
    characterAddedConn = LocalPlayer.CharacterAdded:Connect(function(char)
        wait(1) -- Esperar character carregar
        
        -- Bypass de checks de velocidade padrão do jogo
        local humanoid = char:WaitForChild("Humanoid", 5)
        if humanoid then
            humanoid.WalkSpeed = humanoid.WalkSpeed -- Reset para valor padrão se necessário
            
            -- Conectar para prevenir modificações indesejadas do jogo
            humanoid:GetPropertyChangedSignal("WalkSpeed"):Connect(function()
                if humanoid.WalkSpeed < 16 then -- Se o jogo tentar reduzir velocidade (stun/slow)
                    humanoid.WalkSpeed = Settings.Teleport.Speed / 5 -- Manter velocidade razoável mas não muito alta para evitar detecção
                end
            end)
        end
        
        -- Proteger root part contra modificações indesejadas do jogo (anti-grab/anti-stun)
        local rootPart = char:WaitForChild("HumanoidRootPart", 5)
        if rootPart then
