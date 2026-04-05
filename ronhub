--[[
╔══════════════════════════════════════════════════════════════╗
║                  RAVEN HUB  •  v9.0                         ║
║            Rise of Nations  —  Full Custom UI               ║
║         No external libraries. 100% self-contained.         ║
║                  Toggle: RightControl                        ║
╚══════════════════════════════════════════════════════════════╝
--]]

-- ══════════════════════════════════════════════════════════════
--  SERVICES
-- ══════════════════════════════════════════════════════════════
local Players          = game:GetService("Players")
local TweenService     = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService       = game:GetService("RunService")

local player = Players.LocalPlayer
local pGui   = player.PlayerGui

-- ══════════════════════════════════════════════════════════════
--  WAIT FOR GAME MANAGER
-- ══════════════════════════════════════════════════════════════
local gm
for i = 1, 30 do
    gm = workspace:FindFirstChild("GameManager")
    if gm then break end
    task.wait(0.5)
end

if not gm then
    -- Show error and exit
    local errSg = Instance.new("ScreenGui", pGui)
    errSg.Name = "RavenError"; errSg.ResetOnSpawn = false
    local f = Instance.new("Frame", errSg)
    f.Size = UDim2.new(0, 360, 0, 70); f.Position = UDim2.new(0.5, -180, 0.02, 0)
    f.BackgroundColor3 = Color3.fromRGB(20, 8, 8); f.BorderSizePixel = 0
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 10)
    local l = Instance.new("TextLabel", f)
    l.Size = UDim2.new(1, 0, 1, 0); l.BackgroundTransparency = 1
    l.Text = "🦅  Raven Hub: Join a Rise of Nations server first!"
    l.TextColor3 = Color3.fromRGB(255, 90, 90); l.Font = Enum.Font.GothamBold; l.TextSize = 13; l.TextWrapped = true
    task.delay(6, function() errSg:Destroy() end)
    return
end

-- ══════════════════════════════════════════════════════════════
--  CORE HELPERS
-- ══════════════════════════════════════════════════════════════
_G.RAVEN_ON = true

local function dig(obj, ...)
    local cur = obj
    for _, k in ipairs({...}) do
        if not cur then return nil end
        cur = cur:FindFirstChild(k)
    end
    return cur
end

local function fire(remote, ...)
    if not remote then return end
    local a = {...}
    pcall(function() remote:FireServer(table.unpack(a)) end)
end

local SFX = {"K","M","B","T","Qa","Qi","Sx"}
local function fmt(n)
    n = tonumber(n) or 0
    for i = #SFX, 1, -1 do
        local v = 10^(i*3)
        if n >= v then return ("%.1f%s"):format(n/v, SFX[i]) end
    end
    return tostring(math.floor(n))
end

local function getCountry()
    local ls = dig(player,"leaderstats")
    local c  = ls and dig(ls,"Country")
    return c and c.Value or nil
end

local function getCities()
    local c = getCountry(); if not c then return {} end
    local f = dig(workspace,"Baseplate","Cities",c)
    return f and f:GetChildren() or {}
end

local function hasB(city, name)
    local b = city:FindFirstChild("Buildings")
    return b ~= nil and b:FindFirstChild(name) ~= nil
end

local function getRes(city)
    local t = {}
    local r = city:FindFirstChild("Resources")
    if r then for _, v in ipairs(r:GetChildren()) do t[v.Name] = v.Value end end
    return t
end

local function allCountries()
    local t = {}
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= player then
            local ls = dig(p,"leaderstats")
            if ls and dig(ls,"Country") then t[#t+1] = dig(ls,"Country").Value end
        end
    end
    return t
end

local function myUnits()
    local t = {}
    for _, u in ipairs(workspace.Units:GetChildren()) do
        local o = dig(u,"Owner")
        if o and o.Value == player.Name then t[#t+1] = u end
    end
    return t
end

local function enemyCities()
    local t, me = {}, getCountry(); if not me then return t end
    local base = dig(workspace,"Baseplate","Cities"); if not base then return t end
    for _, folder in ipairs(base:GetChildren()) do
        if folder.Name ~= me then
            for _, city in ipairs(folder:GetChildren()) do t[#t+1] = city end
        end
    end
    return t
end

-- ══════════════════════════════════════════════════════════════
--  CONSTANTS
-- ══════════════════════════════════════════════════════════════
local ORE_SET = {Oil=1,Copper=1,Diamond=1,Gold=1,Iron=1,Phosphate=1,
    Tungsten=1,Uranium=1,Steel=1,Titanium=1,Chromium=1,Aluminum=1}

local UNIT_TYPES = {"Infantry","Tank","Anti Aircraft","Artillery",
    "Fighter","Attacker","Bomber","Destroyer","Frigate","Battleship","Aircraft Carrier","Submarine"}

local UNIT_REQ = {Fighter="Airport",Attacker="Airport",Bomber="Airport",
    Destroyer="Port",Frigate="Port",Battleship="Port",["Aircraft Carrier"]="Port",Submarine="Port"}

local FACTORY_TYPES = {"Factory","Arms Factory","Fertilizer Factory","Oil Refinery",
    "Research Lab","Recruitment Center","Nuclear Plant","Shipyard","Airport","Port"}

local ALL_RES = {"Oil","Copper","Diamond","Gold","Iron","Phosphate","Tungsten",
    "Uranium","Steel","Titanium","Chromium","Aluminum","Electronics","Fertilizer","Food","Money","Coal","Consumer Goods"}

local TECH_CATS = {"Military","Economic","Industry","Naval","Air","Nuclear","Recon","Infrastructure"}

-- ══════════════════════════════════════════════════════════════
--  STATE
-- ══════════════════════════════════════════════════════════════
local S = {
    minesMin=1, selFactory="Factory",
    ubAmt=10, ubType="Infantry", ubDelay=0.12, balCount=5,
    espOn=false, showEnemy=true,
    autoClearAlerts=false,
    selCountry=nil, warTarget=nil, allyT={}, puppetT={},
    justifType="Conquest", spamJustif=false,
    sellRes="Oil", sellAmt=1000, buyRes="Titanium", buyAmt=1000,
    aiOnly=true, autoCGSell=false, massBuyAI=false, autoBuyType="Titanium",
    autoDeclare=false, autoAssign=false, autoBomber=false,
    autoAircraft=false, tgtAttacker=0, tgtFighter=0, tgtBomber=1,
    antiBomber=false, warCrime=false,
    autoTankSpawn=false, tankSpawnAmt=5, tankSpawnInt=30,
    autoTankAtk=false, tankAtkInt=15,
    autoInfSpawn=false, infSpawnAmt=5, infSpawnInt=30,
    autoInfAtk=false, infAtkInt=15,
    autoConst=false, autoConstFactory="Factory", autoConstQty=2,
    autoMine=false, autoMineInt=60,
    autoFactory=false, autoFactInt=120,
    autoUnit=false, autoUnitInt=30, autoUnitType="Infantry",
    autoSell=false, autoSellInt=45,
    autoAlly=false, autoPuppet=false,
    autoFarm=false, farmPhase=0,
    smartResearch=false, bestLaws=false,
    selTax="Maximum", selCons="Total War", selSkin="Default",
    actionLog={},
}

local function addLog(msg)
    table.insert(S.actionLog, 1, os.date("%H:%M:%S").."  "..msg)
    if #S.actionLog > 40 then table.remove(S.actionLog) end
end

-- ══════════════════════════════════════════════════════════════
--  DESIGN TOKENS
-- ══════════════════════════════════════════════════════════════
local C = {
    bg         = Color3.fromRGB(10,  10,  16),
    surface    = Color3.fromRGB(16,  16,  26),
    surface2   = Color3.fromRGB(22,  22,  36),
    surface3   = Color3.fromRGB(28,  28,  46),
    border     = Color3.fromRGB(38,  38,  60),
    accent     = Color3.fromRGB(99,  155, 255),
    accentDim  = Color3.fromRGB(55,  90,  170),
    green      = Color3.fromRGB(72,  220, 128),
    red        = Color3.fromRGB(240, 70,  70),
    yellow     = Color3.fromRGB(255, 200, 50),
    orange     = Color3.fromRGB(255, 140, 50),
    text       = Color3.fromRGB(220, 228, 255),
    textDim    = Color3.fromRGB(130, 140, 175),
    textMuted  = Color3.fromRGB(70,  80,  110),
    white      = Color3.fromRGB(255, 255, 255),
}

local FONT_BOLD  = Enum.Font.GothamBold
local FONT_MED   = Enum.Font.Gotham
local FONT_MONO  = Enum.Font.Code

local function tw(obj, props, t, style, dir)
    TweenService:Create(obj,
        TweenInfo.new(t or 0.18, style or Enum.EasingStyle.Quart, dir or Enum.EasingDirection.Out),
        props
    ):Play()
end

-- ══════════════════════════════════════════════════════════════
--  UI BUILDER HELPERS
-- ══════════════════════════════════════════════════════════════
local function frame(parent, props)
    local f = Instance.new("Frame", parent)
    f.BorderSizePixel = 0
    for k,v in pairs(props or {}) do f[k] = v end
    return f
end

local function corner(parent, radius)
    local c = Instance.new("UICorner", parent)
    c.CornerRadius = UDim.new(0, radius or 8)
    return c
end

local function label(parent, props)
    local l = Instance.new("TextLabel", parent)
    l.BackgroundTransparency = 1
    l.BorderSizePixel = 0
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.Font = FONT_MED
    l.TextSize = 13
    l.TextColor3 = C.text
    for k,v in pairs(props or {}) do l[k] = v end
    return l
end

local function listLayout(parent, pad, dir)
    local ll = Instance.new("UIListLayout", parent)
    ll.SortOrder = Enum.SortOrder.LayoutOrder
    ll.Padding = UDim.new(0, pad or 6)
    ll.FillDirection = dir or Enum.FillDirection.Vertical
    return ll
end

local function padding(parent, all, top, left, right, bottom)
    local p = Instance.new("UIPadding", parent)
    if all then
        p.PaddingTop    = UDim.new(0, all)
        p.PaddingBottom = UDim.new(0, all)
        p.PaddingLeft   = UDim.new(0, all)
        p.PaddingRight  = UDim.new(0, all)
    end
    if top    then p.PaddingTop    = UDim.new(0, top)    end
    if bottom then p.PaddingBottom = UDim.new(0, bottom) end
    if left   then p.PaddingLeft   = UDim.new(0, left)   end
    if right  then p.PaddingRight  = UDim.new(0, right)  end
    return p
end

-- ══════════════════════════════════════════════════════════════
--  TOAST NOTIFICATION SYSTEM
-- ══════════════════════════════════════════════════════════════
local toastQueue = {}
local toastActive = false

local function showToast(title, body, accent)
    table.insert(toastQueue, {title=title, body=body, accent=accent or C.accent})
    if toastActive then return end
    toastActive = true
    task.spawn(function()
        while #toastQueue > 0 do
            local data = table.remove(toastQueue, 1)

            local sg = Instance.new("ScreenGui", pGui)
            sg.Name = "RavenToast"; sg.ResetOnSpawn = false; sg.DisplayOrder = 9999

            local card = frame(sg, {
                Size = UDim2.new(0, 300, 0, 64),
                Position = UDim2.new(1, 10, 0, 20),
                BackgroundColor3 = C.surface,
            })
            corner(card, 10)

            local accentBar = frame(card, {
                Size = UDim2.new(0, 3, 1, -12),
                Position = UDim2.new(0, 0, 0, 6),
                BackgroundColor3 = data.accent,
            })
            corner(accentBar, 3)

            label(card, {
                Size = UDim2.new(1, -18, 0, 20),
                Position = UDim2.new(0, 14, 0, 8),
                Text = data.title,
                TextColor3 = data.accent,
                Font = FONT_BOLD,
                TextSize = 12,
            })
            label(card, {
                Size = UDim2.new(1, -18, 0, 18),
                Position = UDim2.new(0, 14, 0, 30),
                Text = data.body,
                TextColor3 = C.textDim,
                TextSize = 11,
            })

            -- shadow
            local shadow = Instance.new("ImageLabel", card)
            shadow.Size = UDim2.new(1, 24, 1, 24)
            shadow.Position = UDim2.new(0, -12, 0, -8)
            shadow.BackgroundTransparency = 1
            shadow.Image = "rbxassetid://6014261993"
            shadow.ImageColor3 = Color3.fromRGB(0,0,0)
            shadow.ImageTransparency = 0.65
            shadow.ScaleType = Enum.ScaleType.Slice
            shadow.SliceCenter = Rect.new(49,49,450,450)
            shadow.ZIndex = -1

            tw(card, {Position = UDim2.new(1, -314, 0, 20)})
            task.wait(3.2)
            tw(card, {Position = UDim2.new(1, 10, 0, 20)})
            task.wait(0.25)
            sg:Destroy()
            task.wait(0.1)
        end
        toastActive = false
    end)
end

local function N(title, body, col)
    showToast(title, body, col)
end

-- ══════════════════════════════════════════════════════════════
--  MAIN GUI STRUCTURE
-- ══════════════════════════════════════════════════════════════
local MainGui = Instance.new("ScreenGui", pGui)
MainGui.Name = "RavenHub_v9"
MainGui.ResetOnSpawn = false
MainGui.DisplayOrder = 100
MainGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- Root panel
local Root = frame(MainGui, {
    Size = UDim2.new(0, 700, 0, 500),
    Position = UDim2.new(0.5, -350, 0.5, -250),
    BackgroundColor3 = C.bg,
    ClipsDescendants = true,
})
corner(Root, 14)

-- Subtle gradient
local rootGrad = Instance.new("UIGradient", Root)
rootGrad.Rotation = 135
rootGrad.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(13,13,22)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(8,8,14)),
})

-- Drop shadow
local rootShadow = Instance.new("ImageLabel", Root)
rootShadow.Size = UDim2.new(1, 40, 1, 40)
rootShadow.Position = UDim2.new(0, -20, 0, -16)
rootShadow.BackgroundTransparency = 1
rootShadow.Image = "rbxassetid://6014261993"
rootShadow.ImageColor3 = Color3.fromRGB(0,0,0)
rootShadow.ImageTransparency = 0.55
rootShadow.ScaleType = Enum.ScaleType.Slice
rootShadow.SliceCenter = Rect.new(49,49,450,450)
rootShadow.ZIndex = -1

-- ── Topbar ───────────────────────────────────────────────────
local topbar = frame(Root, {
    Size = UDim2.new(1, 0, 0, 48),
    BackgroundColor3 = C.surface,
})
corner(topbar, 14)
-- square bottom
frame(topbar, {Size=UDim2.new(1,0,0.5,0), Position=UDim2.new(0,0,0.5,0), BackgroundColor3=C.surface})

local accentStripe = frame(topbar, {
    Size = UDim2.new(0, 3, 0, 28),
    Position = UDim2.new(0, 14, 0, 10),
    BackgroundColor3 = C.accent,
})
corner(accentStripe, 2)

label(topbar, {
    Position = UDim2.new(0, 26, 0, 7),
    Size = UDim2.new(0, 200, 0, 20),
    Text = "🦅  RAVEN HUB",
    TextColor3 = C.white,
    Font = FONT_BOLD,
    TextSize = 15,
})
label(topbar, {
    Position = UDim2.new(0, 26, 0, 28),
    Size = UDim2.new(0, 250, 0, 14),
    Text = "Rise of Nations  •  v9.0",
    TextColor3 = C.textMuted,
    Font = FONT_MED,
    TextSize = 10,
})

-- live stats
local statsTroops = label(topbar, {
    Position = UDim2.new(1, -330, 0, 8),
    Size = UDim2.new(0, 140, 0, 14),
    Text = "Troops: —",
    TextColor3 = C.green,
    Font = FONT_BOLD,
    TextSize = 11,
    TextXAlignment = Enum.TextXAlignment.Right,
})
local statsCities = label(topbar, {
    Position = UDim2.new(1, -330, 0, 26),
    Size = UDim2.new(0, 140, 0, 14),
    Text = "Cities: —",
    TextColor3 = C.accent,
    Font = FONT_BOLD,
    TextSize = 11,
    TextXAlignment = Enum.TextXAlignment.Right,
})

-- close & minimise
local function makeTopBtn(xOffset, text, bgColor, callback)
    local btn = Instance.new("TextButton", topbar)
    btn.Size = UDim2.new(0, 28, 0, 28)
    btn.Position = UDim2.new(1, xOffset, 0.5, -14)
    btn.BackgroundColor3 = bgColor
    btn.BorderSizePixel = 0
    btn.Text = text
    btn.TextColor3 = C.white
    btn.Font = FONT_BOLD
    btn.TextSize = 13
    corner(btn, 7)
    btn.MouseButton1Click:Connect(callback)
    btn.MouseEnter:Connect(function() tw(btn,{BackgroundTransparency=0.3}) end)
    btn.MouseLeave:Connect(function() tw(btn,{BackgroundTransparency=0}) end)
    return btn
end

makeTopBtn(-10,  "✕", Color3.fromRGB(185,50,50),   function() Root.Visible = false end)
makeTopBtn(-44,  "−", Color3.fromRGB(40,40,65),     function()
    Root.Size = Root.Size.Y.Offset > 50
        and UDim2.new(0,700,0,48) or UDim2.new(0,700,0,500)
end)

-- Drag
local dragActive, dragStart, rootStart = false, nil, nil
topbar.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then
        dragActive = true; dragStart = i.Position; rootStart = Root.Position
    end
end)
UserInputService.InputChanged:Connect(function(i)
    if dragActive and i.UserInputType == Enum.UserInputType.MouseMovement then
        local d = i.Position - dragStart
        Root.Position = UDim2.new(rootStart.X.Scale, rootStart.X.Offset + d.X,
                                  rootStart.Y.Scale, rootStart.Y.Offset + d.Y)
    end
end)
UserInputService.InputEnded:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then dragActive = false end
end)

-- Toggle with RightControl
UserInputService.InputBegan:Connect(function(i, gp)
    if not gp and i.KeyCode == Enum.KeyCode.RightControl then
        Root.Visible = not Root.Visible
    end
end)

-- ── Sidebar (tab buttons) ─────────────────────────────────────
local sidebar = frame(Root, {
    Size = UDim2.new(0, 140, 1, -48),
    Position = UDim2.new(0, 0, 0, 48),
    BackgroundColor3 = C.surface,
})
listLayout(sidebar, 2)
padding(sidebar, nil, 8, 6, 6, 8)

-- ── Content area ──────────────────────────────────────────────
local contentArea = frame(Root, {
    Size = UDim2.new(1, -140, 1, -48),
    Position = UDim2.new(0, 140, 0, 48),
    BackgroundTransparency = 1,
})

-- Sidebar-content divider
frame(Root, {
    Size = UDim2.new(0, 1, 1, -48),
    Position = UDim2.new(0, 140, 0, 48),
    BackgroundColor3 = C.border,
})

-- ══════════════════════════════════════════════════════════════
--  TAB SYSTEM
-- ══════════════════════════════════════════════════════════════
local tabs = {}
local activeTab = nil

local function makeContent()
    local scroll = Instance.new("ScrollingFrame", contentArea)
    scroll.Size = UDim2.new(1, 0, 1, 0)
    scroll.BackgroundTransparency = 1
    scroll.BorderSizePixel = 0
    scroll.ScrollBarThickness = 3
    scroll.ScrollBarImageColor3 = C.accent
    scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
    scroll.Visible = false
    local ll = listLayout(scroll, 6)
    padding(scroll, nil, 10, 10, 10, 10)
    -- auto-resize canvas
    ll.Changed:Connect(function()
        scroll.CanvasSize = UDim2.new(0, 0, 0, ll.AbsoluteContentSize.Y + 20)
    end)
    return scroll
end

local function addTab(icon, name)
    local btn = Instance.new("TextButton", sidebar)
    btn.Size = UDim2.new(1, 0, 0, 36)
    btn.BackgroundColor3 = C.surface2
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = false
    btn.TextXAlignment = Enum.TextXAlignment.Left
    btn.Text = icon .. "  " .. name
    btn.TextColor3 = C.textDim
    btn.Font = FONT_MED
    btn.TextSize = 12
    corner(btn, 7)
    padding(btn, nil, nil, 10, nil, nil)

    local content = makeContent()

    btn.MouseEnter:Connect(function()
        if activeTab ~= name then
            tw(btn, {BackgroundColor3 = C.surface3})
        end
    end)
    btn.MouseLeave:Connect(function()
        if activeTab ~= name then
            tw(btn, {BackgroundColor3 = C.surface2})
        end
    end)
    btn.MouseButton1Click:Connect(function()
        if activeTab == name then return end
        -- deactivate old
        if activeTab and tabs[activeTab] then
            tw(tabs[activeTab].btn, {BackgroundColor3=C.surface2, TextColor3=C.textDim})
            tabs[activeTab].content.Visible = false
        end
        -- activate new
        activeTab = name
        tw(btn, {BackgroundColor3=C.accentDim, TextColor3=C.white})
        content.Visible = true
    end)

    tabs[name] = {btn=btn, content=content}
    return content
end

-- ══════════════════════════════════════════════════════════════
--  COMPONENT LIBRARY (builds into a scroll content frame)
-- ══════════════════════════════════════════════════════════════

local function section(parent, text)
    local s = label(parent, {
        Size = UDim2.new(1, 0, 0, 22),
        Text = "  " .. text:upper(),
        TextColor3 = C.accent,
        Font = FONT_BOLD,
        TextSize = 10,
        LayoutOrder = 0,
    })
    local line = frame(parent, {
        Size = UDim2.new(1, 0, 0, 1),
        BackgroundColor3 = C.border,
    })
    return s
end

local function infoCard(parent, text)
    local card = frame(parent, {
        Size = UDim2.new(1, 0, 0, 0),
        AutomaticSize = Enum.AutomaticSize.Y,
        BackgroundColor3 = C.surface2,
    })
    corner(card, 8)
    padding(card, 10)
    local l = label(card, {
        Size = UDim2.new(1, 0, 0, 0),
        AutomaticSize = Enum.AutomaticSize.Y,
        Text = text,
        TextColor3 = C.textDim,
        TextSize = 11,
        TextWrapped = true,
    })
    return card
end

local function statusCard(parent, initial, accentColor)
    local card = frame(parent, {
        Size = UDim2.new(1, 0, 0, 32),
        BackgroundColor3 = C.surface2,
    })
    corner(card, 8)
    local bar = frame(card, {
        Size = UDim2.new(0, 3, 0, 20),
        Position = UDim2.new(0, 0, 0.5, -10),
        BackgroundColor3 = accentColor or C.textMuted,
    })
    corner(bar, 2)
    local lbl = label(card, {
        Position = UDim2.new(0, 12, 0, 0),
        Size = UDim2.new(1, -16, 1, 0),
        Text = initial or "Inactive.",
        TextColor3 = C.textDim,
        TextSize = 11,
    })
    local function setStatus(text, col)
        lbl.Text = text
        if col then bar.BackgroundColor3 = col; lbl.TextColor3 = col end
    end
    return card, setStatus
end

local BTN_COLORS = {
    default  = Color3.fromRGB(28, 28, 52),
    hover    = Color3.fromRGB(42, 42, 72),
    active   = Color3.fromRGB(55, 100, 200),
    green    = Color3.fromRGB(20, 80, 40),
    greenH   = Color3.fromRGB(30, 110, 55),
    red      = Color3.fromRGB(90, 20, 20),
    redH     = Color3.fromRGB(130, 30, 30),
    orange   = Color3.fromRGB(90, 50, 10),
    orangeH  = Color3.fromRGB(130, 75, 15),
}

local function button(parent, text, callback, variant)
    variant = variant or "default"
    local bg   = BTN_COLORS[variant]    or BTN_COLORS.default
    local bgH  = BTN_COLORS[variant.."H"] or BTN_COLORS.hover

    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(1, 0, 0, 36)
    btn.BackgroundColor3 = bg
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = false
    btn.Text = text
    btn.TextColor3 = C.text
    btn.Font = FONT_MED
    btn.TextSize = 12
    btn.TextXAlignment = Enum.TextXAlignment.Left
    corner(btn, 8)
    padding(btn, nil, nil, 12, nil, nil)

    btn.MouseEnter:Connect(function() tw(btn,{BackgroundColor3=bgH}) end)
    btn.MouseLeave:Connect(function() tw(btn,{BackgroundColor3=bg}) end)
    btn.MouseButton1Click:Connect(function()
        tw(btn,{BackgroundColor3=BTN_COLORS.active})
        task.delay(0.15, function() tw(btn,{BackgroundColor3=bg}) end)
        task.spawn(callback)
    end)
    return btn
end

local function toggle(parent, text, initial, callback)
    local row = frame(parent, {
        Size = UDim2.new(1, 0, 0, 38),
        BackgroundColor3 = C.surface2,
    })
    corner(row, 8)

    label(row, {
        Position = UDim2.new(0, 12, 0, 0),
        Size = UDim2.new(1, -60, 1, 0),
        Text = text,
        TextSize = 12,
        TextColor3 = C.text,
    })

    local track = frame(row, {
        Size = UDim2.new(0, 36, 0, 20),
        Position = UDim2.new(1, -46, 0.5, -10),
        BackgroundColor3 = initial and C.accentDim or C.surface3,
    })
    corner(track, 10)

    local knob = frame(track, {
        Size = UDim2.new(0, 14, 0, 14),
        Position = UDim2.new(0, initial and 18 or 4, 0, 3),
        BackgroundColor3 = C.white,
    })
    corner(knob, 7)

    local state = initial or false

    local clickTarget = Instance.new("TextButton", row)
    clickTarget.Size = UDim2.new(1, 0, 1, 0)
    clickTarget.BackgroundTransparency = 1
    clickTarget.Text = ""

    clickTarget.MouseButton1Click:Connect(function()
        state = not state
        tw(track, {BackgroundColor3 = state and C.accentDim or C.surface3})
        tw(knob,  {Position = UDim2.new(0, state and 18 or 4, 0, 3)})
        task.spawn(callback, state)
    end)
    return row, function(v)
        state = v
        tw(track, {BackgroundColor3 = state and C.accentDim or C.surface3})
        tw(knob,  {Position = UDim2.new(0, state and 18 or 4, 0, 3)})
    end
end

local function slider(parent, text, min, max, default, callback)
    local container = frame(parent, {
        Size = UDim2.new(1, 0, 0, 54),
        BackgroundColor3 = C.surface2,
    })
    corner(container, 8)

    local valLbl = label(container, {
        Position = UDim2.new(1, -50, 0, 8),
        Size = UDim2.new(0, 44, 0, 16),
        Text = tostring(default),
        TextColor3 = C.accent,
        Font = FONT_BOLD,
        TextSize = 12,
        TextXAlignment = Enum.TextXAlignment.Right,
    })
    label(container, {
        Position = UDim2.new(0, 12, 0, 8),
        Size = UDim2.new(1, -70, 0, 16),
        Text = text,
        TextSize = 12,
        TextColor3 = C.text,
    })

    local track = frame(container, {
        Size = UDim2.new(1, -24, 0, 4),
        Position = UDim2.new(0, 12, 0, 36),
        BackgroundColor3 = C.surface3,
    })
    corner(track, 2)

    local fill = frame(track, {
        Size = UDim2.new((default-min)/(max-min), 0, 1, 0),
        BackgroundColor3 = C.accent,
    })
    corner(fill, 2)

    local knob = frame(track, {
        Size = UDim2.new(0, 14, 0, 14),
        Position = UDim2.new((default-min)/(max-min), -7, 0, -5),
        BackgroundColor3 = C.white,
    })
    corner(knob, 7)

    local current = default
    local dragging = false

    local function updatePos(mouseX)
        local absPos  = track.AbsolutePosition.X
        local absSize = track.AbsoluteSize.X
        local t = math.clamp((mouseX - absPos) / absSize, 0, 1)
        current = math.floor(min + t*(max-min) + 0.5)
        t = (current-min)/(max-min)
        fill.Size = UDim2.new(t, 0, 1, 0)
        knob.Position = UDim2.new(t, -7, 0, -5)
        valLbl.Text = tostring(current)
        callback(current)
    end

    local clickBtn = Instance.new("TextButton", track)
    clickBtn.Size = UDim2.new(1, 0, 1, 22)
    clickBtn.Position = UDim2.new(0, 0, 0, -9)
    clickBtn.BackgroundTransparency = 1
    clickBtn.Text = ""
    clickBtn.ZIndex = 2

    clickBtn.MouseButton1Down:Connect(function() dragging = true end)
    clickBtn.MouseButton1Up:Connect(function() dragging = false end)
    clickBtn.MouseLeave:Connect(function() dragging = false end)

    UserInputService.InputChanged:Connect(function(i)
        if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then
            updatePos(i.Position.X)
        end
    end)
    UserInputService.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)
    clickBtn.MouseButton1Click:Connect(function(x, y)
        updatePos(UserInputService:GetMouseLocation().X)
    end)

    return container
end

local function dropdown(parent, text, options, default, callback)
    local current = default or options[1]

    local container = frame(parent, {
        Size = UDim2.new(1, 0, 0, 38),
        BackgroundColor3 = C.surface2,
        ZIndex = 2,
    })
    corner(container, 8)
    container.ClipsDescendants = false

    label(container, {
        Position = UDim2.new(0, 12, 0, 0),
        Size = UDim2.new(0.5, 0, 1, 0),
        Text = text,
        TextSize = 12,
        TextColor3 = C.text,
        ZIndex = 3,
    })
    local valLbl = label(container, {
        Position = UDim2.new(0.5, 0, 0, 0),
        Size = UDim2.new(0.5, -36, 1, 0),
        Text = current,
        TextSize = 11,
        TextColor3 = C.accent,
        Font = FONT_BOLD,
        TextXAlignment = Enum.TextXAlignment.Right,
        ZIndex = 3,
    })
    label(container, {
        Position = UDim2.new(1, -28, 0, 0),
        Size = UDim2.new(0, 24, 1, 0),
        Text = "▾",
        TextSize = 13,
        TextColor3 = C.textDim,
        TextXAlignment = Enum.TextXAlignment.Center,
        ZIndex = 3,
    })

    local dropOpen = false
    local dropFrame = nil

    local btn = Instance.new("TextButton", container)
    btn.Size = UDim2.new(1, 0, 1, 0)
    btn.BackgroundTransparency = 1
    btn.Text = ""
    btn.ZIndex = 4

    btn.MouseButton1Click:Connect(function()
        dropOpen = not dropOpen
        if dropFrame then dropFrame:Destroy(); dropFrame = nil end
        if not dropOpen then return end

        dropFrame = frame(container, {
            Size = UDim2.new(1, 0, 0, math.min(#options, 6) * 30 + 8),
            Position = UDim2.new(0, 0, 1, 4),
            BackgroundColor3 = C.surface3,
            ZIndex = 20,
        })
        corner(dropFrame, 8)

        local ll = listLayout(dropFrame, 2)
        padding(dropFrame, 4)

        for _, opt in ipairs(options) do
            local ob = Instance.new("TextButton", dropFrame)
            ob.Size = UDim2.new(1, 0, 0, 26)
            ob.BackgroundColor3 = opt == current and C.accentDim or Color3.fromRGB(0,0,0)
            ob.BackgroundTransparency = opt == current and 0 or 1
            ob.BorderSizePixel = 0
            ob.AutoButtonColor = false
            ob.Text = opt
            ob.TextColor3 = opt == current and C.white or C.textDim
            ob.Font = FONT_MED
            ob.TextSize = 11
            ob.TextXAlignment = Enum.TextXAlignment.Left
            corner(ob, 6)
            padding(ob, nil, nil, 8, nil, nil)
            ob.ZIndex = 21
            ob.MouseEnter:Connect(function()
                if opt ~= current then tw(ob,{BackgroundColor3=C.surface2, BackgroundTransparency=0}) end
            end)
            ob.MouseLeave:Connect(function()
                if opt ~= current then tw(ob,{BackgroundTransparency=1}) end
            end)
            ob.MouseButton1Click:Connect(function()
                current = opt
                valLbl.Text = opt
                callback(opt)
                dropFrame:Destroy(); dropFrame = nil; dropOpen = false
            end)
        end
    end)

    return container
end

local function input(parent, text, placeholder, callback)
    local container = frame(parent, {
        Size = UDim2.new(1, 0, 0, 38),
        BackgroundColor3 = C.surface2,
    })
    corner(container, 8)

    label(container, {
        Position = UDim2.new(0, 12, 0, 0),
        Size = UDim2.new(0.42, 0, 1, 0),
        Text = text,
        TextSize = 12,
        TextColor3 = C.text,
    })

    local tb = Instance.new("TextBox", container)
    tb.Size = UDim2.new(0.55, 0, 0, 24)
    tb.Position = UDim2.new(0.43, 0, 0.5, -12)
    tb.BackgroundColor3 = C.surface3
    tb.BorderSizePixel = 0
    tb.Text = ""
    tb.PlaceholderText = placeholder or "type here..."
    tb.PlaceholderColor3 = C.textMuted
    tb.TextColor3 = C.text
    tb.Font = FONT_MED
    tb.TextSize = 11
    tb.ClearTextOnFocus = false
    corner(tb, 6)
    padding(tb, nil, nil, 8, nil, nil)

    tb:GetPropertyChangedSignal("Text"):Connect(function()
        callback(tb.Text)
    end)
    return container
end

-- ══════════════════════════════════════════════════════════════
--  POPULATE TABS
-- ══════════════════════════════════════════════════════════════

-- ─── OVERVIEW ─────────────────────────────────────────────────
do
    local c = addTab("🏠","Overview")
    section(c, "Quick Stats")
    local liveCityLbl = label(c, {Size=UDim2.new(1,0,0,18), Text="Cities: loading…", TextColor3=C.accent, TextSize=12})
    local liveTroopLbl = label(c, {Size=UDim2.new(1,0,0,18), Text="Troops: loading…", TextColor3=C.green, TextSize=12})
    local liveFarmLbl = label(c, {Size=UDim2.new(1,0,0,18), Text="AutoFarm: Inactive", TextColor3=C.textDim, TextSize=12})

    task.spawn(function()
        while task.wait(7) do
            if not _G.RAVEN_ON then break end
            local cc = getCities()
            liveCityLbl.Text = "🏙  Cities: " .. #cc
            local tot = 0
            for _, u in ipairs(myUnits()) do
                local cur = dig(u,"Current"); if cur then tot = tot + cur.Value end
            end
            liveTroopLbl.Text = "⚔  Troops: " .. fmt(tot)
            local fp = S.farmPhase == 0 and "Inactive" or ("Phase "..S.farmPhase)
            liveFarmLbl.Text = "⚡ AutoFarm: " .. fp
            -- topbar stats
            statsCities.Text = "Cities: " .. #cc
            statsTroops.Text = "Troops: " .. fmt(tot)
        end
    end)

    section(c, "Quick Actions")
    button(c, "⚡  One-Click Full City Development", function()
        local n = 0
        for _, city in ipairs(getCities()) do
            local res, ok = getRes(city), false
            for ore in pairs(ORE_SET) do if (res[ore] or 0) >= 1 then ok=true; break end end
            if ok and not hasB(city,"Mine") then task.wait(0.1); fire(gm.CreateBuilding,{[1]=city},"Mines"); n=n+1 end
            for _, b in ipairs({"Factory","Arms Factory","Recruitment Center","Research Lab"}) do
                if not hasB(city,b) then task.wait(0.1); fire(gm.CreateBuilding,{[1]=city},b); n=n+1 end
            end
        end
        N("Full Dev","Queued "..n.." buildings",C.green); addLog("Full dev: "..n)
    end, "green")

    button(c, "⚔️  Quick Max Army (Land+Air+Navy ×10)", function()
        local land={"Infantry","Tank","Anti Aircraft","Artillery"}
        local airT={"Fighter","Attacker","Bomber"}; local seaT={"Destroyer","Frigate","Battleship"}
        local cc=getCities(); local airC,portC={},{}
        for _,city in ipairs(cc) do
            if hasB(city,"Airport") then airC[#airC+1]=city end
            if hasB(city,"Port")    then portC[#portC+1]=city end
        end
        local n=0
        if #cc>0 then for _,t in ipairs(land) do for i=1,10 do fire(gm.CreateUnit,{[1]=cc[((i-1)%#cc)+1]},t); n=n+1; task.wait(0.07) end end end
        if #airC>0 then for _,t in ipairs(airT) do for i=1,10 do fire(gm.CreateUnit,{[1]=airC[((i-1)%#airC)+1]},t); n=n+1; task.wait(0.07) end end end
        if #portC>0 then for _,t in ipairs(seaT) do for i=1,10 do fire(gm.CreateUnit,{[1]=portC[((i-1)%#portC)+1]},t); n=n+1; task.wait(0.07) end end end
        N("Max Army","Built "..n.." units",C.orange); addLog("Max army: "..n)
    end, "orange")

    button(c, "📋  Print Action Log (F9)", function()
        print("╔═══ RAVEN HUB ACTION LOG ═══════╗")
        for _,e in ipairs(S.actionLog) do print("║  "..e) end
        print("╚════════════════════════════════╝")
        N("Log",#S.actionLog.." entries printed")
    end)

    button(c, "📊  Full Nation Report (F9)", function()
        local cc = getCities()
        local mines,facts,ports,air=0,0,0,0
        for _,city in ipairs(cc) do
            if hasB(city,"Mine") then mines=mines+1 end
            if hasB(city,"Factory") then facts=facts+1 end
            if hasB(city,"Port") then ports=ports+1 end
            if hasB(city,"Airport") then air=air+1 end
        end
        print("╔═══ RAVEN HUB — NATION REPORT ══╗")
        print("║  ".. (getCountry() or "Unknown"))
        print(("║  Cities:%d  Mines:%d  Factories:%d"):format(#cc,mines,facts))
        print(("║  Ports:%d  Airports:%d"):format(ports,air))
        print("╚════════════════════════════════╝")
        N("Report","Printed to F9",C.accent)
    end)
end

-- ─── BUILDING ─────────────────────────────────────────────────
do
    local c = addTab("🏗","Building")
    section(c, "Mines")
    slider(c, "Min Ore Amount", 1, 20, 1, function(v) S.minesMin=v end)
    button(c, "⛏️  Build Mines (Smart)", function()
        local built,skip=0,0
        for _,city in ipairs(getCities()) do
            if not hasB(city,"Mine") then
                local res,ok=getRes(city),false
                for ore in pairs(ORE_SET) do if (res[ore] or 0)>=S.minesMin then ok=true;break end end
                if ok then task.wait(0.1);fire(gm.CreateBuilding,{[1]=city},"Mines");built=built+1 else skip=skip+1 end
            else skip=skip+1 end
        end
        N("Mines","Built "..built.."  Skipped "..skip,C.green); addLog("Mines: "..built)
    end, "green")

    section(c, "Factories")
    dropdown(c, "Building Type", FACTORY_TYPES, "Factory", function(v) S.selFactory=v end)
    button(c, "🏭  Build in All Cities", function()
        local n=0
        for _,city in ipairs(getCities()) do
            if not hasB(city,S.selFactory) then task.wait(0.1);fire(gm.CreateBuilding,{[1]=city},S.selFactory);n=n+1 end
        end
        N("Factories","Built "..n.." × "..S.selFactory,C.green); addLog("Factory: "..n)
    end, "green")

    section(c, "Auto Construction")
    infoCard(c, "Select factory type and quantity, then build in random eligible cities. Toggle to run every 90s automatically.")
    dropdown(c, "Factory Type", FACTORY_TYPES, "Fertilizer Factory", function(v) S.autoConstFactory=v end)
    slider(c, "Quantity to Build", 1, 20, 2, function(v) S.autoConstQty=v end)
    button(c, "🔨  Build Factories Now", function()
        local eligible={}
        for _,city in ipairs(getCities()) do
            if not hasB(city,S.autoConstFactory) then eligible[#eligible+1]=city end
        end
        if #eligible==0 then N("Construction","All cities have "..S.autoConstFactory); return end
        for i=#eligible,2,-1 do local j=math.random(1,i); eligible[i],eligible[j]=eligible[j],eligible[i] end
        local built=0
        for i=1,math.min(S.autoConstQty,#eligible) do
            fire(gm.CreateBuilding,{[1]=eligible[i]},S.autoConstFactory); built=built+1; task.wait(0.12)
        end
        N("Construction","Built "..built.." × "..S.autoConstFactory,C.green); addLog("AutoConst: "..built)
    end, "green")

    local _, setConstSt = statusCard(c, "Auto Construction: Inactive.")
    toggle(c, "Auto Construction (Loop)", false, function(v)
        S.autoConst=v
        setConstSt(v and "Active — building "..S.autoConstFactory or "Inactive.", v and C.yellow or nil)
        N("Construction",v and "Auto Construction ON" or "OFF",v and C.yellow or C.textDim)
    end)

    section(c, "Mass Build")
    button(c, "✈️  Airports + Ports Everywhere", function()
        local n=0
        for _,city in ipairs(getCities()) do
            for _,b in ipairs({"Airport","Port"}) do
                if not hasB(city,b) then task.wait(0.1);fire(gm.CreateBuilding,{[1]=city},b);n=n+1 end
            end
        end
        N("Infra","Built "..n.." airports/ports",C.green)
    end, "green")
    button(c, "☢️  Nuclear Plants Everywhere", function()
        local n=0
        for _,city in ipairs(getCities()) do
            if not hasB(city,"Nuclear Plant") then task.wait(0.15);fire(gm.CreateBuilding,{[1]=city},"Nuclear Plant");n=n+1 end
        end
        N("Nuclear","Built "..n.." plants",C.green)
    end, "green")
    button(c, "🔄  Full Development (All Types)", function()
        local n=0
        local blds={"Recruitment Center","Factory","Arms Factory","Research Lab","Airport","Port"}
        for _,city in ipairs(getCities()) do
            local res,ok=getRes(city),false
            for ore in pairs(ORE_SET) do if (res[ore] or 0)>=1 then ok=true;break end end
            if ok and not hasB(city,"Mine") then task.wait(0.1);fire(gm.CreateBuilding,{[1]=city},"Mines");n=n+1 end
            for _,b in ipairs(blds) do
                if not hasB(city,b) then task.wait(0.1);fire(gm.CreateBuilding,{[1]=city},b);n=n+1 end
            end
        end
        N("Full Dev","Queued "..n.." buildings",C.green); addLog("Full dev: "..n)
    end, "green")
end

-- ─── UNITS ────────────────────────────────────────────────────
do
    local c = addTab("⚔️","Units")
    section(c, "Unit Builder")
    slider(c, "Count", 1, 500, 10, function(v) S.ubAmt=v end)
    slider(c, "Delay (ms)", 50, 2000, 120, function(v) S.ubDelay=v/1000 end)
    dropdown(c, "Unit Type", UNIT_TYPES, "Infantry", function(v) S.ubType=v end)
    button(c, "⚔️  Mass Build Units", function()
        local req=UNIT_REQ[S.ubType]; local cc={}
        for _,city in ipairs(getCities()) do
            if req then if hasB(city,req) then cc[#cc+1]=city end else cc[#cc+1]=city end
        end
        if #cc==0 then N("Units","No valid cities for "..S.ubType,C.red); return end
        local built=0
        for i=1,S.ubAmt do
            fire(gm.CreateUnit,{[1]=cc[((i-1)%#cc)+1]},S.ubType); built=built+1; task.wait(S.ubDelay)
        end
        N("Units","Built "..built.." × "..S.ubType,C.green); addLog("Units: "..built.." "..S.ubType)
    end, "green")

    section(c, "Balanced Armies")
    slider(c, "Per Unit Type", 1, 100, 5, function(v) S.balCount=v end)
    button(c, "🪖  Balanced Land Army", function()
        local types={"Infantry","Tank","Anti Aircraft","Artillery"}; local cc=getCities()
        if #cc==0 then return end; local n=0
        for _,t in ipairs(types) do for i=1,S.balCount do fire(gm.CreateUnit,{[1]=cc[((i-1)%#cc)+1]},t); n=n+1; task.wait(0.1) end end
        N("Land Army","Queued "..n.." units",C.green)
    end, "green")
    button(c, "✈️  Balanced Air Force", function()
        local types={"Fighter","Attacker","Bomber"}; local airC={}
        for _,city in ipairs(getCities()) do if hasB(city,"Airport") then airC[#airC+1]=city end end
        if #airC==0 then N("Air","No airports!",C.red); return end; local n=0
        for _,t in ipairs(types) do for i=1,S.balCount do fire(gm.CreateUnit,{[1]=airC[((i-1)%#airC)+1]},t); n=n+1; task.wait(0.1) end end
        N("Air Force","Queued "..n.." aircraft",C.green)
    end, "green")
    button(c, "⚓  Full Naval Fleet", function()
        local types={"Destroyer","Frigate","Battleship","Aircraft Carrier","Submarine"}; local portC={}
        for _,city in ipairs(getCities()) do if hasB(city,"Port") then portC[#portC+1]=city end end
        if #portC==0 then N("Navy","No ports!",C.red); return end; local n=0
        for _,t in ipairs(types) do for i=1,S.balCount do fire(gm.CreateUnit,{[1]=portC[((i-1)%#portC)+1]},t); n=n+1; task.wait(0.1) end end
        N("Navy","Queued "..n.." ships",C.green)
    end, "green")
end

-- ─── MILITARY ─────────────────────────────────────────────────
do
    local c = addTab("🛡","Military")

    section(c, "Auto Declare War")
    infoCard(c, "Automatically declares war on countries with a Conquest justification that still have cities.")
    toggle(c, "Auto Declare Conquest War", false, function(v)
        S.autoDeclare=v; N("War",v and "Auto War ON" or "OFF",v and C.red or C.textDim)
    end)

    section(c, "Auto Assign Troops")
    infoCard(c, "Groups all Tanks + Infantry into 'Army' and assigns a Military Leader every 25s.")
    local _,setAssignSt = statusCard(c, "Status: Inactive.")
    toggle(c, "Enable Auto Assign Troops", false, function(v)
        S.autoAssign=v
        setAssignSt(v and "Active — grouping units." or "Inactive.", v and C.green or nil)
        N("Troops",v and "Auto Assign ON" or "OFF",v and C.green or C.textDim)
    end)

    section(c, "Auto Bomber Attack")
    infoCard(c, "Sends all Bombers to attack random enemy cities every 20s.")
    local _,setBomberSt = statusCard(c, "Status: Inactive.")
    toggle(c, "Enable Auto Bomber Attack", false, function(v)
        S.autoBomber=v
        setBomberSt(v and "Active — bombers attacking." or "Inactive.", v and C.yellow or nil)
        N("Bomber",v and "Bombers attacking!" or "OFF",v and C.yellow or C.textDim)
    end)

    section(c, "Anti Bomber Spam")
    infoCard(c, "Scrambles your Fighters to intercept and destroy incoming enemy Bombers.")
    local _,setAntiBSt = statusCard(c, "Status: Inactive.")
    toggle(c, "Enable Anti Bomber Spam", false, function(v)
        S.antiBomber=v
        setAntiBSt(v and "Active — intercepting bombers." or "Inactive.", v and C.accent or nil)
        N("Defense",v and "Anti Bomber ON" or "OFF",v and C.accent or C.textDim)
    end)

    section(c, "Auto Aircraft Creation")
    slider(c, "Target Attackers", 0, 50, 0, function(v) S.tgtAttacker=v end)
    slider(c, "Target Fighters",  0, 50, 0, function(v) S.tgtFighter=v  end)
    slider(c, "Target Bombers",   0, 50, 1, function(v) S.tgtBomber=v   end)
    local _,setAirSt = statusCard(c, "Status: Inactive.")
    toggle(c, "Enable Auto Aircraft Creation", false, function(v)
        S.autoAircraft=v; N("Aircraft",v and "ON" or "OFF",v and C.accent or C.textDim)
    end)

    section(c, "Auto Tank Spawner + Attack")
    slider(c, "Spawn Count",    1, 50,  5,  function(v) S.tankSpawnAmt=v end)
    slider(c, "Spawn Interval", 10,300, 30, function(v) S.tankSpawnInt=v end)
    slider(c, "Attack Interval",5, 120, 15, function(v) S.tankAtkInt=v   end)
    local _,setTankSt = statusCard(c, "Status: Inactive.")
    toggle(c, "Auto Spawn Tanks", false, function(v) S.autoTankSpawn=v; N("Tanks",v and "Spawner ON" or "OFF",v and C.yellow or C.textDim) end)
    toggle(c, "Auto Tank Attack", false, function(v)
        S.autoTankAtk=v
        setTankSt(v and "Active — tanks capturing." or "Inactive.", v and C.orange or nil)
        N("Tanks",v and "Attack ON" or "OFF",v and C.orange or C.textDim)
    end)

    section(c, "Auto Infantry Spawner + Attack")
    slider(c, "Spawn Count",    1, 100, 5,  function(v) S.infSpawnAmt=v end)
    slider(c, "Spawn Interval", 10,300, 30, function(v) S.infSpawnInt=v end)
    slider(c, "Attack Interval",5, 120, 15, function(v) S.infAtkInt=v   end)
    local _,setInfSt = statusCard(c, "Status: Inactive.")
    toggle(c, "Auto Spawn Infantry", false, function(v) S.autoInfSpawn=v; N("Infantry",v and "Spawner ON" or "OFF",v and C.yellow or C.textDim) end)
    toggle(c, "Auto Infantry Attack", false, function(v)
        S.autoInfAtk=v
        setInfSt(v and "Active — infantry pushing lines." or "Inactive.", v and C.orange or nil)
        N("Infantry",v and "Attack ON" or "OFF",v and C.orange or C.textDim)
    end)

    section(c, "War Crime")
    infoCard(c, "Automatically burns every city you capture from a country you are at war with.")
    local _,setWCSt = statusCard(c, "Status: Inactive.")
    toggle(c, "Enable War Crime (Burn Captured Cities)", false, function(v)
        S.warCrime=v
        setWCSt(v and "ACTIVE — burning all captures!" or "Inactive.", v and C.red or nil)
        N("War Crime",v and "ENABLED — burn all!" or "Disabled",v and C.red or C.textDim)
    end)
    button(c, "🚀  Launch Nuke (at War Target)", function()
        if S.warTarget then
            pcall(function() gm.LaunchNuke:FireServer(S.warTarget) end)
            N("☢️ Nuke","Launched at "..S.warTarget,C.red); addLog("Nuke → "..S.warTarget)
        else N("Nuke","Set War Target in Diplomacy first",C.red) end
    end, "red")
end

-- ─── DIPLOMACY ────────────────────────────────────────────────
do
    local c = addTab("🤝","Diplomacy")
    section(c, "Target Country")
    input(c, "Country Name", "e.g. Germany", function(v) S.selCountry = v ~= "" and v or nil end)

    section(c, "Alliance")
    button(c, "🤝  Send Alliance Request", function()
        if not S.selCountry then N("Diplo","Enter a country",C.red); return end
        S.allyT[S.selCountry]=true; fire(gm.ManageAlliance,S.selCountry,"SendRequest")
        N("Alliance","Request → "..S.selCountry,C.green); addLog("Alliance → "..S.selCountry)
    end, "green")
    button(c, "🌐  Ally ALL Players", function()
        local n=0
        for _,country in ipairs(allCountries()) do
            S.allyT[country]=true; fire(gm.ManageAlliance,country,"SendRequest"); n=n+1; task.wait(0.4)
        end
        N("Alliance","Sent to "..n.." countries",C.green); addLog("Allied all: "..n)
    end, "green")
    toggle(c, "Auto Alliance (every 2 min)", false, function(v) S.autoAlly=v; N("Auto","Alliance: "..(v and "ON" or "OFF")) end)

    section(c, "Puppet")
    button(c, "🎭  Send Puppet Request", function()
        if not S.selCountry then N("Puppet","Enter a country",C.red); return end
        S.puppetT[S.selCountry]=true; pcall(function() gm.ManagePuppet:FireServer(S.selCountry,"SendRequest") end)
        N("Puppet","Request → "..S.selCountry,C.yellow); addLog("Puppet → "..S.selCountry)
    end, "orange")
    button(c, "🌐  Puppet ALL Players", function()
        local n=0
        for _,country in ipairs(allCountries()) do
            S.puppetT[country]=true; pcall(function() gm.ManagePuppet:FireServer(country,"SendRequest") end); n=n+1; task.wait(0.5)
        end
        N("Puppet","Sent to "..n.." countries",C.yellow)
    end, "orange")
    toggle(c, "Auto Puppet (every 3 min)", false, function(v) S.autoPuppet=v; N("Auto","Puppet: "..(v and "ON" or "OFF")) end)

    section(c, "Justifications")
    dropdown(c, "Justification Type", {"Conquest","Liberate","Puppet","Annex"}, "Conquest", function(v) S.justifType=v end)
    button(c, "🎯  Justify on Selected Country", function()
        if not S.selCountry then N("Justify","Enter a country",C.red); return end
        pcall(function() gm.AddJustification:FireServer(S.selCountry,S.justifType) end)
        N("Justify",S.justifType.." on "..S.selCountry,C.accent)
    end)
    toggle(c, "Spam Justifications on All Players", false, function(v) S.spamJustif=v; N("Justif",v and "Spam ON" or "OFF") end)

    section(c, "War")
    input(c, "War Target", "country to declare war on", function(v) S.warTarget = v ~= "" and v or nil end)
    button(c, "⚔️  Declare War", function()
        if not S.warTarget then N("War","Set a war target",C.red); return end
        pcall(function() gm.ManageWar:FireServer(S.warTarget,"Declare") end)
        N("War","Declared on "..S.warTarget,C.red); addLog("War → "..S.warTarget)
    end, "red")
    button(c, "🕊️  Request Peace", function()
        if not S.warTarget then N("War","Set a war target",C.red); return end
        pcall(function() gm.ManageWar:FireServer(S.warTarget,"Peace") end)
        N("Peace","Sent to "..S.warTarget,C.green)
    end, "green")
    button(c, "💣  Declare War on EVERYONE", function()
        local n=0
        for _,country in ipairs(allCountries()) do
            pcall(function() gm.ManageWar:FireServer(country,"Declare") end); n=n+1; task.wait(0.3)
        end
        N("WAR","Declared on "..n.." countries",C.red); addLog("War on all: "..n)
    end, "red")

    gm.AlertPopup.OnClientEvent:Connect(function(a, b)
        if type(b) ~= "string" then return end
        if a == "Alliance declined" then
            local country = b:gsub(" has refused to join our alliance!","")
            if country ~= b and S.allyT[country] then
                task.delay(60, function() if S.allyT[country] then fire(gm.ManageAlliance,country,"SendRequest") end end)
            end
        elseif a == "Alliance Accepted" then
            local country = b:gsub(" has accepted our offer of an alliance%.","")
            if country ~= b then S.allyT[country]=nil; N("Alliance ✓",country.." accepted!",C.green) end
        elseif a == "Puppet declined" then
            local country = b:gsub(" has refused to become our puppet%.","")
            if country ~= b and S.puppetT[country] then
                task.delay(90, function() if S.puppetT[country] then pcall(function() gm.ManagePuppet:FireServer(country,"SendRequest") end) end end)
            end
        elseif a == "Puppet Accepted" then
            local country = b:gsub(" is now your puppet%.","")
            if country ~= b then S.puppetT[country]=nil; N("Puppet ✓",country.." is your puppet!",C.yellow) end
        end
    end)
end

-- ─── TRADING ──────────────────────────────────────────────────
do
    local c = addTab("📈","Trading")
    section(c, "Sell Resources")
    dropdown(c, "Resource to Sell", ALL_RES, "Oil",      function(v) S.sellRes=v end)
    slider(c,   "Sell Amount", 100, 50000, 1000, function(v) S.sellAmt=v end)
    toggle(c,   "AI Market Only", true, function(v) S.aiOnly=v end)
    button(c,   "💸  Sell Now", function()
        local mode = S.aiOnly and "AI" or "All"
        pcall(function() gm.TradeResource:FireServer(S.sellRes,-S.sellAmt,mode) end)
        N("Sell",fmt(S.sellAmt).." × "..S.sellRes,C.orange)
    end, "orange")
    toggle(c, "Auto Sell Resources", false, function(v) S.autoSell=v; N("Auto","Sell: "..(v and "ON" or "OFF")) end)

    section(c, "Buy Resources")
    dropdown(c, "Resource to Buy", ALL_RES, "Titanium", function(v) S.buyRes=v end)
    slider(c,   "Buy Amount", 100, 50000, 1000, function(v) S.buyAmt=v end)
    button(c,   "💰  Buy Now", function()
        pcall(function() gm.TradeResource:FireServer(S.buyRes,S.buyAmt,"All") end)
        N("Buy",fmt(S.buyAmt).." × "..S.buyRes,C.green)
    end, "green")
    dropdown(c, "Auto Buy Resource", ALL_RES, "Titanium", function(v) S.autoBuyType=v end)
    toggle(c, "Auto Buy (AI Market)", false, function(v) S.massBuyAI=v; N("Buy",v and "Auto Buy ON" or "OFF") end)

    section(c, "Consumer Goods")
    dropdown(c, "CG Target", {"AI Only","Player Only","Both"}, "Both", function(v) S.cgTarget=v end)
    toggle(c, "Auto Sell Consumer Goods", false, function(v) S.autoCGSell=v; N("CG",v and "Auto CG ON" or "OFF") end)

    section(c, "Quick Buy")
    button(c, "🔩  Mass Buy Titanium",    function() for i=1,10 do pcall(function() gm.TradeResource:FireServer("Titanium",5000,"All") end); task.wait(0.2) end; N("Buy","Mass bought Titanium",C.green) end, "green")
    button(c, "⚡  Mass Buy Electronics", function() for i=1,10 do pcall(function() gm.TradeResource:FireServer("Electronics",5000,"All") end); task.wait(0.2) end; N("Buy","Mass bought Electronics",C.green) end, "green")
    button(c, "🛢️  Mass Buy Oil",          function() for i=1,10 do pcall(function() gm.TradeResource:FireServer("Oil",5000,"All") end); task.wait(0.2) end; N("Buy","Mass bought Oil",C.green) end, "green")
    button(c, "🗑️  Dump All Surplus",      function()
        for _,r in ipairs({"Oil","Iron","Copper","Coal","Aluminum","Chromium","Phosphate"}) do
            pcall(function() gm.TradeResource:FireServer(r,-99999,"AI") end); task.wait(0.2)
        end
        N("Dump","Sold all surplus",C.orange)
    end, "orange")
end

-- ─── AUTO PLAY ────────────────────────────────────────────────
do
    local c = addTab("⚡","Auto Play")
    section(c, "Auto Farm")
    infoCard(c, "Phase 1: Builds RC + Factory + Arms Factory + Research Lab while selling Electronics/Consumer Goods.\n\nPhase 2 (auto-unlocks): Builds Airports, Ports, Nuclear Plants, maxes laws, researches all tech, mass-builds balanced army.\n\nExperimental — updates ongoing.")

    local farmStatusLbl = label(c, {Size=UDim2.new(1,0,0,16), Text="AutoFarm: Inactive", TextColor3=C.textDim, TextSize=11})
    toggle(c, "Enable Auto Farm", false, function(v)
        S.autoFarm=v
        if v then S.farmPhase=1; farmStatusLbl.Text="AutoFarm: Phase 1 — Building base…"; farmStatusLbl.TextColor3=C.yellow
        else S.farmPhase=0; farmStatusLbl.Text="AutoFarm: Inactive"; farmStatusLbl.TextColor3=C.textDim end
        N("AutoFarm",v and "STARTED ✓" or "Stopped",v and C.yellow or C.textDim)
    end)

    section(c, "Auto Timers")
    slider(c, "Mine Interval (s)",    10, 600, 60,  function(v) S.autoMineInt=v   end)
    slider(c, "Factory Interval (s)", 30, 600, 120, function(v) S.autoFactInt=v   end)
    slider(c, "Unit Interval (s)",    5,  300, 30,  function(v) S.autoUnitInt=v   end)
    slider(c, "Sell Interval (s)",    10, 300, 45,  function(v) S.autoSellInt=v   end)

    section(c, "Individual Toggles")
    toggle(c, "Auto Build Mines",      false, function(v) S.autoMine=v;    N("Auto","Mine: "..(v and "ON" or "OFF"))    end)
    toggle(c, "Auto Build Factories",  false, function(v) S.autoFactory=v; N("Auto","Factory: "..(v and "ON" or "OFF")) end)
    dropdown(c, "Auto Unit Type", UNIT_TYPES, "Infantry", function(v) S.autoUnitType=v end)
    toggle(c, "Auto Build Units",      false, function(v) S.autoUnit=v;    N("Auto","Units: "..(v and "ON" or "OFF"))   end)
    toggle(c, "Smart Auto Research",   false, function(v) S.smartResearch=v;N("Research",v and "Smart ON" or "OFF")     end)
    toggle(c, "Best Law Setup",        false, function(v) S.bestLaws=v;    N("Laws",v and "Best Laws ON" or "OFF")      end)

    section(c, "Controls")
    button(c, "⛔  STOP ALL AUTO TASKS", function()
        S.autoFarm=false; S.autoMine=false; S.autoUnit=false; S.autoFactory=false
        S.autoSell=false; S.autoAlly=false; S.autoPuppet=false
        S.autoTankSpawn=false; S.autoTankAtk=false; S.autoInfSpawn=false; S.autoInfAtk=false
        S.autoBomber=false; S.autoAircraft=false; S.autoDeclare=false; S.autoAssign=false
        S.antiBomber=false; S.warCrime=false; S.smartResearch=false; S.bestLaws=false; S.autoConst=false
        S.farmPhase=0; farmStatusLbl.Text="AutoFarm: Stopped"; farmStatusLbl.TextColor3=C.red
        _G.RAVEN_ON=false
        N("STOPPED","All auto tasks killed",C.red); addLog("ALL STOPPED")
    end, "red")
    button(c, "▶️  Restart Engine", function()
        _G.RAVEN_ON=true; N("Engine","Restarted",C.green); addLog("Engine restarted")
    end, "green")
end

-- ─── TECH & LAWS ──────────────────────────────────────────────
do
    local c = addTab("🔬","Tech & Laws")
    section(c, "Research")
    dropdown(c, "Tech Category", TECH_CATS, "Military", function(v) S.selTech=v end)
    button(c, "🔬  Research Selected", function()
        pcall(function() gm.ResearchTech:FireServer(S.selTech) end)
        N("Tech","Researching "..S.selTech,C.accent)
    end)
    button(c, "🚀  Research ALL Categories", function()
        for i, cat in ipairs(TECH_CATS) do
            task.delay(i*0.5, function() pcall(function() gm.ResearchTech:FireServer(cat) end) end)
        end
        N("Tech","Queued all "..#TECH_CATS.." categories",C.accent)
    end, "active")
    button(c, "🌐  Share Tech with ALL Players", function()
        local n=0
        for _,country in ipairs(allCountries()) do
            pcall(function() gm.ShareTech:FireServer(country) end); n=n+1; task.wait(0.3)
        end
        N("Tech","Shared with "..n.." countries",C.green); addLog("Tech shared: "..n)
    end, "green")

    section(c, "Laws")
    dropdown(c, "Tax Level", {"Low","Medium","High","Maximum"}, "Maximum", function(v) S.selTax=v end)
    button(c, "💰  Set Tax Law", function()
        pcall(function() gm.SetTaxLaw:FireServer(S.selTax) end); N("Tax","Set to "..S.selTax,C.yellow)
    end)
    dropdown(c, "Conscription", {"Volunteer","Selective","Universal","Total War"}, "Total War", function(v) S.selCons=v end)
    button(c, "🪖  Set Conscription Law", function()
        pcall(function() gm.SetConscription:FireServer(S.selCons) end); N("Laws",S.selCons,C.yellow)
    end)
    button(c, "⚡  Max All Laws Instantly", function()
        for _,t in ipairs({"Low","Medium","High","Maximum"}) do
            pcall(function() gm.SetTaxLaw:FireServer(t) end); task.wait(0.2)
        end
        pcall(function() gm.SetConscription:FireServer("Total War") end)
        N("Laws","All laws maxed",C.yellow)
    end, "orange")
end

-- ─── SETTINGS ─────────────────────────────────────────────────
do
    local c = addTab("⚙️","Settings")
    section(c, "Alerts")
    local _, setAlertSt = statusCard(c, "Auto Clear Alerts: Inactive.")
    toggle(c, "Auto Clear Future Alerts  ✦ NEW", false, function(v)
        S.autoClearAlerts=v
        setAlertSt(v and "Active — all alerts auto-dismissed." or "Inactive.", v and C.accent or nil)
        N("Alerts",v and "Auto Clear ON" or "OFF",v and C.accent or C.textDim)
    end)
    button(c, "🔇  Clear All Alerts Right Now", function()
        local n=0
        for _, g in ipairs(pGui:GetChildren()) do
            local nm = g.Name:lower()
            if nm:find("alert") or nm:find("popup") or nm:find("notif") then
                pcall(function() g.Enabled=false end); n=n+1
            end
        end
        N("Alerts","Cleared "..n.." alert GUIs",C.accent); addLog("Cleared "..n.." alerts")
    end)

    section(c, "ESP")
    toggle(c, "Show Enemy Units", true, function(v) S.showEnemy=v end)
    toggle(c, "Enable Unit ESP", false, function(v)
        S.espOn=v
        if not v then
            for _, u in ipairs(workspace.Units:GetChildren()) do
                local t = u:FindFirstChild("RVN_ESP"); if t then t:Destroy() end
            end
        end
        N("ESP",v and "ON" or "OFF",v and C.green or C.textDim)
    end)

    section(c, "Info")
    button(c, "📋  Print Player Info", function()
        local ls = dig(player,"leaderstats")
        print("╔═══ RAVEN HUB ═══════════════╗")
        print("║  "..player.Name.."  |  "..(getCountry() or "?"))
        if ls then for _,s in ipairs(ls:GetChildren()) do
            print(("║  %-14s %s"):format(s.Name..":",tostring(s.Value)))
        end end
        print("║  Cities: "..#getCities())
        print("╚══════════════════════════════╝")
        N("Info","Printed to F9",C.accent)
    end)
    button(c, "🌍  List All Countries", function()
        print("╔═══ IN-GAME COUNTRIES ════════╗")
        for _,p in ipairs(Players:GetPlayers()) do
            local ls=dig(p,"leaderstats")
            local co=ls and dig(ls,"Country") and dig(ls,"Country").Value or "?"
            print(("║  %-20s %s"):format(p.Name,co))
        end
        print("╚══════════════════════════════╝")
        N("Countries","Printed",C.accent)
    end)

    section(c, "About")
    infoCard(c, "Raven Hub v9.0  •  100% custom UI\nNo external libraries. No loadstring. Fully self-contained.\n\nToggle: RightControl\nAll remotes are pcall-wrapped — one broken remote never crashes the script.")
end

-- ══════════════════════════════════════════════════════════════
--  ACTIVATE FIRST TAB
-- ══════════════════════════════════════════════════════════════
do
    local firstBtn = sidebar:FindFirstChildWhichIsA("TextButton")
    if firstBtn then firstBtn.MouseButton1Click:Fire() end
end

-- ══════════════════════════════════════════════════════════════
--  ESP SYSTEM
-- ══════════════════════════════════════════════════════════════
local function espTag(unit)
    if unit:FindFirstChild("RVN_ESP") then return end
    local ut = dig(unit,"Type"); local uc = dig(unit,"Current"); local ow = dig(unit,"Owner")
    if not (ut and uc and ow) then return end
    local isOwn = ow.Value == player.Name
    if not isOwn and not S.showEnemy then return end

    local bb = Instance.new("BillboardGui", unit)
    bb.Name = "RVN_ESP"; bb.AlwaysOnTop = true; bb.MaxDistance = 800
    bb.Size = UDim2.new(0,140,0,42); bb.Adornee = unit

    local bg = frame(bb, {BackgroundColor3=Color3.fromRGB(8,8,18), BackgroundTransparency=0.18})
    bg.Size = UDim2.new(1,0,1,0)
    corner(bg, 7)

    local col = isOwn and C.green or C.red
    local bar = frame(bg, {Size=UDim2.new(0,3,1,0), BackgroundColor3=col})
    corner(bar, 2)

    local t1 = label(bg, {Position=UDim2.new(0,8,0,3),Size=UDim2.new(1,-10,0.55,0),Text=ut.Value,TextColor3=col,Font=FONT_BOLD,TextSize=11})
    local t2 = label(bg, {Position=UDim2.new(0,8,0.58,0),Size=UDim2.new(1,-10,0.38,0),Text=fmt(uc.Value).." troops",TextColor3=C.textMuted,Font=FONT_MONO,TextSize=9})

    -- update periodically
    task.spawn(function()
        while task.wait(5) do
            if not unit.Parent or not _G.RAVEN_ON then break end
            local cur = dig(unit,"Current")
            if cur and t2 then t2.Text = fmt(cur.Value).." troops" end
        end
    end)
end

workspace.Units.ChildAdded:Connect(function(child)
    if S.espOn then task.wait(0.5); espTag(child) end
end)

task.spawn(function()
    while task.wait(12) do
        if not _G.RAVEN_ON then break end
        if S.espOn then
            for _,u in ipairs(workspace.Units:GetChildren()) do espTag(u) end
        end
    end
end)

-- ══════════════════════════════════════════════════════════════
--  AUTO LOOPS
-- ══════════════════════════════════════════════════════════════

-- Auto clear alerts
task.spawn(function()
    while task.wait(3) do
        if not _G.RAVEN_ON then break end
        if S.autoClearAlerts then
            for _, g in ipairs(pGui:GetChildren()) do
                local nm = g.Name:lower()
                if nm:find("alert") or nm:find("popup") or nm:find("notif") then
                    pcall(function() g.Enabled = false end)
                end
            end
        end
    end
end)
pGui.ChildAdded:Connect(function(g)
    if S.autoClearAlerts then
        local nm = g.Name:lower()
        if nm:find("alert") or nm:find("popup") or nm:find("notif") then
            task.wait(0.15); pcall(function() g.Enabled = false end)
        end
    end
end)

-- Auto mine
task.spawn(function()
    while task.wait(S.autoMineInt) do
        if not _G.RAVEN_ON then break end
        if S.autoMine or (S.autoFarm and S.farmPhase >= 1) then
            for _,city in ipairs(getCities()) do
                if not hasB(city,"Mine") then
                    local res,ok=getRes(city),false
                    for ore in pairs(ORE_SET) do if (res[ore] or 0)>=1 then ok=true;break end end
                    if ok then fire(gm.CreateBuilding,{[1]=city},"Mines"); task.wait(0.2) end
                end
            end
        end
    end
end)

-- Auto construction
task.spawn(function()
    while task.wait(90) do
        if not _G.RAVEN_ON then break end
        if S.autoConst then
            local eligible={}
            for _,city in ipairs(getCities()) do
                if not hasB(city,S.autoConstFactory) then eligible[#eligible+1]=city end
            end
            for i=#eligible,2,-1 do
                local j=math.random(1,i); eligible[i],eligible[j]=eligible[j],eligible[i]
            end
            for i=1,math.min(S.autoConstQty,#eligible) do
                fire(gm.CreateBuilding,{[1]=eligible[i]},S.autoConstFactory); task.wait(0.15)
            end
        end
    end
end)

-- Auto factory + autofarm phase
task.spawn(function()
    while task.wait(S.autoFactInt) do
        if not _G.RAVEN_ON then break end
        if S.autoFactory or (S.autoFarm and S.farmPhase >= 1) then
            local base   = {"Recruitment Center","Factory","Arms Factory","Research Lab"}
            local expand = {"Recruitment Center","Factory","Arms Factory","Research Lab","Airport","Port","Nuclear Plant"}
            local targets = (S.autoFarm and S.farmPhase >= 2) and expand or base
            local allDone = true
            for _,city in ipairs(getCities()) do
                for _,b in ipairs(targets) do
                    if not hasB(city,b) then allDone=false; fire(gm.CreateBuilding,{[1]=city},b); task.wait(0.15) end
                end
            end
            if S.autoFarm and S.farmPhase == 1 and allDone then
                S.farmPhase = 2
                N("AutoFarm 🚀","Phase 2 unlocked — Economy Expansion!",C.green)
                addLog("AutoFarm → Phase 2")
                task.spawn(function()
                    for _,t in ipairs({"Low","Medium","High","Maximum"}) do
                        pcall(function() gm.SetTaxLaw:FireServer(t) end); task.wait(0.3)
                    end
                    pcall(function() gm.SetConscription:FireServer("Total War") end)
                    for i,cat in ipairs(TECH_CATS) do
                        task.delay(i*0.5, function() pcall(function() gm.ResearchTech:FireServer(cat) end) end)
                    end
                end)
            end
        end
    end
end)

-- Auto unit
task.spawn(function()
    while task.wait(S.autoUnitInt) do
        if not _G.RAVEN_ON then break end
        if S.autoUnit then
            local req=UNIT_REQ[S.autoUnitType]; local cc={}
            for _,city in ipairs(getCities()) do
                if req then if hasB(city,req) then cc[#cc+1]=city end else cc[#cc+1]=city end
            end
            if #cc>0 then fire(gm.CreateUnit,{[1]=cc[math.random(1,#cc)]},S.autoUnitType) end
        end
        if S.autoFarm and S.farmPhase == 2 then
            local cc=getCities()
            if #cc>0 then
                local land={"Infantry","Tank","Artillery","Anti Aircraft"}
                fire(gm.CreateUnit,{[1]=cc[math.random(1,#cc)]},land[math.random(1,#land)])
            end
        end
    end
end)

-- Auto sell + CG + buy
task.spawn(function()
    while task.wait(S.autoSellInt) do
        if not _G.RAVEN_ON then break end
        if S.autoSell then
            local mode = S.aiOnly and "AI" or "All"
            pcall(function() gm.TradeResource:FireServer(S.sellRes,-S.sellAmt,mode) end)
        end
        if S.autoFarm and S.farmPhase >= 1 then
            pcall(function() gm.TradeResource:FireServer("Electronics",-5000,"AI") end)
            task.wait(0.3)
            pcall(function() gm.TradeResource:FireServer("Consumer Goods",-5000,"AI") end)
        end
        if S.autoCGSell then
            local m = S.cgTarget=="AI Only" and "AI" or S.cgTarget=="Player Only" and "Players" or "All"
            pcall(function() gm.TradeResource:FireServer("Consumer Goods",-1000,m) end)
        end
        if S.massBuyAI then
            pcall(function() gm.TradeResource:FireServer(S.autoBuyType,5000,"AI") end)
        end
    end
end)

-- Auto ally
task.spawn(function()
    while task.wait(120) do
        if not _G.RAVEN_ON then break end
        if S.autoAlly then
            for _,country in ipairs(allCountries()) do
                if not S.allyT[country] then
                    S.allyT[country]=true; fire(gm.ManageAlliance,country,"SendRequest"); task.wait(0.8)
                end
            end
        end
    end
end)

-- Auto puppet
task.spawn(function()
    while task.wait(180) do
        if not _G.RAVEN_ON then break end
        if S.autoPuppet then
            for _,country in ipairs(allCountries()) do
                if not S.puppetT[country] then
                    S.puppetT[country]=true
                    pcall(function() gm.ManagePuppet:FireServer(country,"SendRequest") end); task.wait(1)
                end
            end
        end
    end
end)

-- Auto declare war
task.spawn(function()
    while task.wait(30) do
        if not _G.RAVEN_ON then break end
        if S.autoDeclare then
            local base = dig(workspace,"Baseplate","Cities"); if not base then task.wait(5) end
            if base then
                for _,folder in ipairs(base:GetChildren()) do
                    if folder.Name ~= getCountry() and #folder:GetChildren() > 0 then
                        pcall(function() gm.ManageWar:FireServer(folder.Name,"Declare") end); task.wait(0.5)
                    end
                end
            end
        end
    end
end)

-- Auto bomber
task.spawn(function()
    while task.wait(20) do
        if not _G.RAVEN_ON then break end
        if S.autoBomber then
            local ec = enemyCities()
            if #ec > 0 then
                for _,u in ipairs(myUnits()) do
                    local ut = dig(u,"Type")
                    if ut and ut.Value=="Bomber" then
                        local t = ec[math.random(1,#ec)]; local tp = t:FindFirstChildWhichIsA("BasePart")
                        if tp then pcall(function() gm.MoveUnit:FireServer(u,tp.Position,"Attack") end) end
                    end
                end
            end
        end
    end
end)

-- Anti bomber
task.spawn(function()
    while task.wait(10) do
        if not _G.RAVEN_ON then break end
        if S.antiBomber then
            for _,u in ipairs(workspace.Units:GetChildren()) do
                local ut = dig(u,"Type"); local ow = dig(u,"Owner")
                if ut and ow and ut.Value=="Bomber" and ow.Value~=player.Name then
                    for _,mine in ipairs(myUnits()) do
                        local mt = dig(mine,"Type")
                        if mt and mt.Value=="Fighter" then
                            local bp = u:FindFirstChildWhichIsA("BasePart")
                            if bp then pcall(function() gm.MoveUnit:FireServer(mine,bp.Position,"Attack") end); break end
                        end
                    end
                end
            end
        end
    end
end)

-- Auto aircraft
task.spawn(function()
    while task.wait(20) do
        if not _G.RAVEN_ON then break end
        if S.autoAircraft then
            local airC={}
            for _,c in ipairs(getCities()) do if hasB(c,"Airport") then airC[#airC+1]=c end end
            if #airC>0 then
                local counts={Attacker=0,Fighter=0,Bomber=0}
                for _,u in ipairs(myUnits()) do
                    local ut=dig(u,"Type")
                    if ut and counts[ut.Value]~=nil then counts[ut.Value]=counts[ut.Value]+1 end
                end
                local city=airC[math.random(1,#airC)]
                if counts.Attacker<S.tgtAttacker then fire(gm.CreateUnit,{[1]=city},"Attacker"); task.wait(0.2) end
                if counts.Fighter <S.tgtFighter  then fire(gm.CreateUnit,{[1]=city},"Fighter");  task.wait(0.2) end
                if counts.Bomber  <S.tgtBomber   then fire(gm.CreateUnit,{[1]=city},"Bomber");   task.wait(0.2) end
            end
        end
    end
end)

-- Auto assign troops
task.spawn(function()
    while task.wait(25) do
        if not _G.RAVEN_ON then break end
        if S.autoAssign then
            local army={}
            for _,u in ipairs(myUnits()) do
                local ut=dig(u,"Type")
                if ut and (ut.Value=="Tank" or ut.Value=="Infantry") then army[#army+1]=u end
            end
            if #army >= 2 then pcall(function() gm.GroupUnits:FireServer(army,"Army") end); addLog("Assigned "..#army.." to Army") end
        end
    end
end)

-- War crime
local capConn = nil
task.spawn(function()
    while task.wait(2) do
        if not _G.RAVEN_ON then break end
        if S.warCrime then
            local cf = dig(workspace,"Baseplate","Cities",getCountry() or "__none__")
            if cf and not capConn then
                capConn = cf.ChildAdded:Connect(function(nc)
                    if S.warCrime then
                        task.wait(2); pcall(function() gm.BurnCity:FireServer(nc) end)
                        N("War Crime","Burned "..nc.Name,C.red); addLog("WAR CRIME: burned "..nc.Name)
                    end
                end)
            end
        else
            if capConn then capConn:Disconnect(); capConn=nil end
        end
    end
end)

-- Auto tank spawn
task.spawn(function()
    while task.wait(S.tankSpawnInt) do
        if not _G.RAVEN_ON then break end
        if S.autoTankSpawn then
            local cc=getCities()
            if #cc>0 then
                for i=1,S.tankSpawnAmt do fire(gm.CreateUnit,{[1]=cc[((i-1)%#cc)+1]},"Tank"); task.wait(0.1) end
            end
        end
    end
end)

-- Auto tank attack
task.spawn(function()
    while task.wait(S.tankAtkInt) do
        if not _G.RAVEN_ON then break end
        if S.autoTankAtk then
            local ec=enemyCities()
            if #ec>0 then
                for _,u in ipairs(myUnits()) do
                    local ut=dig(u,"Type")
                    if ut and ut.Value=="Tank" then
                        local t=ec[math.random(1,#ec)]; local tp=t:FindFirstChildWhichIsA("BasePart")
                        if tp then pcall(function() gm.MoveUnit:FireServer(u,tp.Position,"Capture") end) end
                    end
                end
            end
        end
    end
end)

-- Auto infantry spawn
task.spawn(function()
    while task.wait(S.infSpawnInt) do
        if not _G.RAVEN_ON then break end
        if S.autoInfSpawn then
            local cc=getCities()
            if #cc>0 then
                for i=1,S.infSpawnAmt do fire(gm.CreateUnit,{[1]=cc[((i-1)%#cc)+1]},"Infantry"); task.wait(0.1) end
            end
        end
    end
end)

-- Auto infantry attack
task.spawn(function()
    while task.wait(S.infAtkInt) do
        if not _G.RAVEN_ON then break end
        if S.autoInfAtk then
            local ec=enemyCities()
            if #ec>0 then
                for _,u in ipairs(myUnits()) do
                    local ut=dig(u,"Type")
                    if ut and ut.Value=="Infantry" then
                        local t=ec[math.random(1,#ec)]; local tp=t:FindFirstChildWhichIsA("BasePart")
                        if tp then pcall(function() gm.MoveUnit:FireServer(u,tp.Position,"Capture") end) end
                    end
                end
            end
        end
    end
end)

-- Smart research
task.spawn(function()
    local rOrder={"Military","Industry","Economic","Air","Naval","Infrastructure","Recon","Nuclear"}
    local idx=1
    while task.wait(45) do
        if not _G.RAVEN_ON then break end
        if S.smartResearch then
            pcall(function() gm.ResearchTech:FireServer(rOrder[idx]) end)
            addLog("Smart Research: "..rOrder[idx])
            idx = (idx % #rOrder) + 1
        end
    end
end)

-- Spam justifications
task.spawn(function()
    while task.wait(60) do
        if not _G.RAVEN_ON then break end
        if S.spamJustif then
            for _,country in ipairs(allCountries()) do
                pcall(function() gm.AddJustification:FireServer(country,S.justifType) end); task.wait(0.5)
            end
        end
    end
end)

-- Best law setup
task.spawn(function()
    while task.wait(15) do
        if not _G.RAVEN_ON then break end
        if S.bestLaws then
            for _,l in ipairs({"Low","Medium","High","Maximum"}) do
                pcall(function() gm.SetTaxLaw:FireServer(l) end); task.wait(0.5)
            end
            pcall(function() gm.SetConscription:FireServer("Total War") end); task.wait(0.5)
            pcall(function() gm.SetPoliticalLaw:FireServer("Unique_Monarchy",2) end)
        end
    end
end)

-- ══════════════════════════════════════════════════════════════
--  DONE
-- ══════════════════════════════════════════════════════════════
addLog("Raven Hub v9.0 loaded — "..(getCountry() or "No country yet"))
N("🦅  Raven Hub v9.0", #getCities().." cities  •  RCtrl to toggle", C.accent)
