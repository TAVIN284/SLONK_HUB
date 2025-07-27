-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- GUI Principal
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "UltimateHackUI"

-- Janela principal
local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 220, 0, 210)
frame.Position = UDim2.new(0.03, 0, 0.3, 0)
frame.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
frame.BorderSizePixel = 0
Instance.new("UICorner", frame)
Instance.new("UIStroke", frame).Color = Color3.fromRGB(0, 200, 255)

local layout = Instance.new("UIListLayout", frame)
layout.Padding = UDim.new(0, 10)
layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
layout.VerticalAlignment = Enum.VerticalAlignment.Center

-- Botão para reabrir
local openButton = Instance.new("TextButton", gui)
openButton.Size = UDim2.new(0, 120, 0, 35)
openButton.Position = UDim2.new(0.03, 0, 0.52, 0)
openButton.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
openButton.Text = "Abrir Script"
openButton.Visible = false
openButton.TextColor3 = Color3.new(1, 1, 1)
openButton.Font = Enum.Font.GothamBold
openButton.TextSize = 14
Instance.new("UICorner", openButton)

-- Estados
local toggles = {
	Speed = false,
	Jump = false,
	ESP = false
}

-- Utilidades
local function getHumanoid()
	local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
	return char:WaitForChild("Humanoid")
end

-- Criar botão
local function createButton(name, color)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(0, 180, 0, 40)
	btn.BackgroundColor3 = color
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 16
	btn.TextColor3 = Color3.new(1, 1, 1)
	btn.Text = "Ativar " .. name
	Instance.new("UICorner", btn)
	btn.Parent = frame
	return btn
end

-- SPEED
local speedConn
local function toggleSpeed()
	toggles.Speed = not toggles.Speed
	if toggles.Speed then
		speedConn = RunService.Heartbeat:Connect(function()
			getHumanoid().WalkSpeed = 70
		end)
	else
		if speedConn then speedConn:Disconnect() end
		getHumanoid().WalkSpeed = 16
	end
end

-- JUMP
local jumpConn
local function toggleJump()
	toggles.Jump = not toggles.Jump
	local hum = getHumanoid()
	if toggles.Jump then
		hum.JumpPower = 320
		hum.UseJumpPower = true
		jumpConn = RunService.RenderStepped:Connect(function()
			if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
				hum:ChangeState(Enum.HumanoidStateType.Jumping)
			end
		end)
	else
		if jumpConn then jumpConn:Disconnect() end
		hum.JumpPower = 50
	end
end

-- ESP
local function createESP(player)
	if player == LocalPlayer then return end
	local function apply()
		local char = player.Character
		if not char or not char:FindFirstChild("Head") then return end
		local head = char.Head
		if head:FindFirstChild("ESP") then return end

		local esp = Instance.new("BillboardGui", head)
		esp.Name = "ESP"
		esp.Size = UDim2.new(0, 100, 0, 40)
		esp.AlwaysOnTop = true
		esp.Adornee = head
		esp.StudsOffset = Vector3.new(0, 2.5, 0)

		local tag = Instance.new("TextLabel", esp)
		tag.Size = UDim2.new(1, 0, 1, 0)
		tag.BackgroundTransparency = 0.3
		tag.BackgroundColor3 = Color3.new(0, 0, 0)
		tag.Text = player.Name
		tag.TextColor3 = Color3.new(1, 0, 0)
		tag.TextStrokeTransparency = 0.3
		tag.Font = Enum.Font.GothamBold
		tag.TextSize = 14
	end

	apply()
	player.CharacterAdded:Connect(function()
		wait(1)
		if toggles.ESP then apply() end
	end)
end

local function removeESP(player)
	local char = player.Character
	if char and char:FindFirstChild("Head") then
		local esp = char.Head:FindFirstChild("ESP")
		if esp then esp:Destroy() end
	end
end

local function toggleESP()
	toggles.ESP = not toggles.ESP
	for _, p in pairs(Players:GetPlayers()) do
		if toggles.ESP then
			createESP(p)
		else
			removeESP(p)
		end
	end
end

-- Botões
local btnSpeed = createButton("Speed", Color3.fromRGB(60, 200, 120))
local btnJump = createButton("Super Jump", Color3.fromRGB(0, 160, 255))
local btnESP = createButton("ESP", Color3.fromRGB(255, 80, 80))
local btnClose = createButton("Fechar Script", Color3.fromRGB(100, 100, 100))

btnSpeed.MouseButton1Click:Connect(function()
	toggleSpeed()
	btnSpeed.Text = toggles.Speed and "Desativar Speed" or "Ativar Speed"
end)

btnJump.MouseButton1Click:Connect(function()
	toggleJump()
	btnJump.Text = toggles.Jump and "Desativar Jump" or "Ativar Jump"
end)

btnESP.MouseButton1Click:Connect(function()
	toggleESP()
	btnESP.Text = toggles.ESP and "Desativar ESP" or "Ativar ESP"
end)

btnClose.MouseButton1Click:Connect(function()
	frame.Visible = false
	openButton.Visible = true
end)

openButton.MouseButton1Click:Connect(function()
	frame.Visible = true
	openButton.Visible = false
end)
