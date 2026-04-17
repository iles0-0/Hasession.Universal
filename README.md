--[[
    Hasession Hub - Final Polish V2
    - Исправлено вертикальное положение текста (теперь не прижат вверх)
    - Добавлены отступы Padding для всех элементов
    - Блюр синхронизирован со сворачиванием
    - Анимации и Drag работают стабильно
]]

local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local TS = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local isMobile = UIS.TouchEnabled and not UIS.KeyboardEnabled

-- Чистка
local old = LP.PlayerGui:FindFirstChild("HasessionGui")
if old then old:Destroy() end

local Gui = Instance.new("ScreenGui", LP.PlayerGui)
Gui.Name = "HasessionGui"
Gui.ResetOnSpawn = false
Gui.IgnoreGuiInset = true

local Settings = { Fly = false, Noclip = false, InfJump = false }
local AccentColor = Color3.fromRGB(0, 200, 255)

-- Блюр
local blur = Instance.new("BlurEffect", game:GetService("Lighting"))
blur.Size = 10
blur.Enabled = true

local windowWidth = isMobile and 380 or 480
local windowHeight = isMobile and 275 or 340

-- ==================== ГЛАВНОЕ ОКНО ====================
local Main = Instance.new("Frame", Gui)
Main.BackgroundColor3 = Color3.fromRGB(18, 22, 28)
Main.BackgroundTransparency = 0.1
Main.Size = UDim2.fromOffset(windowWidth, windowHeight)
Main.Position = UDim2.fromScale(0.5, 0.5)
Main.AnchorPoint = Vector2.new(0.5, 0.5)
Main.ClipsDescendants = true
Main.ZIndex = 10
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 14)

local Shadow = Instance.new("ImageLabel", Main)
Shadow.BackgroundTransparency = 1
Shadow.Position = UDim2.new(0, -15, 0, -15)
Shadow.Size = UDim2.new(1, 30, 1, 30)
Shadow.Image = "rbxassetid://6014261993"
Shadow.ImageColor3 = Color3.new(0, 0, 0)
Shadow.ImageTransparency = 0.6
Shadow.ZIndex = 9

-- ==================== ХЕДЕР ====================
local Header = Instance.new("Frame", Main)
Header.Size = UDim2.new(1, 0, 0, 40)
Header.BackgroundTransparency = 1
Header.ZIndex = 15

local Logo = Instance.new("TextLabel", Header)
Logo.Position = UDim2.fromOffset(15, 0)
Logo.Size = UDim2.new(0, 120, 1, 0)
Logo.BackgroundTransparency = 1
Logo.Font = Enum.Font.GothamBold
Logo.RichText = true
Logo.Text = '<font color="rgb(0, 200, 255)">H</font>asession'
Logo.TextColor3 = Color3.new(1, 1, 1)
Logo.TextSize = 16
Logo.TextXAlignment = 0

local AlphaTag = Instance.new("TextLabel", Header)
AlphaTag.AnchorPoint = Vector2.new(0.5, 0.5)
AlphaTag.Position = UDim2.fromScale(0.5, 0.5)
AlphaTag.Size = UDim2.fromOffset(60, 20)
AlphaTag.BackgroundTransparency = 1
AlphaTag.Font = Enum.Font.GothamMedium
AlphaTag.Text = "Alpha"
AlphaTag.TextColor3 = AccentColor
AlphaTag.TextSize = 11
AlphaTag.TextTransparency = 0.4

local CloseBtn = Instance.new("TextButton", Header)
CloseBtn.Position = UDim2.new(1, -30, 0.5, -11)
CloseBtn.Size = UDim2.fromOffset(22, 22)
CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 60, 60)
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.new(1, 1, 1)
CloseBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(1, 0)

local MinimizeBtn = Instance.new("TextButton", Header)
MinimizeBtn.Position = UDim2.new(1, -58, 0.5, -11)
MinimizeBtn.Size = UDim2.fromOffset(22, 22)
MinimizeBtn.BackgroundColor3 = Color3.fromRGB(255, 180, 0)
MinimizeBtn.Text = "—"
MinimizeBtn.TextColor3 = Color3.new(0, 0, 0)
MinimizeBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", MinimizeBtn).CornerRadius = UDim.new(1, 0)

-- ==================== DRAG & MINIMIZE ====================
local dragToggle, dragStart, startPos
Header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragToggle = true
        dragStart = input.Position
        startPos = Main.Position
    end
end)
UIS.InputChanged:Connect(function(input)
    if dragToggle and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UIS.InputEnded:Connect(function() dragToggle = false end)

local isMinimized = false
MinimizeBtn.MouseButton1Click:Connect(function()
    isMinimized = not isMinimized
    local targetHeight = isMinimized and 40 or windowHeight
    TS:Create(Main, TweenInfo.new(0.4, Enum.EasingStyle.Quart), {Size = UDim2.fromOffset(windowWidth, targetHeight)}):Play()
    blur.Enabled = not isMinimized
end)

CloseBtn.MouseButton1Click:Connect(function() 
    blur:Destroy()
    Gui:Destroy() 
end)

-- ==================== САЙДБАР ====================
local sidebarWidth = isMobile and 90 or 120
local Sidebar = Instance.new("Frame", Main)
Sidebar.Position = UDim2.fromOffset(10, 45)
Sidebar.Size = UDim2.new(0, sidebarWidth, 1, -55)
Sidebar.BackgroundTransparency = 1
Instance.new("UIListLayout", Sidebar).Padding = UDim.new(0, 8)

-- ==================== КОНТЕНТ ====================
local Content = Instance.new("ScrollingFrame", Main)
Content.Position = UDim2.fromOffset(sidebarWidth + 20, 45)
Content.Size = UDim2.new(1, -sidebarWidth - 35, 1, -55)
Content.BackgroundTransparency = 1
Content.ScrollBarThickness = 0
Content.AutomaticCanvasSize = Enum.AutomaticSize.Y
local cLayout = Instance.new("UIListLayout", Content)
cLayout.Padding = UDim.new(0, 8)

-- Фикс: Общий отступ для всего списка кнопок
local cPadding = Instance.new("UIPadding", Content)
cPadding.PaddingTop = UDim.new(0, 5) 

-- ==================== ВКЛАДКИ ====================
local Tabs = {}
local function CreateTab(name, active)
    local tabFrame = Instance.new("Frame", Content)
    tabFrame.Size = UDim2.new(1, 0, 0, 0)
    tabFrame.BackgroundTransparency = 1
    tabFrame.Visible = active
    tabFrame.AutomaticSize = Enum.AutomaticSize.Y
    Instance.new("UIListLayout", tabFrame).Padding = UDim.new(0, 8)
    
    local btn = Instance.new("TextButton", Sidebar)
    btn.Size = UDim2.new(1, 0, 0, 34)
    btn.BackgroundColor3 = active and Color3.fromRGB(40, 50, 65) or Color3.fromRGB(25, 29, 36)
    btn.Text = name
    btn.TextColor3 = active and Color3.new(1, 1, 1) or Color3.fromRGB(140, 140, 140)
    btn.Font = Enum.Font.GothamMedium
    btn.TextSize = 12
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 17)
    
    local stroke = Instance.new("UIStroke", btn)
    stroke.Color = AccentColor
    stroke.Thickness = 1.4
    stroke.Transparency = active and 0.2 or 1

    btn.MouseButton1Click:Connect(function()
        for _, t in pairs(Tabs) do 
            t.F.Visible = false
            TS:Create(t.B, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(25, 29, 36), TextColor3 = Color3.fromRGB(140, 140, 140)}):Play()
            TS:Create(t.S, TweenInfo.new(0.2), {Transparency = 1}):Play()
        end
        tabFrame.Visible = true
        TS:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(40, 50, 65), TextColor3 = Color3.new(1, 1, 1)}):Play()
        TS:Create(stroke, TweenInfo.new(0.2), {Transparency = 0.2}):Play()
    end)
    Tabs[name] = {F = tabFrame, B = btn, S = stroke}
    return tabFrame
end

-- ==================== ТОГГЛЫ (ФИКС ВЫРАВНИВАНИЯ) ====================
local function AddToggle(tab, title, desc, callback)
    local card = Instance.new("TextButton", tab)
    card.Size = UDim2.new(1, 0, 0, 52) -- Чуть увеличили высоту для баланса
    card.BackgroundColor3 = Color3.fromRGB(30, 36, 45)
    card.Text = ""
    card.AutoButtonColor = false
    Instance.new("UICorner", card).CornerRadius = UDim.new(0, 10)

    -- Контейнер для текста с отступами (Центрирует текст по вертикали)
    local textContainer = Instance.new("Frame", card)
    textContainer.Size = UDim2.new(1, -50, 1, 0)
    textContainer.BackgroundTransparency = 1
    
    local tPadding = Instance.new("UIPadding", textContainer)
    tPadding.PaddingLeft = UDim.new(0, 15)
    tPadding.PaddingTop = UDim.new(0, 10) -- Фикс: Опускает текст Fly ниже

    local t = Instance.new("TextLabel", textContainer)
    t.Size = UDim2.new(1, 0, 0, 15)
    t.Text = title
    t.Font = Enum.Font.GothamBold
    t.TextColor3 = Color3.new(1, 1, 1)
    t.TextSize = 13
    t.BackgroundTransparency = 1
    t.TextXAlignment = 0

    local d = Instance.new("TextLabel", textContainer)
    d.Position = UDim2.fromOffset(0, 18)
    d.Size = UDim2.new(1, 0, 0, 12)
    d.Text = desc
    d.Font = Enum.Font.Gotham
    d.TextColor3 = Color3.fromRGB(160, 160, 160)
    d.TextSize = 9
    d.BackgroundTransparency = 1
    d.TextXAlignment = 0

    local sw = Instance.new("Frame", card)
    sw.Position = UDim2.new(1, -42, 0.5, -8)
    sw.Size = UDim2.fromOffset(28, 16)
    sw.BackgroundColor3 = Color3.fromRGB(55, 55, 55)
    Instance.new("UICorner", sw).CornerRadius = UDim.new(1, 0)

    local dot = Instance.new("Frame", sw)
    dot.Position = UDim2.fromOffset(2, 2)
    dot.Size = UDim2.fromOffset(12, 12)
    dot.BackgroundColor3 = Color3.new(1, 1, 1)
    Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)

    local en = false
    card.MouseButton1Click:Connect(function()
        en = not en
        TS:Create(sw, TweenInfo.new(0.2), {BackgroundColor3 = en and AccentColor or Color3.fromRGB(55, 55, 55)}):Play()
        TS:Create(dot, TweenInfo.new(0.2), {Position = en and UDim2.fromOffset(14, 2) or UDim2.fromOffset(2, 2)}):Play()
        callback(en)
    end)
end

-- Вкладки
local home = CreateTab("Home", true)
CreateTab("Executor", false)
CreateTab("Hubs", false)

-- Функции
AddToggle(home, "Fly", "Abilty to fly around map", function(v) Settings.Fly = v end)
AddToggle(home, "Noclip", "Walk through objects", function(v) Settings.Noclip = v end)
AddToggle(home, "Inf Jump", "Jump in the air", function(v) Settings.InfJump = v end)

-- Старт
Main.Size = UDim2.fromOffset(windowWidth, 0)
TS:Create(Main, TweenInfo.new(0.5, Enum.EasingStyle.Back), {Size = UDim2.fromOffset(windowWidth, windowHeight)}):Play()

