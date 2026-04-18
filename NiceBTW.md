--[[
    Hasession Hub v1.0 — HButton Fixed
    - HButton не застревает
    - Плавное закрытие/открытие
    - Блюр плавный
    - Эмодзи только в сайдбаре и Basic
]]

local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local TS = game:GetService("TweenService")
local RunService = game:GetService("RunService")

-- Чистка
local old = LP.PlayerGui:FindFirstChild("HasessionGui")
if old then old:Destroy() end

local Gui = Instance.new("ScreenGui", LP.PlayerGui)
Gui.Name = "HasessionGui"
Gui.ResetOnSpawn = false
Gui.IgnoreGuiInset = true
Gui.DisplayOrder = 100

-- ==================== ФРЕЙМВОРК ====================
local Framework = {}
Framework.Accent = Color3.fromRGB(0, 200, 255)
Framework.IsMobile = UIS.TouchEnabled and not UIS.KeyboardEnabled

function Framework:Create(className, properties)
    local element = Instance.new(className)
    local parent = properties.Parent
    properties.Parent = nil
    
    for prop, value in pairs(properties) do
        if prop ~= "Children" and prop ~= "CornerRadius" and prop ~= "Stroke" then
            element[prop] = value
        end
    end
    
    if className == "TextButton" or className == "TextBox" then
        if properties.BackgroundTransparency == nil or properties.BackgroundTransparency < 1 then
            element.BackgroundTransparency = 0
        end
    end
    
    if properties.CornerRadius then
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0, properties.CornerRadius)
        corner.Parent = element
    end
    
    if properties.Stroke then
        local stroke = Instance.new("UIStroke")
        stroke.Color = properties.Stroke.Color or Framework.Accent
        stroke.Thickness = properties.Stroke.Thickness or 1
        stroke.Transparency = properties.Stroke.Transparency or 0
        stroke.Parent = element
    end
    
    if properties.Children then
        for _, childData in ipairs(properties.Children) do
            local child = self:Create(childData[1], childData[2])
            child.Parent = element
        end
    end
    
    if parent then
        element.Parent = parent
    end
    
    return element
end

-- ==================== НАСТРОЙКИ ====================
local Settings = { Fly = false, Noclip = false, InfJump = false }
local blur = Instance.new("BlurEffect", game:GetService("Lighting"))
blur.Size = 8
blur.Enabled = true

-- ==================== АДАПТИВНЫЕ РАЗМЕРЫ ====================
local viewport = workspace.CurrentCamera.ViewportSize
local scaleX = math.clamp(viewport.X / 1920, 0.8, 1.2)
local scaleY = math.clamp(viewport.Y / 1080, 0.8, 1.2)
local windowWidth = math.clamp(560 * scaleX, 350, 650)
local windowHeight = math.clamp(380 * scaleY, 260, 480)

-- ==================== ГЛАВНОЕ ОКНО ====================
local Main = Framework:Create("Frame", {
    BackgroundColor3 = Color3.fromRGB(25, 30, 40),
    BackgroundTransparency = 0.3,
    Size = UDim2.fromOffset(windowWidth, windowHeight),
    Position = UDim2.fromScale(0.5, 0.5),
    AnchorPoint = Vector2.new(0.5, 0.5),
    ClipsDescendants = true,
    ZIndex = 10,
    CornerRadius = 14,
    Stroke = { Color = Color3.fromRGB(60, 70, 90), Thickness = 1, Transparency = 0.5 },
    Parent = Gui
})

local SizeConstraint = Instance.new("UISizeConstraint", Main)
SizeConstraint.MinSize = Vector2.new(300, 250)
SizeConstraint.MaxSize = Vector2.new(650, 500)

-- ==================== ХЕДЕР ====================
local Header = Framework:Create("TextButton", {
    Size = UDim2.new(1, 0, 0, 55),
    BackgroundTransparency = 1,
    Text = "",
    AutoButtonColor = false,
    ZIndex = 5,
    Parent = Main
})

local Avatar = Framework:Create("ImageLabel", {
    Position = UDim2.fromOffset(12, 8),
    Size = UDim2.fromOffset(38, 38),
    BackgroundColor3 = Color3.fromRGB(35, 40, 50),
    CornerRadius = 19,
    Stroke = { Color = Framework.Accent, Thickness = 1.5 },
    ZIndex = 15,
    Parent = Header
})

task.spawn(function()
    local success, image = pcall(function()
        return Players:GetUserThumbnailAsync(LP.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size48x48)
    end)
    if success then
        Avatar.Image = image
    end
end)

-- DisplayName
Framework:Create("TextLabel", {
    Position = UDim2.fromOffset(58, 8),
    Size = UDim2.new(0, 140, 0, 20),
    Text = LP.DisplayName,
    Font = Enum.Font.GothamBold,
    TextColor3 = Color3.new(1, 1, 1),
    TextSize = 14,
    BackgroundTransparency = 1,
    TextXAlignment = 0,
    ZIndex = 15,
    Parent = Header
})

-- Username
Framework:Create("TextLabel", {
    Position = UDim2.fromOffset(58, 28),
    Size = UDim2.new(0, 140, 0, 16),
    Text = "@" .. LP.Name,
    Font = Enum.Font.Gotham,
    TextColor3 = Color3.fromRGB(180, 180, 190),
    TextSize = 11,
    BackgroundTransparency = 1,
    TextXAlignment = 0,
    ZIndex = 15,
    Parent = Header
})

-- План Basic
local PlanBadge = Framework:Create("Frame", {
    Position = UDim2.new(0, 58 + 38 + 8, 0, 12),
    Size = UDim2.fromOffset(60, 20),
    BackgroundColor3 = Framework.Accent,
    CornerRadius = 10,
    ZIndex = 15,
    Parent = Header
})

local PlanText = Instance.new("TextLabel")
PlanText.Size = UDim2.new(1, 0, 1, 0)
PlanText.Text = "✨BASIC"
PlanText.Font = Enum.Font.GothamBold
PlanText.TextColor3 = Color3.new(0, 0, 0)
PlanText.TextSize = 10
PlanText.BackgroundTransparency = 1
PlanText.ZIndex = 16
PlanText.Parent = PlanBadge

-- Версия
Framework:Create("TextLabel", {
    Position = UDim2.new(0, 58 + 38 + 8 + 60 + 5, 0, 12),
    Size = UDim2.fromOffset(50, 20),
    Text = "v1.0",
    Font = Enum.Font.GothamMedium,
    TextColor3 = Framework.Accent,
    TextSize = 11,
    BackgroundTransparency = 1,
    ZIndex = 15,
    Parent = Header
})

-- Кнопка закрытия
local CloseBtn = Framework:Create("TextButton", {
    Position = UDim2.new(1, -30, 0, 12),
    Size = UDim2.fromOffset(22, 22),
    BackgroundColor3 = Color3.fromRGB(255, 60, 60),
    Text = "✕",
    Font = Enum.Font.GothamBold,
    TextColor3 = Color3.new(1, 1, 1),
    Active = true,
    ZIndex = 25,
    CornerRadius = 11,
    Parent = Header
})

-- ==================== КНОПКА H ====================
local HButton = Framework:Create("TextButton", {
    Size = UDim2.fromOffset(Framework.IsMobile and 45 or 50, Framework.IsMobile and 45 or 50),
    Position = UDim2.new(1, Framework.IsMobile and -55 or -70, 1, Framework.IsMobile and -55 or -70),
    BackgroundColor3 = Color3.fromRGB(0, 180, 255),
    Text = "H",
    Font = Enum.Font.GothamBold,
    TextColor3 = Color3.new(1, 1, 1),
    TextSize = Framework.IsMobile and 24 or 28,
    Visible = true,
    Active = true,
    ZIndex = 200,
    CornerRadius = 25,
    Stroke = { 
        Color = Color3.fromRGB(40, 40, 40),
        Thickness = 2, 
        Transparency = 0.3
    },
    Parent = Gui
})

-- ==================== DRAG ====================
local dragging = false
local dragStartPos = nil
local startPos = nil
local targetPosition = Main.Position

Header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStartPos = input.Position
        startPos = Main.Position
        targetPosition = Main.Position
    end
end)

UIS.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStartPos
        local newPos = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y
        )
        targetPosition = newPos
        Main.Position = Main.Position:Lerp(targetPosition, 0.25)
    end
end)

UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

-- ==================== ЛОГИКА H (БЕЗ ЗАСТРЕВАНИЙ) ====================
local isHidden = false
local hideTween = nil
local showTween = nil
local hTween = nil

local function hideMenu()
    if isHidden then return end
    isHidden = true
    
    -- Останавливаем все текущие анимации
    if hideTween then hideTween:Cancel() end
    if showTween then showTween:Cancel() end
    if hTween then hTween:Cancel() end
    
    -- Крутим H
    HButton.Rotation = 0
    hTween = TS:Create(HButton, TweenInfo.new(0.6, Enum.EasingStyle.Back), { Rotation = 360 })
    hTween:Play()
    
    -- Плавно выключаем блюр
    TS:Create(blur, TweenInfo.new(0.4), { Size = 0 }):Play()
    
    -- Анимация закрытия
    hideTween = TS:Create(Main, TweenInfo.new(0.6, Enum.EasingStyle.Back, Enum.EasingDirection.In), { 
        Size = UDim2.fromOffset(windowWidth, 0)
    })
    hideTween.Completed:Connect(function()
        Main.Visible = false
        blur.Enabled = false
        hideTween = nil
    end)
    hideTween:Play()
end

local function showMenu()
    if not isHidden then return end
    isHidden = false
    
    -- Останавливаем все текущие анимации
    if hideTween then hideTween:Cancel() end
    if showTween then showTween:Cancel() end
    if hTween then hTween:Cancel() end
    
    Main.Visible = true
    Main.Size = UDim2.fromOffset(windowWidth, 0)
    blur.Enabled = true
    blur.Size = 0
    
    -- Крутим H обратно
    hTween = TS:Create(HButton, TweenInfo.new(0.6, Enum.EasingStyle.Back), { Rotation = 0 })
    hTween:Play()
    
    -- Плавно включаем блюр
    TS:Create(blur, TweenInfo.new(0.4), { Size = 8 }):Play()
    
    -- Анимация открытия
    showTween = TS:Create(Main, TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out), { 
        Size = UDim2.fromOffset(windowWidth, windowHeight)
    })
    showTween.Completed:Connect(function()
        showTween = nil
    end)
    showTween:Play()
end

HButton.MouseButton1Click:Connect(function()
    if isHidden then
        showMenu()
    else
        hideMenu()
    end
end)

-- ==================== САЙДБАР ====================
local sidebarWidth = Framework.IsMobile and 95 or 130
local Sidebar = Framework:Create("Frame", {
    Position = UDim2.fromOffset(10, 62),
    Size = UDim2.new(0, sidebarWidth, 1, -72),
    BackgroundTransparency = 1,
    Parent = Main,
    Children = {{"UIListLayout", { Padding = UDim.new(0, 8) }}}
})

local Content = Framework:Create("ScrollingFrame", {
    Position = UDim2.fromOffset(sidebarWidth + 20, 62),
    Size = UDim2.new(1, -sidebarWidth - 35, 1, -72),
    BackgroundTransparency = 1,
    ScrollBarThickness = 0,
    AutomaticCanvasSize = Enum.AutomaticSize.Y,
    Parent = Main,
    Children = {
        {"UIListLayout", { Padding = UDim.new(0, 8) }},
        {"UIPadding", { PaddingTop = UDim.new(0, 5) }}
    }
})

-- ==================== ВКЛАДКИ ====================
local Tabs = {}
local CurrentTab = nil

local function switchTab(tabFrame, btn)
    if CurrentTab == tabFrame then return end
    if CurrentTab then CurrentTab.Visible = false end
    tabFrame.Visible = true
    CurrentTab = tabFrame
    
    for _, t in pairs(Tabs) do
        TS:Create(t.B, TweenInfo.new(0.2), {
            BackgroundColor3 = Color3.fromRGB(60, 70, 90),
            TextColor3 = Color3.new(1, 1, 1)
        }):Play()
    end
    TS:Create(btn, TweenInfo.new(0.2), {
        BackgroundColor3 = Framework.Accent,
        TextColor3 = Color3.new(0, 0, 0)
    }):Play()
end

local function createTab(name, icon, active)
    local tabFrame = Framework:Create("Frame", {
        Size = UDim2.new(1, 0, 0, 0),
        BackgroundTransparency = 1,
        Visible = false,
        AutomaticSize = Enum.AutomaticSize.Y,
        Parent = Content,
        Children = {{"UIListLayout", { Padding = UDim.new(0, 8) }}}
    })
    
    local btn = Framework:Create("TextButton", {
        Size = UDim2.new(1, 0, 0, 36),
        BackgroundColor3 = Color3.fromRGB(60, 70, 90),
        Text = icon .. name,
        Font = Enum.Font.GothamMedium,
        TextColor3 = Color3.new(1, 1, 1),
        TextSize = 13,
        CornerRadius = 17,
        ZIndex = 20,
        Parent = Sidebar
    })
    
    btn.MouseButton1Click:Connect(function() switchTab(tabFrame, btn) end)
    Tabs[name] = {F = tabFrame, B = btn}
    
    if active then
        tabFrame.Visible = true
        CurrentTab = tabFrame
        btn.BackgroundColor3 = Framework.Accent
        btn.TextColor3 = Color3.new(0, 0, 0)
    end
    
    return tabFrame
end

-- ==================== ТОГГЛЫ ====================
local function addToggle(tab, title, desc, callback)
    local card = Framework:Create("TextButton", {
        Size = UDim2.new(1, 0, 0, 56),
        BackgroundColor3 = Color3.fromRGB(55, 65, 85),
        Text = "",
        AutoButtonColor = false,
        CornerRadius = 10,
        ZIndex = 20,
        Parent = tab
    })
    
    local info = Framework:Create("Frame", {
        Size = UDim2.new(1, -50, 1, 0),
        BackgroundTransparency = 1,
        Parent = card,
        Children = {
            {"UIListLayout", { VerticalAlignment = Enum.VerticalAlignment.Center, Padding = UDim.new(0, 2) }},
            {"UIPadding", { PaddingLeft = UDim.new(0, 15) }}
        }
    })
    
    Framework:Create("TextLabel", {
        Size = UDim2.new(1, 0, 0, 18),
        Text = title,
        Font = Enum.Font.GothamBold,
        TextColor3 = Color3.new(1, 1, 1),
        TextSize = 14,
        BackgroundTransparency = 1,
        TextXAlignment = 0,
        ZIndex = 21,
        Parent = info
    })
    
    Framework:Create("TextLabel", {
        Size = UDim2.new(1, 0, 0, 14),
        Text = desc,
        Font = Enum.Font.Gotham,
        TextColor3 = Color3.fromRGB(210, 210, 220),
        TextSize = 10,
        BackgroundTransparency = 1,
        TextXAlignment = 0,
        ZIndex = 21,
        Parent = info
    })
    
    local sw = Framework:Create("Frame", {
        Position = UDim2.new(1, -42, 0.5, -8),
        Size = UDim2.fromOffset(28, 16),
        BackgroundColor3 = Color3.fromRGB(110, 115, 125),
        CornerRadius = 8,
        ZIndex = 22,
        Parent = card
    })
    
    local dot = Framework:Create("Frame", {
        Position = UDim2.fromOffset(2, 2),
        Size = UDim2.fromOffset(12, 12),
        BackgroundColor3 = Color3.new(1, 1, 1),
        CornerRadius = 6,
        ZIndex = 23,
        Parent = sw
    })
    
    local en = false
    card.MouseButton1Click:Connect(function()
        en = not en
        TS:Create(sw, TweenInfo.new(0.2), { BackgroundColor3 = en and Framework.Accent or Color3.fromRGB(110, 115, 125) }):Play()
        TS:Create(dot, TweenInfo.new(0.2), { Position = en and UDim2.fromOffset(14, 2) or UDim2.fromOffset(2, 2) }):Play()
        callback(en)
    end)
end

-- ==================== СОЗДАНИЕ ВКЛАДОК ====================
local home = createTab("Home", "🏠", true)
local executor = createTab("Executor", "💻", false)
local hubs = createTab("Hubs", "📁", false)
local settings = createTab("Settings", "⚙️", false)

addToggle(home, "Fly", "Ability to fly around map", function(v) Settings.Fly = v end)
addToggle(home, "Noclip", "Walk through objects", function(v) Settings.Noclip = v end)
addToggle(home, "Inf Jump", "Jump in the air anytime", function(v) Settings.InfJump = v end)

-- Executor
local ScriptBox = Framework:Create("TextBox", {
    Size = UDim2.new(1, 0, 0, 100),
    BackgroundColor3 = Color3.fromRGB(35, 40, 50),
    TextColor3 = Color3.new(1, 1, 1),
    Font = Enum.Font.Code,
    TextSize = 12,
    Text = "-- Paste script here",
    MultiLine = true,
    ClearTextOnFocus = false,
    Active = true,
    CornerRadius = 8,
    Parent = executor
})

Framework:Create("TextButton", {
    Size = UDim2.new(1, 0, 0, 32),
    BackgroundColor3 = Framework.Accent,
    Text = "Execute",
    Font = Enum.Font.GothamBold,
    TextColor3 = Color3.new(0, 0, 0),
    TextSize = 13,
    CornerRadius = 8,
    ZIndex = 20,
    Parent = executor
}).MouseButton1Click:Connect(function()
    pcall(function() loadstring(ScriptBox.Text)() end)
end)

-- Hubs
Framework:Create("TextBox", {
    Size = UDim2.new(1, 0, 0, 32),
    BackgroundColor3 = Color3.fromRGB(35, 40, 50),
    TextColor3 = Color3.new(1, 1, 1),
    Font = Enum.Font.Gotham,
    TextSize = 12,
    PlaceholderText = "🔍 Search scripts...",
    PlaceholderColor3 = Color3.fromRGB(150, 150, 150),
    Active = true,
    CornerRadius = 8,
    Parent = hubs
})

Framework:Create("TextLabel", {
    Size = UDim2.new(1, 0, 0, 60),
    Text = "📁 Scripts coming soon...",
    Font = Enum.Font.GothamMedium,
    TextColor3 = Color3.fromRGB(180, 180, 180),
    TextSize = 12,
    BackgroundTransparency = 1,
    ZIndex = 20,
    Parent = hubs
})

addToggle(settings, "Dark Theme", "Switch between dark and light", function(v)
    print("Theme:", v and "Dark" or "Light")
end)

-- ==================== NOCLIP ====================
local cachedParts = {}
local characterConnections = {}

local function applyNoclip()
    for _, part in pairs(cachedParts) do
        part.CanCollide = false
    end
end

local function cacheCharacterParts(character)
    task.wait(0.05)
    local hrp = character:WaitForChild("HumanoidRootPart", 2)
    if not hrp then return end
    
    cachedParts = {}
    for _, part in pairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            table.insert(cachedParts, part)
        end
    end
    
    if characterConnections[character] then
        characterConnections[character]:Disconnect()
    end
    characterConnections[character] = character.DescendantAdded:Connect(function(descendant)
        if descendant:IsA("BasePart") then
            table.insert(cachedParts, descendant)
            if Settings.Noclip and character and character:FindFirstChild("Humanoid") and character.Humanoid.Health > 0 then
                descendant.CanCollide = false
            end
        end
    end)
    
    if Settings.Noclip and character.Humanoid.Health > 0 then
        applyNoclip()
    end
end

LP.CharacterAdded:Connect(cacheCharacterParts)

if LP.Character then
    task.spawn(function() cacheCharacterParts(LP.Character) end)
end

local noclipConnection = RunService.Stepped:Connect(function()
    if Settings.Noclip and LP.Character and LP.Character:FindFirstChild("Humanoid") and LP.Character.Humanoid.Health > 0 then
        applyNoclip()
    end
end)

-- ==================== INF JUMP ====================
local jumpConnection = UIS.JumpRequest:Connect(function()
    if Settings.InfJump and LP.Character and LP.Character:FindFirstChild("Humanoid") then
        LP.Character.Humanoid:ChangeState(3)
    end
end)

-- ==================== CLEANUP ====================
CloseBtn.MouseButton1Click:Connect(function()
    noclipConnection:Disconnect()
    jumpConnection:Disconnect()
    for _, conn in pairs(characterConnections) do
        conn:Disconnect()
    end
    blur:Destroy()
    HButton:Destroy()
    Gui:Destroy()
end)

-- ==================== АНИМАЦИЯ ПОЯВЛЕНИЯ ПРИ ЗАПУСКЕ ====================
Main.Visible = true
Main.Size = UDim2.fromOffset(windowWidth, 0)
Main.BackgroundTransparency = 1
TS:Create(Main, TweenInfo.new(0.5, Enum.EasingStyle.Back), { 
    Size = UDim2.fromOffset(windowWidth, windowHeight),
    BackgroundTransparency = 0.3
}):Play()
