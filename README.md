--[[ AZARO HUB: Menu Branco Moderno com Hover, Fontes Médias, Realce, Logo Customizado, Linha Azul Bebê e Créditos
Feito por: bona 💀 | Modernização e Hover por ChatGPT | ESP Brain por bona71
]]

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Char = LocalPlayer.Character or Players.LocalPlayer.CharacterAdded:Wait()
local Hum = Char:WaitForChild("Humanoid")
local HRP = Char:WaitForChild("HumanoidRootPart")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")
local VirtualInputManager = game:GetService("VirtualInputManager")

local SPEED = 35
local HIGH_JUMP = 100
local SKY_Y = 150
local NORMAL_Y = 10 -- Altura padrão para "voltar pra baixo"
local NORMAL_SPEED = 16
local PlaceId = game.PlaceId

local toggles = { Speed = false, HighJump = false, Steal = false, ESP = false, BaseESP = false, ["ESP Brain do Watch"] = false, AutoSlap = false }
local stealPlatform
local boosted = false
local speedMultiplier = 1.65
local isInSky = false

-- Auto Slap (AUTO BATIDA + AUTO 1)
local autoSlapRunning = false
local autoSlapConn

local function startAutoSlap()
    if autoSlapRunning then return end
    autoSlapRunning = true
    autoSlapConn = RunService.RenderStepped:Connect(function()
        -- Tenta clicar, depois aperta o 1
        pcall(function()
            -- Clique no mouse (simula slap)
            VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 0)
            VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 0)
            -- Aperta tecla 1
            VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.One, false, game)
            VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.One, false, game)
        end)
        task.wait(0.001)
    end)
end

local function stopAutoSlap()
    autoSlapRunning = false
    if autoSlapConn then
        autoSlapConn:Disconnect()
        autoSlapConn = nil
    end
end

-- ESP Brain do Watch
local activeLockTimeEsp = false
local lteInstances = {}
local plotName = nil

do
    local success, result = pcall(function()
        for _, plot in pairs(workspace.Plots:GetChildren()) do
            if plot.Owner and plot.Owner.Value == LocalPlayer then
                plotName = plot.Name
                break
            end
        end
    end)
end

local function updatelock()
    if not activeLockTimeEsp then
        for _, instance in pairs(lteInstances) do
            if instance then instance:Destroy() end
        end
        lteInstances = {}
        return
    end

    for _, plot in pairs(workspace.Plots:GetChildren()) do
        local billboardName = "LockTimeESP_" .. plot.Name
        local timeLabel = plot:FindFirstChild("Purchases", true)
            and plot.Purchases:FindFirstChild("PlotBlock", true)
            and plot.Purchases.PlotBlock.Main:FindFirstChild("BillboardGui", true)
            and plot.Purchases.PlotBlock.Main.BillboardGui:FindFirstChild("RemainingTime", true)

        if timeLabel and timeLabel:IsA("TextLabel") then
            local existing = lteInstances[plot.Name]
            local isUnlocked = timeLabel.Text == "0s"
            local displayText = isUnlocked and "Unlocked" or ("Lock: " .. timeLabel.Text)

            local color = Color3.fromRGB(255, 204, 0)
            if plot.Name == plotName then
                color = Color3.fromRGB(80, 220, 80)
            elseif isUnlocked then
                color = Color3.fromRGB(255, 100, 100)
            end

            if not existing then
                local gui = Instance.new("BillboardGui")
                gui.Name = billboardName
                gui.Size = UDim2.new(0, 100, 0, 20)
                gui.StudsOffset = Vector3.new(0, 3, 0)
                gui.AlwaysOnTop = true
                gui.Adornee = plot.Purchases.PlotBlock.Main
                gui.Parent = plot

                local label = Instance.new("TextLabel")
                label.Size = UDim2.new(1, 0, 1, 0)
                label.BackgroundTransparency = 1
                label.TextScaled = true
                label.TextColor3 = color
                label.TextStrokeTransparency = 0.3
                label.TextStrokeColor3 = Color3.fromRGB(235,235,235)
                label.Font = Enum.Font.GothamBold
                label.Text = displayText
                label.Parent = gui

                lteInstances[plot.Name] = gui
            else
                local label = existing:FindFirstChildOfClass("TextLabel")
                if label then
                    label.Text = displayText
                    label.TextColor3 = color
                end
            end
        end
    end
end

------------------ NOTIFICAÇÃO ---------------------
local function AzaroNotify(msg)
    if not game.CoreGui:FindFirstChild("AzaroHubWhiteModern") then return end
    local gui = game.CoreGui.AzaroHubWhiteModern
    if gui:FindFirstChild("AzaroNotification") then
        gui.AzaroNotification:Destroy()
    end
    local notif = Instance.new("Frame", gui)
    notif.Name = "AzaroNotification"
    notif.AnchorPoint = Vector2.new(1, 1)
    notif.Size = UDim2.new(0, 220, 0, 38)
    notif.Position = UDim2.new(1, -16, 1, -16)
    notif.BackgroundColor3 = Color3.fromRGB(150, 220, 255)
    notif.BackgroundTransparency = 0.05
    notif.ZIndex = 100
    notif.BorderSizePixel = 0
    local function createUICorner(parent, rad)
        local c = Instance.new("UICorner", parent)
        c.CornerRadius = UDim.new(0, rad or 8)
        return c
    end
    createUICorner(notif, 8)

    local label = Instance.new("TextLabel", notif)
    label.Size = UDim2.new(1, -14, 1, 0)
    label.Position = UDim2.new(0, 7, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = msg or "Parabéns, função ativada\nobrigado por usar o Azaro HUB"
    label.Font = Enum.Font.GothamBold
    label.TextSize = 14
    label.TextColor3 = Color3.fromRGB(40, 40, 50)
    label.TextWrapped = true
    label.TextYAlignment = Enum.TextYAlignment.Center
    label.ZIndex = 101

    notif.Visible = true
    notif.ClipsDescendants = false

    task.spawn(function()
        wait(2.5)
        for i = 0, 1, 0.08 do
            notif.BackgroundTransparency = i
            label.TextTransparency = i
            wait(0.03)
        end
        notif:Destroy()
    end)
end

---- STEAL GUI FUNCTION ----
local function openStealGui()
    if game.CoreGui:FindFirstChild("AzaroStealGui") then
        game.CoreGui.AzaroStealGui:Destroy()
    end

    local stealGui = Instance.new("ScreenGui")
    stealGui.Name = "AzaroStealGui"
    stealGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    stealGui.Parent = game.CoreGui

    local bg = Instance.new("Frame")
    bg.Name = "MainFrame"
    bg.Size = UDim2.new(0, 350, 0, 160)
    bg.Position = UDim2.new(0.5, -175, 0.5, -80)
    bg.BackgroundColor3 = Color3.fromRGB(255,255,255)
    bg.BorderSizePixel = 0
    bg.Active = true
    bg.Draggable = true
    bg.Parent = stealGui

    local function createUICorner(parent, rad)
        local c = Instance.new("UICorner")
        c.CornerRadius = UDim.new(0, rad or 12)
        c.Parent = parent
        return c
    end
    createUICorner(bg, 13)

    -- TopBar
    local top = Instance.new("Frame")
    top.Size = UDim2.new(1, 0, 0, 36)
    top.BackgroundTransparency = 1
    top.Parent = bg

    local logo = Instance.new("ImageLabel")
    logo.Size = UDim2.new(0, 26, 0, 26)
    logo.Position = UDim2.new(0, 8, 0, 5)
    logo.BackgroundTransparency = 1
    logo.Image = "rbxassetid://130132379530355"
    logo.ScaleType = Enum.ScaleType.Fit
    logo.Parent = top

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, -40-26, 1, 0)
    title.Position = UDim2.new(0, 38, 0, 1)
    title.BackgroundTransparency = 1
    title.Text = "Steal - Azaro Hub"
    title.Font = Enum.Font.GothamBlack
    title.TextSize = 19
    title.TextColor3 = Color3.fromRGB(80, 80, 110)
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = top

    local babyBlueLine = Instance.new("Frame")
    babyBlueLine.Size = UDim2.new(1, -14, 0, 3)
    babyBlueLine.Position = UDim2.new(0, 6, 1, -4)
    babyBlueLine.BackgroundColor3 = Color3.fromRGB(150, 220, 255)
    babyBlueLine.BorderSizePixel = 0
    babyBlueLine.Parent = top
    local lineCorner = Instance.new("UICorner", babyBlueLine)
    lineCorner.CornerRadius = UDim.new(0, 2)

    -- Botão Fechar
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 32, 0, 32)
    closeBtn.Position = UDim2.new(1, -36, 0, 2)
    closeBtn.BackgroundColor3 = Color3.fromRGB(230, 60, 80)
    closeBtn.BackgroundTransparency = 0
    closeBtn.Text = "✕"
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.TextSize = 16
    closeBtn.TextColor3 = Color3.fromRGB(255,255,255)
    closeBtn.AutoButtonColor = false
    closeBtn.Parent = top
    createUICorner(closeBtn, 9)
    closeBtn.MouseEnter:Connect(function()
        closeBtn.BackgroundColor3 = Color3.fromRGB(255, 80, 80)
    end)
    closeBtn.MouseLeave:Connect(function()
        closeBtn.BackgroundColor3 = Color3.fromRGB(230, 60, 80)
    end)
    closeBtn.MouseButton1Click:Connect(function()
        stealGui:Destroy()
    end)

    -- Conteúdo central
    local info = Instance.new("TextLabel")
    info.Size = UDim2.new(1, -18, 0, 48)
    info.Position = UDim2.new(0, 9, 0, 48)
    info.BackgroundTransparency = 1
    info.Text = "Clique no botão abaixo para ir para o sky (150) ou voltar para baixo."
    info.TextColor3 = Color3.fromRGB(100, 100, 130)
    info.Font = Enum.Font.Gotham
    info.TextSize = 16
    info.TextWrapped = true
    info.TextYAlignment = Enum.TextYAlignment.Top
    info.TextXAlignment = Enum.TextXAlignment.Left
    info.Parent = bg

    -- Botão STEAL
    local stealBtn = Instance.new("TextButton")
    stealBtn.Size = UDim2.new(0, 110, 0, 38)
    stealBtn.Position = UDim2.new(0.5, -55, 0, 108)
    stealBtn.BackgroundColor3 = Color3.fromRGB(150, 220, 255)
    stealBtn.Text = "STEAL"
    stealBtn.Font = Enum.Font.GothamBlack
    stealBtn.TextSize = 18
    stealBtn.TextColor3 = Color3.fromRGB(55, 85, 120)
    stealBtn.AutoButtonColor = false
    stealBtn.Parent = bg
    createUICorner(stealBtn, 7)

    local function updateStealBtn()
        if isInSky then
            stealBtn.Text = "VOLTAR"
        else
            stealBtn.Text = "STEAL"
        end
    end
    updateStealBtn()

    stealBtn.MouseEnter:Connect(function()
        stealBtn.BackgroundColor3 = Color3.fromRGB(120,190,250)
    end)
    stealBtn.MouseLeave:Connect(function()
        stealBtn.BackgroundColor3 = Color3.fromRGB(150,220,255)
    end)
    stealBtn.MouseButton1Click:Connect(function()
        if not isInSky then
            HRP.CFrame = CFrame.new(HRP.Position.X, SKY_Y, HRP.Position.Z)
            if not stealPlatform or not stealPlatform.Parent then
                stealPlatform = Instance.new("Part", workspace)
                stealPlatform.Name = "SkyPlatform"
                stealPlatform.Size = Vector3.new(30,1,30)
                stealPlatform.Position = HRP.Position - Vector3.new(0,3,0)
                stealPlatform.Anchored = true
                stealPlatform.CanCollide = true
                stealPlatform.Transparency = 1
            end
            isInSky = true
            AzaroNotify("Você foi para o sky (150)!")
        else
            if stealPlatform then stealPlatform:Destroy() end
            -- Descida rápida e suave, sem travar o boneco
            local targetY = 10
            local goal = {CFrame = CFrame.new(HRP.Position.X, targetY, HRP.Position.Z)}
            local tween = TweenService:Create(HRP, TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), goal)
            tween:Play()
            isInSky = false
            AzaroNotify("Você voltou para baixo!")
        end
        updateStealBtn()
    end)

    -- Créditos rodapé
    local credit = Instance.new("TextLabel")
    credit.Size = UDim2.new(1, -10, 0, 18)
    credit.Position = UDim2.new(0, 5, 1, -22)
    credit.BackgroundTransparency = 1
    credit.Text = "Feito por Faíska & Mt | Modern GUI by Copilot"
    credit.TextColor3 = Color3.fromRGB(130,130,140)
    credit.Font = Enum.Font.Gotham
    credit.TextSize = 12
    credit.TextYAlignment = Enum.TextYAlignment.Center
    credit.TextXAlignment = Enum.TextXAlignment.Center
    credit.Parent = bg

    local rgb = 0
    game:GetService("RunService").RenderStepped:Connect(function()
        if not bg or not bg.Parent then return end
        rgb = (rgb + 0.5) % 360
        bg.BackgroundColor3 = Color3.fromHSV(rgb/360, 0.09, 1)
    end)
end

local function closeStealGui()
    local gui = game.CoreGui:FindFirstChild("AzaroStealGui")
    if gui then gui:Destroy() end
end

-- GUI Principal
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "AzaroHubWhiteModern"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 410, 0, 325)
frame.Position = UDim2.new(0, 30, 0, 30)
frame.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true

local function createUICorner(parent, rad)
    local c = Instance.new("UICorner", parent)
    c.CornerRadius = UDim.new(0, rad or 12)
    return c
end
createUICorner(frame, 12)

-- RGB Effect bem sutil
local rgb = 0
RunService.RenderStepped:Connect(function()
    rgb = (rgb + 0.3) % 360
    local color = Color3.fromHSV(rgb/360, 0.12, 1)
    frame.BackgroundColor3 = color:lerp(Color3.fromRGB(255, 255, 255), 0.97)
end)

-- Minimizar: vira um botão flutuante para restaurar
local minimized = false
local miniBtn

-- TopBar com Logo, Título, Linha Azul Bebê e Botão Minimizar Azul Bebê
local topBar = Instance.new("Frame", frame)
topBar.Size = UDim2.new(1, 0, 0, 36)
topBar.BackgroundTransparency = 1

local logo = Instance.new("ImageLabel", topBar)
logo.Size = UDim2.new(0, 28, 0, 28)
logo.Position = UDim2.new(0, 6, 0, 4)
logo.BackgroundTransparency = 1
logo.Image = "rbxassetid://130132379530355"
logo.ScaleType = Enum.ScaleType.Fit

local title = Instance.new("TextLabel", topBar)
title.Size = UDim2.new(1, -40-26, 1, 0)
title.Position = UDim2.new(0, 36, 0, 1)
title.BackgroundTransparency = 1
title.Text = "Azaro Hub"
title.Font = Enum.Font.GothamBlack
title.TextSize = 24
title.TextColor3 = Color3.fromRGB(80, 80, 110)
title.TextXAlignment = Enum.TextXAlignment.Left

local babyBlueLine = Instance.new("Frame", topBar)
babyBlueLine.Size = UDim2.new(1, -14, 0, 3)
babyBlueLine.Position = UDim2.new(0, 6, 1, -4)
babyBlueLine.BackgroundColor3 = Color3.fromRGB(150, 220, 255)
babyBlueLine.BorderSizePixel = 0
local lineCorner = Instance.new("UICorner", babyBlueLine)
lineCorner.CornerRadius = UDim.new(0, 2)

-- Botão minimizar azul bebê e funcional
local minimizeBtn = Instance.new("TextButton", topBar)
minimizeBtn.Size = UDim2.new(0, 20, 0, 20)
minimizeBtn.Position = UDim2.new(1, -28, 0, 8)
minimizeBtn.Text = "_"
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.TextSize = 16
minimizeBtn.BackgroundColor3 = Color3.fromRGB(150, 220, 255)
minimizeBtn.BackgroundTransparency = 0
minimizeBtn.TextColor3 = Color3.fromRGB(60, 80, 120)
minimizeBtn.AutoButtonColor = false
createUICorner(minimizeBtn, 6)

minimizeBtn.MouseEnter:Connect(function()
    minimizeBtn.BackgroundColor3 = Color3.fromRGB(110,190,250)
end)
minimizeBtn.MouseLeave:Connect(function()
    minimizeBtn.BackgroundColor3 = Color3.fromRGB(150, 220, 255)
end)

local function minimizeMenu()
    minimized = true
    frame.Visible = false
    if not miniBtn then
        miniBtn = Instance.new("TextButton", gui)
        miniBtn.Size = UDim2.new(0, 90, 0, 28)
        miniBtn.Position = UDim2.new(0, 18, 0, 18)
        miniBtn.BackgroundColor3 = Color3.fromRGB(150, 220, 255)
        miniBtn.Text = "⏫ Azaro Hub"
        miniBtn.Font = Enum.Font.GothamBlack
        miniBtn.TextSize = 14
        miniBtn.TextColor3 = Color3.fromRGB(60, 80, 120)
        miniBtn.AutoButtonColor = false
        createUICorner(miniBtn, 8)
        miniBtn.MouseEnter:Connect(function()
            miniBtn.BackgroundColor3 = Color3.fromRGB(110,190,250)
        end)
        miniBtn.MouseLeave:Connect(function()
            miniBtn.BackgroundColor3 = Color3.fromRGB(150, 220, 255)
        end)
        miniBtn.MouseButton1Click:Connect(function()
            frame.Visible = true
            minimized = false
            miniBtn:Destroy()
            miniBtn = nil
        end)
    end
end

minimizeBtn.MouseButton1Click:Connect(minimizeMenu)

-- Tabs (restante do seu script igual, como já montado acima)
local tabNames = { "Funções", "Visual", "Servidor", "Créditos" }
local tabs = {}
local selectedTab = 1

local tabBar = Instance.new("Frame", frame)
tabBar.Size = UDim2.new(1, -32, 0, 30)
tabBar.Position = UDim2.new(0, 16, 0, 38)
tabBar.BackgroundTransparency = 1

local tabWidth = 68
for i, name in ipairs(tabNames) do
    local tab = Instance.new("TextButton", tabBar)
    tab.Size = UDim2.new(0, tabWidth, 0, 24)
    tab.Position = UDim2.new(0, (i-1)*(tabWidth+7), 0, 0)
    tab.Text = name
    tab.Font = Enum.Font.GothamBold
    tab.TextSize = 13
    tab.BackgroundColor3 = (i==1) and Color3.fromRGB(240,240,255) or Color3.fromRGB(245,245,245)
    tab.TextColor3 = (i==1) and Color3.fromRGB(85,60,220) or Color3.fromRGB(110,110,110)
    tab.AutoButtonColor = false
    createUICorner(tab, 5)

    tab.MouseEnter:Connect(function()
        tab.BackgroundColor3 = Color3.fromRGB(220,220,255)
        tab.TextColor3 = Color3.fromRGB(85,60,220)
    end)
    tab.MouseLeave:Connect(function()
        if selectedTab == i then
            tab.BackgroundColor3 = Color3.fromRGB(240,240,255)
            tab.TextColor3 = Color3.fromRGB(85,60,220)
        else
            tab.BackgroundColor3 = Color3.fromRGB(245,245,245)
            tab.TextColor3 = Color3.fromRGB(110,110,110)
        end
    end)

    tabs[i] = tab
end

local tabFrames = {}
for i=1,#tabNames do
    local f = Instance.new("Frame", frame)
    f.Size = UDim2.new(1, -32, 1, -90)
    f.Position = UDim2.new(0, 16, 0, 70)
    f.BackgroundTransparency = 1
    f.Visible = (i==1)
    tabFrames[i] = f
end

local function selectTab(idx)
    for i=1,#tabFrames do
        tabFrames[i].Visible = (i==idx)
        tabs[i].BackgroundColor3 = (i==idx) and Color3.fromRGB(240,240,255) or Color3.fromRGB(245,245,245)
        tabs[i].TextColor3 = (i==idx) and Color3.fromRGB(85,60,220) or Color3.fromRGB(110,110,110)
    end
    selectedTab = idx
end

for i,tab in ipairs(tabs) do
    tab.MouseButton1Click:Connect(function() selectTab(i) end)
end

local function makeScrollBarRGB(scrollingFrame)
    scrollingFrame.ScrollBarImageColor3 = Color3.fromRGB(255,255,255)
    local hue = 0
    RunService.RenderStepped:Connect(function()
        hue = (hue + 1) % 360
        scrollingFrame.ScrollBarImageColor3 = Color3.fromHSV(hue/360, 0.8, 1)
    end)
end

local function addToggleLine(parent, name, order, callback)
    local container = Instance.new("Frame", parent)
    container.LayoutOrder = order
    container.Size = UDim2.new(1, -12, 0, 44)
    container.Position = UDim2.new(0, 6, 0, (order-1)*54)
    container.BackgroundColor3 = Color3.fromRGB(245,245,255)
    local function createUICorner(parent, rad)
        local c = Instance.new("UICorner", parent)
        c.CornerRadius = UDim.new(0, rad or 8)
        c.Parent = parent
        return c
    end
    createUICorner(container, 8)

    local label = Instance.new("TextLabel", container)
    label.Size = UDim2.new(0.76, 0, 1, 0)
    label.Position = UDim2.new(0, 14, 0, 0)
    label.BackgroundTransparency = 1
    label.Font = Enum.Font.GothamBold
    label.TextSize = 18
    label.TextColor3 = Color3.fromRGB(70,70,120)
    label.Text = name

    if name == "Steal" then
        -- Botão RGB médio para STEAL
        local stealBtn = Instance.new("TextButton", container)
        stealBtn.Size = UDim2.new(0, 86, 0, 34)
        stealBtn.Position = UDim2.new(1, -104, 0.5, -17)
        stealBtn.BackgroundColor3 = Color3.fromRGB(120,180,255)
        stealBtn.Text = "STEAL"
        stealBtn.Font = Enum.Font.GothamBlack
        stealBtn.TextSize = 16
        stealBtn.TextColor3 = Color3.fromRGB(255,255,255)
        stealBtn.AutoButtonColor = false
        createUICorner(stealBtn, 7)

        -- RGB animado
        local hue = 0
        RunService.RenderStepped:Connect(function()
            hue = (hue + 2) % 360
            if toggles.Steal then
                stealBtn.BackgroundColor3 = Color3.fromHSV(hue/360, 0.8, 1)
                stealBtn.TextColor3 = Color3.fromRGB(255,255,255)
            else
                stealBtn.BackgroundColor3 = Color3.fromRGB(210,210,220)
                stealBtn.TextColor3 = Color3.fromRGB(150,150,150)
            end
        end)

        stealBtn.MouseButton1Click:Connect(function()
            toggles.Steal = not toggles.Steal
            callback(toggles.Steal)
            if toggles.Steal then
                openStealGui()
            else
                closeStealGui()
            end
        end)
    else
        -- Para outros toggles normais
        local box = Instance.new("TextButton", container)
        box.Size = UDim2.new(0, 28, 0, 28)
        box.Position = UDim2.new(1, -44, 0.5, -14)
        box.BackgroundColor3 = Color3.fromRGB(222,222,235)
        box.Text = ""
        box.AutoButtonColor = false
        createUICorner(box, 5)

        container.MouseEnter:Connect(function()
            container.BackgroundColor3 = Color3.fromRGB(220, 220, 255)
            label.TextColor3 = Color3.fromRGB(85,60,220)
            box.BackgroundColor3 = toggles[name] and Color3.fromRGB(85, 170, 255) or Color3.fromRGB(210,210,255)
        end)
        container.MouseLeave:Connect(function()
            container.BackgroundColor3 = Color3.fromRGB(245,245,255)
            label.TextColor3 = Color3.fromRGB(70,70,120)
            box.BackgroundColor3 = toggles[name] and Color3.fromRGB(85, 170, 255) or Color3.fromRGB(222,222,235)
        end)

        box.MouseEnter:Connect(function()
            box.BackgroundColor3 = Color3.fromRGB(120,180,255)
        end)
        box.MouseLeave:Connect(function()
            box.BackgroundColor3 = toggles[name] and Color3.fromRGB(85, 170, 255) or Color3.fromRGB(222,222,235)
        end)

        box.MouseButton1Click:Connect(function()
            toggles[name] = not toggles[name]
            box.BackgroundColor3 = toggles[name] and Color3.fromRGB(85, 170, 255) or Color3.fromRGB(222,222,235)
            callback(toggles[name])
            if toggles[name] then AzaroNotify() end
        end)
    end
    return container
end

-- Funções
local scroll1 = Instance.new("ScrollingFrame", tabFrames[1])
scroll1.Size = UDim2.new(1, 0, 1, 0)
scroll1.CanvasSize = UDim2.new(0,0,0,420)
scroll1.ScrollBarThickness = 6
scroll1.BackgroundTransparency = 1
createUICorner(scroll1, 6)
makeScrollBarRGB(scroll1)

addToggleLine(scroll1, "HighJump",  1, function(v) end)
addToggleLine(scroll1, "Steal",     2, function(v) end)
addToggleLine(scroll1, "Speed", 3, function(v)
    toggles.Speed = v
    if v then
        boosted = true
        fakeWalk()
    else
        boosted = false
    end
end)
-- Adicionando o AutoSlap ao menu
addToggleLine(scroll1, "AutoSlap", 4, function(v)
    toggles.AutoSlap = v
    if v then
        AzaroNotify("AutoSlap ativado!")
        startAutoSlap()
    else
        AzaroNotify("AutoSlap desativado!")
        stopAutoSlap()
    end
end)

-- Visual
local scroll2 = Instance.new("ScrollingFrame", tabFrames[2])
scroll2.Size = UDim2.new(1, 0, 1, 0)
scroll2.CanvasSize = UDim2.new(0,0,0,360)
scroll2.ScrollBarThickness = 6
scroll2.BackgroundTransparency = 1
createUICorner(scroll2, 6)
makeScrollBarRGB(scroll2)

addToggleLine(scroll2, "ESP", 1, function(v)
    for _, plr in pairs(Players:GetPlayers()) do
        local head = plr.Character and plr.Character:FindFirstChild("Head")
        if head then
            local existing = head:FindFirstChild("ESP")
            if v and not existing then
                local g = Instance.new("BillboardGui", head)
                g.Name = "ESP"
                g.Size = UDim2.new(0,60,0,18)
                g.AlwaysOnTop = true
                g.Adornee = head
                local t = Instance.new("TextLabel", g)
                t.Size = UDim2.new(1,0,1,0)
                t.BackgroundTransparency = 1
                t.TextColor3 = Color3.fromRGB(60, 60, 60)
                t.TextScaled = true
                t.Font = Enum.Font.GothamBold
                t.Text = plr.Name
            elseif not v and existing then
                existing:Destroy()
            end
        end
    end
end)

addToggleLine(scroll2, "BaseESP", 2, function(v)
    if not v then
        for _, gui in pairs(workspace:GetDescendants()) do
            if gui:IsA("BillboardGui") and gui.Name == "BaseTimerESP" then
                gui:Destroy()
            end
        end
    else
        for _, part in pairs(workspace:GetDescendants()) do
            if part:IsA("BasePart") and part:FindFirstChild("UnlockTime") then
                local g = Instance.new("BillboardGui", part)
                g.Name = "BaseTimerESP"
                g.Size = UDim2.new(0,80,0,16)
                g.Adornee = part
                g.AlwaysOnTop = true
                local l = Instance.new("TextLabel", g)
                l.Size = UDim2.new(1,0,1,0)
                l.BackgroundTransparency = 1
                l.TextScaled = true
                l.Font = Enum.Font.GothamBold
                l.TextColor3 = Color3.fromRGB(60,60,60)
                task.spawn(function()
                    while part and g and toggles.BaseESP do
                        local diff = part.UnlockTime.Value - os.time()
                        l.Text = diff>0 and ("⏳ "..diff.."s") or "🔓 Liberada!"
                        task.wait(1)
                    end
                    if g then g:Destroy() end
                end)
            end
        end
    end
end)

addToggleLine(scroll2, "ESP Brain do Watch", 3, function(v)
    activeLockTimeEsp = v
    if v then
        task.spawn(function()
            while activeLockTimeEsp do
                updatelock()
                task.wait(1)
            end
        end)
    else
        updatelock()
    end
end)

-- Servidor
local scroll3 = Instance.new("Frame", tabFrames[3])
scroll3.Size = UDim2.new(1, 0, 1, 0)
scroll3.BackgroundTransparency = 1

local procurarBtn = Instance.new("TextButton", scroll3)
procurarBtn.Size = UDim2.new(0, 220, 0, 38)
procurarBtn.Position = UDim2.new(0.5, -110, 0.18, 0)
procurarBtn.BackgroundColor3 = Color3.fromRGB(235,235,255)
procurarBtn.TextColor3 = Color3.fromRGB(85, 60, 220)
procurarBtn.TextSize = 18
procurarBtn.Font = Enum.Font.FredokaOne
procurarBtn.Text = "Procurar server bom"
createUICorner(procurarBtn, 9)

procurarBtn.MouseEnter:Connect(function()
    procurarBtn.BackgroundColor3 = Color3.fromRGB(220, 220, 255)
    procurarBtn.TextColor3 = Color3.fromRGB(100, 80, 230)
end)
procurarBtn.MouseLeave:Connect(function()
    procurarBtn.BackgroundColor3 = Color3.fromRGB(235,235,255)
    procurarBtn.TextColor3 = Color3.fromRGB(85, 60, 220)
end)

local rgbLine = Instance.new("Frame", scroll3)
rgbLine.Size = UDim2.new(0, 220, 0, 4)
rgbLine.Position = UDim2.new(0.5, -110, 0.18, 42)
rgbLine.BackgroundColor3 = Color3.fromRGB(150, 220, 255)
rgbLine.BorderSizePixel = 0
local rgbLineCorner = Instance.new("UICorner", rgbLine)
rgbLineCorner.CornerRadius = UDim.new(0, 2)

local rgbHue = 0
RunService.RenderStepped:Connect(function()
    rgbHue = (rgbHue + 1) % 360
    rgbLine.BackgroundColor3 = Color3.fromHSV(rgbHue/360, 0.7, 1)
end)

local function serverHop()
    local req = syn and syn.request or http_request or request
    if not req then
        print("Executor não suporta HTTP request")
        return
    end
    local ok, response = pcall(function()
        return req({
            Url = "https://games.roblox.com/v1/games/" .. PlaceId .. "/servers/Public?sortOrder=Asc&limit=100"
        })
    end)
    if not ok or not response then
        print("Erro ao buscar servidores.")
        return
    end
    local success, body = pcall(HttpService.JSONDecode, HttpService, response.Body)
    if not success or not body or not body.data then
        print("Falha ao decodificar resposta.")
        return
    end
    local servers = {}
    for _, v in ipairs(body.data) do
        if v.playing < v.maxPlayers and v.id ~= game.JobId then
            table.insert(servers, v.id)
        end
    end
    if #servers > 0 then
        local serverId = servers[math.random(#servers)]
        print("Server Hop para:", serverId)
        TeleportService:TeleportToPlaceInstance(PlaceId, serverId, LocalPlayer)
    else
        print("Nenhum servidor disponível.")
    end
end
procurarBtn.MouseButton1Click:Connect(function()
    serverHop()
    AzaroNotify()
end)

function fakeWalk()
    local char = LocalPlayer.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    local conn
    conn = RunService.Heartbeat:Connect(function()
        if not boosted then conn:Disconnect() return end
        local moveDir = humanoid.MoveDirection
        if moveDir.Magnitude > 0 then
            hrp.CFrame = hrp.CFrame + moveDir.Unit * speedMultiplier * 0.2
        end
    end)
end

UIS.InputBegan:Connect(function(i)
    if i.KeyCode == Enum.KeyCode.Space and toggles.HighJump then
        HRP.Velocity = Vector3.new(0, HIGH_JUMP, 0)
    end
end)

Hum:GetPropertyChangedSignal("FloorMaterial"):Connect(function()
    if Hum.FloorMaterial == Enum.Material.Air and HRP.Position.Y < -20 then
        HRP.Velocity = Vector3.new(0,100,0)
    end
end)

LocalPlayer.CharacterAdded:Connect(function()
    boosted = false
end)

for _, scroll in ipairs({scroll1, scroll2}) do
    local layout = Instance.new("UIListLayout", scroll)
    layout.Padding = UDim.new(0, 12)
    layout.SortOrder = Enum.SortOrder.LayoutOrder
end

topBar.Parent = frame
tabBar.Parent = frame
for _, f in ipairs(tabFrames) do f.Parent = frame end

scroll1.Parent = tabFrames[1]
scroll2.Parent = tabFrames[2]
scroll3.Parent = tabFrames[3]
procurarBtn.Parent = scroll3
rgbLine.Parent = scroll3

local creditFrame = tabFrames[4]
local creditLabel1 = Instance.new("TextLabel", creditFrame)
creditLabel1.Size = UDim2.new(1, -18, 0, 18)
creditLabel1.Position = UDim2.new(0, 9, 0, 38)
creditLabel1.BackgroundTransparency = 1
creditLabel1.Text = "CRIADORES: Faíska & Mt"
creditLabel1.TextColor3 = Color3.fromRGB(85, 60, 220)
creditLabel1.Font = Enum.Font.FredokaOne
creditLabel1.TextSize = 13
creditLabel1.TextYAlignment = Enum.TextYAlignment.Center
creditLabel1.TextXAlignment = Enum.TextXAlignment.Center
creditLabel1.TextWrapped = true

local creditLabel2 = Instance.new("TextLabel", creditFrame)
creditLabel2.Size = UDim2.new(1, -18, 0, 18)
creditLabel2.Position = UDim2.new(0, 9, 0, 56)
creditLabel2.BackgroundTransparency = 1
creditLabel2.Text = "DISCORD: Em breve"
creditLabel2.TextColor3 = Color3.fromRGB(110, 110, 110)
creditLabel2.Font = Enum.Font.Gotham
creditLabel2.TextSize = 12
creditLabel2.TextYAlignment = Enum.TextYAlignment.Center
creditLabel2.TextXAlignment = Enum.TextXAlignment.Center
creditLabel2.TextWrapped = true

-- Fim do script
