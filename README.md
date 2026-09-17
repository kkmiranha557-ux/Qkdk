-- LocalScript em:
-- StarterPlayer > StarterPlayerScripts

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Lighting = game:GetService("Lighting")

local localPlayer = Players.LocalPlayer
local playerGui = localPlayer:WaitForChild("PlayerGui")
local camera = workspace.CurrentCamera
local tpRemote = ReplicatedStorage:WaitForChild("TPPlayerRemote", 10)
if not tpRemote then
	warn("TPPlayerRemote não foi encontrado. Instale TPPlayerServer em ServerScriptService.")
end

-- Configurações alteráveis pelo painel
local settings = {
	esp = false,
	flying = false,
	box = true,
	skeleton = false,
	name = true,
	distance = true,
	lines = true,
		maxDistance = 1000,
		flightSpeed = 70,
			colorIndex = 1,
		lineOriginIndex = 1, -- 1=Baixo, 2=Cima, 3=Esquerda, 4=Direita
	}

local colors = {
	{ name = "Branco", value = Color3.fromRGB(255, 255, 255) },
	{ name = "Verde", value = Color3.fromRGB(80, 255, 120) },
	{ name = "Azul", value = Color3.fromRGB(80, 160, 255) },
	{ name = "Amarelo", value = Color3.fromRGB(255, 220, 70) },
	{ name = "Roxo", value = Color3.fromRGB(190, 100, 255) },
}

local lineOrigins = {
	{ name = "Baixo" },
	{ name = "Cima" },
	{ name = "Esquerda" },
	{ name = "Direita" },
}

local entries = {}
local panelOpen = true

-- Identidade visual do painel: alto contraste, neon e estados fáceis de ler.
local THEME = {
	background = Color3.fromRGB(7, 9, 20),
	panel = Color3.fromRGB(15, 18, 36),
	panelAlt = Color3.fromRGB(20, 24, 48),
	card = Color3.fromRGB(24, 29, 58),
	cardAlt = Color3.fromRGB(17, 21, 44),
	surface = Color3.fromRGB(31, 37, 72),
	surfaceHover = Color3.fromRGB(47, 54, 98),
	accent = Color3.fromRGB(139, 92, 246),
	accentDark = Color3.fromRGB(93, 55, 190),
	cyan = Color3.fromRGB(45, 211, 255),
	success = Color3.fromRGB(35, 190, 118),
	danger = Color3.fromRGB(255, 92, 125),
	text = Color3.fromRGB(248, 249, 255),
	muted = Color3.fromRGB(169, 178, 207),
	border = Color3.fromRGB(78, 91, 145),
}


local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ESPControlPanel"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Global
screenGui.Parent = playerGui

-- ////////////////////////////////////////////////////////////////////////////
-- SISTEMA DE KEY
-- ////////////////////////////////////////////////////////////////////////////
-- Altere somente esta linha para trocar a Key.
local REQUIRED_KEY = "157-TODA-VIDA"
local keyUnlocked = false

local keyFrame = Instance.new("Frame")
keyFrame.Name = "KeyFrame"
keyFrame.Size = UDim2.new(1, 0, 1, 0)
keyFrame.Position = UDim2.fromScale(0, 0)
keyFrame.BackgroundColor3 = THEME.background
keyFrame.BorderSizePixel = 0
keyFrame.ZIndex = 100
keyFrame.Parent = screenGui

local keyBox = Instance.new("Frame")
keyBox.Name = "KeyBox"
keyBox.Size = UDim2.fromOffset(380, 292)
keyBox.AnchorPoint = Vector2.new(0.5, 0.5)
keyBox.Position = UDim2.fromScale(0.5, 0.5)
keyBox.BackgroundColor3 = THEME.panel
keyBox.BorderSizePixel = 0
keyBox.ZIndex = 101
keyBox.Parent = keyFrame
do
	local c = Instance.new("UICorner")
	c.CornerRadius = UDim.new(0, 12)
	c.Parent = keyBox
end
do
	local st = Instance.new("UIStroke")
	st.Color = THEME.accent
	st.Thickness = 1
	st.Transparency = 0.15
	st.Parent = keyBox
end

local keyTitle = Instance.new("TextLabel")
keyTitle.Size = UDim2.new(1, -24, 0, 38)
keyTitle.Position = UDim2.fromOffset(12, 15)
keyTitle.BackgroundTransparency = 1
keyTitle.Font = Enum.Font.GothamBold
keyTitle.TextSize = 20
keyTitle.TextColor3 = THEME.text
keyTitle.Text = "ITALLO7 • ACESSO SEGURO"
keyTitle.ZIndex = 102
keyTitle.Parent = keyBox

local keyInfo = Instance.new("TextLabel")
keyInfo.Size = UDim2.new(1, -24, 0, 28)
keyInfo.Position = UDim2.fromOffset(12, 52)
keyInfo.BackgroundTransparency = 1
keyInfo.Font = Enum.Font.Gotham
keyInfo.TextSize = 12
keyInfo.TextColor3 = Color3.fromRGB(180, 180, 190)
keyInfo.Text = "Digite sua Key para desbloquear o painel de controle."
keyInfo.ZIndex = 102
keyInfo.Parent = keyBox

local keyInput = Instance.new("TextBox")
keyInput.Name = "KeyInput"
keyInput.Size = UDim2.new(1, -40, 0, 42)
keyInput.Position = UDim2.fromOffset(20, 88)
keyInput.BackgroundColor3 = THEME.surface
keyInput.BorderSizePixel = 0
keyInput.ClearTextOnFocus = false
keyInput.Font = Enum.Font.GothamSemibold
keyInput.TextSize = 14
keyInput.TextColor3 = Color3.fromRGB(245, 245, 250)
keyInput.PlaceholderText = "Digite a Key..."
keyInput.PlaceholderColor3 = Color3.fromRGB(145, 145, 155)
keyInput.Text = ""
keyInput.ZIndex = 102
keyInput.Parent = keyBox
do
	local c = Instance.new("UICorner")
	c.CornerRadius = UDim.new(0, 7)
	c.Parent = keyInput
end

local keyButton = Instance.new("TextButton")
keyButton.Name = "KeyButton"
keyButton.Size = UDim2.new(1, -40, 0, 38)
keyButton.Position = UDim2.fromOffset(20, 140)
keyButton.BackgroundColor3 = THEME.accentDark
keyButton.BorderSizePixel = 0
keyButton.Font = Enum.Font.GothamBold
keyButton.TextSize = 13
keyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
keyButton.Text = "ENTRAR"
keyButton.AutoButtonColor = false
keyButton.ZIndex = 102
keyButton.Parent = keyBox
do
	local c = Instance.new("UICorner")
	c.CornerRadius = UDim.new(0, 7)
	c.Parent = keyButton
end

-- Gerador de Key
local keyGeneratorButton = Instance.new("TextButton")
keyGeneratorButton.Name = "KeyGeneratorButton"
keyGeneratorButton.Size = UDim2.new(1, -40, 0, 38)
keyGeneratorButton.Position = UDim2.fromOffset(20, 184)
keyGeneratorButton.BackgroundColor3 = THEME.surface
keyGeneratorButton.BorderSizePixel = 0
keyGeneratorButton.Font = Enum.Font.GothamBold
keyGeneratorButton.TextSize = 13
keyGeneratorButton.TextColor3 = THEME.text
keyGeneratorButton.Text = "GERAR KEY"
keyGeneratorButton.AutoButtonColor = false
keyGeneratorButton.ZIndex = 102
keyGeneratorButton.Parent = keyBox

local keyGeneratorCorner = Instance.new("UICorner")
keyGeneratorCorner.CornerRadius = UDim.new(0, 7)
keyGeneratorCorner.Parent = keyGeneratorButton

local keyGeneratorStroke = Instance.new("UIStroke")
keyGeneratorStroke.Color = THEME.cyan
keyGeneratorStroke.Thickness = 1
keyGeneratorStroke.Transparency = 0.15
keyGeneratorStroke.Parent = keyGeneratorButton

local function generateKey()
    local characters = "ABCDEFGHJKLMNPQRSTUVWXYZ23456789"
    local parts = {}

    for partIndex = 1, 3 do
        local part = {}
        for characterIndex = 1, 4 do
            local randomIndex = math.random(1, #characters)
            table.insert(part, characters:sub(randomIndex, randomIndex))
        end
        table.insert(parts, table.concat(part))
    end

    return table.concat(parts, "-")
end

keyGeneratorButton.MouseEnter:Connect(function()
    TweenService:Create(keyGeneratorButton, TweenInfo.new(0.12), {
        BackgroundColor3 = THEME.surfaceHover
    }):Play()
end)

keyGeneratorButton.MouseLeave:Connect(function()
    TweenService:Create(keyGeneratorButton, TweenInfo.new(0.12), {
        BackgroundColor3 = THEME.surface
    }):Play()
end)

keyGeneratorButton.Activated:Connect(function()
    local newKey = generateKey()
    REQUIRED_KEY = newKey
    keyInput.Text = newKey
    keyStatus.Text = "Nova key gerada!"
    keyStatus.TextColor3 = THEME.cyan
    keyGeneratorButton.Text = "KEY GERADA"

    task.delay(1.5, function()
        if keyGeneratorButton and keyGeneratorButton.Parent then
            keyGeneratorButton.Text = "GERAR KEY"
        end
    end)
end)

local keyStatus = Instance.new("TextLabel")
keyStatus.Size = UDim2.new(1, -40, 0, 22)
keyStatus.Position = UDim2.fromOffset(20, 238)
keyStatus.BackgroundTransparency = 1
keyStatus.Font = Enum.Font.GothamSemibold
keyStatus.TextSize = 11
keyStatus.TextColor3 = THEME.danger
keyStatus.Text = ""
keyStatus.ZIndex = 102
keyStatus.Parent = keyBox


-- Utilitários da interface
local function addCorner(object, radius)
	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, radius or 8)
	corner.Parent = object
	return corner
end

local function addStroke(object, color, thickness)
	local stroke = Instance.new("UIStroke")
	stroke.Color = color or THEME.border
	stroke.Thickness = thickness or 1
	stroke.Transparency = 0.15
	stroke.Parent = object
	return stroke
end

local function addGradient(object, firstColor, secondColor, rotation)
	local gradient = Instance.new("UIGradient")
	gradient.Color = ColorSequence.new(firstColor, secondColor)
	gradient.Rotation = rotation or 90
	gradient.Parent = object
	return gradient
end

local function styleCard(object)
	addCorner(object, 10)
	addStroke(object, THEME.border, 1)
	addGradient(object, THEME.card, THEME.cardAlt, 90)
end

local function makeButton(parent, text, size, position)
	local button = Instance.new("TextButton")
	button.Size = size
	button.Position = position
	button.BackgroundColor3 = THEME.surface
	button.BorderSizePixel = 0
	button.Font = Enum.Font.GothamSemibold
	button.TextSize = 13
	button.TextColor3 = THEME.text
	button.Text = text
	button.AutoButtonColor = false
	button.Parent = parent
	addCorner(button, 7)
	local stroke = addStroke(button, THEME.border, 1)
	local padding = Instance.new("UIPadding")
	padding.PaddingLeft = UDim.new(0, 8)
	padding.PaddingRight = UDim.new(0, 8)
	padding.Parent = button

	button.MouseEnter:Connect(function()
		stroke.Color = THEME.cyan
		TweenService:Create(button, TweenInfo.new(0.12), {BackgroundColor3 = THEME.surfaceHover}):Play()
	end)
	button.MouseLeave:Connect(function()
		stroke.Color = THEME.border
		TweenService:Create(button, TweenInfo.new(0.12), {BackgroundColor3 = button:GetAttribute("StateColor") or THEME.surface}):Play()
	end)
	button:SetAttribute("StateColor", THEME.surface)
	return button
end

local function setButtonColor(button, color)
	button:SetAttribute("StateColor", color)
	button.BackgroundColor3 = color
end

-- Painel principal
local panel = Instance.new("Frame")
panel.Name = "MainPanel"
panel.Size = UDim2.fromOffset(390, 470)
panel.AnchorPoint = Vector2.new(0.5, 0.5)
panel.Position = UDim2.fromScale(0.5, 0.5)
panel.BackgroundColor3 = THEME.panel
panel.BackgroundTransparency = 0.02
panel.BorderSizePixel = 0
panel.Parent = screenGui
panel.Visible = false
addCorner(panel, 10)
addStroke(panel, THEME.accent, 1)
addGradient(panel, THEME.panel, THEME.panelAlt, 90)

-- Escala interna: conserva o layout e permite redimensionar sem deformar os controles.
local panelScale = Instance.new("UIScale")
panelScale.Name = "ResponsiveScale"
panelScale.Scale = 0.65
panelScale.Parent = panel

local resizeHandle = Instance.new("TextButton")
resizeHandle.Name = "ResizeHandle"
resizeHandle.Size = UDim2.fromOffset(24, 24)
resizeHandle.AnchorPoint = Vector2.new(1, 1)
resizeHandle.Position = UDim2.new(1, -5, 1, -5)
resizeHandle.BackgroundTransparency = 1
resizeHandle.BorderSizePixel = 0
resizeHandle.Font = Enum.Font.GothamBold
resizeHandle.TextSize = 18
resizeHandle.TextColor3 = THEME.cyan
resizeHandle.Text = "◢"
resizeHandle.AutoButtonColor = false
resizeHandle.ZIndex = 250
resizeHandle.Parent = panel

local header = Instance.new("Frame")
header.Size = UDim2.new(1, 0, 0, 64)
header.BackgroundColor3 = THEME.panelAlt
header.BorderSizePixel = 0
header.Parent = panel
addCorner(header, 10)
addGradient(header, THEME.accentDark, THEME.panelAlt, 0)

local headerFix = Instance.new("Frame")
headerFix.Size = UDim2.new(1, 0, 0, 12)
headerFix.Position = UDim2.new(0, 0, 1, -12)
headerFix.BackgroundColor3 = header.BackgroundColor3
headerFix.BorderSizePixel = 0
headerFix.Parent = header

local tabsBar = Instance.new("Frame")
tabsBar.Name = "TabsBar"
tabsBar.Size = UDim2.new(1, -28, 0, 42)
tabsBar.Position = UDim2.fromOffset(14, 68)
tabsBar.BackgroundColor3 = THEME.cardAlt
tabsBar.BorderSizePixel = 0
tabsBar.Parent = panel
styleCard(tabsBar)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -145, 0, 27)
title.Position = UDim2.fromOffset(17, 8)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.TextSize = 17
title.TextXAlignment = Enum.TextXAlignment.Left
title.TextColor3 = THEME.text
title.Text = "Itallo7"
title.Parent = header

local subtitle = Instance.new("TextLabel")
subtitle.Size = UDim2.new(1, -205, 0, 18)
subtitle.Position = UDim2.fromOffset(18, 34)
subtitle.BackgroundTransparency = 1
subtitle.Font = Enum.Font.Gotham
subtitle.TextSize = 10
subtitle.TextXAlignment = Enum.TextXAlignment.Left
subtitle.TextColor3 = THEME.muted
subtitle.Text = "RIGHTSHIFT: ABRIR/FECHAR  •  ARRASTE ◢ PARA REDIMENSIONAR"
subtitle.Parent = header

local headerStatus = Instance.new("TextLabel")
headerStatus.Size = UDim2.fromOffset(125, 18)
headerStatus.Position = UDim2.new(1, -180, 0, 7)
headerStatus.BackgroundTransparency = 1
headerStatus.Font = Enum.Font.GothamBold
headerStatus.TextSize = 10
headerStatus.TextColor3 = THEME.cyan
headerStatus.Text = "●  ONLINE"
headerStatus.Parent = header

-- Perfil compacto ao lado do título Itallo7: avatar e nome do jogador.
local profileAvatar = Instance.new("ImageLabel")
profileAvatar.Name = "HeaderAvatar"
profileAvatar.Size = UDim2.fromOffset(28, 28)
profileAvatar.Position = UDim2.fromOffset(112, 7)
profileAvatar.BackgroundColor3 = THEME.surface
profileAvatar.BorderSizePixel = 0
profileAvatar.ScaleType = Enum.ScaleType.Crop
profileAvatar.Parent = header
addCorner(profileAvatar, 14)
addStroke(profileAvatar, THEME.cyan, 1)

task.spawn(function()
	local ok, image = pcall(function()
		return Players:GetUserThumbnailAsync(localPlayer.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size48x48)
	end)
	if ok and image then
		profileAvatar.Image = image
	end
end)

local profileButton = makeButton(header, "Perfil: " .. localPlayer.DisplayName, UDim2.fromOffset(112, 24), UDim2.fromOffset(144, 9))
profileButton.TextSize = 9
profileButton.TextXAlignment = Enum.TextXAlignment.Left
profileButton.TextTruncate = Enum.TextTruncate.AtEnd
local showingUsername = false
profileButton.MouseButton1Click:Connect(function()
	showingUsername = not showingUsername
	profileButton.Text = showingUsername and ("@" .. localPlayer.Name) or ("Perfil: " .. localPlayer.DisplayName)
end)

local minimizeButton = makeButton(header, "−", UDim2.fromOffset(35, 30), UDim2.new(1, -45, 0, 17))
minimizeButton.Font = Enum.Font.GothamBold
minimizeButton.TextSize = 20

local content = Instance.new("ScrollingFrame")
content.Name = "Content"
content.Size = UDim2.new(1, -28, 1, -132)
content.Position = UDim2.fromOffset(14, 118)
content.BackgroundTransparency = 1
content.BorderSizePixel = 0
content.CanvasSize = UDim2.fromOffset(0, 650)
content.ScrollBarThickness = 5
content.ScrollBarImageColor3 = THEME.accent
content.ScrollingDirection = Enum.ScrollingDirection.Y
content.Active = true
content.ClipsDescendants = true
content.Parent = panel

-- Abas compactas do painel.
local espTab = makeButton(tabsBar, "ESP", UDim2.fromOffset(48, 32), UDim2.fromOffset(6, 5))
local aimTab = makeButton(tabsBar, "AIM", UDim2.fromOffset(48, 32), UDim2.fromOffset(57, 5))
local flightTab = makeButton(tabsBar, "VOAR", UDim2.fromOffset(48, 32), UDim2.fromOffset(108, 5))
local tpTab = makeButton(tabsBar, "TP", UDim2.fromOffset(48, 32), UDim2.fromOffset(159, 5))
local godTab = makeButton(tabsBar, "MODS", UDim2.fromOffset(48, 32), UDim2.fromOffset(210, 5))
local fpsTab = makeButton(tabsBar, "FPS", UDim2.fromOffset(48, 32), UDim2.fromOffset(261, 5))

local espTitle = Instance.new("TextLabel")
espTitle.Size = UDim2.fromOffset(350, 22)
espTitle.Position = UDim2.fromOffset(0, 42)
espTitle.BackgroundTransparency = 1
espTitle.Font = Enum.Font.GothamBold
espTitle.TextSize = 12
espTitle.TextXAlignment = Enum.TextXAlignment.Left
espTitle.TextColor3 = THEME.cyan
espTitle.Text = "VISÃO E RASTREAMENTO"
espTitle.Parent = content

local espButton = makeButton(content, "ESP: DESLIGADO", UDim2.fromOffset(350, 34), UDim2.fromOffset(0, 70))

local function makeOptionButton(text, y)
	return makeButton(content, text, UDim2.fromOffset(350, 30), UDim2.fromOffset(0, y + 67))
end

local boxButton = makeOptionButton("Caixa: LIGADA", 45)
local skeletonButton = makeOptionButton("Skeleton: DESLIGADO", 80)
local nameButton = makeOptionButton("Nome: LIGADO", 115)
local distanceButton = makeOptionButton("Distância: LIGADA", 150)
local linesButton = makeOptionButton("Linhas: LIGADAS", 185)
local colorButton = makeOptionButton("Cor: Branco", 220)
local lineOriginButton = makeOptionButton("Linha sai: Baixo", 255)

local maxDistanceLabel = Instance.new("TextLabel")
maxDistanceLabel.Size = UDim2.fromOffset(350, 24)
maxDistanceLabel.Position = UDim2.fromOffset(0, 366)
maxDistanceLabel.BackgroundTransparency = 1
maxDistanceLabel.Font = Enum.Font.GothamSemibold
maxDistanceLabel.TextSize = 12
maxDistanceLabel.TextXAlignment = Enum.TextXAlignment.Left
maxDistanceLabel.TextColor3 = Color3.fromRGB(205, 205, 215)
maxDistanceLabel.Parent = content

local distanceMinus = makeButton(content, "−", UDim2.fromOffset(42, 30), UDim2.fromOffset(0, 392))
local distanceValue = makeButton(content, "1000 studs", UDim2.fromOffset(262, 30), UDim2.fromOffset(48, 392))
local distancePlus = makeButton(content, "+", UDim2.fromOffset(42, 30), UDim2.fromOffset(306, 392))

local godSection = Instance.new("Frame")
godSection.Name = "GodModeSection"
godSection.Size = UDim2.fromOffset(350, 535)
godSection.Position = UDim2.fromOffset(0, 42)
godSection.BackgroundColor3 = THEME.card
godSection.BorderSizePixel = 0
godSection.Parent = content
styleCard(godSection)

local godTitle = Instance.new("TextLabel")
godTitle.Size = UDim2.new(1, -24, 0, 30)
godTitle.Position = UDim2.fromOffset(12, 8)
godTitle.BackgroundTransparency = 1
godTitle.Font = Enum.Font.GothamBold
godTitle.TextSize = 15
godTitle.TextXAlignment = Enum.TextXAlignment.Left
godTitle.TextColor3 = THEME.text
godTitle.Text = "MODERAÇÃO & UTILITÁRIOS"
godTitle.Parent = godSection

local godButton = makeButton(godSection, "God Mod: DESLIGADO", UDim2.new(1, -24, 0, 36), UDim2.fromOffset(12, 48))
local noClipButton = makeButton(godSection, "Atravessar paredes: DESLIGADO", UDim2.new(1, -24, 0, 32), UDim2.fromOffset(12, 128))
local gokuSkinButton = makeButton(godSection, "PUXAR SKIN GOKU SUKUNA", UDim2.new(1, -24, 0, 32), UDim2.fromOffset(12, 168))

local timeTitle = Instance.new("TextLabel")
timeTitle.Size = UDim2.new(1, -24, 0, 22)
timeTitle.Position = UDim2.fromOffset(12, 212)
timeTitle.BackgroundTransparency = 1
timeTitle.Font = Enum.Font.GothamSemibold
timeTitle.TextSize = 12
timeTitle.TextXAlignment = Enum.TextXAlignment.Left
timeTitle.TextColor3 = Color3.fromRGB(205, 205, 215)
timeTitle.Text = "CONTROLE DE TEMPO"
timeTitle.Parent = godSection

local dayButton = makeButton(godSection, "DIA", UDim2.fromOffset(159, 32), UDim2.fromOffset(12, 236))
local nightButton = makeButton(godSection, "NOITE", UDim2.fromOffset(159, 32), UDim2.fromOffset(179, 236))

local godInfo = Instance.new("TextLabel")
godInfo.Size = UDim2.new(1, -24, 0, 70)
godInfo.Position = UDim2.fromOffset(12, 278)
godInfo.BackgroundTransparency = 1
godInfo.Font = Enum.Font.Gotham
godInfo.TextSize = 11
godInfo.TextWrapped = true
godInfo.TextXAlignment = Enum.TextXAlignment.Left
godInfo.TextYAlignment = Enum.TextYAlignment.Top
godInfo.TextColor3 = Color3.fromRGB(175, 175, 185)
godInfo.Text = "Apenas o UserId autorizado no Script do servidor pode usar estas funções."
godInfo.Parent = godSection

-- Controle de velocidade do personagem na aba MODE.
local characterSpeed = 16
local speedStep = 8
local minimumCharacterSpeed = 8
local maximumCharacterSpeed = 200

local speedTitle = Instance.new("TextLabel")
speedTitle.Size = UDim2.new(1, -24, 0, 22)
speedTitle.Position = UDim2.fromOffset(12, 354)
speedTitle.BackgroundTransparency = 1
speedTitle.Font = Enum.Font.GothamSemibold
speedTitle.TextSize = 12
speedTitle.TextXAlignment = Enum.TextXAlignment.Left
speedTitle.TextColor3 = Color3.fromRGB(205, 205, 215)
speedTitle.Text = "VELOCIDADE DO PERSONAGEM"
speedTitle.Parent = godSection

local speedDownButton = makeButton(godSection, "DIMINUIR", UDim2.fromOffset(124, 32), UDim2.fromOffset(12, 382))
local speedUpButton = makeButton(godSection, "AUMENTAR", UDim2.fromOffset(124, 32), UDim2.fromOffset(142, 382))
local characterSpeedValue = makeButton(godSection, "Velocidade: 16", UDim2.new(1, -24, 0, 30), UDim2.fromOffset(12, 420))

local function getHumanoid()
	local character = localPlayer.Character
	return character and character:FindFirstChildOfClass("Humanoid")
end

local function applyCharacterSpeed()
	local humanoid = getHumanoid()
	if humanoid then
		humanoid.WalkSpeed = characterSpeed
	end
	characterSpeedValue.Text = "Velocidade: " .. tostring(characterSpeed)
end

local function changeCharacterSpeed(amount)
	characterSpeed = math.clamp(characterSpeed + amount, minimumCharacterSpeed, maximumCharacterSpeed)
	applyCharacterSpeed()
end

speedDownButton.MouseButton1Click:Connect(function()
	changeCharacterSpeed(-speedStep)
end)

speedUpButton.MouseButton1Click:Connect(function()
	changeCharacterSpeed(speedStep)
end)

localPlayer.CharacterAdded:Connect(function(character)
	character:WaitForChild("Humanoid", 5)
	task.wait(0.1)
	applyCharacterSpeed()
end)

-- Sistema Anti-Lag: desativa efeitos visuais pesados e pode restaurá-los.
local antiLagEnabled = false
local antiLagOriginalStates = {}
local antiLagConnections = {}

local function isAntiLagEffect(object)
	return object:IsA("ParticleEmitter")
		or object:IsA("Trail")
		or object:IsA("Beam")
		or object:IsA("Smoke")
		or object:IsA("Fire")
		or object:IsA("Sparkles")
		or object:IsA("PointLight")
		or object:IsA("SpotLight")
		or object:IsA("SurfaceLight")
		or object:IsA("PostEffect")
end

local function setAntiLagEffect(object, disabled)
	if not isAntiLagEffect(object) then return end
	if disabled then
		if antiLagOriginalStates[object] == nil then
			antiLagOriginalStates[object] = object.Enabled
		end
		object.Enabled = false
	elseif antiLagOriginalStates[object] ~= nil then
		object.Enabled = antiLagOriginalStates[object]
		antiLagOriginalStates[object] = nil
	end
end

local function applyAntiLag(enabled)
	antiLagEnabled = enabled
	if enabled then
		for _, root in ipairs({workspace, Lighting}) do
			for _, object in ipairs(root:GetDescendants()) do
				setAntiLagEffect(object, true)
			end
		end
		for _, connection in ipairs(antiLagConnections) do connection:Disconnect() end
		table.clear(antiLagConnections)
		table.insert(antiLagConnections, workspace.DescendantAdded:Connect(function(object)
			if antiLagEnabled then setAntiLagEffect(object, true) end
		end))
		table.insert(antiLagConnections, Lighting.DescendantAdded:Connect(function(object)
			if antiLagEnabled then setAntiLagEffect(object, true) end
		end))
	else
		for object in pairs(antiLagOriginalStates) do
			if object and object.Parent then setAntiLagEffect(object, false) end
		end
		for _, connection in ipairs(antiLagConnections) do connection:Disconnect() end
		table.clear(antiLagConnections)
	end
end

local antiLagTitle = Instance.new("TextLabel")
antiLagTitle.Size = UDim2.new(1, -24, 0, 22)
antiLagTitle.Position = UDim2.fromOffset(12, 458)
antiLagTitle.BackgroundTransparency = 1
antiLagTitle.Font = Enum.Font.GothamSemibold
antiLagTitle.TextSize = 12
antiLagTitle.TextXAlignment = Enum.TextXAlignment.Left
antiLagTitle.TextColor3 = Color3.fromRGB(205, 205, 215)
antiLagTitle.Text = "DESEMPENHO GRÁFICO"
antiLagTitle.Parent = godSection

local antiLagButton = makeButton(godSection, "ANTI-LAG: DESLIGADO", UDim2.new(1, -24, 0, 32), UDim2.fromOffset(12, 486))

antiLagButton.MouseButton1Click:Connect(function()
	applyAntiLag(not antiLagEnabled)
	antiLagButton.Text = antiLagEnabled and "ANTI-LAG: LIGADO" or "ANTI-LAG: DESLIGADO"
	setButtonColor(antiLagButton, antiLagEnabled and THEME.success or THEME.surface)
end)

godSection.Visible = false

-- Painel de monitoramento de FPS.
local fpsSection = Instance.new("Frame")
fpsSection.Name = "FPSSection"
fpsSection.Size = UDim2.fromOffset(350, 170)
fpsSection.Position = UDim2.fromOffset(0, 42)
fpsSection.BackgroundColor3 = THEME.card
fpsSection.BorderSizePixel = 0
fpsSection.Parent = content
styleCard(fpsSection)

local fpsTitle = Instance.new("TextLabel")
fpsTitle.Size = UDim2.new(1, -24, 0, 30)
fpsTitle.Position = UDim2.fromOffset(12, 8)
fpsTitle.BackgroundTransparency = 1
fpsTitle.Font = Enum.Font.GothamBold
fpsTitle.TextSize = 15
fpsTitle.TextXAlignment = Enum.TextXAlignment.Left
fpsTitle.TextColor3 = THEME.text
fpsTitle.Text = "MONITOR DE FPS"
fpsTitle.Parent = fpsSection

local fpsInfo = Instance.new("TextLabel")
fpsInfo.Size = UDim2.new(1, -24, 0, 92)
fpsInfo.Position = UDim2.fromOffset(12, 90)
fpsInfo.BackgroundTransparency = 1
fpsInfo.Font = Enum.Font.GothamSemibold
fpsInfo.TextSize = 14
fpsInfo.TextXAlignment = Enum.TextXAlignment.Left
fpsInfo.TextYAlignment = Enum.TextYAlignment.Top
fpsInfo.TextColor3 = Color3.fromRGB(205, 255, 215)
fpsInfo.Text = "Atual: -- FPS\nMédia: -- FPS\nMínimo: -- FPS"
fpsInfo.Parent = fpsSection

local fpsToggleButton = makeButton(fpsSection, "FPS: DESLIGADO", UDim2.new(1, -24, 0, 32), UDim2.fromOffset(12, 48))

local fpsCounter = Instance.new("TextLabel")
fpsCounter.Name = "FPSCounter"
fpsCounter.Size = UDim2.fromOffset(150, 30)
fpsCounter.Position = UDim2.new(1, -165, 0, 12)
fpsCounter.BackgroundColor3 = THEME.panel
fpsCounter.BackgroundTransparency = 0.15
fpsCounter.BorderSizePixel = 0
fpsCounter.Font = Enum.Font.GothamBold
fpsCounter.TextSize = 14
fpsCounter.TextColor3 = THEME.cyan
fpsCounter.Text = "FPS: --"
fpsCounter.Parent = screenGui
addCorner(fpsCounter, 6)
addStroke(fpsCounter, Color3.fromRGB(80, 130, 95), 1)
fpsCounter.Visible = false

local fpsEnabled = false

local fpsFrames = 0
local fpsElapsed = 0
local fpsCurrent = 0
local fpsAverage = 0
local fpsMinimum = math.huge

local function updateFPS(deltaTime)
	if deltaTime <= 0 then return end
	fpsFrames += 1
	fpsElapsed += deltaTime
	fpsCurrent = math.floor((1 / deltaTime) + 0.5)
	fpsMinimum = math.min(fpsMinimum, fpsCurrent)
	if fpsElapsed >= 0.5 then
		fpsAverage = math.floor((fpsFrames / fpsElapsed) + 0.5)
		fpsFrames = 0
		fpsElapsed = 0
	end
	fpsCounter.Text = string.format("FPS: %d", fpsCurrent)
	fpsInfo.Text = string.format("Atual: %d FPS\nMédia: %d FPS\nMínimo: %d FPS", fpsCurrent, fpsAverage, fpsMinimum == math.huge and fpsCurrent or fpsMinimum)
end

local flyButton = makeButton(content, "Voar: DESLIGADO", UDim2.fromOffset(236, 30), UDim2.fromOffset(12, 90))

-- Coluna de voo ao lado das funções do ESP.
local flightSection = Instance.new("Frame")
flightSection.Name = "FlightSection"
flightSection.Size = UDim2.fromOffset(350, 170)
flightSection.Position = UDim2.fromOffset(0, 42)
flightSection.BackgroundColor3 = THEME.card
flightSection.BorderSizePixel = 0
flightSection.Parent = content
styleCard(flightSection)

local flightTitle = Instance.new("TextLabel")
flightTitle.Size = UDim2.new(1, -24, 0, 30)
flightTitle.Position = UDim2.fromOffset(12, 8)
flightTitle.BackgroundTransparency = 1
flightTitle.Font = Enum.Font.GothamBold
flightTitle.TextSize = 15
flightTitle.TextXAlignment = Enum.TextXAlignment.Left
flightTitle.TextColor3 = THEME.text
flightTitle.Text = "CONTROLE DE VOO"
flightTitle.Parent = flightSection

flyButton.Parent = flightSection
flyButton.Size = UDim2.new(1, -24, 0, 34)
flyButton.Position = UDim2.fromOffset(12, 48)

local flightInfo = Instance.new("TextLabel")
flightInfo.Size = UDim2.new(1, -24, 0, 58)
flightInfo.Position = UDim2.fromOffset(12, 92)
flightInfo.BackgroundTransparency = 1
flightInfo.Font = Enum.Font.Gotham
flightInfo.TextSize = 11
flightInfo.TextWrapped = true
flightInfo.TextXAlignment = Enum.TextXAlignment.Left
flightInfo.TextYAlignment = Enum.TextYAlignment.Top
flightInfo.TextColor3 = Color3.fromRGB(175, 175, 185)
flightInfo.Text = "Use o joystick ou WASD.\nA câmera define a direção do voo."
flightInfo.Parent = flightSection

local aimSection = Instance.new("Frame")
aimSection.Name = "AimSection"
aimSection.Size = UDim2.fromOffset(350, 180)
aimSection.Position = UDim2.fromOffset(0, 42)
aimSection.BackgroundColor3 = THEME.card
aimSection.BorderSizePixel = 0
aimSection.Parent = content
styleCard(aimSection)

local aimTitle = Instance.new("TextLabel")
aimTitle.Size = UDim2.new(1, -24, 0, 30)
aimTitle.Position = UDim2.fromOffset(12, 8)
aimTitle.BackgroundTransparency = 1
aimTitle.Font = Enum.Font.GothamBold
aimTitle.TextSize = 15
aimTitle.TextXAlignment = Enum.TextXAlignment.Left
aimTitle.TextColor3 = THEME.text
aimTitle.Text = "CONTROLE DE AIM / FOV"
aimTitle.Parent = aimSection

local fovEnabled = false
local fovRadius = 150
local fovToggleButton = makeButton(aimSection, "FOV: DESLIGADO", UDim2.new(1, -24, 0, 34), UDim2.fromOffset(12, 48))
local fovMinus = makeButton(aimSection, "−", UDim2.fromOffset(42, 30), UDim2.fromOffset(12, 92))
local fovValue = makeButton(aimSection, "150 px", UDim2.fromOffset(228, 30), UDim2.fromOffset(60, 92))
local fovPlus = makeButton(aimSection, "+", UDim2.fromOffset(42, 30), UDim2.fromOffset(298, 92))
local fovInfo = Instance.new("TextLabel")
fovInfo.Size = UDim2.new(1, -24, 0, 38)
fovInfo.Position = UDim2.fromOffset(12, 132)
fovInfo.BackgroundTransparency = 1
fovInfo.Font = Enum.Font.Gotham
fovInfo.TextSize = 11
fovInfo.TextWrapped = true
fovInfo.TextXAlignment = Enum.TextXAlignment.Left
fovInfo.TextColor3 = Color3.fromRGB(175, 175, 185)
fovInfo.Text = "Círculo visual para orientar a mira."
fovInfo.Parent = aimSection

local fovCircle = Instance.new("Frame")
fovCircle.Name = "FOVCircle"
fovCircle.AnchorPoint = Vector2.new(0.5, 0.5)
fovCircle.Position = UDim2.fromScale(0.5, 0.5)
fovCircle.Size = UDim2.fromOffset(fovRadius * 2, fovRadius * 2)
fovCircle.BackgroundTransparency = 1
fovCircle.BorderSizePixel = 0
fovCircle.Visible = false
fovCircle.Parent = screenGui
addCorner(fovCircle, fovRadius)
addStroke(fovCircle, Color3.fromRGB(255, 255, 255), 2)

local fovCenterDot = Instance.new("Frame")
fovCenterDot.Name = "FOVCenterDot"
fovCenterDot.AnchorPoint = Vector2.new(0.5, 0.5)
fovCenterDot.Position = UDim2.fromScale(0.5, 0.5)
fovCenterDot.Size = UDim2.fromOffset(8, 8)
fovCenterDot.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
fovCenterDot.BorderSizePixel = 0
fovCenterDot.Visible = false
fovCenterDot.Parent = screenGui
addCorner(fovCenterDot, 8)
aimSection.Visible = false

local tpSection = Instance.new("Frame")
tpSection.Name = "TPPlayerSection"
tpSection.Size = UDim2.fromOffset(350, 270)
tpSection.Position = UDim2.fromOffset(0, 42)
tpSection.BackgroundColor3 = THEME.card
tpSection.BorderSizePixel = 0
tpSection.Parent = content
styleCard(tpSection)

local tpTitle = Instance.new("TextLabel")
tpTitle.Size = UDim2.new(1, -24, 0, 28)
tpTitle.Position = UDim2.fromOffset(12, 8)
tpTitle.BackgroundTransparency = 1
tpTitle.Font = Enum.Font.GothamBold
tpTitle.TextSize = 15
tpTitle.TextXAlignment = Enum.TextXAlignment.Left
tpTitle.TextColor3 = THEME.text
tpTitle.Text = "ESCOLHER JOGADOR"
tpTitle.Parent = tpSection

local selectedPlayer
local playerList = Instance.new("ScrollingFrame")
playerList.Size = UDim2.new(1, -24, 0, 145)
playerList.Position = UDim2.fromOffset(12, 42)
playerList.BackgroundColor3 = THEME.cardAlt
playerList.BorderSizePixel = 0
playerList.ScrollBarThickness = 4
playerList.CanvasSize = UDim2.fromOffset(0, 0)
playerList.Parent = tpSection
addCorner(playerList, 6)

local goToButton = makeButton(tpSection, "IR ATÉ O JOGADOR", UDim2.fromOffset(159, 32), UDim2.fromOffset(12, 198))
local pullButton = makeButton(tpSection, "PUXAR JOGADOR", UDim2.fromOffset(159, 32), UDim2.fromOffset(179, 198))

local function refreshPlayerList()
	for _, child in ipairs(playerList:GetChildren()) do
		if child:IsA("TextButton") then child:Destroy() end
	end
	local y = 5
	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= localPlayer then
			local playerButton = makeButton(playerList, player.DisplayName .. " (@" .. player.Name .. ")", UDim2.new(1, -10, 0, 28), UDim2.fromOffset(5, y))
			playerButton.TextSize = 11
				playerButton.MouseButton1Click:Connect(function()
					selectedPlayer = player
					for _, item in ipairs(playerList:GetChildren()) do
						if item:IsA("TextButton") then setButtonColor(item, THEME.surface) end
					end
				setButtonColor(playerButton, THEME.success)
			end)
			y += 33
		end
	end
	playerList.CanvasSize = UDim2.fromOffset(0, y)
end

local function sendTP(action)
	if not selectedPlayer or not selectedPlayer.Character then
		return
	end

	-- Fallback local para o teste: garante que IR ATÉ funcione
	-- mesmo quando o Script do servidor ainda não foi colocado.
	if action == "GoTo" then
		local targetRoot = selectedPlayer.Character:FindFirstChild("HumanoidRootPart")
		local myCharacter = localPlayer.Character
		if targetRoot and myCharacter then
			myCharacter:PivotTo(targetRoot.CFrame + Vector3.new(0, 3, 0))
		end
	elseif action == "Pull" then
		local myCharacter = localPlayer.Character
		local myRoot = myCharacter and myCharacter:FindFirstChild("HumanoidRootPart")
		local targetCharacter = selectedPlayer.Character
		if myRoot and targetCharacter and RunService:IsStudio() then
			-- Fallback visual para teste no Studio caso o Script do servidor não esteja ativo.
			targetCharacter:PivotTo(CFrame.lookAt(
				myRoot.Position + myRoot.CFrame.LookVector * 4,
				myRoot.Position
			))
		end
	end

	if tpRemote then
		tpRemote:FireServer(action, selectedPlayer.UserId)
	end
end

goToButton.MouseButton1Click:Connect(function() sendTP("GoTo") end)
pullButton.MouseButton1Click:Connect(function() sendTP("Pull") end)
Players.PlayerAdded:Connect(refreshPlayerList)
Players.PlayerRemoving:Connect(function(player)
	if selectedPlayer == player then selectedPlayer = nil end
	refreshPlayerList()
end)
refreshPlayerList()

local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.fromOffset(100, 24)
speedLabel.Position = UDim2.fromOffset(12, 144)
speedLabel.BackgroundTransparency = 1
speedLabel.Font = Enum.Font.GothamSemibold
speedLabel.TextSize = 12
speedLabel.TextXAlignment = Enum.TextXAlignment.Left
speedLabel.TextColor3 = Color3.fromRGB(205, 205, 215)
speedLabel.Text = "Velocidade: 70"
speedLabel.Parent = flightSection

local speedMinus = makeButton(flightSection, "−", UDim2.fromOffset(36, 26), UDim2.fromOffset(130, 142))
local speedValue = makeButton(flightSection, "70", UDim2.fromOffset(58, 26), UDim2.fromOffset(170, 142))
local speedPlus = makeButton(flightSection, "+", UDim2.fromOffset(36, 26), UDim2.fromOffset(232, 142))

local helpLabel = Instance.new("TextLabel")
helpLabel.Size = UDim2.fromOffset(350, 45)
helpLabel.Position = UDim2.fromOffset(0, 232)
helpLabel.BackgroundTransparency = 1
helpLabel.Font = Enum.Font.Gotham
helpLabel.TextSize = 11
helpLabel.TextWrapped = true
helpLabel.TextXAlignment = Enum.TextXAlignment.Left
helpLabel.TextYAlignment = Enum.TextYAlignment.Top
helpLabel.TextColor3 = Color3.fromRGB(155, 155, 165)
helpLabel.Text = "RightShift: abrir/fechar painel\nDurante o voo: WASD + Espaço/Ctrl"
helpLabel.Parent = content

local espControls = {espTitle, espButton, boxButton, skeletonButton, nameButton, distanceButton, linesButton, colorButton, lineOriginButton, maxDistanceLabel, distanceMinus, distanceValue, distancePlus}

local godModeEnabled = false
local noClipEnabled = false
local localNoClipOriginalCollision = {}

local function setLocalNoClip(enabled)
	local character = localPlayer.Character
	if not character then return end
	if enabled then
		for _, descendant in ipairs(character:GetDescendants()) do
			if descendant:IsA("BasePart") then
				if localNoClipOriginalCollision[descendant] == nil then
					localNoClipOriginalCollision[descendant] = descendant.CanCollide
				end
				descendant.CanCollide = false
			end
		end
	else
		for part, canCollide in pairs(localNoClipOriginalCollision) do
			if part and part.Parent then
				part.CanCollide = canCollide
			end
		end
		localNoClipOriginalCollision = {}
	end
end

local function selectTab(tabName)
	local showESP = tabName == "ESP"
	local showAim = tabName == "AIM"
	local showFlight = tabName == "VOAR"
	local showGod = tabName == "GOD"
	local showFPS = tabName == "FPS"
	for _, control in ipairs(espControls) do
		control.Visible = showESP
	end
	flightSection.Visible = showFlight
	aimSection.Visible = showAim
	tpSection.Visible = tabName == "TP"
	godSection.Visible = showGod
	fpsSection.Visible = showFPS
	helpLabel.Visible = showFlight
	setButtonColor(espTab, showESP and THEME.accent or THEME.surface)
	setButtonColor(aimTab, showAim and THEME.accent or THEME.surface)
	setButtonColor(flightTab, showFlight and THEME.accent or THEME.surface)
	setButtonColor(tpTab, tabName == "TP" and THEME.accent or THEME.surface)
	setButtonColor(godTab, showGod and THEME.accent or THEME.surface)
	setButtonColor(fpsTab, showFPS and THEME.accent or THEME.surface)
end

espTab.MouseButton1Click:Connect(function()
		selectTab("ESP")
end)

aimTab.MouseButton1Click:Connect(function()
	selectTab("AIM")
end)

flightTab.MouseButton1Click:Connect(function()
		selectTab("VOAR")
end)

tpTab.MouseButton1Click:Connect(function()
	selectTab("TP")
end)

godTab.MouseButton1Click:Connect(function()
	selectTab("GOD")
end)

fpsTab.MouseButton1Click:Connect(function()
	selectTab("FPS")
end)


fpsToggleButton.MouseButton1Click:Connect(function()
	fpsEnabled = not fpsEnabled
	fpsCounter.Visible = fpsEnabled
	fpsToggleButton.Text = fpsEnabled and "FPS: LIGADO" or "FPS: DESLIGADO"
	setButtonColor(fpsToggleButton, fpsEnabled and THEME.success or THEME.surface)
end)

local function updateFOVInterface()
	fovToggleButton.Text = fovEnabled and "FOV: LIGADO" or "FOV: DESLIGADO"
	setButtonColor(fovToggleButton, fovEnabled and THEME.success or THEME.surface)
	fovValue.Text = tostring(fovRadius) .. " px"
	fovCircle.Size = UDim2.fromOffset(fovRadius * 2, fovRadius * 2)
	fovCircle.Visible = fovEnabled
	fovCenterDot.Visible = fovEnabled
end

fovToggleButton.MouseButton1Click:Connect(function()
	fovEnabled = not fovEnabled
	updateFOVInterface()
end)

fovMinus.MouseButton1Click:Connect(function()
	fovRadius = math.max(50, fovRadius - 25)
	updateFOVInterface()
end)

fovPlus.MouseButton1Click:Connect(function()
	fovRadius = math.min(500, fovRadius + 25)
	updateFOVInterface()
end)

godButton.MouseButton1Click:Connect(function()
	godModeEnabled = not godModeEnabled
	godButton.Text = godModeEnabled and "God Mod: LIGADO" or "God Mod: DESLIGADO"
	setButtonColor(godButton, godModeEnabled and THEME.success or THEME.surface)
	if tpRemote then
		tpRemote:FireServer("GodMode", godModeEnabled)
	end
end)

noClipButton.MouseButton1Click:Connect(function()
	noClipEnabled = not noClipEnabled
	setLocalNoClip(noClipEnabled)
	noClipButton.Text = noClipEnabled and "Atravessar paredes: LIGADO" or "Atravessar paredes: DESLIGADO"
	setButtonColor(noClipButton, noClipEnabled and THEME.success or THEME.surface)
	if tpRemote then
		tpRemote:FireServer("NoClip", noClipEnabled)
	end
end)

gokuSkinButton.MouseButton1Click:Connect(function()
	if tpRemote then
		gokuSkinButton.Text = "APLICANDO SKIN..."
		tpRemote:FireServer("GokuSkin")
	end
end)

local selectedTimeMode

local function applyTimeOfDay(mode)
	if mode == "DIA" then
		Lighting.ClockTime = 12
		Lighting.Brightness = 3
		Lighting.Ambient = Color3.fromRGB(170, 170, 170)
		Lighting.OutdoorAmbient = Color3.fromRGB(190, 190, 190)
		Lighting.FogEnd = 100000
		dayButton.BackgroundColor3 = Color3.fromRGB(35, 130, 65)
		nightButton.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
	else
		Lighting.ClockTime = 0
		Lighting.Brightness = 1.5
		Lighting.Ambient = Color3.fromRGB(75, 85, 125)
		Lighting.OutdoorAmbient = Color3.fromRGB(55, 65, 100)
		Lighting.FogEnd = 100000
		dayButton.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
		nightButton.BackgroundColor3 = Color3.fromRGB(35, 90, 130)
	end
end

local function setTimeOfDay(mode)
	selectedTimeMode = mode
	applyTimeOfDay(selectedTimeMode)
end

dayButton.MouseButton1Click:Connect(function()
	setTimeOfDay("DIA")
end)

nightButton.MouseButton1Click:Connect(function()
	setTimeOfDay("NOITE")
end)

-- Botão flutuante quando o painel estiver minimizado.
local openButton = makeButton(screenGui, "FIU FIU", UDim2.fromOffset(42, 38), UDim2.fromOffset(20, 100))
openButton.Font = Enum.Font.GothamBold
openButton.TextSize = 22
-- Posiciona o botão fechado no cabeçalho, alinhado à área do título Itallo7.
local function getOpenButtonPosition()
	local panelSize = panel.AbsoluteSize
	return UDim2.new(
		panel.Position.X.Scale,
		panel.Position.X.Offset - panelSize.X / 2 + 38,
		panel.Position.Y.Scale,
		panel.Position.Y.Offset - panelSize.Y / 2 + 22
	)
end
openButton.AnchorPoint = Vector2.new(0.5, 0.5)
openButton.Position = getOpenButtonPosition()
openButton.Visible = false

if tpRemote then
	tpRemote.OnClientEvent:Connect(function(action, enabled, message)
			if action == "NoClipState" then
			noClipEnabled = enabled == true
			setLocalNoClip(noClipEnabled)
			noClipButton.Text = noClipEnabled and "Atravessar paredes: LIGADO" or "Atravessar paredes: DESLIGADO"
			setButtonColor(noClipButton, noClipEnabled and THEME.success or THEME.surface)
			if message then godInfo.Text = message end
				return
			end
			if action == "GokuSkinState" then
				gokuSkinButton.Text = "PUXAR SKIN GOKU SUKUNA"
				if message then godInfo.Text = message end
				return
			end
			if action ~= "GodModeState" then return end
	godModeEnabled = enabled == true
	godButton.Text = godModeEnabled and "God Mod: LIGADO" or "God Mod: DESLIGADO"
	setButtonColor(godButton, godModeEnabled and THEME.success or THEME.surface)
	if message then godInfo.Text = message end
	end)
end

selectTab("ESP")
openButton.BackgroundColor3 = THEME.accentDark
openButton:SetAttribute("StateColor", THEME.accentDark)

-- Arrastar o painel pela barra de título
local dragging = false
local dragStart
local startPosition
local savedPanelPosition = panel.Position
local resizing = false
local resizeStart
local resizeScale
local openButtonDragging = false
local openButtonDragStart
local openButtonStartPosition
local openButtonMoved = false

-- Permite mover o botão enquanto o painel está fechado.
openButton.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		openButtonDragging = true
		openButtonMoved = false
		openButtonDragStart = input.Position
		openButtonStartPosition = openButton.Position
	end
end)

openButton.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		openButtonDragging = false
	end
end)

header.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPosition = panel.Position
	end
end)

header.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = false
	end
end)

resizeHandle.MouseEnter:Connect(function()
	resizeHandle.TextColor3 = THEME.text
end)

resizeHandle.MouseLeave:Connect(function()
	resizeHandle.TextColor3 = THEME.cyan
end)

resizeHandle.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		resizing = true
		resizeStart = input.Position
		resizeScale = panelScale.Scale
	end
end)

resizeHandle.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		resizing = false
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if resizing and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
		local delta = input.Position - resizeStart
		local newScale = math.clamp(resizeScale + (delta.X / 390), 0.55, 1.60)
		panelScale.Scale = newScale
		return
	end
	if openButtonDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
		local delta = input.Position - openButtonDragStart
		if delta.Magnitude > 5 then
			openButtonMoved = true
		end
		openButton.Position = UDim2.new(
			openButtonStartPosition.X.Scale,
			openButtonStartPosition.X.Offset + delta.X,
			openButtonStartPosition.Y.Scale,
			openButtonStartPosition.Y.Offset + delta.Y
		)
		return
	end
	if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
		local delta = input.Position - dragStart
			panel.Position = UDim2.new(
				startPosition.X.Scale,
				startPosition.X.Offset + delta.X,
				startPosition.Y.Scale,
				startPosition.Y.Offset + delta.Y
			)
			savedPanelPosition = panel.Position
			openButton.Position = getOpenButtonPosition()
		end
end)

local function getColor()
	return colors[settings.colorIndex].value
end

local function hideEntry(entry)
	if entry.box then
		entry.box.Enabled = false
	end
	if entry.billboard then
		entry.billboard.Enabled = false
	end
	if entry.line then
		entry.line.Visible = false
	end
	if entry.skeletonLines then
		for _, skeletonLine in ipairs(entry.skeletonLines) do
			skeletonLine.Visible = false
		end
	end
end

local function destroyVisuals(entry)
	if entry.box then
		entry.box:Destroy()
		entry.box = nil
	end
	if entry.billboard then
		entry.billboard:Destroy()
		entry.billboard = nil
	end
	if entry.line then
		entry.line:Destroy()
		entry.line = nil
	end
	if entry.skeletonLines then
		for _, skeletonLine in ipairs(entry.skeletonLines) do
			skeletonLine:Destroy()
		end
		entry.skeletonLines = nil
	end
	entry.label = nil
end

local function createVisuals(player, character)
	local entry = entries[player]
	if not entry then
		return
	end

	destroyVisuals(entry)

	local root = character:WaitForChild("HumanoidRootPart", 5)
	if not root then
		return
	end

	-- Contorno que acompanha todo o personagem (R6 e R15).
	local box = Instance.new("Highlight")
	box.Name = "ESPCharacterOutline"
	box.Adornee = character
	box.FillTransparency = 1
	box.OutlineColor = getColor()
	box.OutlineTransparency = 0
	box.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
	box.Enabled = settings.esp and settings.box
	box.Parent = character

	local billboard = Instance.new("BillboardGui")
	billboard.Name = "ESPInfo"
	billboard.Adornee = root
	billboard.Size = UDim2.fromOffset(180, 32)
	billboard.StudsOffset = Vector3.new(0, 4.2, 0)
	billboard.AlwaysOnTop = true
	billboard.Enabled = settings.esp and (settings.name or settings.distance)
	billboard.Parent = root

	local label = Instance.new("TextLabel")
	label.Name = "PlayerInfo"
	label.BackgroundTransparency = 1
	label.Size = UDim2.fromScale(1, 1)
	label.Font = Enum.Font.GothamBold
	label.TextSize = 11
	label.TextColor3 = Color3.fromRGB(255, 255, 255)
	label.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
	label.TextStrokeTransparency = 0.2
	label.TextWrapped = true
	label.Parent = billboard

	local line = Instance.new("Frame")
	line.Name = "ESPLine"
	line.AnchorPoint = Vector2.new(0.5, 0.5)
	line.BackgroundColor3 = getColor()
	line.BorderSizePixel = 0
	line.Size = UDim2.fromOffset(2, 0)
	line.Visible = settings.esp and settings.lines
	line.Parent = screenGui

	local skeletonLines = {}
	local skeletonConnections = {
		{"Head", "UpperTorso"}, {"UpperTorso", "LowerTorso"},
		{"UpperTorso", "LeftUpperArm"}, {"LeftUpperArm", "LeftLowerArm"},
		{"LeftLowerArm", "LeftHand"}, {"UpperTorso", "RightUpperArm"},
		{"RightUpperArm", "RightLowerArm"}, {"RightLowerArm", "RightHand"},
		{"LowerTorso", "LeftUpperLeg"}, {"LeftUpperLeg", "LeftLowerLeg"},
		{"LeftLowerLeg", "LeftFoot"}, {"LowerTorso", "RightUpperLeg"},
		{"RightUpperLeg", "RightLowerLeg"}, {"RightLowerLeg", "RightFoot"},
		{"Head", "Torso"}, {"Torso", "Left Arm"}, {"Left Arm", "Left Hand"},
		{"Torso", "Right Arm"}, {"Right Arm", "Right Hand"},
		{"Torso", "Left Leg"}, {"Left Leg", "Left Foot"},
		{"Torso", "Right Leg"}, {"Right Leg", "Right Foot"},
	}
	for _ = 1, #skeletonConnections do
		local skeletonLine = Instance.new("Frame")
		skeletonLine.Name = "ESPSkeletonLine"
		skeletonLine.AnchorPoint = Vector2.new(0.5, 0.5)
		skeletonLine.BackgroundColor3 = getColor()
		skeletonLine.BorderSizePixel = 0
		skeletonLine.Size = UDim2.fromOffset(2, 0)
		skeletonLine.Visible = false
		skeletonLine.Parent = screenGui
		table.insert(skeletonLines, skeletonLine)
	end

	entry.box = box
	entry.billboard = billboard
	entry.label = label
	entry.line = line
	entry.skeletonLines = skeletonLines
	entry.skeletonConnections = skeletonConnections
end

local function updateButton(button, label, enabled)
	button.Text = label .. (enabled and ": LIGADO" or ": DESLIGADO")
	setButtonColor(button, enabled and THEME.success or THEME.surface)
end

local function updateInterface()
	espButton.Text = settings.esp and "ESP: LIGADO" or "ESP: DESLIGADO"
	setButtonColor(espButton, settings.esp and THEME.success or THEME.surface)
	updateButton(boxButton, "Caixa", settings.box)
	updateButton(skeletonButton, "Skeleton", settings.skeleton)
	updateButton(nameButton, "Nome", settings.name)
	updateButton(distanceButton, "Distância", settings.distance)
	updateButton(linesButton, "Linhas", settings.lines)
	colorButton.Text = "Cor do ESP: " .. colors[settings.colorIndex].name
	lineOriginButton.Text = "Origem da linha: " .. lineOrigins[settings.lineOriginIndex].name
	distanceValue.Text = tostring(settings.maxDistance) .. " studs"
	flyButton.Text = settings.flying and "Voar: LIGADO" or "Voar: DESLIGADO"
	setButtonColor(flyButton, settings.flying and THEME.success or THEME.surface)
	speedLabel.Text = "Velocidade: " .. tostring(settings.flightSpeed)
	speedValue.Text = tostring(settings.flightSpeed)
end

local function refreshVisuals()
	for _, entry in pairs(entries) do
		if entry.box then
			entry.box.OutlineColor = getColor()
			entry.box.Enabled = settings.esp and settings.box
		end
		if entry.line then
			entry.line.BackgroundColor3 = getColor()
			entry.line.Visible = settings.esp and settings.lines
		end
		if entry.skeletonLines then
			for _, skeletonLine in ipairs(entry.skeletonLines) do
				skeletonLine.BackgroundColor3 = getColor()
				skeletonLine.Visible = false
			end
		end
		if entry.billboard then
			entry.billboard.Enabled = settings.esp and (settings.name or settings.distance)
		end
	end
	updateInterface()
end

local function watchPlayer(player)
	if player == localPlayer or entries[player] then
		return
	end

	local entry = {}
	entries[player] = entry

	entry.characterConnection = player.CharacterAdded:Connect(function(character)
		createVisuals(player, character)
	end)

	if player.Character then
		task.spawn(function()
			createVisuals(player, player.Character)
		end)
	end
end

local function togglePanel()
	if panelOpen then
			-- Guarda a última posição usada para reabrir no mesmo lugar.
			savedPanelPosition = panel.Position
		openButton.Position = getOpenButtonPosition()
		panelOpen = false
		panel.Visible = false
		openButton.Text = "+"
		openButton.Visible = true
	else
			-- Reabre exatamente onde o painel foi deixado.
			panel.Position = savedPanelPosition or UDim2.fromScale(0.5, 0.5)
		openButton.Position = getOpenButtonPosition()
		panelOpen = true
		panel.Visible = true
		openButton.Text = "+"
		openButton.Visible = false
	end
end

espButton.MouseButton1Click:Connect(function()
	settings.esp = not settings.esp
	refreshVisuals()
end)

boxButton.MouseButton1Click:Connect(function()
	settings.box = not settings.box
	refreshVisuals()
end)

skeletonButton.MouseButton1Click:Connect(function()
	settings.skeleton = not settings.skeleton
	refreshVisuals()
end)

nameButton.MouseButton1Click:Connect(function()
	settings.name = not settings.name
	refreshVisuals()
end)

distanceButton.MouseButton1Click:Connect(function()
	settings.distance = not settings.distance
	refreshVisuals()
end)

linesButton.MouseButton1Click:Connect(function()
	settings.lines = not settings.lines
	refreshVisuals()
end)

colorButton.MouseButton1Click:Connect(function()
	settings.colorIndex = settings.colorIndex % #colors + 1
	refreshVisuals()
end)

lineOriginButton.MouseButton1Click:Connect(function()
	settings.lineOriginIndex = settings.lineOriginIndex % #lineOrigins + 1
	updateInterface()
end)

distanceMinus.MouseButton1Click:Connect(function()
	settings.maxDistance = math.max(100, settings.maxDistance - 100)
	updateInterface()
end)

distancePlus.MouseButton1Click:Connect(function()
	settings.maxDistance = math.min(5000, settings.maxDistance + 100)
	updateInterface()
end)

speedMinus.MouseButton1Click:Connect(function()
	settings.flightSpeed = math.max(10, settings.flightSpeed - 10)
	updateInterface()
end)

speedPlus.MouseButton1Click:Connect(function()
	settings.flightSpeed = math.min(300, settings.flightSpeed + 10)
	updateInterface()
end)

local flightVelocity
local flightGyro

local function stopLocalFlight()
	if flightVelocity then flightVelocity:Destroy(); flightVelocity = nil end
	if flightGyro then flightGyro:Destroy(); flightGyro = nil end
	local character = localPlayer.Character
	local humanoid = character and character:FindFirstChildOfClass("Humanoid")
	if humanoid then
		humanoid.PlatformStand = false
		humanoid.AutoRotate = true
	end
end

local function startLocalFlight()
	stopLocalFlight()
	local character = localPlayer.Character
	local root = character and character:FindFirstChild("HumanoidRootPart")
	local humanoid = character and character:FindFirstChildOfClass("Humanoid")
	if not root or not humanoid then return end
	flightVelocity = Instance.new("BodyVelocity")
	flightVelocity.Name = "TestFlightVelocity"
	flightVelocity.MaxForce = Vector3.new(100000, 100000, 100000)
	flightVelocity.P = 10000
	flightVelocity.Velocity = Vector3.zero
	flightVelocity.Parent = root
	flightGyro = Instance.new("BodyGyro")
	flightGyro.Name = "TestFlightGyro"
	flightGyro.MaxTorque = Vector3.new(100000, 100000, 100000)
	flightGyro.P = 10000
	flightGyro.CFrame = root.CFrame
	flightGyro.Parent = root
	humanoid.PlatformStand = true
	humanoid.AutoRotate = false
	camera.CameraType = Enum.CameraType.Custom
	camera.CameraSubject = humanoid
end

flyButton.MouseButton1Click:Connect(function()
	settings.flying = not settings.flying
	if settings.flying then startLocalFlight() else stopLocalFlight() end
	flyButton.Text = settings.flying and "Voar: LIGADO" or "Voar: DESLIGADO"
	setButtonColor(flyButton, settings.flying and THEME.success or THEME.surface)
end)

minimizeButton.MouseButton1Click:Connect(togglePanel)
openButton.MouseButton1Click:Connect(function()
	if openButtonMoved then
		-- O arraste não deve abrir o painel por acidente.
		openButtonMoved = false
		return
	end
	togglePanel()
end)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then
		return
	end
	if input.KeyCode == Enum.KeyCode.RightShift then
		togglePanel()
	end
end)

local function sendFlightDirection()
	if not settings.flying or not flightVelocity then
		return
	end

	local character = localPlayer.Character
	local humanoid = character and character:FindFirstChildOfClass("Humanoid")
	-- Converte o joystick para o espaço da câmera, preservando o sentido para trás.
	local moveDirection = humanoid and humanoid.MoveDirection or Vector3.zero
	local direction = Vector3.zero
	if moveDirection.Magnitude > 0.05 then
		local cameraLook = camera.CFrame.LookVector
		local flatLook = Vector3.new(cameraLook.X, 0, cameraLook.Z)
		local right = camera.CFrame.RightVector
		right = Vector3.new(right.X, 0, right.Z)
		if flatLook.Magnitude > 0.05 and right.Magnitude > 0.05 then
			flatLook = flatLook.Unit
			right = right.Unit
			local forwardAmount = moveDirection:Dot(flatLook)
			local sideAmount = moveDirection:Dot(right)
			direction = cameraLook.Unit * forwardAmount + right * sideAmount
		end
	end

	flightVelocity.Velocity = direction * settings.flightSpeed
	if direction.Magnitude > 0.05 and flightGyro then
		local root = localPlayer.Character and localPlayer.Character:FindFirstChild("HumanoidRootPart")
		if root then flightGyro.CFrame = CFrame.lookAt(root.Position, root.Position + direction) end
	end
end

RunService.RenderStepped:Connect(function(deltaTime)
	updateFPS(deltaTime)
	if noClipEnabled then
		setLocalNoClip(true)
	end
	if selectedTimeMode then
		applyTimeOfDay(selectedTimeMode)
	end
	sendFlightDirection()
	if not settings.esp then
		return
	end

	local myCharacter = localPlayer.Character
	local myRoot = myCharacter and myCharacter:FindFirstChild("HumanoidRootPart")
	if not myRoot then
		return
	end

	local viewportSize = camera.ViewportSize
	local startPoint
	if settings.lineOriginIndex == 1 then -- Baixo
		startPoint = Vector2.new(viewportSize.X / 2, viewportSize.Y - 20)
	elseif settings.lineOriginIndex == 2 then -- Cima
		startPoint = Vector2.new(viewportSize.X / 2, 20)
	elseif settings.lineOriginIndex == 3 then -- Esquerda
		startPoint = Vector2.new(20, viewportSize.Y / 2)
	else -- Direita
		startPoint = Vector2.new(viewportSize.X - 20, viewportSize.Y / 2)
	end

	for player, entry in pairs(entries) do
		local character = player.Character
		local root = character and character:FindFirstChild("HumanoidRootPart")
		local humanoid = character and character:FindFirstChildOfClass("Humanoid")

		if root and humanoid and humanoid.Health > 0 and entry.label and entry.box and entry.billboard and entry.line then
			local distance = (myRoot.Position - root.Position).Magnitude

			if distance <= settings.maxDistance then
				local textParts = {}
				if settings.name then
					table.insert(textParts, player.DisplayName)
				end
				if settings.distance then
					table.insert(textParts, string.format("%d studs", math.floor(distance + 0.5)))
				end

				entry.label.Text = table.concat(textParts, "\n")
				entry.box.Enabled = settings.box
				entry.billboard.Enabled = settings.name or settings.distance

				local screenPosition, onScreen = camera:WorldToViewportPoint(root.Position)
				if settings.lines and onScreen and screenPosition.Z > 0 then
					local endPoint = Vector2.new(screenPosition.X, screenPosition.Y)
					local difference = endPoint - startPoint
					local length = difference.Magnitude
					local midpoint = (startPoint + endPoint) / 2
					entry.line.Position = UDim2.fromOffset(midpoint.X, midpoint.Y)
					entry.line.Size = UDim2.fromOffset(1, length)
					entry.line.Rotation = math.deg(math.atan2(-difference.X, difference.Y))
					entry.line.Visible = true
					else
						entry.line.Visible = false
					end

					for index, connection in ipairs(entry.skeletonConnections or {}) do
						local firstPart = character:FindFirstChild(connection[1])
						local secondPart = character:FindFirstChild(connection[2])
						local skeletonLine = entry.skeletonLines[index]
						if firstPart and secondPart and skeletonLine then
							local firstPoint, firstVisible = camera:WorldToViewportPoint(firstPart.Position)
							local secondPoint, secondVisible = camera:WorldToViewportPoint(secondPart.Position)
							if settings.skeleton and firstVisible and secondVisible and firstPoint.Z > 0 and secondPoint.Z > 0 then
								local start = Vector2.new(firstPoint.X, firstPoint.Y)
								local finish = Vector2.new(secondPoint.X, secondPoint.Y)
								local difference = finish - start
								skeletonLine.Position = UDim2.fromOffset((start.X + finish.X) / 2, (start.Y + finish.Y) / 2)
								skeletonLine.Size = UDim2.fromOffset(1, difference.Magnitude)
								skeletonLine.Rotation = math.deg(math.atan2(-difference.X, difference.Y))
								skeletonLine.Visible = true
							else
								skeletonLine.Visible = false
							end
						elseif skeletonLine then
							skeletonLine.Visible = false
						end
					end
				else
				hideEntry(entry)
			end
		else
			hideEntry(entry)
		end
	end
end)

Players.PlayerAdded:Connect(watchPlayer)

for _, player in ipairs(Players:GetPlayers()) do
	watchPlayer(player)
end

Players.PlayerRemoving:Connect(function(player)
	local entry = entries[player]
	if not entry then
		return
	end

	if entry.characterConnection then
		entry.characterConnection:Disconnect()
	end

	destroyVisuals(entry)
	entries[player] = nil
end)

updateInterface()

-- ===== LIBERAÇÃO FINAL DO PAINEL APÓS A KEY =====
-- O painel só fica visível depois que a Key for aceita.
-- Garante que todos os controles do painel fiquem acima da camada da key.
local function fixPanelLayer()
	panel.ZIndex = 200
	for _, item in ipairs(panel:GetDescendants()) do
		if item:IsA("GuiObject") then
			item.ZIndex = 201
		end
	end
end

local function updateKeyVisibility()
	if keyUnlocked then
		keyFrame.Visible = false
		panel.Visible = true
		openButton.Visible = false
	else
		keyFrame.Visible = true
		panel.Visible = false
		openButton.Visible = false
	end
end

-- ===== SISTEMA DE CARREGAMENTO =====
local loadingFrame
local loadingTitle
local loadingStatus
local loadingBarBackground
local loadingBar
local loadingPercent

local function createLoadingScreen()
	loadingFrame = Instance.new("Frame")
	loadingFrame.Name = "LoadingFrame"
	loadingFrame.Size = UDim2.fromScale(1, 1)
	loadingFrame.BackgroundColor3 = THEME.background
	loadingFrame.BorderSizePixel = 0
	loadingFrame.ZIndex = 300
	loadingFrame.Parent = screenGui

	local loadingBox = Instance.new("Frame")
	loadingBox.Name = "LoadingBox"
	loadingBox.Size = UDim2.fromOffset(330, 170)
	loadingBox.AnchorPoint = Vector2.new(0.5, 0.5)
	loadingBox.Position = UDim2.fromScale(0.5, 0.5)
	loadingBox.BackgroundColor3 = THEME.panel
	loadingBox.BorderSizePixel = 0
	loadingBox.ZIndex = 301
	loadingBox.Parent = loadingFrame
	addCorner(loadingBox, 12)
	addStroke(loadingBox, THEME.accent, 1)
	addGradient(loadingBox, THEME.panel, THEME.panelAlt, 90)

	loadingTitle = Instance.new("TextLabel")
	loadingTitle.Size = UDim2.new(1, -24, 0, 32)
	loadingTitle.Position = UDim2.fromOffset(12, 18)
	loadingTitle.BackgroundTransparency = 1
	loadingTitle.Font = Enum.Font.GothamBold
	loadingTitle.TextSize = 19
	loadingTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
	loadingTitle.Text = "CARREGANDO PAINEL"
	loadingTitle.ZIndex = 302
	loadingTitle.Parent = loadingBox

	loadingStatus = Instance.new("TextLabel")
	loadingStatus.Size = UDim2.new(1, -24, 0, 24)
	loadingStatus.Position = UDim2.fromOffset(12, 58)
	loadingStatus.BackgroundTransparency = 1
	loadingStatus.Font = Enum.Font.Gotham
	loadingStatus.TextSize = 12
	loadingStatus.TextColor3 = Color3.fromRGB(190, 190, 200)
	loadingStatus.Text = "Preparando..."
	loadingStatus.ZIndex = 302
	loadingStatus.Parent = loadingBox

	loadingBarBackground = Instance.new("Frame")
	loadingBarBackground.Size = UDim2.new(1, -40, 0, 18)
	loadingBarBackground.Position = UDim2.fromOffset(20, 96)
	loadingBarBackground.BackgroundColor3 = THEME.surface
	loadingBarBackground.BorderSizePixel = 0
	loadingBarBackground.ZIndex = 302
	loadingBarBackground.Parent = loadingBox
	addCorner(loadingBarBackground, 9)

	loadingBar = Instance.new("Frame")
	loadingBar.Size = UDim2.new(0, 0, 1, 0)
	loadingBar.BackgroundColor3 = THEME.accent
	loadingBar.BorderSizePixel = 0
	loadingBar.ZIndex = 303
	loadingBar.Parent = loadingBarBackground
	addCorner(loadingBar, 9)

	loadingPercent = Instance.new("TextLabel")
	loadingPercent.Size = UDim2.new(1, -24, 0, 22)
	loadingPercent.Position = UDim2.fromOffset(12, 126)
	loadingPercent.BackgroundTransparency = 1
	loadingPercent.Font = Enum.Font.GothamSemibold
	loadingPercent.TextSize = 13
	loadingPercent.TextColor3 = THEME.cyan
	loadingPercent.Text = "1%"
	loadingPercent.ZIndex = 302
	loadingPercent.Parent = loadingBox
end

local function runLoadingScreen()
	createLoadingScreen()
	for percent = 1, 100 do
		loadingBar.Size = UDim2.new(percent / 100, 0, 1, 0)
		loadingPercent.Text = tostring(percent) .. "%"
		loadingStatus.Text = percent < 100 and "Carregando..." or "Concluído!"
		task.wait(0.025)
	end
	task.wait(0.6)
	loadingFrame:Destroy()
	loadingFrame = nil
end

local function validateKey()
	local enteredKey = string.upper(tostring(keyInput.Text or ""))
	enteredKey = enteredKey:gsub("%s+", "")

	local requiredKey = string.upper(REQUIRED_KEY):gsub("%s+", "")

	if enteredKey == requiredKey then
		keyUnlocked = true
		keyStatus.Text = "Key aceita!"
		keyStatus.TextColor3 = Color3.fromRGB(120, 255, 150)
		keyButton.Text = "LIBERADO"

		screenGui.Enabled = true
		keyFrame.Visible = false
		panel.Visible = false
		openButton.Visible = false

		-- Mostra o carregamento antes de liberar as funções.
		task.spawn(function()
			runLoadingScreen()
			panelOpen = true
			panel.Visible = true
			panel.ZIndex = 200
			openButton.Position = getOpenButtonPosition()
			openButton.Visible = false
		end)
	else
		keyStatus.Text = "Key inválida."
		keyStatus.TextColor3 = Color3.fromRGB(255, 120, 120)
		keyInput.Text = ""
	end
end

keyButton.Activated:Connect(validateKey)

keyInput.FocusLost:Connect(function(enterPressed)
	if enterPressed then
		validateKey()
	end
end)

fixPanelLayer()
selectTab("ESP")
updateKeyVisibility()

-- ===== FIM DO LOCAL SCRIPT =====

-- ////////////////////////////////////////////////////////////////////////////
-- FIM DO LOCAL SCRIPT / INÍCIO DO SERVER SCRIPT
-- ////////////////////////////////////////////////////////////////////////////
-- Cole a partir da linha abaixo no ServerScriptService como Script.

-- Script em: ServerScriptService > TPPlayerServer
-- Para o seu próprio jogo.

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

-- Para jogo pessoal, o dono é reconhecido automaticamente.
-- Para jogo pertencente a um grupo, coloque seu UserId aqui.
local ADMINS = {
	[123456789] = true, -- substitua pelo seu UserId numérico
}

-- Altere estes valores antes de publicar o jogo.

local remote = ReplicatedStorage:FindFirstChild("TPPlayerRemote")
if remote and not remote:IsA("RemoteEvent") then
	remote:Destroy()
	remote = nil
end
if not remote then
	remote = Instance.new("RemoteEvent")
	remote.Name = "TPPlayerRemote"
	remote.Parent = ReplicatedStorage
end

local lastAction = {}
local godMode = {}
local godConnections = {}
local previousMaxHealth = {}
local noClip = {}
local noClipOriginalCollision = {}

local COOLDOWN = 1
local GOD_MAX_HEALTH = 1000000000

local function isAdmin(player)
	-- Em Studio, libera para testes.
	if RunService:IsStudio() then
		return true
	end

	-- Experiência pessoal: CreatorId é o UserId do dono.
	if game.CreatorType == Enum.CreatorType.User and player.UserId == game.CreatorId then
		return true
	end

	-- Experiência de grupo: libera o dono do grupo e os administradores definidos abaixo.
	if game.CreatorType == Enum.CreatorType.Group then
		local ok, rank = pcall(function()
			return player:GetRankInGroup(game.CreatorId)
		end)
		if ok and rank >= 254 then
			return true
		end
	end

	return ADMINS[player.UserId] == true
end

local function sendGodState(player, enabled, message)
	remote:FireClient(player, "GodModeState", enabled == true, message)
end

local function disconnectGodConnection(player)
	if godConnections[player] then
		godConnections[player]:Disconnect()
		godConnections[player] = nil
	end
end

local function setGodMode(player, enabled)
	enabled = enabled == true
	godMode[player] = enabled
	disconnectGodConnection(player)

	local character = player.Character
	local humanoid = character and character:FindFirstChildOfClass("Humanoid")
	if not humanoid then
		character = player.CharacterAdded:Wait()
		humanoid = character:WaitForChild("Humanoid", 5)
		if not humanoid then
			return false
		end
	end

	local forceField = character:FindFirstChild("AdminGodMode")
	if enabled then
		if not previousMaxHealth[player] then
			previousMaxHealth[player] = humanoid.MaxHealth
		end

		if not forceField then
			forceField = Instance.new("ForceField")
			forceField.Name = "AdminGodMode"
			forceField.Visible = false
			forceField.Parent = character
		end

		humanoid.MaxHealth = GOD_MAX_HEALTH
		humanoid.Health = GOD_MAX_HEALTH

		-- Impede dano normal e scripts que reduzem a vida.
		godConnections[player] = humanoid.HealthChanged:Connect(function()
			if godMode[player] and humanoid.Parent and humanoid.Health < GOD_MAX_HEALTH then
				humanoid.Health = GOD_MAX_HEALTH
			end
		end)
	else
		if forceField then
			forceField:Destroy()
		end

		humanoid.MaxHealth = previousMaxHealth[player] or 100
		humanoid.Health = math.min(math.max(humanoid.Health, 1), humanoid.MaxHealth)
		previousMaxHealth[player] = nil
	end

	return true
end

local applyNoClipToCharacter

local function hookCharacter(player, character)
	local humanoid = character:WaitForChild("Humanoid", 5)
	if humanoid and godMode[player] then
		setGodMode(player, true)
	end
	if noClip[player] then
		applyNoClipToCharacter(player, character)
	end
end

function applyNoClipToCharacter(player, character)
	if not character then return end
	noClipOriginalCollision[player] = noClipOriginalCollision[player] or {}
	for _, descendant in ipairs(character:GetDescendants()) do
		if descendant:IsA("BasePart") then
			if noClipOriginalCollision[player][descendant] == nil then
				noClipOriginalCollision[player][descendant] = descendant.CanCollide
			end
			if noClip[player] then
				descendant.CanCollide = false
			end
		end
	end
end

local function setNoClip(player, enabled)
	enabled = enabled == true
	noClip[player] = enabled
	local character = player.Character
	if enabled then
		applyNoClipToCharacter(player, character)
	else
		local original = noClipOriginalCollision[player]
		if original then
			for part, canCollide in pairs(original) do
				if part and part.Parent then
					part.CanCollide = canCollide
				end
			end
		end
		noClipOriginalCollision[player] = {}
	end
	return true
end

local function applyGokuSkin(player)
	-- Sempre usa o jogador que clicou no botão, nunca outro alvo.
	local character = player.Character or player.CharacterAdded:Wait()
	local humanoid = character and character:FindFirstChildOfClass("Humanoid")
	if not humanoid then
		humanoid = character:WaitForChild("Humanoid", 10)
	end
	if not humanoid then
		return false, "Seu personagem ainda não carregou. Tente novamente."
	end

	local ok, result = pcall(function()
		local userId = Players:GetUserIdFromNameAsync("yaniuspro")
		local description = Players:GetHumanoidDescriptionFromUserIdAsync(userId)
		-- ResetAsync remove a aparência antiga antes de aplicar a nova.
		humanoid:ApplyDescriptionResetAsync(description)
	end)
	if not ok then
		warn("Falha ao aplicar a skin de @yaniuspro: " .. tostring(result))
		return false, "Falha ao carregar @yaniuspro: " .. tostring(result)
	end
	if not humanoid.Parent or player.Character ~= character then
		return false, "O personagem foi recarregado durante a aplicação. Tente novamente."
	end
	return true, "Skin de Goku Sukuna aplicada."
end

local function validTarget(player, target)
	return typeof(target) == "Instance"
		and target:IsA("Player")
		and target ~= player
		and target.Parent == Players
		and target.Character
		and target.Character:FindFirstChild("HumanoidRootPart")
		and player.Character
		and player.Character:FindFirstChild("HumanoidRootPart")
end

remote.OnServerEvent:Connect(function(player, action, target)
		if action == "GokuSkin" then
			if not isAdmin(player) then
				remote:FireClient(player, "GokuSkinState", false, "Sem permissão para aplicar esta skin.")
				return
			end
			local success, message = applyGokuSkin(player)
			remote:FireClient(player, "GokuSkinState", success, message)
			return
		end

		if action == "GodMode" then
		if not isAdmin(player) then
			sendGodState(player, false, "Sem permissão. Adicione seu UserId à tabela ADMINS no Script do servidor.")
			return
		end

		local enabled = target == true
			local applied = setGodMode(player, enabled)
			if applied then
				sendGodState(player, enabled, enabled and "God Mod ativado pelo servidor." or "God Mod desativado pelo servidor.")
			else
				sendGodState(player, false, "Não foi possível encontrar o personagem. Tente novamente.")
			end
		return
	end

	if action == "NoClip" then
		if not isAdmin(player) then
			remote:FireClient(player, "NoClipState", false, "Sem permissão para atravessar paredes.")
			return
		end
		local enabled = target == true
		setNoClip(player, enabled)
		remote:FireClient(player, "NoClipState", enabled, enabled and "Atravessar paredes ativado pelo servidor." or "Atravessar paredes desativado.")
		return
	end

		if typeof(target) == "number" then
		target = Players:GetPlayerByUserId(target)
	end

	if not isAdmin(player) or os.clock() - (lastAction[player] or 0) < COOLDOWN then
		return
	end
	if not validTarget(player, target) then
		return
	end

	local myRoot = player.Character.HumanoidRootPart
	local targetRoot = target.Character.HumanoidRootPart
	lastAction[player] = os.clock()

	if action == "GoTo" then
		myRoot.CFrame = targetRoot.CFrame + Vector3.new(0, 3, 0)
	elseif action == "Pull" then
		local pullPosition = myRoot.Position + myRoot.CFrame.LookVector * 4
		local pullCFrame = CFrame.lookAt(pullPosition, myRoot.Position)
		targetRoot.AssemblyLinearVelocity = Vector3.zero
		targetRoot.AssemblyAngularVelocity = Vector3.zero
		target.Character:PivotTo(pullCFrame)
	end
end)

-- Reforço server-side para evitar que um dano muito grande deixe a vida em zero.
RunService.Heartbeat:Connect(function()
	for player, enabled in pairs(godMode) do
		if enabled and player.Parent == Players then
			local character = player.Character
			local humanoid = character and character:FindFirstChildOfClass("Humanoid")
			if humanoid and humanoid.Health < GOD_MAX_HEALTH then
				humanoid.Health = GOD_MAX_HEALTH
			end
		end
	end
	for player, enabled in pairs(noClip) do
		if enabled and player.Parent == Players then
			applyNoClipToCharacter(player, player.Character)
		end
	end
end)

Players.PlayerAdded:Connect(function(player)
	player.CharacterAdded:Connect(function(character)
		hookCharacter(player, character)
	end)
end)

for _, player in ipairs(Players:GetPlayers()) do
	player.CharacterAdded:Connect(function(character)
		hookCharacter(player, character)
	end)
	if player.Character then
		hookCharacter(player, player.Character)
	end
end

Players.PlayerRemoving:Connect(function(player)
	lastAction[player] = nil
	godMode[player] = nil
	previousMaxHealth[player] = nil
	setNoClip(player, false)
	noClip[player] = nil
	noClipOriginalCollision[player] = nil
	disconnectGodConnection(player)
end)

-- ===== FIM DO SCRIPT DO SERVIDOR =====

-- ////////////////////////////////////////////////////////////////////////////
-- FIM DO PACOTE ÚNICO
-- ////////////////////////////////////////////////////////////////////////////
