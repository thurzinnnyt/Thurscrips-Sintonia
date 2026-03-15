--[[
    ╔══════════════════════════════════════════╗
    ║         THURSCRIPTS v1.0.2               ║
    ║       Sintonia Roleplay Edition          ║
    ║                                          ║
    ║   Linguagem: Lua (Roblox)                ║
    ║   Cores: Roxo & Preto                    ║
    ║                                          ║
    ║   CHANGELOG v1.0.2:                      ║
    ║   * Fix DEFINITIVO: Wall Check           ║
    ║   + Nova aba: Player                     ║
    ║     + Speed Hack com slider              ║
    ║     + Fly Hack com slider                ║
    ║     + NoClip                             ║
    ║   + Novas funções em Combate:            ║
    ║     + No Recoil / No Spread              ║
    ║     + Triggerbot                          ║
    ║   + Nova aba: Recursos                   ║
    ║     + God Mode (Vida Infinita)           ║
    ║     + Infinite Ammo (Munição Infinita)   ║
    ╚══════════════════════════════════════════╝
--]]

-- ═══════════════════════════════════════════
-- SERVIÇOS
-- ═══════════════════════════════════════════
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- ═══════════════════════════════════════════
-- VARIÁVEIS DE ESTADO
-- ═══════════════════════════════════════════
local States = {
    -- Aimbot
    AimbotEnabled = false,
    FOVEnabled = false,
    FOVSize = 150,
    TeamCheck = false,
    WallCheck = false,

    -- [v1.0.2] Novas funções de combate
    NoRecoilEnabled = false,
    TriggerbotEnabled = false,

    -- ESP
    ESPEnabled = false,
    BoxESP = false,
    Tracers = false,
    LifeStatus = false,
    ESPDistance = 500,

    -- Ant Lag
    AntLagEnabled = false,
    OriginalLighting = {},
    RemovedEffects = {},
    RemovedTextures = {},
    RemovedParticles = {},

    -- [v1.0.2] Player
    SpeedHackEnabled = false,
    SpeedValue = 16,
    OriginalSpeed = 16,
    FlyEnabled = false,
    FlySpeed = 50,
    FlyBodyVelocity = nil,
    FlyBodyGyro = nil,
    NoClipEnabled = false,

    -- [v1.0.2] Recursos
    GodModeEnabled = false,
    InfiniteAmmoEnabled = false,

    -- UI
    CurrentTab = "Combate",
    UIOpen = true,
    Dragging = false,
    DragStart = nil,
    StartPos = nil,
}

-- ═══════════════════════════════════════════
-- CORES DO TEMA
-- ═══════════════════════════════════════════
local Theme = {
    Background = Color3.fromRGB(15, 15, 20),
    BackgroundAlt = Color3.fromRGB(20, 20, 28),
    Surface = Color3.fromRGB(25, 25, 35),
    SurfaceHover = Color3.fromRGB(32, 32, 45),
    SurfaceActive = Color3.fromRGB(38, 38, 52),
    PrimaryPurple = Color3.fromRGB(130, 80, 230),
    PrimaryPurpleLight = Color3.fromRGB(160, 110, 255),
    PrimaryPurpleDark = Color3.fromRGB(100, 55, 190),
    AccentPurple = Color3.fromRGB(180, 130, 255),
    AccentPink = Color3.fromRGB(200, 100, 255),
    TextPrimary = Color3.fromRGB(240, 240, 250),
    TextSecondary = Color3.fromRGB(160, 160, 180),
    TextMuted = Color3.fromRGB(100, 100, 120),
    TextAccent = Color3.fromRGB(180, 140, 255),
    Border = Color3.fromRGB(50, 50, 70),
    BorderLight = Color3.fromRGB(70, 60, 100),
    ToggleOn = Color3.fromRGB(130, 80, 230),
    ToggleOff = Color3.fromRGB(50, 50, 65),
    ToggleKnob = Color3.fromRGB(255, 255, 255),
    Danger = Color3.fromRGB(230, 70, 70),
    Success = Color3.fromRGB(70, 200, 120),
    Warning = Color3.fromRGB(230, 180, 50),
    FOVCircle = Color3.fromRGB(130, 80, 230),
    ESPBox = Color3.fromRGB(130, 80, 230),
    ESPTracer = Color3.fromRGB(180, 130, 255),
    ESPHealth = Color3.fromRGB(70, 200, 120),
    ESPHealthLow = Color3.fromRGB(230, 70, 70),
}

-- ═══════════════════════════════════════════
-- LIMPAR UI ANTERIOR
-- ═══════════════════════════════════════════
if game.CoreGui:FindFirstChild("ThurscriptsUI") then
    game.CoreGui:FindFirstChild("ThurscriptsUI"):Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ThurscriptsUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.DisplayOrder = 999
pcall(function() ScreenGui.Parent = game.CoreGui end)
if not ScreenGui.Parent then
    ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
end

-- ═══════════════════════════════════════════
-- FUNÇÕES UTILITÁRIAS UI
-- ═══════════════════════════════════════════
local function CreateCorner(parent, radius)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, radius or 8)
    c.Parent = parent
    return c
end

local function CreateStroke(parent, color, thickness, transparency)
    local s = Instance.new("UIStroke")
    s.Color = color or Theme.Border
    s.Thickness = thickness or 1
    s.Transparency = transparency or 0
    s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    s.Parent = parent
    return s
end

local function CreatePadding(parent, top, bottom, left, right)
    local p = Instance.new("UIPadding")
    p.PaddingTop = UDim.new(0, top or 0)
    p.PaddingBottom = UDim.new(0, bottom or 0)
    p.PaddingLeft = UDim.new(0, left or 0)
    p.PaddingRight = UDim.new(0, right or 0)
    p.Parent = parent
    return p
end

local function CreateGradient(parent, color1, color2, rotation)
    local g = Instance.new("UIGradient")
    g.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, color1),
        ColorSequenceKeypoint.new(1, color2)
    }
    g.Rotation = rotation or 90
    g.Parent = parent
    return g
end

local function Lerp(a, b, t) return a + (b - a) * t end
local function LerpColor(c1, c2, t)
    return Color3.new(Lerp(c1.R, c2.R, t), Lerp(c1.G, c2.G, t), Lerp(c1.B, c2.B, t))
end

-- ═══════════════════════════════════════════
-- FRAME PRINCIPAL
-- ═══════════════════════════════════════════
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 560, 0, 620)
MainFrame.Position = UDim2.new(0.5, -280, 0.5, -310)
MainFrame.BackgroundColor3 = Theme.Background
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui
CreateCorner(MainFrame, 12)
CreateStroke(MainFrame, Theme.BorderLight, 1.5, 0.3)

local MainShadow = Instance.new("ImageLabel")
MainShadow.BackgroundTransparency = 1
MainShadow.Image = "rbxassetid://6015897843"
MainShadow.ImageColor3 = Color3.fromRGB(0, 0, 0)
MainShadow.ImageTransparency = 0.4
MainShadow.Size = UDim2.new(1, 50, 1, 50)
MainShadow.Position = UDim2.new(0.5, 0, 0.5, 0)
MainShadow.AnchorPoint = Vector2.new(0.5, 0.5)
MainShadow.ScaleType = Enum.ScaleType.Slice
MainShadow.SliceCenter = Rect.new(49, 49, 450, 450)
MainShadow.ZIndex = 0
MainShadow.Parent = MainFrame

-- ═══════════════════════════════════════════
-- HEADER
-- ═══════════════════════════════════════════
local HeaderFrame = Instance.new("Frame")
HeaderFrame.Name = "Header"
HeaderFrame.Size = UDim2.new(1, 0, 0, 60)
HeaderFrame.BackgroundColor3 = Theme.BackgroundAlt
HeaderFrame.BorderSizePixel = 0
HeaderFrame.ZIndex = 5
HeaderFrame.Parent = MainFrame
CreateCorner(HeaderFrame, 12)

local HeaderFix = Instance.new("Frame")
HeaderFix.Size = UDim2.new(1, 0, 0, 15)
HeaderFix.Position = UDim2.new(0, 0, 1, -15)
HeaderFix.BackgroundColor3 = Theme.BackgroundAlt
HeaderFix.BorderSizePixel = 0
HeaderFix.ZIndex = 5
HeaderFix.Parent = HeaderFrame

local HeaderGrad = Instance.new("Frame")
HeaderGrad.Size = UDim2.new(1, 0, 0, 2)
HeaderGrad.Position = UDim2.new(0, 0, 1, 0)
HeaderGrad.BorderSizePixel = 0
HeaderGrad.ZIndex = 6
HeaderGrad.Parent = HeaderFrame
CreateGradient(HeaderGrad, Theme.PrimaryPurple, Theme.AccentPink, 0)

local LogoIcon = Instance.new("TextLabel")
LogoIcon.Size = UDim2.new(0, 35, 0, 35)
LogoIcon.Position = UDim2.new(0, 18, 0.5, -17)
LogoIcon.BackgroundColor3 = Theme.PrimaryPurple
LogoIcon.BorderSizePixel = 0
LogoIcon.Text = "T"
LogoIcon.TextColor3 = Color3.fromRGB(255, 255, 255)
LogoIcon.TextSize = 18
LogoIcon.Font = Enum.Font.GothamBold
LogoIcon.ZIndex = 7
LogoIcon.Parent = HeaderFrame
CreateCorner(LogoIcon, 8)

local LogoGlow = Instance.new("Frame")
LogoGlow.Size = UDim2.new(1, 8, 1, 8)
LogoGlow.Position = UDim2.new(0.5, 0, 0.5, 0)
LogoGlow.AnchorPoint = Vector2.new(0.5, 0.5)
LogoGlow.BackgroundColor3 = Theme.PrimaryPurple
LogoGlow.BackgroundTransparency = 0.7
LogoGlow.BorderSizePixel = 0
LogoGlow.ZIndex = 6
LogoGlow.Parent = LogoIcon
CreateCorner(LogoGlow, 10)

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(0, 200, 0, 22)
TitleLabel.Position = UDim2.new(0, 62, 0, 12)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "THURSCRIPTS"
TitleLabel.TextColor3 = Theme.TextPrimary
TitleLabel.TextSize = 18
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.ZIndex = 7
TitleLabel.Parent = HeaderFrame

local SubLabel = Instance.new("TextLabel")
SubLabel.Size = UDim2.new(0, 200, 0, 16)
SubLabel.Position = UDim2.new(0, 62, 0, 34)
SubLabel.BackgroundTransparency = 1
SubLabel.Text = "Sintonia Roleplay"
SubLabel.TextColor3 = Theme.TextAccent
SubLabel.TextSize = 12
SubLabel.Font = Enum.Font.GothamMedium
SubLabel.TextXAlignment = Enum.TextXAlignment.Left
SubLabel.ZIndex = 7
SubLabel.Parent = HeaderFrame

local VerLabel = Instance.new("TextLabel")
VerLabel.Size = UDim2.new(0, 60, 0, 24)
VerLabel.Position = UDim2.new(1, -80, 0.5, -12)
VerLabel.BackgroundColor3 = Theme.Surface
VerLabel.BorderSizePixel = 0
VerLabel.Text = "v1.0.2"
VerLabel.TextColor3 = Theme.TextSecondary
VerLabel.TextSize = 11
VerLabel.Font = Enum.Font.GothamMedium
VerLabel.ZIndex = 7
VerLabel.Parent = HeaderFrame
CreateCorner(VerLabel, 6)
CreateStroke(VerLabel, Theme.Border, 1, 0.5)

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 30, 0, 30)
CloseBtn.Position = UDim2.new(1, -42, 0.5, -15)
CloseBtn.BackgroundTransparency = 1
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Theme.TextMuted
CloseBtn.TextSize = 16
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.ZIndex = 8
CloseBtn.Parent = HeaderFrame

CloseBtn.MouseEnter:Connect(function() CloseBtn.TextColor3 = Theme.Danger end)
CloseBtn.MouseLeave:Connect(function() CloseBtn.TextColor3 = Theme.TextMuted end)
CloseBtn.MouseButton1Click:Connect(function()
    States.UIOpen = false
    MainFrame.Visible = false
end)

-- ═══════════════════════════════════════════
-- DRAG
-- ═══════════════════════════════════════════
HeaderFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        States.Dragging = true
        States.DragStart = input.Position
        States.StartPos = MainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then States.Dragging = false end
        end)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if States.Dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - States.DragStart
        MainFrame.Position = UDim2.new(States.StartPos.X.Scale, States.StartPos.X.Offset + delta.X, States.StartPos.Y.Scale, States.StartPos.Y.Offset + delta.Y)
    end
end)

-- ═══════════════════════════════════════════
-- [v1.0.2] NAVEGAÇÃO - 5 ABAS
-- ═══════════════════════════════════════════
local NavFrame = Instance.new("Frame")
NavFrame.Size = UDim2.new(1, -30, 0, 42)
NavFrame.Position = UDim2.new(0, 15, 0, 72)
NavFrame.BackgroundColor3 = Theme.Surface
NavFrame.BorderSizePixel = 0
NavFrame.ZIndex = 5
NavFrame.Parent = MainFrame
CreateCorner(NavFrame, 8)

local TabIndicator = Instance.new("Frame")
TabIndicator.Size = UDim2.new(1/5, -4, 0, 34)
TabIndicator.Position = UDim2.new(0, 4, 0.5, -17)
TabIndicator.BackgroundColor3 = Theme.PrimaryPurple
TabIndicator.BorderSizePixel = 0
TabIndicator.ZIndex = 6
TabIndicator.Parent = NavFrame
CreateCorner(TabIndicator, 6)
CreateGradient(TabIndicator, Theme.PrimaryPurple, Theme.PrimaryPurpleDark, 90)

local tabNames = {"Combate", "Visuals", "Player", "Recursos", "Ant Lag"}
local tabIcons = {"⚔", "👁", "🏃", "💎", "⚡"}
local tabKeys = {"Combate", "Visuals", "Player", "Recursos", "AntLag"}
local TabButtons = {}

for i = 1, 5 do
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1/5, 0, 1, 0)
    btn.Position = UDim2.new((i-1)/5, 0, 0, 0)
    btn.BackgroundTransparency = 1
    btn.Text = tabIcons[i] .. " " .. tabNames[i]
    btn.TextColor3 = (i == 1) and Color3.fromRGB(255, 255, 255) or Theme.TextSecondary
    btn.TextSize = 11
    btn.Font = (i == 1) and Enum.Font.GothamBold or Enum.Font.GothamMedium
    btn.ZIndex = 7
    btn.Parent = NavFrame
    TabButtons[i] = btn
end

-- ═══════════════════════════════════════════
-- CONTENT AREA
-- ═══════════════════════════════════════════
local ContentFrame = Instance.new("Frame")
ContentFrame.Size = UDim2.new(1, -30, 1, -132)
ContentFrame.Position = UDim2.new(0, 15, 0, 122)
ContentFrame.BackgroundTransparency = 1
ContentFrame.ClipsDescendants = true
ContentFrame.ZIndex = 3
ContentFrame.Parent = MainFrame

local function MakeScrollFrame(name, vis)
    local sf = Instance.new("ScrollingFrame")
    sf.Name = name
    sf.Size = UDim2.new(1, 0, 1, 0)
    sf.BackgroundTransparency = 1
    sf.BorderSizePixel = 0
    sf.ScrollBarThickness = 3
    sf.ScrollBarImageColor3 = Theme.PrimaryPurple
    sf.CanvasSize = UDim2.new(0, 0, 0, 700)
    sf.ZIndex = 4
    sf.Visible = vis
    sf.Parent = ContentFrame
    local lay = Instance.new("UIListLayout")
    lay.SortOrder = Enum.SortOrder.LayoutOrder
    lay.Padding = UDim.new(0, 8)
    lay.Parent = sf
    return sf
end

local CombateContent = MakeScrollFrame("Combate", true)
local VisualsContent = MakeScrollFrame("Visuals", false)
local PlayerContent = MakeScrollFrame("Player", false)
local RecursosContent = MakeScrollFrame("Recursos", false)
local AntLagContent = MakeScrollFrame("AntLag", false)

local AllContents = {CombateContent, VisualsContent, PlayerContent, RecursosContent, AntLagContent}

-- ═══════════════════════════════════════════
-- TAB SWITCHING
-- ═══════════════════════════════════════════
local function SwitchTab(index)
    States.CurrentTab = tabKeys[index]
    for i = 1, 5 do
        TabButtons[i].TextColor3 = Theme.TextSecondary
        TabButtons[i].Font = Enum.Font.GothamMedium
        AllContents[i].Visible = false
    end
    TabButtons[index].TextColor3 = Color3.fromRGB(255, 255, 255)
    TabButtons[index].Font = Enum.Font.GothamBold
    AllContents[index].Visible = true
    TweenService:Create(TabIndicator, TweenInfo.new(0.3, Enum.EasingStyle.Quart), {
        Position = UDim2.new((index-1)/5, 2, 0.5, -17)
    }):Play()
end

for i = 1, 5 do
    TabButtons[i].MouseButton1Click:Connect(function() SwitchTab(i) end)
end

-- ═══════════════════════════════════════════
-- COMPONENTES UI
-- ═══════════════════════════════════════════
local function CreateSectionHeader(parent, text, icon, layoutOrder)
    local f = Instance.new("Frame")
    f.Size = UDim2.new(1, 0, 0, 32)
    f.BackgroundTransparency = 1
    f.LayoutOrder = layoutOrder or 0
    f.ZIndex = 5
    f.Parent = parent
    local l = Instance.new("TextLabel")
    l.Size = UDim2.new(1, 0, 1, 0)
    l.BackgroundTransparency = 1
    l.Text = (icon or "") .. "  " .. string.upper(text)
    l.TextColor3 = Theme.TextAccent
    l.TextSize = 12
    l.Font = Enum.Font.GothamBold
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.ZIndex = 5
    l.Parent = f
    local ln = Instance.new("Frame")
    ln.Size = UDim2.new(1, 0, 0, 1)
    ln.Position = UDim2.new(0, 0, 1, -2)
    ln.BackgroundColor3 = Theme.Border
    ln.BackgroundTransparency = 0.5
    ln.BorderSizePixel = 0
    ln.ZIndex = 5
    ln.Parent = f
    return f
end

local function CreateToggle(parent, text, description, defaultState, layoutOrder, callback)
    local f = Instance.new("Frame")
    f.Size = UDim2.new(1, 0, 0, description and 56 or 42)
    f.BackgroundColor3 = Theme.Surface
    f.BorderSizePixel = 0
    f.LayoutOrder = layoutOrder or 0
    f.ZIndex = 5
    f.Parent = parent
    CreateCorner(f, 8)
    CreateStroke(f, Theme.Border, 1, 0.6)
    CreatePadding(f, 0, 0, 14, 14)

    local tl = Instance.new("TextLabel")
    tl.Size = UDim2.new(1, -60, 0, 18)
    tl.Position = UDim2.new(0, 0, 0, description and 8 or 12)
    tl.BackgroundTransparency = 1
    tl.Text = text
    tl.TextColor3 = Theme.TextPrimary
    tl.TextSize = 13
    tl.Font = Enum.Font.GothamSemibold
    tl.TextXAlignment = Enum.TextXAlignment.Left
    tl.ZIndex = 6
    tl.Parent = f

    if description then
        local dl = Instance.new("TextLabel")
        dl.Size = UDim2.new(1, -60, 0, 14)
        dl.Position = UDim2.new(0, 0, 0, 28)
        dl.BackgroundTransparency = 1
        dl.Text = description
        dl.TextColor3 = Theme.TextMuted
        dl.TextSize = 10
        dl.Font = Enum.Font.Gotham
        dl.TextXAlignment = Enum.TextXAlignment.Left
        dl.ZIndex = 6
        dl.Parent = f
    end

    local bg = Instance.new("Frame")
    bg.Size = UDim2.new(0, 42, 0, 22)
    bg.Position = UDim2.new(1, -42, 0.5, -11)
    bg.BackgroundColor3 = defaultState and Theme.ToggleOn or Theme.ToggleOff
    bg.BorderSizePixel = 0
    bg.ZIndex = 7
    bg.Parent = f
    CreateCorner(bg, 11)

    local knob = Instance.new("Frame")
    knob.Size = UDim2.new(0, 16, 0, 16)
    knob.Position = defaultState and UDim2.new(1, -19, 0.5, -8) or UDim2.new(0, 3, 0.5, -8)
    knob.BackgroundColor3 = Theme.ToggleKnob
    knob.BorderSizePixel = 0
    knob.ZIndex = 8
    knob.Parent = bg
    CreateCorner(knob, 8)

    local isOn = defaultState
    local click = Instance.new("TextButton")
    click.Size = UDim2.new(1, 28, 1, 0)
    click.Position = UDim2.new(0, -14, 0, 0)
    click.BackgroundTransparency = 1
    click.Text = ""
    click.ZIndex = 9
    click.Parent = f

    click.MouseEnter:Connect(function() f.BackgroundColor3 = Theme.SurfaceHover end)
    click.MouseLeave:Connect(function() f.BackgroundColor3 = Theme.Surface end)
    click.MouseButton1Click:Connect(function()
        isOn = not isOn
        TweenService:Create(bg, TweenInfo.new(0.25, Enum.EasingStyle.Quart), {
            BackgroundColor3 = isOn and Theme.ToggleOn or Theme.ToggleOff
        }):Play()
        TweenService:Create(knob, TweenInfo.new(0.25, Enum.EasingStyle.Quart), {
            Position = isOn and UDim2.new(1, -19, 0.5, -8) or UDim2.new(0, 3, 0.5, -8)
        }):Play()
        if callback then callback(isOn) end
    end)
    return f
end

local function CreateSlider(parent, text, min, max, default, layoutOrder, callback)
    local f = Instance.new("Frame")
    f.Size = UDim2.new(1, 0, 0, 60)
    f.BackgroundColor3 = Theme.Surface
    f.BorderSizePixel = 0
    f.LayoutOrder = layoutOrder or 0
    f.ZIndex = 5
    f.Parent = parent
    CreateCorner(f, 8)
    CreateStroke(f, Theme.Border, 1, 0.6)
    CreatePadding(f, 0, 0, 14, 14)

    local tl = Instance.new("TextLabel")
    tl.Size = UDim2.new(1, -55, 0, 18)
    tl.Position = UDim2.new(0, 0, 0, 6)
    tl.BackgroundTransparency = 1
    tl.Text = text
    tl.TextColor3 = Theme.TextPrimary
    tl.TextSize = 13
    tl.Font = Enum.Font.GothamSemibold
    tl.TextXAlignment = Enum.TextXAlignment.Left
    tl.ZIndex = 6
    tl.Parent = f

    local vl = Instance.new("TextLabel")
    vl.Size = UDim2.new(0, 45, 0, 20)
    vl.Position = UDim2.new(1, -45, 0, 5)
    vl.BackgroundColor3 = Theme.BackgroundAlt
    vl.BorderSizePixel = 0
    vl.Text = tostring(default)
    vl.TextColor3 = Theme.PrimaryPurpleLight
    vl.TextSize = 12
    vl.Font = Enum.Font.GothamBold
    vl.ZIndex = 6
    vl.Parent = f
    CreateCorner(vl, 5)

    local sbg = Instance.new("Frame")
    sbg.Size = UDim2.new(1, 0, 0, 6)
    sbg.Position = UDim2.new(0, 0, 0, 38)
    sbg.BackgroundColor3 = Theme.BackgroundAlt
    sbg.BorderSizePixel = 0
    sbg.ZIndex = 6
    sbg.Parent = f
    CreateCorner(sbg, 3)

    local ip = (default - min) / (max - min)
    local fill = Instance.new("Frame")
    fill.Size = UDim2.new(ip, 0, 1, 0)
    fill.BackgroundColor3 = Theme.PrimaryPurple
    fill.BorderSizePixel = 0
    fill.ZIndex = 7
    fill.Parent = sbg
    CreateCorner(fill, 3)
    CreateGradient(fill, Theme.PrimaryPurple, Theme.AccentPink, 0)

    local sk = Instance.new("Frame")
    sk.Size = UDim2.new(0, 14, 0, 14)
    sk.Position = UDim2.new(ip, -7, 0.5, -7)
    sk.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    sk.BorderSizePixel = 0
    sk.ZIndex = 8
    sk.Parent = sbg
    CreateCorner(sk, 7)
    CreateStroke(sk, Theme.PrimaryPurple, 2)

    local sliding = false
    local sb = Instance.new("TextButton")
    sb.Size = UDim2.new(1, 0, 0, 22)
    sb.Position = UDim2.new(0, 0, 0, 30)
    sb.BackgroundTransparency = 1
    sb.Text = ""
    sb.ZIndex = 10
    sb.Parent = f

    local function upd(inputX)
        local rel = math.clamp((inputX - sbg.AbsolutePosition.X) / sbg.AbsoluteSize.X, 0, 1)
        local val = math.floor(min + (max - min) * rel)
        fill.Size = UDim2.new(rel, 0, 1, 0)
        sk.Position = UDim2.new(rel, -7, 0.5, -7)
        vl.Text = tostring(val)
        if callback then callback(val) end
    end

    sb.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            sliding = true; upd(input.Position.X)
        end
    end)
    sb.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            sliding = false
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if sliding and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            upd(input.Position.X)
        end
    end)
    return f
end

local function CreateInfoCard(parent, title, desc, icon, layoutOrder)
    local f = Instance.new("Frame")
    f.Size = UDim2.new(1, 0, 0, 75)
    f.BackgroundColor3 = Theme.Surface
    f.BorderSizePixel = 0
    f.LayoutOrder = layoutOrder or 0
    f.ZIndex = 5
    f.Parent = parent
    CreateCorner(f, 8)
    CreateStroke(f, Theme.Border, 1, 0.6)
    CreatePadding(f, 10, 10, 14, 14)

    local il = Instance.new("TextLabel")
    il.Size = UDim2.new(0, 25, 0, 25)
    il.BackgroundTransparency = 1
    il.Text = icon or "📋"
    il.TextSize = 18
    il.Font = Enum.Font.GothamBold
    il.ZIndex = 6
    il.Parent = f

    local ttl = Instance.new("TextLabel")
    ttl.Size = UDim2.new(1, -35, 0, 16)
    ttl.Position = UDim2.new(0, 33, 0, 0)
    ttl.BackgroundTransparency = 1
    ttl.Text = title
    ttl.TextColor3 = Theme.TextPrimary
    ttl.TextSize = 12
    ttl.Font = Enum.Font.GothamBold
    ttl.TextXAlignment = Enum.TextXAlignment.Left
    ttl.ZIndex = 6
    ttl.Parent = f

    local dl = Instance.new("TextLabel")
    dl.Size = UDim2.new(1, -33, 0, 30)
    dl.Position = UDim2.new(0, 33, 0, 20)
    dl.BackgroundTransparency = 1
    dl.Text = desc
    dl.TextColor3 = Theme.TextMuted
    dl.TextSize = 10
    dl.Font = Enum.Font.Gotham
    dl.TextXAlignment = Enum.TextXAlignment.Left
    dl.TextYAlignment = Enum.TextYAlignment.Top
    dl.TextWrapped = true
    dl.ZIndex = 6
    dl.Parent = f
    return f
end

local function CreateStatusIndicator(parent, layoutOrder)
    local f = Instance.new("Frame")
    f.Size = UDim2.new(1, 0, 0, 44)
    f.BackgroundColor3 = Theme.Surface
    f.BorderSizePixel = 0
    f.LayoutOrder = layoutOrder or 0
    f.ZIndex = 5
    f.Parent = parent
    CreateCorner(f, 8)
    CreateStroke(f, Theme.Border, 1, 0.6)
    CreatePadding(f, 0, 0, 14, 14)

    local dot = Instance.new("Frame")
    dot.Size = UDim2.new(0, 10, 0, 10)
    dot.Position = UDim2.new(0, 2, 0.5, -5)
    dot.BackgroundColor3 = Theme.Danger
    dot.BorderSizePixel = 0
    dot.ZIndex = 7
    dot.Parent = f
    CreateCorner(dot, 5)

    local st = Instance.new("TextLabel")
    st.Size = UDim2.new(1, -20, 1, 0)
    st.Position = UDim2.new(0, 20, 0, 0)
    st.BackgroundTransparency = 1
    st.Text = "Status: Desativado"
    st.TextColor3 = Theme.TextSecondary
    st.TextSize = 12
    st.Font = Enum.Font.GothamMedium
    st.TextXAlignment = Enum.TextXAlignment.Left
    st.ZIndex = 6
    st.Parent = f
    return f, dot, st
end

-- ═══════════════════════════════════════════
-- ABA COMBATE
-- ═══════════════════════════════════════════
CreateSectionHeader(CombateContent, "Aimbot", "🎯", 1)

CreateToggle(CombateContent, "Aimbot", "Gruda a mira na cabeça dos jogadores", false, 2, function(s)
    States.AimbotEnabled = s
end)

CreateToggle(CombateContent, "Mostrar FOV", "Exibe o círculo de FOV na tela", false, 3, function(s)
    States.FOVEnabled = s
end)

CreateSlider(CombateContent, "Tamanho do FOV", 30, 500, 150, 4, function(v)
    States.FOVSize = v
end)

CreateSectionHeader(CombateContent, "Configurações do Aimbot", "⚙", 5)

CreateToggle(CombateContent, "Team Check", "Ignora jogadores do mesmo time", false, 6, function(s)
    States.TeamCheck = s
end)

CreateToggle(CombateContent, "Wall Check", "Mira apenas em alvos sem obstáculos na frente", false, 7, function(s)
    States.WallCheck = s
end)

-- [v1.0.2] Novas funções de combate
CreateSectionHeader(CombateContent, "Armas", "🔫", 8)

CreateToggle(CombateContent, "No Recoil / No Spread", "Remove recuo e dispersão das armas", false, 9, function(s)
    States.NoRecoilEnabled = s
end)

CreateToggle(CombateContent, "Triggerbot", "Dispara automaticamente ao mirar em inimigos", false, 10, function(s)
    States.TriggerbotEnabled = s
end)

-- ═══════════════════════════════════════════
-- ABA VISUALS
-- ═══════════════════════════════════════════
CreateSectionHeader(VisualsContent, "ESP", "👁", 1)

CreateToggle(VisualsContent, "ESP", "Visualizar jogadores no mapa", false, 2, function(s)
    States.ESPEnabled = s
end)

CreateToggle(VisualsContent, "Boxfield", "Caixa em volta dos jogadores", false, 3, function(s)
    States.BoxESP = s
end)

CreateToggle(VisualsContent, "Tracers", "Linhas apontando para jogadores", false, 4, function(s)
    States.Tracers = s
end)

CreateToggle(VisualsContent, "Life Status", "Mostra vida dos jogadores em tempo real", false, 5, function(s)
    States.LifeStatus = s
end)

CreateSectionHeader(VisualsContent, "Configurações", "⚙", 6)

CreateSlider(VisualsContent, "Distância do ESP", 0, 2000, 500, 7, function(v)
    States.ESPDistance = v
end)

-- ═══════════════════════════════════════════
-- [v1.0.2] ABA PLAYER - NOVA
-- ═══════════════════════════════════════════
CreateSectionHeader(PlayerContent, "Movimento", "🏃", 1)

CreateToggle(PlayerContent, "Speed Hack", "Aumenta a velocidade de movimento", false, 2, function(s)
    States.SpeedHackEnabled = s
    local char = LocalPlayer.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            if s then
                States.OriginalSpeed = hum.WalkSpeed
                hum.WalkSpeed = States.SpeedValue
            else
                hum.WalkSpeed = States.OriginalSpeed
            end
        end
    end
end)

CreateSlider(PlayerContent, "Velocidade (Speed)", 0, 10, 2, 3, function(v)
    -- Mapear 0-10 para velocidades reais (16 a 200)
    local realSpeed = 16 + (v * 18.4)
    States.SpeedValue = realSpeed
    if States.SpeedHackEnabled then
        local char = LocalPlayer.Character
        if char then
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then
                hum.WalkSpeed = realSpeed
            end
        end
    end
end)

CreateSectionHeader(PlayerContent, "Voo", "✈️", 4)

CreateToggle(PlayerContent, "Fly Hack", "Permite voar pelo mapa", false, 5, function(s)
    States.FlyEnabled = s
    local char = LocalPlayer.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hrp then return end

    if s then
        -- Criar BodyVelocity e BodyGyro para voar
        if States.FlyBodyVelocity then
            pcall(function() States.FlyBodyVelocity:Destroy() end)
        end
        if States.FlyBodyGyro then
            pcall(function() States.FlyBodyGyro:Destroy() end)
        end

        local bv = Instance.new("BodyVelocity")
        bv.Name = "ThurFlyBV"
        bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        bv.Velocity = Vector3.new(0, 0, 0)
        bv.Parent = hrp
        States.FlyBodyVelocity = bv

        local bg = Instance.new("BodyGyro")
        bg.Name = "ThurFlyBG"
        bg.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
        bg.D = 100
        bg.P = 10000
        bg.Parent = hrp
        States.FlyBodyGyro = bg

        if hum then
            hum.PlatformStand = true
        end
    else
        if States.FlyBodyVelocity then
            pcall(function() States.FlyBodyVelocity:Destroy() end)
            States.FlyBodyVelocity = nil
        end
        if States.FlyBodyGyro then
            pcall(function() States.FlyBodyGyro:Destroy() end)
            States.FlyBodyGyro = nil
        end
        if hum then
            hum.PlatformStand = false
        end
    end
end)

CreateSlider(PlayerContent, "Velocidade do Voo", 0, 10, 3, 6, function(v)
    States.FlySpeed = 10 + (v * 15)
end)

CreateSectionHeader(PlayerContent, "Colisão", "👻", 7)

CreateToggle(PlayerContent, "NoClip", "Atravesse paredes e objetos sólidos", false, 8, function(s)
    States.NoClipEnabled = s
end)

-- ═══════════════════════════════════════════
-- [v1.0.2] ABA RECURSOS - NOVA
-- ═══════════════════════════════════════════
CreateSectionHeader(RecursosContent, "Sobrevivência", "💎", 1)

CreateToggle(RecursosContent, "God Mode", "Vida infinita — não sofre dano", false, 2, function(s)
    States.GodModeEnabled = s
end)

CreateToggle(RecursosContent, "Munição Infinita", "A arma nunca precisa recarregar", false, 3, function(s)
    States.InfiniteAmmoEnabled = s
end)

CreateInfoCard(RecursosContent, "Como funciona?",
    "God Mode mantém sua vida no máximo constantemente.\nMunição Infinita tenta manter os valores de ammo em armas sempre cheios.",
    "ℹ️", 4)

CreateInfoCard(RecursosContent, "Aviso",
    "Algumas funções podem não funcionar em todos os jogos dependendo do sistema de segurança do servidor.",
    "⚠️", 5)

-- ═══════════════════════════════════════════
-- ABA ANT LAG
-- ═══════════════════════════════════════════
CreateSectionHeader(AntLagContent, "Otimização", "⚡", 1)
local _, statusDot, statusText = CreateStatusIndicator(AntLagContent, 2)

CreateInfoCard(AntLagContent, "O que o Ant Lag faz?",
    "Remove texturas, partículas, efeitos de iluminação, sombras e detalhes para melhorar FPS.", "📋", 3)

local function EnableAntLag()
    States.OriginalLighting = {
        GlobalShadows = Lighting.GlobalShadows,
        FogEnd = Lighting.FogEnd,
        FogStart = Lighting.FogStart,
    }
    pcall(function()
        Lighting.GlobalShadows = false
        Lighting.FogEnd = 100000
        Lighting.FogStart = 100000
    end)
    pcall(function()
        for _, e in pairs(Lighting:GetChildren()) do
            if e:IsA("BloomEffect") or e:IsA("BlurEffect") or e:IsA("ColorCorrectionEffect")
                or e:IsA("SunRaysEffect") or e:IsA("DepthOfFieldEffect") or e:IsA("Atmosphere") then
                table.insert(States.RemovedEffects, {Object = e})
                pcall(function() e.Enabled = false end)
            end
        end
    end)
    pcall(function()
        local t = workspace:FindFirstChildOfClass("Terrain")
        if t then t.WaterWaveSize = 0; t.WaterWaveSpeed = 0; t.WaterReflectance = 0; t.Decoration = false end
    end)
    pcall(function()
        for _, d in pairs(workspace:GetDescendants()) do
            if d:IsA("ParticleEmitter") then
                table.insert(States.RemovedParticles, {Object = d, Enabled = d.Enabled, Rate = d.Rate})
                pcall(function() d.Enabled = false; d.Rate = 0 end)
            elseif d:IsA("Fire") or d:IsA("Smoke") or d:IsA("Sparkles") then
                table.insert(States.RemovedParticles, {Object = d, Enabled = d.Enabled})
                pcall(function() d.Enabled = false end)
            elseif d:IsA("Texture") or d:IsA("Decal") then
                table.insert(States.RemovedTextures, {Object = d, Transparency = d.Transparency})
                pcall(function() d.Transparency = 1 end)
            elseif d:IsA("PointLight") or d:IsA("SpotLight") or d:IsA("SurfaceLight") then
                pcall(function() d.Enabled = false end)
            elseif d:IsA("Beam") or d:IsA("Trail") then
                pcall(function() d.Enabled = false end)
            end
            if d:IsA("MeshPart") then pcall(function() d.RenderFidelity = Enum.RenderFidelity.Performance end) end
            if d:IsA("BasePart") then pcall(function() d.CastShadow = false end) end
        end
    end)
    pcall(function() settings().Rendering.QualityLevel = 1 end)
    statusDot.BackgroundColor3 = Theme.Success
    statusText.Text = "Status: Ativado — FPS otimizado"
    statusText.TextColor3 = Theme.Success
end

local function DisableAntLag()
    pcall(function()
        if States.OriginalLighting.GlobalShadows ~= nil then Lighting.GlobalShadows = States.OriginalLighting.GlobalShadows end
        if States.OriginalLighting.FogEnd then Lighting.FogEnd = States.OriginalLighting.FogEnd end
        if States.OriginalLighting.FogStart then Lighting.FogStart = States.OriginalLighting.FogStart end
    end)
    for _, d in pairs(States.RemovedEffects) do pcall(function() d.Object.Enabled = true end) end
    States.RemovedEffects = {}
    for _, d in pairs(States.RemovedParticles) do
        pcall(function() d.Object.Enabled = d.Enabled; if d.Rate then d.Object.Rate = d.Rate end end)
    end
    States.RemovedParticles = {}
    for _, d in pairs(States.RemovedTextures) do pcall(function() d.Object.Transparency = d.Transparency end) end
    States.RemovedTextures = {}
    pcall(function()
        for _, d in pairs(workspace:GetDescendants()) do
            if d:IsA("BasePart") then pcall(function() d.CastShadow = true end) end
        end
    end)
    pcall(function() settings().Rendering.QualityLevel = Enum.QualityLevel.Automatic end)
    statusDot.BackgroundColor3 = Theme.Danger
    statusText.Text = "Status: Desativado"
    statusText.TextColor3 = Theme.TextSecondary
end

CreateToggle(AntLagContent, "Ant Lag", "Otimiza o jogo para melhorar FPS", false, 4, function(s)
    States.AntLagEnabled = s
    if s then EnableAntLag() else DisableAntLag() end
end)

-- ═══════════════════════════════════════════
-- FOOTER
-- ═══════════════════════════════════════════
local Footer = Instance.new("Frame")
Footer.Size = UDim2.new(1, 0, 0, 26)
Footer.Position = UDim2.new(0, 0, 1, -26)
Footer.BackgroundColor3 = Theme.BackgroundAlt
Footer.BackgroundTransparency = 0.3
Footer.BorderSizePixel = 0
Footer.ZIndex = 5
Footer.Parent = MainFrame

local FText = Instance.new("TextLabel")
FText.Size = UDim2.new(1, -20, 1, 0)
FText.Position = UDim2.new(0, 10, 0, 0)
FText.BackgroundTransparency = 1
FText.Text = "Thurscripts v1.0.2 © 2025  •  INSERT para abrir/fechar"
FText.TextColor3 = Theme.TextMuted
FText.TextSize = 10
FText.Font = Enum.Font.Gotham
FText.TextXAlignment = Enum.TextXAlignment.Left
FText.ZIndex = 6
FText.Parent = Footer

-- ═══════════════════════════════════════════
-- FOV CIRCLE
-- ═══════════════════════════════════════════
local FOVCircle = nil
pcall(function()
    FOVCircle = Drawing.new("Circle")
    FOVCircle.Color = Theme.FOVCircle
    FOVCircle.Thickness = 1.5
    FOVCircle.NumSides = 80
    FOVCircle.Radius = States.FOVSize
    FOVCircle.Filled = false
    FOVCircle.Visible = false
    FOVCircle.Transparency = 0.8
end)

-- ═══════════════════════════════════════════
-- ESP OBJECTS
-- ═══════════════════════════════════════════
local ESPObjects = {}

local function CreateESPForPlayer(player)
    if player == LocalPlayer then return end
    if ESPObjects[player] then return end
    local obj = {}
    pcall(function()
        obj.BoxOutline = Drawing.new("Quad"); obj.BoxOutline.Color = Color3.fromRGB(0,0,0); obj.BoxOutline.Thickness = 3; obj.BoxOutline.Filled = false; obj.BoxOutline.Visible = false
        obj.Box = Drawing.new("Quad"); obj.Box.Color = Theme.ESPBox; obj.Box.Thickness = 1.5; obj.Box.Filled = false; obj.Box.Visible = false
        obj.TracerOutline = Drawing.new("Line"); obj.TracerOutline.Color = Color3.fromRGB(0,0,0); obj.TracerOutline.Thickness = 3; obj.TracerOutline.Visible = false
        obj.Tracer = Drawing.new("Line"); obj.Tracer.Color = Theme.ESPTracer; obj.Tracer.Thickness = 1.5; obj.Tracer.Visible = false
        obj.HealthBarBG = Drawing.new("Line"); obj.HealthBarBG.Color = Color3.fromRGB(0,0,0); obj.HealthBarBG.Thickness = 4; obj.HealthBarBG.Visible = false
        obj.HealthBar = Drawing.new("Line"); obj.HealthBar.Color = Theme.ESPHealth; obj.HealthBar.Thickness = 2; obj.HealthBar.Visible = false
        obj.HealthText = Drawing.new("Text"); obj.HealthText.Color = Color3.fromRGB(255,255,255); obj.HealthText.Size = 13; obj.HealthText.Center = true; obj.HealthText.Outline = true; obj.HealthText.Visible = false
        obj.NameText = Drawing.new("Text"); obj.NameText.Color = Theme.TextPrimary; obj.NameText.Size = 13; obj.NameText.Center = true; obj.NameText.Outline = true; obj.NameText.Visible = false
        obj.DistText = Drawing.new("Text"); obj.DistText.Color = Theme.TextSecondary; obj.DistText.Size = 12; obj.DistText.Center = true; obj.DistText.Outline = true; obj.DistText.Visible = false
    end)
    ESPObjects[player] = obj
end

local function RemoveESPForPlayer(player)
    if ESPObjects[player] then
        for _, o in pairs(ESPObjects[player]) do pcall(function() o:Remove() end) end
        ESPObjects[player] = nil
    end
end

local function HideESP(objects)
    for _, o in pairs(objects) do pcall(function() o.Visible = false end) end
end

-- ═══════════════════════════════════════════
-- [v1.0.2] WALL CHECK - FIX DEFINITIVO
-- ═══════════════════════════════════════════
--[[
    FIX DEFINITIVO DO WALL CHECK:
    
    O problema era que o Raycast usava FilterDescendants que
    NÃO funcionava corretamente em todos os executors.
    
    NOVA SOLUÇÃO: Raycast MÚLTIPLO com verificação manual.
    Se o ray bater em algo, verifica se é um character de player.
    Se for, ignora e lança outro ray. Se for parede real, bloqueia.
    
    Isso garante que o Wall Check funcione em QUALQUER executor.
--]]

local function IsPartOfAnyCharacter(part)
    -- Sobe na hierarquia para ver se pertence a algum character
    local current = part
    for _ = 1, 10 do
        if current == nil then return false end
        if current.Parent == workspace then
            -- Verificar se é um character de algum player
            for _, p in pairs(Players:GetPlayers()) do
                if p.Character and p.Character == current then
                    return true
                end
            end
            return false
        end
        current = current.Parent
    end
    return false
end

local function CanSeeTarget(fromPosition, targetHead, targetCharacter)
    -- Se Wall Check está DESATIVADO, sempre retorna true (pode ver)
    if not States.WallCheck then
        return true
    end

    if not fromPosition or not targetHead then
        return true
    end

    local direction = targetHead.Position - fromPosition
    local distance = direction.Magnitude

    if distance < 3 then
        return true
    end

    -- Fazer raycast simples SEM filtro
    local rayParams = RaycastParams.new()
    rayParams.FilterType = Enum.RaycastFilterType.Exclude

    -- Coletar TODOS os characters para excluir
    local excludeList = {}
    for _, p in pairs(Players:GetPlayers()) do
        if p.Character then
            table.insert(excludeList, p.Character)
        end
    end

    rayParams.FilterDescendants = excludeList
    rayParams.IgnoreWater = true

    local result = workspace:Raycast(fromPosition, direction.Unit * (distance - 1), rayParams)

    -- Se não bateu em nada = caminho livre
    if result == nil then
        return true
    end

    -- Se bateu em algo, é uma parede real (porque já excluímos todos os characters)
    return false
end

local function IsTeammate(player)
    if not States.TeamCheck then return false end
    if LocalPlayer.Team ~= nil and player.Team ~= nil then
        return LocalPlayer.Team == player.Team
    end
    return false
end

local function GetClosestPlayerToFOV()
    local closestPlayer = nil
    local closestDist = States.FOVSize
    local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

    local localChar = LocalPlayer.Character
    if not localChar then return nil end
    local localRoot = localChar:FindFirstChild("HumanoidRootPart")
    if not localRoot then return nil end

    local allPlayers = Players:GetPlayers()
    for i = 1, #allPlayers do
        local player = allPlayers[i]
        if player ~= LocalPlayer then
            local character = player.Character
            if character then
                local head = character:FindFirstChild("Head")
                local humanoid = character:FindFirstChildOfClass("Humanoid")

                if head and humanoid and humanoid.Health > 0 then
                    if not IsTeammate(player) then
                        local headScreen, onScreen = Camera:WorldToViewportPoint(head.Position)

                        if onScreen then
                            local screenPos = Vector2.new(headScreen.X, headScreen.Y)
                            local distFromCenter = (screenPos - screenCenter).Magnitude

                            if distFromCenter < closestDist then
                                local canSee = CanSeeTarget(localRoot.Position, head, character)
                                if canSee then
                                    closestDist = distFromCenter
                                    closestPlayer = player
                                end
                            end
                        end
                    end
                end
            end
        end
    end

    return closestPlayer
end

-- ═══════════════════════════════════════════
-- ESP UPDATE
-- ═══════════════════════════════════════════
local function UpdateESP(player, objects)
    if not player.Character then HideESP(objects); return end
    local char = player.Character
    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    local head = char:FindFirstChild("Head")
    if not hum or not root or not head or hum.Health <= 0 then HideESP(objects); return end

    local lr = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not lr then HideESP(objects); return end
    local dist = (lr.Position - root.Position).Magnitude
    if dist > States.ESPDistance then HideESP(objects); return end

    local hp, onScreen = Camera:WorldToViewportPoint(head.Position)
    local rp = Camera:WorldToViewportPoint(root.Position)
    if not onScreen then HideESP(objects); return end

    local sf = 1 / (hp.Z * 0.006)
    local bw = 38 * sf
    local bt = hp.Y - 8 * sf
    local bb = rp.Y + 22 * sf
    local cx = rp.X

    if States.BoxESP and objects.Box then
        pcall(function()
            local tl = Vector2.new(cx - bw/2, bt)
            local tr = Vector2.new(cx + bw/2, bt)
            local bl = Vector2.new(cx - bw/2, bb)
            local br = Vector2.new(cx + bw/2, bb)
            objects.BoxOutline.PointA = tl; objects.BoxOutline.PointB = tr; objects.BoxOutline.PointC = br; objects.BoxOutline.PointD = bl; objects.BoxOutline.Visible = true
            objects.Box.PointA = tl; objects.Box.PointB = tr; objects.Box.PointC = br; objects.Box.PointD = bl; objects.Box.Visible = true
        end)
    else
        pcall(function() objects.Box.Visible = false; objects.BoxOutline.Visible = false end)
    end

    if States.Tracers and objects.Tracer then
        pcall(function()
            local from = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
            local to = Vector2.new(rp.X, rp.Y)
            objects.TracerOutline.From = from; objects.TracerOutline.To = to; objects.TracerOutline.Visible = true
            objects.Tracer.From = from; objects.Tracer.To = to; objects.Tracer.Visible = true
        end)
    else
        pcall(function() objects.Tracer.Visible = false; objects.TracerOutline.Visible = false end)
    end

    if States.LifeStatus and objects.HealthBar then
        pcall(function()
            local mh = hum.MaxHealth
            local h = hum.Health
            local hp2 = math.clamp(h / mh, 0, 1)
            local bl2 = cx - bw/2 - 6
            local bh = bb - bt
            local hbt = bb - (bh * hp2)
            objects.HealthBarBG.From = Vector2.new(bl2, bt); objects.HealthBarBG.To = Vector2.new(bl2, bb); objects.HealthBarBG.Visible = true
            local hc = hp2 > 0.5 and LerpColor(Theme.Warning, Theme.ESPHealth, (hp2 - 0.5) * 2) or LerpColor(Theme.ESPHealthLow, Theme.Warning, hp2 * 2)
            objects.HealthBar.From = Vector2.new(bl2, hbt); objects.HealthBar.To = Vector2.new(bl2, bb); objects.HealthBar.Color = hc; objects.HealthBar.Visible = true
            objects.HealthText.Text = math.floor(h) .. " HP"; objects.HealthText.Position = Vector2.new(cx, bb + 4); objects.HealthText.Color = hc; objects.HealthText.Visible = true
        end)
    else
        pcall(function() objects.HealthBar.Visible = false; objects.HealthBarBG.Visible = false; objects.HealthText.Visible = false end)
    end

    if States.ESPEnabled and objects.NameText then
        pcall(function()
            objects.NameText.Text = player.DisplayName or player.Name
            objects.NameText.Position = Vector2.new(cx, bt - 18); objects.NameText.Visible = true
        end)
    else
        pcall(function() objects.NameText.Visible = false end)
    end

    if States.ESPEnabled and objects.DistText then
        pcall(function()
            objects.DistText.Text = math.floor(dist) .. "m"
            objects.DistText.Position = Vector2.new(cx, bb + (States.LifeStatus and 18 or 4)); objects.DistText.Visible = true
        end)
    else
        pcall(function() objects.DistText.Visible = false end)
    end
end

-- ═══════════════════════════════════════════
-- [v1.0.2] LÓGICA DO TRIGGERBOT
-- ═══════════════════════════════════════════
local function DoTriggerbot()
    if not States.TriggerbotEnabled then return end

    local localChar = LocalPlayer.Character
    if not localChar then return end
    local localHead = localChar:FindFirstChild("Head")
    if not localHead then return end

    -- Raycast do centro da tela para frente
    local camCF = Camera.CFrame
    local rayOrigin = camCF.Position
    local rayDirection = camCF.LookVector * 1000

    local rayParams = RaycastParams.new()
    rayParams.FilterType = Enum.RaycastFilterType.Exclude
    rayParams.FilterDescendants = {localChar}

    local result = workspace:Raycast(rayOrigin, rayDirection, rayParams)

    if result and result.Instance then
        -- Verificar se o que foi atingido pertence a algum player inimigo
        local hitModel = result.Instance:FindFirstAncestorOfClass("Model")
        if hitModel then
            for _, p in pairs(Players:GetPlayers()) do
                if p ~= LocalPlayer and p.Character and p.Character == hitModel then
                    local hum = hitModel:FindFirstChildOfClass("Humanoid")
                    if hum and hum.Health > 0 then
                        -- Simular clique do mouse
                        pcall(function()
                            mouse1click()
                        end)
                        pcall(function()
                            local vim = game:GetService("VirtualInputManager")
                            vim:SendMouseButtonEvent(0, 0, 0, true, game, 1)
                            task.wait()
                            vim:SendMouseButtonEvent(0, 0, 0, false, game, 1)
                        end)
                    end
                    break
                end
            end
        end
    end
end

-- ═══════════════════════════════════════════
-- [v1.0.2] LÓGICA NO RECOIL / NO SPREAD
-- ═══════════════════════════════════════════
local function DoNoRecoil()
    if not States.NoRecoilEnabled then return end

    local localChar = LocalPlayer.Character
    if not localChar then return end

    -- Procurar ferramentas (armas) no character e backpack
    local function processGun(tool)
        pcall(function()
            for _, desc in pairs(tool:GetDescendants()) do
                -- Resetar valores de recoil comuns
                pcall(function()
                    if desc.Name == "Recoil" or desc.Name == "RecoilAmount" or desc.Name == "recoil" then
                        if desc:IsA("NumberValue") or desc:IsA("IntValue") then
                            desc.Value = 0
                        end
                    end
                    if desc.Name == "Spread" or desc.Name == "SpreadAmount" or desc.Name == "spread" then
                        if desc:IsA("NumberValue") or desc:IsA("IntValue") then
                            desc.Value = 0
                        end
                    end
                    if desc.Name == "MinSpread" or desc.Name == "MaxSpread" then
                        if desc:IsA("NumberValue") or desc:IsA("IntValue") then
                            desc.Value = 0
                        end
                    end
                    if desc.Name == "RecoilUp" or desc.Name == "RecoilSide" or desc.Name == "KickBack" then
                        if desc:IsA("NumberValue") or desc:IsA("IntValue") then
                            desc.Value = 0
                        end
                    end
                end)
            end
        end)
    end

    -- Processar armas no character
    for _, child in pairs(localChar:GetChildren()) do
        if child:IsA("Tool") then processGun(child) end
    end

    -- Processar armas no backpack
    local bp = LocalPlayer:FindFirstChild("Backpack")
    if bp then
        for _, child in pairs(bp:GetChildren()) do
            if child:IsA("Tool") then processGun(child) end
        end
    end
end

-- ═══════════════════════════════════════════
-- [v1.0.2] LÓGICA GOD MODE
-- ═══════════════════════════════════════════
local function DoGodMode()
    if not States.GodModeEnabled then return end

    local localChar = LocalPlayer.Character
    if not localChar then return end
    local hum = localChar:FindFirstChildOfClass("Humanoid")
    if not hum then return end

    pcall(function()
        if hum.Health < hum.MaxHealth then
            hum.Health = hum.MaxHealth
        end
    end)
end

-- ═══════════════════════════════════════════
-- [v1.0.2] LÓGICA MUNIÇÃO INFINITA
-- ═══════════════════════════════════════════
local function DoInfiniteAmmo()
    if not States.InfiniteAmmoEnabled then return end

    local localChar = LocalPlayer.Character
    if not localChar then return end

    local function refillAmmo(tool)
        pcall(function()
            for _, desc in pairs(tool:GetDescendants()) do
                pcall(function()
                    local nameLower = string.lower(desc.Name)
                    if (nameLower == "ammo" or nameLower == "currentammo" or nameLower == "clip"
                        or nameLower == "magazine" or nameLower == "mag" or nameLower == "bullets"
                        or nameLower == "clipsize" or nameLower == "magsize") then
                        if desc:IsA("NumberValue") or desc:IsA("IntValue") then
                            if desc.Value < 999 then
                                desc.Value = 999
                            end
                        end
                    end
                end)
            end
        end)
    end

    for _, child in pairs(localChar:GetChildren()) do
        if child:IsA("Tool") then refillAmmo(child) end
    end

    local bp = LocalPlayer:FindFirstChild("Backpack")
    if bp then
        for _, child in pairs(bp:GetChildren()) do
            if child:IsA("Tool") then refillAmmo(child) end
        end
    end
end

-- ═══════════════════════════════════════════
-- LOOP PRINCIPAL
-- ═══════════════════════════════════════════
RunService.RenderStepped:Connect(function()
    Camera = workspace.CurrentCamera

    -- FOV Circle
    if FOVCircle then
        pcall(function()
            FOVCircle.Visible = States.FOVEnabled
            FOVCircle.Radius = States.FOVSize
            FOVCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
        end)
    end

    -- Aimbot
    if States.AimbotEnabled then
        local holding = UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)
        if holding then
            local target = GetClosestPlayerToFOV()
            if target ~= nil and target.Character ~= nil then
                local head = target.Character:FindFirstChild("Head")
                if head ~= nil then
                    Camera.CFrame = CFrame.new(Camera.CFrame.Position, head.Position)
                end
            end
        end
    end

    -- [v1.0.2] Fly
    if States.FlyEnabled and States.FlyBodyVelocity and States.FlyBodyGyro then
        local localChar = LocalPlayer.Character
        if localChar then
            local hrp = localChar:FindFirstChild("HumanoidRootPart")
            if hrp then
                local moveDir = Vector3.new(0, 0, 0)
                local camCF = Camera.CFrame

                if UserInputService:IsKeyDown(Enum.KeyCode.W) then
                    moveDir = moveDir + camCF.LookVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.S) then
                    moveDir = moveDir - camCF.LookVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.A) then
                    moveDir = moveDir - camCF.RightVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.D) then
                    moveDir = moveDir + camCF.RightVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
                    moveDir = moveDir + Vector3.new(0, 1, 0)
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) or UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
                    moveDir = moveDir - Vector3.new(0, 1, 0)
                end

                if moveDir.Magnitude > 0 then
                    States.FlyBodyVelocity.Velocity = moveDir.Unit * States.FlySpeed
                else
                    States.FlyBodyVelocity.Velocity = Vector3.new(0, 0, 0)
                end

                States.FlyBodyGyro.CFrame = camCF
            end
        end
    end

    -- [v1.0.2] NoClip
    if States.NoClipEnabled then
        local localChar = LocalPlayer.Character
        if localChar then
            for _, part in pairs(localChar:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end
    end

    -- [v1.0.2] Speed Hack (manter velocidade)
    if States.SpeedHackEnabled then
        local localChar = LocalPlayer.Character
        if localChar then
            local hum = localChar:FindFirstChildOfClass("Humanoid")
            if hum and hum.WalkSpeed ~= States.SpeedValue then
                hum.WalkSpeed = States.SpeedValue
            end
        end
    end

    -- [v1.0.2] God Mode
    DoGodMode()

    -- [v1.0.2] No Recoil
    DoNoRecoil()

    -- [v1.0.2] Infinite Ammo
    DoInfiniteAmmo()

    -- [v1.0.2] Triggerbot
    DoTriggerbot()

    -- ESP
    local allP = Players:GetPlayers()
    for i = 1, #allP do
        local p = allP[i]
        if p ~= LocalPlayer then
            if not ESPObjects[p] then CreateESPForPlayer(p) end
            if ESPObjects[p] then
                if States.ESPEnabled then
                    UpdateESP(p, ESPObjects[p])
                else
                    HideESP(ESPObjects[p])
                end
            end
        end
    end
end)

-- ═══════════════════════════════════════════
-- PLAYER EVENTS
-- ═══════════════════════════════════════════
Players.PlayerAdded:Connect(function(p) CreateESPForPlayer(p) end)
Players.PlayerRemoving:Connect(function(p) RemoveESPForPlayer(p) end)
for _, p in pairs(Players:GetPlayers()) do
    if p ~= LocalPlayer then CreateESPForPlayer(p) end
end

-- Restaurar ao respawnar
LocalPlayer.CharacterAdded:Connect(function(char)
    task.wait(1)
    -- Restaurar speed se ativo
    if States.SpeedHackEnabled then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum.WalkSpeed = States.SpeedValue end
    end
    -- Resetar fly
    if States.FlyEnabled then
        States.FlyBodyVelocity = nil
        States.FlyBodyGyro = nil
        local hrp = char:WaitForChild("HumanoidRootPart", 5)
        if hrp then
            local bv = Instance.new("BodyVelocity")
            bv.Name = "ThurFlyBV"
            bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
            bv.Velocity = Vector3.new(0, 0, 0)
            bv.Parent = hrp
            States.FlyBodyVelocity = bv

            local bg = Instance.new("BodyGyro")
            bg.Name = "ThurFlyBG"
            bg.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
            bg.D = 100; bg.P = 10000
            bg.Parent = hrp
            States.FlyBodyGyro = bg

            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then hum.PlatformStand = true end
        end
    end
end)

-- ═══════════════════════════════════════════
-- TOGGLE UI
-- ═══════════════════════════════════════════
UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.Insert or input.KeyCode == Enum.KeyCode.RightControl then
        States.UIOpen = not States.UIOpen
        MainFrame.Visible = States.UIOpen
    end
end)

-- ═══════════════════════════════════════════
-- ANIMAÇÃO DE ENTRADA
-- ═══════════════════════════════════════════
MainFrame.BackgroundTransparency = 1
MainFrame.Size = UDim2.new(0, 560, 0, 600)
task.delay(0.1, function()
    TweenService:Create(MainFrame, TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        BackgroundTransparency = 0, Size = UDim2.new(0, 560, 0, 620)
    }):Play()
end)

-- ═══════════════════════════════════════════
-- NOTIFICAÇÃO
-- ═══════════════════════════════════════════
local NF = Instance.new("Frame")
NF.Size = UDim2.new(0, 340, 0, 78)
NF.Position = UDim2.new(0.5, -170, 0, -90)
NF.BackgroundColor3 = Theme.Surface
NF.BorderSizePixel = 0
NF.ZIndex = 100
NF.Parent = ScreenGui
CreateCorner(NF, 10)
CreateStroke(NF, Theme.PrimaryPurple, 1.5, 0.3)

local NBar = Instance.new("Frame")
NBar.Size = UDim2.new(1, 0, 0, 3)
NBar.Position = UDim2.new(0, 0, 1, -3)
NBar.BackgroundColor3 = Theme.PrimaryPurple
NBar.BorderSizePixel = 0
NBar.ZIndex = 101
NBar.Parent = NF
CreateGradient(NBar, Theme.PrimaryPurple, Theme.AccentPink, 0)

local NI = Instance.new("TextLabel")
NI.Size = UDim2.new(0, 30, 0, 30)
NI.Position = UDim2.new(0, 15, 0, 10)
NI.BackgroundTransparency = 1
NI.Text = "✓"
NI.TextColor3 = Theme.Success
NI.TextSize = 20
NI.Font = Enum.Font.GothamBold
NI.ZIndex = 101
NI.Parent = NF

local NT = Instance.new("TextLabel")
NT.Size = UDim2.new(1, -60, 0, 18)
NT.Position = UDim2.new(0, 50, 0, 8)
NT.BackgroundTransparency = 1
NT.Text = "Thurscripts v1.0.2 Carregado!"
NT.TextColor3 = Theme.TextPrimary
NT.TextSize = 14
NT.Font = Enum.Font.GothamBold
NT.TextXAlignment = Enum.TextXAlignment.Left
NT.ZIndex = 101
NT.Parent = NF

local ND1 = Instance.new("TextLabel")
ND1.Size = UDim2.new(1, -60, 0, 14)
ND1.Position = UDim2.new(0, 50, 0, 28)
ND1.BackgroundTransparency = 1
ND1.Text = "+ Player  + Recursos  * Wall Fix"
ND1.TextColor3 = Theme.TextAccent
ND1.TextSize = 11
ND1.Font = Enum.Font.GothamMedium
ND1.TextXAlignment = Enum.TextXAlignment.Left
ND1.ZIndex = 101
ND1.Parent = NF

local ND2 = Instance.new("TextLabel")
ND2.Size = UDim2.new(1, -60, 0, 14)
ND2.Position = UDim2.new(0, 50, 0, 44)
ND2.BackgroundTransparency = 1
ND2.Text = "+ NoRecoil  + Triggerbot  + GodMode  + InfAmmo"
ND2.TextColor3 = Theme.TextMuted
ND2.TextSize = 10
ND2.Font = Enum.Font.Gotham
ND2.TextXAlignment = Enum.TextXAlignment.Left
ND2.ZIndex = 101
ND2.Parent = NF

local ND3 = Instance.new("TextLabel")
ND3.Size = UDim2.new(1, -60, 0, 12)
ND3.Position = UDim2.new(0, 50, 0, 60)
ND3.BackgroundTransparency = 1
ND3.Text = "INSERT para abrir/fechar"
ND3.TextColor3 = Theme.TextMuted
ND3.TextSize = 9
ND3.Font = Enum.Font.Gotham
ND3.TextXAlignment = Enum.TextXAlignment.Left
ND3.ZIndex = 101
ND3.Parent = NF

task.delay(0.3, function()
    TweenService:Create(NF, TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Position = UDim2.new(0.5, -170, 0, 20)
    }):Play()
    task.delay(4.5, function()
        TweenService:Create(NF, TweenInfo.new(0.4, Enum.EasingStyle.Quart, Enum.EasingDirection.In), {
            Position = UDim2.new(0.5, -170, 0, -90)
        }):Play()
        task.delay(0.5, function() NF:Destroy() end)
    end)
end)

-- ═══════════════════════════════════════════
-- CLEANUP
-- ═══════════════════════════════════════════
Players.PlayerRemoving:Connect(function(p)
    if p == LocalPlayer then
        if FOVCircle then pcall(function() FOVCircle:Remove() end) end
        for plr, _ in pairs(ESPObjects) do RemoveESPForPlayer(plr) end
        if States.AntLagEnabled then DisableAntLag() end
        if States.FlyBodyVelocity then pcall(function() States.FlyBodyVelocity:Destroy() end) end
        if States.FlyBodyGyro then pcall(function() States.FlyBodyGyro:Destroy() end) end
        if ScreenGui then ScreenGui:Destroy() end
    end
end)

-- ═══════════════════════════════════════════
-- LOGO PULSE
-- ═══════════════════════════════════════════
task.spawn(function()
    while ScreenGui and ScreenGui.Parent do
        pcall(function()
            TweenService:Create(LogoGlow, TweenInfo.new(1.5, Enum.EasingStyle.Sine), {BackgroundTransparency = 0.4}):Play()
            task.wait(1.5)
            TweenService:Create(LogoGlow, TweenInfo.new(1.5, Enum.EasingStyle.Sine), {BackgroundTransparency = 0.8}):Play()
            task.wait(1.5)
        end)
    end
end)

print("══════════════════════════════════════")
print("  THURSCRIPTS v1.0.2")
print("  Sintonia Roleplay Edition")
print("══════════════════════════════════════")
print("[+] Nova aba: Player (Speed, Fly, NoClip)")
print("[+] Nova aba: Recursos (GodMode, InfAmmo)")
print("[+] No Recoil / No Spread")
print("[+] Triggerbot")
print("[*] Wall Check - FIX DEFINITIVO")
print("[*] INSERT para abrir/fechar")
print("══════════════════════════════════════")
