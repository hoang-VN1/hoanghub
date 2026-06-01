-- ╔══════════════════════════════════════╗
-- ║      BeeZ Fix Lag + Notes UI v3      ║
-- ╚══════════════════════════════════════╝
local Players       = game:GetService("Players")
local RunService    = game:GetService("RunService")
local Lighting      = game:GetService("Lighting")
local TweenService  = game:GetService("TweenService")
local UIS           = game:GetService("UserInputService")
local player        = Players.LocalPlayer
local playerGui     = player:WaitForChild("PlayerGui")

-- ═══════════════════════════════════════
--              LOADING UI
-- ═══════════════════════════════════════
local loadGui = Instance.new("ScreenGui", playerGui)
loadGui.Name = "BeeZLoading"
loadGui.ResetOnSpawn = false

local frame = Instance.new("Frame", loadGui)
frame.Size = UDim2.new(0, 320, 0, 130)
frame.Position = UDim2.new(0.5, -160, 0.5, -65)
frame.BackgroundColor3 = Color3.fromRGB(8, 8, 12)
frame.BackgroundTransparency = 0.05
frame.BorderSizePixel = 0
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 16)

local loadStroke = Instance.new("UIStroke", frame)
loadStroke.Color = Color3.fromRGB(255, 215, 0)
loadStroke.Thickness = 2

local loadGrad = Instance.new("UIGradient", loadStroke)
loadGrad.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0,   Color3.fromRGB(255, 215, 0)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255, 255, 255)),
    ColorSequenceKeypoint.new(1,   Color3.fromRGB(255, 215, 0)),
})

local titleLabel = Instance.new("TextLabel", frame)
titleLabel.Size = UDim2.new(1, 0, 0, 38)
titleLabel.Position = UDim2.new(0, 0, 0, 8)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "⚡ BeeZ Fix Lag"
titleLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 22

local statusLabel = Instance.new("TextLabel", frame)
statusLabel.Size = UDim2.new(1, -20, 0, 20)
statusLabel.Position = UDim2.new(0, 10, 0, 46)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "🔄 Đang khởi động..."
statusLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
statusLabel.Font = Enum.Font.Gotham
statusLabel.TextSize = 13
statusLabel.TextXAlignment = Enum.TextXAlignment.Left

local barBg = Instance.new("Frame", frame)
barBg.Size = UDim2.new(0.88, 0, 0, 10)
barBg.Position = UDim2.new(0.06, 0, 0, 74)
barBg.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
barBg.BorderSizePixel = 0
Instance.new("UICorner", barBg).CornerRadius = UDim.new(1, 0)

local barFill = Instance.new("Frame", barBg)
barFill.Size = UDim2.new(0, 0, 1, 0)
barFill.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
barFill.BorderSizePixel = 0
Instance.new("UICorner", barFill).CornerRadius = UDim.new(1, 0)

local barGlow = Instance.new("UIGradient", barFill)
barGlow.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 180, 0)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 240, 120)),
})

local pctLabel = Instance.new("TextLabel", frame)
pctLabel.Size = UDim2.new(1, 0, 0, 20)
pctLabel.Position = UDim2.new(0, 0, 0, 90)
pctLabel.BackgroundTransparency = 1
pctLabel.Text = "0%"
pctLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
pctLabel.Font = Enum.Font.GothamBold
pctLabel.TextSize = 13

task.spawn(function()
    local r = 0
    while loadGui.Parent do
        r = (r + 3) % 360
        loadGrad.Rotation = r
        task.wait(0.01)
    end
end)

-- ═══════════════════════════════════════
--               FIX LAG
-- ═══════════════════════════════════════
local function doFixLag()
    statusLabel.Text = "💡 Đang xử lý Lighting..."
    task.wait(0.1)

    Lighting.GlobalShadows  = false
    Lighting.FogEnd         = 100000
    Lighting.FogStart       = 100000
    Lighting.Brightness     = 2
    Lighting.Ambient        = Color3.fromRGB(178, 178, 178)
    Lighting.OutdoorAmbient = Color3.fromRGB(178, 178, 178)
    Lighting.ShadowSoftness = 0

    for _, v in ipairs(Lighting:GetChildren()) do
        if v:IsA("PostEffect") or v:IsA("Sky") or v:IsA("Atmosphere")
        or v:IsA("BloomEffect") or v:IsA("BlurEffect")
        or v:IsA("ColorCorrectionEffect") or v:IsA("SunRaysEffect")
        or v:IsA("DepthOfFieldEffect") then
            v:Destroy()
        end
    end

    statusLabel.Text = "🗑 Đang xóa Texture & Effect..."
    task.wait(0.1)

    local function ultraClean(obj)
        if obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam")
        or obj:IsA("Smoke") or obj:IsA("Fire") or obj:IsA("Sparkles")
        or obj:IsA("SelectionBox") or obj:IsA("SelectionSphere")
        or obj:IsA("SurfaceAppearance") then
            pcall(function() obj:Destroy() end); return
        end
        if obj:IsA("Texture") or obj:IsA("Decal") then
            pcall(function() obj:Destroy() end); return
        end
        if obj:IsA("BasePart") then
            pcall(function()
                obj.CastShadow  = false
                obj.Material    = Enum.Material.SmoothPlastic
                obj.Reflectance = 0
            end); return
        end
        if obj:IsA("SpecialMesh") then
            pcall(function() obj.TextureId = "" end)
        end
    end

    local descendants = workspace:GetDescendants()
    local total = #descendants
    for i, obj in ipairs(descendants) do
        ultraClean(obj)
        if i % 500 == 0 then
            local pct = math.floor(i / total * 100)
            barFill.Size   = UDim2.new(pct / 100, 0, 1, 0)
            pctLabel.Text  = pct .. "%"
            task.wait()
        end
    end

    barFill.Size  = UDim2.new(1, 0, 1, 0)
    pctLabel.Text = "100%"

    workspace.DescendantAdded:Connect(function(obj)
        task.defer(function() ultraClean(obj) end)
    end)

    statusLabel.Text = "✅ Hoàn tất! BeeZ đã sẵn sàng"
    task.wait(0.6)

    local tw = TweenService:Create(frame, TweenInfo.new(0.4), {
        BackgroundTransparency = 1, Size = UDim2.new(0, 320, 0, 0)
    })
    TweenService:Create(titleLabel,  TweenInfo.new(0.4), {TextTransparency=1}):Play()
    TweenService:Create(statusLabel, TweenInfo.new(0.4), {TextTransparency=1}):Play()
    TweenService:Create(pctLabel,    TweenInfo.new(0.4), {TextTransparency=1}):Play()
    TweenService:Create(loadStroke,  TweenInfo.new(0.4), {Transparency=1}):Play()
    tw:Play(); tw.Completed:Wait()
    loadGui:Destroy()

    -- ═══════════════════════════════════════
    --           MINI FPS COUNTER
    -- ═══════════════════════════════════════
    local fpsGui = Instance.new("ScreenGui", playerGui)
    fpsGui.Name = "BeeZ_FPS"
    fpsGui.ResetOnSpawn = false
    fpsGui.DisplayOrder = 10

    local fpsFrame = Instance.new("Frame", fpsGui)
    fpsFrame.Size = UDim2.new(0, 95, 0, 30)
    fpsFrame.Position = UDim2.new(0, 10, 0, 10)
    fpsFrame.BackgroundColor3 = Color3.fromRGB(8, 8, 12)
    fpsFrame.BackgroundTransparency = 0.2
    fpsFrame.Active = true
    fpsFrame.Draggable = true
    Instance.new("UICorner", fpsFrame).CornerRadius = UDim.new(0, 8)
    local fpsFrameStr = Instance.new("UIStroke", fpsFrame)
    fpsFrameStr.Color = Color3.fromRGB(255, 215, 0)
    fpsFrameStr.Thickness = 1.5

    local fpsLbl = Instance.new("TextLabel", fpsFrame)
    fpsLbl.Size = UDim2.new(1, 0, 1, 0)
    fpsLbl.BackgroundTransparency = 1
    fpsLbl.Font = Enum.Font.GothamBold
    fpsLbl.TextSize = 13
    fpsLbl.Text = "⚡ FPS: --"

    local fList = {}
    RunService.RenderStepped:Connect(function(dt)
        table.insert(fList, dt)
        if #fList > 30 then table.remove(fList, 1) end
    end)
    task.spawn(function()
        while fpsGui.Parent do
            local s = 0
            for _, v in ipairs(fList) do s += v end
            local fps = #fList > 0 and math.floor(1/(s/#fList)) or 0
            fpsLbl.TextColor3 = fps >= 50 and Color3.fromRGB(0,255,100)
                or fps >= 30 and Color3.fromRGB(255,165,0)
                or Color3.fromRGB(255,60,60)
            fpsLbl.Text = "⚡ " .. fps .. " FPS"
            task.wait(0.5)
        end
    end)

    -- ═══════════════════════════════════════
    --           SERVICES & CONFIG
    -- ═══════════════════════════════════════
    local Players2       = game:GetService("Players")
    local player2        = Players2.LocalPlayer
    local playerGui2     = player2:WaitForChild("PlayerGui")
    local RunService2    = game:GetService("RunService")
    local Stats          = game:GetService("Stats")
    local TeleportSvc    = game:GetService("TeleportService")
    local HttpSvc        = game:GetService("HttpService")

    local fileName = "note_save.txt"
    if isfile and isfile(fileName) then
        getgenv().SavedText = readfile(fileName)
    else
        getgenv().SavedText = ""
    end

    -- SIZE CONFIG
    local sizeConfig = {
        {key="Bình thường", w=430, h=260, titleSize=16, textSize=13, tabSize=10, icon="📦"},
        {key="Trung bình",  w=340, h=215, titleSize=14, textSize=12, tabSize=9,  icon="📋"},
        {key="Nhỏ",         w=260, h=180, titleSize=12, textSize=11, tabSize=8,  icon="🔹"},
    }
    local currentSizeIdx = 1

    -- THEME CONFIG
    local themeList = {
        {key="Vàng",       color=Color3.fromRGB(255,215,0)},
        {key="Xanh lá",    color=Color3.fromRGB(80,220,120)},
        {key="Xanh dương", color=Color3.fromRGB(60,160,255)},
        {key="Đỏ",         color=Color3.fromRGB(255,80,80)},
        {key="Tím",        color=Color3.fromRGB(180,80,255)},
        {key="Trắng",      color=Color3.fromRGB(240,240,240)},
        {key="Cam",        color=Color3.fromRGB(255,140,0)},
        {key="Hồng",       color=Color3.fromRGB(255,100,200)},
    }
    local currentThemeIdx = 1
    local function getC() return themeList[currentThemeIdx].color end

    -- ═══════════════════════════════════════
    --           MAIN GUI BUILD
    -- ═══════════════════════════════════════
    local gui = Instance.new("ScreenGui", playerGui2)
    gui.Name = "BeeZ_Notes"
    gui.ResetOnSpawn = false
    gui.DisplayOrder = 5

    local cfg0 = sizeConfig[currentSizeIdx]
    local main = Instance.new("Frame", gui)
    main.Size = UDim2.new(0, cfg0.w, 0, cfg0.h)
    main.Position = UDim2.new(0.5, 0, 0.4, 0)
    main.AnchorPoint = Vector2.new(0.5, 0.5)
    main.BackgroundColor3 = Color3.fromRGB(8, 8, 12)
    main.BackgroundTransparency = 0.08
    main.Active = true
    main.Draggable = true
    main.ClipsDescendants = true
    Instance.new("UICorner", main).CornerRadius = UDim.new(0, 16)

    -- Animated border
    local mainStroke = Instance.new("UIStroke", main)
    mainStroke.Thickness = 2
    mainStroke.Color = getC()
    local mainGrad = Instance.new("UIGradient", mainStroke)
    local function rebuildGrad()
        local c = getC()
        mainGrad.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0,   c),
            ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255,255,255)),
            ColorSequenceKeypoint.new(1,   c),
        })
    end
    rebuildGrad()
    task.spawn(function()
        local r = 0
        while main.Parent do
            r = (r + 1) % 360
            mainGrad.Rotation = r
            task.wait(0.02)
        end
    end)

    -- Title bar
    local titleBar = Instance.new("Frame", main)
    titleBar.Size = UDim2.new(1, 0, 0, 38)
    titleBar.BackgroundColor3 = getC()
    titleBar.BackgroundTransparency = 0.82
    titleBar.BorderSizePixel = 0
    Instance.new("UICorner", titleBar).CornerRadius = UDim.new(0, 16)

    -- Title bar bottom fill (to hide bottom radius)
    local titleBarFill = Instance.new("Frame", titleBar)
    titleBarFill.Size = UDim2.new(1, 0, 0.5, 0)
    titleBarFill.Position = UDim2.new(0, 0, 0.5, 0)
    titleBarFill.BackgroundColor3 = Color3.fromRGB(8, 8, 12)
    titleBarFill.BackgroundTransparency = 0.08
    titleBarFill.BorderSizePixel = 0

    local titleText = Instance.new("TextLabel", titleBar)
    titleText.Size = UDim2.new(1, -16, 1, 0)
    titleText.Position = UDim2.new(0, 14, 0, 0)
    titleText.BackgroundTransparency = 1
    titleText.Text = "🐝 BeeZ Notes"
    titleText.TextColor3 = getC()
    titleText.Font = Enum.Font.GothamBold
    titleText.TextSize = cfg0.titleSize
    titleText.TextXAlignment = Enum.TextXAlignment.Left

    -- Tab bar
    local tabBar = Instance.new("Frame", main)
    tabBar.Size = UDim2.new(1, 0, 0, 28)
    tabBar.Position = UDim2.new(0, 0, 0, 38)
    tabBar.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
    tabBar.BorderSizePixel = 0

    -- Tab indicator (sliding underline)
    local tabIndicator = Instance.new("Frame", tabBar)
    tabIndicator.Size = UDim2.new(1/5, -4, 0, 2)
    tabIndicator.Position = UDim2.new(0, 2, 1, -2)
    tabIndicator.BackgroundColor3 = getC()
    tabIndicator.BorderSizePixel = 0
    Instance.new("UICorner", tabIndicator).CornerRadius = UDim.new(1, 0)

    -- Content area
    local contentArea = Instance.new("Frame", main)
    contentArea.Position = UDim2.new(0, 6, 0, 66)
    contentArea.BackgroundTransparency = 1
    contentArea.ClipsDescendants = true

    -- ─── Forward declarations ───
    local refreshSizeBtns
    local sizeBtns = {}
    local colorRefreshList = {}
    local pages = {}
    local tabBtns = {}

    -- ─── applyTheme ───
    local function applyTheme(idx)
        currentThemeIdx = idx
        local c = getC()
        mainStroke.Color = c
        rebuildGrad()
        titleBar.BackgroundColor3 = c
        titleText.TextColor3 = c
        tabIndicator.BackgroundColor3 = c
        fpsFrameStr.Color = c
        for _, info in ipairs(colorRefreshList) do
            pcall(function() info[1][info[2]] = c end)
        end
        if refreshSizeBtns then refreshSizeBtns() end
    end

    -- ─── applySize ───
    local function applySize(idx)
        currentSizeIdx = idx
        local cfg = sizeConfig[idx]
        TweenService:Create(main, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {
            Size = UDim2.new(0, cfg.w, 0, cfg.h)
        }):Play()
        titleText.TextSize = cfg.titleSize
        contentArea.Size = UDim2.new(1, -12, 1, -72)
        -- Update all scalable children
        for _, child in ipairs(main:GetDescendants()) do
            if child:GetAttribute("scalable") and
               (child:IsA("TextLabel") or child:IsA("TextButton") or child:IsA("TextBox")) then
                child.TextSize = cfg.textSize
            end
        end
        -- Tab text size
        for _, b in ipairs(tabBtns) do
            b.TextSize = cfg.tabSize
        end
    end

    -- ─── Helpers ───
    local function mkLabel(parent, text, posY, bold, color)
        local l = Instance.new("TextLabel", parent)
        l.Size = UDim2.new(1, 0, 0, 20)
        l.Position = UDim2.new(0, 0, 0, posY)
        l.BackgroundTransparency = 1
        l.Text = text
        l.TextColor3 = color or Color3.fromRGB(210, 210, 210)
        l.Font = bold and Enum.Font.GothamBold or Enum.Font.Gotham
        l.TextSize = sizeConfig[currentSizeIdx].textSize
        l.TextXAlignment = Enum.TextXAlignment.Left
        l:SetAttribute("scalable", true)
        return l
    end

    local function mkBtn(parent, text, posX, posY, w, h, bg, tc)
        local b = Instance.new("TextButton", parent)
        b.Size = UDim2.new(0, w, 0, h)
        b.Position = UDim2.new(0, posX, 0, posY)
        b.BackgroundColor3 = bg or Color3.fromRGB(25, 25, 32)
        b.BackgroundTransparency = 0.2
        b.TextColor3 = tc or Color3.fromRGB(240, 240, 240)
        b.Font = Enum.Font.GothamBold
        b.TextSize = sizeConfig[currentSizeIdx].textSize
        b.BorderSizePixel = 0
        b.Text = text
        b:SetAttribute("scalable", true)
        Instance.new("UICorner", b).CornerRadius = UDim.new(0, 8)
        return b
    end

    local function mkSep(parent, posY)
        local s = Instance.new("Frame", parent)
        s.Size = UDim2.new(1, 0, 0, 1)
        s.Position = UDim2.new(0, 0, 0, posY)
        s.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
        s.BorderSizePixel = 0
        return s
    end

    local function mkCard(parent, posY, h)
        local bg = Instance.new("Frame", parent)
        bg.Size = UDim2.new(1, 0, 0, h or 38)
        bg.Position = UDim2.new(0, 0, 0, posY)
        bg.BackgroundColor3 = Color3.fromRGB(18, 18, 24)
        bg.BackgroundTransparency = 0.3
        bg.BorderSizePixel = 0
        Instance.new("UICorner", bg).CornerRadius = UDim.new(0, 10)
        local str = Instance.new("UIStroke", bg)
        str.Color = getC()
        str.Thickness = 1
        str.Transparency = 0.6
        table.insert(colorRefreshList, {str, "Color"})
        return bg, str
    end

    -- ─── Build pages ───
    contentArea.Size = UDim2.new(1, -12, 1, -72)
    for i = 1, 5 do
        local sf = Instance.new("ScrollingFrame", contentArea)
        sf.Size = UDim2.new(1, 0, 1, 0)
        sf.BackgroundTransparency = 1
        sf.BorderSizePixel = 0
        sf.ScrollBarThickness = 3
        sf.ScrollBarImageColor3 = getC()
        sf.CanvasSize = UDim2.new(0, 0, 0, 0)
        sf.Visible = false
        pages[i] = sf
        table.insert(colorRefreshList, {sf, "ScrollBarImageColor3"})
    end

    -- ════════════════════════════════
    --   PAGE 1 · 📝 NOTE / ĐƠN HÀNG
    -- ════════════════════════════════
    local p1 = pages[1]

    -- Status cycle
    local statusStates = {
        {text="🔴  Chưa nhận",        color=Color3.fromRGB(200,50,50)},
        {text="🟡  Đang cày",          color=Color3.fromRGB(220,160,0)},
        {text="🟢  Hoàn thành",        color=Color3.fromRGB(50,200,80)},
        {text="💰  Chờ thanh toán",    color=Color3.fromRGB(60,160,255)},
    }
    local curStatus = 1
    local statusBtn = Instance.new("TextButton", p1)
    statusBtn.Size = UDim2.new(1, 0, 0, 30)
    statusBtn.Position = UDim2.new(0, 0, 0, 0)
    statusBtn.BackgroundColor3 = statusStates[1].color
    statusBtn.BackgroundTransparency = 0.1
    statusBtn.TextColor3 = Color3.fromRGB(255,255,255)
    statusBtn.Font = Enum.Font.GothamBold
    statusBtn.TextSize = sizeConfig[currentSizeIdx].textSize
    statusBtn.Text = statusStates[1].text
    statusBtn.BorderSizePixel = 0
    statusBtn:SetAttribute("scalable", true)
    Instance.new("UICorner", statusBtn).CornerRadius = UDim.new(0, 10)
    statusBtn.MouseButton1Click:Connect(function()
        curStatus = (curStatus % #statusStates) + 1
        statusBtn.BackgroundColor3 = statusStates[curStatus].color
        statusBtn.Text = statusStates[curStatus].text
    end)

    -- Player info card
    local infoCard, _ = mkCard(p1, 36, 30)
    local nameLbl = Instance.new("TextLabel", infoCard)
    nameLbl.Size = UDim2.new(0.55, 0, 1, 0)
    nameLbl.Position = UDim2.new(0, 8, 0, 0)
    nameLbl.BackgroundTransparency = 1
    nameLbl.Text = "👤 " .. player2.Name
    nameLbl.TextColor3 = Color3.fromRGB(220, 220, 220)
    nameLbl.Font = Enum.Font.GothamBold
    nameLbl.TextSize = sizeConfig[currentSizeIdx].textSize
    nameLbl.TextXAlignment = Enum.TextXAlignment.Left
    nameLbl:SetAttribute("scalable", true)

    local serverLbl = Instance.new("TextLabel", infoCard)
    serverLbl.Size = UDim2.new(0.44, 0, 1, 0)
    serverLbl.Position = UDim2.new(0.56, 0, 0, 0)
    serverLbl.BackgroundTransparency = 1
    serverLbl.TextColor3 = Color3.fromRGB(130, 130, 130)
    serverLbl.Font = Enum.Font.Gotham
    serverLbl.TextSize = 10
    serverLbl.TextXAlignment = Enum.TextXAlignment.Right
    serverLbl.Text = "🔗 ..."
    serverLbl:SetAttribute("scalable", true)

    -- Note section
    local noteLbl = mkLabel(p1, "📝  Ghi chú đơn hàng", 74, true, getC())
    table.insert(colorRefreshList, {noteLbl, "TextColor3"})

    local noteCard, noteStr = mkCard(p1, 96, 76)
    noteStr.Transparency = 0.3
    local noteBox = Instance.new("TextBox", noteCard)
    noteBox.Size = UDim2.new(1, -8, 1, -6)
    noteBox.Position = UDim2.new(0, 4, 0, 3)
    noteBox.BackgroundTransparency = 1
    noteBox.TextColor3 = Color3.fromRGB(230, 230, 230)
    noteBox.PlaceholderText = "Nhập ghi chú đơn hàng cày thuê..."
    noteBox.PlaceholderColor3 = Color3.fromRGB(80, 80, 90)
    noteBox.Text = getgenv().SavedText
    noteBox.Font = Enum.Font.Gotham
    noteBox.TextSize = sizeConfig[currentSizeIdx].textSize
    noteBox.ClearTextOnFocus = false
    noteBox.TextWrapped = true
    noteBox.MultiLine = true
    noteBox.TextXAlignment = Enum.TextXAlignment.Left
    noteBox.TextYAlignment = Enum.TextYAlignment.Top
    noteBox.BorderSizePixel = 0
    noteBox:SetAttribute("scalable", true)
    noteBox.FocusLost:Connect(function()
        getgenv().SavedText = noteBox.Text
        if writefile then writefile(fileName, noteBox.Text) end
    end)

    -- Action buttons
    local clearBtn = mkBtn(p1, "🗑  Xóa ghi chú", 0, 180, 128, 28,
        Color3.fromRGB(160, 35, 35), Color3.fromRGB(255,255,255))
    clearBtn.BackgroundTransparency = 0.1
    clearBtn.MouseButton1Click:Connect(function()
        noteBox.Text = ""
        getgenv().SavedText = ""
        if writefile then writefile(fileName, "") end
    end)

    local copyNoteBtn = mkBtn(p1, "📋  Copy ghi chú", 134, 180, 128, 28,
        Color3.fromRGB(25,25,32), Color3.fromRGB(255,255,255))
    local copyNoteStr = Instance.new("UIStroke", copyNoteBtn)
    copyNoteStr.Color = getC(); copyNoteStr.Thickness = 1
    table.insert(colorRefreshList, {copyNoteStr, "Color"})
    copyNoteBtn.MouseButton1Click:Connect(function()
        if setclipboard then
            setclipboard(noteBox.Text)
            copyNoteBtn.Text = "✅  Đã Copy!"
            task.wait(1.5)
            copyNoteBtn.Text = "📋  Copy ghi chú"
        end
    end)

    local autoSaveTag = Instance.new("TextLabel", p1)
    autoSaveTag.Size = UDim2.new(1, 0, 0, 16)
    autoSaveTag.Position = UDim2.new(0, 0, 0, 214)
    autoSaveTag.BackgroundTransparency = 1
    autoSaveTag.Text = "💾  Auto Save: BẬT"
    autoSaveTag.TextColor3 = Color3.fromRGB(80, 200, 100)
    autoSaveTag.Font = Enum.Font.Gotham
    autoSaveTag.TextSize = 10
    autoSaveTag.TextXAlignment = Enum.TextXAlignment.Left

    p1.CanvasSize = UDim2.new(0, 0, 0, 235)

    -- ════════════════════════════════
    --   PAGE 2 · 📊 STATUS / THỐNG KÊ
    -- ════════════════════════════════
    local p2 = pages[2]

    local function mkStatCard(parent, icon, label, posY)
        local bg, _ = mkCard(parent, posY, 40)
        local iconLbl = Instance.new("TextLabel", bg)
        iconLbl.Size = UDim2.new(0, 32, 1, 0)
        iconLbl.BackgroundTransparency = 1
        iconLbl.Text = icon
        iconLbl.Font = Enum.Font.GothamBold
        iconLbl.TextSize = 18
        iconLbl.TextColor3 = getC()
        table.insert(colorRefreshList, {iconLbl, "TextColor3"})

        local lLbl = Instance.new("TextLabel", bg)
        lLbl.Size = UDim2.new(0.42, 0, 1, 0)
        lLbl.Position = UDim2.new(0, 34, 0, 0)
        lLbl.BackgroundTransparency = 1
        lLbl.Text = label
        lLbl.TextColor3 = Color3.fromRGB(150, 150, 160)
        lLbl.Font = Enum.Font.GothamBold
        lLbl.TextSize = 11
        lLbl.TextXAlignment = Enum.TextXAlignment.Left
        lLbl:SetAttribute("scalable", true)

        local vLbl = Instance.new("TextLabel", bg)
        vLbl.Size = UDim2.new(0.48, 0, 1, 0)
        vLbl.Position = UDim2.new(0.52, 0, 0, 0)
        vLbl.BackgroundTransparency = 1
        vLbl.Text = "..."
        vLbl.TextColor3 = getC()
        vLbl.Font = Enum.Font.GothamBold
        vLbl.TextSize = 11
        vLbl.TextXAlignment = Enum.TextXAlignment.Right
        vLbl:SetAttribute("scalable", true)
        table.insert(colorRefreshList, {vLbl, "TextColor3"})
        return vLbl
    end

    local placeVal  = mkStatCard(p2, "🌍", "Place ID",   0)
    local playerVal = mkStatCard(p2, "👥", "Players",    46)
    local jobVal    = mkStatCard(p2, "🔗", "Job ID",     92)
    local fpsVal    = mkStatCard(p2, "⚡", "FPS",       138)
    local pingVal   = mkStatCard(p2, "📶", "Ping",      184)

    p2.CanvasSize = UDim2.new(0, 0, 0, 230)

    -- Live update
    local fpsTrack, pingTrack = 60, 0
    local fpsFrames2 = {}
    RunService2.RenderStepped:Connect(function(dt)
        table.insert(fpsFrames2, dt)
        if #fpsFrames2 > 30 then table.remove(fpsFrames2, 1) end
    end)
    task.spawn(function()
        while gui.Parent do
            local s = 0
            for _, v in ipairs(fpsFrames2) do s += v end
            if #fpsFrames2 > 0 then fpsTrack = math.floor(1/(s/#fpsFrames2)) end
            local ok, raw = pcall(function()
                return Stats.Network.ServerStatsItem["Data Ping"]:GetValue()
            end)
            if ok then pingTrack = math.floor(raw) end
            placeVal.Text  = tostring(game.PlaceId)
            playerVal.Text = #Players2:GetPlayers() .. " người"
            jobVal.Text    = string.sub(game.JobId,1,14) .. "..."
            fpsVal.Text    = fpsTrack .. " fps"
            pingVal.Text   = pingTrack .. " ms"
            nameLbl.Text   = "👤  " .. player2.Name
            serverLbl.Text = "🔗  " .. string.sub(game.JobId,1,8).."..."
            task.wait(0.8)
        end
    end)

    -- ════════════════════════════════
    --   PAGE 3 · ⚙️ SETTING
    -- ════════════════════════════════
    local p3 = pages[3]
    mkLabel(p3, "🛠️  Server Tools", 0, true, getC())

    -- Copy Job ID
    local cjCard, _ = mkCard(p3, 22, 34)
    local cjIcon = Instance.new("TextLabel", cjCard)
    cjIcon.Size = UDim2.new(0,30,1,0); cjIcon.BackgroundTransparency=1
    cjIcon.Text="📋"; cjIcon.Font=Enum.Font.GothamBold; cjIcon.TextSize=18
    cjIcon.TextColor3=getC(); table.insert(colorRefreshList,{cjIcon,"TextColor3"})
    local cjLabel = Instance.new("TextLabel", cjCard)
    cjLabel.Size=UDim2.new(0.55,0,1,0); cjLabel.Position=UDim2.new(0,32,0,0)
    cjLabel.BackgroundTransparency=1; cjLabel.Text="Copy Job ID"
    cjLabel.TextColor3=Color3.fromRGB(210,210,210); cjLabel.Font=Enum.Font.GothamBold
    cjLabel.TextSize=11; cjLabel.TextXAlignment=Enum.TextXAlignment.Left
    cjLabel:SetAttribute("scalable",true)
    local cjBtn2 = mkBtn(cjCard, "📋 Copy", -76, 4, 70, 26, Color3.fromRGB(25,25,32))
    cjBtn2.Position = UDim2.new(1,-74,0,4)
    local cjStr2 = Instance.new("UIStroke",cjBtn2); cjStr2.Color=getC(); cjStr2.Thickness=1
    table.insert(colorRefreshList,{cjStr2,"Color"})
    cjBtn2.MouseButton1Click:Connect(function()
        if setclipboard then
            setclipboard(game.JobId)
            cjBtn2.Text="✅ OK!"; task.wait(1.5); cjBtn2.Text="📋 Copy"
        end
    end)

    -- Rejoin
    local rejCard, _ = mkCard(p3, 62, 34)
    local rejIcon = Instance.new("TextLabel",rejCard)
    rejIcon.Size=UDim2.new(0,30,1,0); rejIcon.BackgroundTransparency=1
    rejIcon.Text="🔄"; rejIcon.Font=Enum.Font.GothamBold; rejIcon.TextSize=18
    rejIcon.TextColor3=getC(); table.insert(colorRefreshList,{rejIcon,"TextColor3"})
    local rejLabel = Instance.new("TextLabel",rejCard)
    rejLabel.Size=UDim2.new(0.55,0,1,0); rejLabel.Position=UDim2.new(0,32,0,0)
    rejLabel.BackgroundTransparency=1; rejLabel.Text="Rejoin Server"
    rejLabel.TextColor3=Color3.fromRGB(210,210,210); rejLabel.Font=Enum.Font.GothamBold
    rejLabel.TextSize=11; rejLabel.TextXAlignment=Enum.TextXAlignment.Left
    rejLabel:SetAttribute("scalable",true)
    local rejBtn2 = mkBtn(rejCard, "🔄 Join", 0, 4, 70, 26, Color3.fromRGB(25,25,32))
    rejBtn2.Position = UDim2.new(1,-74,0,4)
    local rejStr2 = Instance.new("UIStroke",rejBtn2); rejStr2.Color=getC(); rejStr2.Thickness=1
    table.insert(colorRefreshList,{rejStr2,"Color"})
    rejBtn2.MouseButton1Click:Connect(function()
        pcall(function() TeleportSvc:Teleport(game.PlaceId, player2) end)
    end)

    -- Hop Server
    local hopCard, _ = mkCard(p3, 102, 34)
    local hopIcon = Instance.new("TextLabel",hopCard)
    hopIcon.Size=UDim2.new(0,30,1,0); hopIcon.BackgroundTransparency=1
    hopIcon.Text="⚡"; hopIcon.Font=Enum.Font.GothamBold; hopIcon.TextSize=18
    hopIcon.TextColor3=getC(); table.insert(colorRefreshList,{hopIcon,"TextColor3"})
    local hopLabel = Instance.new("TextLabel",hopCard)
    hopLabel.Size=UDim2.new(0.55,0,1,0); hopLabel.Position=UDim2.new(0,32,0,0)
    hopLabel.BackgroundTransparency=1; hopLabel.Text="Hop Server"
    hopLabel.TextColor3=Color3.fromRGB(210,210,210); hopLabel.Font=Enum.Font.GothamBold
    hopLabel.TextSize=11; hopLabel.TextXAlignment=Enum.TextXAlignment.Left
    hopLabel:SetAttribute("scalable",true)
    local hopBtn2 = mkBtn(hopCard, "⚡ Hop", 0, 4, 70, 26, Color3.fromRGB(25,25,32))
    hopBtn2.Position = UDim2.new(1,-74,0,4)
    local hopStr2 = Instance.new("UIStroke",hopBtn2); hopStr2.Color=getC(); hopStr2.Thickness=1
    table.insert(colorRefreshList,{hopStr2,"Color"})
    hopBtn2.MouseButton1Click:Connect(function()
        hopBtn2.Text = "🔍 ..."
        pcall(function()
            local data = game:HttpGet(
                "https://games.roblox.com/v1/games/"..game.PlaceId..
                "/servers/Public?sortOrder=Asc&limit=100"
            )
            local decoded = HttpSvc:JSONDecode(data)
            for _, sv in ipairs(decoded.data) do
                if sv.id ~= game.JobId and sv.playing < sv.maxPlayers then
                    TeleportSvc:TeleportToPlaceInstance(game.PlaceId, sv.id, player2)
                    return
                end
            end
        end)
        task.wait(2); hopBtn2.Text="⚡ Hop"
    end)

    mkSep(p3, 143)
    mkLabel(p3, "🛡️  Anti AFK", 150, true, getC())

    -- Anti AFK card
    local afkCard, _ = mkCard(p3, 172, 34)
    local afkIcon = Instance.new("TextLabel",afkCard)
    afkIcon.Size=UDim2.new(0,30,1,0); afkIcon.BackgroundTransparency=1
    afkIcon.Text="🛡️"; afkIcon.Font=Enum.Font.GothamBold; afkIcon.TextSize=18
    afkIcon.TextColor3=getC(); table.insert(colorRefreshList,{afkIcon,"TextColor3"})
    local afkLabel = Instance.new("TextLabel",afkCard)
    afkLabel.Size=UDim2.new(0.55,0,1,0); afkLabel.Position=UDim2.new(0,32,0,0)
    afkLabel.BackgroundTransparency=1; afkLabel.Text="Anti AFK"
    afkLabel.TextColor3=Color3.fromRGB(210,210,210); afkLabel.Font=Enum.Font.GothamBold
    afkLabel.TextSize=11; afkLabel.TextXAlignment=Enum.TextXAlignment.Left
    afkLabel:SetAttribute("scalable",true)
    local afkToggleBtn = mkBtn(afkCard, "❌ TẮT", 0, 4, 70, 26, Color3.fromRGB(160,35,35))
    afkToggleBtn.Position = UDim2.new(1,-74,0,4)
    afkToggleBtn.BackgroundTransparency = 0.1
    local afkOn = false
    afkToggleBtn.MouseButton1Click:Connect(function()
        afkOn = not afkOn
        if afkOn then
            afkToggleBtn.Text = "✅ BẬT"
            afkToggleBtn.BackgroundColor3 = Color3.fromRGB(35,140,35)
            game:GetService("Players").LocalPlayer.Idled:Connect(function()
                if afkOn then
                    local VU = game:GetService("VirtualUser")
                    VU:Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
                    task.wait(1)
                    VU:Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
                end
            end)
        else
            afkToggleBtn.Text = "❌ TẮT"
            afkToggleBtn.BackgroundColor3 = Color3.fromRGB(160,35,35)
        end
    end)

    p3.CanvasSize = UDim2.new(0,0,0,212)

    -- ════════════════════════════════
    --   PAGE 4 · 🔗 JOB ID
    -- ════════════════════════════════
    local p4 = pages[4]
    mkLabel(p4, "🔗  Join Server bằng Job ID", 0, true, getC())

    local jobInputCard, jobInputStr = mkCard(p4, 22, 34)
    jobInputStr.Transparency = 0.3
    local jobBox = Instance.new("TextBox", jobInputCard)
    jobBox.Size = UDim2.new(1,-10,1,-6)
    jobBox.Position = UDim2.new(0,5,0,3)
    jobBox.BackgroundTransparency = 1
    jobBox.TextColor3 = Color3.fromRGB(230,230,230)
    jobBox.PlaceholderText = "  🔍 Dán Job ID vào đây..."
    jobBox.PlaceholderColor3 = Color3.fromRGB(80,80,90)
    jobBox.Text = ""
    jobBox.Font = Enum.Font.Gotham
    jobBox.TextSize = 11
    jobBox.ClearTextOnFocus = false
    jobBox.BorderSizePixel = 0
    jobBox:SetAttribute("scalable",true)

    -- Action buttons row
    local joinBtn4 = mkBtn(p4, "🚀  Join Server", 0, 62, 140, 30,
        Color3.fromRGB(255,215,0), Color3.fromRGB(0,0,0))
    joinBtn4.BackgroundTransparency = 0.05
    joinBtn4.MouseButton1Click:Connect(function()
        local id = jobBox.Text
        if id ~= "" and id ~= game.JobId then
            joinBtn4.Text = "⏳ Đang chuyển..."
            pcall(function() TeleportSvc:TeleportToPlaceInstance(game.PlaceId, id, player2) end)
            task.wait(3); joinBtn4.Text = "🚀  Join Server"
        end
    end)

    local copyJob4 = mkBtn(p4, "📋  Copy", 146, 62, 90, 30)
    local cj4Str = Instance.new("UIStroke",copyJob4)
    cj4Str.Color=getC(); cj4Str.Thickness=1
    table.insert(colorRefreshList,{cj4Str,"Color"})
    copyJob4.MouseButton1Click:Connect(function()
        if setclipboard then
            setclipboard(game.JobId)
            copyJob4.Text="✅ OK!"; task.wait(1.5); copyJob4.Text="📋  Copy"
        end
    end)

    -- Current Job ID (read-only label)
    mkLabel(p4, "🔑  Job ID hiện tại:", 100, true, Color3.fromRGB(130,130,140))

    local jobDisplayCard, _ = mkCard(p4, 120, 30)
    local jobDisplayLbl = Instance.new("TextLabel", jobDisplayCard)
    jobDisplayLbl.Size = UDim2.new(1,-10,1,0)
    jobDisplayLbl.Position = UDim2.new(0,8,0,0)
    jobDisplayLbl.BackgroundTransparency = 1
    jobDisplayLbl.TextColor3 = getC()
    jobDisplayLbl.Text = game.JobId
    jobDisplayLbl.Font = Enum.Font.Gotham
    jobDisplayLbl.TextSize = 10
    jobDisplayLbl.TextXAlignment = Enum.TextXAlignment.Left
    jobDisplayLbl.TextTruncate = Enum.TextTruncate.AtEnd
    table.insert(colorRefreshList,{jobDisplayLbl,"TextColor3"})

    p4.CanvasSize = UDim2.new(0,0,0,158)

    -- ════════════════════════════════
    --   PAGE 5 · 🎨 UI SETTINGS
    -- ════════════════════════════════
    local p5 = pages[5]

    -- Size section
    local szLbl = mkLabel(p5, "📐  Kích cỡ UI", 0, true, getC())
    table.insert(colorRefreshList,{szLbl,"TextColor3"})

    for i, cfg in ipairs(sizeConfig) do
        local xOff = (i-1) * 86
        local sb = Instance.new("TextButton", p5)
        sb.Size = UDim2.new(0, 80, 0, 30)
        sb.Position = UDim2.new(0, xOff, 0, 22)
        sb.Text = cfg.icon .. "  " .. cfg.key
        sb.Font = Enum.Font.GothamBold
        sb.TextSize = 10
        sb.BorderSizePixel = 0
        Instance.new("UICorner", sb).CornerRadius = UDim.new(0, 8)
        local sbStr = Instance.new("UIStroke", sb)
        sbStr.Thickness = 2
        sizeBtns[i] = {btn=sb, str=sbStr}
    end

    -- Define refreshSizeBtns
    refreshSizeBtns = function()
        for i, info in ipairs(sizeBtns) do
            local active = (i == currentSizeIdx)
            local c = getC()
            if active then
                info.btn.BackgroundColor3 = c
                info.btn.BackgroundTransparency = 0.05
                info.btn.TextColor3 = Color3.fromRGB(0,0,0)
                info.str.Color = c
                info.str.Transparency = 0
            else
                info.btn.BackgroundColor3 = Color3.fromRGB(40,40,50)
                info.btn.BackgroundTransparency = 0.1
                info.btn.TextColor3 = Color3.fromRGB(120,120,130)
                info.str.Color = Color3.fromRGB(60,60,70)
                info.str.Transparency = 0
            end
        end
    end
    refreshSizeBtns()

    for i, info in ipairs(sizeBtns) do
        info.btn.MouseButton1Click:Connect(function()
            applySize(i); refreshSizeBtns()
        end)
    end

    mkSep(p5, 58)

    -- Color section
    local clrLbl = mkLabel(p5, "🎨  Màu sắc", 65, true, getC())
    table.insert(colorRefreshList,{clrLbl,"TextColor3"})

    local colorBtnMap = {}
    for i, theme in ipairs(themeList) do
        local row = math.ceil(i/4)
        local col = (i-1) % 4
        local cb = Instance.new("TextButton", p5)
        cb.Size = UDim2.new(0, 58, 0, 28)
        cb.Position = UDim2.new(0, col*62, 0, 84 + (row-1)*34)
        cb.BackgroundColor3 = theme.color
        cb.BackgroundTransparency = 0.1
        cb.Text = theme.key
        cb.Font = Enum.Font.GothamBold
        cb.TextSize = 9
        cb.BorderSizePixel = 0
        cb.TextColor3 = Color3.fromRGB(0,0,0)
        Instance.new("UICorner", cb).CornerRadius = UDim.new(0, 7)
        local cbStr = Instance.new("UIStroke", cb)
        cbStr.Thickness = 2
        cbStr.Color = Color3.fromRGB(0,0,0)
        cbStr.Transparency = 0.7
        colorBtnMap[i] = {btn=cb, str=cbStr}
    end

    local function refreshColorBtns()
        for i, info in ipairs(colorBtnMap) do
            if i == currentThemeIdx then
                info.str.Color = Color3.fromRGB(255,255,255)
                info.str.Transparency = 0
                info.btn.BackgroundTransparency = 0.0
            else
                info.str.Color = Color3.fromRGB(0,0,0)
                info.str.Transparency = 0.7
                info.btn.BackgroundTransparency = 0.25
            end
        end
        refreshSizeBtns()
    end
    refreshColorBtns()

    for i, info in ipairs(colorBtnMap) do
        info.btn.MouseButton1Click:Connect(function()
            applyTheme(i); refreshColorBtns()
        end)
    end

    -- Current color label
    local curClrLbl = mkLabel(p5, "", 158, false)
    task.spawn(function()
        while gui.Parent do
            curClrLbl.Text = "✨  Màu hiện tại: " .. themeList[currentThemeIdx].key
            curClrLbl.TextColor3 = getC()
            task.wait(0.3)
        end
    end)

    p5.CanvasSize = UDim2.new(0, 0, 0, 178)

    -- ════════════════════════════════
    --      TABS WITH ICONS
    -- ════════════════════════════════
    local TABS = {
        {label="📝 Note",   idx=1},
        {label="📊 Status", idx=2},
        {label="⚙️ Setting",   idx=3},
        {label="🔗 Job",    idx=4},
        {label="🎨 UI",     idx=5},
    }

    for i, tab in ipairs(TABS) do
        local b = Instance.new("TextButton", tabBar)
        b.Size = UDim2.new(1/#TABS, 0, 1, 0)
        b.Position = UDim2.new((i-1)/#TABS, 0, 0, 0)
        b.Text = tab.label
        b.BackgroundTransparency = 1
        b.Font = Enum.Font.GothamBold
        b.TextSize = cfg0.tabSize
        b.TextColor3 = Color3.fromRGB(120, 120, 130)
        tabBtns[i] = b

        b.MouseButton1Click:Connect(function()
            for j = 1, #TABS do
                pages[j].Visible = false
                tabBtns[j].TextColor3 = Color3.fromRGB(120,120,130)
            end
            pages[i].Visible = true
            b.TextColor3 = getC()
            -- Slide indicator
            TweenService:Create(tabIndicator, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
                Position = UDim2.new((i-1)/#TABS, 2, 1, -2)
            }):Play()
            tabIndicator.BackgroundColor3 = getC()
        end)
    end

    -- Keep active tab color synced to theme
    task.spawn(function()
        while gui.Parent do
            for i, pg in ipairs(pages) do
                if pg.Visible then
                    tabBtns[i].TextColor3 = getC()
                    tabIndicator.BackgroundColor3 = getC()
                end
            end
            task.wait(0.3)
        end
    end)

    pages[1].Visible = true
    tabBtns[1].TextColor3 = getC()

    -- ════════════════════════════════
    --   TOGGLE BUTTON (DRAGGABLE)
    -- ════════════════════════════════
    local toggle = Instance.new("TextButton", gui)
    toggle.Size = UDim2.new(0, 62, 0, 62)
    toggle.Position = UDim2.new(0, 20, 1, -82)
    toggle.BackgroundColor3 = Color3.fromRGB(8, 8, 12)
    toggle.BackgroundTransparency = 0.1
    toggle.Text = ""
    toggle.BorderSizePixel = 0
    toggle.Active = true
    toggle.ClipsDescendants = true
    Instance.new("UICorner", toggle).CornerRadius = UDim.new(1, 0)

    local tgStr = Instance.new("UIStroke", toggle)
    tgStr.Thickness = 2.5
    tgStr.Color = getC()
    table.insert(colorRefreshList, {tgStr, "Color"})

    -- Gradient border on toggle
    local tgGrad = Instance.new("UIGradient", tgStr)
    tgGrad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0,   getC()),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255,255,255)),
        ColorSequenceKeypoint.new(1,   getC()),
    })
    task.spawn(function()
        local r = 0
        while toggle.Parent do
            r = (r + 2) % 360
            tgGrad.Rotation = r
            local c = getC()
            tgGrad.Color = ColorSequence.new({
                ColorSequenceKeypoint.new(0, c),
                ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255,255,255)),
                ColorSequenceKeypoint.new(1, c),
            })
            task.wait(0.03)
        end
    end)

    local tgIcon = Instance.new("ImageLabel", toggle)
    tgIcon.Size = UDim2.new(0.72, 0, 0.72, 0)
    tgIcon.Position = UDim2.new(0.14, 0, 0.14, 0)
    tgIcon.BackgroundTransparency = 1
    tgIcon.Image = "rbxassetid://135314302105271"
    tgIcon.ScaleType = Enum.ScaleType.Fit

    -- ── Drag logic ──
    local dragging   = false
    local dragStart  = nil
    local dragOrigin = nil
    local wasMoved   = false

    toggle.InputBegan:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1
        or inp.UserInputType == Enum.UserInputType.Touch then
            dragging   = true
            wasMoved   = false
            dragStart  = inp.Position
            dragOrigin = Vector2.new(toggle.Position.X.Offset, toggle.Position.Y.Offset)
        end
    end)

    UIS.InputChanged:Connect(function(inp)
        if not dragging then return end
        if inp.UserInputType == Enum.UserInputType.MouseMovement
        or inp.UserInputType == Enum.UserInputType.Touch then
            local delta = inp.Position - dragStart
            if delta.Magnitude > 4 then wasMoved = true end
            local newX = math.clamp(dragOrigin.X + delta.X, 0,
                workspace.CurrentCamera.ViewportSize.X - 62)
            local newY = math.clamp(dragOrigin.Y + delta.Y, 0,
                workspace.CurrentCamera.ViewportSize.Y - 62)
            toggle.Position = UDim2.new(0, newX, 0, newY)
        end
    end)

    UIS.InputEnded:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1
        or inp.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)

    local mainVisible = true
    toggle.MouseButton1Click:Connect(function()
        if wasMoved then return end
        mainVisible = not mainVisible
        if mainVisible then
            main.Visible = true
            TweenService:Create(main, TweenInfo.new(0.2), {
                BackgroundTransparency = 0.08
            }):Play()
        else
            TweenService:Create(main, TweenInfo.new(0.15), {
                BackgroundTransparency = 1
            }):Play()
            task.delay(0.15, function() main.Visible = false end)
        end
    end)

end

task.spawn(doFixLag)
