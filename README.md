-- ============ MELHOR BRAINROT NAS BASES - v2.0 ============
local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- PlaceId do jogo (mude se necessário)
local PLACE_ID = game.PlaceId

-- ============ DADOS DAS RARIDADES ============
local RARIDADES = {
    ["⭐ MÍTICO"] = {"Tigroline", "Frutonni"},
    ["💎 SECRETO"] = {"Garama", "Madudung", "La Vaca Saturnita", "Tralaledon", "Esok"},
    ["🌟 STELLAR"] = {"Strawberry Elephant", "Capitano Clash", "Warnini", "Meowl"}
}

local TARGET_BRAINROT = nil
local SEARCHING = false
local basesFolder = game.Workspace:WaitForChild("Client"):WaitForChild("Bases")
local bestBrainrot = nil

-- ============ LIMPA UI ANTIGA ============
local oldGui = playerGui:FindFirstChild("BrainrotSearchGui")
if oldGui then oldGui:Destroy() end

for _, h in pairs(game.Workspace:GetDescendants()) do
    if h:IsA("Highlight") and h.Name == "BestBaseHighlight" then
        h:Destroy()
    end
end

-- ============ FUNÇÕES ============
local function parseEarn(text)
    local raw = text or ""
    local num = tonumber(raw:match("[%d%.]+"))
    if not num then return 0 end
    if raw:find("B") or raw:find("b") then
        num = num * 1_000_000_000
    elseif raw:find("M") or raw:find("m") then
        num = num * 1_000_000
    elseif raw:find("K") or raw:find("k") then
        num = num * 1_000
    end
    return num
end

local function findBrainrotInBase(targetName)
    local bestEarn = -math.huge
    local bestBrainrot = nil
    local bestEarnText = ""
    local bestName = ""

    for _, base in pairs(basesFolder:GetChildren()) do
        for _, entity in pairs(base:GetDescendants()) do
            local ui = entity:FindFirstChild("Brainrot_UI")
            local BrainrotInfo = entity:FindFirstChild("BrainrotInfo")
            
            -- Verifica tanto pela UI quanto pelo nome
            local matches = false
            local brainrotName = entity.Name

            if BrainrotInfo and BrainrotInfo:IsA("ObjectValue") then
                -- Tenta pegar o nome correto
            end

            -- Verifica se é o brainrot alvo
            for _, part in pairs(entity:GetChildren()) do
                if part:IsA("Part") or part:IsA("Model") then
                    local nameLower = part.Name:lower()
                    local targetLower = targetName:lower()
                    if nameLower:find(targetLower) or targetLower:find(nameLower) then
                        matches = true
                        brainrotName = part.Name
                        break
                    end
                end
            end
            
            if not matches then
                local nameLower = entity.Name:lower()
                local targetLower = targetName:lower()
                if nameLower:find(targetLower) or targetLower:find(nameLower) then
                    matches = true
                    brainrotName = entity.Name
                end
            end

            if matches then
                -- Busca o valor de Earn na UI
                local earnValue = 0
                if ui and ui:IsA("BillboardGui") then
                    local frame = ui:FindFirstChild("Frame")
                    if frame then
                        local earnLabel = frame:FindFirstChild("Earn")
                        if earnLabel and earnLabel:IsA("TextLabel") then
                            earnValue = parseEarn(earnLabel.Text)
                            local titleLabel = frame:FindFirstChild("Title")
                            if titleLabel and titleLabel:IsA("TextLabel") then
                                brainrotName = titleLabel.Text
                            end
                        end
                    end
                end

                if earnValue > bestEarn then
                    bestEarn = earnValue
                    bestBrainrot = entity
                    bestEarnText = tostring(bestEarn)
                    bestName = brainrotName
                end
            end
        end
    end

    return bestBrainrot, bestEarn, bestEarnText, bestName
end

local function teleportToNewServer()
    SEARCHING = true
    print("🔄 Mudando de servidor...")
    
    -- Tenta teleportar para um servidor aleatório
    local success, err = pcall(function()
        TeleportService:TeleportToPlaceId(PLACE_ID, nil, true)
    end)
    
    if not success then
        print("❌ Erro ao teleportar: " .. tostring(err))
        SEARCHING = false
    end
end

-- ============ CRIA A UI ============
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "BrainrotSearchGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

-- Frame principal
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 320, 0, 50)
mainFrame.Position = UDim2.new(0.5, -160, 0, 20)
mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 12)
mainCorner.Parent = mainFrame

local mainStroke = Instance.new("UIStroke")
mainStroke.Color = Color3.fromRGB(255, 215, 0)
mainStroke.Thickness = 2
mainStroke.Parent = mainFrame

-- Título
local header = Instance.new("TextLabel")
header.Size = UDim2.new(1, 0, 0.3, 0)
header.Position = UDim2.new(0, 0, 0, 0)
header.BackgroundTransparency = 1
header.Text = "🔍 BUSCA POR RARIDADE"
header.TextColor3 = Color3.fromRGB(255, 215, 0)
header.TextScaled = true
header.Font = Enum.Font.GothamBold
header.Parent = mainFrame

-- Container para os botões de raridade
local raritiesContainer = Instance.new("Frame")
raritiesContainer.Size = UDim2.new(1, -20, 0.4, 0)
raritiesContainer.Position = UDim2.new(0, 10, 0.35, 0)
raritiesContainer.BackgroundTransparency = 1
raritiesContainer.Parent = mainFrame

local listLayout = Instance.new("UIListLayout")
listLayout.FillDirection = Enum.FillDirection.Horizontal
listLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
listLayout.SortOrder = Enum.SortOrder.LayoutOrder
listLayout.Padding = UDim.new(0, 8)
listLayout.Parent = raritiesContainer

-- ============ CRIA BOTÕES DAS RARIDADES ============
local rarityFrames = {}

for raridade, brainrots in pairs(RARIDADES) do
    local rarBtn = Instance.new("TextButton")
    rarBtn.Size = UDim2.new(0, 95, 1, 0)
    rarBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    rarBtn.Text = raridade
    rarBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    rarBtn.TextScaled = true
    rarBtn.Font = Enum.Font.GothamBold
    rarBtn.BorderSizePixel = 0
    rarBtn.AutoButtonColor = true
    rarBtn.Parent = raritiesContainer
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 8)
    btnCorner.Parent = rarBtn

    -- Cores específicas por raridade
    if raridade:find("MÍTICO") then
        rarBtn.BackgroundColor3 = Color3.fromRGB(128, 0, 128) -- Roxo
    elseif raridade:find("SECRETO") then
        rarBtn.BackgroundColor3 = Color3.fromRGB(255, 215, 0) -- Dourado
        rarBtn.TextColor3 = Color3.fromRGB(0, 0, 0)
    elseif raridade:find("STELLAR") then
        rarBtn.BackgroundColor3 = Color3.fromRGB(0, 191, 255) -- Azul claro
    end
    
    local subFrame = nil
    
    rarBtn.MouseButton1Click:Connect(function()
        -- Se já existe um submenu, destrua
        if subFrame then subFrame:Destroy() end
        
        -- Mostra/esconde submenu
        subFrame = Instance.new("Frame")
        subFrame.Size = UDim2.new(0, 300, 0, 0)
        subFrame.Position = UDim2.new(0.5, -150, 0, 80)
        subFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
        subFrame.BorderSizePixel = 0
        subFrame.ZIndex = 10
        subFrame.Parent = screenGui
        
        local subCorner = Instance.new("UICorner")
        subCorner.CornerRadius = UDim.new(0, 10)
        subCorner.Parent = subFrame
        
        local subStroke = Instance.new("UIStroke")
        subStroke.Color = rarBtn.BackgroundColor3
        subStroke.Thickness = 2
        subStroke.Parent = subFrame
        
        -- Título
        local subHeader = Instance.new("TextLabel")
        subHeader.Size = UDim2.new(1, 0, 0, 25)
        subHeader.Position = UDim2.new(0, 0, 0, 0)
        subHeader.BackgroundTransparency = 1
        subHeader.Text = "Selecione um Brainrot:"
        subHeader.TextColor3 = Color3.fromRGB(255, 255, 255)
        subHeader.TextScaled = true
        subHeader.Font = Enum.Font.Gotham
        subHeader.ZIndex = 11
        subHeader.Parent = subFrame
        
        -- Container dos brainrots
        local brainrotsContainer = Instance.new("Frame")
        brainrotsContainer.Size = UDim2.new(1, -16, 0, 0)
        brainrotsContainer.Position = UDim2.new(0, 8, 0, 30)
        brainrotsContainer.BackgroundTransparency = 1
        brainrotsContainer.ZIndex = 11
        brainrotsContainer.Parent = subFrame
        
        local brainrotLayout = Instance.new("UIListLayout")
        brainrotLayout.FillDirection = Enum.FillDirection.Horizontal
        brainrotLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
        brainrotLayout.SortOrder = Enum.SortOrder.LayoutOrder
        brainrotLayout.Padding = UDim.new(0, 8)
        brainrotLayout.Parent = brainrotsContainer
        
        local totalHeight = 35
        
        for i, brainrotName in ipairs(brainrots) do
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(0, 85, 0, 30)
            btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
            btn.Text = brainrotName
            btn.TextColor3 = Color3.fromRGB(255, 255, 255)
            btn.TextScaled = true
            btn.Font = Enum.Font.GothamBold
            btn.BorderSizePixel = 0
            btn.AutoButtonColor = true
            btn.ZIndex = 12
            btn.Parent = brainrotsContainer
            
            local btnCorner = Instance.new("UICorner")
            btnCorner.CornerRadius = UDim.new(0, 6)
            btnCorner.Parent = btn
            
            totalHeight = totalHeight + 35 + 8
            
            btn.MouseButton1Click:Connect(function()
                TARGET_BRAINROT = brainrotName
                SEARCHING = true
                
                -- Atualiza UI
                subHeader.Text = "🔍 BUSCANDO: " .. brainrotName
                mainFrame.BackgroundColor3 = Color3.fromRGB(80, 20, 20)
                
                -- Fecha submenu
                if subFrame then
                    subFrame:Destroy()
                    subFrame = nil
                end
                
                print("🎯 Procurando: " .. brainrotName)
                teleportToNewServer()
            end)
        end
        
        -- Altura dinâmica
        subFrame.Size = UDim2.new(0, 300, 0, totalHeight + 5)
        brainrotsContainer.Size = UDim2.new(1, -16, 0, totalHeight - 35)
        
        -- Botão fechar
        local closeBtn = Instance.new("TextButton")
        closeBtn.Size = UDim2.new(0, 20, 0, 20)
        closeBtn.Position = UDim2.new(1, -28, 0, 4)
        closeBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
        closeBtn.Text = "X"
        closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        closeBtn.TextScaled = true
        closeBtn.Font = Enum.Font.GothamBold
        closeBtn.BorderSizePixel = 0
        closeBtn.ZIndex = 12
        closeBtn.Parent = subFrame
        
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 4)
        btnCorner.Parent = closeBtn
        
        closeBtn.MouseButton1Click:Connect(function()
            if subFrame then subFrame:Destroy() end
        end)
    end)
end

-- ============ BOTÃO PARAR BUSCA ============
local stopBtn = Instance.new("TextButton")
stopBtn.Size = UDim2.new(0, 80, 0, 30)
stopBtn.Position = UDim2.new(1, -90, 0.7, 0)
stopBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
stopBtn.Text = "⏹ PARAR"
stopBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
stopBtn.TextScaled = true
stopBtn.Font = Enum.Font.GothamBold
stopBtn.BorderSizePixel = 0
stopBtn.AutoButtonColor = true
stopBtn.Parent = mainFrame

local stopCorner = Instance.new("UICorner")
stopCorner.CornerRadius = UDim.new(0, 6)
stopCorner.Parent = stopBtn

stopBtn.MouseButton1Click:Connect(function()
    SEARCHING = false
    TARGET_BRAINROT = nil
    mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    print("🛑 Busca parada!")
    
    -- Remove highlights
    for _, h in pairs(game.Workspace:GetDescendants()) do
        if h:IsA("Highlight") and h.Name == "BestBaseHighlight" then
            h:Destroy()
        end
    end
end)

-- ============ LOOP DE VERIFICAÇÃO ============
task.spawn(function()
    while task.wait(2) do
        if SEARCHING and TARGET_BRAINROT then
            local found, earn, earnText, name = findBrainrotInBase(TARGET_BRAINROT)
            
            if found then
                SEARCHING = false
                mainFrame.BackgroundColor3 = Color3.fromRGB(20, 80, 20)
                print("✅ BRAINROT ENCONTRADO: " .. name .. " | Earn: " .. earnText)
                
                -- Aplica highlight
                local highlight = Instance.new("Highlight")
                highlight.Name = "BestBaseHighlight"
                highlight.FillColor = Color3.fromRGB(0, 255, 0)
                highlight.OutlineColor = Color3.fromRGB(0, 200, 0)
                highlight.FillTransparency = 0.3
                highlight.OutlineTransparency = 0
                highlight.Parent = found
                
                -- Cria UI de sucesso
                local successGui = Instance.new("ScreenGui")
                successGui.Name = "FoundGui"
                successGui.ResetOnSpawn = false
                successGui.Parent = playerGui
                
                local foundFrame = Instance.new("Frame")
                foundFrame.Size = UDim2.new(0, 250, 0, 80)
                foundFrame.Position = UDim2.new(0.5, -125, 0.5, -40)
                foundFrame.BackgroundColor3 = Color3.fromRGB(20, 70, 20)
                foundFrame.BorderSizePixel = 0
                foundFrame.Parent = successGui
                
                local foundCorner = Instance.new("UICorner")
                foundCorner.CornerRadius = UDim.new(0, 12)
                foundCorner.Parent = foundFrame
                
                local foundStroke = Instance.new("UIStroke")
                foundStroke.Color = Color3.fromRGB(0, 255, 0)
                foundStroke.Thickness = 3
                foundStroke.Parent = foundFrame
                
                local foundLabel = Instance.new("TextLabel")
                foundLabel.Size = UDim2.new(1, 0, 0.5, 0)
                foundLabel.Position = UDim2.new(0, 0, 0, 0)
                foundLabel.BackgroundTransparency = 1
                foundLabel.Text = "🎉 BRAINROT ENCONTRADO!"
                foundLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
                foundLabel.TextScaled = true
                foundLabel.Font = Enum.Font.GothamBold
                foundLabel.Parent = foundFrame
                
                local foundName = Instance.new("TextLabel")
                foundName.Size = UDim2.new(1, 0, 0.5, 0)
                foundName.Position = UDim2.new(0, 0, 0.5, 0)
                foundName.BackgroundTransparency = 1
                foundName.Text = "🧠 " .. name
                foundName.TextColor3 = Color3.fromRGB(255, 255, 255)
                foundName.TextScaled = true
                foundName.Font = Enum.Font.GothamBold
                foundName.Parent = foundFrame
                
                wait(3)
                successGui:Destroy()
            else
                print("❌ " .. TARGET_BRAINROT .. " não encontrado, mudando de servidor...")
                teleportToNewServer()
            end
        end
    end
end)

print("✅ UI de busca carregada! Selecione uma raridade.")
