-- Espera o jogo carregar para evitar erro de interface sumindo
if not game:IsLoaded() then game.Loaded:Wait() end

local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")

-- Tenta pegar o PlayerGui (mais garantido que apareça)
local PlayerGui = LP:WaitForChild("PlayerGui")

-- Remove UI antiga se existir
if PlayerGui:FindFirstChild("BestBrainrot_UI") then
    PlayerGui.BestBrainrot_UI:Destroy()
end

-- // INTERFACE PRINCIPAL
local sg = Instance.new("ScreenGui", PlayerGui)
sg.Name = "BestBrainrot_UI"
sg.ResetOnSpawn = false -- Para a UI não sumir quando você morrer

local Main = Instance.new("Frame", sg)
Main.Size = UDim2.fromOffset(160, 70)
Main.Position = UDim2.new(0.5, -80, 0.2, 0) -- Um pouco mais para baixo
Main.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Main.BorderSizePixel = 0
Main.Active = true
Main.Draggable = true -- Ativa o arrastar nativo (mais simples)

local Corner = Instance.new("UICorner", Main)
Corner.CornerRadius = UDim.new(0, 10)

local Stroke = Instance.new("UIStroke", Main)
Stroke.Color = Color3.fromRGB(0, 255, 150)
Stroke.Thickness = 2

local EspBtn = Instance.new("TextButton", Main)
EspBtn.Size = UDim2.new(0.9, 0, 0, 40)
EspBtn.Position = UDim2.new(0.05, 0, 0.2, 0)
EspBtn.Text = "ATIVAR ESP"
EspBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
EspBtn.TextColor3 = Color3.new(1, 1, 1)
EspBtn.Font = Enum.Font.GothamBold
EspBtn.TextSize = 14
Instance.new("UICorner", EspBtn)

-- // LÓGICA DO ESP
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
        if v:IsA("TextLabel") and v.Text:find("%$") then
            local t = v.Text
            local valorStr = t:match("%$(%d+%.?%d*[KMB]?)/s")
            
            if valorStr then
                local num = tonumber(valorStr:match("%d+%.?%d*")) or 0
                if valorStr:find("K") then num = num * 1000 
                elseif valorStr:find("M") then num = num * 1000000 
                elseif valorStr:find("B") then num = num * 1000000000 end

                if num > melhorValor then
                    melhorValor = num
                    melhorObjeto = v:FindFirstAncestorWhichIsA("BasePart") or v.Parent
                    textoFinal = t
                end
            end
        end
    end

    LimparTags()

    if melhorObjeto and melhorObjeto:IsA("BasePart") then
        local hl = Instance.new("Highlight", melhorObjeto)
        hl.Name = "BestHighlight"
        hl.FillColor = Color3.fromRGB(0, 255, 150)
        
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
        tl.Font = Enum.Font.GothamBold
        tl.TextSize = 16
    end
end

EspBtn.MouseButton1Click:Connect(function()
    EspAtivo = not EspAtivo
    if EspAtivo then
        EspBtn.Text = "ESP: ON"
        EspBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 80)
        task.spawn(function()
            while EspAtivo do
                Escanear()
                task.wait(2)
            end
        end)
    else
        EspBtn.Text = "ATIVAR ESP"
        EspBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
        LimparTags()
    end
end)

print("Script Carregado! Se não vir a UI, verifique se o Xeno injetou corretamente.")
