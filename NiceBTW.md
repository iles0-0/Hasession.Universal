-- Hasession v1.1 — Smooth H Animation + ZIndex Fixes + Full Logic

local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local TS = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")

-- Чистка старого GUI
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
local Settings = {
    Fly = false,
    Noclip = false,
    InfJump = false,
    AntiAFK = false,
    WalkSpeed = 16,
    JumpPower = 50
}
local blur = Instance.new("BlurEffect", game:GetService("Lighting"))
blur.Size = 8
blur.Enabled = true

-- ==================== АДАПТИВНЫЕ РАЗМЕРЫ ====================
local viewport = workspace.CurrentCamera.ViewportSize
local scaleX = math.clamp(viewport.X / 1920, 0.8, 1.2)
local scaleY = math.clamp(viewport.Y / 1080, 0.8, 1.2)
local W = math.clamp(560 * scaleX, 350, 650)
local H = math.clamp(380 * scaleY, 260, 480)

-- ==================== ГЛАВНОЕ ОКНО ====================
local Main = Framework:Create("Frame", {
    BackgroundColor3 = Color3.fromRGB(25, 30, 40),
    BackgroundTransparency = 0.3,
    Size = UDim2.fromOffset(W, H),
    Position = UDim2.fromScale(0.5, 0.5),
    AnchorPoint = Vector2.new(0.5, 0.5),
    ClipsDescendants = true,
    ZIndex = 1,
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
    ZIndex = 10,
    Parent = Main
})

local Avatar = Framework:Create("ImageLabel", {
    Position = UDim2.fromOffset(12, 8),
    Size = UDim2.fromOffset(38, 38),
    BackgroundColor3 = Color3.fromRGB(35, 40, 50),
    CornerRadius = 19,
    Stroke = { Color = Framework.Accent, Thickness = 1.5 },
    ZIndex = 11,
    Parent = Header
})

task.spawn(function()
    local success, image = pcall(function()
        return Players:GetUserThumbnailAsync(LP.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size48x48)
    end)
    if success then Avatar.Image = image end
end)

Framework:Create("TextLabel", {
    Position = UDim2.fromOffset(58, 8),
    Size = UDim2.new(0, 140, 0, 20),
    Text = LP.DisplayName,
    Font = Enum.Font.GothamBold,
    TextColor3 = Color3.new(1, 1, 1),
    TextSize = 14,
    BackgroundTransparency = 1,
    TextXAlignment = 0,
    ZIndex = 11,
    Parent = Header
})

Framework:Create("TextLabel", {
    Position = UDim2.fromOffset(58, 28),
    Size = UDim2.new(0, 140, 0, 16),
    Text = "@" .. LP.Name,
    Font = Enum.Font.Gotham,
    TextColor3 = Color3.fromRGB(180, 180, 190),
    TextSize = 11,
    BackgroundTransparency = 1,
    TextXAlignment = 0,
    ZIndex = 11,
    Parent = Header
})

local PlanBadge = Framework:Create("Frame", {
    Position = UDim2.new(0, 58 + 38 + 8, 0, 12),
    Size = UDim2.fromOffset(60, 20),
    BackgroundColor3 = Framework.Accent,
    CornerRadius = 10,
    ZIndex = 11,
    Parent = Header
})

local PlanText = Instance.new("TextLabel")
PlanText.Size = UDim2.new(1, 0, 1, 0)
PlanText.Text = "✨BASIC"
PlanText.Font = Enum.Font.GothamBold
PlanText.TextColor3 = Color3.new(0, 0, 0)
PlanText.TextSize = 10
PlanText.BackgroundTransparency = 1
PlanText.ZIndex = 12
PlanText.Parent = PlanBadge

Framework:Create("TextLabel", {
    Position = UDim2.new(0, 58 + 38 + 8 + 60 + 5, 0, 12),
    Size = UDim2.fromOffset(50, 20),
    Text = "v1.1",
    Font = Enum.Font.GothamMedium,
    TextColor3 = Framework.Accent,
    TextSize = 11,
    BackgroundTransparency = 1,
    ZIndex = 11,
    Parent = Header
})

local CloseBtn = Framework:Create("TextButton", {
    Position = UDim2.new(1, -30, 0, 12),
    Size = UDim2.fromOffset(22, 22),
    BackgroundColor3 = Color3.fromRGB(255, 60, 60),
    Text = "✕",
    Font = Enum.Font.GothamBold,
    TextColor3 = Color3.new(1, 1, 1),
    Active = true,
    ZIndex = 15,
    CornerRadius = 11,
    Parent = Header
})
CloseBtn.MouseButton1Click:Connect(function() Gui:Destroy() blur:Destroy() end)

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
    Stroke = { Color = Color3.fromRGB(40, 40, 40), Thickness = 2, Transparency = 0.3 },
    Parent = Gui
})

-- ==================== DRAG ====================
local dragging, dragStartPos, startPos = false, nil, nil
Header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStartPos = input.Position
        startPos = Main.Position
    end
end)
UIS.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStartPos
        Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then dragging = false end
end)

-- ==================== ЛОГИКА H (НОВАЯ АНИМАЦИЯ) ====================
local isHidden = false
local function toggleMenu()
    isHidden = not isHidden

    if isHidden then
        -- Закрытие
        blur.Enabled = false
        local closeTween = TS:Create(Main, TweenInfo.new(0.4, Enum.EasingStyle.Quart, Enum.EasingDirection.In), {
            Size = UDim2.fromOffset(W, 0),
            BackgroundTransparency = 1
        })
        closeTween:Play()
        closeTween.Completed:Connect(function()
            if isHidden then Main.Visible = false end
        end)
        TS:Create(HButton, TweenInfo.new(0.4), { Rotation = 180 }):Play()
    else
        -- Открытие
        Main.Visible = true
        blur.Enabled = true
        TS:Create(Main, TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
            Size = UDim2.fromOffset(W, H),
            BackgroundTransparency = 0.3
        }):Play()
        TS:Create(HButton, TweenInfo.new(0.5), { Rotation = 0 }):Play()
    end
end
HButton.MouseButton1Click:Connect(toggleMenu)

-- ==================== САЙДБАР И КОНТЕНТ ====================
local sidebarWidth = Framework.IsMobile and 95 or 130
local Sidebar = Framework:Create("Frame", {
    Position = UDim2.fromOffset(10, 62),
    Size = UDim2.new(0, sidebarWidth, 1, -72),
    BackgroundTransparency = 1,
    ZIndex = 2,
    Parent = Main,
    Children = {{"UIListLayout", { Padding = UDim.new(0, 8) }}}
})
local Content = Framework:Create("ScrollingFrame", {
    Position = UDim2.fromOffset(sidebarWidth + 20, 62),
    Size = UDim2.new(1, -sidebarWidth - 35, 1, -72),
    BackgroundTransparency = 1,
    ScrollBarThickness = 0,
    AutomaticCanvasSize = Enum.AutomaticSize.Y,
    ZIndex = 2,
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
        TS:Create(t.B, TweenInfo.new(0.2), { BackgroundColor3 = Color3.fromRGB(60, 70, 90), TextColor3 = Color3.new(1, 1, 1) }):Play()
    end
    TS:Create(btn, TweenInfo.new(0.2), { BackgroundColor3 = Framework.Accent, TextColor3 = Color3.new(0, 0, 0) }):Play()
end

local function createTab(name, icon, active)
    local tabFrame = Framework:Create("Frame", {
        Size = UDim2.new(1, 0, 0, 0),
        BackgroundTransparency = 1,
        Visible = false,
        AutomaticSize = Enum.AutomaticSize.Y,
        ZIndex = 3,
        Parent = Content,
        Children = {{"UIListLayout", { Padding = UDim.new(0, 8) }}}
    })
    local btn = Framework:Create("TextButton", {
        Size = UDim2.new(1, 0, 0, 36),
        BackgroundColor3 = Color3.fromRGB(60, 70, 90),
        Text = icon .. " " .. name,
        Font = Enum.Font.GothamMedium,
        TextColor3 = Color3.new(1, 1, 1),
        TextSize = 13,
        CornerRadius = 17,
        ZIndex = 5,
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

-- ==================== КОМПОНЕНТЫ ====================
local function addToggle(tab, title, desc, callback)
    local card = Framework:Create("TextButton", {
        Size = UDim2.new(1, 0, 0, 46),
        BackgroundColor3 = Color3.fromRGB(55, 65, 85),
        Text = "",
        AutoButtonColor = false,
        CornerRadius = 10,
        ZIndex = 5,
        Parent = tab
    })
    local info = Framework:Create("Frame", {
        Size = UDim2.new(1, -50, 1, 0),
        BackgroundTransparency = 1,
        ZIndex = 6,
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
        ZIndex = 7,
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
        ZIndex = 7,
        Parent = info
    })
    local sw = Framework:Create("Frame", {
        Position = UDim2.new(1, -42, 0.5, -8),
        Size = UDim2.fromOffset(28, 16),
        BackgroundColor3 = Color3.fromRGB(110, 115, 125),
        CornerRadius = 8,
        ZIndex = 7,
        Parent = card
    })
    local dot = Framework:Create("Frame", {
        Position = UDim2.fromOffset(2, 2),
        Size = UDim2.fromOffset(12, 12),
        BackgroundColor3 = Color3.new(1, 1, 1),
        CornerRadius = 6,
        ZIndex = 8,
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

local function addSlider(tab, title, min, max, default, callback)
    local card = Framework:Create("Frame", {
        Size = UDim2.new(1, 0, 0, 50),
        BackgroundColor3 = Color3.fromRGB(55, 65, 85),
        CornerRadius = 10,
        ZIndex = 5,
        Parent = tab
    })
    local titleLabel = Framework:Create("TextLabel", {
        Position = UDim2.fromOffset(15, 6),
        Size = UDim2.new(1, -30, 0, 16),
        Text = title .. ": " .. default,
        Font = Enum.Font.GothamBold,
        TextColor3 = Color3.new(1, 1, 1),
        TextSize = 13,
        BackgroundTransparency = 1,
        TextXAlignment = 0,
        ZIndex = 6,
        Parent = card
    })
    local slider = Framework:Create("Frame", {
        Position = UDim2.fromOffset(15, 28),
        Size = UDim2.new(1, -30, 0, 4),
        BackgroundColor3 = Color3.fromRGB(110, 115, 125),
        CornerRadius = 2,
        ZIndex = 6,
        Parent = card
    })
    local fill = Framework:Create("Frame", {
        Size = UDim2.fromScale((default - min) / (max - min), 1),
        BackgroundColor3 = Framework.Accent,
        CornerRadius = 2,
        ZIndex = 7,
        Parent = slider
    })
    local knob = Framework:Create("TextButton", {
        Size = UDim2.fromOffset(12, 12),
        Position = UDim2.fromScale((default - min) / (max - min), -0.5),
        BackgroundColor3 = Color3.new(1, 1, 1),
        Text = "",
        CornerRadius = 6,
        ZIndex = 9,
        Parent = slider
    })
    local function updateSlider(input)
        local relPos = math.clamp((input.Position.X - slider.AbsolutePosition.X) / slider.AbsoluteSize.X, 0, 1)
        local value = math.floor(min + (max - min) * relPos)
        fill.Size = UDim2.fromScale(relPos, 1)
        knob.Position = UDim2.fromScale(relPos, -0.5)
        titleLabel.Text = title .. ": " .. value
        callback(value)
    end
    local sliding = false
    knob.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then sliding = true end
    end)
    UIS.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then sliding = false end
    end)
    UIS.InputChanged:Connect(function(input)
        if sliding and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then updateSlider(input) end
    end)
end

-- ==================== ВКЛАДКИ ====================
local home = createTab("Home", "🏠", true)
local utility = createTab("Utility", "🛠️", false)
local executor = createTab("Executor", "💻", false)
local hubs = createTab("Hubs", "📁", false)
local settings = createTab("Settings", "⚙️", false)

addToggle(home, "Fly", "Flying mode (WASD)", function(v) Settings.Fly = v end)
addToggle(home, "Noclip", "Walk through walls", function(v) Settings.Noclip = v end)
addToggle(home, "Inf Jump", "No jump cooldown", function(v) Settings.InfJump = v end)
addSlider(home, "WalkSpeed", 16, 250, 16, function(v) Settings.WalkSpeed = v end)
addSlider(home, "JumpPower", 50, 500, 50, function(v) Settings.JumpPower = v end)

addToggle(utility, "Anti-AFK", "Prevent disconnect", function(v) Settings.AntiAFK = v end)
Framework:Create("TextButton", {
    Size = UDim2.new(1, 0, 0, 32),
    BackgroundColor3 = Color3.fromRGB(60, 70, 90),
    Text = "Server Hop",
    Font = Enum.Font.GothamBold,
    TextColor3 = Color3.new(1, 1, 1),
    CornerRadius = 8,
    ZIndex = 5,
    Parent = utility
}).MouseButton1Click:Connect(function() TeleportService:Teleport(game.PlaceId, LP) end)

local ScriptBox = Framework:Create("TextBox", {
    Size = UDim2.new(1, 0, 0, 150),
    BackgroundColor3 = Color3.fromRGB(35, 40, 50),
    TextColor3 = Color3.new(1, 1, 1),
    Font = Enum.Font.Code,
    TextSize = 12,
    Text = "-- Paste code here",
    MultiLine = true,
    ClearTextOnFocus = false,
    ZIndex = 5,
    CornerRadius = 8,
    Parent = executor
})
Framework:Create("TextButton", {
    Size = UDim2.new(1, 0, 0, 35),
    BackgroundColor3 = Framework.Accent,
    Text = "Execute",
    Font = Enum.Font.GothamBold,
    TextColor3 = Color3.new(0,0,0),
    ZIndex = 6,
    CornerRadius = 8,
    Parent = executor
}).MouseButton1Click:Connect(function() pcall(loadstring(ScriptBox.Text)) end)

-- ==================== ЛОГИКА ====================
local cachedParts = {}
local function updateNoclipCache(char)
    cachedParts = {}
    for _, p in pairs(char:GetDescendants()) do if p:IsA("BasePart") then table.insert(cachedParts, p) end end
    char.DescendantAdded:Connect(function(d) if d:IsA("BasePart") then table.insert(cachedParts, d) end end)
end

if LP.Character then updateNoclipCache(LP.Character) end
LP.CharacterAdded:Connect(updateNoclipCache)

RunService.RenderStepped:Connect(function()
    local char = LP.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.WalkSpeed = Settings.WalkSpeed
        char.Humanoid.JumpPower = Settings.JumpPower
        if Settings.Noclip then
            for _, part in pairs(cachedParts) do
                if part and part.Parent then part.CanCollide = false end
            end
        end
    end
end)

UIS.JumpRequest:Connect(function()
    if Settings.InfJump and LP.Character and LP.Character:FindFirstChild("Humanoid") then
        LP.Character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end
end)

local VU = game:GetService("VirtualUser")
LP.Idled:Connect(function()
    if Settings.AntiAFK then
        VU:CaptureController()
        VU:ClickButton2(Vector2.new())
    end
end)

print("✅ Hasession v1.1 — Smooth Animation Ready!")
