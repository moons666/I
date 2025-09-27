-- moons bypass (KRNL) completo
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

local MENU_NAME = "moons bypass"
local DEFAULT_WALKSPEED = 16
local FAST_WALKSPEED = 100
local SUPERJUMP_FORCE = 200

-- Estados
local uiEnabled = true
local noclipEnabled = false
local teleportClickEnabled = false
local flyEnabled = false
local speedToggled = false
local superJumpEnabled = false

local conns = {
	noclip = nil,
	teleport = nil,
	fly_loop = nil,
	fly_inputs = {},
}

-- GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "MoonsBypass_GUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = game:GetService("CoreGui")

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 360, 0, 230)
mainFrame.AnchorPoint = Vector2.new(0.5,0.5)
mainFrame.Position = UDim2.new(0.5,0,0.5,0)
mainFrame.BackgroundColor3 = Color3.fromRGB(18,16,30)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1,0,0,36)
title.BackgroundTransparency = 1
title.Text = MENU_NAME
title.TextScaled = true
title.Font = Enum.Font.SourceSansBold
title.TextColor3 = Color3.fromRGB(200,180,255)
title.Parent = mainFrame

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0,26,0,26)
closeBtn.Position = UDim2.new(1,-30,0,6)
closeBtn.Text = "X"
closeBtn.Font = Enum.Font.SourceSansBold
closeBtn.TextSize = 18
closeBtn.Parent = mainFrame

local openBtn = Instance.new("TextButton")
openBtn.Size = UDim2.new(0,180,0,30)
openBtn.Position = UDim2.new(1,-190,0,10)
openBtn.Text = "abrir moons bypass"
openBtn.Font = Enum.Font.SourceSansBold
openBtn.TextSize = 16
openBtn.BackgroundColor3 = Color3.fromRGB(28,26,48)
openBtn.BorderSizePixel = 0
openBtn.TextColor3 = Color3.fromRGB(230,230,230)
openBtn.Visible = false
openBtn.Parent = screenGui

-- Função criar botão
local function makeButton(text,posX,posY)
	local b = Instance.new("TextButton")
	b.Size = UDim2.new(0,160,0,34)
	b.Position = UDim2.new(0,posX,0,posY)
	b.Text = text
	b.Font = Enum.Font.SourceSans
	b.TextSize = 16
	b.BackgroundColor3 = Color3.fromRGB(28,26,48)
	b.BorderSizePixel = 0
	b.TextColor3 = Color3.fromRGB(230,230,230)
	b.Parent = mainFrame
	return b
end

local btnSpeed = makeButton("WalkSpeed 100",12,46)
local btnNoclip = makeButton("Noclip (OFF)",188,46)
local btnTeleport = makeButton("Teleport (Click) (OFF)",12,92)
local btnFly = makeButton("Fly (OFF)",188,92)
local btnSuperJump = makeButton("Super Jump (OFF)",12,138)

-- Dragging
do
	local UIS = UserInputService
	local dragging = false
	local dragStart
	local startPos
	local dragInput
	local function update(input)
		local delta = input.Position - dragStart
		mainFrame.Position = UDim2.new(startPos.X.Scale,startPos.X.Offset+delta.X,startPos.Y.Scale,startPos.Y.Offset+delta.Y)
	end
	mainFrame.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
			dragStart = input.Position
			startPos = mainFrame.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)
	mainFrame.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement then
			dragInput = input
		end
	end)
	UIS.InputChanged:Connect(function(input)
		if input == dragInput and dragging then update(input) end
	end)
end

-- Mostrar/ocultar menu
local function showMenu()
	mainFrame.Visible = true
	openBtn.Visible = false
	uiEnabled = true
end
local function hideMenu()
	mainFrame.Visible = false
	openBtn.Visible = true
	uiEnabled = false
end

closeBtn.MouseButton1Click:Connect(hideMenu)
openBtn.MouseButton1Click:Connect(showMenu)
UserInputService.InputBegan:Connect(function(inp, gameProcessed)
	if gameProcessed then return end
	if inp.KeyCode == Enum.KeyCode.RightControl then
		if uiEnabled then hideMenu() else showMenu() end
	end
end)

-- WalkSpeed toggle
btnSpeed.MouseButton1Click:Connect(function()
	local char = LocalPlayer.Character
	if not char then return end
	local humanoid = char:FindFirstChildOfClass("Humanoid")
	if not humanoid then return end
	speedToggled = not speedToggled
	if speedToggled then
		humanoid.WalkSpeed = FAST_WALKSPEED
		btnSpeed.Text = "WalkSpeed: 100 (ON)"
	else
		humanoid.WalkSpeed = DEFAULT_WALKSPEED
		btnSpeed.Text = "WalkSpeed 100"
	end
end)

-- Noclip toggle
btnNoclip.MouseButton1Click:Connect(function()
	noclipEnabled = not noclipEnabled
	btnNoclip.Text = "Noclip ("..(noclipEnabled and "ON" or "OFF")..")"
	if noclipEnabled then
		conns.noclip = RunService.Stepped:Connect(function()
			local char = LocalPlayer.Character
			if not char then return end
			for _, part in ipairs(char:GetDescendants()) do
				if part:IsA("BasePart") then part.CanCollide = false end
			end
		end)
	else
		if conns.noclip then pcall(function() conns.noclip:Disconnect() end) conns.noclip=nil end
	end
end)

-- Teleport toggle
btnTeleport.MouseButton1Click:Connect(function()
	teleportClickEnabled = not teleportClickEnabled
	btnTeleport.Text = "Teleport (Click) ("..(teleportClickEnabled and "ON" or "OFF")..")"
	if teleportClickEnabled then
		if not conns.teleport then
			conns.teleport = Mouse.Button1Down:Connect(function()
				local hit = Mouse.Hit
				local char = LocalPlayer.Character
				if hit and char and char:FindFirstChild("HumanoidRootPart") then
					local targetPos = hit.p + Vector3.new(0,3,0)
					pcall(function() char.HumanoidRootPart.CFrame = CFrame.new(targetPos) end)
				end
			end)
		end
	else
		if conns.teleport then pcall(function() conns.teleport:Disconnect() end) conns.teleport=nil end
	end
end)

-- Fly externo
btnFly.MouseButton1Click:Connect(function()
	flyEnabled = not flyEnabled
	btnFly.Text = "Fly ("..(flyEnabled and "ON" or "OFF")..")"
	if flyEnabled then
		pcall(function()
			loadstring(game:HttpGet("https://raw.githubusercontent.com/ahmedmahmoudawaysali123456789-sys/script1/refs/heads/main/Protected_7380455261315382.lua.txt"))()
		end)
	end
end)

-- Super Jump ON/OFF
btnSuperJump.MouseButton1Click:Connect(function()
	superJumpEnabled = not superJumpEnabled
	btnSuperJump.Text = "Super Jump ("..(superJumpEnabled and "ON" or "OFF")..")"
end)

-- Conectar humanoid atual e futuros
local function setupSuperJump(char)
	local humanoid = char:FindFirstChildOfClass("Humanoid")
	if not humanoid then return end
	humanoid.StateChanged:Connect(function(_,newState)
		if superJumpEnabled and newState==Enum.HumanoidStateType.Jumping then
			local hrp = char:FindFirstChild("HumanoidRootPart")
			if hrp then
				local current = hrp.AssemblyLinearVelocity
				hrp.AssemblyLinearVelocity = Vector3.new(current.X,SUPERJUMP_FORCE,current.Z)
			end
		end
	end)
end

if LocalPlayer.Character then setupSuperJump(LocalPlayer.Character) end
LocalPlayer.CharacterAdded:Connect(setupSuperJump)

print("[moons bypass] carregado com sucesso! Todas as funções ativas.")
