local gui = Instance.new("ScreenGui")
gui.Name = "EggGUI"
gui.ResetOnSpawn = false
gui.Parent = (gethui and gethui()) or LP:WaitForChild("PlayerGui")
_G.EggGUI = gui

-- =================== BOTÃO FLUTUANTE (reabrir) ===================
local reopenBtn = Instance.new("TextButton")
reopenBtn.Size = UDim2.new(0, 50, 0, 50)
reopenBtn.Position = UDim2.new(0, 20, 0, 100)
reopenBtn.BackgroundColor3 = Color3.fromRGB(0, 180, 110)
reopenBtn.Text = "🥚"
reopenBtn.TextSize = 24
reopenBtn.TextColor3 = Color3.new(1,1,1)
reopenBtn.Font = Enum.Font.GothamBold
reopenBtn.Visible = false
reopenBtn.Parent = gui
reopenBtn.Draggable = true
Instance.new("UICorner", reopenBtn).CornerRadius = UDim.new(0, 25)

-- =================== FRAME PRINCIPAL ===================
local frame = Instance.new("Frame")
frame.Name = "Main"
frame.Size = UDim2.new(0, 280, 0, 400)
frame.Position = UDim2.new(0, 20, 0, 100)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.ClipsDescendants = true
frame.Parent = gui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)
local stroke = Instance.new("UIStroke", frame)
stroke.Color = Color3.fromRGB(0, 180, 110)
stroke.Thickness = 1.5

-- =================== BARRA DE TÍTULO ===================
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 35)
titleBar.BackgroundColor3 = Color3.fromRGB(0, 180, 110)
titleBar.BorderSizePixel = 0
titleBar.Parent = frame
Instance.new("UICorner", titleBar).CornerRadius = UDim.new(0, 10)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -80, 1, 0)
title.Position = UDim2.new(0, 10, 0, 0)
title.BackgroundTransparency = 1
title.Text = "🥚 EGG v4.0"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 15
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = titleBar

-- Botão minimizar
local minBtn = Instance.new("TextButton")
minBtn.Size = UDim2.new(0, 28, 0, 24)
minBtn.Position = UDim2.new(1, -66, 0, 6)
minBtn.BackgroundColor3 = Color3.fromRGB(255, 180, 0)
minBtn.Text = "—"
minBtn.TextColor3 = Color3.new(1,1,1)
minBtn.Font = Enum.Font.GothamBold
minBtn.TextSize = 16
minBtn.Parent = titleBar
Instance.new("UICorner", minBtn).CornerRadius = UDim.new(0, 5)

-- Botão fechar
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 28, 0, 24)
closeBtn.Position = UDim2.new(1, -34, 0, 6)
closeBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.new(1,1,1)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 14
closeBtn.Parent = titleBar
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 5)

-- =================== CONTEÚDO ===================
local content = Instance.new("Frame")
content.Name = "Content"
content.Size = UDim2.new(1, 0, 1, -35)
content.Position = UDim2.new(0, 0, 0, 35)
content.BackgroundTransparency = 1
content.Parent = frame

local scroll = Instance.new("ScrollingFrame")
scroll.Size = UDim2.new(1, -16, 1, -16)
scroll.Position = UDim2.new(0, 8, 0, 8)
scroll.BackgroundTransparency = 1
scroll.BorderSizePixel = 0
scroll.ScrollBarThickness = 4
scroll.ScrollBarImageColor3 = Color3.fromRGB(0, 180, 110)
scroll.CanvasSize = UDim2.new(0, 0, 0, 500)
scroll.Parent = content
Instance.new("UIListLayout", scroll).Padding = UDim.new(0, 5)

-- =================== LÓGICA MINIMIZAR ===================
local minimized = false
local originalSize = frame.Size

local function toggleMinimize()
    minimized = not minimized
    if minimized then
        -- Salva estado atual
        originalSize = frame.Size
        local tween = Tween:Create(frame, TweenInfo.new(0.2), {
            Size = UDim2.new(0, 280, 0, 35)
        })
        tween:Play()
        content.Visible = false
        minBtn.Text = "▢"
    else
        local tween = Tween:Create(frame, TweenInfo.new(0.2), {
            Size = originalSize
        })
        tween:Play()
        content.Visible = true
        minBtn.Text = "—"
    end
end

minBtn.MouseButton1Click:Connect(toggleMinimize)

-- Clique duplo no título também minimiza
titleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        toggleMinimize()
    end
end)

-- =================== LÓGICA FECHAR ===================
closeBtn.MouseButton1Click:Connect(function()
    frame.Visible = false
    reopenBtn.Visible = true
end)

reopenBtn.MouseButton1Click:Connect(function()
    frame.Visible = true
    reopenBtn.Visible = false
end)

-- =================== TOGGLES ===================
local function addToggle(name, key)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 30)
    btn.BackgroundColor3 = Config[key] and Color3.fromRGB(0,150,80) or Color3.fromRGB(50,50,70)
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.GothamSemibold
    btn.TextSize = 13
    btn.Text = (Config[key] and "✅ " or "❌ ") .. name
    btn.Parent = scroll
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    btn.MouseButton1Click:Connect(function()
        Config[key] = not Config[key]
        btn.BackgroundColor3 = Config[key] and Color3.fromRGB(0,150,80) or Color3.fromRGB(50,50,70)
        btn.Text = (Config[key] and "✅ " or "❌ ") .. name
    end)
end

addToggle("Auto Steal",  "AutoSteal")
addToggle("Auto Fusion", "AutoFusion")
addToggle("Auto Event",  "AutoEvent")
addToggle("ESP Ovos",    "ESP")
addToggle("Anti-AFK",    "AntiAFK")
addToggle("TP Suave",    "TPWalk")
addToggle("Só Melhores", "OnlyBest")

-- =================== STATUS ===================
local status = Instance.new("TextLabel")
status.Size = UDim2.new(1, 0, 0, 90)
status.BackgroundTransparency = 1
status.TextColor3 = Color3.fromRGB(180,180,200)
status.Font = Enum.Font.Gotham
status.TextSize = 11
status.TextWrapped = true
status.TextXAlignment = Enum.TextXAlignment.Left
status.TextYAlignment = Enum.TextYAlignment.Top
status.Text = "📡 Remotes detectados:\n  • Steal: "..#Remotes.steal.."\n  • Fusion: "..#Remotes.fusion.."\n  • Hatch: "..#Remotes.hatch.."\n\n💡 Botão direito no título = minimizar"
status.Parent = scroll

-- =================== RESIZE (canto inferior direito) ===================
local resizeHandle = Instance.new("TextButton")
resizeHandle.Size = UDim2.new(0, 14, 0, 14)
resizeHandle.Position = UDim2.new(1, -16, 1, -16)
resizeHandle.BackgroundColor3 = Color3.fromRGB(0, 180, 110)
resizeHandle.BackgroundTransparency = 0.5
resizeHandle.Text = ""
resizeHandle.Parent = frame
Instance.new("UICorner", resizeHandle).CornerRadius = UDim.new(0, 3)

local dragging, dragStart, startSize
resizeHandle.MouseButton1Down:Connect(function()
    dragging = true
    dragStart = UserInput:GetMouseLocation()
    startSize = frame.Size
end)
UserInput.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = UserInput:GetMouseLocation() - dragStart
        frame.Size = UDim2.new(
            startSize.X.Scale, math.max(220, startSize.X.Offset + delta.X),
            startSize.Y.Scale, math.max(120, startSize.Y.Offset + delta.Y)
        )
    end
end)
UserInput.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)
