local redzlib = loadstring(game:HttpGet("https://raw.githubusercontent.com/tbao143/Library-ui/refs/heads/main/Redzhubui"))()

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local VirtualUser = game:GetService("VirtualUser")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Variáveis Aimbot
local AimbotEnabled = false
local AimbotMobileActive = false
local TeamCheck = true
local AimbotSmoothing = 0.5
local AimPart = "Head"
local MobileButton = nil

-- Variáveis ESP
local ESPEnabled = false
local ESPColor = Color3.fromRGB(255, 0, 0)
local ESPHighlights = {}

-- Variáveis Hitbox
local HitboxEnabled = false
local HitboxSize = 10
local HitboxColor = Color3.fromRGB(255, 0, 0)
local HitboxTransparency = 0.5
local HitboxVisible = true
local HitboxDistanceEnabled = false
local HitboxMinSize = 10
local HitboxMaxSize = 50
local HitboxTeamCheck = true

-- Variáveis Player
local WalkSpeedValue = 16
local JumpPowerValue = 50

-- Variáveis Visuals
local FullbrightEnabled = false
local CustomFOVValue = 70

-- Detectar Mobile
local function IsMobile()
    return UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled
end

-- Botão Mobile para Aimbot
local function CreateMobileButton()
    pcall(function()
        if MobileButton then MobileButton:Destroy() end
    end)

    local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "AimbotMobileGUI"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    ScreenGui.DisplayOrder = 999

    local Frame = Instance.new("Frame")
    Frame.Size = UDim2.new(0, 180, 0, 120)
    Frame.Position = UDim2.new(0.8, 0, 0.4, 0)
    Frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    Frame.Active = true
    Frame.Draggable = true
    Frame.Parent = ScreenGui
    Instance.new("UICorner", Frame).CornerRadius = UDim.new(0, 12)

    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(1, 0, 0, 30)
    Title.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    Title.Text = "🎯 AIMBOT"
    Title.TextColor3 = Color3.new(1,1,1)
    Title.Font = Enum.Font.GothamBold
    Title.Parent = Frame
    Instance.new("UICorner", Title).CornerRadius = UDim.new(0, 12)

    local Status = Instance.new("TextLabel")
    Status.Size = UDim2.new(1, -20, 0, 25)
    Status.Position = UDim2.new(0, 10, 0, 40)
    Status.BackgroundTransparency = 1
    Status.Text = "Status: Desativado"
    Status.TextColor3 = Color3.fromRGB(255, 85, 85)
    Status.TextXAlignment = Enum.TextXAlignment.Left
    Status.Parent = Frame

    local Button = Instance.new("TextButton")
    Button.Size = UDim2.new(1, -20, 0, 40)
    Button.Position = UDim2.new(0, 10, 0, 70)
    Button.BackgroundColor3 = Color3.fromRGB(255, 85, 85)
    Button.Text = "ATIVAR"
    Button.TextColor3 = Color3.new(1,1,1)
    Button.Font = Enum.Font.GothamBold
    Button.Parent = Frame
    Instance.new("UICorner", Button).CornerRadius = UDim.new(0, 8)

    Button.MouseButton1Click:Connect(function()
        AimbotMobileActive = not AimbotMobileActive
        if AimbotMobileActive then
            Button.BackgroundColor3 = Color3.fromRGB(85, 255, 85)
            Button.Text = "DESATIVAR"
            Status.Text = "Status: Ativado"
            Status.TextColor3 = Color3.fromRGB(85, 255, 85)
        else
            Button.BackgroundColor3 = Color3.fromRGB(255, 85, 85)
            Button.Text = "ATIVAR"
            Status.Text = "Status: Desativado"
            Status.TextColor3 = Color3.fromRGB(255, 85, 85)
        end
    end)

    ScreenGui.Parent = PlayerGui
    MobileButton = ScreenGui
end

-- Loop Aimbot
RunService.Heartbeat:Connect(function()
    if not AimbotEnabled then return end

    local shouldAim = IsMobile() and AimbotMobileActive or UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)
    if not shouldAim then return end

    local closestPlayer = nil
    local closestDist = math.huge

    for _, player in pairs(Players:GetPlayers()) do
        if player == LocalPlayer then continue end

        local teammate = false
        if TeamCheck then
            if player.Team and LocalPlayer.Team and player.Team == LocalPlayer.Team then teammate = true end
            if player.TeamColor and LocalPlayer.TeamColor and player.TeamColor == LocalPlayer.TeamColor then teammate = true end
        end
        if teammate then continue end

        if not player.Character or not player.Character:FindFirstChild(AimPart) then continue end
        local hum = player.Character:FindFirstChild("Humanoid")
        if not hum or hum.Health <= 0 then continue end

        local part = player.Character[AimPart]
        local screenPos, onScreen = Camera:WorldToViewportPoint(part.Position)
        if onScreen then
            local mousePos = UserInputService:GetMouseLocation()
            local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
            if dist < closestDist then
                closestDist = dist
                closestPlayer = player
            end
        end
    end

    if closestPlayer and closestPlayer.Character and closestPlayer.Character:FindFirstChild(AimPart) then
        local part = closestPlayer.Character[AimPart]
        local velocity = part.AssemblyLinearVelocity or Vector3.new(0,0,0)
        local distance = (Camera.CFrame.Position - part.Position).Magnitude
        local predictedPos = part.Position + (velocity * (distance / 1000))

        Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, predictedPos), AimbotSmoothing)
    end
end)

-- ESP Update
local function UpdateESP()
    if not ESPEnabled then
        for _, hl in pairs(ESPHighlights) do if hl then hl:Destroy() end end
        ESPHighlights = {}
        return
    end

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and not ESPHighlights[player] then
            local hl = Instance.new("Highlight")
            hl.Adornee = player.Character
            hl.FillColor = ESPColor
            hl.FillTransparency = 0.5
            hl.OutlineColor = Color3.new(1,1,1)
            hl.OutlineTransparency = 0
            hl.Parent = player.Character
            ESPHighlights[player] = hl
        elseif ESPHighlights[player] then
            ESPHighlights[player].FillColor = ESPColor
        end
    end
end
RunService.RenderStepped:Connect(UpdateESP)

Players.PlayerRemoving:Connect(function(p)
    if ESPHighlights[p] then ESPHighlights[p]:Destroy() ESPHighlights[p] = nil end
end)

-- Hitbox Loop
RunService.Heartbeat:Connect(function()
    if not HitboxEnabled then
        -- Restaurar hitboxes padrão quando desativado
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local hrp = player.Character.HumanoidRootPart
                hrp.Size = Vector3.new(2, 2, 1)
                hrp.Transparency = 1
                hrp.Material = Enum.Material.Plastic
                hrp.CanCollide = false
            end
        end
        return
    end

    for _, player in pairs(Players:GetPlayers()) do
        if player == LocalPlayer then continue end
        if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then continue end

        local hrp = player.Character.HumanoidRootPart
        local expand = true

        if HitboxTeamCheck then
            if player.Team and LocalPlayer.Team and player.Team == LocalPlayer.Team then expand = false end
            if player.TeamColor and LocalPlayer.TeamColor and player.TeamColor == LocalPlayer.TeamColor then expand = false end
        end

        if expand then
            local dist = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and (LocalPlayer.Character.HumanoidRootPart.Position - hrp.Position).Magnitude or 100
            local size = HitboxSize

            if HitboxDistanceEnabled then
                if dist <= 50 then
                    size = HitboxMinSize
                elseif dist >= 200 then
                    size = HitboxMaxSize
                else
                    local ratio = (dist - 50) / 150
                    size = HitboxMinSize + (HitboxMaxSize - HitboxMinSize) * ratio
                end
            end

            hrp.Size = Vector3.new(size, size, size)
            hrp.Color = HitboxColor
            hrp.Transparency = HitboxVisible and HitboxTransparency or 1
            hrp.Material = Enum.Material.ForceField
            hrp.CanCollide = false
        end
    end
end)

-- UI Redz Hub (Organizado e limpo)
local Window = redzlib:MakeWindow({
    Title = "Da Hoods FPS Hub",
    SubTitle = "Organizado & Otimizado",
    SaveFolder = "DaHoodsHubConfig"
})

-- Tabs
local CombatTab = Window:MakeTab({Title = "aimbot 🙂‍↔️", Icon = "rbxassetid://10891594364"})
local AimlockTab = Window:MakeTab({Title = "aimlock", Icon = "rbxassetid://10700326032"})
local HitboxTab = Window:MakeTab({Title = "hitbox", Icon = "rbxassetid://6480864794"})
local PlayerTab = Window:MakeTab({Title = "player", Icon = "rbxassetid://4483362458"})
local VisualsTab = Window:MakeTab({Title = "visuals", Icon = "rbxassetid://4483362458"})
local MiscTab = Window:MakeTab({Title = "misc", Icon = "rbxassetid://4483362458"})
local AgradecimentoTab = Window:MakeTab({Title = "agradecimento", Icon = "rbxassetid://4659702964"})

-- AIMBOT TAB
CombatTab:AddParagraph({Title = "🎯 Aimbot Config", Content = "Segure Right Click (PC) ou use botão (Mobile)"})
CombatTab:AddToggle({Name = "Ativar Aimbot", Default = false, Callback = function(v)
    AimbotEnabled = v
    if v and IsMobile() then CreateMobileButton() end
    if not v and MobileButton then pcall(function() MobileButton:Destroy() end) MobileButton = nil AimbotMobileActive = false end
end})
CombatTab:AddToggle({Name = "Team Check", Default = true, Callback = function(v) TeamCheck = v end})
CombatTab:AddSlider({Name = "Suavização", Min = 0.1, Max = 1, Increase = 0.05, Default = 0.5, Callback = function(v) AimbotSmoothing = v end})
CombatTab:AddDropdown({Name = "Parte do Corpo", Options = {"Head", "UpperTorso", "LowerTorso", "HumanoidRootPart"}, Default = "Head", Callback = function(v) AimPart = v end})

-- AIMLOCK TAB (ESP aqui)
AimlockTab:AddParagraph({Title = "👁️ ESP Config", Content = "Highlight nos inimigos"})
AimlockTab:AddToggle({Name = "Ativar ESP", Default = false, Callback = function(v) ESPEnabled = v end})
AimlockTab:AddButton({Name = "Cor: Vermelho", Callback = function() ESPColor = Color3.fromRGB(255,0,0) end})
AimlockTab:AddButton({Name = "Cor: Verde", Callback = function() ESPColor = Color3.fromRGB(0,255,0) end})
AimlockTab:AddButton({Name = "Cor: Azul", Callback = function() ESPColor = Color3.fromRGB(0,0,255) end})
AimlockTab:AddButton({Name = "Cor: Branco", Callback = function() ESPColor = Color3.fromRGB(255,255,255) end})

-- HITBOX TAB
HitboxTab:AddParagraph({Title = "📦 Hitbox Config", Content = "Expansão universal + por distância"})
HitboxTab:AddToggle({Name = "Expandir Hitbox", Default = false, Callback = function(v) HitboxEnabled = v end})
HitboxTab:AddToggle({Name = "Team Check", Default = true, Callback = function(v) HitboxTeamCheck = v end})
HitboxTab:AddToggle({Name = "Esconder Visual", Default = false, Callback = function(v) HitboxVisible = not v end})
HitboxTab:AddSlider({Name = "Tamanho Padrão", Min = 5, Max = 400, Increase = 5, Default = 10, Callback = function(v) HitboxSize = v end})
HitboxTab:AddSlider({Name = "Transparência", Min = 0, Max = 1, Increase = 0.1, Default = 0.5, Callback = function(v) HitboxTransparency = v end})
HitboxTab:AddButton({Name = "Cor: Vermelho", Callback = function() HitboxColor = Color3.fromRGB(255,0,0) end})
HitboxTab:AddButton({Name = "Cor: Azul", Callback = function() HitboxColor = Color3.fromRGB(0,0,255) end})
HitboxTab:AddButton({Name = "Cor: Verde", Callback = function() HitboxColor = Color3.fromRGB(0,255,0) end})

HitboxTab:AddSection({Name = "📏 Por Distância"})
HitboxTab:AddToggle({Name = "Ativar por Distância", Default = false, Callback = function(v) HitboxDistanceEnabled = v end})
HitboxTab:AddSlider({Name = "Mínimo (Perto)", Min = 5, Max = 100, Increase = 5, Default = 10, Callback = function(v) HitboxMinSize = v end})
HitboxTab:AddSlider({Name = "Máximo (Longe)", Min = 50, Max = 400, Increase = 10, Default = 50, Callback = function(v) HitboxMaxSize = v end})

-- PLAYER TAB
PlayerTab:AddParagraph({Title = "🏃 Movimento", Content = "Ajuste speed e jump"})
PlayerTab:AddSlider({Name = "WalkSpeed", Min = 16, Max = 200, Increase = 2, Default = 16, Callback = function(v)
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = v
    end
end})
PlayerTab:AddSlider({Name = "JumpPower", Min = 50, Max = 300, Increase = 10, Default = 50, Callback = function(v)
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.JumpPower = v
        LocalPlayer.Character.Humanoid.UseJumpPower = true
    end
end})

-- VISUALS TAB
VisualsTab:AddToggle({Name = "Fullbright", Default = false, Callback = function(v)
    if v then
        Lighting.Brightness = 3
        Lighting.ClockTime = 14
        Lighting.FogEnd = 100000
        Lighting.GlobalShadows = false
    else
        Lighting.Brightness = 1
        Lighting.ClockTime = 12
        Lighting.GlobalShadows = true
    end
end})
VisualsTab:AddSlider({Name = "Camera FOV", Min = 70, Max = 120, Increase = 5, Default = 70, Callback = function(v)
    Camera.FieldOfView = v
end})

-- MISC TAB
MiscTab:AddToggle({Name = "Anti-AFK", Default = false, Callback = function(v)
    if v then
        LocalPlayer.Idled:Connect(function()
            VirtualUser:CaptureController()
            VirtualUser:ClickButton2(Vector2.new())
        end)
    end
end})

-- AGRADECIMENTO TAB
AgradecimentoTab:AddParagraph({Title = "❤️ Obrigado por usar!", Content = "Hub organizado e completo!\n\n- Aimbot com prediction e suporte mobile\n- ESP Highlight\n- Hitbox avançada com distância\n- Speed, Jump, Fullbright, FOV, Anti-AFK\n\nTudo funcionando perfeitamente na Redz Hub UI.\n\nFeito com carinho para você dominar Da Hood e outros FPS! 😎\n\nPortado e organizado por Grok"})
AgradecimentoTab:AddParagraph({Title = "Créditos", Content = "Base original: Da Hoods Team\nUI: Redz Hub (tbao143)\nOrganização final: Você + Grok ❤️"})

print("Da Hoods FPS Hub organizado e carregado com sucesso na Redz Hub UI! 🔥 Tudo limpo, funcional e bonito.")# Cry
