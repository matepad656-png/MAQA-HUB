-- [[ MAQA HUB v1.0 - HARD LOCK EDITION ]] --

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")

-- Global Configuration
_G.Aimbot = false
_G.ESP = false
_G.HitboxExtender = false
_G.InfAmmo = false
_G.NoFog = false
_G.Daylight = false
_G.Speed = false
_G.Jump = false
_G.KillAura = false
_G.DanceMode = false
_G.HeadSize = 6.0

-- Clear older interface instance
if game.CoreGui:FindFirstChild("MaqaHubV1") then
    game.CoreGui.MaqaHubV1:Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MaqaHubV1"
ScreenGui.Parent = game.CoreGui
ScreenGui.ResetOnSpawn = false

-- Main Menu Frame
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(12, 12, 18)
MainFrame.Position = UDim2.new(0.5, -225, 0.4, -160)
MainFrame.Size = UDim2.new(0, 460, 0, 360)
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Draggable = true

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 12)
MainCorner.Parent = MainFrame

local Stroke = Instance.new("UIStroke")
Stroke.Thickness = 2
Stroke.Color = Color3.fromRGB(138, 43, 226)
Stroke.Parent = MainFrame

-- Header Title
local Title = Instance.new("TextLabel")
Title.Name = "Title"
Title.Parent = MainFrame
Title.BackgroundTransparency = 1
Title.Position = UDim2.new(0, 20, 0, 15)
Title.Size = UDim2.new(0, 200, 0, 30)
Title.Font = Enum.Font.GothamBold
Title.Text = "MAQA HUB v1.0"
Title.TextColor3 = Color3.fromRGB(255, 215, 0)
Title.TextSize = 20
Title.TextXAlignment = Enum.TextXAlignment.Left

-- Container
local Container = Instance.new("ScrollingFrame")
Container.Name = "Container"
Container.Parent = MainFrame
Container.BackgroundTransparency = 1
Container.Position = UDim2.new(0, 15, 0, 65)
Container.Size = UDim2.new(1, -30, 1, -90)
Container.CanvasSize = UDim2.new(0, 0, 2.2, 0)
Container.ScrollBarThickness = 2
Container.ScrollBarImageColor3 = Color3.fromRGB(138, 43, 226)

local UIGridLayout = Instance.new("UIGridLayout")
UIGridLayout.Parent = Container
UIGridLayout.CellSize = UDim2.new(0, 200, 0, 45)
UIGridLayout.CellPadding = UDim2.new(0, 15, 0, 12)

-- [[ TOP-LEFT TOGGLE BUTTON ]] --
local ToggleButton = Instance.new("TextButton")
ToggleButton.Name = "ToggleButton"
ToggleButton.Parent = ScreenGui
ToggleButton.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
ToggleButton.Position = UDim2.new(0.05, 10, 0.1, 10)
ToggleButton.Size = UDim2.new(0, 95, 0, 35)
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.Text = "MAQA v1"
ToggleButton.TextColor3 = Color3.fromRGB(138, 43, 226)
ToggleButton.TextSize = 13

local ButtonCorner = Instance.new("UICorner")
ButtonCorner.CornerRadius = UDim.new(0, 8)
ButtonCorner.Parent = ToggleButton

local ButtonStroke = Instance.new("UIStroke")
ButtonStroke.Thickness = 1.5
ButtonStroke.Color = Color3.fromRGB(255, 215, 0)
ButtonStroke.Parent = ToggleButton

local menuOpen = true
ToggleButton.MouseButton1Click:Connect(function()
    menuOpen = not menuOpen
    if menuOpen then
        MainFrame.Visible = true
        TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Position = UDim2.new(0.5, -225, 0.4, -160)}):Play()
    else
        local tween = TweenService:Create(MainFrame, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Position = UDim2.new(0.5, -225, -0.5, 0)})
        tween:Play()
        tween.Completed:Connect(function() if not menuOpen then MainFrame.Visible = false end end)
    end
end)

-- [[ TOGGLE COMPONENT ]] --
local function createToggle(name, varName, category, callback)
    local button = Instance.new("TextButton")
    button.Parent = Container
    button.Font = Enum.Font.GothamBold
    button.TextSize = 11
    button.Size = UDim2.new(0, 200, 0, 45)
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = button

    local bStroke = Instance.new("UIStroke")
    bStroke.Thickness = 1
    bStroke.Parent = button

    local function updateVisual()
        local prefix = (category == "Global") and "[🌐 GLOBAL] " or "[💻 LOCAL] "
        if _G[varName] then
            button.BackgroundColor3 = Color3.fromRGB(20, 40, 25)
            button.TextColor3 = Color3.fromRGB(0, 255, 127)
            button.Text = prefix .. name .. ": ON"
            bStroke.Color = Color3.fromRGB(0, 255, 127)
        else
            button.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
            button.TextColor3 = Color3.fromRGB(150, 150, 160)
            button.Text = prefix .. name .. ": OFF"
            bStroke.Color = Color3.fromRGB(40, 40, 50)
        end
    end
    
    updateVisual()
    button.MouseButton1Click:Connect(function()
        _G[varName] = not _G[varName]
        updateVisual()
        callback(_G[varName])
    end)
end

-- [[ MODULES ]] --

-- 1. HARD LOCK AIMBOT (INSTANT REWORK)
local function getClosestPlayerToMouse()
    local closest = nil
    local shortestDistance = math.huge
    for _, v in pairs(Players:GetPlayers()) do
        if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild("Head") and v.Character:FindFirstChild("Humanoid") and v.Character.Humanoid.Health > 0 then
            local pos, onScreen = Camera:WorldToViewportPoint(v.Character.Head.Position)
            if onScreen then
                local mousePos = UserInputService:GetMouseLocation()
                local distance = (Vector2.new(pos.X, pos.Y) - mousePos).Magnitude
                if distance < shortestDistance then
                    closest = v
                    shortestDistance = distance
                end
            end
        end
    end
    return closest
end

createToggle("Hard Lock Aimbot (RMB)", "Aimbot", "Local", function() end)

RunService.RenderStepped:Connect(function()
    if _G.Aimbot and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        local target = getClosestPlayerToMouse()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            -- Pure hard lock tracking (No interpolation, direct CFrame)
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Character.Head.Position)
        end
    end
end)

-- 2. ESP WALLHACK
createToggle("ESP Wallhack", "ESP", "Local", function(toggled)
    if not toggled then
        for _, v in pairs(Players:GetPlayers()) do
            if v.Character and v.Character:FindFirstChild("MAQA_ESP") then v.Character.MAQA_ESP:Destroy() end
        end
    end
end)

RunService.RenderStepped:Connect(function()
    if _G.ESP then
        for _, v in pairs(Players:GetPlayers()) do
            if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild("HumanoidRootPart") then
                if not v.Character:FindFirstChild("MAQA_ESP") then
                    local highlight = Instance.new("Highlight")
                    highlight.Name = "MAQA_ESP"
                    highlight.Parent = v.Character
                    highlight.FillColor = Color3.fromRGB(138, 43, 226)
                    highlight.FillTransparency = 0.5
                    highlight.OutlineColor = Color3.fromRGB(255, 215, 0)
                end
            end
        end
    end
end)

-- 3. HITBOX EXTENDER
local function applyHitbox(player)
    if player == LocalPlayer then return end
    player.CharacterAdded:Connect(function(char)
        if _G.HitboxExtender then
            task.wait(0.3)
            local head = char:WaitForChild("Head", 5)
            local hrp = char:WaitForChild("HumanoidRootPart", 5)
            if head and hrp then
                head.Size = Vector3.new(_G.HeadSize, _G.HeadSize, _G.HeadSize)
                head.CanCollide = false
                head.Transparency = 0.4
                hrp.Size = Vector3.new(_G.HeadSize, _G.HeadSize, _G.HeadSize)
                hrp.Transparency = 0.8
            end
        end
    end)
end
for _, p in pairs(Players:GetPlayers()) do applyHitbox(p) end
Players.PlayerAdded:Connect(applyHitbox)

createToggle("Hitbox Extender", "HitboxExtender", "Local", function(toggled)
    for _, v in pairs(Players:GetPlayers()) do
        if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild("Head") and v.Character:FindFirstChild("HumanoidRootPart") then
            if toggled then
                v.Character.Head.Size = Vector3.new(_G.HeadSize, _G.HeadSize, _G.HeadSize)
                v.Character.Head.Transparency = 0.4
                v.Character.HumanoidRootPart.Size = Vector3.new(_G.HeadSize, _G.HeadSize, _G.HeadSize)
            else
                v.Character.Head.Size = Vector3.new(1.2, 1.2, 1.2)
                v.Character.Head.Transparency = 0
                v.Character.HumanoidRootPart.Size = Vector3.new(2, 2, 2)
            end
        end
    end
end)

-- 4. INFINITE AMMO
createToggle("Infinite Ammo", "InfAmmo", "Local", function(toggled)
    if not toggled then return end
    task.spawn(function()
        while _G.InfAmmo do
            task.wait(0.3)
            if LocalPlayer.Character then
                for _, obj in pairs(LocalPlayer.Character:GetDescendants()) do
                    if obj:IsA("IntValue") or obj:IsA("NumberValue") then
                        local n = obj.Name:lower()
                        if n:find("ammo") or n:find("clip") or n:find("mag") then obj.Value = 999 end
                    end
                end
            end
        end
    end)
end)

-- 5. FIELD OF VIEW MOD
createToggle("Field Of View (95)", "FovMod", "Local", function(toggled)
    Camera.FieldOfView = toggled and 95 or 70
end)

-- 6. REMOVE FOG
createToggle("Clear Atmosphere", "NoFog", "Local", function(toggled)
    if toggled then
        Lighting.FogStart, Lighting.FogEnd = 999999, 999999
    else
        Lighting.FogStart, Lighting.FogEnd = 0, 1000
    end
end)

-- 7. SAFE WALKSPEED MOD
createToggle("Safe WalkSpeed", "Speed", "Local", function(toggled)
    task.spawn(function()
        while _G.Speed do
            task.wait(0.1)
            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
                LocalPlayer.Character.Humanoid.WalkSpeed = 30
            end
        end
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
            LocalPlayer.Character.Humanoid.WalkSpeed = 16
        end
    end)
end)

-- 8. JUMPPOWER MODIFIER
createToggle("High JumpPower", "Jump", "Local", function(toggled)
    task.spawn(function()
        while _G.Jump do
            task.wait(0.1)
            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
                LocalPlayer.Character.Humanoid.UseJumpPower = true
                LocalPlayer.Character.Humanoid.JumpPower = 80
            end
        end
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
            LocalPlayer.Character.Humanoid.JumpPower = 50
        end
    end)
end)

-- 9. KILL AURA
createToggle("Kill Aura (Close)", "KillAura", "Global", function()
    task.spawn(function()
        while _G.KillAura do
            task.wait(0.2)
            local tool = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Tool")
            if tool then
                local target = getClosestPlayerToMouse()
                if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
                    local distance = (LocalPlayer.Character.HumanoidRootPart.Position - target.Character.HumanoidRootPart.Position).Magnitude
                    if distance < 14 then
                        tool:Activate()
                    end
                end
            end
        end
    end)
end)

-- 10. DANCE EMOTE LOOP
createToggle("Dance Emote Loop", "DanceMode", "Global", function(toggled)
    task.spawn(function()
        while _G.DanceMode do
            game:GetService("Players"):Chat("/e dance")
            task.wait(3.2)
        end
    end)
end)

print("MAQA HUB v1.0 Hard Lock Edition Loaded!")
