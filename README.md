-- Local Script para Laboratório Brainrot (Clash War GGPro)
-- Instruções: Coloque esse script em StarterPlayerScripts ou StarterGui no Roblox Studio (ou use um executor de scripts compatível)

-- === CONFIGURAÇÕES (AJUSTE SE O JOGO MUDAR A ESTRUTURA) ===
local BRAINROT_PASTA = workspace:WaitForChild("BrainrotBases") -- Pasta onde estão os Brainrots no workspace
local VALOR_RENDIMENTO = "MoneyPerSecond" -- Nome do NumberValue que armazena o rendimento por segundo
local COR_DESTAQUE = Color3.fromRGB(255,255,0) -- Cor amarela para o destaque
intervalo_atualizacao = 2 -- Tempo em segundos para atualizar o melhor Brainrot

-- === CRIAÇÃO DO DESTAQUE E DA GUI ===
-- Destaque amarelo para o melhor Brainrot
local destaque_melhor = Instance.new("Highlight")
destaque_melhor.Name = "MelhorBrainrotDestaque"
destaque_melhor.FillColor = COR_DESTAQUE
destaque_melhor.OutlineColor = COR_DESTAQUE
destaque_melhor.FillTransparency = 0.4
destaque_melhor.Parent = workspace

-- GUI para mostrar nome e rendimento
local gui_tela = Instance.new("ScreenGui")
gui_tela.Name = "BrainrotInfoGUI"
gui_tela.Parent = game.Players.LocalPlayer.PlayerGui

local frame_info = Instance.new("Frame")
frame_info.Size = UDim2.new(0,280,0,90)
frame_info.Position = UDim2.new(0.03,0,0.88,0)
frame_info.BackgroundColor3 = Color3.fromRGB(20,20,20)
frame_info.BackgroundTransparency = 0.3
frame_info.BorderSizePixel = 0
frame_info.Parent = gui_tela

local label_nome = Instance.new("TextLabel")
label_nome.Size = UDim2.new(1,0,0.4,0)
label_nome.Position = UDim2.new(0,0,0,0)
label_nome.Text = "Nenhum Brainrot encontrado"
label_nome.TextColor3 = Color3.fromRGB(255,255,255)
label_nome.BackgroundTransparency = 1
label_nome.Font = Enum.Font.SourceSansBold
label_nome.TextScaled = true
label_nome.Parent = frame_info

local label_rendimento = Instance.new("Text
