local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- // CONFIGURAÇÃO DO ESP
local ESP_COLOR = Color3.fromRGB(0, 255, 127) -- Verde brilhante
local ATUALIZAR_A_CADA = 2 -- Segundos para re-escanear o melhor do mapa

local function CriarTag(part, nome, valor)
    -- Remove tags antigas para não sobrepor
    if part:FindFirstChild("BestBrainrotTag") then part.BestBrainrotTag:Destroy() end
    if part:FindFirstChild("BestHighlight") then part.BestHighlight:Destroy() end

    -- Criar o Texto Flutuante
    local bbg = Instance.new("BillboardGui", part)
    bbg.Name = "BestBrainrotTag"
    bbg.Size = UDim2.new(0, 200, 0, 50)
    bbg.Adornee = part
    bbg.AlwaysOnTop = true
    bbg.ExtentsOffset = Vector3.new(0, 3, 0)

    local tl = Instance.new("TextLabel", bbg)
    tl.Size = UDim2.new(1, 0, 1, 0)
    tl.BackgroundTransparency = 1
    tl.Text = "⭐ MELHOR: " .. nome .. "\n💰 " .. valor
    tl.TextColor3 = ESP_COLOR
    tl.TextStrokeTransparency = 0
    tl.Font = Enum.Font.GothamBold
    tl.TextSize = 14

    -- Criar o Efeito de Brilho (Highlight)
    local hl = Instance.new("Highlight", part)
    hl.Name = "BestHighlight"
    hl.FillColor = ESP_COLOR
    hl.FillAlpha = 0.5
    hl.OutlineColor = Color3.new(1, 1, 1)
end

local function EscanearMelhorBrainrot()
    local melhorValor = -1
    local melhorObjeto = nil
    local nomeDisplay = ""

    -- Varre as bases no Workspace (ajuste o caminho se o jogo mudar o local das bases)
    for _, v in pairs(workspace:GetDescendants()) do
        -- Procura por objetos que tenham o valor de $/s no nome ou em um atributo
        -- Geralmente no 'Steal a Brainrot', o valor está no ProximityPrompt ou em uma StringValue
        if v:IsA("TextLabel") or v:IsA("SurfaceGui") or v:IsA("BillboardGui") then
            -- Tenta capturar o valor "$...M/s" ou "$.../s"
            local texto = v:IsA("TextLabel") and v.Text or ""
            local valorStr = texto:match("%$(%d+%.?%d*[KMB]?)/s")
            
            if valorStr then
                -- Converte K, M, B para números reais para comparar
                local num = tonumber(valorStr:match("%d+%.?%d*")) or 0
                if valorStr:find("K") then num = num * 1000 
                elseif valorStr:find("M") then num = num * 1000000 
                elseif valorStr:find("B") then num = num * 1000000000 end

                if num > melhorValor then
                    melhorValor = num
                    melhorObjeto = v.Parent -- Pega a peça do item
                    nomeDisplay = v.Parent.Name
                end
            end
        end
    end

    -- Aplica o ESP no campeão do servidor
    if melhorObjeto and melhorObjeto:IsA("BasePart") then
        CriarTag(melhorObjeto, nomeDisplay, "$" .. melhorValor .. "/s")
    end
end

-- Loop de atualização automática
task.spawn(function()
    while true do
        EscanearMelhorBrainrot()
        task.wait(ATUALIZAR_A_CADA)
    end
end)
