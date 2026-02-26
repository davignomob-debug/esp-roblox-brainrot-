local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- // INTERFACE PRINCIPAL
local sg = Instance.new("ScreenGui", (gethui and gethui()) or game:GetService("CoreGui"))
sg.Name = "BrainrotUltimate_UI"
sg.ResetOnSpawn = false

local Main = Instance.new("Frame", sg)
Main.Size = UDim2.fromOffset(210, 70)
Main.Position = UDim2.new(0.5, -105, 0.1, 0)
Main.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Main.BorderSizePixel = 0
Main.Active = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 8)

-- Borda Neon
local Stroke = Instance.new("UIStroke", Main)
Stroke.Color = Color3.fromRGB(255, 215, 0)
Stroke.Thickness = 2

-- // BOTÃO DE FECHAR (X)
local CloseBtn = Instance.new("TextButton", Main)
CloseBtn.Size = UDim2.fromOffset(25, 25)
CloseBtn.Position = UDim2.new(1, -30, 0, 5)
CloseBtn.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.new(1, 1, 1)
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 14
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(1, 0) -- Botão redondo

-- Botão de Ativar ESP
local EspBtn = Instance.new("TextButton", Main)
EspBtn.Size = UDim2.new(0.85, 0, 0, 35)
EspBtn.Position = UDim2.new(0.075, 0, 0.4, 0)
EspBtn.Text = "BUSCAR MELHOR $/s"
EspBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
EspBtn.TextColor3 = Color3.new(1, 1, 1)
EspBtn.Font = Enum.Font.GothamBold
EspBtn.TextSize = 12
Instance.new("UICorner", EspBtn)

-- // LÓGICA DO ESP
local EspAtivo = false

local function ParseValor(texto)
    if not texto then return 0 end
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
    local melhorValor, melhorObjeto, nomeBrainrot, valorTexto = -1, nil, "Desconhecido", ""

    pcall(function()
        for _, base in pairs(workspace.Server.Bases:GetChildren()) do
            local slots = base:FindFirstChild("Slots")
            if not slots then continue end
            for _, slot in pairs(slots:GetChildren()) do
                for _, v in pairs(slot:GetDescendants()) do
                    if v:IsA("TextLabel") and v.Text:find("/s") then
                        local valorNumerico = ParseValor(v.Text)
                        if valorNumerico > melhorValor then
                            melhorValor = valorNumerico
                            valorTexto = v.Text
                            melhorObjeto = slot:FindFirstChild("Handle") or slot:FindFirstChildWhichIsA("BasePart")
                            nomeBrainrot = slot.Name
                            for _, child in pairs(slot:GetChildren()) do if child:IsA("Model") then nomeBrainrot = child.Name end end
                        end
                    end
                end
            end
        end
    end)

    LimparESP()
    if melhorObjeto then
        local hl = Instance.new("Highlight", melhorObjeto)
        hl.Name = "BestHighlight"
        hl.FillColor = Color3.fromRGB(255, 215, 0)

        local bbg = Instance.new("BillboardGui", melhorObjeto)
        bbg.Name = "BestBrainrotTag"
        bbg.Size = UDim2.new(0, 180, 0, 50)
        bbg.AlwaysOnTop = true
        bbg.ExtentsOffset = Vector3.new(0, 4, 0)

        local f = Instance.new("Frame", bbg)
        f.Size = UDim2.fromScale(1, 1)
        f.BackgroundColor3 = Color3.new(0,0,0)
        f.BackgroundTransparency = 0.4
        Instance.new("UICorner", f)

        local t1 = Instance.new("TextLabel", f)
        t1.Size = UDim2.fromScale(1, 0.5); t1.Text = nomeBrainrot:upper(); t1.TextColor3 = Color3.new(1,1,1)
        t1.Font = Enum.Font.GothamBold; t1.TextScaled = true; t1.BackgroundTransparency = 1

        local t2 = Instance.new("TextLabel", f)
        t2.Size = UDim2.fromScale(1, 0.4); t2.Position = UDim2.fromScale(0, 0.5); t2.Text = "💰 " .. valorTexto
        t2.TextColor3 = Color3.fromRGB(255, 215, 0); t2.Font = Enum.Font.GothamBold; t2.TextScaled = true; t2.BackgroundTransparency = 1
    end
end

-- // EVENTOS
EspBtn.MouseButton1Click:Connect(function()
    EspAtivo = not EspAtivo
    EspBtn.Text = EspAtivo and "ESP: ATIVADO" or "BUSCAR MELHOR $/s"
    EspBtn.TextColor3 = EspAtivo and Color3.fromRGB(0, 255, 150) or Color3.new(1,1,1)
    if EspAtivo then
        task.spawn(function() while EspAtivo do EscanearMelhor(); task.wait(3) end end)
    else
        LimparESP()
    end
end)

CloseBtn.MouseButton1Click:Connect(function()
    EspAtivo = false
    LimparESP()
    sg:Destroy()
end)

-- // DRAG SYSTEM
local dragging, dragStart, startPos
Main.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true; dragStart = i.Position; startPos = Main.Position end end)
UserInputService.InputChanged:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseMovement and dragging then local delta = i.Position - dragStart; Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y) end end)
UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)
