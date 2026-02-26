-- ══════════════════════════════════════════
--   BRAINROT FINDER + SERVER HOPPER - v3.0
-- ══════════════════════════════════════════

local Players        = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local player         = Players.LocalPlayer
local playerGui      = player:WaitForChild("PlayerGui")
local PLACE_ID       = game.PlaceId

-- ══ RARIDADES ══════════════════════════════
local RARIDADES = {
    {
        nome     = "MÍTICO",
        emoji    = "🔥",
        cor      = Color3.fromRGB(180, 0, 255),
        textCor  = Color3.fromRGB(255, 255, 255),
        brainrots = {"Tigroline", "Frutonni"}
    },
    {
        nome     = "SECRETO",
        emoji    = "💎",
        cor      = Color3.fromRGB(255, 215, 0),
        textCor  = Color3.fromRGB(0, 0, 0),
        brainrots = {"Garama", "Madudung", "La Vaca Saturnita", "Tralaledon", "Esok"}
    },
    {
        nome     = "STELLAR",
        emoji    = "🌟",
        cor      = Color3.fromRGB(0, 191, 255),
        textCor  = Color3.fromRGB(255, 255, 255),
        brainrots = {"Strawberry Elephant", "Capitano Clash", "Warnini", "Meowl"}
    }
}

-- ══ VARIÁVEIS ══════════════════════════════
local TARGET_BRAINROT  = nil
local SEARCHING        = false
local currentSubFrame  = nil

-- ══ FUNÇÕES UTILITÁRIAS ════════════════════
local function parseEarn(text)
    local raw = text or ""
    local num = tonumber(raw:match("[%d%.]+"))
    if not num then return 0 end
    if raw:find("B") or raw:find("b") then num = num * 1e9
    elseif raw:find("M") or raw:find("m") then num = num * 1e6
    elseif raw:find("K") or raw:find("k") then num = num * 1e3
    end
    return num
end

local function clearHighlights()
    for _, h in ipairs(game.Workspace:GetDescendants()) do
        if h:IsA("Highlight") and h.Name == "BestBaseHighlight" then
            h:Destroy()
        end
    end
end

local function findBrainrot(targetName)
    local clientFolder = game.Workspace:FindFirstChild("Client")
    if not clientFolder then return nil, 0, "", "" end
    local basesFolder = clientFolder:FindFirstChild("Bases")
    if not basesFolder then return nil, 0, "", "" end

    local bestEarn   = -math.huge
    local bestEntity = nil
    local bestText   = ""
    local bestName   = ""

    for _, base in ipairs(basesFolder:GetChildren()) do
        for _, entity in ipairs(base:GetDescendants()) do
            local tLow = targetName:lower()
            local eLow = entity.Name:lower()
            if eLow:find(tLow) or tLow:find(eLow) then
                local earnVal  = 0
                local earnText = "?"
                local dispName = entity.Name

                local ui = entity:FindFirstChild("Brainrot_UI")
                if ui and ui:IsA("BillboardGui") then
                    local frame = ui:FindFirstChild("Frame")
                    if frame then
                        local eLabel = frame:FindFirstChild("Earn")
                        if eLabel then
                            earnVal  = parseEarn(eLabel.Text)
                            earnText = eLabel.Text
                        end
                        local tLabel = frame:FindFirstChild("Title")
                        if tLabel then dispName = tLabel.Text end
                    end
                end

                if earnVal > bestEarn then
                    bestEarn   = earnVal
                    bestEntity = entity
                    bestText   = earnText
                    bestName   = dispName
                end
            end
        end
    end

    return bestEntity, bestEarn, bestText, bestName
end

local function teleportToNewServer()
    print("🔄 Trocando servidor...")
    local ok, err = pcall(function()
        TeleportService:TeleportToPlaceId(PLACE_ID, nil, true)
    end)
    if not ok then warn("❌ Erro: " .. tostring(err)) end
end

-- ══ LIMPA GUIs ANTIGAS ════════════════════
for _, name in ipairs({"BrainrotSearchGui", "FoundGui"}) do
    local old = playerGui:FindFirstChild(name)
    if old then old:Destroy() end
end
clearHighlights()

-- ══ MAIN GUI ══════════════════════════════
local screenGui = Instance.new("ScreenGui")
screenGui.Name          = "BrainrotSearchGui"
screenGui.ResetOnSpawn  = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent        = playerGui

local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size     = UDim2.new(0, 360, 0, 115)
mainFrame.Position = UDim2.new(0.5, -180, 0, 20)
mainFrame.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
mainFrame.BackgroundTransparency = 0.1
mainFrame.BorderSizePixel = 0
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 14)
local mainStroke = Instance.new("UIStroke", mainFrame)
mainStroke.Color     = Color3.fromRGB(255, 215, 0)
mainStroke.Thickness = 2

-- Título
local titleLabel = Instance.new("TextLabel", mainFrame)
titleLabel.Size               = UDim2.new(1, -90, 0, 28)
titleLabel.Position           = UDim2.new(0, 8, 0, 5)
titleLabel.BackgroundTransparency = 1
titleLabel.Text               = "🔍 BUSCA DE BRAINROT"
titleLabel.TextColor3         = Color3.fromRGB(255, 215, 0)
titleLabel.TextScaled         = true
titleLabel.Font               = Enum.Font.GothamBold

-- Status
local statusLabel = Instance.new("TextLabel", mainFrame)
statusLabel.Size               = UDim2.new(1, -16, 0, 20)
statusLabel.Position           = UDim2.new(0, 8, 0, 36)
statusLabel.BackgroundTransparency = 1
statusLabel.Text               = "Selecione uma raridade abaixo"
statusLabel.TextColor3         = Color3.fromRGB(160, 160, 160)
statusLabel.TextScaled         = true
statusLabel.Font               = Enum.Font.Gotham

-- Botão PARAR
local stopBtn = Instance.new("TextButton", mainFrame)
stopBtn.Size     = UDim2.new(0, 78, 0, 26)
stopBtn.Position = UDim2.new(1, -86, 0, 5)
stopBtn.BackgroundColor3 = Color3.fromRGB(180, 40, 40)
stopBtn.Text      = "⏹ PARAR"
stopBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
stopBtn.TextScaled = true
stopBtn.Font      = Enum.Font.GothamBold
stopBtn.BorderSizePixel = 0
Instance.new("UICorner", stopBtn).CornerRadius = UDim.new(0, 7)

stopBtn.MouseButton1Click:Connect(function()
    SEARCHING     = false
    TARGET_BRAINROT = nil
    mainFrame.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
    statusLabel.Text      = "⏹ Busca parada."
    statusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
    clearHighlights()
    if currentSubFrame then
        currentSubFrame:Destroy()
        currentSubFrame = nil
    end
    print("🛑 Busca parada!")
end)

-- Container dos 3 botões de raridade
local rarContainer = Instance.new("Frame", mainFrame)
rarContainer.Size     = UDim2.new(1, -20, 0, 38)
rarContainer.Position = UDim2.new(0, 10, 0, 68)
rarContainer.BackgroundTransparency = 1

local rarLayout = Instance.new("UIListLayout", rarContainer)
rarLayout.FillDirection        = Enum.FillDirection.Horizontal
rarLayout.HorizontalAlignment  = Enum.HorizontalAlignment.Center
rarLayout.VerticalAlignment    = Enum.VerticalAlignment.Center
rarLayout.SortOrder            = Enum.SortOrder.LayoutOrder
rarLayout.Padding              = UDim.new(0, 10)

-- ══ CRIA OS 3 BOTÕES DE RARIDADE ══════════
for i, rar in ipairs(RARIDADES) do
    local btn = Instance.new("TextButton", rarContainer)
    btn.Size     = UDim2.new(0, 105, 1, 0)
    btn.BackgroundColor3 = rar.cor
    btn.Text      = rar.emoji .. " " .. rar.nome
    btn.TextColor3 = rar.textCor
    btn.TextScaled = true
    btn.Font      = Enum.Font.GothamBold
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = true
    btn.LayoutOrder = i
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 9)

    btn.MouseButton1Click:Connect(function()
        -- Toggle: fecha se já estiver aberto
        if currentSubFrame then
            currentSubFrame:Destroy()
            currentSubFrame = nil
            return
        end

        -- Cria submenu
        local ROW_H   = 36
        local PADDING = 8
        local subH    = 44 + (#rar.brainrots * (ROW_H + PADDING))

        local sub = Instance.new("Frame", screenGui)
        sub.Name     = "SubMenu"
        sub.Size     = UDim2.new(0, 300, 0, subH)
        sub.Position = UDim2.new(0.5, -150, 0, 148)
        sub.BackgroundColor3 = Color3.fromRGB(22, 22, 22)
        sub.BorderSizePixel  = 0
        sub.ZIndex           = 5
        Instance.new("UICorner", sub).CornerRadius = UDim.new(0, 12)
        local subStroke = Instance.new("UIStroke", sub)
        subStroke.Color     = rar.cor
        subStroke.Thickness = 2
        currentSubFrame = sub

        -- Título do submenu
        local subTitle = Instance.new("TextLabel", sub)
        subTitle.Size     = UDim2.new(1, -35, 0, 32)
        subTitle.Position = UDim2.new(0, 8, 0, 6)
        subTitle.BackgroundTransparency = 1
        subTitle.Text      = rar.emoji .. " Escolha o brainrot:"
        subTitle.TextColor3 = rar.cor
        subTitle.TextScaled = true
        subTitle.Font      = Enum.Font.GothamBold
        subTitle.ZIndex    = 6

        -- Botão fechar submenu
        local closeBtn = Instance.new("TextButton", sub)
        closeBtn.Size     = UDim2.new(0, 24, 0, 24)
        closeBtn.Position = UDim2.new(1, -28, 0, 6)
        closeBtn.BackgroundColor3 = Color3.fromRGB(180, 40, 40)
        closeBtn.Text      = "✕"
        closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        closeBtn.TextScaled = true
        closeBtn.Font      = Enum.Font.GothamBold
        closeBtn.BorderSizePixel = 0
        closeBtn.ZIndex    = 7
        Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 5)
        closeBtn.MouseButton1Click:Connect(function()
            sub:Destroy()
            currentSubFrame = nil
        end)

        -- Botões dos brainrots
        for j, brName in ipairs(rar.brainrots) do
            local brBtn = Instance.new("TextButton", sub)
            brBtn.Size     = UDim2.new(1, -20, 0, ROW_H)
            brBtn.Position = UDim2.new(0, 10, 0, 42 + (j - 1) * (ROW_H + PADDING))
            brBtn.BackgroundColor3 = Color3.fromRGB(42, 42, 42)
            brBtn.Text      = "🧠 " .. brName
            brBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
            brBtn.TextScaled = true
            brBtn.Font      = Enum.Font.GothamBold
            brBtn.BorderSizePixel = 0
            brBtn.AutoButtonColor = true
            brBtn.ZIndex    = 6
            Instance.new("UICorner", brBtn).CornerRadius = UDim.new(0, 8)

            brBtn.MouseButton1Click:Connect(function()
                TARGET_BRAINROT = brName
                SEARCHING       = true
                mainFrame.BackgroundColor3 = Color3.fromRGB(70, 20, 20)
                statusLabel.Text       = "🔍 Buscando: " .. brName .. "..."
                statusLabel.TextColor3 = Color3.fromRGB(255, 200, 50)
                sub:Destroy()
                currentSubFrame = nil
                clearHighlights()
                print("🎯 Iniciando busca por: " .. brName)
                teleportToNewServer()
            end)
        end
    end)
end

-- ══ LOOP DE VERIFICAÇÃO ═══════════════════
task.spawn(function()
    while task.wait(3) do
        if SEARCHING and TARGET_BRAINROT then
            local found, _, earnText, foundName = findBrainrot(TARGET_BRAINROT)

            if found then
                SEARCHING       = false
                TARGET_BRAINROT = nil
                mainFrame.BackgroundColor3 = Color3.fromRGB(15, 65, 15)
                statusLabel.Text       = "✅ ENCONTRADO: " .. foundName
                statusLabel.TextColor3 = Color3.fromRGB(80, 255, 80)

                -- Highlight verde
                clearHighlights()
                local hl = Instance.new("Highlight", found)
                hl.Name               = "BestBaseHighlight"
                hl.FillColor          = Color3.fromRGB(0, 255, 0)
                hl.OutlineColor       = Color3.fromRGB(0, 220, 0)
                hl.FillTransparency   = 0.3
                hl.OutlineTransparency = 0

                -- UI de sucesso
                local sGui = Instance.new("ScreenGui", playerGui)
                sGui.Name         = "FoundGui"
                sGui.ResetOnSpawn = false

                local sFrame = Instance.new("Frame", sGui)
                sFrame.Size     = UDim2.new(0, 290, 0, 90)
                sFrame.Position = UDim2.new(0.5, -145, 0.45, 0)
                sFrame.BackgroundColor3 = Color3.fromRGB(12, 55, 12)
                sFrame.BorderSizePixel  = 0
                Instance.new("UICorner", sFrame).CornerRadius = UDim.new(0, 14)
                local sStroke = Instance.new("UIStroke", sFrame)
                sStroke.Color     = Color3.fromRGB(0, 255, 0)
                sStroke.Thickness = 3

                local sTitle = Instance.new("TextLabel", sFrame)
                sTitle.Size     = UDim2.new(1, 0, 0.5, 0)
                sTitle.BackgroundTransparency = 1
                sTitle.Text      = "🎉 BRAINROT ENCONTRADO!"
                sTitle.TextColor3 = Color3.fromRGB(0, 255, 0)
                sTitle.TextScaled = true
                sTitle.Font      = Enum.Font.GothamBold

                local sInfo = Instance.new("TextLabel", sFrame)
                sInfo.Size     = UDim2.new(1, 0, 0.5, 0)
                sInfo.Position = UDim2.new(0, 0, 0.5, 0)
                sInfo.BackgroundTransparency = 1
                sInfo.Text      = "🧠 " .. foundName .. "  💰 " .. earnText
                sInfo.TextColor3 = Color3.fromRGB(200, 255, 200)
                sInfo.TextScaled = true
                sInfo.Font      = Enum.Font.Gotham

                print("✅ ENCONTRADO: " .. foundName .. " | Earn: " .. earnText)

                task.delay(5, function()
                    if sGui and sGui.Parent then sGui:Destroy() end
                end)
            else
                print("❌ " .. TARGET_BRAINROT .. " não encontrado. Trocando servidor...")
                task.wait(1)
                teleportToNewServer()
            end
        end
    end
end)

print("✅ Script carregado! Clique numa raridade para começar.")
