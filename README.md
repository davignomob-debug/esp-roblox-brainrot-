local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local PLACE_ID = game.PlaceId

-- Ordem completa de raridades (as “outras” entram aqui também)
local RARITY_ORDER = {
	"Comum",
	"Incomum",
	"Raro",
	"Épico",
	"Lendário",
	"Mítico",
	"Secreto",
	"Stellar",
}

-- Só as que você passou lista (as outras ficam vazias)
local RARIDADES = {
	["Mítico"] = {"Tigroline", "Frutonni"},
	["Secreto"] = {"Garama", "Madudung", "La Vaca Saturnita", "Tralaledon", "Esok"},
	["Stellar"] = {"Strawberry Elephant", "Capitano Clash", "Warnini", "Meowl"},
}

local function rarityColor(r)
	if r == "Mítico" then return Color3.fromRGB(128, 0, 128) end
	if r == "Secreto" then return Color3.fromRGB(255, 215, 0) end
	if r == "Stellar" then return Color3.fromRGB(0, 191, 255) end
	if r == "Lendário" then return Color3.fromRGB(255, 140, 0) end
	if r == "Épico" then return Color3.fromRGB(170, 0, 255) end
	if r == "Raro" then return Color3.fromRGB(0, 120, 255) end
	if r == "Incomum" then return Color3.fromRGB(0, 200, 0) end
	return Color3.fromRGB(120, 120, 120) -- Comum / padrão
end

-- UI base
local old = playerGui:FindFirstChild("BrainrotSearchGui")
if old then old:Destroy() end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "BrainrotSearchGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 520, 0, 90)
mainFrame.Position = UDim2.new(0.5, -260, 0, 20)
mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 12)
local st = Instance.new("UIStroke", mainFrame)
st.Color = Color3.fromRGB(255, 215, 0)
st.Thickness = 2

local header = Instance.new("TextLabel")
header.Size = UDim2.new(1, 0, 0, 26)
header.BackgroundTransparency = 1
header.Text = "🔍 BUSCA POR RARIDADE"
header.TextColor3 = Color3.fromRGB(255, 215, 0)
header.TextScaled = true
header.Font = Enum.Font.GothamBold
header.Parent = mainFrame

local raritiesContainer = Instance.new("Frame")
raritiesContainer.Size = UDim2.new(1, -16, 0, 46)
raritiesContainer.Position = UDim2.new(0, 8, 0, 34)
raritiesContainer.BackgroundTransparency = 1
raritiesContainer.Parent = mainFrame

local grid = Instance.new("UIGridLayout")
grid.CellSize = UDim2.new(0, 120, 0, 20)
grid.CellPadding = UDim2.new(0, 8, 0, 6)
grid.SortOrder = Enum.SortOrder.LayoutOrder
grid.Parent = raritiesContainer

-- Menu flutuante (reutiliza)
local floatingMenu
local function openMenu(titleText, color, items)
	if floatingMenu then floatingMenu:Destroy() end

	floatingMenu = Instance.new("Frame")
	floatingMenu.Size = UDim2.new(0, 420, 0, 140)
	floatingMenu.Position = UDim2.new(0.5, -210, 0, 120)
	floatingMenu.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
	floatingMenu.BorderSizePixel = 0
	floatingMenu.ZIndex = 50
	floatingMenu.Parent = screenGui
	Instance.new("UICorner", floatingMenu).CornerRadius = UDim.new(0, 12)
	local s = Instance.new("UIStroke", floatingMenu)
	s.Color = color
	s.Thickness = 2

	local title = Instance.new("TextLabel")
	title.Size = UDim2.new(1, -40, 0, 26)
	title.Position = UDim2.new(0, 10, 0, 6)
	title.BackgroundTransparency = 1
	title.Text = titleText
	title.TextColor3 = Color3.fromRGB(255, 255, 255)
	title.TextScaled = true
	title.Font = Enum.Font.GothamBold
	title.ZIndex = 51
	title.Parent = floatingMenu

	local close = Instance.new("TextButton")
	close.Size = UDim2.new(0, 24, 0, 24)
	close.Position = UDim2.new(1, -30, 0, 6)
	close.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
	close.Text = "X"
	close.TextColor3 = Color3.fromRGB(255, 255, 255)
	close.TextScaled = true
	close.Font = Enum.Font.GothamBold
	close.BorderSizePixel = 0
	close.ZIndex = 52
	close.Parent = floatingMenu
	Instance.new("UICorner", close).CornerRadius = UDim.new(0, 6)
	close.MouseButton1Click:Connect(function()
		if floatingMenu then floatingMenu:Destroy(); floatingMenu=nil end
	end)

	local container = Instance.new("Frame")
	container.Size = UDim2.new(1, -20, 1, -44)
	container.Position = UDim2.new(0, 10, 0, 36)
	container.BackgroundTransparency = 1
	container.ZIndex = 51
	container.Parent = floatingMenu

	local layout = Instance.new("UIListLayout")
	layout.FillDirection = Enum.FillDirection.Horizontal
	layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
	layout.VerticalAlignment = Enum.VerticalAlignment.Top
	layout.Padding = UDim.new(0, 8)
	layout.Parent = container

	if not items or #items == 0 then
		local msg = Instance.new("TextLabel")
		msg.Size = UDim2.new(1, 0, 1, 0)
		msg.BackgroundTransparency = 1
		msg.Text = "Sem brainrots cadastrados pra essa raridade."
		msg.TextColor3 = Color3.fromRGB(200, 200, 200)
		msg.TextScaled = true
		msg.Font = Enum.Font.Gotham
		msg.ZIndex = 51
		msg.Parent = container
		return
	end

	for _, name in ipairs(items) do
		local b = Instance.new("TextButton")
		b.Size = UDim2.new(0, 130, 0, 34)
		b.BackgroundColor3 = Color3.fromRGB(55, 55, 55)
		b.Text = name
		b.TextColor3 = Color3.fromRGB(255, 255, 255)
		b.TextScaled = true
		b.Font = Enum.Font.GothamBold
		b.BorderSizePixel = 0
		b.ZIndex = 52
		b.Parent = container
		Instance.new("UICorner", b).CornerRadius = UDim.new(0, 8)

		b.MouseButton1Click:Connect(function()
			-- Aqui você ligaria sua lógica de “entrar em servers até achar”
			-- Mantive só o teleport simples pra não quebrar seu script atual.
			print("🎯 Selecionado:", name, "| raridade:", titleText)
			pcall(function()
				TeleportService:TeleportToPlaceId(PLACE_ID, player)
			end)
		end)
	end
end

-- Botões de raridade
for idx, raridade in ipairs(RARITY_ORDER) do
	local btn = Instance.new("TextButton")
	btn.LayoutOrder = idx
	btn.Size = UDim2.new(0, 120, 0, 20)
	btn.BackgroundColor3 = rarityColor(raridade)
	btn.Text = raridade
	btn.TextScaled = true
	btn.Font = Enum.Font.GothamBold
	btn.BorderSizePixel = 0
	btn.Parent = raritiesContainer
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)

	if raridade == "Secreto" then
		btn.TextColor3 = Color3.fromRGB(0, 0, 0)
	else
		btn.TextColor3 = Color3.fromRGB(255, 255, 255)
	end

	btn.MouseButton1Click:Connect(function()
		openMenu("Raridade: " .. raridade, rarityColor(raridade), RARIDADES[raridade] or {})
	end)
end

print("✅ Raridades carregadas (incluindo as outras).")
