-- LocalScript: HighlightBestBrainrot.lua
-- Explicação: procura por instâncias chamadas "Brainrot" nas "Bases" dentro do Workspace,
-- encontra a que tem maior "MoneyPerSecond" e destaca ela em amarelo, mostrando nome e valor (apenas número).
-- Não realiza coleta.

local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

-- Configurações (ajuste se preciso)
local BASES_FOLDER_NAME = "Bases"         -- pasta onde estão as bases (ajuste)
local BRAINROT_NAME = "Brainrot"         -- nome do objeto de cada brainrot (ajuste)
local MONEY_VALUE_NAME = "MoneyPerSecond" -- nome do NumberValue que contém dinheiro/segundo
local DISPLAY_NAME_VALUE = "DisplayName"  -- nome do StringValue com o nome (ou use obj.Name)
local UPDATE_INTERVAL = 1                -- segundos entre checagens (1s padrão)

-- Cria/gera uma Highlight para aplicar ao modelo
local function createHighlight()
    local highlight = Instance.new("Highlight")
    highlight.FillColor = Color3.fromRGB(255, 255, 0)    -- amarelo
    highlight.OutlineColor = Color3.fromRGB(100, 100, 0)
    highlight.RobloxLocked = true
    return highlight
end

-- Função para buscar todos os brainrots
local function findAllBrainrots()
    local list = {}
    local basesFolder = Workspace:FindFirstChild(BASES_FOLDER_NAME)
    if not basesFolder then
        return list
    end

    for _, base in pairs(basesFolder:GetDescendants()) do
        if base.Name == BRAINROT_NAME then
            table.insert(list, base)
        end
    end

    return list
end

-- Pega o valor MoneyPerSecond de um brainrot (retorna número ou nil)
local function getMoneyPerSecond(brainrot)
    if not brainrot then return nil end
    local val = brainrot:FindFirstChild(MONEY_VALUE_NAME)
    if val and val:IsA("NumberValue") then
        return val.Value
    end
    -- se não for NumberValue, tenta propriedade direta (ex: brainrot.MoneyPerSecond)
    local ok, result = pcall(function() return brainrot:FindFirstChildWhichIsA and brainrot:FindFirstChildWhichIsA("NumberValue") end)
    if ok and result then
        return result.Value
    end
    -- fallback: se houver uma propriedade chamada MoneyPerSecond
    local success, prop = pcall(function() return brainrot:FindFirstChild(MONEY_VALUE_NAME) end)
    if success and prop and prop.Value then
        return prop.Value
    end
    return nil
end

-- Pega o nome de display do brainrot (apenas string)
local function getDisplayName(brainrot)
    if not brainrot then return "Unknown" end
    local nameObj = brainrot:FindFirstChild(DISPLAY_NAME_VALUE)
    if nameObj and nameObj:IsA("StringValue") then
        return tostring(nameObj.Value)
    end
    -- fallback para usar o próprio Name
    return tostring(brainrot.Name or "Unknown")
end

-- Remove highlights antigos na cena (opcional)
local function clearHighlights()
    for _, child in pairs(Workspace:GetDescendants()) do
        if child:IsA("Highlight") and child.Name == "AutoBrainrotHighlight" then
            child:Destroy()
        end
    end
end

-- Principal: encontrar melhor brainrot e destacar
local currentHighlightTarget = nil

local function updateBestBrainrot()
    local brainrots = findAllBrainrots()
    if #brainrots == 0 then
        clearHighlights()
        currentHighlightTarget = nil
        return
    end

    local best = nil
    local bestValue = -math.huge

    for _, b in ipairs(brainrots) do
        local val = getMoneyPerSecond(b)
        if val and val > bestValue then
            bestValue = val
            best = b
        end
    end

    -- Se não encontrou valor válido
    if not best then
        clearHighlights()
        currentHighlightTarget = nil
        return
    end

    -- Se o mesmo alvo já está destacado, só atualiza a etiqueta com o valor se for necessário
    if currentHighlightTarget ~= best then
        -- limpar highlights antigos
        clearHighlights()

        -- criar highlight e parent para o modelo (ou workspace)
        local highlight = createHighlight()
        highlight.Name = "AutoBrainrotHighlight"
        highlight.Adornee = best
        highlight.Parent = Workspace

        currentHighlightTarget = best
    end

    -- Criar/atualizar uma BillboardGui para mostrar nome e valor (apenas número)
    -- Exibe apenas o número do dinheiro por segundo.
    local guiName = "AutoBrainrotLabel"
    local existingGui = currentHighlightTarget:FindFirstChild(guiName)
    if not existingGui then
        local billboard = Instance.new("BillboardGui")
        billboard.Name = guiName
        billboard.Adornee = currentHighlightTarget
        billboard.Size = UDim2.new(0, 150, 0, 50)
        billboard.StudsOffset = Vector3.new(0, 3, 0)
        billboard.AlwaysOnTop = true

        local textLabelName = Instance.new("TextLabel")
        textLabelName.Name = "NameLabel"
        textLabelName.Size = UDim2.new(1, 0, 0.5, 0)
        textLabelName.Position = UDim2.new(0, 0, 0, 0)
        textLabelName.BackgroundTransparency = 1
        textLabelName.TextScaled = true
        textLabelName.Text = getDisplayName(currentHighlightTarget)
        textLabelName.TextColor3 = Color3.new(1, 1, 1)
        textLabelName.Parent = billboard

        local textLabelValue = Instance.new("TextLabel")
        textLabelValue.Name = "ValueLabel"
        textLabelValue.Size = UDim2.new(1, 0, 0.5, 0)
        textLabelValue.Position = UDim2.new(0, 0, 0.5, 0)
        textLabelValue.BackgroundTransparency = 1
        textLabelValue.TextScaled = true
        textLabelValue.Text = tostring(math.floor(bestValue)) -- só o número inteiro
        textLabelValue.TextColor3 = Color3.new(1, 1, 0) -- amarelo
        textLabelValue.Parent = billboard

        billboard.Parent = currentHighlightTarget
    else
        -- atualiza valores do GUI existente
        local nameLabel = existingGui:FindFirstChild("NameLabel")
        local valueLabel = existingGui:FindFirstChild("ValueLabel")
        if nameLabel then nameLabel.Text = getDisplayName(currentHighlightTarget) end
        if valueLabel then valueLabel.Text = tostring(math.floor(bestValue)) end
    end
end

-- Loop de atualização periódico
spawn(function()
    while true do
        local ok, err = pcall(updateBestBrainrot)
        if not ok then
            warn("Erro ao atualizar best brainrot:", err)
        end
        wait(UPDATE_INTERVAL)
    end
end)
