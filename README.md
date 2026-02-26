local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- // INTERFACE PRINCIPAL
local sg = Instance.new("ScreenGui", (gethui and gethui()) or game:GetService("CoreGui"))
sg.Name = "TechPerfect_ESP"
sg.ResetOnSpawn = false

-- Botão Arrastável Principal
local MainBtn = Instance.new("TextButton", sg)
MainBtn.Size = UDim2.fromOffset(180, 45)
MainBtn.Position = UDim2.new(0.5, -90, 0.1, 0)
MainBtn.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
MainBtn.Text = "BRAINROT ESP: OFF"
MainBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
MainBtn.Font = Enum.Font.GothamBold
MainBtn.TextSize = 14
MainBtn.AutoButtonColor = false

local Corner = Instance.new("UICorner", MainBtn)
Corner.CornerRadius = UDim.new(0, 8)

local Stroke = Instance.new("UIStroke", MainBtn)
Stroke.Color = Color3.fromRGB(255, 215, 0) -- Dourado
Stroke.Thickness = 2

-- // LÓGICA DO ESP (AJUSTADA)
local highlights = {}
local labels = {}
local EspAtivo = false

local function ParseValor(texto)
    if not texto then return 0 end
    local s = texto:gsub("%%", ""):gsub("%$", ""):gsub(",", ""):gsub("/s", "")
    local mult = 1
    s = s:upper()
    if s:find("B") then mult = 1e9 s = s:gsub("B","") end
    if s:find("M") then mult = 1e6 s = s:gsub("M","") end
    if s:find("K") then mult = 1e3 s = s:gsub("K","") end
    return (tonumber(s) or 0) * mult
end

local function LimparESP()
    for _, h in pairs(highlights) do pcall(function() h:Destroy() end) end
    for _, l in pairs(labels) do pcall(function() l:Destroy() end) end
    highlights = {}
    labels = {}
end

local function AtualizarESP()
    LimparESP()
    if not EspAtivo then return end

    local melhorValor = 0
    local melhorHandle = nil
    local melhorTexto = ""
    local melhorBase = ""

    local ok, bases = pcall(function() return workspace.Server.Bases:GetChildren() end)
    if not ok then return end

    for _, base in pairs(bases) do
        local slots = base:FindFirstChild("Slots")
        if not slots then continue end
        for _, slot in pairs(slots:GetChildren()) do
            local handle = slot:FindFirstChild("Handle")
            local collect = slot:FindFirstChild("Collect")
            if not handle or not collect then continue end
            
            if handle:FindFirstChild("PlacePrompt") then continue end -- Slot vazio
            
            local valueLabel = collect:FindFirstChild("Value", true)
            if valueLabel and valueLabel:IsA("TextLabel") then
                local valor = ParseValor(valueLabel.Text)
                if valor > melhorValor then
                    melhorValor = valor
                    melhorHandle = handle
                    melhorTexto = valueLabel.Text
                    melhorBase = base.Name
                end
            end
        end
    end

    if melhorHandle then
        -- Selection Box (Contorno)
        local sb = Instance.new("SelectionBox")
        sb.Adornee = melhorHandle
        sb.Color3 = Color3.fromRGB(255, 215, 0)
        sb.LineThickness = 0.04
        sb.SurfaceTransparency = 0.8
        sb.SurfaceColor3 = Color3.fromRGB(255, 215, 0)
        sb.Parent = melhorHandle
        table.insert(highlights, sb)

        -- Tag Flutuante
        local bill = Instance.new("BillboardGui")
        bill.Adornee = melhorHandle
        bill.Size = UDim2.fromOffset(180, 50)
        bill.StudsOffset = Vector3.new(0, 4, 0)
        bill.AlwaysOnTop = true
        bill.Parent = melhorHandle
        table.insert(labels, bill)

        local f = Instance.new("Frame", bill)
        f.Size = UDim2.fromScale(1, 1)
        f.BackgroundColor3 = Color3.new(0,0,0)
        f.BackgroundTransparency = 0.3
        Instance.new("UICorner", f)

        local t1 = Instance.new("TextLabel", f)
        t1.Size = UDim2.fromScale(1, 0.6)
        t1.Text = "⭐ MELHOR: " .. melhorTexto
        t1.TextColor3 = Color3.new(1, 0.84, 0)
        t1.Font = Enum.Font.GothamBold
        t1.TextScaled = true
        t1.BackgroundTransparency = 1

        local t2 = Instance.new("TextLabel", f)
        t2.Size = UDim2.fromScale(1, 0.4)
        t2.Position = UDim2.fromScale(0, 0.55)
        t2.Text = "Base: " .. melhorBase
        t2.TextColor3 = Color3.new(1, 1, 1)
        t2.Font = Enum.Font.Gotham
        t2.TextScaled = true
        t2.BackgroundTransparency = 1
    end
end

-- Ativa/Desativa
MainBtn.MouseButton1Click:Connect(function()
    EspAtivo = not EspAtivo
    if EspAtivo then
        MainBtn.Text = "BRAINROT ESP: ON"
        MainBtn.TextColor3 = Color3.fromRGB(0, 255, 150)
        Stroke.Color = Color3.fromRGB(0, 255, 150)
    else
        MainBtn.Text = "BRAINROT ESP: OFF"
        MainBtn.TextColor3 = Color3.new(1,1,1)
        Stroke.Color = Color3.fromRGB(255, 215, 0)
        LimparESP()
    end
end)

-- Loop de Atualização
task.spawn(function()
    while true do
        if EspAtivo then AtualizarESP() end
        task.wait(2.5)
    end
end)

-- // SISTEMA DE ARRASTAR (DRAG)
local dragging, dragInput, dragStart, startPos
MainBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true; dragStart = input.Position; startPos = MainBtn.Position
    end
end)
MainBtn.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then dragInput = input end
end)
UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        MainBtn.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)
