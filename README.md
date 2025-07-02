local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/GhostDuckyy/UI-Libraries/main/Neverlose/source.lua"))()

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local Workspace = game:GetService("Workspace")
local GuiService = game:GetService("GuiService")

-- Local Player
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

-- Settings
local Settings = {
    TeamCheck = false, -- Global team check setting
    Aimbot = {
        Enabled = false,
        Locked = false,
        Target = nil,
        Keybind = Enum.KeyCode.Q,
        IgnoreKnocked = true,
        Smoothness = 10
    },
    FOV = {
        Visible = false,
        Radius = 100,
        Color = Color3.new(0, 1, 0),
        Thickness = 2,
        Sides = 50
    },
    ESP = {
        Highlight = {
            Enabled = false,
            Color = Color3.new(1, 0, 0),
            FillTransparency = 0.5,
            OutlineColor = Color3.new(1, 1, 1)
        },
        Name = {
            Enabled = false,
            Color = Color3.new(1, 1, 1),
            Size = 16,
            RefreshRate = 10 -- Seconds
        },
        Line = {
            Enabled = false,
            Color = Color3.new(1, 1, 1),
            Thickness = 1
        },
        Corner = {
            Enabled = false,
            Color = Color3.fromRGB(255, 0, 0),
            Thickness = 2
        },
        HealthBar = {
            Enabled = false,
            TeamCheck = false
        },
        Box3D = {
            Enabled = false,
            Color = Color3.fromRGB(255, 255, 255),
            Transparency = 0.25
        },
        Skeleton = {
            Enabled = false,
            Color = Color3.new(1, 1, 1),
            Thickness = 1,
            RefreshRate = 10 -- Seconds
        }
    },
    Misc = {
        Speed = {
            Enabled = false,
            Keybind = Enum.KeyCode.F,
            Active = false,
            Amount = 20,
            Original = 16
        },
        Fly = {
            Enabled = false,
            Keybind = Enum.KeyCode.G,
            Active = false,
            Speed = 50
        },
        RadarESP = {
            Enabled = false,
            Position = Vector2.new(200, 200),
            Radius = 100,
            Scale = 1,
            RadarBack = Color3.fromRGB(10, 10, 10),
            RadarBorder = Color3.fromRGB(75, 75, 75),
            LocalPlayerDot = Color3.fromRGB(255, 255, 255),
            PlayerDot = Color3.fromRGB(60, 170, 255),
            Team = Color3.fromRGB(0, 255, 0),
            Enemy = Color3.fromRGB(255, 0, 0),
            Health_Color = true,
            Team_Check = true
        }
    }
}

-- Drawing Objects
local Drawings = {
    FOVCircle = Drawing.new("Circle"),
    Lines = {},
    NameTags = {},
    Highlights = {},
    Skeletons = {},
    CornerBoxes = {},
    HealthBars = {},
    Box3DLines = {},
    Box3DQuads = {},
    Radar = {
        Background = nil,
        Border = nil,
        LocalDot = nil,
        PlayerDots = {},
        MouseDot = nil
    }
}

-- Radar ESP Functions
local function NewCircle(Transparency, Color, Radius, Filled, Thickness)
    local c = Drawing.new("Circle")
    c.Transparency = Transparency
    c.Color = Color
    c.Visible = false
    c.Thickness = Thickness
    c.Position = Vector2.new(0, 0)
    c.Radius = Radius
    c.NumSides = math.clamp(Radius*55/100, 10, 75)
    c.Filled = Filled
    return c
end

local function NewLocalDot()
    local d = Drawing.new("Triangle")
    d.Visible = true
    d.Thickness = 1
    d.Filled = true
    d.Color = Settings.Misc.RadarESP.LocalPlayerDot
    d.PointA = Settings.Misc.RadarESP.Position + Vector2.new(0, -6)
    d.PointB = Settings.Misc.RadarESP.Position + Vector2.new(-3, 6)
    d.PointC = Settings.Misc.RadarESP.Position + Vector2.new(3, 6)
    return d
end

local function GetRelative(pos)
    local char = LocalPlayer.Character
    if char ~= nil and char.PrimaryPart ~= nil then
        local pmpart = char.PrimaryPart
        local camerapos = Vector3.new(Camera.CFrame.Position.X, pmpart.Position.Y, Camera.CFrame.Position.Z)
        local newcf = CFrame.new(pmpart.Position, camerapos)
        local r = newcf:PointToObjectSpace(pos)
        return r.X, r.Z
    else
        return 0, 0
    end
end

local function PlaceDot(plr)
    local PlayerDot = NewCircle(1, Settings.Misc.RadarESP.PlayerDot, 3, true, 1)

    local function Update()
        local c 
        c = RunService.RenderStepped:Connect(function()
            if not Settings.Misc.RadarESP.Enabled then
                PlayerDot.Visible = false
                PlayerDot:Remove()
                c:Disconnect()
                return
            end
            
            local char = plr.Character
            if char and char:FindFirstChildOfClass("Humanoid") and char.PrimaryPart ~= nil and char:FindFirstChildOfClass("Humanoid").Health > 0 then
                local hum = char:FindFirstChildOfClass("Humanoid")
                local scale = Settings.Misc.RadarESP.Scale
                local relx, rely = GetRelative(char.PrimaryPart.Position)
                local newpos = Settings.Misc.RadarESP.Position - Vector2.new(relx * scale, rely * scale) 
                
                if (newpos - Settings.Misc.RadarESP.Position).magnitude < Settings.Misc.RadarESP.Radius-2 then 
                    PlayerDot.Radius = 3   
                    PlayerDot.Position = newpos
                    PlayerDot.Visible = true
                else 
                    local dist = (Settings.Misc.RadarESP.Position - newpos).magnitude
                    local calc = (Settings.Misc.RadarESP.Position - newpos).unit * (dist - Settings.Misc.RadarESP.Radius)
                    local inside = Vector2.new(newpos.X + calc.X, newpos.Y + calc.Y)
                    PlayerDot.Radius = 2
                    PlayerDot.Position = inside
                    PlayerDot.Visible = true
                end

                PlayerDot.Color = Settings.Misc.RadarESP.PlayerDot
                if Settings.Misc.RadarESP.Team_Check then
                    if plr.TeamColor == LocalPlayer.TeamColor then
                        PlayerDot.Color = Settings.Misc.RadarESP.Team
                    else
                        PlayerDot.Color = Settings.Misc.RadarESP.Enemy
                    end
                end

                if Settings.Misc.RadarESP.Health_Color then
                    local HealthBarLerp = function(t)
                        return Color3.new(1 - t, t, 0)
                    end
                    PlayerDot.Color = HealthBarLerp(hum.Health / hum.MaxHealth)
                end
            else 
                PlayerDot.Visible = false
                if Players:FindFirstChild(plr.Name) == nil then
                    PlayerDot:Remove()
                    c:Disconnect()
                end
            end
        end)
    end
    coroutine.wrap(Update)()
    return PlayerDot
end

local function InitializeRadarESP()
    -- Clear existing radar elements
    if Drawings.Radar.Background then Drawings.Radar.Background:Remove() end
    if Drawings.Radar.Border then Drawings.Radar.Border:Remove() end
    if Drawings.Radar.LocalDot then Drawings.Radar.LocalDot:Remove() end
    if Drawings.Radar.MouseDot then Drawings.Radar.MouseDot:Remove() end
    
    for _, dot in pairs(Drawings.Radar.PlayerDots) do
        if dot then dot:Remove() end
    end
    Drawings.Radar.PlayerDots = {}

    if not Settings.Misc.RadarESP.Enabled then return end

    -- Create new radar elements
    Drawings.Radar.Background = NewCircle(0.9, Settings.Misc.RadarESP.RadarBack, Settings.Misc.RadarESP.Radius, true, 1)
    Drawings.Radar.Background.Visible = true
    Drawings.Radar.Background.Position = Settings.Misc.RadarESP.Position

    Drawings.Radar.Border = NewCircle(0.75, Settings.Misc.RadarESP.RadarBorder, Settings.Misc.RadarESP.Radius, false, 3)
    Drawings.Radar.Border.Visible = true
    Drawings.Radar.Border.Position = Settings.Misc.RadarESP.Position

    Drawings.Radar.LocalDot = NewLocalDot()

    -- Create dots for existing players
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            Drawings.Radar.PlayerDots[player] = PlaceDot(player)
        end
    end

    -- Create mouse dot
    Drawings.Radar.MouseDot = NewCircle(1, Color3.fromRGB(255, 255, 255), 3, true, 1)
    Drawings.Radar.MouseDot.Visible = false

    -- Dragging functionality
    local dragging = false
    local offset = Vector2.new(0, 0)
    
    UserInputService.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 and Settings.Misc.RadarESP.Enabled then
            local mousePos = Vector2.new(Mouse.X, Mouse.Y + GuiService:GetGuiInset().Y)
            if (mousePos - Settings.Misc.RadarESP.Position).magnitude < Settings.Misc.RadarESP.Radius then
                offset = Settings.Misc.RadarESP.Position - Vector2.new(Mouse.X, Mouse.Y)
                dragging = true
            end
        end
    end)

    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)

    -- Update loop for radar
    coroutine.wrap(function()
        while Settings.Misc.RadarESP.Enabled do
            -- Update radar position if dragging
            if dragging then
                Settings.Misc.RadarESP.Position = Vector2.new(Mouse.X, Mouse.Y) + offset
            end

            -- Update radar elements
            if Drawings.Radar.Background then
                Drawings.Radar.Background.Position = Settings.Misc.RadarESP.Position
                Drawings.Radar.Background.Radius = Settings.Misc.RadarESP.Radius
                Drawings.Radar.Background.Color = Settings.Misc.RadarESP.RadarBack
            end

            if Drawings.Radar.Border then
                Drawings.Radar.Border.Position = Settings.Misc.RadarESP.Position
                Drawings.Radar.Border.Radius = Settings.Misc.RadarESP.Radius
                Drawings.Radar.Border.Color = Settings.Misc.RadarESP.RadarBorder
            end

            if Drawings.Radar.LocalDot then
                Drawings.Radar.LocalDot.Color = Settings.Misc.RadarESP.LocalPlayerDot
                Drawings.Radar.LocalDot.PointA = Settings.Misc.RadarESP.Position + Vector2.new(0, -6)
                Drawings.Radar.LocalDot.PointB = Settings.Misc.RadarESP.Position + Vector2.new(-3, 6)
                Drawings.Radar.LocalDot.PointC = Settings.Misc.RadarESP.Position + Vector2.new(3, 6)
            end

            -- Update mouse dot visibility
            if Drawings.Radar.MouseDot then
                local mousePos = Vector2.new(Mouse.X, Mouse.Y + GuiService:GetGuiInset().Y)
                if (mousePos - Settings.Misc.RadarESP.Position).magnitude < Settings.Misc.RadarESP.Radius then
                    Drawings.Radar.MouseDot.Position = mousePos
                    Drawings.Radar.MouseDot.Visible = true
                else 
                    Drawings.Radar.MouseDot.Visible = false
                end
            end

            wait()
        end
    end)()
end

-- Player added/removed connections for radar
local function SetupRadarPlayerConnections()
    Players.PlayerAdded:Connect(function(player)
        if player ~= LocalPlayer and Settings.Misc.RadarESP.Enabled then
            Drawings.Radar.PlayerDots[player] = PlaceDot(player)
            
            -- Update local dot
            if Drawings.Radar.LocalDot then
                Drawings.Radar.LocalDot:Remove()
                Drawings.Radar.LocalDot = NewLocalDot()
            end
        end
    end)

    Players.PlayerRemoving:Connect(function(player)
        if Drawings.Radar.PlayerDots[player] then
            Drawings.Radar.PlayerDots[player]:Remove()
            Drawings.Radar.PlayerDots[player] = nil
        end
    end)
end

-- Initialize FOV Circle
local function InitFOV()
    Drawings.FOVCircle.Visible = Settings.FOV.Visible
    Drawings.FOVCircle.Radius = Settings.FOV.Radius
    Drawings.FOVCircle.Color = Settings.FOV.Color
    Drawings.FOVCircle.Thickness = Settings.FOV.Thickness
    Drawings.FOVCircle.NumSides = Settings.FOV.Sides
    Drawings.FOVCircle.Filled = false
    Drawings.FOVCircle.Transparency = 0.5
    Drawings.FOVCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
end

-- Team Check Function
local function IsEnemy(player)
    if not Settings.TeamCheck then return true end
    return player.Team ~= LocalPlayer.Team
end

-- Aimbot Functions
local function FindNearestTarget()
    local nearestTarget = nil
    local nearestDistance = Settings.FOV.Radius
    local mousePos = UserInputService:GetMouseLocation()

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") and IsEnemy(player) then
            local shouldIgnore = false
            if Settings.Aimbot.IgnoreKnocked then
                local humanoid = player.Character.Humanoid
                if humanoid:GetState() == Enum.HumanoidStateType.Physics or 
                   humanoid:GetState() == Enum.HumanoidStateType.Dead or
                   (humanoid:FindFirstChild("Health") and humanoid.Health <= 0) then
                    shouldIgnore = true
                end
            end
            
            if not shouldIgnore and player.Character.Humanoid.Health > 0 then
                local head = player.Character:FindFirstChild("Head")
                if head then
                    local screenPos, onScreen = Camera:WorldToViewportPoint(head.Position)
                    if onScreen then
                        local distance = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
                        if distance < nearestDistance then
                            nearestDistance = distance
                            nearestTarget = head
                        end
                    end
                end
            end
        end
    end

    return nearestTarget
end

local function SmoothAim(targetPos)
    if not targetPos then return end
    
    local currentCFrame = Camera.CFrame
    local targetCFrame = CFrame.new(currentCFrame.Position, targetPos)
    Camera.CFrame = currentCFrame:Lerp(targetCFrame, 1/Settings.Aimbot.Smoothness)
end

-- ESP Functions
local function ClearNameESP()
    for _, tag in pairs(Drawings.NameTags) do
        if tag then 
            tag.Visible = false
            tag:Remove() 
        end
    end
    Drawings.NameTags = {}
end

local function UpdateNameESP()
    -- Clear existing name tags first
    ClearNameESP()

    if not Settings.ESP.Name.Enabled then return end

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and IsEnemy(player) then
            local head = player.Character:FindFirstChild("Head")
            if head then
                local nameTag = Drawing.new("Text")
                nameTag.Text = player.Name
                nameTag.Size = Settings.ESP.Name.Size
                nameTag.Center = true
                nameTag.Outline = true
                nameTag.Color = Settings.ESP.Name.Color
                
                local connection
                connection = RunService.RenderStepped:Connect(function()
                    if not Settings.ESP.Name.Enabled or not player.Character or not player.Character.Parent then
                        nameTag.Visible = false
                        connection:Disconnect()
                        return
                    end
                    
                    local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
                    if humanoid and humanoid.Health > 0 then
                        local headPos, onScreen = Camera:WorldToViewportPoint(head.Position)
                        if onScreen then
                            nameTag.Position = Vector2.new(headPos.X, headPos.Y - 25)
                            nameTag.Visible = true
                        else
                            nameTag.Visible = false
                        end
                    else
                        nameTag.Visible = false
                    end
                end)
                
                table.insert(Drawings.NameTags, nameTag)
            end
        end
    end
end

-- Auto-refresh Name ESP
coroutine.wrap(function()
    while true do
        if Settings.ESP.Name.Enabled then
            UpdateNameESP()
        end
        wait(Settings.ESP.Name.RefreshRate)
    end
end)()

local function UpdateLineESP()
    for _, line in pairs(Drawings.Lines) do
        if line then line:Remove() end
    end
    Drawings.Lines = {}

    if Settings.ESP.Line.Enabled then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and IsEnemy(player) then
                local root = player.Character:FindFirstChild("HumanoidRootPart")
                if root then
                    local line = Drawing.new("Line")
                    line.Thickness = Settings.ESP.Line.Thickness
                    line.Color = Settings.ESP.Line.Color
                    line.From = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y)
                    
                    RunService.RenderStepped:Connect(function()
                        if player.Character and player.Character.Parent and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
                            local rootPos, onScreen = Camera:WorldToViewportPoint(root.Position)
                            if onScreen then
                                line.To = Vector2.new(rootPos.X, rootPos.Y)
                                line.Visible = true
                            else
                                line.Visible = false
                            end
                        else
                            line.Visible = false
                        end
                    end)
                    
                    table.insert(Drawings.Lines, line)
                end
            end
        end
    end
end

local function UpdateHighlightESP()
    for _, player in pairs(Players:GetPlayers()) do
        if player.Character then
            local highlight = player.Character:FindFirstChild("HighlightESP")
            if highlight then highlight:Destroy() end
        end
    end

    if Settings.ESP.Highlight.Enabled then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and IsEnemy(player) then
                local highlight = Instance.new("Highlight")
                highlight.Name = "HighlightESP"
                highlight.Parent = player.Character
                highlight.Adornee = player.Character
                highlight.FillColor = Settings.ESP.Highlight.Color
                highlight.FillTransparency = Settings.ESP.Highlight.FillTransparency
                highlight.OutlineColor = Settings.ESP.Highlight.OutlineColor
                highlight.OutlineTransparency = 0
                Drawings.Highlights[player] = highlight
                
                player.CharacterAdded:Connect(function(character)
                    task.wait(1)
                    if Settings.ESP.Highlight.Enabled then
                        local newHighlight = Instance.new("Highlight")
                        newHighlight.Name = "HighlightESP"
                        newHighlight.Parent = character
                        newHighlight.Adornee = character
                        newHighlight.FillColor = Settings.ESP.Highlight.Color
                        newHighlight.FillTransparency = Settings.ESP.Highlight.FillTransparency
                        newHighlight.OutlineColor = Settings.ESP.Highlight.OutlineColor
                        newHighlight.OutlineTransparency = 0
                        Drawings.Highlights[player] = newHighlight
                    end
                end)
            end
        end
    end
end

local function UpdateCornerESP()
    for _, box in pairs(Drawings.CornerBoxes) do
        for _, line in pairs(box) do
            if line then line:Remove() end
        end
    end
    Drawings.CornerBoxes = {}

    if Settings.ESP.Corner.Enabled then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and IsEnemy(player) then
                local cornerBox = {
                    TL1 = Drawing.new("Line"),
                    TL2 = Drawing.new("Line"),
                    TR1 = Drawing.new("Line"),
                    TR2 = Drawing.new("Line"),
                    BL1 = Drawing.new("Line"),
                    BL2 = Drawing.new("Line"),
                    BR1 = Drawing.new("Line"),
                    BR2 = Drawing.new("Line")
                }
                
                for _, line in pairs(cornerBox) do
                    line.Visible = false
                    line.Color = Settings.ESP.Corner.Color
                    line.Thickness = Settings.ESP.Corner.Thickness
                end
                
                Drawings.CornerBoxes[player] = cornerBox
                
                local oripart = Instance.new("Part")
                oripart.Parent = workspace
                oripart.Transparency = 1
                oripart.CanCollide = false
                oripart.Anchored = true
                oripart.Size = Vector3.new(1, 1, 1)
                oripart.Position = Vector3.new(0, 0, 0)
                
                RunService.RenderStepped:Connect(function()
                    if not Settings.ESP.Corner.Enabled then
                        for _, line in pairs(cornerBox) do
                            line:Remove()
                        end
                        oripart:Destroy()
                        return
                    end
                    
                    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
                        local hrp = player.Character.HumanoidRootPart
                        local hrpPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                        
                        if onScreen then
                            oripart.Size = Vector3.new(hrp.Size.X, hrp.Size.Y * 1.5, hrp.Size.Z)
                            oripart.CFrame = CFrame.new(hrp.CFrame.Position, Camera.CFrame.Position)
                            
                            local sizeX = oripart.Size.X
                            local sizeY = oripart.Size.Y
                            
                            local tl = Camera:WorldToViewportPoint((oripart.CFrame * CFrame.new(sizeX, sizeY, 0)).p)
                            local tr = Camera:WorldToViewportPoint((oripart.CFrame * CFrame.new(-sizeX, sizeY, 0)).p)
                            local bl = Camera:WorldToViewportPoint((oripart.CFrame * CFrame.new(sizeX, -sizeY, 0)).p)
                            local br = Camera:WorldToViewportPoint((oripart.CFrame * CFrame.new(-sizeX, -sizeY, 0)).p)
                            
                            local ratio = (Camera.CFrame.p - hrp.Position).magnitude
                            local offset = math.clamp(1/ratio*750, 2, 300)
                            
                            -- Top Left
                            cornerBox.TL1.From = Vector2.new(tl.X, tl.Y)
                            cornerBox.TL1.To = Vector2.new(tl.X + offset, tl.Y)
                            cornerBox.TL2.From = Vector2.new(tl.X, tl.Y)
                            cornerBox.TL2.To = Vector2.new(tl.X, tl.Y + offset)
                            
                            -- Top Right
                            cornerBox.TR1.From = Vector2.new(tr.X, tr.Y)
                            cornerBox.TR1.To = Vector2.new(tr.X - offset, tr.Y)
                            cornerBox.TR2.From = Vector2.new(tr.X, tr.Y)
                            cornerBox.TR2.To = Vector2.new(tr.X, tr.Y + offset)
                            
                            -- Bottom Left
                            cornerBox.BL1.From = Vector2.new(bl.X, bl.Y)
                            cornerBox.BL1.To = Vector2.new(bl.X + offset, bl.Y)
                            cornerBox.BL2.From = Vector2.new(bl.X, bl.Y)
                            cornerBox.BL2.To = Vector2.new(bl.X, bl.Y - offset)
                            
                            -- Bottom Right
                            cornerBox.BR1.From = Vector2.new(br.X, br.Y)
                            cornerBox.BR1.To = Vector2.new(br.X - offset, br.Y)
                            cornerBox.BR2.From = Vector2.new(br.X, br.Y)
                            cornerBox.BR2.To = Vector2.new(br.X, br.Y - offset)
                            
                            for _, line in pairs(cornerBox) do
                                line.Visible = true
                            end
                        else
                            for _, line in pairs(cornerBox) do
                                line.Visible = false
                            end
                        end
                    else
                        for _, line in pairs(cornerBox) do
                            line.Visible = false
                        end
                    end
                end)
            end
        end
    end
end

-- Health Bar ESP
local function NewLine(thickness, color)
    local line = Drawing.new("Line")
    line.Visible = false
    line.From = Vector2.new(0, 0)
    line.To = Vector2.new(0, 0)
    line.Color = color 
    line.Thickness = thickness
    line.Transparency = 1
    return line
end

local function UpdateHealthBarESP()
    for _, healthBar in pairs(Drawings.HealthBars) do
        if healthBar.healthbar then healthBar.healthbar:Remove() end
        if healthBar.greenhealth then healthBar.greenhealth:Remove() end
    end
    Drawings.HealthBars = {}

    if not Settings.ESP.HealthBar.Enabled then return end

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and IsEnemy(player) then
            local healthBarData = {
                healthbar = NewLine(3, Color3.fromRGB(0, 0, 0)),
                greenhealth = NewLine(1.5, Color3.fromRGB(0, 255, 0))
            }
            
            Drawings.HealthBars[player] = healthBarData
            
            RunService.RenderStepped:Connect(function()
                if not Settings.ESP.HealthBar.Enabled then
                    healthBarData.healthbar:Remove()
                    healthBarData.greenhealth:Remove()
                    return
                end
                
                if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
                    local hrp = player.Character.HumanoidRootPart
                    local hrpPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                    
                    if onScreen then
                        local head = player.Character:FindFirstChild("Head")
                        if head then
                            local headPos = Camera:WorldToViewportPoint(head.Position)
                            local distanceY = math.clamp((Vector2.new(headPos.X, headPos.Y) - Vector2.new(hrpPos.X, hrpPos.Y)).magnitude, 2, math.huge)
                            
                            -- Calculate health bar positions
                            local d = (Vector2.new(hrpPos.X - distanceY, hrpPos.Y - distanceY*2) - Vector2.new(hrpPos.X - distanceY, hrpPos.Y + distanceY*2)).magnitude 
                            local healthoffset = player.Character.Humanoid.Health/player.Character.Humanoid.MaxHealth * d
                            
                            -- Update health bar positions
                            healthBarData.greenhealth.From = Vector2.new(hrpPos.X - distanceY - 4, hrpPos.Y + distanceY*2)
                            healthBarData.greenhealth.To = Vector2.new(hrpPos.X - distanceY - 4, hrpPos.Y + distanceY*2 - healthoffset)
                            
                            healthBarData.healthbar.From = Vector2.new(hrpPos.X - distanceY - 4, hrpPos.Y + distanceY*2)
                            healthBarData.healthbar.To = Vector2.new(hrpPos.X - distanceY - 4, hrpPos.Y - distanceY*2)
                            
                            -- Update health bar color based on health
                            local green = Color3.fromRGB(0, 255, 0)
                            local red = Color3.fromRGB(255, 0, 0)
                            healthBarData.greenhealth.Color = red:lerp(green, player.Character.Humanoid.Health/player.Character.Humanoid.MaxHealth)
                            
                            -- Team check
                            if Settings.ESP.HealthBar.TeamCheck then
                                if player.Team == LocalPlayer.Team then
                                    healthBarData.greenhealth.Color = Color3.fromRGB(0, 255, 0)
                                else
                                    healthBarData.greenhealth.Color = Color3.fromRGB(255, 0, 0)
                                end
                            end
                            
                            healthBarData.healthbar.Visible = true
                            healthBarData.greenhealth.Visible = true
                        else
                            healthBarData.healthbar.Visible = false
                            healthBarData.greenhealth.Visible = false
                        end
                    else
                        healthBarData.healthbar.Visible = false
                        healthBarData.greenhealth.Visible = false
                    end
                else
                    healthBarData.healthbar.Visible = false
                    healthBarData.greenhealth.Visible = false
                end
            end)
        end
    end
end

local function Clear3DBoxESP()
    for _, line in pairs(Drawings.Box3DLines) do
        if line then line:Remove() end
    end
    Drawings.Box3DLines = {}

    for _, quad in pairs(Drawings.Box3DQuads) do
        if quad then quad:Remove() end
    end
    Drawings.Box3DQuads = {}
end

local function Draw3DQuad(posA, posB, posC, posD)
    local posAScreen, posAVisible = Camera:WorldToViewportPoint(posA)
    local posBScreen, posBVisible = Camera:WorldToViewportPoint(posB)
    local posCScreen, posCVisible = Camera:WorldToViewportPoint(posC)
    local posDScreen, posDVisible = Camera:WorldToViewportPoint(posD)

    if not (posAVisible or posBVisible or posCVisible or posDVisible) then return end

    local quad = Drawing.new("Quad")
    quad.Thickness = 0.5
    quad.Color = Settings.ESP.Box3D.Color
    quad.Transparency = Settings.ESP.Box3D.Transparency
    quad.Filled = true
    quad.Visible = true

    quad.PointA = Vector2.new(posAScreen.X, posAScreen.Y)
    quad.PointB = Vector2.new(posBScreen.X, posBScreen.Y)
    quad.PointC = Vector2.new(posCScreen.X, posCScreen.Y)
    quad.PointD = Vector2.new(posDScreen.X, posDScreen.Y)

    table.insert(Drawings.Box3DQuads, quad)
end

local function Draw3DLine(from, to)
    local fromScreen, fromVisible = Camera:WorldToViewportPoint(from)
    local toScreen, toVisible = Camera:WorldToViewportPoint(to)

    if not (fromVisible or toVisible) then return end

    local line = Drawing.new("Line")
    line.Thickness = 1
    line.Color = Settings.ESP.Box3D.Color
    line.Transparency = 1
    line.Visible = true

    line.From = Vector2.new(fromScreen.X, fromScreen.Y)
    line.To = Vector2.new(toScreen.X, toScreen.Y)

    table.insert(Drawings.Box3DLines, line)
end

local function GetCorners(part)
    local cf, size, corners = part.CFrame, part.Size / 2, {}
    for x = -1, 1, 2 do 
        for y = -1, 1, 2 do 
            for z = -1, 1, 2 do
                table.insert(corners, (cf * CFrame.new(size * Vector3.new(x, y, z))).Position)
            end
        end
    end
    return corners
end

local function Update3DBoxESP()
    Clear3DBoxESP()

    if not Settings.ESP.Box3D.Enabled then return end

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and IsEnemy(player) and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            local hrp = player.Character.HumanoidRootPart
            local cubeVertices = GetCorners({
                CFrame = hrp.CFrame * CFrame.new(0, -0.5, 0),
                Size = Vector3.new(3, 5, 3)
            })

            -- Bottom face
            Draw3DLine(cubeVertices[1], cubeVertices[2])
            Draw3DLine(cubeVertices[2], cubeVertices[6])
            Draw3DLine(cubeVertices[6], cubeVertices[5])
            Draw3DLine(cubeVertices[5], cubeVertices[1])
            Draw3DQuad(cubeVertices[1], cubeVertices[2], cubeVertices[6], cubeVertices[5])

            -- Side faces
            Draw3DLine(cubeVertices[1], cubeVertices[3])
            Draw3DLine(cubeVertices[2], cubeVertices[4])
            Draw3DLine(cubeVertices[6], cubeVertices[8])
            Draw3DLine(cubeVertices[5], cubeVertices[7])

            Draw3DQuad(cubeVertices[2], cubeVertices[4], cubeVertices[8], cubeVertices[6])
            Draw3DQuad(cubeVertices[1], cubeVertices[2], cubeVertices[4], cubeVertices[3])
            Draw3DQuad(cubeVertices[1], cubeVertices[5], cubeVertices[7], cubeVertices[3])
            Draw3DQuad(cubeVertices[5], cubeVertices[7], cubeVertices[8], cubeVertices[6])

            -- Top face
            Draw3DLine(cubeVertices[3], cubeVertices[4])
            Draw3DLine(cubeVertices[4], cubeVertices[8])
            Draw3DLine(cubeVertices[8], cubeVertices[7])
            Draw3DLine(cubeVertices[7], cubeVertices[3])
            Draw3DQuad(cubeVertices[3], cubeVertices[4], cubeVertices[8], cubeVertices[7])
        end
    end
end

-- Skeleton ESP
local SkeletonLibrary = loadstring(game:HttpGet("https://raw.githubusercontent.com/Blissful4992/ESPs/main/UniversalSkeleton.lua"))()

local function UpdateSkeletonESP()
    for _, skeleton in pairs(Drawings.Skeletons) do
        if skeleton and skeleton.Remove then
            skeleton:Remove()
        end
    end
    Drawings.Skeletons = {}

    if Settings.ESP.Skeleton.Enabled then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and IsEnemy(player) then
                if player.Character and Settings.ESP.Skeleton.Enabled then
                    local skeleton = SkeletonLibrary:NewSkeleton(player, true)
                    skeleton.Color = Settings.ESP.Skeleton.Color
                    skeleton.Thickness = Settings.ESP.Skeleton.Thickness
                    Drawings.Skeletons[player] = skeleton
                end
                
                player.CharacterAdded:Connect(function(character)
                    task.wait(1)
                    if Settings.ESP.Skeleton.Enabled then
                        if Drawings.Skeletons[player] then
                            Drawings.Skeletons[player]:Remove()
                        end
                        local skeleton = SkeletonLibrary:NewSkeleton(player, true)
                        skeleton.Color = Settings.ESP.Skeleton.Color
                        skeleton.Thickness = Settings.ESP.Skeleton.Thickness
                        Drawings.Skeletons[player] = skeleton
                    end
                end)
            end
        end
    end
end

-- Auto-refresh skeleton ESP
coroutine.wrap(function()
    while true do
        if Settings.ESP.Skeleton.Enabled then
            UpdateSkeletonESP()
        end
        wait(Settings.ESP.Skeleton.RefreshRate)
    end
end)()

-- Speed Changer
local function SpeedChanger()
    if not Settings.Misc.Speed.Enabled then return end
    
    local character = LocalPlayer.Character
    if not character then 
        warn("SpeedChanger: Character not found!")
        return 
    end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then 
        warn("SpeedChanger: Humanoid not found!")
        return 
    end

    if Settings.Misc.Speed.Active then
        humanoid.WalkSpeed = Settings.Misc.Speed.Amount
        print("Speed activated:", Settings.Misc.Speed.Amount)
    else
        humanoid.WalkSpeed = Settings.Misc.Speed.Original
        print("Speed deactivated")
    end
end

-- Fly Function
local function Fly()
    if not Settings.Misc.Fly.Enabled then return end
    
    local character = LocalPlayer.Character
    if not character then 
        warn("Fly: Character not found!")
        return 
    end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then 
        warn("Fly: Humanoid not found!")
        return 
    end
    
    local rootPart = character:FindFirstChild("HumanoidRootPart") or humanoid.RootPart
    if not rootPart then 
        warn("Fly: Root part not found!")
        return 
    end
    
    -- Get or create fly parts
    local bodyVelocity = rootPart:FindFirstChild("FlyBodyVelocity")
    local bodyGyro = rootPart:FindFirstChild("FlyBodyGyro")
    
    if Settings.Misc.Fly.Active then
        if not bodyVelocity then
            bodyVelocity = Instance.new("BodyVelocity")
            bodyVelocity.Name = "FlyBodyVelocity"
            bodyVelocity.Parent = rootPart
            bodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
        end
        
        if not bodyGyro then
            bodyGyro = Instance.new("BodyGyro")
            bodyGyro.Name = "FlyBodyGyro"
            bodyGyro.P = 1000
            bodyGyro.D = 50
            bodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
            bodyGyro.Parent = rootPart
        end
        
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        bodyGyro.CFrame = Camera.CFrame
        
        -- Movement controls
        local moveDirection = Vector3.new(0, 0, 0)
        
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then
            moveDirection = moveDirection + Camera.CFrame.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then
            moveDirection = moveDirection - Camera.CFrame.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then
            moveDirection = moveDirection - Camera.CFrame.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then
            moveDirection = moveDirection + Camera.CFrame.RightVector
        end
        
        -- Apply movement
        if moveDirection.Magnitude > 0 then
            moveDirection = moveDirection.Unit * Settings.Misc.Fly.Speed
        end
        
        -- Vertical movement
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
            moveDirection = moveDirection + Vector3.new(0, Settings.Misc.Fly.Speed/2, 0)
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
            moveDirection = moveDirection - Vector3.new(0, Settings.Misc.Fly.Speed/2, 0)
        end
        
        bodyVelocity.Velocity = moveDirection
        print("Fly activated")
    else
        if bodyVelocity then bodyVelocity:Destroy() end
        if bodyGyro then bodyGyro:Destroy() end
        print("Fly deactivated")
    end
end

-- Disable Fly Function
local function DisableFly()
    local character = LocalPlayer.Character
    if not character then return end
    
    local rootPart = character:FindFirstChild("HumanoidRootPart") or character:FindFirstChildOfClass("Humanoid").RootPart
    if not rootPart then return end
    
    local bodyVelocity = rootPart:FindFirstChild("FlyBodyVelocity")
    local bodyGyro = rootPart:FindFirstChild("FlyBodyGyro")
    
    if bodyVelocity then bodyVelocity:Destroy() end
    if bodyGyro then bodyGyro:Destroy() end
end

-- Keybind Handlers
local function handleKeybinds(input, gameProcessed)
    if gameProcessed then return end
    
    -- Speed Toggle
    if input.KeyCode == Settings.Misc.Speed.Keybind and Settings.Misc.Speed.Enabled then
        Settings.Misc.Speed.Active = not Settings.Misc.Speed.Active
        print("Speed toggled:", Settings.Misc.Speed.Active)
        SpeedChanger()
    end
    
    -- Fly Toggle
    if input.KeyCode == Settings.Misc.Fly.Keybind and Settings.Misc.Fly.Enabled then
        Settings.Misc.Fly.Active = not Settings.Misc.Fly.Active
        print("Fly toggled:", Settings.Misc.Fly.Active)
        Fly()
    end
    
    -- Aimbot Lock
    if input.KeyCode == Settings.Aimbot.Keybind and Settings.Aimbot.Enabled then
        if Settings.Aimbot.Locked then
            Settings.Aimbot.Locked = false
            Settings.Aimbot.Target = nil
        else
            Settings.Aimbot.Target = FindNearestTarget()
            Settings.Aimbot.Locked = true
        end
    end
end

UserInputService.InputBegan:Connect(handleKeybinds)

-- Main Render Loop
RunService.RenderStepped:Connect(function()
    -- Update FOV Circle position
    Drawings.FOVCircle.Position = UserInputService:GetMouseLocation()
    
    -- Handle Aimbot
    if Settings.Aimbot.Enabled and Settings.Aimbot.Locked then
        if not Settings.Aimbot.Target then
            Settings.Aimbot.Target = FindNearestTarget()
        end
        
        if Settings.Aimbot.Target then
            SmoothAim(Settings.Aimbot.Target.Position)
        end
    end
    
    -- Update 3D Box ESP
    if Settings.ESP.Box3D.Enabled then
        Update3DBoxESP()
    end
    
    -- Fly (continuous update when active)
    if Settings.Misc.Fly.Enabled and Settings.Misc.Fly.Active then
        Fly()
    end
end)

-- GUI
local Window = Library:Window({
    text = "Advanced ESP"
})

local TabSection = Window:TabSection({
    text = "Features"
})

-- Main Tab
local MainTab = TabSection:Tab({
    text = "Main",
    icon = "rbxassetid://7999345313"
})

-- Aimbot Section
local AimbotSection = MainTab:Section({
    text = "Aimbot"
})

AimbotSection:Toggle({
    text = "Enable Aimbot",
    state = false,
    callback = function(state)
        Settings.Aimbot.Enabled = state
    end
})

AimbotSection:Keybind({
    text = "Lock Target",
    default = Enum.KeyCode.Q,
    callback = function(key)
        Settings.Aimbot.Keybind = key
    end
})

AimbotSection:Slider({
    text = "Smoothness",
    min = 1,
    max = 20,
    default = 10,
    callback = function(value)
        Settings.Aimbot.Smoothness = value
    end
})

-- FOV Section
local FOVSection = MainTab:Section({
    text = "FOV Circle"
})

FOVSection:Toggle({
    text = "Show FOV",
    state = false,
    callback = function(state)
        Settings.FOV.Visible = state
        Drawings.FOVCircle.Visible = state
    end
})

FOVSection:Slider({
    text = "FOV Radius",
    min = 50,
    max = 500,
    default = 100,
    callback = function(value)
        Settings.FOV.Radius = value
        Drawings.FOVCircle.Radius = value
    end
})

FOVSection:Colorpicker({
    text = "FOV Color",
    color = Color3.new(0, 1, 0),
    callback = function(color)
        Settings.FOV.Color = color
        Drawings.FOVCircle.Color = color
    end
})

-- ESP Tab
local ESPTab = TabSection:Tab({
    text = "ESP",
    icon = "rbxassetid://7999345313"
})

-- Highlight ESP
local HighlightSection = ESPTab:Section({
    text = "Highlight ESP"
})

HighlightSection:Toggle({
    text = "Enable Highlight",
    state = false,
    callback = function(state)
        Settings.ESP.Highlight.Enabled = state
        UpdateHighlightESP()
    end
})

HighlightSection:Colorpicker({
    text = "Highlight Color",
    color = Color3.new(1, 0, 0),
    callback = function(color)
        Settings.ESP.Highlight.Color = color
        UpdateHighlightESP()
    end
})

-- Name ESP
local NameSection = ESPTab:Section({
    text = "Name ESP"
})

NameSection:Toggle({
    text = "Enable Names",
    state = false,
    callback = function(state)
        Settings.ESP.Name.Enabled = state
        UpdateNameESP()
    end
})

NameSection:Colorpicker({
    text = "Name Color",
    color = Color3.new(1, 1, 1),
    callback = function(color)
        Settings.ESP.Name.Color = color
        UpdateNameESP()
    end
})

NameSection:Slider({
    text = "Refresh Rate",
    min = 1,
    max = 30,
    default = 10,
    callback = function(value)
        Settings.ESP.Name.RefreshRate = value
    end
})

-- Line ESP
local LineSection = ESPTab:Section({
    text = "Line ESP"
})

LineSection:Toggle({
    text = "Enable Lines",
    state = false,
    callback = function(state)
        Settings.ESP.Line.Enabled = state
        UpdateLineESP()
    end
})

LineSection:Colorpicker({
    text = "Line Color",
    color = Color3.new(1, 1, 1),
    callback = function(color)
        Settings.ESP.Line.Color = color
        UpdateLineESP()
    end
})

-- Box ESP Section
local BoxSection = ESPTab:Section({
    text = "Box ESP"
})

BoxSection:Toggle({
    text = "Corner Box",
    state = false,
    callback = function(state)
        Settings.ESP.Corner.Enabled = state
        UpdateCornerESP()
    end
})

BoxSection:Colorpicker({
    text = "Corner Color",
    color = Color3.fromRGB(255, 0, 0),
    callback = function(color)
        Settings.ESP.Corner.Color = color
        UpdateCornerESP()
    end
})

BoxSection:Toggle({
    text = "Health Bar",
    state = false,
    callback = function(state)
        Settings.ESP.HealthBar.Enabled = state
        UpdateHealthBarESP()
    end
})

BoxSection:Toggle({
    text = "Team Check (Health Bar)",
    state = false,
    callback = function(state)
        Settings.ESP.HealthBar.TeamCheck = state
        UpdateHealthBarESP()
    end
})

BoxSection:Toggle({
    text = "3D Box",
    state = false,
    callback = function(state)
        Settings.ESP.Box3D.Enabled = state
        if not state then
            Clear3DBoxESP()
        end
    end
})

BoxSection:Colorpicker({
    text = "3D Box Color",
    color = Color3.fromRGB(255, 255, 255),
    callback = function(color)
        Settings.ESP.Box3D.Color = color
    end
})

-- Skeleton ESP Section
local SkeletonSection = ESPTab:Section({
    text = "Skeleton ESP"
})

SkeletonSection:Toggle({
    text = "Enable Skeleton",
    state = false,
    callback = function(state)
        Settings.ESP.Skeleton.Enabled = state
        UpdateSkeletonESP()
    end
})

SkeletonSection:Colorpicker({
    text = "Skeleton Color",
    color = Color3.new(1, 1, 1),
    callback = function(color)
        Settings.ESP.Skeleton.Color = color
        UpdateSkeletonESP()
    end
})

SkeletonSection:Slider({
    text = "Refresh Rate",
    min = 1,
    max = 30,
    default = 10,
    callback = function(value)
        Settings.ESP.Skeleton.RefreshRate = value
    end
})

-- Misc Tab
local MiscTab = TabSection:Tab({
    text = "Misc",
    icon = "rbxassetid://7999345313"
})

-- Team Check Section
local TeamCheckSection = MiscTab:Section({
    text = "Team Settings"
})

TeamCheckSection:Toggle({
    text = "Team Check (All ESP)",
    state = false,
    callback = function(state)
        Settings.TeamCheck = state
        -- Update all ESP features when team check is toggled
        if Settings.ESP.Highlight.Enabled then UpdateHighlightESP() end
        if Settings.ESP.Name.Enabled then UpdateNameESP() end
        if Settings.ESP.Line.Enabled then UpdateLineESP() end
        if Settings.ESP.Corner.Enabled then UpdateCornerESP() end
        if Settings.ESP.HealthBar.Enabled then UpdateHealthBarESP() end
        if Settings.ESP.Box3D.Enabled then Update3DBoxESP() end
        if Settings.ESP.Skeleton.Enabled then UpdateSkeletonESP() end
    end
})

-- Speed Changer Section
local SpeedSection = MiscTab:Section({
    text = "Movement"
})

SpeedSection:Toggle({
    text = "Enable Speed",
    state = false,
    callback = function(state)
        Settings.Misc.Speed.Enabled = state
        if not state then
            Settings.Misc.Speed.Active = false
            SpeedChanger()
        end
    end
})

SpeedSection:Keybind({
    text = "Speed Keybind",
    default = Enum.KeyCode.F,
    callback = function(key)
        Settings.Misc.Speed.Keybind = key
        print("Speed keybind set to:", key.Name)
    end
})

SpeedSection:Slider({
    text = "Speed Amount",
    min = 16,
    max = 200,
    default = 20,
    callback = function(value)
        Settings.Misc.Speed.Amount = value
        if Settings.Misc.Speed.Active then
            SpeedChanger()
        end
    end
})

-- Fly Section
local FlySection = MiscTab:Section({
    text = "Flight"
})

FlySection:Toggle({
    text = "Enable Fly",
    state = false,
    callback = function(state)
        Settings.Misc.Fly.Enabled = state
        if not state then
            Settings.Misc.Fly.Active = false
            DisableFly()
        end
    end
})

FlySection:Keybind({
    text = "Fly Keybind",
    default = Enum.KeyCode.G,
    callback = function(key)
        Settings.Misc.Fly.Keybind = key
        print("Fly keybind set to:", key.Name)
    end
})

FlySection:Slider({
    text = "Fly Speed",
    min = 10,
    max = 200,
    default = 50,
    callback = function(value)
        Settings.Misc.Fly.Speed = value
    end
})

-- Radar ESP Section
local RadarSection = MiscTab:Section({
    text = "Radar ESP"
})

RadarSection:Toggle({
    text = "Enable Radar ESP",
    state = false,
    callback = function(state)
        Settings.Misc.RadarESP.Enabled = state
        InitializeRadarESP()
    end
})

RadarSection:Slider({
    text = "Radar Radius",
    min = 50,
    max = 300,
    default = 100,
    callback = function(value)
        Settings.Misc.RadarESP.Radius = value
    end
})

RadarSection:Slider({
    text = "Radar Scale",
    min = 0.1,
    max = 3,
    default = 1,
    callback = function(value)
        Settings.Misc.RadarESP.Scale = value
    end
})

RadarSection:Colorpicker({
    text = "Background Color",
    color = Color3.fromRGB(10, 10, 10),
    callback = function(color)
        Settings.Misc.RadarESP.RadarBack = color
    end
})

RadarSection:Colorpicker({
    text = "Border Color",
    color = Color3.fromRGB(75, 75, 75),
    callback = function(color)
        Settings.Misc.RadarESP.RadarBorder = color
    end
})

RadarSection:Colorpicker({
    text = "Player Dot Color",
    color = Color3.fromRGB(60, 170, 255),
    callback = function(color)
        Settings.Misc.RadarESP.PlayerDot = color
    end
})

RadarSection:Colorpicker({
    text = "Team Color",
    color = Color3.fromRGB(0, 255, 0),
    callback = function(color)
        Settings.Misc.RadarESP.Team = color
    end
})

RadarSection:Colorpicker({
    text = "Enemy Color",
    color = Color3.fromRGB(255, 0, 0),
    callback = function(color)
        Settings.Misc.RadarESP.Enemy = color
    end
})

RadarSection:Toggle({
    text = "Health Based Colors",
    state = true,
    callback = function(state)
        Settings.Misc.RadarESP.Health_Color = state
    end
})

RadarSection:Toggle({
    text = "Team Check",
    state = true,
    callback = function(state)
        Settings.Misc.RadarESP.Team_Check = state
    end
})

-- Credits Tab
local CreditsTab = TabSection:Tab({
    text = "Credits",
    icon = "rbxassetid://7999345313"
})

local CreditsSection = CreditsTab:Section({
    text = "Information",
    centered = true
})

CreditsSection:Label({
    text = "Advanced ESP Script",
    centered = true
})

CreditsSection:Label({
    text = "Made by ecrxks",
    centered = true
})

CreditsSection:Label({
    text = "Version 2.0",
    centered = true
})

CreditsSection:Button({
    text = "Copy Discord Invite",
    callback = function()
        setclipboard("https://discord.gg/QGeV2A3e")
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "Discord Invite",
            Text = "Copied to clipboard!",
            Duration = 3
        })
    end
})

CreditsSection:Button({
    text = "Join Discord (Browser)",
    callback = function()
        local http = game:GetService("HttpService")
        local url = "https://discord.gg/QGeV2A3e"
        if not pcall(function() http:GetAsync(url) end) then
            warn("Failed to open Discord invite in browser")
        else
            print("Opening Discord invite in browser")
            local success, err = pcall(function()
                http:RequestAsync({
                    Url = url,
                    Method = "GET"
                })
            end)
            if not success then
                warn("Failed to open browser:", err)
            end
        end
    end
})

-- Initialize
InitFOV()
SetupRadarPlayerConnections()

-- Player Connections
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        if Settings.ESP.Skeleton.Enabled then
            task.wait(1)
            UpdateSkeletonESP()
        end
        if Settings.ESP.Highlight.Enabled then
            UpdateHighlightESP()
        end
        if Settings.ESP.Corner.Enabled then
            UpdateCornerESP()
        end
        if Settings.ESP.HealthBar.Enabled then
            UpdateHealthBarESP()
        end
        if Settings.ESP.Name.Enabled then
            UpdateNameESP()
        end
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    if Drawings.Skeletons[player] then
        Drawings.Skeletons[player]:Remove()
        Drawings.Skeletons[player] = nil
    end
    if Drawings.Highlights[player] then
        Drawings.Highlights[player]:Destroy()
        Drawings.Highlights[player] = nil
    end
    if Drawings.HealthBars[player] then
        if Drawings.HealthBars[player].healthbar then Drawings.HealthBars[player].healthbar:Remove() end
        if Drawings.HealthBars[player].greenhealth then Drawings.HealthBars[player].greenhealth:Remove() end
        Drawings.HealthBars[player] = nil
    end
    if Drawings.Radar.PlayerDots[player] then
        Drawings.Radar.PlayerDots[player]:Remove()
        Drawings.Radar.PlayerDots[player] = nil
    end
end)

-- Character added event for local player
LocalPlayer.CharacterAdded:Connect(function(character)
    -- Store original walk speed
    local humanoid = character:WaitForChild("Humanoid")
    Settings.Misc.Speed.Original = humanoid.WalkSpeed
    
    -- If speed was active before respawn, re-enable it
    if Settings.Misc.Speed.Active then
        task.wait() -- Small delay to ensure everything is loaded
        SpeedChanger()
    end
    
    -- If fly was active before respawn, re-enable it
    if Settings.Misc.Fly.Active then
        task.wait()
        Fly()
    end
end)

Library:Toggle()
