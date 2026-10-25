local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local VirtualUser = game:GetService("VirtualUser")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local Lighting = game:GetService("Lighting")

local Config = {
    ESP = true, Aimbot = false, InfiniteAmmo = true, TeamCheck = true, WallCheck = true,
    HitboxExpand = true, SpeedVehicle = false, AutoClick = false, MoneyAmount = 1000000,
    TargetMoneyName = "Cash", ESPColor = Color3.fromRGB(255, 50, 50), AimbotFOV = 150,
    AimbotSmooth = 0.15, AimbotKey = Enum.UserInputType.MouseButton2, CarSpeed = 150,
    FarmCorpses = false, FarmRota = false, FarmInvisible = false
}

local espEnabled = Config.ESP
local aimbotEnabled = Config.Aimbot
local CarESPObjects = {}
local ToolsListGui = nil
local HeldCar = nil
local CurrentSpeedForce = nil
local FarmRotaConnection = nil
local FarmCorpseConnection = nil
local OriginalTransparency = {}

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "PhantomMenu"
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true
ScreenGui.DisplayOrder = 999
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local FloatBtn = Instance.new("TextButton", ScreenGui)
FloatBtn.Size = UDim2.new(0, 100, 0, 35)
FloatBtn.Position = UDim2.new(0, 20, 0, 20)
FloatBtn.BackgroundColor3 = Color3.fromRGB(15, 15, 18)
FloatBtn.Text = "PHANTOM"
FloatBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
FloatBtn.Font = Enum.Font.GothamBold
FloatBtn.TextSize = 14
FloatBtn.BorderSizePixel = 0
Instance.new("UICorner", FloatBtn).CornerRadius = UDim.new(0, 4)

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 650, 0, 450)
MainFrame.Position = UDim2.new(0.5, -325, 0.5, -225)
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 18)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 6)
Instance.new("UIStroke", MainFrame).Color = Color3.fromRGB(40, 40, 45)

local Sidebar = Instance.new("ScrollingFrame", MainFrame)
Sidebar.Size = UDim2.new(0, 160, 1, 0)
Sidebar.BackgroundColor3 = Color3.fromRGB(10, 10, 12)
Sidebar.BorderSizePixel = 0
Sidebar.ScrollBarThickness = 2
Instance.new("UICorner", Sidebar).CornerRadius = UDim.new(0, 6)

local Title = Instance.new("TextLabel", Sidebar)
Title.Size = UDim2.new(1, 0, 0, 50)
Title.BackgroundTransparency = 1
Title.Text = "PHANTOM\nMENU"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 18
Title.Font = Enum.Font.GothamBold
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Position = UDim2.new(0, 15, 0, 10)

local ProfileFrame = Instance.new("Frame", Sidebar)
ProfileFrame.Size = UDim2.new(1, -20, 0, 40)
ProfileFrame.Position = UDim2.new(0, 10, 1, -50)
ProfileFrame.BackgroundTransparency = 1

local ProfileIcon = Instance.new("ImageLabel", ProfileFrame)
ProfileIcon.Size = UDim2.new(0, 25, 0, 25)
ProfileIcon.Position = UDim2.new(0, 0, 0.5, -12.5)
ProfileIcon.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
Instance.new("UICorner", ProfileIcon).CornerRadius = UDim.new(1, 0)

local ProfileName = Instance.new("TextLabel", ProfileFrame)
ProfileName.Size = UDim2.new(1, -30, 0, 15)
ProfileName.Position = UDim2.new(0, 30, 0, 2)
ProfileName.BackgroundTransparency = 1
ProfileName.Text = LocalPlayer.Name
ProfileName.TextColor3 = Color3.fromRGB(255, 255, 255)
ProfileName.TextSize = 12
ProfileName.Font = Enum.Font.GothamBold
ProfileName.TextXAlignment = Enum.TextXAlignment.Left

local ProfileDate = Instance.new("TextLabel", ProfileFrame)
ProfileDate.Size = UDim2.new(1, -30, 0, 15)
ProfileDate.Position = UDim2.new(0, 30, 0, 17)
ProfileDate.BackgroundTransparency = 1
ProfileDate.Text = os.date("%d/%m/%Y %H:%M") 
ProfileDate.TextColor3 = Color3.fromRGB(150, 150, 150)
ProfileDate.TextSize = 10
ProfileDate.Font = Enum.Font.Gotham
ProfileDate.TextXAlignment = Enum.TextXAlignment.Left

local ContentArea = Instance.new("Frame", MainFrame)
ContentArea.Size = UDim2.new(1, -160, 1, 0)
ContentArea.Position = UDim2.new(0, 160, 0, 0)
ContentArea.BackgroundTransparency = 1

local tabButtons = {}
local tabContents = {}
local currentY = 60

local function createCategoryLabel(text)
    local label = Instance.new("TextLabel", Sidebar)
    label.Size = UDim2.new(1, -20, 0, 20)
    label.Position = UDim2.new(0, 10, 0, currentY)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = Color3.fromRGB(100, 100, 100)
    label.TextSize = 11
    label.Font = Enum.Font.GothamBold
    label.TextXAlignment = Enum.TextXAlignment.Left
    currentY = currentY + 20
end

local function createTab(name, index)
    local btn = Instance.new("TextButton", Sidebar)
    btn.Size = UDim2.new(1, -20, 0, 28)
    btn.Position = UDim2.new(0, 10, 0, currentY)
    btn.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
    btn.BackgroundTransparency = 1
    btn.BorderSizePixel = 0
    btn.Text = "  " .. name
    btn.TextColor3 = Color3.fromRGB(180, 180, 180)
    btn.TextSize = 13
    btn.Font = Enum.Font.GothamMedium
    btn.TextXAlignment = Enum.TextXAlignment.Left
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
    
    currentY = currentY + 28
    Sidebar.CanvasSize = UDim2.new(0, 0, 0, currentY + 60)

    local page = Instance.new("Frame", ContentArea)
    page.Size = UDim2.new(1, 0, 1, 0)
    page.BackgroundTransparency = 1
    page.Visible = false
    
    tabButtons[name] = btn
    tabContents[name] = page

    btn.MouseButton1Click:Connect(function()
        for n, b in pairs(tabButtons) do
            b.BackgroundTransparency = 1
            b.TextColor3 = Color3.fromRGB(180, 180, 180)
            tabContents[n].Visible = false
        end
        btn.BackgroundTransparency = 0
        btn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        btn.TextColor3 = Color3.fromRGB(0, 0, 0)
        page.Visible = true
    end)

    if index == 0 then
        btn.BackgroundTransparency = 0
        btn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        btn.TextColor3 = Color3.fromRGB(0, 0, 0)
        page.Visible = true
    end
end

local function createToggle(parent, text, yPos, initialState, callback)
    local frame = Instance.new("Frame", parent)
    frame.Size = UDim2.new(0, 420, 0, 35)
    frame.Position = UDim2.new(0, 20, 0, yPos)
    frame.BackgroundTransparency = 1
    
    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(1, -40, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = Color3.fromRGB(220, 220, 220)
    label.TextSize = 13
    label.Font = Enum.Font.Gotham
    label.TextXAlignment = Enum.TextXAlignment.Left
    
    local toggleBtn = Instance.new("TextButton", frame)
    toggleBtn.Size = UDim2.new(0, 30, 0, 16)
    toggleBtn.Position = UDim2.new(1, -30, 0.5, -8)
    toggleBtn.BackgroundColor3 = initialState and Color3.fromRGB(0, 150, 255) or Color3.fromRGB(50, 50, 60)
    toggleBtn.Text = ""
    toggleBtn.BorderSizePixel = 0
    Instance.new("UICorner", toggleBtn).CornerRadius = UDim.new(0, 8)
    
    local circle = Instance.new("Frame", toggleBtn)
    circle.Size = UDim2.new(0, 12, 0, 12)
    circle.Position = initialState and UDim2.new(1, -14, 0, 2) or UDim2.new(0, 2, 0, 2)
    circle.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    Instance.new("UICorner", circle).CornerRadius = UDim.new(1, 0)
    
    local state = initialState
    toggleBtn.MouseButton1Click:Connect(function()
        state = not state
        toggleBtn.BackgroundColor3 = state and Color3.fromRGB(0, 150, 255) or Color3.fromRGB(50, 50, 60)
        circle.Position = state and UDim2.new(1, -14, 0, 2) or UDim2.new(0, 2, 0, 2)
        if callback then callback(state) end
    end)
end

local function createSlider(parent, text, yPos, min, max, default, callback)
    local frame = Instance.new("Frame", parent)
    frame.Size = UDim2.new(0, 420, 0, 50)
    frame.Position = UDim2.new(0, 20, 0, yPos)
    frame.BackgroundTransparency = 1
    
    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(1, -80, 0, 20)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = Color3.fromRGB(220, 220, 220)
    label.TextSize = 13
    label.Font = Enum.Font.Gotham
    label.TextXAlignment = Enum.TextXAlignment.Left
    
    local valueText = Instance.new("TextLabel", frame)
    valueText.Size = UDim2.new(0, 50, 0, 20)
    valueText.Position = UDim2.new(1, -50, 0, 0)
    valueText.BackgroundTransparency = 1
    valueText.Text = tostring(default)
    valueText.TextColor3 = Color3.fromRGB(0, 150, 255)
    valueText.TextSize = 13
    valueText.Font = Enum.Font.GothamBold
    valueText.TextXAlignment = Enum.TextXAlignment.Right

    local sliderBar = Instance.new("Frame", frame)
    sliderBar.Size = UDim2.new(1, -40, 0, 6)
    sliderBar.Position = UDim2.new(0, 0, 0, 28)
    sliderBar.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
    sliderBar.BorderSizePixel = 0
    Instance.new("UICorner", sliderBar).CornerRadius = UDim.new(0, 3)

    local fill = Instance.new("Frame", sliderBar)
    fill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)
    fill.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
    fill.BorderSizePixel = 0

    local dragBtn = Instance.new("TextButton", sliderBar)
    dragBtn.Size = UDim2.new(0, 14, 0, 14)
    dragBtn.Position = UDim2.new((default - min) / (max - min), -7, 0.5, -7)
    dragBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    dragBtn.Text = ""
    dragBtn.BorderSizePixel = 0
    Instance.new("UICorner", dragBtn).CornerRadius = UDim.new(1, 0)

    local dragging = false
    dragBtn.MouseButton1Down:Connect(function() dragging = true end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then dragging = false end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local mousePos = UserInputService:GetMouseLocation()
            local barPos = sliderBar.AbsolutePosition
            local barSize = sliderBar.AbsoluteSize
            local percent = math.clamp((mousePos.X - barPos.X) / barSize.X, 0, 1)
            fill.Size = UDim2.new(percent, 0, 1, 0)
            dragBtn.Position = UDim2.new(percent, -7, 0.5, -7)
            local val = math.floor(min + (max - min) * percent)
            valueText.Text = tostring(val)
            if callback then callback(val) end
        end
    end)
end

local function createButton(parent, text, yPos, callback)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(0, 420, 0, 35)
    btn.Position = UDim2.new(0, 20, 0, yPos)
    btn.BackgroundColor3 = Color3.fromRGB(30, 30, 38)
    btn.BorderSizePixel = 0
    btn.Text = "  " .. text
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextSize = 13
    btn.Font = Enum.Font.GothamMedium
    btn.TextXAlignment = Enum.TextXAlignment.Left
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
    btn.MouseButton1Click:Connect(callback)
end

local function createTextBox(parent, placeholder, yPos, callback)
    local box = Instance.new("TextBox", parent)
    box.Size = UDim2.new(0, 420, 0, 35)
    box.Position = UDim2.new(0, 20, 0, yPos)
    box.BackgroundColor3 = Color3.fromRGB(30, 30, 38)
    box.BorderSizePixel = 0
    box.Text = ""
    box.PlaceholderText = placeholder
    box.TextColor3 = Color3.fromRGB(255, 255, 255)
    box.PlaceholderColor3 = Color3.fromRGB(150, 150, 150)
    box.TextSize = 13
    box.Font = Enum.Font.GothamMedium
    box.TextXAlignment = Enum.TextXAlignment.Left
    Instance.new("UICorner", box).CornerRadius = UDim.new(0, 4)
    box.FocusLost:Connect(function()
        if callback then callback(box.Text) end
    end)
end
createCategoryLabel("Combate")
createTab("Jogador", 0)
createTab("Armas", 1)
createCategoryLabel("Entidades")
createTab("Veiculos", 2)
createTab("Jogadores", 3)
createCategoryLabel("Visual")
createTab("Mundo", 4)
createCategoryLabel("Outros")
createTab("Exploits", 5)
createTab("Farm", 6)
createTab("Teleporte", 7)
createTab("Creditos", 8) 

createToggle(tabContents["Jogador"], "Ativar Aimbot", 20, aimbotEnabled, function(s) aimbotEnabled = s end)
createToggle(tabContents["Jogador"], "Team Check", 60, Config.TeamCheck, function(s) Config.TeamCheck = s end)
createToggle(tabContents["Jogador"], "Wall Check", 100, Config.WallCheck, function(s) Config.WallCheck = s end)
createToggle(tabContents["Jogador"], "Expandir Hitbox", 140, Config.HitboxExpand, function(s) Config.HitboxExpand = s end)
createSlider(tabContents["Jogador"], "FOV", 180, 10, 500, Config.AimbotFOV, function(v) Config.AimbotFOV = v end)

createToggle(tabContents["Jogadores"], "Enable ESP Box", 20, espEnabled, function(s) espEnabled = s end)

createToggle(tabContents["Armas"], "Munição Infinita", 20, Config.InfiniteAmmo, function(s) Config.InfiniteAmmo = s end)
createButton(tabContents["Armas"], "Puxar Armas (Tools)", 60, function()
    local count = 0
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("Tool") and LocalPlayer.Character then
            if v.Parent ~= LocalPlayer.Character and not v.Parent:IsA("Player") and not v.Parent:IsA("Backpack") then
                v.Parent = LocalPlayer.Character
                count = count + 1
            end
        end
    end
end)

createToggle(tabContents["Veiculos"], "Carro Ultra Rápido", 20, Config.SpeedVehicle, function(s) Config.SpeedVehicle = s end)
createToggle(tabContents["Veiculos"], "Ativar ESP Car", 60, false, function(s)
    Config.CarESP = s
    if not s then
        for _, v in pairs(CarESPObjects) do v:Destroy() end
        CarESPObjects = {}
    end
end)
createButton(tabContents["Veiculos"], "Jogar Carros (Aperte Y)", 100, function() print("Olhe e aperte Y") end)
createButton(tabContents["Veiculos"], "Lista de Carros (Abrir/Fechar)", 140, function()
    if ToolsListGui then ToolsListGui:Destroy() end
    ToolsListGui = Instance.new("ScrollingFrame", ScreenGui)
    ToolsListGui.Size = UDim2.new(0, 300, 0, 400)
    ToolsListGui.Position = UDim2.new(0.5, -150, 0.5, -200)
    ToolsListGui.BackgroundColor3 = Color3.fromRGB(15,15,18)
    ToolsListGui.BorderSizePixel = 0
    ToolsListGui.Active = true
    ToolsListGui.Draggable = true
    
    local closeBtn = Instance.new("TextButton", ToolsListGui)
    closeBtn.Size = UDim2.new(0, 30, 0, 30)
    closeBtn.Position = UDim2.new(1, -35, 0, 5)
    closeBtn.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    closeBtn.Text = "X"
    closeBtn.TextColor3 = Color3.new(1,1,1)
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.MouseButton1Click:Connect(function() ToolsListGui:Destroy() end)
    
    local count = 1
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("VehicleSeat") or v.Name:lower():find("car") or v.Name:lower():find("vehicle") then
            local carModel = v:FindFirstAncestorOfClass("Model") or v
            local label = Instance.new("TextLabel", ToolsListGui)
            label.Size = UDim2.new(1, -10, 0, 30)
            label.Position = UDim2.new(0, 5, 0, (count-1)*30 + 40)
            label.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
            label.Text = " " .. carModel.Name
            label.TextColor3 = Color3.new(1,1,1)
            label.TextXAlignment = Enum.TextXAlignment.Left
            count = count + 1
        end
    end
    ToolsListGui.CanvasSize = UDim2.new(0, 0, 0, count * 30 + 50)
end)

createButton(tabContents["Mundo"], "Deixar de Dia", 20, function()
    Lighting.ClockTime = 14
    Lighting.OutdoorAmbient = Color3.fromRGB(128, 128, 128)
    Lighting.Brightness = 2
end)
createButton(tabContents["Mundo"], "Deixar Pôr do Sol", 60, function()
    Lighting.ClockTime = 18
    Lighting.OutdoorAmbient = Color3.fromRGB(255, 150, 100)
    Lighting.Brightness = 1.5
end)
createButton(tabContents["Mundo"], "Deixar de Noite", 100, function()
    Lighting.ClockTime = 0
    Lighting.OutdoorAmbient = Color3.fromRGB(40, 40, 60)
    Lighting.Brightness = 0.5
end)

createTextBox(tabContents["Exploits"], "Nome da Variável de Dinheiro (Ex: Cash)", 20, function(text) Config.TargetMoneyName = text end)
createButton(tabContents["Exploits"], "Puxar Dinheiro (Setar Valor)", 60, function()
    if not LocalPlayer:FindFirstChild("leaderstats") then return end
    local target = LocalPlayer.leaderstats:FindFirstChild(Config.TargetMoneyName)
    if target and (target:IsA("IntValue") or target:IsA("NumberValue")) then
        target.Value = Config.MoneyAmount
    end
end)
createButton(tabContents["Exploits"], "Clonar Lixo Próximo", 100, function()
    local count = 0
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("Model") and (v.Name:lower():find("saco") or v.Name:lower():find("lixo")) then
            local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                local clone = v:Clone()
                clone.Parent = workspace
                if clone.PrimaryPart then
                    clone:SetPrimaryPartCFrame(hrp.CFrame * CFrame.new(math.random(-5,5), 0, math.random(-5,5)))
                else
                    clone:MoveTo(hrp.Position + Vector3.new(math.random(-5,5), 0, math.random(-5,5)))
                end
                count = count + 1
            end
        end
    end
end)
createButton(tabContents["Exploits"], "Clonar Ação (Auto Pegar)", 140, function()
    task.spawn(function()
        for i = 1, 20 do
            VirtualUser:CaptureController()
            VirtualUser:ClickButton1(Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2))
            task.wait(0.1)
        end
    end)
end)
createToggle(tabContents["Exploits"], "Multi Click", 180, Config.AutoClick, function(s) Config.AutoClick = s end)
createSlider(tabContents["Exploits"], "Velocidade Click", 220, 1, 100, 50, function(v) Config.AutoClickSpeed = 1 / v end)

-- ================= FARM =================
createToggle(tabContents["Farm"], "Ativar Invisibilidade", 20, Config.FarmInvisible, function(s)
    Config.FarmInvisible = s
    if LocalPlayer.Character then
        for _, v in ipairs(LocalPlayer.Character:GetDescendants()) do
            if v:IsA("BasePart") and v.Name ~= "HumanoidRootPart" then
                if s then
                    OriginalTransparency[v] = v.Transparency
                    v.Transparency = 1
                else
                    if OriginalTransparency[v] then
                        v.Transparency = OriginalTransparency[v]
                    end
                end
            end
        end
    end
end)

createToggle(tabContents["Farm"], "Farm de Corpos (Auto Revistar)", 60, Config.FarmCorpses, function(s)
    Config.FarmCorpses = s
    if s then
        FarmCorpseConnection = RunService.Heartbeat:Connect(function()
            for _, player in ipairs(Players:GetPlayers()) do
                if player ~= LocalPlayer and player.Character then
                    local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
                    if humanoid and humanoid.Health <= 0 and player.Character:FindFirstChild("HumanoidRootPart") then
                        local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                        if hrp then
                            local targetPos = player.Character.HumanoidRootPart.Position
                            hrp.CFrame = CFrame.new(targetPos + Vector3.new(0, 3, 0))
                            task.wait(0.1)
                            VirtualUser:CaptureController()
                            VirtualUser:ClickButton1(Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2))
                        end
                    end
                end
            end
        end)
    else
        if FarmCorpseConnection then FarmCorpseConnection:Disconnect() end
    end
end)

createToggle(tabContents["Farm"], "Auto Rota (Peças de Arma)", 100, Config.FarmRota, function(s)
    Config.FarmRota = s
    if s then
        FarmRotaConnection = RunService.Heartbeat:Connect(function()
            if not Config.FarmRota then return end
            
            local rootPart = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            if not rootPart then return end
            
            local searchTerms = {"peça", "rota", "arma", "seta", "interact", "interagir"}
            local closestObj = nil
            local shortestDist = 5000
            
            for _, v in ipairs(workspace:GetDescendants()) do
                if v:IsA("Model") or v:IsA("BasePart") or v:IsA("TextLabel") then
                    local name = v.Name:lower()
                    local matches = false
                    for _, term in ipairs(searchTerms) do
                        if name:find(term) then matches = true; break end
                    end
                    
                    if matches then
                        local objPos
                        if v:IsA("Model") and v.PrimaryPart then objPos = v.PrimaryPart.Position
                        elseif v:IsA("BasePart") then objPos = v.Position
                        elseif v:IsA("TextLabel") then objPos = v.Parent.Position
                        end
                        
                        if objPos then
                            local dist = (rootPart.Position - objPos).Magnitude
                            if dist < shortestDist then
                                shortestDist = dist
                                closestObj = objPos
                            end
                        end
                    end
                end
            end
            
            if closestObj then
                rootPart.CFrame = CFrame.new(closestObj + Vector3.new(0, 3, 0))
                task.wait(0.1)
                VirtualUser:CaptureController()
                VirtualUser:ClickButton1(Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2))
            end
        end)
    else
        if FarmRotaConnection then FarmRotaConnection:Disconnect() end
    end
end)

-- TELEPORTE
local function TeleportTo(placeName)
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("Model") or v:IsA("BasePart") then
            if v.Name:lower():find(placeName:lower()) then
                if (v:IsA("Model") and v.PrimaryPart) or v:IsA("BasePart") then
                    local char = LocalPlayer.Character
                    if char and char:FindFirstChild("HumanoidRootPart") then
                        local targetPos = v:IsA("Model") and v.PrimaryPart.Position or v.Position
                        char.HumanoidRootPart.CFrame = CFrame.new(targetPos + Vector3.new(0, 5, 0))
                        break
                    end
                end
            end
        end
    end
end
createButton(tabContents["Teleporte"], "Ir para a Praça", 20, function() TeleportTo("praca") end)
createButton(tabContents["Teleporte"], "Ir para o Hospital", 60, function() TeleportTo("hospital") end)
createButton(tabContents["Teleporte"], "Ir para a Prefeitura", 100, function() TeleportTo("prefeitura") end)

local creditFrame = Instance.new("Frame", tabContents["Creditos"])
creditFrame.Size = UDim2.new(1, -40, 0, 100)
creditFrame.Position = UDim2.new(0, 20, 0, 40)
creditFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
Instance.new("UICorner", creditFrame).CornerRadius = UDim.new(0, 6)

local creditTitle = Instance.new("TextLabel", creditFrame)
creditTitle.Size = UDim2.new(1, 0, 0, 30)
creditTitle.BackgroundTransparency = 1
creditTitle.Text = "Menu Desenvolvido Por:"
creditTitle.TextColor3 = Color3.fromRGB(150, 150, 150)
creditTitle.Font = Enum.Font.Gotham

local creditName = Instance.new("TextLabel", creditFrame)
creditName.Size = UDim2.new(1, 0, 0, 40)
creditName.Position = UDim2.new(0, 0, 0, 30)
creditName.BackgroundTransparency = 1
creditName.Text = "soy_tristeconta2" 
creditName.TextColor3 = Color3.fromRGB(0, 150, 255) 
creditName.TextSize = 22
creditName.Font = Enum.Font.GothamBold

local creditDesc = Instance.new("TextLabel", creditFrame)
creditDesc.Size = UDim2.new(1, 0, 0, 30)
creditDesc.Position = UDim2.new(0, 0, 0, 70)
creditDesc.BackgroundTransparency = 1
creditDesc.Text = "Obrigado por usar! 👑"
creditDesc.TextColor3 = Color3.fromRGB(200, 200, 200)
creditDesc.Font = Enum.Font.GothamMedium
local espObjects = {}
local function createESP(player)
    if player == LocalPlayer then return end
    local box = Drawing.new("Square"); box.Thickness = 1; box.Color = Config.ESPColor; box.Filled = false; box.Visible = false
    local nameTag = Drawing.new("Text"); nameTag.Size = 14; nameTag.Center = true; nameTag.Outline = true; nameTag.Color = Color3.fromRGB(255, 255, 255); nameTag.Visible = false
    espObjects[player] = { box = box, nameTag = nameTag }
end
for _, p in ipairs(Players:GetPlayers()) do createESP(p) end
Players.PlayerAdded:Connect(createESP)

local function hasWall(origin, target)
    local direction = (target.Position - origin.Position).Unit * (target.Position - origin.Position).Magnitude
    local rayParams = RaycastParams.new()
    rayParams.FilterDescendantsInstances = {LocalPlayer.Character, target.Parent}
    rayParams.FilterType = Enum.RaycastFilterType.Exclude
    return workspace:Raycast(origin.Position, direction, rayParams) ~= nil
end

task.spawn(function()
    while task.wait(1) do
        if Config.HitboxExpand then
            for _, p in ipairs(Players:GetPlayers()) do
                if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                    local hrp = p.Character.HumanoidRootPart
                    hrp.Size = Vector3.new(15, 15, 15); hrp.Transparency = 0.5; hrp.CanCollide = false
                end
            end
        else
            for _, p in ipairs(Players:GetPlayers()) do
                if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                    local hrp = p.Character.HumanoidRootPart
                    hrp.Size = Vector3.new(2, 2, 1); hrp.Transparency = 1; hrp.CanCollide = true
                end
            end
        end
    end
end)

RunService.RenderStepped:Connect(function()
    for player, obj in pairs(espObjects) do
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and espEnabled then
            local hrp = player.Character.HumanoidRootPart
            local top = Camera:WorldToViewportPoint(hrp.Position + Vector3.new(0, 3, 0))
            local bottom = Camera:WorldToViewportPoint(hrp.Position - Vector3.new(0, 3, 0))
            local height = math.abs(top.Y - bottom.Y)
            if top.Z > 0 then
                obj.box.Size = Vector2.new(height * 0.5, height)
                obj.box.Position = Vector2.new(top.X - (height * 0.25), top.Y)
                obj.box.Visible = true
                obj.nameTag.Text = player.Name
                obj.nameTag.Position = Vector2.new(top.X, top.Y - 18)
                obj.nameTag.Visible = true
            else
                obj.box.Visible = false; obj.nameTag.Visible = false
            end
        else
            obj.box.Visible = false; obj.nameTag.Visible = false
        end
    end

    if Config.CarESP then
        for _, v in pairs(workspace:GetDescendants()) do
            if v:IsA("VehicleSeat") then
                local carModel = v:FindFirstAncestorOfClass("Model")
                if carModel and not CarESPObjects[carModel] then
                    local highlight = Instance.new("Highlight")
                    highlight.FillColor = Color3.fromRGB(0, 255, 255)
                    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                    highlight.Parent = carModel
                    CarESPObjects[carModel] = highlight
                end
            end
        end
    end

    if aimbotEnabled and UserInputService:IsMouseButtonPressed(Config.AimbotKey) then
        local closest, shortest = nil, Config.AimbotFOV
        local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local hrp = player.Character.HumanoidRootPart
                local screenPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                if onScreen then
                    if not Config.WallCheck or (Config.WallCheck and not hasWall(Camera.CFrame.Position, hrp)) then
                        local dist = (Vector2.new(screenPos.X, screenPos.Y) - center).Magnitude
                        if dist < shortest then shortest = dist; closest = hrp end
                    end
                end
            end
        end
        if closest then
            local targetPos = Camera:WorldToViewportPoint(closest.Position)
            local delta = (Vector2.new(targetPos.X, targetPos.Y) - center) * Config.AimbotSmooth
            mousemoverel(delta.X, delta.Y)
        end
    end
    
    if Config.SpeedVehicle and LocalPlayer.Character then
        local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if humanoid and humanoid.SeatPart then
            local vehicleModel = humanoid.SeatPart:FindFirstAncestorOfClass("Model") or humanoid.SeatPart.Parent
            if vehicleModel then
                if not vehicleModel:FindFirstChild("PhantomSpeedForce") then
                    local force = Instance.new("VectorForce")
                    force.Name = "PhantomSpeedForce"
                    force.Attachment0 = humanoid.SeatPart:FindFirstChildWhichIsA("Attachment") or Instance.new("Attachment", humanoid.SeatPart)
                    force.ApplyAtCenterOfMass = true
                    force.RelativeTo = Enum.ActuatorRelativeTo.World
                    force.Parent = humanoid.SeatPart
                    CurrentSpeedForce = force
                end
                if CurrentSpeedForce then CurrentSpeedForce.Force = humanoid.SeatPart.CFrame.LookVector * (Config.CarSpeed * 1000) end
            end
        end
    end
end)

task.spawn(function()
    while task.wait(Config.AutoClickSpeed) do
        if Config.AutoClick then
            VirtualUser:CaptureController()
            VirtualUser:ClickButton1(Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2))
        end
    end
end)

UserInputService.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.KeyCode == Enum.KeyCode.K then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

FloatBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)
