local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- // INTERFACE PREMIUM ARRASTÁVEL
local sg = Instance.new("ScreenGui", (gethui and gethui()) or game:GetService("CoreGui"))
sg.Name = "BrainrotBest_ESP"
sg.ResetOnSpawn = false

local Main = Instance.new("Frame", sg)
Main.Size = UDim2.fromOffset(200, 60)
Main.Position = UDim2.new(0.5, -100, 0.1, 0)
Main.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Main.BorderSizePixel = 0
Main.Active = true -- Necessário para o Drag funcionar
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)

local Stroke = Instance.new("UIStroke", Main)
Stroke.Color = Color3.fromRGB(255, 215, 0)
Stroke.Thickness = 2

local EspBtn = Instance.new("TextButton", Main)
EspBtn.Size = UDim2.new(0.9, 0, 0, 40)
EspBtn.Position = UDim2.new(0.05, 0, 0.15, 0)
EspBtn.Text = "BUSCAR MELHOR $/s"
EspBtn.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
EspBtn.TextColor3 = Color3.new(1, 1, 1)
EspBtn.Font = Enum.Font.GothamBold
EspBtn.TextSize = 13
Instance.new("UICorner", EspBtn)

-- // LÓGICA DO ESP MELHORADA
local EspAtivo = false

local function ParseValor(texto)
    if not texto then return 0 end
    -- Remove símbolos e foca no número e multiplicador
    local s = texto:gsub("%%", ""):gsub("%$", ""):gsub(",", ""):gsub("/s", ""):upper()
    local mult = 1
    if s:find("B") then mult = 1e9 s = s:gsub("B","") end
    if s:find("M") then mult = 1e6 s = s:gsub("M","") end
    if s:find("K") then mult = 1e3 s = s:gsub("K","") end
    return (tonumber(s) or 0) * mult
end

local function LimparESP()
    for _, v in pairs(workspace:GetDescendants()) do
        if v.Name == "BestBrainrotTag" or v.Name == "BestHighlight" then
            v:Destroy()
        end
    end
end

local function EscanearMelhor()
    if not EspAtivo then return end
    
    local melhorValor = -1
    local melhorObjeto = nil
    local nomeBrainrot = "Nenhum"
    local valorTexto = ""

    -- Varre as bases para achar o maior rendimento $/s
    for _, base in pairs(workspace.Server.Bases:GetChildren()) do
        local slots = base:FindFirstChild("Slots")
        if not slots then continue end
        
        for _, slot in pairs(slots:GetChildren()) do
            -- O Brainrot real fica dentro do slot, geralmente é um Model ou Part
            -- Procuramos pela Label que indica o ganho por segundo
            for _, v in pairs(slot:GetDescendants()) do
                if v:IsA("TextLabel") and (v.Text:find("/s") or v.Name:find("Value")) then
                    local valorNumerico = ParseValor(v.Text)
                    
                    if valorNumerico > melhorValor then
                        melhorValor = valorNumerico
                        valorTexto = v.Text
                        -- Tenta pegar o nome do Brainrot (geralmente o nome do Model pai)
                        melhorObjeto = slot:FindFirstChild("Handle") or slot:FindFirstChildWhichIsA("BasePart")
                        nomeBrainrot = slot.Name -- Ajuste se o nome estiver em outro lugar
                        
                        -- Se o slot tiver um modelo dentro, pega o nome dele
                        for _, child in pairs(slot:GetChildren()) do
                            if child:IsA("Model") then nomeBrainrot = child.Name end
                        end
                    end
                end
            end
        end
    end

    LimparESP()

    if melhorObjeto then
        -- Brilho Dourado
        local hl = Instance.new("Highlight", melhorObjeto)
        hl.Name = "BestHighlight"
        hl.FillColor = Color3.fromRGB(255, 215, 0)
        hl.OutlineColor = Color3.new(1, 1, 1)

        -- Placa com Nome e $/s
        local bbg = Instance.new("BillboardGui", melhorObjeto)
        bbg.Name = "BestBrainrotTag"
        bbg.Size = UDim2.new(0, 200, 0, 60)
        bbg.AlwaysOnTop = true
        bbg.ExtentsOffset = Vector3.new(0, 4, 0)

        local f = Instance.new("Frame", bbg)
        f.Size = UDim2.fromScale(1, 1)
        f.BackgroundColor3 = Color3.new(0,0,0)
        f.BackgroundTransparency = 0.4
        Instance.new("UICorner", f)

        local t1 = Instance.new("TextLabel", f)
        t1.Size = UDim2.fromScale(1, 0.5)
        t1.Text = nomeBrainrot:upper()
        t1.TextColor3 = Color3.new(1, 1, 1)
        t1.Font = Enum.Font.GothamBold
        t1.TextScaled = true
        t1.BackgroundTransparency = 1

        local t2 = Instance.new("TextLabel", f)
        t2.Size = UDim2.fromScale(1, 0.4)
        t2.Position = UDim2.fromScale(0, 0.5)
        t2.Text = "💰 " .. valorTexto
        t2.TextColor3 = Color3.fromRGB(255, 215, 0)
        t2.Font = Enum.Font.GothamBold
        t2.TextScaled = true
        t2.BackgroundTransparency = 1
    end
end

-- Toggle
EspBtn.MouseButton1Click:Connect(function()
    EspAtivo = not EspAtivo
    if EspAtivo then
        EspBtn.Text = "ESP: ATIVADO"
        EspBtn.TextColor3 = Color3.fromRGB(0, 255, 150)
        task.spawn(function()
            while EspAtivo do
                EscanearMelhor()
                task.wait(3)
            end
        end)
    else
        EspBtn.Text = "BUSCAR MELHOR $/s"
        EspBtn.TextColor3 = Color3.new(1, 1, 1)
        LimparESP()
    end
end)

-- SISTEMA DE ARRASTAR
local dragging, dragInput, dragStart, startPos
Main.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true; dragStart = input.Position; startPos = Main.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
        local delta = input.Position - dragStart
        Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)
