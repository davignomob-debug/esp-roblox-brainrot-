local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

local sg = Instance.new("ScreenGui", (gethui and gethui()) or game:GetService("CoreGui"))
sg.Name = "BestBrainrot_ESP"
sg.ResetOnSpawn = false

local highlights = {}
local labels = {}

-- converte texto de valor pra numero pra comparar
local function ParseValor(texto)
    if not texto then return 0 end
    local s = texto:gsub("%$", ""):gsub(",", "")
    local mult = 1
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

    local melhorValor = 0
    local melhorHandle = nil
    local melhorTexto = ""
    local melhorBase = ""

    -- acha o melhor brainrot de todo server
    local ok, bases = pcall(function() return workspace.Server.Bases:GetChildren() end)
    if not ok then return end

    for _, base in pairs(bases) do
        local slots = base:FindFirstChild("Slots")
        if not slots then continue end
        for _, slot in pairs(slots:GetChildren()) do
            local handle = slot:FindFirstChild("Handle")
            local collect = slot:FindFirstChild("Collect")
            if not handle or not collect then continue end
            -- slot ocupado = sem PlacePrompt
            if handle:FindFirstChild("PlacePrompt") then continue end
            local valueLabel = collect:FindFirstChild("Value", true)
            if not valueLabel then continue end
            local valor = ParseValor(valueLabel.Text)
            if valor > melhorValor then
                melhorValor = valor
                melhorHandle = handle
                melhorTexto = valueLabel.Text .. "/s"
                melhorBase = base.Name
            end
        end
    end

    if not melhorHandle then return end

    -- contorno amarelo no melhor brainrot
    local highlight = Instance.new("SelectionBox")
    highlight.Adornee = melhorHandle
    highlight.Color3 = Color3.fromRGB(255, 215, 0)
    highlight.LineThickness = 0.05
    highlight.SurfaceTransparency = 0.7
    highlight.SurfaceColor3 = Color3.fromRGB(255, 215, 0)
    highlight.Parent = sg
    table.insert(highlights, highlight)

    -- label em cima mostrando valor
    local bill = Instance.new("BillboardGui")
    bill.Adornee = melhorHandle
    bill.Size = UDim2.fromOffset(200, 50)
    bill.StudsOffset = Vector3.new(0, 3, 0)
    bill.AlwaysOnTop = true
    bill.Parent = sg
    table.insert(labels, bill)

    local frame = Instance.new("Frame", bill)
    frame.Size = UDim2.fromScale(1, 1)
    frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    frame.BackgroundTransparency = 0.4
    frame.BorderSizePixel = 0
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 6)

    local txt = Instance.new("TextLabel", frame)
    txt.Size = UDim2.fromScale(1, 0.55)
    txt.Position = UDim2.fromScale(0, 0)
    txt.Text = "🏆 MELHOR: " .. melhorTexto
    txt.TextColor3 = Color3.fromRGB(255, 215, 0)
    txt.BackgroundTransparency = 1
    txt.Font = Enum.Font.GothamBold
    txt.TextSize = 14
    txt.TextScaled = true

    local txt2 = Instance.new("TextLabel", frame)
    txt2.Size = UDim2.fromScale(1, 0.45)
    txt2.Position = UDim2.fromScale(0, 0.55)
    txt2.Text = melhorBase
    txt2.TextColor3 = Color3.fromRGB(200, 200, 200)
    txt2.BackgroundTransparency = 1
    txt2.Font = Enum.Font.Gotham
    txt2.TextSize = 12
    txt2.TextScaled = true
end

-- atualiza a cada 3 segundos
AtualizarESP()
task.spawn(function()
    while sg and sg.Parent do
        task.wait(3)
        AtualizarESP()
    end
end)

-- botao pra fechar
local CloseBtn = Instance.new("TextButton", sg)
CloseBtn.Size = UDim2.fromOffset(120, 30)
CloseBtn.Position = UDim2.new(0, 10, 0, 10)
CloseBtn.Text = "❌ Fechar ESP"
CloseBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
CloseBtn.TextColor3 = Color3.new(1,1,1)
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 13
CloseBtn.BorderSizePixel = 0
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)
CloseBtn.MouseButton1Click:Connect(function()
    LimparESP()
    sg:Destroy()
end)
