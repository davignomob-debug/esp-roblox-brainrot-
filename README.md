local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")

-- // INTERFACE PRINCIPAL
local sg = Instance.new("ScreenGui", (gethui and gethui()) or CoreGui)
sg.Name = "BestBrainrot_UI"

local Main = Instance.new("Frame", sg)
Main.Size = UDim2.fromOffset(160, 70)
Main.Position = UDim2.new(0.5, -80, 0.1, 0)
Main.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
Main.BorderSizePixel = 0
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 8)

local Stroke = Instance.new("UIStroke", Main)
Stroke.Color = Color3.fromRGB(0, 255, 150)
Stroke.Thickness = 2

local EspBtn = Instance.new("TextButton", Main)
EspBtn.Size = UDim2.new(0.9, 0, 0, 40)
EspBtn.Position = UDim2.new(0.05, 0, 0.2, 0)
EspBtn.Text = "ATIVAR ESP"
EspBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
EspBtn.TextColor3 = Color3.new(1, 1, 1)
EspBtn.Font = Enum.Font.GothamBold
EspBtn.TextSize = 14
Instance.new("UICorner", EspBtn)

-- // LÓGICA DO ESP (MELHOR BRAINROT)
local EspAtivo = false

local function LimparTags()
    for _, v in pairs(workspace:GetDescendants()) do
        if v.Name == "BestBrainrotTag" or v.Name == "BestHighlight" then
            v:Destroy()
        end
    end
end

local function Escanear()
    if not EspAtivo then return end
    
    local melhorValor = -1
    local melhorObjeto = nil
    local textoFinal = ""

    for _, v in pairs(workspace:GetDescendants()) do
        if v:IsA("TextLabel") then
            local t = v.Text
            local valorStr = t:match("%$(%d+%.?%d*[KMB]?)/s")
            
            if valorStr then
                local num = tonumber(valorStr:match("%d+%.?%d*")) or 0
                if valorStr:find("K") then num = num * 1000 
                elseif valorStr:find("M") then num = num * 1000000 
                elseif valorStr:find("B") then num = num * 1000000000 end

                if num > melhorValor then
                    melhorValor = num
                    melhorObjeto = v.Parent:IsA("BasePart") and v.Parent or v:FindFirstAncestorWhichIsA("BasePart")
                    textoFinal = t
                end
            end
        end
    end

    LimparTags()

    if melhorObjeto then
        -- Criar Highlight
        local hl = Instance.new("Highlight", melhorObjeto)
        hl.Name = "BestHighlight"
        hl.FillColor = Color3.fromRGB(0, 255, 150)
        hl.FillAlpha = 0.4

        -- Criar Texto
        local bbg = Instance.new("BillboardGui", melhorObjeto)
        bbg.Name = "BestBrainrotTag"
        bbg.Size = UDim2.new(0, 200, 0, 50)
        bbg.AlwaysOnTop = true
        bbg.ExtentsOffset = Vector3.new(0, 3, 0)

        local tl = Instance.new("TextLabel", bbg)
        tl.Size = UDim2.new(1, 0, 1, 0)
        tl.BackgroundTransparency = 1
        tl.Text = "⭐ MELHOR ITEM ⭐\n" .. textoFinal
        tl.TextColor3 = Color3.fromRGB(0, 255, 150)
        tl.TextStrokeTransparency = 0
        tl.Font = Enum.Font.GothamBold
        tl.TextSize = 16
    end
end

EspBtn.MouseButton1Click:Connect(function()
    EspAtivo = not EspAtivo
    if EspAtivo then
        EspBtn.Text = "ESP: ON"
        EspBtn.BackgroundColor3 = Color3.fromRGB(0, 100, 60)
        -- Loop de atualização enquanto estiver ativo
        task.spawn(function()
            while EspAtivo do
                Escanear()
                task.wait(2)
            end
        end)
    else
        EspBtn.Text = "ATIVAR ESP"
        EspBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
        LimparTags()
    end
end)

-- // SISTEMA DE ARRASTAR (DRAG)
local dragging, dragInput, dragStart, startPos
Main.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true; dragStart = input.Position; startPos = Main.Position end end)
UserInputService.InputChanged:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then local delta = input.Position - dragStart; Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y) end end)
UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)
