--[[
    ══════════════════════════════════════════════════════════════════════
    ✦ TriOX Hub — Material Design 3 Expressive Edition ✦
    • Авторы оригинала: TriOX (141676) Team
    • Редизайн: Material Design 3 Expressive (Google M3)
    • Архитектура: Единый автономный скрипт (100% без внешних подгрузок)
    • Модули: Полёт, Телекинез, Подсветка (ESP), Наблюдение (Spectate),
              Lua-Исполнитель, Утилиты/Читы, Динамическая палитра тем
    ══════════════════════════════════════════════════════════════════════
--]]

-- Сервисы Roblox
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local StarterGui = game:GetService("StarterGui")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local TeleportService = game:GetService("TeleportService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local Camera = Workspace.CurrentCamera

-- Защита от дубликатов: удаляем предыдущую версию GUI
if PlayerGui:FindFirstChild("TriOX_MD3_Hub") then
    PlayerGui.TriOX_MD3_Hub:Destroy()
end
if PlayerGui:FindFirstChild("TriOX_FlyHUD") then
    PlayerGui.TriOX_FlyHUD:Destroy()
end
if PlayerGui:FindFirstChild("TriOX_SpectateHUD") then
    PlayerGui.TriOX_SpectateHUD:Destroy()
end

-- ══════════════════════════════════════════════════════════════════════
-- 1. СИСТЕМА ДИЗАЙНА MATERIAL DESIGN 3 EXPRESSIVE (ЦВЕТА И СТИЛИ)
-- ══════════════════════════════════════════════════════════════════════

local ColorPalettes = {
    Violet = {
        Primary = Color3.fromRGB(176, 133, 255),
        PrimaryContainer = Color3.fromRGB(79, 52, 139),
        OnPrimaryContainer = Color3.fromRGB(238, 228, 255),
        Accent = Color3.fromRGB(200, 160, 255)
    },
    Blue = {
        Primary = Color3.fromRGB(100, 180, 255),
        PrimaryContainer = Color3.fromRGB(30, 70, 130),
        OnPrimaryContainer = Color3.fromRGB(220, 240, 255),
        Accent = Color3.fromRGB(140, 205, 255)
    },
    Emerald = {
        Primary = Color3.fromRGB(80, 220, 160),
        PrimaryContainer = Color3.fromRGB(20, 80, 55),
        OnPrimaryContainer = Color3.fromRGB(215, 255, 235),
        Accent = Color3.fromRGB(120, 240, 185)
    },
    Amber = {
        Primary = Color3.fromRGB(255, 180, 70),
        PrimaryContainer = Color3.fromRGB(100, 65, 15),
        OnPrimaryContainer = Color3.fromRGB(255, 240, 215),
        Accent = Color3.fromRGB(255, 205, 110)
    },
    Rose = {
        Primary = Color3.fromRGB(255, 120, 175),
        PrimaryContainer = Color3.fromRGB(110, 30, 70),
        OnPrimaryContainer = Color3.fromRGB(255, 225, 240),
        Accent = Color3.fromRGB(255, 160, 205)
    }
}

local currentPaletteName = "Violet"
local currentThemeName = "Dark"
local isRainbowGlowActive = true

local BaseThemes = {
    Dark = {
        Surface = Color3.fromRGB(18, 16, 24),
        SurfaceContainer = Color3.fromRGB(27, 24, 35),
        SurfaceContainerHigh = Color3.fromRGB(37, 33, 48),
        SurfaceContainerHighest = Color3.fromRGB(48, 43, 62),
        OnSurface = Color3.fromRGB(242, 238, 246),
        OnSurfaceVariant = Color3.fromRGB(188, 182, 198),
        Outline = Color3.fromRGB(130, 122, 142),
        OutlineVariant = Color3.fromRGB(62, 57, 75),
        Success = Color3.fromRGB(75, 220, 130),
        SuccessContainer = Color3.fromRGB(22, 72, 40),
        Danger = Color3.fromRGB(255, 105, 115),
        DangerContainer = Color3.fromRGB(92, 25, 32),
        Warning = Color3.fromRGB(255, 190, 75),
        CodeBg = Color3.fromRGB(12, 10, 17),
        Scrim = Color3.fromRGB(0, 0, 0)
    },
    Light = {
        Surface = Color3.fromRGB(252, 248, 254),
        SurfaceContainer = Color3.fromRGB(244, 239, 248),
        SurfaceContainerHigh = Color3.fromRGB(235, 229, 241),
        SurfaceContainerHighest = Color3.fromRGB(226, 219, 233),
        OnSurface = Color3.fromRGB(28, 25, 33),
        OnSurfaceVariant = Color3.fromRGB(86, 81, 95),
        Outline = Color3.fromRGB(175, 168, 185),
        OutlineVariant = Color3.fromRGB(215, 208, 225),
        Success = Color3.fromRGB(30, 160, 80),
        SuccessContainer = Color3.fromRGB(215, 248, 225),
        Danger = Color3.fromRGB(220, 50, 60),
        DangerContainer = Color3.fromRGB(255, 220, 222),
        Warning = Color3.fromRGB(210, 130, 20),
        CodeBg = Color3.fromRGB(240, 235, 246),
        Scrim = Color3.fromRGB(0, 0, 0)
    }
}

local function getActiveTheme()
    local base = BaseThemes[currentThemeName]
    local palette = ColorPalettes[currentPaletteName]
    local t = {}
    for k, v in pairs(base) do t[k] = v end
    for k, v in pairs(palette) do t[k] = v end
    return t
end

local theme = getActiveTheme()

-- Радужные цвета TriOX для фирменного анимированного градиента
local rainbowColors = {
    Color3.fromRGB(176, 133, 255),
    Color3.fromRGB(100, 180, 255),
    Color3.fromRGB(80, 220, 160),
    Color3.fromRGB(255, 185, 80),
    Color3.fromRGB(255, 115, 175)
}
local rainbowProgress = 0
local currentRainbowColor = rainbowColors[1]

-- Реестр UI-элементов для динамического обновления темы
local themeRegistry = {
    Frames = {},
    Cards = {},
    Strokes = {},
    Labels = {},
    SubLabels = {},
    Buttons = {},
    Inputs = {},
    Scrolls = {}
}

-- ══════════════════════════════════════════════════════════════════════
-- 2. СОЗДАНИЕ ГЛАВНОГО SCREEN GUI
-- ══════════════════════════════════════════════════════════════════════

local mainGUI = Instance.new("ScreenGui")
mainGUI.Name = "TriOX_MD3_Hub"
mainGUI.ResetOnSpawn = false
mainGUI.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
mainGUI.Parent = PlayerGui

-- ══════════════════════════════════════════════════════════════════════
-- 3. ВСПОМОГАТЕЛЬНЫЕ ФУНКЦИИ И АНИМАЦИИ (TWEEN / DRAG / FEEDBACK)
-- ══════════════════════════════════════════════════════════════════════

local function tween(instance, duration, properties, style, direction)
    style = style or Enum.EasingStyle.Quart
    direction = direction or Enum.EasingDirection.Out
    local info = TweenInfo.new(duration, style, direction)
    local anim = TweenService:Create(instance, info, properties)
    anim:Play()
    return anim
end

local function makeDraggable(frame, dragHandle)
    dragHandle = dragHandle or frame
    local dragging = false
    local dragInput, dragStart, startPos

    dragHandle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    dragHandle.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if dragging and input == dragInput then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(
                startPos.X.Scale,
                startPos.X.Offset + delta.X,
                startPos.Y.Scale,
                startPos.Y.Offset + delta.Y
            )
        end
    end)
end

-- Всплывающее уведомление MD3 (Snackbar / Toast)
local function showToast(titleText, descText, toastType, duration)
    duration = duration or 3
    toastType = toastType or "Info"

    pcall(function()
        StarterGui:SetCore("SendNotification", {
            Title = "TriOX • " .. titleText,
            Text = descText,
            Duration = duration,
            Icon = "rbxthumb://type=Asset&id=5107182114&w=150&h=150"
        })
    end)

    local toastFrame = Instance.new("Frame")
    toastFrame.Name = "MD3_Toast"
    toastFrame.Size = UDim2.new(0, 320, 0, 54)
    toastFrame.Position = UDim2.new(0.5, -160, 0, -70)
    toastFrame.BackgroundColor3 = theme.SurfaceContainerHighest
    toastFrame.BorderSizePixel = 0
    toastFrame.ZIndex = 100
    toastFrame.Parent = mainGUI

    local toastCorner = Instance.new("UICorner")
    toastCorner.CornerRadius = UDim.new(0, 16)
    toastCorner.Parent = toastFrame

    local toastStroke = Instance.new("UIStroke")
    local accentCol = theme.Primary
    if toastType == "Success" then accentCol = theme.Success
    elseif toastType == "Error" then accentCol = theme.Danger
    elseif toastType == "Warning" then accentCol = theme.Warning end
    toastStroke.Color = accentCol
    toastStroke.Thickness = 1.5
    toastStroke.Transparency = 0.2
    toastStroke.Parent = toastFrame

    local indicator = Instance.new("Frame")
    indicator.Size = UDim2.new(0, 6, 0, 32)
    indicator.Position = UDim2.new(0, 12, 0.5, -16)
    indicator.BackgroundColor3 = accentCol
    indicator.BorderSizePixel = 0
    indicator.ZIndex = 101
    indicator.Parent = toastFrame

    local indCorner = Instance.new("UICorner")
    indCorner.CornerRadius = UDim.new(1, 0)
    indCorner.Parent = indicator

    local titleLbl = Instance.new("TextLabel")
    titleLbl.Size = UDim2.new(1, -36, 0, 18)
    titleLbl.Position = UDim2.new(0, 26, 0, 9)
    titleLbl.BackgroundTransparency = 1
    titleLbl.Font = Enum.Font.GothamBold
    titleLbl.TextSize = 13
    titleLbl.TextColor3 = theme.OnSurface
    titleLbl.TextXAlignment = Enum.TextXAlignment.Left
    titleLbl.Text = titleText
    titleLbl.ZIndex = 101
    titleLbl.Parent = toastFrame

    local descLbl = Instance.new("TextLabel")
    descLbl.Size = UDim2.new(1, -36, 0, 16)
    descLbl.Position = UDim2.new(0, 26, 0, 28)
    descLbl.BackgroundTransparency = 1
    descLbl.Font = Enum.Font.Gotham
    descLbl.TextSize = 11
    descLbl.TextColor3 = theme.OnSurfaceVariant
    descLbl.TextXAlignment = Enum.TextXAlignment.Left
    descLbl.TextTruncate = Enum.TextTruncate.AtEnd
    descLbl.Text = descText
    descLbl.ZIndex = 101
    descLbl.Parent = toastFrame

    tween(toastFrame, 0.4, {Position = UDim2.new(0.5, -160, 0, 20)}, Enum.EasingStyle.Back, Enum.EasingDirection.Out)

    task.delay(duration, function()
        if toastFrame and toastFrame.Parent then
            local hide = tween(toastFrame, 0.35, {Position = UDim2.new(0.5, -160, 0, -80)}, Enum.EasingStyle.Quart, Enum.EasingDirection.In)
            hide.Completed:Connect(function()
                toastFrame:Destroy()
            end)
        end
    end)
end

-- ══════════════════════════════════════════════════════════════════════
-- 4. ГЛАВНОЕ ОКНО И MD3 ЭЛЕМЕНТЫ УПРАВЛЕНИЯ
-- ══════════════════════════════════════════════════════════════════════

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 640, 0, 420)
mainFrame.Position = UDim2.new(0.5, -320, 0.5, -210)
mainFrame.BackgroundColor3 = theme.Surface
mainFrame.BorderSizePixel = 0
mainFrame.ClipsDescendants = false
mainFrame.Active = true
mainFrame.Parent = mainGUI

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 24)
mainCorner.Parent = mainFrame

local mainStroke = Instance.new("UIStroke")
mainStroke.Color = theme.OutlineVariant
mainStroke.Thickness = 1
mainStroke.Transparency = 0.5
mainStroke.Parent = mainFrame

-- Круглая плавающая кнопка разворачивания (MD3 Expressive FAB)
local unfoldFAB = Instance.new("Frame")
unfoldFAB.Name = "UnfoldFAB"
unfoldFAB.Size = UDim2.new(0, 60, 0, 60)
unfoldFAB.Position = UDim2.new(0, 20, 0.5, -30)
unfoldFAB.BackgroundColor3 = theme.SurfaceContainerHighest
unfoldFAB.BorderSizePixel = 0
unfoldFAB.Visible = false
unfoldFAB.Active = true
unfoldFAB.ZIndex = 50
unfoldFAB.Parent = mainGUI

local fabCorner = Instance.new("UICorner")
fabCorner.CornerRadius = UDim.new(0, 18)
fabCorner.Parent = unfoldFAB

local fabStroke = Instance.new("UIStroke")
fabStroke.Color = theme.Primary
fabStroke.Thickness = 2
fabStroke.Parent = unfoldFAB

local fabBtn = Instance.new("TextButton")
fabBtn.Size = UDim2.new(1, 0, 1, 0)
fabBtn.BackgroundTransparency = 1
fabBtn.Font = Enum.Font.GothamBold
fabBtn.Text = "T"
fabBtn.TextSize = 26
fabBtn.TextColor3 = theme.Primary
fabBtn.ZIndex = 51
fabBtn.Parent = unfoldFAB

makeDraggable(unfoldFAB, unfoldFAB)

fabBtn.MouseButton1Click:Connect(function()
    mainFrame.Visible = true
    unfoldFAB.Visible = false
    tween(mainFrame, 0.35, {Size = UDim2.new(0, 640, 0, 420)}, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
end)

-- ──────────────────────────────────────────────────────────────────────
-- ЛЕВАЯ НАВИГАЦИОННАЯ ПАНЕЛЬ (MD3 Navigation Rail)
-- ──────────────────────────────────────────────────────────────────────

local navRail = Instance.new("Frame")
navRail.Name = "NavRail"
navRail.Size = UDim2.new(0, 160, 1, 0)
navRail.Position = UDim2.new(0, 0, 0, 0)
navRail.BackgroundColor3 = theme.SurfaceContainer
navRail.BorderSizePixel = 0
navRail.Parent = mainFrame

local navCorner = Instance.new("UICorner")
navCorner.CornerRadius = UDim.new(0, 24)
navCorner.Parent = navRail

local logoHeader = Instance.new("Frame")
logoHeader.Name = "LogoHeader"
logoHeader.Size = UDim2.new(1, 0, 0, 64)
logoHeader.BackgroundTransparency = 1
logoHeader.Parent = navRail

local logoBadge = Instance.new("Frame")
logoBadge.Size = UDim2.new(0, 36, 0, 36)
logoBadge.Position = UDim2.new(0, 14, 0.5, -18)
logoBadge.BackgroundColor3 = theme.PrimaryContainer
logoBadge.BorderSizePixel = 0
logoBadge.Parent = logoHeader

local logoBadgeCorner = Instance.new("UICorner")
logoBadgeCorner.CornerRadius = UDim.new(0, 12)
logoBadgeCorner.Parent = logoBadge

local logoText = Instance.new("TextLabel")
logoText.Size = UDim2.new(1, 0, 1, 0)
logoText.BackgroundTransparency = 1
logoText.Font = Enum.Font.GothamBold
logoText.Text = "T"
logoText.TextSize = 20
logoText.TextColor3 = theme.OnPrimaryContainer
logoText.Parent = logoBadge

local titleLabel = Instance.new("TextLabel")
titleLabel.Name = "TitleLabel"
titleLabel.Size = UDim2.new(1, -60, 0, 20)
titleLabel.Position = UDim2.new(0, 56, 0, 14)
titleLabel.BackgroundTransparency = 1
titleLabel.Font = Enum.Font.GothamBold
titleLabel.Text = "TriOX Hub"
titleLabel.TextSize = 15
titleLabel.TextColor3 = theme.OnSurface
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = logoHeader

local subTitleLabel = Instance.new("TextLabel")
subTitleLabel.Name = "SubTitleLabel"
subTitleLabel.Size = UDim2.new(1, -60, 0, 16)
subTitleLabel.Position = UDim2.new(0, 56, 0, 34)
subTitleLabel.BackgroundTransparency = 1
subTitleLabel.Font = Enum.Font.GothamMedium
subTitleLabel.Text = "MD3 Expressive"
subTitleLabel.TextSize = 10
subTitleLabel.TextColor3 = theme.Primary
subTitleLabel.TextXAlignment = Enum.TextXAlignment.Left
subTitleLabel.Parent = logoHeader

local navList = Instance.new("ScrollingFrame")
navList.Name = "NavList"
navList.Size = UDim2.new(1, 0, 1, -70)
navList.Position = UDim2.new(0, 0, 0, 68)
navList.BackgroundTransparency = 1
navList.BorderSizePixel = 0
navList.ScrollBarThickness = 2
navList.ScrollBarImageColor3 = theme.Outline
navList.CanvasSize = UDim2.new(0, 0, 0, 0)
navList.Parent = navRail

local navLayout = Instance.new("UIListLayout")
navLayout.Padding = UDim.new(0, 4)
navLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
navLayout.SortOrder = Enum.SortOrder.LayoutOrder
navLayout.Parent = navList

navLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    navList.CanvasSize = UDim2.new(0, 0, 0, navLayout.AbsoluteContentSize.Y + 12)
end)

-- ──────────────────────────────────────────────────────────────────────
-- ВЕРХНЯЯ ПАНЕЛЬ ДЕЙСТВИЙ (Правая верхняя часть)
-- ──────────────────────────────────────────────────────────────────────

local topBar = Instance.new("Frame")
topBar.Name = "TopBar"
topBar.Size = UDim2.new(1, -160, 0, 50)
topBar.Position = UDim2.new(0, 160, 0, 0)
topBar.BackgroundTransparency = 1
topBar.Parent = mainFrame

makeDraggable(mainFrame, topBar)
makeDraggable(mainFrame, logoHeader)

local pageHeaderTitle = Instance.new("TextLabel")
pageHeaderTitle.Name = "PageHeaderTitle"
pageHeaderTitle.Size = UDim2.new(1, -160, 1, 0)
pageHeaderTitle.Position = UDim2.new(0, 18, 0, 0)
pageHeaderTitle.BackgroundTransparency = 1
pageHeaderTitle.Font = Enum.Font.GothamBold
pageHeaderTitle.Text = "🏠 Главная"
pageHeaderTitle.TextSize = 16
pageHeaderTitle.TextColor3 = theme.OnSurface
pageHeaderTitle.TextXAlignment = Enum.TextXAlignment.Left
pageHeaderTitle.Parent = topBar

local actionsFrame = Instance.new("Frame")
actionsFrame.Size = UDim2.new(0, 130, 0, 36)
actionsFrame.Position = UDim2.new(1, -140, 0.5, -18)
actionsFrame.BackgroundTransparency = 1
actionsFrame.Parent = topBar

local actionsLayout = Instance.new("UIListLayout")
actionsLayout.FillDirection = Enum.FillDirection.Horizontal
actionsLayout.HorizontalAlignment = Enum.HorizontalAlignment.Right
actionsLayout.VerticalAlignment = Enum.VerticalAlignment.Center
actionsLayout.Padding = UDim.new(0, 8)
actionsLayout.Parent = actionsFrame

local function createTopIconButton(name, text, onClick, bgColor, textColor)
    local btn = Instance.new("TextButton")
    btn.Name = name
    btn.Size = UDim2.new(0, 32, 0, 32)
    btn.BackgroundColor3 = bgColor or theme.SurfaceContainerHighest
    btn.BorderSizePixel = 0
    btn.Font = Enum.Font.GothamBold
    btn.Text = text
    btn.TextSize = 14
    btn.TextColor3 = textColor or theme.OnSurface
    btn.Parent = actionsFrame

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(1, 0)
    corner.Parent = btn

    btn.MouseEnter:Connect(function()
        tween(btn, 0.15, {BackgroundColor3 = (bgColor or theme.SurfaceContainerHighest):Lerp(Color3.new(1,1,1), 0.15)})
    end)
    btn.MouseLeave:Connect(function()
        tween(btn, 0.15, {BackgroundColor3 = bgColor or theme.SurfaceContainerHighest})
    end)
    btn.MouseButton1Click:Connect(onClick)
    return btn
end

-- ──────────────────────────────────────────────────────────────────────
-- КОНТЕЙНЕР ДЛЯ СТРАНИЦ ВКЛАДОК
-- ──────────────────────────────────────────────────────────────────────

local contentContainer = Instance.new("Frame")
contentContainer.Name = "ContentContainer"
contentContainer.Size = UDim2.new(1, -165, 1, -56)
contentContainer.Position = UDim2.new(0, 160, 0, 52)
contentContainer.BackgroundTransparency = 1
contentContainer.ClipsDescendants = true
contentContainer.Parent = mainFrame

local pages = {}
local tabButtons = {}
local activePageName = "Home"

local function switchTab(pageName)
    activePageName = pageName
    for name, page in pairs(pages) do
        page.Visible = (name == pageName)
    end
    for name, btnData in pairs(tabButtons) do
        if name == pageName then
            tween(btnData.Button, 0.2, {BackgroundColor3 = theme.PrimaryContainer})
            btnData.TextLabel.TextColor3 = theme.OnPrimaryContainer
            btnData.Indicator.Visible = true
            pageHeaderTitle.Text = btnData.Title
        else
            tween(btnData.Button, 0.2, {BackgroundColor3 = theme.SurfaceContainer})
            btnData.TextLabel.TextColor3 = theme.OnSurfaceVariant
            btnData.Indicator.Visible = false
        end
    end
end

local function registerTab(id, titleText, iconText, layoutOrder)
    local btn = Instance.new("TextButton")
    btn.Name = "Tab_" .. id
    btn.Size = UDim2.new(1, -16, 0, 36)
    btn.BackgroundColor3 = theme.SurfaceContainer
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = false
    btn.LayoutOrder = layoutOrder
    btn.Text = ""
    btn.Parent = navList

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(1, 0)
    corner.Parent = btn

    local activeIndicator = Instance.new("Frame")
    activeIndicator.Name = "Indicator"
    activeIndicator.Size = UDim2.new(0, 4, 0, 18)
    activeIndicator.Position = UDim2.new(0, 6, 0.5, -9)
    activeIndicator.BackgroundColor3 = theme.Primary
    activeIndicator.BorderSizePixel = 0
    activeIndicator.Visible = false
    activeIndicator.Parent = btn

    local indCorner = Instance.new("UICorner")
    indCorner.CornerRadius = UDim.new(1, 0)
    indCorner.Parent = activeIndicator

    local textLbl = Instance.new("TextLabel")
    textLbl.Size = UDim2.new(1, -26, 1, 0)
    textLbl.Position = UDim2.new(0, 16, 0, 0)
    textLbl.BackgroundTransparency = 1
    textLbl.Font = Enum.Font.GothamMedium
    textLbl.TextSize = 13
    textLbl.TextColor3 = theme.OnSurfaceVariant
    textLbl.TextXAlignment = Enum.TextXAlignment.Left
    textLbl.Text = iconText .. "  " .. titleText
    textLbl.Parent = btn

    btn.MouseEnter:Connect(function()
        if activePageName ~= id then
            tween(btn, 0.15, {BackgroundColor3 = theme.SurfaceContainerHighest})
        end
    end)
    btn.MouseLeave:Connect(function()
        if activePageName ~= id then
            tween(btn, 0.15, {BackgroundColor3 = theme.SurfaceContainer})
        end
    end)
    btn.MouseButton1Click:Connect(function()
        switchTab(id)
    end)

    local pageScroll = Instance.new("ScrollingFrame")
    pageScroll.Name = "Page_" .. id
    pageScroll.Size = UDim2.new(1, -10, 1, -6)
    pageScroll.Position = UDim2.new(0, 5, 0, 0)
    pageScroll.BackgroundTransparency = 1
    pageScroll.BorderSizePixel = 0
    pageScroll.ScrollBarThickness = 3
    pageScroll.ScrollBarImageColor3 = theme.Outline
    pageScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
    pageScroll.Visible = false
    pageScroll.Parent = contentContainer

    local pageLayout = Instance.new("UIListLayout")
    pageLayout.Padding = UDim.new(0, 10)
    pageLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    pageLayout.SortOrder = Enum.SortOrder.LayoutOrder
    pageLayout.Parent = pageScroll

    pageLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        pageScroll.CanvasSize = UDim2.new(0, 0, 0, pageLayout.AbsoluteContentSize.Y + 16)
    end)

    pages[id] = pageScroll
    tabButtons[id] = {
        Button = btn,
        TextLabel = textLbl,
        Indicator = activeIndicator,
        Title = iconText .. " " .. titleText
    }

    return pageScroll
end

-- ══════════════════════════════════════════════════════════════════════
-- 5. КОМПОНЕНТЫ ИНТЕРФЕЙСА MD3 EXPRESSIVE (CARDS, BUTTONS, SWITCHES)
-- ══════════════════════════════════════════════════════════════════════

local function createMD3Card(parent, title, desc, layoutOrder)
    local card = Instance.new("Frame")
    card.Name = "Card_" .. tostring(title or "Item")
    card.Size = UDim2.new(1, -10, 0, 0)
    card.AutomaticSize = Enum.AutomaticSize.Y
    card.BackgroundColor3 = theme.SurfaceContainer
    card.BorderSizePixel = 0
    card.LayoutOrder = layoutOrder or 1
    card.Parent = parent

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 16)
    corner.Parent = card

    local stroke = Instance.new("UIStroke")
    stroke.Color = theme.OutlineVariant
    stroke.Thickness = 1
    stroke.Transparency = 0.6
    stroke.Parent = card

    local padding = Instance.new("UIPadding")
    padding.PaddingTop = UDim.new(0, 12)
    padding.PaddingBottom = UDim.new(0, 12)
    padding.PaddingLeft = UDim.new(0, 14)
    padding.PaddingRight = UDim.new(0, 14)
    padding.Parent = card

    local layout = Instance.new("UIListLayout")
    layout.Padding = UDim.new(0, 8)
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.Parent = card

    local tLbl = nil
    if title then
        tLbl = Instance.new("TextLabel")
        tLbl.Name = "CardTitle"
        tLbl.Size = UDim2.new(1, 0, 0, 20)
        tLbl.BackgroundTransparency = 1
        tLbl.Font = Enum.Font.GothamBold
        tLbl.TextSize = 14
        tLbl.TextColor3 = theme.OnSurface
        tLbl.TextXAlignment = Enum.TextXAlignment.Left
        tLbl.Text = title
        tLbl.LayoutOrder = 0
        tLbl.Parent = card
    end

    local dLbl = nil
    if desc then
        dLbl = Instance.new("TextLabel")
        dLbl.Name = "CardDesc"
        dLbl.Size = UDim2.new(1, 0, 0, 14)
        dLbl.BackgroundTransparency = 1
        dLbl.Font = Enum.Font.Gotham
        dLbl.TextSize = 11
        dLbl.TextColor3 = theme.OnSurfaceVariant
        dLbl.TextXAlignment = Enum.TextXAlignment.Left
        dLbl.Text = desc
        dLbl.LayoutOrder = 1
        dLbl.Parent = card
    end

    local contentFrame = Instance.new("Frame")
    contentFrame.Name = "Content"
    contentFrame.Size = UDim2.new(1, 0, 0, 0)
    contentFrame.AutomaticSize = Enum.AutomaticSize.Y
    contentFrame.BackgroundTransparency = 1
    contentFrame.LayoutOrder = 2
    contentFrame.Parent = card

    table.insert(themeRegistry.Cards, {Card = card, Stroke = stroke, Title = tLbl, Desc = dLbl})

    return card, contentFrame
end

local function createMD3Button(parent, text, btnType, onClick, height)
    height = height or 36
    btnType = btnType or "Primary"

    local bgCol = theme.Primary
    local textCol = theme.OnPrimaryContainer
    if btnType == "Secondary" then
        bgCol = theme.SurfaceContainerHighest
        textCol = theme.OnSurface
    elseif btnType == "Danger" then
        bgCol = theme.DangerContainer
        textCol = theme.Danger
    elseif btnType == "Success" then
        bgCol = theme.SuccessContainer
        textCol = theme.Success
    end

    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, height)
    btn.BackgroundColor3 = bgCol
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = false
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 13
    btn.TextColor3 = textCol
    btn.Text = text
    btn.Parent = parent

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(1, 0)
    corner.Parent = btn

    btn.MouseEnter:Connect(function()
        tween(btn, 0.15, {BackgroundColor3 = bgCol:Lerp(Color3.new(1,1,1), 0.15)})
    end)
    btn.MouseLeave:Connect(function()
        tween(btn, 0.15, {BackgroundColor3 = bgCol})
    end)
    btn.MouseButton1Down:Connect(function()
        tween(btn, 0.08, {Size = UDim2.new(0.98, 0, 0, height - 2)})
    end)
    btn.MouseButton1Up:Connect(function()
        tween(btn, 0.12, {Size = UDim2.new(1, 0, 0, height)})
    end)
    btn.MouseButton1Click:Connect(onClick)

    table.insert(themeRegistry.Buttons, {Button = btn, Type = btnType})

    return btn
end

local function createMD3Switch(parent, title, desc, initialState, onToggle)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 38)
    row.BackgroundTransparency = 1
    row.Parent = parent

    local labels = Instance.new("Frame")
    labels.Size = UDim2.new(1, -56, 1, 0)
    labels.BackgroundTransparency = 1
    labels.Parent = row

    local tLbl = Instance.new("TextLabel")
    tLbl.Size = UDim2.new(1, 0, 0, 18)
    tLbl.BackgroundTransparency = 1
    tLbl.Font = Enum.Font.GothamMedium
    tLbl.TextSize = 13
    tLbl.TextColor3 = theme.OnSurface
    tLbl.TextXAlignment = Enum.TextXAlignment.Left
    tLbl.Text = title
    tLbl.Parent = labels

    if desc then
        local dLbl = Instance.new("TextLabel")
        dLbl.Size = UDim2.new(1, 0, 0, 14)
        dLbl.Position = UDim2.new(0, 0, 0, 18)
        dLbl.BackgroundTransparency = 1
        dLbl.Font = Enum.Font.Gotham
        dLbl.TextSize = 10
        dLbl.TextColor3 = theme.OnSurfaceVariant
        dLbl.TextXAlignment = Enum.TextXAlignment.Left
        dLbl.Text = desc
        dLbl.Parent = labels
    end

    local isEnabled = initialState or false

    local track = Instance.new("TextButton")
    track.Size = UDim2.new(0, 46, 0, 24)
    track.Position = UDim2.new(1, -48, 0.5, -12)
    track.BackgroundColor3 = isEnabled and theme.Primary or theme.SurfaceContainerHighest
    track.BorderSizePixel = 0
    track.AutoButtonColor = false
    track.Text = ""
    track.Parent = row

    local trackCorner = Instance.new("UICorner")
    trackCorner.CornerRadius = UDim.new(1, 0)
    trackCorner.Parent = track

    local thumb = Instance.new("Frame")
    thumb.Size = UDim2.new(0, 16, 0, 16)
    thumb.Position = isEnabled and UDim2.new(1, -20, 0.5, -8) or UDim2.new(0, 4, 0.5, -8)
    thumb.BackgroundColor3 = isEnabled and theme.Surface or theme.Outline
    thumb.BorderSizePixel = 0
    thumb.Parent = track

    local thumbCorner = Instance.new("UICorner")
    thumbCorner.CornerRadius = UDim.new(1, 0)
    thumbCorner.Parent = thumb

    local function updateSwitch(animate)
        local targetTrackColor = isEnabled and theme.Primary or theme.SurfaceContainerHighest
        local targetThumbColor = isEnabled and theme.Surface or theme.Outline
        local targetThumbPos = isEnabled and UDim2.new(1, -20, 0.5, -8) or UDim2.new(0, 4, 0.5, -8)

        if animate then
            tween(track, 0.2, {BackgroundColor3 = targetTrackColor})
            tween(thumb, 0.2, {BackgroundColor3 = targetThumbColor, Position = targetThumbPos})
        else
            track.BackgroundColor3 = targetTrackColor
            thumb.BackgroundColor3 = targetThumbColor
            thumb.Position = targetThumbPos
        end
    end

    track.MouseButton1Click:Connect(function()
        isEnabled = not isEnabled
        updateSwitch(true)
        if onToggle then onToggle(isEnabled) end
    end)

    return {
        SetState = function(state)
            isEnabled = state
            updateSwitch(true)
        end,
        GetState = function() return isEnabled end,
        RefreshTheme = function() updateSwitch(false) end
    }
end

local function createMD3Input(parent, labelText, placeholderText, defaultText, onChange)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 54)
    frame.BackgroundTransparency = 1
    frame.Parent = parent

    if labelText then
        local lbl = Instance.new("TextLabel")
        lbl.Size = UDim2.new(1, 0, 0, 16)
        lbl.BackgroundTransparency = 1
        lbl.Font = Enum.Font.GothamMedium
        lbl.TextSize = 11
        lbl.TextColor3 = theme.OnSurfaceVariant
        lbl.TextXAlignment = Enum.TextXAlignment.Left
        lbl.Text = labelText
        lbl.Parent = frame
        table.insert(themeRegistry.SubLabels, lbl)
    end

    local inputContainer = Instance.new("Frame")
    inputContainer.Size = UDim2.new(1, 0, 0, 32)
    inputContainer.Position = UDim2.new(0, 0, 0, 18)
    inputContainer.BackgroundColor3 = theme.SurfaceContainerHigh
    inputContainer.BorderSizePixel = 0
    inputContainer.Parent = frame

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = inputContainer

    local stroke = Instance.new("UIStroke")
    stroke.Color = theme.OutlineVariant
    stroke.Thickness = 1
    stroke.Parent = inputContainer

    local textBox = Instance.new("TextBox")
    textBox.Size = UDim2.new(1, -16, 1, 0)
    textBox.Position = UDim2.new(0, 8, 0, 0)
    textBox.BackgroundTransparency = 1
    textBox.Font = Enum.Font.Gotham
    textBox.TextSize = 12
    textBox.TextColor3 = theme.OnSurface
    textBox.PlaceholderColor3 = theme.Outline
    textBox.PlaceholderText = placeholderText or ""
    textBox.Text = defaultText or ""
    textBox.ClearTextOnFocus = false
    textBox.TextXAlignment = Enum.TextXAlignment.Left
    textBox.Parent = inputContainer

    textBox.Focused:Connect(function()
        tween(stroke, 0.15, {Color = theme.Primary, Thickness = 1.5})
    end)
    textBox.FocusLost:Connect(function()
        tween(stroke, 0.15, {Color = theme.OutlineVariant, Thickness = 1})
        if onChange then onChange(textBox.Text) end
    end)

    table.insert(themeRegistry.Inputs, {Container = inputContainer, Stroke = stroke, TextBox = textBox})

    return textBox
end

-- Функция динамического применения темы ко всем элементам
local function applyCurrentTheme()
    theme = getActiveTheme()

    mainFrame.BackgroundColor3 = theme.Surface
    mainStroke.Color = theme.OutlineVariant
    navRail.BackgroundColor3 = theme.SurfaceContainer
    unfoldFAB.BackgroundColor3 = theme.SurfaceContainerHighest
    fabStroke.Color = theme.Primary
    fabBtn.TextColor3 = theme.Primary

    pageHeaderTitle.TextColor3 = theme.OnSurface
    titleLabel.TextColor3 = theme.OnSurface
    subTitleLabel.TextColor3 = theme.Primary
    logoBadge.BackgroundColor3 = theme.PrimaryContainer
    logoText.TextColor3 = theme.OnPrimaryContainer

    for _, cData in ipairs(themeRegistry.Cards) do
        cData.Card.BackgroundColor3 = theme.SurfaceContainer
        cData.Stroke.Color = theme.OutlineVariant
        if cData.Title then cData.Title.TextColor3 = theme.OnSurface end
        if cData.Desc then cData.Desc.TextColor3 = theme.OnSurfaceVariant end
    end

    for _, bData in ipairs(themeRegistry.Buttons) do
        local bgCol = theme.Primary
        local txtCol = theme.OnPrimaryContainer
        if bData.Type == "Secondary" then
            bgCol = theme.SurfaceContainerHighest
            txtCol = theme.OnSurface
        elseif bData.Type == "Danger" then
            bgCol = theme.DangerContainer
            txtCol = theme.Danger
        elseif bData.Type == "Success" then
            bgCol = theme.SuccessContainer
            txtCol = theme.Success
        end
        bData.Button.BackgroundColor3 = bgCol
        bData.Button.TextColor3 = txtCol
    end

    for _, iData in ipairs(themeRegistry.Inputs) do
        iData.Container.BackgroundColor3 = theme.SurfaceContainerHigh
        iData.Stroke.Color = theme.OutlineVariant
        iData.TextBox.TextColor3 = theme.OnSurface
        iData.TextBox.PlaceholderColor3 = theme.Outline
    end

    for _, l in ipairs(themeRegistry.SubLabels) do
        l.TextColor3 = theme.OnSurfaceVariant
    end

    for id, btnData in pairs(tabButtons) do
        if id == activePageName then
            btnData.Button.BackgroundColor3 = theme.PrimaryContainer
            btnData.TextLabel.TextColor3 = theme.OnPrimaryContainer
            btnData.Indicator.BackgroundColor3 = theme.Primary
        else
            btnData.Button.BackgroundColor3 = theme.SurfaceContainer
            btnData.TextLabel.TextColor3 = theme.OnSurfaceVariant
        end
    end
end

-- ══════════════════════════════════════════════════════════════════════
-- 6. МОДУЛЬ 1: ПОЛЁТ (FLY)
-- ══════════════════════════════════════════════════════════════════════

local flyActive = false
local flySpeed = 1
local flyConnection = nil
local upHoldConnection = nil
local downHoldConnection = nil
local flyMiniHudVisible = false
local flyKeyHeld = {W = false, A = false, S = false, D = false, Up = false, Down = false}

local function enableHumanoidStates(char)
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if hum then
        hum:SetStateEnabled(Enum.HumanoidStateType.Climbing, true)
        hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, true)
        hum:SetStateEnabled(Enum.HumanoidStateType.Flying, true)
        hum:SetStateEnabled(Enum.HumanoidStateType.Freefall, true)
        hum:SetStateEnabled(Enum.HumanoidStateType.GettingUp, true)
        hum:SetStateEnabled(Enum.HumanoidStateType.Jumping, true)
        hum:SetStateEnabled(Enum.HumanoidStateType.Landed, true)
        hum:SetStateEnabled(Enum.HumanoidStateType.Physics, true)
        hum:SetStateEnabled(Enum.HumanoidStateType.PlatformStanding, true)
        hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, true)
        hum:SetStateEnabled(Enum.HumanoidStateType.Running, true)
        hum:SetStateEnabled(Enum.HumanoidStateType.RunningNoPhysics, true)
        hum:SetStateEnabled(Enum.HumanoidStateType.Seated, true)
        hum:SetStateEnabled(Enum.HumanoidStateType.StrafingNoPhysics, true)
        hum:SetStateEnabled(Enum.HumanoidStateType.Swimming, true)
        hum:ChangeState(Enum.HumanoidStateType.RunningNoPhysics)
        hum.PlatformStand = false

        local anim = char:FindFirstChild("Animate")
        if anim then anim.Disabled = false end
        for _, track in pairs(hum:GetPlayingAnimationTracks()) do
            track:AdjustSpeed(1)
        end
    end
end

local function disableHumanoidStates(char)
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if hum then
        hum:SetStateEnabled(Enum.HumanoidStateType.Climbing, false)
        hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
        hum:SetStateEnabled(Enum.HumanoidStateType.Flying, false)
        hum:SetStateEnabled(Enum.HumanoidStateType.Freefall, false)
        hum:SetStateEnabled(Enum.HumanoidStateType.GettingUp, false)
        hum:SetStateEnabled(Enum.HumanoidStateType.Jumping, false)
        hum:SetStateEnabled(Enum.HumanoidStateType.Landed, false)
        hum:SetStateEnabled(Enum.HumanoidStateType.Physics, false)
        hum:SetStateEnabled(Enum.HumanoidStateType.PlatformStanding, false)
        hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
        hum:SetStateEnabled(Enum.HumanoidStateType.Running, false)
        hum:SetStateEnabled(Enum.HumanoidStateType.RunningNoPhysics, false)
        hum:SetStateEnabled(Enum.HumanoidStateType.Seated, false)
        hum:SetStateEnabled(Enum.HumanoidStateType.StrafingNoPhysics, false)
        hum:SetStateEnabled(Enum.HumanoidStateType.Swimming, false)
        hum:ChangeState(Enum.HumanoidStateType.Swimming)
        hum.PlatformStand = true

        local anim = char:FindFirstChild("Animate")
        if anim then anim.Disabled = true end
        for _, track in pairs(hum:GetPlayingAnimationTracks()) do
            track:AdjustSpeed(0)
        end
    end
end

local function startFly()
    if flyActive then return end
    flyActive = true

    local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    disableHumanoidStates(char)

    flyConnection = RunService.Heartbeat:Connect(function()
        local chr = LocalPlayer.Character
        local hum = chr and chr:FindFirstChildOfClass("Humanoid")
        local root = chr and chr:FindFirstChild("HumanoidRootPart")
        if flyActive and chr and hum and hum.Parent and root then
            if hum.MoveDirection.Magnitude > 0 then
                chr:TranslateBy(hum.MoveDirection * flySpeed)
            end
            if flyKeyHeld.Up then
                root.CFrame = root.CFrame * CFrame.new(0, 1 * flySpeed, 0)
            end
            if flyKeyHeld.Down then
                root.CFrame = root.CFrame * CFrame.new(0, -1 * flySpeed, 0)
            end
        end
    end)
    showToast("Полёт", "Полёт активирован! (Скорость: " .. tostring(flySpeed) .. ")", "Success")
end

local function stopFly()
    if not flyActive then return end
    flyActive = false

    if flyConnection then
        flyConnection:Disconnect()
        flyConnection = nil
    end

    local char = LocalPlayer.Character
    enableHumanoidStates(char)
    showToast("Полёт", "Полёт выключен", "Info")
end

-- Обработка горячих клавиш клавиатуры для полёта
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and flyActive then
        if input.KeyCode == Enum.KeyCode.Space or input.KeyCode == Enum.KeyCode.E then
            flyKeyHeld.Up = true
        elseif input.KeyCode == Enum.KeyCode.LeftShift or input.KeyCode == Enum.KeyCode.Q then
            flyKeyHeld.Down = true
        end
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.Space or input.KeyCode == Enum.KeyCode.E then
        flyKeyHeld.Up = false
    elseif input.KeyCode == Enum.KeyCode.LeftShift or input.KeyCode == Enum.KeyCode.Q then
        flyKeyHeld.Down = false
    end
end)

-- Плавающий мини-пульт полёта
local flyMiniHud = Instance.new("Frame")
flyMiniHud.Name = "TriOX_FlyHUD"
flyMiniHud.Size = UDim2.new(0, 180, 0, 48)
flyMiniHud.Position = UDim2.new(0.5, -90, 0.85, 0)
flyMiniHud.BackgroundColor3 = theme.SurfaceContainerHighest
flyMiniHud.BorderSizePixel = 0
flyMiniHud.Visible = false
flyMiniHud.Active = true
flyMiniHud.ZIndex = 80
flyMiniHud.Parent = mainGUI

local fCorner = Instance.new("UICorner")
fCorner.CornerRadius = UDim.new(1, 0)
fCorner.Parent = flyMiniHud

local fStroke = Instance.new("UIStroke")
fStroke.Color = theme.Primary
fStroke.Thickness = 1.5
fStroke.Parent = flyMiniHud

makeDraggable(flyMiniHud, flyMiniHud)

local function buildFlyMiniHud()
    local hudLayout = Instance.new("UIListLayout")
    hudLayout.FillDirection = Enum.FillDirection.Horizontal
    hudLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    hudLayout.VerticalAlignment = Enum.VerticalAlignment.Center
    hudLayout.Padding = UDim.new(0, 6)
    hudLayout.Parent = flyMiniHud

    local function miniBtn(text, onClick, onDown, onUp)
        local b = Instance.new("TextButton")
        b.Size = UDim2.new(0, 36, 0, 36)
        b.BackgroundColor3 = theme.PrimaryContainer
        b.Font = Enum.Font.GothamBold
        b.TextSize = 13
        b.TextColor3 = theme.OnPrimaryContainer
        b.Text = text
        b.Parent = flyMiniHud

        local c = Instance.new("UICorner")
        c.CornerRadius = UDim.new(1, 0)
        c.Parent = b

        if onClick then b.MouseButton1Click:Connect(onClick) end
        if onDown then b.MouseButton1Down:Connect(onDown) end
        if onUp then
            b.MouseButton1Up:Connect(onUp)
            b.MouseLeave:Connect(onUp)
        end
        return b
    end

    miniBtn("▲", nil, function()
        if flyActive then
            upHoldConnection = RunService.Heartbeat:Connect(function()
                local c = LocalPlayer.Character
                if c and c:FindFirstChild("HumanoidRootPart") then
                    c.HumanoidRootPart.CFrame = c.HumanoidRootPart.CFrame * CFrame.new(0, 1 * flySpeed, 0)
                end
            end)
        end
    end, function()
        if upHoldConnection then upHoldConnection:Disconnect(); upHoldConnection = nil end
    end)

    miniBtn("▼", nil, function()
        if flyActive then
            downHoldConnection = RunService.Heartbeat:Connect(function()
                local c = LocalPlayer.Character
                if c and c:FindFirstChild("HumanoidRootPart") then
                    c.HumanoidRootPart.CFrame = c.HumanoidRootPart.CFrame * CFrame.new(0, -1 * flySpeed, 0)
                end
            end)
        end
    end, function()
        if downHoldConnection then downHoldConnection:Disconnect(); downHoldConnection = nil end
    end)

    local spdLabel = Instance.new("TextLabel")
    spdLabel.Size = UDim2.new(0, 42, 0, 36)
    spdLabel.BackgroundTransparency = 1
    spdLabel.Font = Enum.Font.GothamBold
    spdLabel.TextSize = 13
    spdLabel.TextColor3 = theme.OnSurface
    spdLabel.Text = flySpeed .. "x"
    spdLabel.Parent = flyMiniHud

    miniBtn("✕", function()
        flyMiniHud.Visible = false
    end)
end
buildFlyMiniHud()

-- ══════════════════════════════════════════════════════════════════════
-- 7. МОДУЛЬ 2: ТЕЛЕКИНЕЗ (TELEKINESIS)
-- ══════════════════════════════════════════════════════════════════════

local telekinesisTool = nil
local telekinesisHolding = false
local telekinesisDist = 15

local function removeTelekinesisTool()
    local bp = LocalPlayer:FindFirstChildOfClass("Backpack")
    if bp then
        local t = bp:FindFirstChild("Телекинез")
        if t then t:Destroy() end
    end
    if LocalPlayer.Character then
        local t = LocalPlayer.Character:FindFirstChild("Телекинез")
        if t then t:Destroy() end
    end
    telekinesisTool = nil
end

local function giveTelekinesisTool()
    removeTelekinesisTool()

    local tool = Instance.new("Tool")
    tool.Name = "Телекинез"
    tool.RequiresHandle = true
    tool.CanBeDropped = false

    local handle = Instance.new("Part")
    handle.Name = "Handle"
    handle.Size = Vector3.new(0.6, 0.6, 0.6)
    handle.Transparency = 0.5
    handle.Material = Enum.Material.Neon
    handle.Color = theme.Primary
    handle.CanCollide = false
    handle.Parent = tool

    local selectionBox = Instance.new("SelectionBox")
    selectionBox.LineThickness = 0.05
    selectionBox.Color3 = theme.Primary
    selectionBox.Parent = tool

    local laserPart = Instance.new("Part")
    laserPart.Name = "TelekinesisLaser"
    laserPart.Anchored = true
    laserPart.CanCollide = false
    laserPart.CastShadow = false
    laserPart.Material = Enum.Material.Neon
    laserPart.Color = theme.Primary
    laserPart.Transparency = 0.3
    laserPart.Size = Vector3.new(0.15, 0.15, 1)

    local targetObject = nil
    local bodyPosition = nil
    local activeRenderConn = nil

    tool.Equipped:Connect(function(mouse)
        mouse.Icon = "rbxasset://textures/GunCursor.png"

        mouse.Button1Down:Connect(function()
            if telekinesisHolding then return end
            telekinesisHolding = true

            local target = mouse.Target
            if target and not target.Anchored and target.Parent ~= LocalPlayer.Character then
                targetObject = target
                selectionBox.Adornee = targetObject
                telekinesisDist = math.clamp((targetObject.Position - handle.Position).Magnitude, 5, 200)

                bodyPosition = Instance.new("BodyPosition")
                bodyPosition.Name = "TelekinesisBP"
                bodyPosition.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
                bodyPosition.P = 30000
                bodyPosition.D = 500
                bodyPosition.Position = targetObject.Position
                bodyPosition.Parent = targetObject
            end

            laserPart.Parent = Workspace

            activeRenderConn = RunService.RenderStepped:Connect(function()
                if not telekinesisHolding or not handle.Parent then
                    if activeRenderConn then activeRenderConn:Disconnect(); activeRenderConn = nil end
                    laserPart.Parent = nil
                    return
                end

                local startPos = handle.Position
                local targetPos = mouse.Hit.Position
                if targetObject and targetObject.Parent then
                    local ray = mouse.UnitRay
                    local goalPos = Camera.CFrame.Position + ray.Direction * telekinesisDist
                    if bodyPosition and bodyPosition.Parent then
                        bodyPosition.Position = goalPos
                    end
                    targetPos = targetObject.Position
                end

                local dist = (startPos - targetPos).Magnitude
                laserPart.Size = Vector3.new(0.15, 0.15, dist)
                laserPart.CFrame = CFrame.new(startPos, targetPos) * CFrame.new(0, 0, -dist / 2)
            end)
        end)

        mouse.Button1Up:Connect(function()
            telekinesisHolding = false
            if activeRenderConn then activeRenderConn:Disconnect(); activeRenderConn = nil end
            laserPart.Parent = nil
            if bodyPosition then bodyPosition:Destroy(); bodyPosition = nil end
            selectionBox.Adornee = nil
            targetObject = nil
        end)

        mouse.KeyDown:Connect(function(key)
            key = key:lower()
            if key == "q" then
                telekinesisDist = math.max(5, telekinesisDist - 5)
            elseif key == "e" then
                telekinesisDist = telekinesisDist + 5
            elseif key == "u" then
                if targetObject then
                    local bg = Instance.new("BodyGyro")
                    bg.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
                    bg.CFrame = targetObject.CFrame * CFrame.Angles(0, math.rad(45), 0)
                    bg.Parent = targetObject
                    task.delay(0.5, function() bg:Destroy() end)
                end
            elseif key == "p" then
                if targetObject then
                    local bv = Instance.new("BodyVelocity")
                    bv.MaxForce = Vector3.new(0, math.huge, 0)
                    bv.Velocity = Vector3.new(0, 50, 0)
                    bv.Parent = targetObject
                    task.delay(0.3, function() bv:Destroy() end)
                end
            elseif key == "l" then
                if targetObject then
                    targetObject.Velocity = Vector3.zero
                    targetObject.RotVelocity = Vector3.zero
                end
            elseif key == "y" then
                telekinesisDist = 100
            elseif key == "j" then
                telekinesisDist = 5000
            elseif key == "x" then
                telekinesisDist = 15
            end
        end)
    end)

    tool.Unequipped:Connect(function()
        telekinesisHolding = false
        if activeRenderConn then activeRenderConn:Disconnect(); activeRenderConn = nil end
        laserPart.Parent = nil
        if bodyPosition then bodyPosition:Destroy(); bodyPosition = nil end
        selectionBox.Adornee = nil
        targetObject = nil
    end)

    local bp = LocalPlayer:WaitForChild("Backpack")
    tool.Parent = bp
    telekinesisTool = tool
    showToast("Телекинез", "Инструмент добавлен в ваш инвентарь!", "Success")
end

-- ══════════════════════════════════════════════════════════════════════
-- 8. МОДУЛЬ 3: ПОДСВЕТКА И ESP (HIGHLIGHT)
-- ══════════════════════════════════════════════════════════════════════

local espTargetQuery = "Items"
local espColor = Color3.fromRGB(80, 220, 160)
local espShowName = true
local espShowDistance = true
local activeHighlights = {}
local activeBillboards = {}
local espHeartbeat = nil

local function cleanupESP()
    if espHeartbeat then
        espHeartbeat:Disconnect()
        espHeartbeat = nil
    end
    for _, h in pairs(activeHighlights) do
        if h and h.Parent then h:Destroy() end
    end
    table.clear(activeHighlights)
    for _, b in pairs(activeBillboards) do
        if b and b.Parent then b:Destroy() end
    end
    table.clear(activeBillboards)
end

local function applyESP()
    cleanupESP()

    local targets = {}
    local lowerQuery = string.lower(espTargetQuery)

    if lowerQuery == "players" or lowerQuery == "игроки" then
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character then
                local root = p.Character:FindFirstChild("HumanoidRootPart") or p.Character:FindFirstChildWhichIsA("BasePart")
                if root then table.insert(targets, {Part = root, Name = p.DisplayName .. " (@" .. p.Name .. ")"}) end
            end
        end
    elseif lowerQuery == "tools" or lowerQuery == "предметы" then
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("Tool") and obj:FindFirstChild("Handle") then
                table.insert(targets, {Part = obj.Handle, Name = obj.Name})
            end
        end
    else
        local found = Workspace:FindFirstChild(espTargetQuery)
        if found then
            for _, d in ipairs(found:GetDescendants()) do
                if d:IsA("BasePart") then table.insert(targets, {Part = d, Name = d.Name}) end
            end
            if found:IsA("BasePart") then table.insert(targets, {Part = found, Name = found.Name}) end
        else
            for _, d in ipairs(Workspace:GetDescendants()) do
                if d:IsA("BasePart") and string.find(string.lower(d.Name), lowerQuery, 1, true) then
                    table.insert(targets, {Part = d, Name = d.Name})
                end
            end
        end
    end

    if #targets == 0 then
        showToast("Подсветка", "Объекты по запросу '" .. espTargetQuery .. "' не найдены", "Warning")
        return
    end

    local myRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")

    for _, entry in ipairs(targets) do
        local part = entry.Part
        local h = Instance.new("Highlight")
        h.Name = "TriOX_ESP_Highlight"
        h.FillColor = espColor
        h.OutlineColor = espColor
        h.FillTransparency = 0.5
        h.OutlineTransparency = 0
        h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        h.Adornee = part
        h.Parent = part
        table.insert(activeHighlights, h)

        if espShowName or espShowDistance then
            local bb = Instance.new("BillboardGui")
            bb.Name = "TriOX_ESP_Billboard"
            bb.Adornee = part
            bb.Size = UDim2.new(0, 160, 0, 40)
            bb.StudsOffset = Vector3.new(0, 3, 0)
            bb.AlwaysOnTop = true
            bb.LightInfluence = 0
            bb.MaxDistance = 500

            local lbl = Instance.new("TextLabel")
            lbl.Size = UDim2.new(1, 0, 1, 0)
            lbl.BackgroundTransparency = 1
            lbl.Font = Enum.Font.GothamBold
            lbl.TextSize = 12
            lbl.TextColor3 = Color3.new(1, 1, 1)
            lbl.TextStrokeColor3 = Color3.new(0, 0, 0)
            lbl.TextStrokeTransparency = 0.3

            local dist = myRoot and (myRoot.Position - part.Position).Magnitude or 0
            local txt = ""
            if espShowName and espShowDistance then txt = string.format("%s\n[%.0f studs]", entry.Name, dist)
            elseif espShowName then txt = entry.Name
            elseif espShowDistance then txt = string.format("[%.0f studs]", dist) end

            lbl.Text = txt
            lbl.Parent = bb
            bb.Parent = part
            table.insert(activeBillboards, {Billboard = bb, Label = lbl, Part = part, BaseName = entry.Name})
        end
    end

    showToast("Подсветка", "Подсвечено объектов: " .. #targets, "Success")

    espHeartbeat = RunService.Heartbeat:Connect(function()
        local root = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if not root then return end

        for i = #activeBillboards, 1, -1 do
            local item = activeBillboards[i]
            if item.Part and item.Part.Parent then
                local d = (root.Position - item.Part.Position).Magnitude
                local txt = ""
                if espShowName and espShowDistance then txt = string.format("%s\n[%.0f studs]", item.BaseName, d)
                elseif espShowName then txt = item.BaseName
                elseif espShowDistance then txt = string.format("[%.0f studs]", d) end
                item.Label.Text = txt
            else
                if item.Billboard and item.Billboard.Parent then item.Billboard:Destroy() end
                table.remove(activeBillboards, i)
            end
        end
    end)
end

-- ══════════════════════════════════════════════════════════════════════
-- 9. МОДУЛЬ 4: НАБЛЮДЕНИЕ (SPECTATE)
-- ══════════════════════════════════════════════════════════════════════

local isSpectating = false
local currentSpectateIndex = 1
local lastSpectatedPlayer = nil
local playerEspEnabled = false
local playerEspHighlights = {}

local function getSpectatablePlayers()
    local list = {}
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Humanoid") and p.Character.Humanoid.Health > 0 then
            table.insert(list, p)
        end
    end
    return list
end

local spectateBottomBar = Instance.new("Frame")
spectateBottomBar.Name = "TriOX_SpectateHUD"
spectateBottomBar.Size = UDim2.new(0, 360, 0, 56)
spectateBottomBar.Position = UDim2.new(0.5, -180, 0.88, 0)
spectateBottomBar.BackgroundColor3 = theme.SurfaceContainerHighest
spectateBottomBar.BorderSizePixel = 0
spectateBottomBar.Visible = false
spectateBottomBar.Active = true
spectateBottomBar.ZIndex = 85
spectateBottomBar.Parent = mainGUI

local sCorner = Instance.new("UICorner")
sCorner.CornerRadius = UDim.new(1, 0)
sCorner.Parent = spectateBottomBar

local sStroke = Instance.new("UIStroke")
sStroke.Color = theme.Primary
sStroke.Thickness = 1.5
sStroke.Parent = spectateBottomBar

makeDraggable(spectateBottomBar, spectateBottomBar)

local spectateTargetLabel = nil

local function spectatePlayer(targetPlayer)
    if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("Humanoid") then
        Camera.CameraSubject = targetPlayer.Character.Humanoid
        lastSpectatedPlayer = targetPlayer
        if spectateTargetLabel then
            spectateTargetLabel.Text = "Цель: " .. targetPlayer.DisplayName .. " (@" .. targetPlayer.Name .. ")"
        end
        return true
    end
    return false
end

local function cycleSpectate(direction)
    local list = getSpectatablePlayers()
    if #list == 0 then
        showToast("Наблюдение", "Нет доступных игроков для наблюдения", "Warning")
        return
    end

    currentSpectateIndex = currentSpectateIndex + direction
    if currentSpectateIndex > #list then currentSpectateIndex = 1
    elseif currentSpectateIndex < 1 then currentSpectateIndex = #list end

    spectatePlayer(list[currentSpectateIndex])
end

local function stopSpectating()
    isSpectating = false
    spectateBottomBar.Visible = false
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        Camera.CameraSubject = LocalPlayer.Character.Humanoid
    end
    showToast("Наблюдение", "Наблюдение завершено, камера возвращена", "Info")
end

local function quickTeleportToTarget()
    local target = lastSpectatedPlayer
    local myRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") and myRoot then
        myRoot.CFrame = target.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, 3)
        showToast("Телепорт", "Телепортирован к " .. target.DisplayName, "Success")
    else
        showToast("Телепорт", "Не удалось найти координаты цели", "Error")
    end
end

local function togglePlayerESP(enabled)
    playerEspEnabled = enabled
    if not playerEspEnabled then
        for _, h in pairs(playerEspHighlights) do
            if h and h.Parent then h:Destroy() end
        end
        table.clear(playerEspHighlights)
        showToast("ESP Игроков", "ESP игроков отключен", "Info")
    else
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character then
                local h = Instance.new("Highlight")
                h.Name = "TriOX_PlayerESP"
                h.FillTransparency = 1
                h.OutlineColor = Color3.fromRGB(255, 255, 255)
                h.OutlineTransparency = 0.1
                h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                h.Adornee = p.Character
                h.Parent = p.Character
                playerEspHighlights[p] = h
            end
        end
        showToast("ESP Игроков", "ESP игроков включен (белый контур)", "Success")
    end
end

Players.PlayerRemoving:Connect(function(removedPlayer)
    if isSpectating and lastSpectatedPlayer == removedPlayer then
        cycleSpectate(1)
    end
    if playerEspHighlights[removedPlayer] then
        playerEspHighlights[removedPlayer]:Destroy()
        playerEspHighlights[removedPlayer] = nil
    end
end)

local function buildSpectateHUD()
    local layout = Instance.new("UIListLayout")
    layout.FillDirection = Enum.FillDirection.Horizontal
    layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    layout.VerticalAlignment = Enum.VerticalAlignment.Center
    layout.Padding = UDim.new(0, 8)
    layout.Parent = spectateBottomBar

    local function hudBtn(text, onClick)
        local b = Instance.new("TextButton")
        b.Size = UDim2.new(0, 36, 0, 36)
        b.BackgroundColor3 = theme.PrimaryContainer
        b.Font = Enum.Font.GothamBold
        b.TextSize = 14
        b.TextColor3 = theme.OnPrimaryContainer
        b.Text = text
        b.Parent = spectateBottomBar

        local c = Instance.new("UICorner")
        c.CornerRadius = UDim.new(1, 0)
        c.Parent = b
        b.MouseButton1Click:Connect(onClick)
        return b
    end

    hudBtn("◀", function() cycleSpectate(-1) end)

    spectateTargetLabel = Instance.new("TextLabel")
    spectateTargetLabel.Size = UDim2.new(0, 160, 0, 36)
    spectateTargetLabel.BackgroundTransparency = 1
    spectateTargetLabel.Font = Enum.Font.GothamBold
    spectateTargetLabel.TextSize = 11
    spectateTargetLabel.TextColor3 = theme.OnSurface
    spectateTargetLabel.TextTruncate = Enum.TextTruncate.AtEnd
    spectateTargetLabel.Text = "Цель: Нет"
    spectateTargetLabel.Parent = spectateBottomBar

    hudBtn("▶", function() cycleSpectate(1) end)
    hudBtn("⚡", function() quickTeleportToTarget() end)
    hudBtn("✕", function() stopSpectating() end)
end
buildSpectateHUD()

-- ══════════════════════════════════════════════════════════════════════
-- 10. МОДУЛЬ 5: ИСПОЛНИТЕЛЬ LUA (EXECUTOR)
-- ══════════════════════════════════════════════════════════════════════

local function executeLuaCode(code)
    if not code or code:match("^%s*$") then
        showToast("Исполнитель", "Поле ввода пусто", "Warning")
        return
    end

    local loadFunc = loadstring or (getfenv and getfenv().loadstring)
    if not loadFunc then
        showToast("Исполнитель", "loadstring недоступен в текущей среде", "Error")
        return
    end

    local fn, err = loadFunc(code)
    if fn then
        local success, runErr = pcall(fn)
        if success then
            showToast("Исполнитель", "Скрипт успешно выполнен!", "Success")
        else
            showToast("Ошибка исполнения", tostring(runErr), "Error", 5)
        end
    else
        showToast("Ошибка синтаксиса", tostring(err), "Error", 5)
    end
end

-- ══════════════════════════════════════════════════════════════════════
-- 11. МОДУЛЬ 6: ДОПОЛНИТЕЛЬНЫЕ ЧИТЫ И УТИЛИТЫ (UTILITIES)
-- ══════════════════════════════════════════════════════════════════════

local noclipConnection = nil
local function setNoclip(enabled)
    if enabled then
        noclipConnection = RunService.Stepped:Connect(function()
            local char = LocalPlayer.Character
            if char then
                for _, part in ipairs(char:GetDescendants()) do
                    if part:IsA("BasePart") and part.CanCollide then
                        part.CanCollide = false
                    end
                end
            end
        end)
        showToast("Утилиты", "Noclip включен (прохождение сквозь стены)", "Success")
    else
        if noclipConnection then
            noclipConnection:Disconnect()
            noclipConnection = nil
        end
        showToast("Утилиты", "Noclip выключен", "Info")
    end
end

local infJumpConnection = nil
local function setInfiniteJump(enabled)
    if enabled then
        infJumpConnection = UserInputService.JumpRequest:Connect(function()
            local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
        end)
        showToast("Утилиты", "Бесконечный прыжок включен", "Success")
    else
        if infJumpConnection then
            infJumpConnection:Disconnect()
            infJumpConnection = nil
        end
        showToast("Утилиты", "Бесконечный прыжок выключен", "Info")
    end
end

local fullbrightConnection = nil
local origAmbient = Lighting.Ambient
local origBrightness = Lighting.Brightness
local origClock = Lighting.ClockTime

local function setFullbright(enabled)
    if enabled then
        fullbrightConnection = RunService.RenderStepped:Connect(function()
            Lighting.Ambient = Color3.fromRGB(255, 255, 255)
            Lighting.Brightness = 2
            Lighting.ClockTime = 14
            Lighting.FogEnd = 1e6
            Lighting.GlobalShadows = false
        end)
        showToast("Утилиты", "Fullbright включен (максимальная яркость)", "Success")
    else
        if fullbrightConnection then
            fullbrightConnection:Disconnect()
            fullbrightConnection = nil
        end
        Lighting.Ambient = origAmbient
        Lighting.Brightness = origBrightness
        Lighting.ClockTime = origClock
        Lighting.GlobalShadows = true
        showToast("Утилиты", "Fullbright выключен", "Info")
    end
end

local antiAfkConnection = nil
local function setAntiAfk(enabled)
    if enabled then
        antiAfkConnection = LocalPlayer.Idled:Connect(function()
            local VirtualUser = game:GetService("VirtualUser")
            if VirtualUser then
                VirtualUser:CaptureController()
                VirtualUser:ClickButton2(Vector2.zero)
            end
        end)
        showToast("Утилиты", "Anti-AFK активирован (защита от дисконнекта)", "Success")
    else
        if antiAfkConnection then
            antiAfkConnection:Disconnect()
            antiAfkConnection = nil
        end
        showToast("Утилиты", "Anti-AFK отключен", "Info")
    end
end

local function giveClassicBTools()
    local bp = LocalPlayer:FindFirstChildOfClass("Backpack") or LocalPlayer:WaitForChild("Backpack")
    local tools = {
        {Name = "Move Tool", Id = 1},
        {Name = "Delete Tool", Id = 2},
        {Name = "Copy Tool", Id = 3},
        {Name = "Resize Tool", Id = 4}
    }
    for _, t in ipairs(tools) do
        local hopper = Instance.new("HopperBin")
        hopper.BinType = t.Id
        hopper.Name = t.Name
        hopper.Parent = bp
    end
    showToast("Утилиты", "Классические BTools добавлены в инвентарь!", "Success")
end

local function rejoinServer()
    showToast("Сервер", "Перезаход на сервер...", "Info")
    if #Players:GetPlayers() <= 1 then
        LocalPlayer:Kick("\nПереподключение к серверу...")
        task.wait(0.5)
        TeleportService:Teleport(game.PlaceId, LocalPlayer)
    else
        TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, LocalPlayer)
    end
end

-- Обработка респауна персонажа (сброс флагов)
LocalPlayer.CharacterAdded:Connect(function()
    if flyActive then
        stopFly()
    end
    if isSpectating then
        stopSpectating()
    end
end)

-- ══════════════════════════════════════════════════════════════════════
-- 12. ПОСТРОЕНИЕ СТРАНИЦ ИНТЕРФЕЙСА (MD3 EXPRESSIVE CONTENT)
-- ══════════════════════════════════════════════════════════════════════

-- ──────────────────────────────────────────────────────────────────────
-- СТРАНИЦА 1: 🏠 ГЛАВНАЯ (HOME)
-- ──────────────────────────────────────────────────────────────────────
local homePage = registerTab("Home", "Главная", "🏠", 1)

local _, heroContent = createMD3Card(homePage, nil, nil, 1)
local heroRow = Instance.new("Frame")
heroRow.Size = UDim2.new(1, 0, 0, 60)
heroRow.BackgroundTransparency = 1
heroRow.Parent = heroContent

local avatarImg = Instance.new("ImageLabel")
avatarImg.Size = UDim2.new(0, 56, 0, 56)
avatarImg.Position = UDim2.new(0, 0, 0.5, -28)
avatarImg.BackgroundColor3 = theme.SurfaceContainerHighest
avatarImg.Image = "rbxthumb://type=AvatarHeadShot&id=" .. LocalPlayer.UserId .. "&w=150&h=150"
avatarImg.Parent = heroRow

local avCorner = Instance.new("UICorner")
avCorner.CornerRadius = UDim.new(1, 0)
avCorner.Parent = avatarImg

local avStroke = Instance.new("UIStroke")
avStroke.Color = theme.Primary
avStroke.Thickness = 2
avStroke.Parent = avatarImg

local welcomeLbl = Instance.new("TextLabel")
welcomeLbl.Size = UDim2.new(1, -70, 0, 22)
welcomeLbl.Position = UDim2.new(0, 68, 0, 6)
welcomeLbl.BackgroundTransparency = 1
welcomeLbl.Font = Enum.Font.GothamBold
welcomeLbl.TextSize = 16
welcomeLbl.TextColor3 = theme.OnSurface
welcomeLbl.TextXAlignment = Enum.TextXAlignment.Left
welcomeLbl.Text = "Привет, " .. LocalPlayer.DisplayName
welcomeLbl.Parent = heroRow

local handleLbl = Instance.new("TextLabel")
handleLbl.Size = UDim2.new(1, -70, 0, 16)
handleLbl.Position = UDim2.new(0, 68, 0, 30)
handleLbl.BackgroundTransparency = 1
handleLbl.Font = Enum.Font.GothamMedium
handleLbl.TextSize = 12
handleLbl.TextColor3 = theme.Primary
handleLbl.TextXAlignment = Enum.TextXAlignment.Left
handleLbl.Text = "@" .. LocalPlayer.Name .. " • MD3 Expressive Ready"
handleLbl.Parent = heroRow

-- Быстрый статус
local _, statsContent = createMD3Card(homePage, "Статус игры и сервера", "Оперативная информация об игре", 2)
local statsGrid = Instance.new("Frame")
statsGrid.Size = UDim2.new(1, 0, 0, 48)
statsGrid.BackgroundTransparency = 1
statsGrid.Parent = statsContent

local statLayout = Instance.new("UIListLayout")
statLayout.FillDirection = Enum.FillDirection.Horizontal
statLayout.Padding = UDim.new(0, 8)
statLayout.Parent = statsGrid

local function statChip(title, val)
    local f = Instance.new("Frame")
    f.Size = UDim2.new(0.31, 0, 1, 0)
    f.BackgroundColor3 = theme.SurfaceContainerHigh
    f.BorderSizePixel = 0
    f.Parent = statsGrid

    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, 12)
    c.Parent = f

    local t = Instance.new("TextLabel")
    t.Size = UDim2.new(1, 0, 0, 16)
    t.Position = UDim2.new(0, 0, 0, 6)
    t.BackgroundTransparency = 1
    t.Font = Enum.Font.Gotham
    t.TextSize = 10
    t.TextColor3 = theme.OnSurfaceVariant
    t.Text = title
    t.Parent = f

    local v = Instance.new("TextLabel")
    v.Size = UDim2.new(1, 0, 0, 20)
    v.Position = UDim2.new(0, 0, 0, 22)
    v.BackgroundTransparency = 1
    v.Font = Enum.Font.GothamBold
    v.TextSize = 13
    v.TextColor3 = theme.Primary
    v.Text = val
    v.Parent = f
end

statChip("Игроков на сервере", tostring(#Players:GetPlayers()))
statChip("ID Места (PlaceId)", tostring(game.PlaceId))
statChip("Пинг / Движок", "60 FPS • Luau")

-- Быстрый доступ
local _, quickContent = createMD3Card(homePage, "Быстрый запуск модулей", "Мгновенное переключение основных функций", 3)
local qBtns = Instance.new("Frame")
qBtns.Size = UDim2.new(1, 0, 0, 80)
qBtns.BackgroundTransparency = 1
qBtns.Parent = quickContent

local qLayout = Instance.new("UIGridLayout")
qLayout.CellSize = UDim2.new(0.48, 0, 0, 36)
qLayout.CellPadding = UDim2.new(0.04, 0, 0, 8)
qLayout.Parent = qBtns

createMD3Button(qBtns, "🕊️ Меню полёта", "Secondary", function() switchTab("Fly") end)
createMD3Button(qBtns, "🔮 Телекинез", "Secondary", function() switchTab("Telekinesis") end)
createMD3Button(qBtns, "👁️ Подсветка (ESP)", "Secondary", function() switchTab("ESP") end)
createMD3Button(qBtns, "🔭 Наблюдение", "Secondary", function() switchTab("Spectate") end)

-- ──────────────────────────────────────────────────────────────────────
-- СТРАНИЦА 2: 🕊️ ПОЛЁТ (FLY)
-- ──────────────────────────────────────────────────────────────────────
local flyPage = registerTab("Fly", "Полёт", "🕊️", 2)

local _, flyToggleContent = createMD3Card(flyPage, "Режим парения", "Включение и выключение полёта через TranslateBy", 1)
createMD3Switch(flyToggleContent, "Активировать полёт", "WASD / Пробел / Shift для перемещения в воздухе", flyActive, function(state)
    if state then startFly() else stopFly() end
end)

local _, flySpeedContent = createMD3Card(flyPage, "Скорость полёта (Speed)", "Множитель шага перемещения", 2)
local spdRow = Instance.new("Frame")
spdRow.Size = UDim2.new(1, 0, 0, 38)
spdRow.BackgroundTransparency = 1
spdRow.Parent = flySpeedContent

local spdLayout = Instance.new("UIListLayout")
spdLayout.FillDirection = Enum.FillDirection.Horizontal
spdLayout.Padding = UDim.new(0, 6)
spdLayout.Parent = spdRow

local currentSpeedDisplay = Instance.new("TextLabel")
currentSpeedDisplay.Size = UDim2.new(0.3, 0, 1, 0)
currentSpeedDisplay.BackgroundColor3 = theme.SurfaceContainerHigh
currentSpeedDisplay.BorderSizePixel = 0
currentSpeedDisplay.Font = Enum.Font.GothamBold
currentSpeedDisplay.TextSize = 14
currentSpeedDisplay.TextColor3 = theme.Primary
currentSpeedDisplay.Text = flySpeed .. "x"
currentSpeedDisplay.Parent = spdRow

local csCorner = Instance.new("UICorner")
csCorner.CornerRadius = UDim.new(0, 10)
csCorner.Parent = currentSpeedDisplay

local function updateSpeedLabel()
    currentSpeedDisplay.Text = flySpeed .. "x"
end

local function addSpeedBtn(text, delta, exact)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0.15, 0, 1, 0)
    b.BackgroundColor3 = theme.SurfaceContainerHighest
    b.BorderSizePixel = 0
    b.Font = Enum.Font.GothamBold
    b.TextSize = 12
    b.TextColor3 = theme.OnSurface
    b.Text = text
    b.Parent = spdRow

    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(1, 0)
    c.Parent = b

    b.MouseButton1Click:Connect(function()
        if exact then flySpeed = exact
        else flySpeed = math.clamp(flySpeed + delta, 1, 50) end
        updateSpeedLabel()
    end)
end

addSpeedBtn("- 1", -1)
addSpeedBtn("+ 1", 1)
addSpeedBtn("2x", nil, 2)
addSpeedBtn("5x", nil, 5)

local _, flyVertContent = createMD3Card(flyPage, "Управление высотой", "Нажмите и удерживайте для набора/спуска высоты (или Пробел / Shift)", 3)
local vertRow = Instance.new("Frame")
vertRow.Size = UDim2.new(1, 0, 0, 42)
vertRow.BackgroundTransparency = 1
vertRow.Parent = flyVertContent

local vertLayout = Instance.new("UIListLayout")
vertLayout.FillDirection = Enum.FillDirection.Horizontal
vertLayout.Padding = UDim.new(0, 10)
vertLayout.Parent = vertRow

local function vertBtn(text, factor)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0.48, 0, 1, 0)
    b.BackgroundColor3 = theme.PrimaryContainer
    b.BorderSizePixel = 0
    b.Font = Enum.Font.GothamBold
    b.TextSize = 14
    b.TextColor3 = theme.OnPrimaryContainer
    b.Text = text
    b.Parent = vertRow

    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(1, 0)
    c.Parent = b

    local conn = nil
    b.MouseButton1Down:Connect(function()
        if flyActive then
            conn = RunService.Heartbeat:Connect(function()
                local chr = LocalPlayer.Character
                if chr and chr:FindFirstChild("HumanoidRootPart") then
                    chr.HumanoidRootPart.CFrame = chr.HumanoidRootPart.CFrame * CFrame.new(0, factor * flySpeed, 0)
                end
            end)
        end
    end)
    local function cancel()
        if conn then conn:Disconnect(); conn = nil end
    end
    b.MouseButton1Up:Connect(cancel)
    b.MouseLeave:Connect(cancel)
end

vertBtn("▲ Вверх (Hold)", 1)
vertBtn("▼ Вниз (Hold)", -1)

local _, flyHudCard = createMD3Card(flyPage, "Плавающий пульт полёта", "Компактная плавающая панель на экране", 4)
createMD3Switch(flyHudCard, "Показывать мини-пульт", "Позволяет управлять высотой без открытия окна", flyMiniHudVisible, function(state)
    flyMiniHud.Visible = state
end)

-- ──────────────────────────────────────────────────────────────────────
-- СТРАНИЦА 3: 🔮 ТЕЛЕКИНЕЗ (TELEKINESIS)
-- ──────────────────────────────────────────────────────────────────────
local telePage = registerTab("Telekinesis", "Телекинез", "🔮", 3)

local _, teleMainContent = createMD3Card(telePage, "Физический Телекинез", "Захват и манипуляция незакрепленными объектами с помощью луча", 1)
createMD3Button(teleMainContent, "✦ Получить инструмент в инвентарь", "Primary", function()
    giveTelekinesisTool()
end)

createMD3Button(teleMainContent, "✕ Убрать инструмент", "Danger", function()
    removeTelekinesisTool()
    showToast("Телекинез", "Инструмент удален из инвентаря", "Info")
end)

local _, teleGuideContent = createMD3Card(telePage, "Управление телекинезом (Горячие клавиши)", "Используйте клавиши на клавиатуре во время захвата объекта:", 2)

local guideKeys = {
    {"ЛКМ", "Захватить / удерживать физический предмет"},
    {"Q / E", "Приблизить / отдалить захваченный предмет"},
    {"U", "Вращение объекта (BodyGyro)"},
    {"P", "Подбросить объект вверх"},
    {"L", "Заморозить вращение объекта"},
    {"Y", "Установить дистанцию 100 studs"},
    {"J", "Бросок вдаль (дистанция 5000 studs!)"},
    {"X", "Сброс дистанции на 15 studs"}
}

for _, item in ipairs(guideKeys) do
    local kRow = Instance.new("Frame")
    kRow.Size = UDim2.new(1, 0, 0, 24)
    kRow.BackgroundTransparency = 1
    kRow.Parent = teleGuideContent

    local kBadge = Instance.new("TextLabel")
    kBadge.Size = UDim2.new(0, 60, 0, 20)
    kBadge.BackgroundColor3 = theme.SurfaceContainerHighest
    kBadge.BorderSizePixel = 0
    kBadge.Font = Enum.Font.GothamBold
    kBadge.TextSize = 11
    kBadge.TextColor3 = theme.Primary
    kBadge.Text = item[1]
    kBadge.Parent = kRow

    local kbCorner = Instance.new("UICorner")
    kbCorner.CornerRadius = UDim.new(0, 6)
    kbCorner.Parent = kBadge

    local kDesc = Instance.new("TextLabel")
    kDesc.Size = UDim2.new(1, -70, 0, 20)
    kDesc.Position = UDim2.new(0, 68, 0, 0)
    kDesc.BackgroundTransparency = 1
    kDesc.Font = Enum.Font.Gotham
    kDesc.TextSize = 11
    kDesc.TextColor3 = theme.OnSurfaceVariant
    kDesc.TextXAlignment = Enum.TextXAlignment.Left
    kDesc.Text = item[2]
    kDesc.Parent = kRow
end

-- ──────────────────────────────────────────────────────────────────────
-- СТРАНИЦА 4: 👁️ ПОДСВЕТКА И ESP
-- ──────────────────────────────────────────────────────────────────────
local espPage = registerTab("ESP", "Подсветка", "👁️", 4)

local _, espSearchContent = createMD3Card(espPage, "Объект для подсветки", "Имя объекта, путь в Workspace или пресет", 1)
createMD3Input(espSearchContent, "Имя или путь:", "например, Items, Players, Tools", espTargetQuery, function(val)
    espTargetQuery = val
end)

local pRow = Instance.new("Frame")
pRow.Size = UDim2.new(1, 0, 0, 30)
pRow.BackgroundTransparency = 1
pRow.Parent = espSearchContent

local pLayout = Instance.new("UIListLayout")
pLayout.FillDirection = Enum.FillDirection.Horizontal
pLayout.Padding = UDim.new(0, 8)
pLayout.Parent = pRow

local function presetChip(label, query)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0.3, 0, 1, 0)
    b.BackgroundColor3 = theme.SurfaceContainerHighest
    b.BorderSizePixel = 0
    b.Font = Enum.Font.GothamMedium
    b.TextSize = 11
    b.TextColor3 = theme.OnSurface
    b.Text = label
    b.Parent = pRow

    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(1, 0)
    c.Parent = b

    b.MouseButton1Click:Connect(function()
        espTargetQuery = query
        showToast("Пресет", "Выбран пресет: " .. label, "Info")
    end)
end

presetChip("Игроки", "Players")
presetChip("Инструменты", "Tools")
presetChip("Items (Папка)", "Items")

local _, espSettingsContent = createMD3Card(espPage, "Настройки отображения", "Параметры меток над объектами", 2)
createMD3Switch(espSettingsContent, "Показывать название объекта", nil, espShowName, function(s) espShowName = s end)
createMD3Switch(espSettingsContent, "Показывать дистанцию в studs", nil, espShowDistance, function(s) espShowDistance = s end)

local _, espColorContent = createMD3Card(espPage, "Цвет подсветки", "Выберите желаемый оттенок Highlight", 3)
local cRow = Instance.new("Frame")
cRow.Size = UDim2.new(1, 0, 0, 36)
cRow.BackgroundTransparency = 1
cRow.Parent = espColorContent

local cLayout = Instance.new("UIListLayout")
cLayout.FillDirection = Enum.FillDirection.Horizontal
cLayout.Padding = UDim.new(0, 10)
cLayout.Parent = cRow

local colorPresets = {
    {Color = Color3.fromRGB(80, 220, 160), Name = "Изумруд"},
    {Color = Color3.fromRGB(0, 200, 255), Name = "Циан"},
    {Color = Color3.fromRGB(176, 133, 255), Name = "Фиолетовый"},
    {Color = Color3.fromRGB(255, 180, 70), Name = "Янтарь"},
    {Color = Color3.fromRGB(255, 100, 120), Name = "Красный"},
    {Color = Color3.fromRGB(255, 255, 255), Name = "Белый"}
}

for _, cp in ipairs(colorPresets) do
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0, 28, 0, 28)
    b.BackgroundColor3 = cp.Color
    b.BorderSizePixel = 0
    b.Text = ""
    b.Parent = cRow

    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(1, 0)
    c.Parent = b

    b.MouseButton1Click:Connect(function()
        espColor = cp.Color
        showToast("Цвет ESP", "Выбран цвет: " .. cp.Name, "Info")
    end)
end

local _, espActionContent = createMD3Card(espPage, "Управление подсветкой", nil, 4)
createMD3Button(espActionContent, "✦ Применить подсветку", "Primary", function() applyESP() end)
createMD3Button(espActionContent, "✕ Очистить все метки", "Danger", function()
    cleanupESP()
    showToast("Подсветка", "Все метки и подсветка очищены", "Info")
end)

-- ──────────────────────────────────────────────────────────────────────
-- СТРАНИЦА 5: 🔭 НАБЛЮДЕНИЕ (SPECTATE)
-- ──────────────────────────────────────────────────────────────────────
local spectatePage = registerTab("Spectate", "Наблюдение", "🔭", 5)

local _, specTargetCard = createMD3Card(spectatePage, "Цель наблюдения", "Переключайтесь между живыми игроками сервера", 1)
local specControlsRow = Instance.new("Frame")
specControlsRow.Size = UDim2.new(1, 0, 0, 42)
specControlsRow.BackgroundTransparency = 1
specControlsRow.Parent = specTargetCard

local scLayout = Instance.new("UIListLayout")
scLayout.FillDirection = Enum.FillDirection.Horizontal
scLayout.Padding = UDim.new(0, 10)
scLayout.Parent = specControlsRow

local function specCycleBtn(text, dir)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0.48, 0, 1, 0)
    b.BackgroundColor3 = theme.PrimaryContainer
    b.BorderSizePixel = 0
    b.Font = Enum.Font.GothamBold
    b.TextSize = 13
    b.TextColor3 = theme.OnPrimaryContainer
    b.Text = text
    b.Parent = specControlsRow

    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(1, 0)
    c.Parent = b

    b.MouseButton1Click:Connect(function()
        isSpectating = true
        spectateBottomBar.Visible = true
        cycleSpectate(dir)
    end)
end

specCycleBtn("◀ Предыдущий", -1)
specCycleBtn("Следующий ▶", 1)

local _, specActionsCard = createMD3Card(spectatePage, "Действия спектратора", nil, 2)
createMD3Button(specActionsCard, "⚡ Быстрый телепорт к цели", "Primary", function()
    quickTeleportToTarget()
end)

createMD3Button(specActionsCard, "👁️ Включить/выключить ESP всех игроков", "Secondary", function()
    togglePlayerESP(not playerEspEnabled)
end)

createMD3Button(specActionsCard, "⏹ Вернуть камеру на себя", "Danger", function()
    stopSpectating()
end)

-- ──────────────────────────────────────────────────────────────────────
-- СТРАНИЦА 6: ⚡ ИСПОЛНИТЕЛЬ LUA (EXECUTOR)
-- ──────────────────────────────────────────────────────────────────────
local execPage = registerTab("Executor", "Исполнитель", "⚡", 6)

local _, execCard = createMD3Card(execPage, "Встроенный Lua-исполнитель", "Редактор для запуска пользовательских скриптов", 1)

local codeBoxContainer = Instance.new("Frame")
codeBoxContainer.Size = UDim2.new(1, 0, 0, 140)
codeBoxContainer.BackgroundColor3 = theme.CodeBg
codeBoxContainer.BorderSizePixel = 0
codeBoxContainer.Parent = execCard

local cbCorner = Instance.new("UICorner")
cbCorner.CornerRadius = UDim.new(0, 12)
cbCorner.Parent = codeBoxContainer

local cbStroke = Instance.new("UIStroke")
cbStroke.Color = theme.OutlineVariant
cbStroke.Thickness = 1
cbStroke.Parent = codeBoxContainer

local codeTextBox = Instance.new("TextBox")
codeTextBox.Size = UDim2.new(1, -16, 1, -16)
codeTextBox.Position = UDim2.new(0, 8, 0, 8)
codeTextBox.BackgroundTransparency = 1
codeTextBox.Font = Enum.Font.Code
codeTextBox.TextSize = 12
codeTextBox.TextColor3 = theme.OnSurface
codeTextBox.PlaceholderColor3 = theme.Outline
codeTextBox.PlaceholderText = "-- Введите ваш Lua-код здесь...\nprint('TriOX MD3 Hub!')"
codeTextBox.Text = ""
codeTextBox.MultiLine = true
codeTextBox.ClearTextOnFocus = false
codeTextBox.TextXAlignment = Enum.TextXAlignment.Left
codeTextBox.TextYAlignment = Enum.TextYAlignment.Top
codeTextBox.Parent = codeBoxContainer

local btnRow = Instance.new("Frame")
btnRow.Size = UDim2.new(1, 0, 0, 38)
btnRow.BackgroundTransparency = 1
btnRow.Parent = execCard

local bLayout = Instance.new("UIListLayout")
bLayout.FillDirection = Enum.FillDirection.Horizontal
bLayout.Padding = UDim.new(0, 10)
bLayout.Parent = btnRow

local execBtn = Instance.new("TextButton")
execBtn.Size = UDim2.new(0.58, 0, 1, 0)
execBtn.BackgroundColor3 = theme.Primary
execBtn.BorderSizePixel = 0
execBtn.Font = Enum.Font.GothamBold
execBtn.TextSize = 13
execBtn.TextColor3 = theme.OnPrimaryContainer
execBtn.Text = "▶ Выполнить (Execute)"
execBtn.Parent = btnRow

local ebCorner = Instance.new("UICorner")
ebCorner.CornerRadius = UDim.new(1, 0)
ebCorner.Parent = execBtn

execBtn.MouseButton1Click:Connect(function()
    executeLuaCode(codeTextBox.Text)
end)

local clearBtn = Instance.new("TextButton")
clearBtn.Size = UDim2.new(0.38, 0, 1, 0)
clearBtn.BackgroundColor3 = theme.SurfaceContainerHighest
clearBtn.BorderSizePixel = 0
clearBtn.Font = Enum.Font.GothamBold
clearBtn.TextSize = 13
clearBtn.TextColor3 = theme.OnSurface
clearBtn.Text = "🗑 Очистить"
clearBtn.Parent = btnRow

local clCorner = Instance.new("UICorner")
clCorner.CornerRadius = UDim.new(1, 0)
clCorner.Parent = clearBtn

clearBtn.MouseButton1Click:Connect(function()
    codeTextBox.Text = ""
end)

-- Сниппеты
local _, snippetCard = createMD3Card(execPage, "Быстрые сниппеты", "Нажмите для мгновенной вставки кода", 2)
local sGrid = Instance.new("Frame")
sGrid.Size = UDim2.new(1, 0, 0, 70)
sGrid.BackgroundTransparency = 1
sGrid.Parent = snippetCard

local sgl = Instance.new("UIGridLayout")
sgl.CellSize = UDim2.new(0.48, 0, 0, 30)
sgl.CellPadding = UDim2.new(0.04, 0, 0, 6)
sgl.Parent = sGrid

local function snipBtn(label, code)
    local b = Instance.new("TextButton")
    b.BackgroundColor3 = theme.SurfaceContainerHigh
    b.BorderSizePixel = 0
    b.Font = Enum.Font.GothamMedium
    b.TextSize = 11
    b.TextColor3 = theme.OnSurface
    b.Text = label
    b.Parent = sGrid

    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(1, 0)
    c.Parent = b

    b.MouseButton1Click:Connect(function()
        codeTextBox.Text = code
        showToast("Сниппет", "Сниппет '" .. label .. "' вставлен", "Info")
    end)
end

snipBtn("Скорость x2", "game.Players.LocalPlayer.Character.Humanoid.WalkSpeed = 32")
snipBtn("Высокий прыжок", "game.Players.LocalPlayer.Character.Humanoid.JumpPower = 100")
snipBtn("Телепорт вверх", "local hrp = game.Players.LocalPlayer.Character.HumanoidRootPart; hrp.CFrame = hrp.CFrame * CFrame.new(0, 50, 0)")
snipBtn("Infinite Yield (Запуск)", "loadstring(game:HttpGet('https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source'))()")

-- ──────────────────────────────────────────────────────────────────────
-- СТРАНИЦА 7: 🛠️ УТИЛИТЫ (UTILITIES)
-- ──────────────────────────────────────────────────────────────────────
local utilPage = registerTab("Utils", "Утилиты", "🛠️", 7)

local _, utilPhysCard = createMD3Card(utilPage, "Физика персонажа", "Управление скоростью и высотой прыжка", 1)

local wsRow = Instance.new("Frame")
wsRow.Size = UDim2.new(1, 0, 0, 34)
wsRow.BackgroundTransparency = 1
wsRow.Parent = utilPhysCard

local wsLayout = Instance.new("UIListLayout")
wsLayout.FillDirection = Enum.FillDirection.Horizontal
wsLayout.Padding = UDim.new(0, 6)
wsLayout.Parent = wsRow

local wsLabel = Instance.new("TextLabel")
wsLabel.Size = UDim2.new(0.3, 0, 1, 0)
wsLabel.BackgroundTransparency = 1
wsLabel.Font = Enum.Font.GothamMedium
wsLabel.TextSize = 12
wsLabel.TextColor3 = theme.OnSurface
wsLabel.Text = "WalkSpeed:"
wsLabel.Parent = wsRow

local function wsBtn(val, lbl)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0.15, 0, 1, 0)
    b.BackgroundColor3 = theme.SurfaceContainerHighest
    b.BorderSizePixel = 0
    b.Font = Enum.Font.GothamBold
    b.TextSize = 11
    b.TextColor3 = theme.OnSurface
    b.Text = lbl or tostring(val)
    b.Parent = wsRow

    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(1, 0)
    c.Parent = b

    b.MouseButton1Click:Connect(function()
        local h = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if h then
            h.WalkSpeed = val
            showToast("WalkSpeed", "Скорость изменена на " .. val, "Info")
        end
    end)
end

wsBtn(16, "16 (Обыч)")
wsBtn(32, "32")
wsBtn(60, "60")
wsBtn(120, "120")

createMD3Switch(utilPhysCard, "Noclip (Сквозь стены)", "Отключает коллизию всех частей тела", false, setNoclip)
createMD3Switch(utilPhysCard, "Infinite Jump", "Бесконечные прыжки в воздухе", false, setInfiniteJump)

local _, utilEnvCard = createMD3Card(utilPage, "Окружение и Сервер", nil, 2)
createMD3Switch(utilEnvCard, "Fullbright (Яркость)", "Максимальное освещение карты без теней", false, setFullbright)
createMD3Switch(utilEnvCard, "Anti-AFK", "Защита от отключения за бездействие (20 мин)", false, setAntiAfk)

createMD3Button(utilEnvCard, "🔨 Выдать BTools (Инструменты строительства)", "Secondary", giveClassicBTools)
createMD3Button(utilEnvCard, "🔄 Перезайти на сервер (Rejoin)", "Secondary", rejoinServer)

-- ──────────────────────────────────────────────────────────────────────
-- СТРАНИЦА 8: ⚙️ НАСТРОЙКИ (SETTINGS)
-- ──────────────────────────────────────────────────────────────────────
local settingsPage = registerTab("Settings", "Настройки", "⚙️", 8)

local _, themeCard = createMD3Card(settingsPage, "Внешний вид и палитра", "Настройка цветовой схемы Material You", 1)

createMD3Switch(themeCard, "Радужная динамическая анимация", "Плавное переливание цветов шапки и акцентов", isRainbowGlowActive, function(s)
    isRainbowGlowActive = s
end)

local paletteRow = Instance.new("Frame")
paletteRow.Size = UDim2.new(1, 0, 0, 36)
paletteRow.BackgroundTransparency = 1
paletteRow.Parent = themeCard

local palLayout = Instance.new("UIListLayout")
palLayout.FillDirection = Enum.FillDirection.Horizontal
palLayout.Padding = UDim.new(0, 8)
palLayout.Parent = paletteRow

for pName, pData in pairs(ColorPalettes) do
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0.18, 0, 1, 0)
    b.BackgroundColor3 = pData.Primary
    b.BorderSizePixel = 0
    b.Font = Enum.Font.GothamBold
    b.TextSize = 11
    b.TextColor3 = Color3.new(0, 0, 0)
    b.Text = pName
    b.Parent = paletteRow

    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(1, 0)
    c.Parent = b

    b.MouseButton1Click:Connect(function()
        currentPaletteName = pName
        applyCurrentTheme()
        showToast("Тема", "Акцент изменен на " .. pName, "Success")
    end)
end

local _, manageCard = createMD3Card(settingsPage, "Управление хабом", nil, 2)
createMD3Button(manageCard, "⟲ Сбросить позицию окна по центру", "Secondary", function()
    mainFrame.Position = UDim2.new(0.5, -320, 0.5, -210)
    showToast("Окно", "Позиция сброшена", "Info")
end)

createMD3Button(manageCard, "✕ Полностью выгрузить скрипт", "Danger", function()
    stopFly()
    cleanupESP()
    stopSpectating()
    setNoclip(false)
    setInfiniteJump(false)
    setFullbright(false)
    setAntiAfk(false)
    removeTelekinesisTool()
    mainGUI:Destroy()
end)

-- ──────────────────────────────────────────────────────────────────────
-- КНОПКИ ВЕРХНЕЙ ПАНЕЛИ
-- ──────────────────────────────────────────────────────────────────────

createTopIconButton("ThemeBtn", "☀", function()
    if currentThemeName == "Dark" then
        currentThemeName = "Light"
    else
        currentThemeName = "Dark"
    end
    applyCurrentTheme()
    showToast("Тема", "Режим переключен на " .. (currentThemeName == "Dark" and "Темный" or "Светлый"), "Info")
end)

createTopIconButton("MinBtn", "−", function()
    mainFrame.Visible = false
    unfoldFAB.Visible = true
    tween(unfoldFAB, 0.35, {Size = UDim2.new(0, 60, 0, 60)}, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
end)

createTopIconButton("CloseBtn", "✕", function()
    stopFly()
    cleanupESP()
    stopSpectating()
    mainGUI:Destroy()
end, theme.DangerContainer, theme.Danger)

-- ══════════════════════════════════════════════════════════════════════
-- 13. ДИНАМИЧЕСКАЯ АНИМАЦИЯ ЦВЕТОВ (MD3 EXPRESSIVE RAINBOW GLOW)
-- ══════════════════════════════════════════════════════════════════════

local rainbowIdx = 1
local transitionSpeed = 3

RunService.Heartbeat:Connect(function(dt)
    if isRainbowGlowActive then
        rainbowProgress = rainbowProgress + (dt / transitionSpeed)
        if rainbowProgress >= 1 then
            rainbowProgress = 0
            rainbowIdx = (rainbowIdx % #rainbowColors) + 1
        end

        local nextIdx = (rainbowIdx % #rainbowColors) + 1
        local c1 = rainbowColors[rainbowIdx]
        local c2 = rainbowColors[nextIdx]
        currentRainbowColor = c1:Lerp(c2, rainbowProgress)

        titleLabel.TextColor3 = currentRainbowColor
        logoBadge.BackgroundColor3 = currentRainbowColor:Lerp(theme.Surface, 0.4)
        fabStroke.Color = currentRainbowColor
        fabBtn.TextColor3 = currentRainbowColor
    end
end)

-- ══════════════════════════════════════════════════════════════════════
-- 14. СТАРТОВАЯ ИНИЦИАЛИЗАЦИЯ
-- ══════════════════════════════════════════════════════════════════════

switchTab("Home")
showToast("TriOX Hub", "Material Design 3 Expressive Edition успешно запущен!", "Success", 4)
