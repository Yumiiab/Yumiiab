local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Camera = Workspace.CurrentCamera

-- GUI
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "PainelCombate"
gui.ResetOnSpawn = false

local main = Instance.new("Frame", gui)
main.Size = UDim2.new(0, 260, 0, 310)
main.Position = UDim2.new(0, 40, 0, 100)
main.BackgroundColor3 = Color3.fromRGB(40, 0, 70)
main.BorderSizePixel = 0
main.Active = true
main.Draggable = true

local title = Instance.new("TextLabel", main)
title.Size = UDim2.new(1, 0, 0, 25)
title.BackgroundColor3 = Color3.fromRGB(100, 0, 150)
title.Text = "Painel: Combate V3"
title.Font = Enum.Font.GothamBold
title.TextColor3 = Color3.new(1, 1, 1)
title.TextSize = 16

local tabButtons = Instance.new("Frame", main)
tabButtons.Size = UDim2.new(1, 0, 0, 30)
tabButtons.Position = UDim2.new(0, 0, 0, 25)
tabButtons.BackgroundColor3 = Color3.fromRGB(60, 0, 100)

local contentFrame = Instance.new("Frame", main)
contentFrame.Position = UDim2.new(0, 0, 0, 55)
contentFrame.Size = UDim2.new(1, 0, 1, -55)
contentFrame.BackgroundColor3 = Color3.fromRGB(30, 0, 50)
contentFrame.BorderSizePixel = 0

local tabs, settings = {}, {
	Aimbot = false,
	AimStrength = 0.15,
	NoRecoil = false,
	NoSpread = false,
	OneTap = false,
	ESPBox = false,
	ESPNome = false,
	ESPTracer = false
}

local function createTab(name, order)
	local btn = Instance.new("TextButton", tabButtons)
	btn.Size = UDim2.new(0, 80, 1, 0)
	btn.Position = UDim2.new(0, order * 80, 0, 0)
	btn.Text = name
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 13
	btn.TextColor3 = Color3.new(1, 1, 1)
	btn.BackgroundColor3 = Color3.fromRGB(100, 0, 150)

	local frame = Instance.new("Frame", contentFrame)
	frame.Size = UDim2.new(1, 0, 1, 0)
	frame.Visible = order == 0
	frame.BackgroundTransparency = 1

	tabs[name] = frame

	btn.MouseButton1Click:Connect(function()
		for tabName, tabFrame in pairs(tabs) do
			tabFrame.Visible = (tabName == name)
		end
	end)

	return frame
end

local aimTab = createTab("Aimbot", 0)
local armaTab = createTab("Arma", 1)
local visualTab = createTab("Visual", 2)

local function addToggle(parent, text, yPos, callback)
	local btn = Instance.new("TextButton", parent)
	btn.Size = UDim2.new(0, 220, 0, 30)
	btn.Position = UDim2.new(0, 20, 0, yPos)
	btn.BackgroundColor3 = Color3.fromRGB(120, 0, 180)
	btn.TextColor3 = Color3.new(1, 1, 1)
	btn.Font = Enum.Font.Gotham
	btn.TextSize = 14
	btn.Text = text .. ": OFF"

	local state = false
	btn.MouseButton1Click:Connect(function()
		state = not state
		btn.Text = text .. ": " .. (state and "ON" or "OFF")
		btn.BackgroundColor3 = state and Color3.fromRGB(180, 0, 255) or Color3.fromRGB(120, 0, 180)
		callback(state)
	end)
end

-- Aimbot Funções
local aimCircle = Drawing.new("Circle")
aimCircle.Color = Color3.fromRGB(255, 0, 255)
aimCircle.Thickness = 1
aimCircle.NumSides = 100
aimCircle.Radius = 100
aimCircle.Filled = false
aimCircle.Visible = true
aimCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)

local function getClosestInCircle()
	local closest, dist = nil, math.huge
	for _, p in ipairs(Players:GetPlayers()) do
		if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
			local pos, onScreen = Camera:WorldToViewportPoint(p.Character.HumanoidRootPart.Position)
			if onScreen then
				local mousePos = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
				local diff = (Vector2.new(pos.X, pos.Y) - mousePos)
				if diff.Magnitude <= aimCircle.Radius and diff.Magnitude < dist then
					closest = p
					dist = diff.Magnitude
				end
			end
		end
	end
	return closest
end

RunService.RenderStepped:Connect(function()
	if settings.Aimbot then
		local target = getClosestInCircle()
		if target and target.Character and target.Character:FindFirstChild("Head") then
			local headPos = target.Character.Head.Position
			Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, headPos), settings.AimStrength)
		end
	end
end)

-- Arma
addToggle(armaTab, "No Recoil", 10, function(on)
	settings.NoRecoil = on
end)

addToggle(armaTab, "No Spread", 50, function(on)
	settings.NoSpread = on
end)

addToggle(armaTab, "One Tap Kill", 90, function(on)
	settings.OneTap = on
end)

-- Aimbot GUI
addToggle(aimTab, "Aimbot Magnético", 10, function(on)
	settings.Aimbot = on
end)
addToggle(aimTab, "Força: Suave", 50, function(on)
	if on then settings.AimStrength = 0.05 end
end)
addToggle(aimTab, "Força: Médio", 90, function(on)
	if on then settings.AimStrength = 0.15 end
end)
addToggle(aimTab, "Força: Forte", 130, function(on)
	if on then settings.AimStrength = 0.3 end
end)

-- ESP
function createESP(plr)
	if plr == LocalPlayer then return end
	local box = Drawing.new("Square")
	local name = Drawing.new("Text")
	local tracer = Drawing.new("Line")
	local hpBar = Drawing.new("Line")

	box.Color = Color3.fromRGB(255, 0, 255)
	box.Filled = false
	box.Thickness = 1

	name.Color = Color3.new(1, 1, 1)
	name.Size = 13
	name.Center = true
	name.Outline = true

	tracer.Color = Color3.fromRGB(255, 0, 255)
	tracer.Thickness = 1

	hpBar.Color = Color3.fromRGB(0, 255, 0)
	hpBar.Thickness = 2

	RunService.RenderStepped:Connect(function()
		local char = plr.Character
		if not char or not char:FindFirstChild("HumanoidRootPart") then
			box.Visible, name.Visible, tracer.Visible, hpBar.Visible = false, false, false, false
			return
		end

		local hrp = char.HumanoidRootPart
		local head = char:FindFirstChild("Head")
		local hum = char:FindFirstChildOfClass("Humanoid")
		local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)

		if settings.ESPBox and onScreen then
			local size = Vector2.new(40, 60)
			box.Position = Vector2.new(pos.X - size.X/2, pos.Y - size.Y/2)
			box.Size = size
			box.Visible = true
		else
			box.Visible = false
		end

		if settings.ESPNome and head then
			local namePos = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 1.5, 0))
			name.Position = Vector2.new(namePos.X, namePos.Y)
			name.Text = plr.Name
			name.Visible = true
		else
			name.Visible = false
		end

		if settings.ESPTracer and onScreen then
			tracer.From = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y)
			tracer.To = Vector2.new(pos.X, pos.Y)
			tracer.Visible = true
		else
			tracer.Visible = false
		end

		if settings.ESPBox and hum then
			local ratio = hum.Health / hum.MaxHealth
			hpBar.From = Vector2.new(pos.X - 25, pos.Y + 30)
			hpBar.To = Vector2.new(pos.X - 25, pos.Y + 30 - (60 * ratio))
			hpBar.Visible = true
		else
			hpBar.Visible = false
		end
	end)
end

for _, p in ipairs(Players:GetPlayers()) do
	createESP(p)
end
Players.PlayerAdded:Connect(createESP)

-- Visual tab
addToggle(visualTab, "ESP Box", 10, function(on)
	settings.ESPBox = on
end)
addToggle(visualTab, "Nomes", 50, function(on)
	settings.ESPNome = on
end)
addToggle(visualTab, "Tracers", 90, function(on)
	settings.ESPTracer = on
end)
