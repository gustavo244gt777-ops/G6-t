if _G.EggMasterLoaded then
    pcall(function() _G.EggMasterGUI:Destroy() end)
    _G.EggMasterLoaded = nil
end
_G.EggMasterLoaded = true

local Players = game:GetService("Players")
local RS      = game:GetService("ReplicatedStorage")
local WS      = game:GetService("Workspace")
local VU      = game:GetService("VirtualUser")
local UIS     = game:GetService("UserInputService")
local LP      = Players.LocalPlayer

-- ═══════════════════════════════════════════════
-- CONFIG
-- ═══════════════════════════════════════════════
local C = {
    AutoSteal  = true,
    AutoFusion = true,
    AutoHatch  = true,
    AutoClaim  = true,
    AutoSell   = false,
    ESP        = true,
    AntiAFK    = true,
    TP         = true,
    Delay      = 0.35,
    MaxOvos    = 30,
}

local function log(...) print("[EGG MASTER]", ...) end

-- ═══════════════════════════════════════════════
-- DETECÇÃO AUTOMÁTICA DE REMOTES
-- ═══════════════════════════════════════════════
local R = { steal={}, fusion={}, hatch={}, claim={}, sell={} }

local function scanRemotes()
    R = { steal={}, fusion={}, hatch={}, claim={}, sell={} }
    for _, o in pairs(RS:GetDescendants()) do
        if o:IsA("RemoteEvent") or o:IsA("RemoteFunction") or o.ClassName == "UnreliableRemoteEvent" then
            local n = o.Name:lower()
            if n:find("steal") or n:find("grab") or n:find("pick") or n:find("take") or n:find("collect") then
                table.insert(R.steal, o)
            elseif n:find("fus") or n:find("merge") or n:find("combine") then
                table.insert(R.fusion, o)
            elseif n:find("hatch") or n:find("open") or n:find("unbox") then
                table.insert(R.hatch, o)
            elseif n:find("claim") or n:find("reward") or n:find("event") then
                table.insert(R.claim, o)
            elseif n:find("sell") or n:find("drop") then
                table.insert(R.sell, o)
            end
        end
    end
    log(("Remotes → steal:%d fusion:%d hatch:%d claim:%d sell:%d"):format(
        #R.steal, #R.fusion, #R.hatch, #R.claim, #R.sell))
end
scanRemotes()
task.delay(8, scanRemotes) -- re-scan depois que o jogo carregar tudo

-- ═══════════════════════════════════════════════
-- UTILITÁRIOS
-- ═══════════════════════════════════════════════
local function hrp()
    local c = LP.Character
    return c and c:FindFirstChild("HumanoidRootPart")
end

local function isEgg(o)
    if not (o:IsA("Model") or o:IsA("BasePart")) then return false end
    local n = o.Name:lower()
    return n:find("egg") or n:find("ovo") or n:find("pet") or o:FindFirstChild("Rarity") ~= nil
end

local function getEggs()
    local t = {}
    for _, o in pairs(WS:GetDescendants()) do
        if isEgg(o) then table.insert(t, o) end
    end
    return t
end

local function eggPos(e)
    if e.PrimaryPart then return e.PrimaryPart.Position end
    if e:IsA("BasePart") then return e.Position end
    local p = e:FindFirstChildWhichIsA("BasePart")
    return p and p.Position
end

local function countOwned()
    local n = 0
    local c = LP.Character
    if c then
        for _, o in pairs(c:GetChildren()) do
            if o:IsA("Tool") then n = n + 1 end
        end
    end
    local bp = LP:FindFirstChildOfClass("Backpack")
    if bp then
        for _, o in pairs(bp:GetChildren()) do
            if o:IsA("Tool") then n = n + 1 end
        end
    end
    return n
end

local function fire(r, ...)
    pcall(function()
        if r:IsA("RemoteEvent") or r.ClassName == "UnreliableRemoteEvent" then
            r:FireServer(...)
        else
            r:InvokeServer(...)
        end
    end)
end

-- ═══════════════════════════════════════════════
-- PEGAR OVO (3 métodos em sequência)
-- ═══════════════════════════════════════════════
local function grab(egg)
    local pos = eggPos(egg)
    if not pos then return end
    local h = hrp()
    if not h then return end

    if C.TP then
        h.CFrame = CFrame.new(pos + Vector3.new(0, 3, 0))
    end

    -- método 1: ProximityPrompt
    local pp = egg:FindFirstChildWhichIsA("ProximityPrompt", true)
    if pp then
        pcall(function()
            if fireproximityprompt then
                fireproximityprompt(pp)
            else
                pp:InputHoldBegin()
                task.wait(0.05)
                pp:InputHoldEnd()
            end
        end)
    end

    -- método 2: ClickDetector
    local cd = egg:FindFirstChildWhichIsA("ClickDetector", true)
    if cd then
        pcall(function() fireclickdetector(cd) end)
    end

    -- método 3: remotes genéricos
    for _, r in pairs(R.steal) do fire(r, egg) end
end

-- ═══════════════════════════════════════════════
-- LOOP: AUTO STEAL
-- ═══════════════════════════════════════════════
task.spawn(function()
    while _G.EggMasterLoaded and C.AutoSteal do
        pcall(function()
            if C.MaxOvos > 0 and countOwned() >= C.MaxOvos then
                if C.AutoSell then
                    for _, r in pairs(R.sell) do fire(r) end
                    task.wait(2)
                end
            else
                local eggs = getEggs()
                if #eggs > 0 then
                    local h = hrp()
                    if h then
                        table.sort(eggs, function(a, b)
                            local pa, pb = eggPos(a), eggPos(b)
                            if not pa or not pb then return false end
                            return (pa - h.Position).Magnitude < (pb - h.Position).Magnitude
                        end)
                    end
                    grab(eggs[1])
                end
            end
        end)
        task.wait(C.Delay)
    end
end)

-- ═══════════════════════════════════════════════
-- LOOP: AUTO FUSION
-- ═══════════════════════════════════════════════
task.spawn(function()
    while _G.EggMasterLoaded and C.AutoFusion do
        pcall(function()
            local list = {}
            local c = LP.Character
            if c then
                for _, o in pairs(c:GetChildren()) do
                    if o:IsA("Tool") then table.insert(list, o) end
                end
            end
            local bp = LP:FindFirstChildOfClass("Backpack")
            if bp then
                for _, o in pairs(bp:GetChildren()) do
                    if o:IsA("Tool") then table.insert(list, o) end
                end
            end
            if #list >= 2 and #R.fusion > 0 then
                for _, r in pairs(R.fusion) do
                    fire(r, list[1], list[2])
                end
            end
        end)
        task.wait(1.5)
    end
end)

-- ═══════════════════════════════════════════════
-- LOOP: AUTO HATCH + CLAIM
-- ═══════════════════════════════════════════════
task.spawn(function()
    while _G.EggMasterLoaded do
        pcall(function()
            if C.AutoHatch then
                for _, r in pairs(R.hatch) do fire(r) end
            end
            if C.AutoClaim then
                for _, r in pairs(R.claim) do fire(r) end
            end
        end)
        task.wait(3)
    end
end)

-- ═══════════════════════════════════════════════
-- LOOP: ESP
-- ═══════════════════════════════════════════════
task.spawn(function()
    while _G.EggMasterLoaded and C.ESP do
        pcall(function()
            for _, e in pairs(getEggs()) do
                local p = e.PrimaryPart or (e:IsA("BasePart") and e)
                if p and not p:FindFirstChild("EggESP") then
                    local bb = Instance.new("BillboardGui")
                    bb.Name = "EggESP"
                    bb.AlwaysOnTop = true
                    bb.Size = UDim2.new(0, 100, 0, 24)
                    bb.StudsOffset = Vector3.new(0, 3, 0)
                    bb.Parent = p
                    local lbl = Instance.new("TextLabel")
                    lbl.Size = UDim2.new(1, 0, 1, 0)
                    lbl.BackgroundTransparency = 1
                    lbl.Text = "🥚 " .. e.Name
                    lbl.TextColor3 = Color3.fromRGB(0, 255, 150)
                    lbl.TextStrokeTransparency = 0
                    lbl.TextScaled = true
                    lbl.Font = Enum.Font.GothamBold
                    lbl.Parent = bb
                end
            end
        end)
        task.wait(3)
    end
end)

-- ═══════════════════════════════════════════════
-- ANTI-AFK
-- ═══════════════════════════════════════════════
LP.Idled:Connect(function()
    if C.AntiAFK then
        VU:CaptureController()
        VU:ClickButton2(Vector2.new())
    end
end)

-- ═══════════════════════════════════════════════
-- GUI MOBILE (minimizar / fechar / arrastar)
-- ═══════════════════════════════════════════════
local pg = (gethui and gethui()) or LP:WaitForChild("PlayerGui")
local gui = Instance.new("ScreenGui")
gui.Name = "EggMasterGUI"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.DisplayOrder = 999999
gui.Parent = pg
_G.EggMasterGUI = gui

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 240, 0, 340)
frame.Position = UDim2.new(0, 10, 0, 80)
frame.BackgroundColor3 = Color3.fromRGB(20, 22, 32)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.ClipsDescendants = true
frame.Parent = gui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)
local st = Instance.new("UIStroke", frame)
st.Color = Color3.fromRGB(0, 200, 120)
st.Thickness = 1.5

-- Barra de título
local top = Instance.new("Frame")
top.Size = UDim2.new(1, 0, 0, 34)
top.BackgroundColor3 = Color3.fromRGB(0, 180, 110)
top.Parent = frame
Instance.new("UICorner", top).CornerRadius = UDim.new(0, 10)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -75, 1, 0)
title.Position = UDim2.new(0, 10, 0, 0)
title.BackgroundTransparency = 1
title.Text = "🥚 EGG MASTER v7"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 14
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = top

local minB = Instance.new("TextButton")
minB.Size = UDim2.new(0, 28, 0, 24)
minB.Position = UDim2.new(1, -62, 0, 5)
minB.BackgroundColor3 = Color3.fromRGB(255, 180, 0)
minB.Text = "—"
minB.TextColor3 = Color3.new(1,1,1)
minB.Font = Enum.Font.GothamBold
minB.TextSize = 16
minB.Parent = top
Instance.new("UICorner", minB).CornerRadius = UDim.new(0, 5)

local closeB = Instance.new("TextButton")
closeB.Size = UDim2.new(0, 28, 0, 24)
closeB.Position = UDim2.new(1, -32, 0, 5)
closeB.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
closeB.Text = "✕"
closeB.TextColor3 = Color3.new(1,1,1)
closeB.Font = Enum.Font.GothamBold
closeB.TextSize = 14
closeB.Parent = top
Instance.new("UICorner", closeB).CornerRadius = UDim.new(0, 5)

-- Conteúdo scroll
local content = Instance.new("ScrollingFrame")
content.Size = UDim2.new(1, -12, 1, -42)
content.Position = UDim2.new(0, 6, 0, 38)
content.BackgroundTransparency = 1
content.BorderSizePixel = 0
content.ScrollBarThickness = 4
content.ScrollBarImageColor3 = Color3.fromRGB(0, 180, 110)
content.CanvasSize = UDim2.new(0, 0, 0, 500)
content.Parent = frame
Instance.new("UIListLayout", content).Padding = UDim.new(0, 5)

-- Toggle helper
local function addToggle(label, key)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(1, 0, 0, 30)
    b.BackgroundColor3 = C[key] and Color3.fromRGB(0, 150, 80) or Color3.fromRGB(45, 48, 65)
    b.TextColor3 = Color3.new(1,1,1)
    b.Font = Enum.Font.GothamSemibold
    b.TextSize = 13
    b.Text = (C[key] and "✅ " or "❌ ") .. label
    b.Parent = content
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 6)
    b.MouseButton1Click:Connect(function()
        C[key] = not C[key]
        b.BackgroundColor3 = C[key] and Color3.fromRGB(0,150,80) or Color3.fromRGB(45,48,65)
        b.Text = (C[key] and "✅ " or "❌ ") .. label
    end)
end

addToggle("Auto Steal", "AutoSteal")
addToggle("Auto Fusão", "AutoFusion")
addToggle("Auto Hatch", "AutoHatch")
addToggle("Auto Recompensa", "AutoClaim")
addToggle("Auto Vender", "AutoSell")
addToggle("ESP Ovos", "ESP")
addToggle("Anti-AFK", "AntiAFK")
addToggle("Teleporte", "TP")

-- Contador
local counter = Instance.new("TextLabel")
counter.Size = UDim2.new(1, 0, 0, 36)
counter.BackgroundTransparency = 1
counter.TextColor3 = Color3.fromRGB(0, 255, 150)
counter.Font = Enum.Font.GothamBold
counter.TextSize = 13
counter.Text = "Inventário: 0 ovos"
counter.Parent = content

task.spawn(function()
    while _G.EggMasterLoaded and gui.Parent do
        counter.Text = "Inventário: " .. countOwned() .. " ovos"
        task.wait(1)
    end
end)

-- Minimizar
local minimized = false
local savedSize = frame.Size
local function toggleMin()
    minimized = not minimized
    if minimized then
        savedSize = frame.Size
        frame:TweenSize(UDim2.new(0, 240, 0, 34), "Out", "Quad", 0.15, true)
        content.Visible = false
        minB.Text = "▢"
    else
        frame:TweenSize(savedSize, "Out", "Quad", 0.15, true)
        content.Visible = true
        minB.Text = "—"
    end
end
minB.MouseButton1Click:Connect(toggleMin)

-- Fechar + botão flutuante
local reopen = Instance.new("TextButton")
reopen.Size = UDim2.new(0, 50, 0, 50)
reopen.Position = UDim2.new(0, 10, 0, 80)
reopen.BackgroundColor3 = Color3.fromRGB(0, 180, 110)
reopen.Text = "🥚"
reopen.TextSize = 24
reopen.TextColor3 = Color3.new(1,1,1)
reopen.Font = Enum.Font.GothamBold
reopen.Visible = false
reopen.Draggable = true
reopen.Parent = gui
Instance.new("UICorner", reopen).CornerRadius = UDim.new(0, 25)

closeB.MouseButton1Click:Connect(function()
    frame.Visible = false
    reopen.Visible = true
end)
reopen.MouseButton1Click:Connect(function()
    frame.Visible = true
    reopen.Visible = false
end)

log("✅ Carregado! Ovos no mapa: " .. #getEggs())
log("Se algum botão não pegar ovo, mande print do F9 que eu ajusto o nome do remote.")
