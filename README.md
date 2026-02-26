local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- // INTERFACE
local sg = Instance.new("ScreenGui", (gethui and gethui()) or game:GetService("CoreGui"))
sg.Name = "BrainrotScanner_V2"
sg.ResetOnSpawn = false

local Main = Instance.new("Frame", sg)
Main.Size = UDim2.fromOffset(210, 80)
Main.Position = UDim2.new(0.5, -105, 0.1, 0)
Main.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Main.BorderSizePixel = 0
Main.Active = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)

local Stroke = Instance.new("UIStroke", Main)
Stroke.Color = Color3.fromRGB(255, 215, 0)
Stroke.Thickness = 2

-- Botão de Fechar
local CloseBtn = Instance.new("TextButton", Main)
CloseBtn.Size = UDim2.fromOffset(22, 22)
CloseBtn.Position = UDim2.new(1, -27, 0, 5)
CloseBtn.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.new(1, 1, 1)
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 12
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(1, 0)

-- Botão Ativar
local EspBtn = Instance.new("TextButton", Main)
EspBtn.Size = UDim2.new(0.9, 0, 0, 35)
EspBtn.Position = UDim2.new(0.05, 0, 0.45, 0)
EspBtn.Text = "FILTRAR MELHOR $/s"
EspBtn.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
EspBtn.TextColor3 = Color3.new(1, 1, 1)
EspBtn.Font = Enum.Font.GothamBold
EspBtn.TextSize = 12
Instance.new("UICorner", EspBtn)

-- // LÓGICA DE FILTRO
local EspAtivo = false

local function ParseValor(texto)
    if not texto then return 0 end
    -- Remove $, /s e espaços para converter K, M, B
    local s = texto:gsub("%%", ""):gsub("%$", ""):gsub(",", ""):gsub("/s", ""):upper()
    local mult = 1
    if s:find("B") then mult = 1e9 s = s:gsub("B","") end
    if s:find("M") then mult = 1e6 s = s:gsub("M","") end
    if s:find("K") then mult = 1e3 s = s:gsub("K","") end
    return (tonumber(s) or 0) * mult
end

local function LimparESP()
    for _, v in pairs(workspace:GetDescendants()) do
        if v.Name == "BestHighlight" or v.Name == "BestTag" then
            v:Destroy()
        end
    end
end

local function EscanearMelhorBrainrot()
    if not EspAtivo then return end
    
    local melhorValor = -1
    local melhorObjeto = nil
    local nomeFinal = "Desconhecido"
    local textoValor = ""

    -- Varre as bases do jogo
    local bases = workspace:FindFirstChild("Server") and workspace.Server:FindFirstChild("Bases")
    if not bases then return end

    for _, base in pairs(bases:GetChildren()) do
        local slots = base:FindFirstChild("Slots")
        if not slots then continue end
        
        for _, slot in pairs(slots:GetChildren()) do
            -- Ignora slots vazios (que têm o prompt de compra)
            if slot:FindFirstChild("PlacePrompt") then continue end
            
            -- Procura o valor de rendimento ($/s) dentro do slot
            -- Geralmente o jogo coloca um TextLabel indicando quanto aquele item gera
            for _, d in pairs(slot:GetDescendants()) do
                if d:IsA("TextLabel") and d.Text:find("/s") then
                    -- Ignora o botão de coletar (Collect)
                    if d:FindFirstAncestor("Collect") then continue end
                    
                    local valorNumerico = ParseValor(d.Text)
                    if valorNumerico > melhorValor then
                        melhorValor = valorNumerico
                        textoValor = d.Text
                        melhorObjeto = slot:FindFirstChild("Handle") or slot:FindFirstChildWhichIsA("BasePart")
                        
                        -- Tenta achar o nome do Brainrot no Modelo
                        nomeFinal = slot.Name
                        for _, model in pairs(slot:GetChildren()) do
                            if model:IsA("Model") then
                                nomeFinal = model.Name
                                break
                            end
                        end
                    end
                end
            end
        end
    end

    LimparESP()

    if melhorObjeto then
        -- Highlight Amarelo
        local hl = Instance.new("Highlight", melhorObjeto)
        hl.Name = "BestHighlight"
        hl.FillColor = Color3.fromRGB(255, 215, 0)
        hl.OutlineColor = Color3.new(1, 1, 1)

        -- Tag com Nome e Rendimento
        local bbg = Instance.new("BillboardGui", melhorObjeto)
        bbg.Name = "BestTag"
        bbg.Size = UDim2.new(0, 180, 0, 50)
        bbg.AlwaysOnTop = true
        bbg.ExtentsOffset = Vector3.new(0, 4, 0)

        local f = Instance.new("Frame", bbg)
        f.Size = UDim2.fromScale(1, 1)
        f.BackgroundColor3 = Color3.new(0,0,0)
        f.BackgroundTransparency = 0.4
        Instance.new("UICorner", f)

        local t1 = Instance.new("TextLabel", f)
        t1.Size = UDim2.fromScale(1, 0.5)
        t1.Text = nomeFinal:upper()
        t1.TextColor3 = Color3.new(1, 1, 1)
        t1.Font = Enum.Font.GothamBold
        t1.TextScaled = true
        t1.BackgroundTransparency = 1

        local t2 = Instance.new("TextLabel", f)
        t2.Size = UDim2.fromScale(1, 0.4)
        t2.Position = UDim2.fromScale(0, 0.5)
        t2.Text = "⭐ " .. textoValor
        t2.TextColor3 = Color3.fromRGB(255, 215, 0)
        t2.Font = Enum.Font.GothamBold
        t2.TextScaled = true
        t2.BackgroundTransparency = 1
    end
end

-- // BOTÕES
EspBtn.MouseButton1Click:Connect(function()
    EspAtivo = not EspAtivo
    EspBtn.Text = EspAtivo and "SCANNER: LIGADO" or "FILTRAR MELHOR $/s"
    EspBtn.TextColor3 = EspAtivo and Color3.fromRGB(0, 255, 150) or Color3.new(1, 1, 1)
    if EspAtivo then
        task.spawn(function()
            while EspAtivo do
                EscanearMelhorBrainrot()
                task.wait(3)
            end
        end)
    else
        LimparESP()
    end
end)

CloseBtn.MouseButton1Click:Connect(function()
    EspAtivo = false
    LimparESP()
    sg:Destroy()
end)

-- // DRAG (ARRASTAR)
local d, ds, sp
Main.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then d = true; ds = i.Position; sp = Main.Position end end)
UserInputService.InputChanged:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseMovement and d then local del = i.Position - ds; Main.Position = UDim2.new(sp.X.Scale, sp.X.Offset + del.X, sp.Y.Scale, sp.Y.Offset + del.Y) end end)
UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then d = false end end)
