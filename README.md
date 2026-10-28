-- Onyx Rivals Script
-- Aimbot + Hitbox Expander + ESP + Mobile UI
-- Delta / Fluxus / Wave 등에서 실행

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local VirtualUser = game:GetService("VirtualUser")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Settings
getgenv().Onyx = {
    Aimbot = false,
    SilentAim = false,
    Ragebot = false,
    Hitbox = false,
    HitboxSize = 8,
    ESP = true,
    Tracer = false,
    NameESP = true,
    InfJump = false,
    Speed = false,
    SpeedValue = 28,
    AimFOV = 150,
    Smooth = 0.12,
    TeamCheck = false,
}

-- Anti AFK + basic protect
pcall(function()
    LocalPlayer.Idled:Connect(function()
        VirtualUser:CaptureController()
        VirtualUser:ClickButton2(Vector2.new())
    end)
end)

-- ===================== UI =====================
local SG = Instance.new("ScreenGui")
SG.Name = "OnyxRivals"
SG.ResetOnSpawn = false
SG.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
SG.IgnoreGuiInset = true
pcall(function() SG.Parent = game.CoreGui end)
if not SG.Parent then SG.Parent = LocalPlayer:WaitForChild("PlayerGui") end

local Main = Instance.new("Frame")
Main.Size = UDim2.new(0, 290, 0, 400)
Main.Position = UDim2.new(0.05, 0, 0.2, 0)
Main.BackgroundColor3 = Color3.fromRGB(12, 12, 16)
Main.BorderSizePixel = 0
Main.Active = true
Main.Parent = SG

Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)

local Top = Instance.new("Frame")
Top.Size = UDim2.new(1, 0, 0, 38)
Top.BackgroundColor3 = Color3.fromRGB(22, 22, 28)
Top.BorderSizePixel = 0
Top.Parent = Main
Instance.new("UICorner", Top).CornerRadius = UDim.new(0, 10)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -40, 1, 0)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "ONYX RIVALS"
Title.TextColor3 = Color3.fromRGB(255, 45, 45)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = Top

local Close = Instance.new("TextButton")
Close.Size = UDim2.new(0, 32, 0, 32)
Close.Position = UDim2.new(1, -35, 0.5, -16)
Close.BackgroundColor3 = Color3.fromRGB(40, 40, 48)
Close.Text = "X"
Close.TextColor3 = Color3.fromRGB(255, 255, 255)
Close.Font = Enum.Font.GothamBold
Close.TextSize = 14
Close.Parent = Top
Instance.new("UICorner", Close).CornerRadius = UDim.new(0, 6)
Close.MouseButton1Click:Connect(function() SG:Destroy() end)

-- Drag
local drag, start, startPos, inputObj
Top.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        drag = true
        start = input.Position
        startPos = Main.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then drag = false end
        end)
    end
end)
Top.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        inputObj = input
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if input == inputObj and drag then
        local delta = input.Position - start
        Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

local Scroll = Instance.new("ScrollingFrame")
Scroll.Size = UDim2.new(1, -12, 1, -48)
Scroll.Position = UDim2.new(0, 6, 0, 42)
Scroll.BackgroundTransparency = 1
Scroll.ScrollBarThickness = 3
Scroll.CanvasSize = UDim2.new(0, 0, 0, 720)
Scroll.Parent = Main

local Layout = Instance.new("UIListLayout")
Layout.Padding = UDim.new(0, 6)
Layout.Parent = Scroll

local function Toggle(name, default, cb)
    local f = Instance.new("Frame")
    f.Size = UDim2.new(1, 0, 0, 36)
    f.BackgroundColor3 = Color3.fromRGB(24, 24, 30)
    f.BorderSizePixel = 0
    f.Parent = Scroll
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 7)

    local l = Instance.new("TextLabel")
    l.Size = UDim2.new(0.62, 0, 1, 0)
    l.Position = UDim2.new(0, 10, 0, 0)
    l.BackgroundTransparency = 1
    l.Text = name
    l.TextColor3 = Color3.fromRGB(225, 225, 225)
    l.Font = Enum.Font.Gotham
    l.TextSize = 14
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.Parent = f

    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0, 64, 0, 26)
    b.Position = UDim2.new(1, -74, 0.5, -13)
    b.BackgroundColor3 = default and Color3.fromRGB(255, 45, 45) or Color3.fromRGB(50, 50, 58)
    b.Text = default and "ON" or "OFF"
    b.TextColor3 = Color3.fromRGB(255, 255, 255)
    b.Font = Enum.Font.GothamBold
    b.TextSize = 13
    b.Parent = f
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 5)

    local state = default
    b.MouseButton1Click:Connect(function()
        state = not state
        b.Text = state and "ON" or "OFF"
        b.BackgroundColor3 = state and Color3.fromRGB(255, 45, 45) or Color3.fromRGB(50, 50, 58)
        cb(state)
    end)
end

local function Btn(name, cb)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(1, 0, 0, 34)
    b.BackgroundColor3 = Color3.fromRGB(32, 32, 40)
    b.Text = name
    b.TextColor3 = Color3.fromRGB(220, 220, 220)
    b.Font = Enum.Font.Gotham
    b.TextSize = 13
    b.Parent = Scroll
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 7)
    b.MouseButton1Click:Connect(cb)
end

local function Label(t)
    local l = Instance.new("TextLabel")
    l.Size = UDim2.new(1, 0, 0, 20)
    l.BackgroundTransparency = 1
    l.Text = t
    l.TextColor3 = Color3.fromRGB(255, 55, 55)
    l.Font = Enum.Font.GothamBold
    l.TextSize = 13
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.Parent = Scroll
end

-- UI Elements
Label("  COMBAT")
Toggle("Aimbot", false, function(v) getgenv().Onyx.Aimbot = v end)
Toggle("Silent Aim", false, function(v) getgenv().Onyx.SilentAim = v end)
Toggle("Ragebot", false, function(v) getgenv().Onyx.Ragebot = v end)
Toggle("Hitbox Expander", false, function(v) getgenv().Onyx.Hitbox = v end)

Label("  MOVEMENT")
Toggle("Infinite Jump", false, function(v) getgenv().Onyx.InfJump = v end)
Toggle("Speed", false, function(v) getgenv().Onyx.Speed = v end)

Label("  VISUALS")
Toggle("ESP", true, function(v) getgenv().Onyx.ESP = v end)
Toggle("Tracers", false, function(v) getgenv().Onyx.Tracer = v end)
Toggle("Names", true, function(v) getgenv().Onyx.NameESP = v end)

Label("  EXTRA")
Btn("Unlock Skins (try)", function()
    pcall(function()
        for _, r in ipairs(ReplicatedStorage:GetDescendants()) do
            if r:IsA("RemoteEvent") then
                local n = r.Name:lower()
                if n:find("skin") or n:find("unlock") or n:find("cosmetic") then
                    for i = 1, 20 do pcall(function() r:FireServer(i) end) end
                end
            end
        end
    end)
end)

Btn("Destroy UI", function() SG:Destroy() end)

-- ===================== FUNCTIONS =====================
local function IsAlive(plr)
    local char = plr.Character
    if not char then return false end
    local hum = char:FindFirstChildOfClass("Humanoid")
    return hum and hum.Health > 0
end

local function GetClosest(fov)
    local closest, shortest = nil, fov
    local mouse = UserInputService:GetMouseLocation()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr \~= LocalPlayer and IsAlive(plr) then
            if getgenv().Onyx.TeamCheck and plr.Team == LocalPlayer.Team then continue end
            local root = plr.Character:FindFirstChild("HumanoidRootPart")
            if root then
                local pos, onScreen = Camera:WorldToViewportPoint(root.Position)
                if onScreen or getgenv().Onyx.Ragebot then
                    local dist = (Vector2.new(pos.X, pos.Y) - mouse).Magnitude
                    if dist < shortest then
                        shortest = dist
                        closest = plr.Character:FindFirstChild("Head") or root
                    end
                end
            end
        end
    end
    return closest
end

-- Hitbox Expander
RunService.Heartbeat:Connect(function()
    if not getgenv().Onyx.Hitbox then return end
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr \~= LocalPlayer and plr.Character then
            local root = plr.Character:FindFirstChild("HumanoidRootPart")
            if root then
                root.Size = Vector3.new(getgenv().Onyx.HitboxSize, getgenv().Onyx.HitboxSize, getgenv().Onyx.HitboxSize)
                root.Transparency = 0.7
                root.CanCollide = false
                root.Massless = true
            end
        end
    end
end)

-- Silent Aim Hook
pcall(function()
    local old
    old = hookmetamethod(game, "__namecall", newcclosure(function(self, ...)
        local method = getnamecallmethod()
        local args = {...}
        if getgenv().Onyx.SilentAim and method == "FireServer" then
            local name = tostring(self):lower()
            if name:find("shoot") or name:find("fire") or name:find("bullet") or name:find("ray") or name:find("weapon") or name:find("gun") or name:find("hit") then
                local target = GetClosest(getgenv().Onyx.AimFOV + 100)
                if target then
                    for i, v in ipairs(args) do
                        if typeof(v) == "Vector3" then
                            args[i] = target.Position
                        elseif typeof(v) == "CFrame" then
                            args[i] = CFrame.new(v.Position, target.Position)
                        end
                    end
                    return old(self, unpack(args))
                end
            end
        end
        return old(self, ...)
    end))
end)

-- Aimbot / Ragebot
RunService.RenderStepped:Connect(function()
    local s = getgenv().Onyx
    if s.Aimbot or s.Ragebot then
        local target = GetClosest(s.Ragebot and 500 or s.AimFOV)
        if target then
            local cf = CFrame.new(Camera.CFrame.Position, target.Position)
            if s.Ragebot then
                Camera.CFrame = cf
            else
                Camera.CFrame = Camera.CFrame:Lerp(cf, s.Smooth)
            end
        end
    end
end)

-- Infinite Jump
UserInputService.JumpRequest:Connect(function()
    if getgenv().Onyx.InfJump and LocalPlayer.Character then
        local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if hum then
            hum:ChangeState(Enum.HumanoidStateType.Jumping)
        end
    end
end)

-- Speed
RunService.Heartbeat:Connect(function()
    if getgenv().Onyx.Speed and LocalPlayer.Character then
        local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if hum then
            hum.WalkSpeed = getgenv().Onyx.SpeedValue
        end
    end
end)

-- ESP
pcall(function()
    local drawings = {}

    local function AddESP(plr)
        if plr == LocalPlayer then return end
        local box = Drawing.new("Square")
        box.Thickness = 1
        box.Filled = false
        box.Color = Color3.fromRGB(255, 40, 40)
        box.Visible = false

        local tracer = Drawing.new("Line")
        tracer.Thickness = 1
        tracer.Color = Color3.fromRGB(255, 40, 40)
        tracer.Visible = false

        local text = Drawing.new("Text")
        text.Size = 13
        text.Center = true
        text.Outline = true
        text.Color = Color3.fromRGB(255, 255, 255)
        text.Visible = false

        drawings[plr] = {box = box, tracer = tracer, text = text}
    end

    for _, plr in ipairs(Players:GetPlayers()) do AddESP(plr) end
    Players.PlayerAdded:Connect(AddESP)

    RunService.RenderStepped:Connect(function()
        for plr, d in pairs(drawings) do
            if not getgenv().Onyx.ESP or not plr.Character or not IsAlive(plr) then
                d.box.Visible = false
                d.tracer.Visible = false
                d.text.Visible = false
                continue
            end
            local root = plr.Character:FindFirstChild("HumanoidRootPart")
            if not root then
                d.box.Visible = false
                d.tracer.Visible = false
                d.text.Visible = false
                continue
            end
            local pos, onScreen = Camera:WorldToViewportPoint(root.Position)
            if onScreen then
                local dist = (Camera.CFrame.Position - root.Position).Magnitude
                local scale = math.clamp(750 / dist, 2.5, 11)
                d.box.Size = Vector2.new(scale * 2.5, scale * 4.2)
                d.box.Position = Vector2.new(pos.X - d.box.Size.X / 2, pos.Y - d.box.Size.Y / 2)
                d.box.Visible = true

                if getgenv().Onyx.Tracer then
                    d.tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                    d.tracer.To = Vector2.new(pos.X, pos.Y)
                    d.tracer.Visible = true
                else
                    d.tracer.Visible = false
                end

                if getgenv().Onyx.NameESP then
                    d.text.Text = plr.Name .. " [" .. math.floor(dist) .. "m]"
                    d.text.Position = Vector2.new(pos.X, pos.Y - d.box.Size.Y / 2 - 12)
                    d.text.Visible = true
                else
                    d.text.Visible = false
                end
            else
                d.box.Visible = false
                d.tracer.Visible = false
                d.text.Visible = false
            end
        end
    end)
end)

print("[Onyx] Rivals script loaded")
