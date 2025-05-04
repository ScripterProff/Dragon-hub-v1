-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Configurações
local Settings = {
    Aimbot = false,
    ESP = false,
    FOV = 120,
    AimPart = "Head",
    TeamCheck = true
}

-- GUI - Dragon Hub Mobile
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "DragonHub"
gui.ResetOnSpawn = false

local main = Instance.new("Frame", gui)
main.Size = UDim2.new(0, 220, 0, 260)
main.Position = UDim2.new(0.5, -110, 0.5, -130)
main.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
main.AnchorPoint = Vector2.new(0.5, 0.5)
main.Active = true
main.Draggable = true

Instance.new("UICorner", main)
Instance.new("UIScale", main).Scale = 1.0

local function createBtn(text, y, callback)
	local b = Instance.new("TextButton", main)
	b.Size = UDim2.new(0.9, 0, 0, 30)
	b.Position = UDim2.new(0.05, 0, 0, y)
	b.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
	b.Text = text
	b.TextColor3 = Color3.new(1, 1, 1)
	b.Font = Enum.Font.Gotham
	b.TextSize = 14
	b.MouseButton1Click:Connect(callback)
end

Instance.new("TextLabel", main) {
	Size = UDim2.new(1, 0, 0, 30),
	Text = "Dragon Hub",
	TextColor3 = Color3.new(1, 1, 1),
	BackgroundTransparency = 1,
	Font = Enum.Font.GothamBold,
	TextSize = 16
}

createBtn("Toggle Aimbot", 40, function()
	Settings.Aimbot = not Settings.Aimbot
end)

createBtn("Toggle ESP", 80, function()
	Settings.ESP = not Settings.ESP
end)

createBtn("Minimizar", 170, function()
	main.Visible = false
end)

createBtn("Fechar", 210, function()
	gui:Destroy()
end)

-- FOV Circle
local FOV = Drawing.new("Circle")
FOV.Thickness = 2
FOV.Filled = false
FOV.Radius = Settings.FOV
FOV.Color = Color3.fromRGB(255, 255, 0)
FOV.Visible = true

-- ESP Table
local espBoxes = {}

local function CreateESP(player)
	local box = Drawing.new("Square")
	box.Thickness = 2
	box.Filled = false
	box.Color = Color3.fromRGB(255, 0, 0)
	box.Visible = false

	local line = Drawing.new("Line")
	line.Thickness = 1.5
	line.Color = Color3.fromRGB(255, 0, 0)
	line.Visible = false

	espBoxes[player] = {Box = box, Line = line}
end

local function RemoveESP(player)
	if espBoxes[player] then
		espBoxes[player].Box:Remove()
		espBoxes[player].Line:Remove()
		espBoxes[player] = nil
	end
end

-- Criar ESP para jogadores existentes
for _, p in pairs(Players:GetPlayers()) do
	if p ~= LocalPlayer then
		CreateESP(p)
	end
end

-- Criar e remover ESP conforme jogadores entram ou saem
Players.PlayerAdded:Connect(function(p)
	if p ~= LocalPlayer then CreateESP(p) end
end)
Players.PlayerRemoving:Connect(RemoveESP)

-- Melhor Aimbot: Inimigo mais próximo fisicamente e dentro do FOV
local function GetClosestTarget()
	local closest = nil
	local closestDist = math.huge
	for _, p in ipairs(Players:GetPlayers()) do
		if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild(Settings.AimPart) then
			if not Settings.TeamCheck or p.Team ~= LocalPlayer.Team then
				local head = p.Character[Settings.AimPart]
				local screenPos, onScreen = Camera:WorldToViewportPoint(head.Position)
				local distFromCenter = (Vector2.new(screenPos.X, screenPos.Y) - Camera.ViewportSize / 2).Magnitude
				local distance = (head.Position - Camera.CFrame.Position).Magnitude
				if onScreen and distFromCenter < Settings.FOV and distance < closestDist then
					closestDist = distance
					closest = head
				end
			end
		end
	end
	return closest
end

-- Mobile Aimbot ativado por toque
local aiming = false
UserInputService.TouchStarted:Connect(function() aiming = true end)
UserInputService.TouchEnded:Connect(function() aiming = false end)

-- Loop principal
RunService.RenderStepped:Connect(function()
	FOV.Position = Camera.ViewportSize / 2

	-- Aimbot
	if Settings.Aimbot and aiming then
		local target = GetClosestTarget()
		if target then
			Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Position)
		end
	end

	-- ESP
	for player, drawings in pairs(espBoxes) do
		local char = player.Character
		local box, line = drawings.Box, drawings.Line
		if char and char:FindFirstChild("HumanoidRootPart") and char:FindFirstChild("Head") and player.Team ~= LocalPlayer.Team and Settings.ESP then
			local hrp = char.HumanoidRootPart
			local head = char.Head
			local pos, vis = Camera:WorldToViewportPoint(hrp.Position)
			if vis then
				local sizeY = 3.5 * (1 / (hrp.Position - Camera.CFrame.Position).Magnitude) * 1000
				box.Size = Vector2.new(sizeY * 0.6, sizeY)
				box.Position = Vector2.new(pos.X - box.Size.X / 2, pos.Y - box.Size.Y / 2)
				box.Visible = true

				local headPos, onScreen = Camera:WorldToViewportPoint(head.Position)
				line.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
				line.To = Vector2.new(headPos.X, headPos.Y)
				line.Visible = true
			else
				box.Visible = false
				line.Visible = false
			end
		else
			box.Visible = false
			line.Visible = false
		end
	end
end)
