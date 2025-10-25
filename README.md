local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer
local pg = player:WaitForChild("PlayerGui")

-- ===== util =====
local function makeCorner(parent, rad)
	local c = Instance.new("UICorner", parent)
	c.CornerRadius = UDim.new(0, rad or 8)
	return c
end
local function makeStroke(parent, thickness)
	local s = Instance.new("UIStroke", parent)
	s.Thickness = thickness or 2
	s.LineJoinMode = Enum.LineJoinMode.Round
	return s
end

-- ===== Interface principal (menu) =====
local screenGuiMain = Instance.new("ScreenGui")
screenGuiMain.Name = "KGN_LocalGui_Main"
screenGuiMain.ResetOnSpawn = false
screenGuiMain.Parent = pg

local mainFrame = Instance.new("Frame", screenGuiMain)
mainFrame.Size = UDim2.new(0,420,0,280)
mainFrame.Position = UDim2.new(0.3,0,0.3,0)
mainFrame.BackgroundColor3 = Color3.fromRGB(25,25,25)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
makeCorner(mainFrame, 12)
local stroke = makeStroke(mainFrame, 3)
spawn(function()
	while mainFrame and mainFrame.Parent do
		local t = tick()*0.12
		stroke.Color = Color3.fromHSV(t%1, 0.95, 1)
		task.wait(0.03)
	end
end)

local top = Instance.new("Frame", mainFrame)
top.Size = UDim2.new(1,0,0,42)
top.Position = UDim2.new(0,0,0,0)
top.BackgroundColor3 = Color3.fromRGB(20,20,20)
makeCorner(top, 10)

local title = Instance.new("TextLabel", top)
title.Text = "KGN"
title.Size = UDim2.new(1,-50,1,0)
title.Position = UDim2.new(0,10,0,0)
title.BackgroundTransparency = 1
title.TextColor3 = Color3.fromRGB(240,240,240)
title.Font = Enum.Font.GothamBold
title.TextScaled = true

local closeBtn = Instance.new("TextButton", top)
closeBtn.Size = UDim2.new(0,36,0,28)
closeBtn.Position = UDim2.new(1,-44,0.5,-14)
closeBtn.Text = "✕"
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextScaled = true
closeBtn.BackgroundColor3 = Color3.fromRGB(60,60,60)
closeBtn.TextColor3 = Color3.fromRGB(255,255,255)
makeCorner(closeBtn, 8)

local openGui = Instance.new("ScreenGui", pg)
openGui.Name = "KGN_OpenGui"
openGui.ResetOnSpawn = false
openGui.Enabled = false
local openBtn = Instance.new("TextButton", openGui)
openBtn.Text = "KGN"
openBtn.Size = UDim2.new(0,90,0,36)
openBtn.Position = UDim2.new(0,12,0,12)
openBtn.BackgroundColor3 = Color3.fromRGB(50,50,50)
openBtn.Font = Enum.Font.GothamBold
openBtn.TextScaled = true
openBtn.TextColor3 = Color3.fromRGB(245,245,245)
makeCorner(openBtn, 8)

local content = Instance.new("Frame", mainFrame)
content.Size = UDim2.new(1,-20,1,-70)
content.Position = UDim2.new(0,10,0,50)
content.BackgroundTransparency = 1

local leftCol = Instance.new("Frame", content)
leftCol.Size = UDim2.new(0,140,1,0)
leftCol.Position = UDim2.new(0,0,0,0)
leftCol.BackgroundColor3 = Color3.fromRGB(30,30,30)
makeCorner(leftCol, 10)

local mainTab = Instance.new("TextButton", leftCol)
mainTab.Size = UDim2.new(1,-20,0,40)
mainTab.Position = UDim2.new(0,10,0,12)
mainTab.Text = "Main"
mainTab.Font = Enum.Font.Gotham
mainTab.TextScaled = true
mainTab.BackgroundColor3 = Color3.fromRGB(40,40,40)
mainTab.TextColor3 = Color3.fromRGB(235,235,235)
makeCorner(mainTab, 8)

local rightPanel = Instance.new("Frame", content)
rightPanel.Size = UDim2.new(1,-160,1,0)
rightPanel.Position = UDim2.new(0,160,0,0)
rightPanel.BackgroundColor3 = Color3.fromRGB(22,22,22)
makeCorner(rightPanel, 10)

local rightLayout = Instance.new("UIListLayout", rightPanel)
rightLayout.Padding = UDim.new(0,10)
rightLayout.FillDirection = Enum.FillDirection.Vertical
rightLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

local function clearRight()
	for _,c in ipairs(rightPanel:GetChildren()) do
		if c:IsA("Frame") then c:Destroy() end
	end
end

local function makeToggleRow(name)
	local row = Instance.new("Frame", rightPanel)
	row.Size = UDim2.new(0.95,0,0,44)
	row.BackgroundColor3 = Color3.fromRGB(36,36,38)
	makeCorner(row, 8)
	local lbl = Instance.new("TextLabel", row)
	lbl.Size = UDim2.new(0.7,0,1,0)
	lbl.Position = UDim2.new(0,12,0,0)
	lbl.BackgroundTransparency = 1
	lbl.Text = name
	lbl.Font = Enum.Font.Gotham
	lbl.TextColor3 = Color3.fromRGB(235,235,235)
	lbl.TextScaled = true
	lbl.TextXAlignment = Enum.TextXAlignment.Left

	local btn = Instance.new("TextButton", row)
	btn.Size = UDim2.new(0,72,0,30)
	btn.Position = UDim2.new(1,-86,0.5,-15)
	btn.BackgroundColor3 = Color3.fromRGB(70,70,75)
	btn.Text = "OFF"
	btn.Font = Enum.Font.GothamBold
	btn.TextScaled = true
	btn.TextColor3 = Color3.fromRGB(245,245,245)
	makeCorner(btn, 8)

	return {Frame=row, Label=lbl, Button=btn}
end

local state = { esp = false, fly = false, jump = false, aimbot = false }
local espHighlights = {}

-- ESP com team check
local function enableESP()
	for _, pl in pairs(Players:GetPlayers()) do
		if pl ~= player and pl.Character and not espHighlights[pl] then
			if pl.Team and player.Team and pl.Team ~= player.Team then
				local h = Instance.new("Highlight")
				h.Adornee = pl.Character
				h.FillTransparency = 0.65
				h.OutlineTransparency = 0
				h.Parent = pl.Character
				espHighlights[pl] = h
			end
		end
	end
end
local function disableESP()
	for pl,h in pairs(espHighlights) do
		if h and h.Parent then h:Destroy() end
	end
	espHighlights = {}
end

-- Infinite jump
local jumpConn
local function setInfiniteJump(on)
	state.jump = on
	if on then
		jumpConn = UserInputService.JumpRequest:Connect(function()
			local char = player.Character
			if char and char:FindFirstChildOfClass("Humanoid") then
				local hum = char:FindFirstChildOfClass("Humanoid")
				hum:ChangeState(Enum.HumanoidStateType.Jumping)
			end
		end)
	else
		if jumpConn then
			jumpConn:Disconnect()
			jumpConn = nil
		end
	end
end

-- Fly (local, resumido)
local flying = false
local flyVelocityConnection
local flySpeed = 50
local flyControls = {F=false,B=false,L=false,R=false,Up=false,Down=false}
local inputConn, inputEndConn

local function startFlyLocal()
	if flying then return end
	local char = player.Character
	if not char then return end
	local hrp = char:FindFirstChild("HumanoidRootPart")
	local hum = char:FindFirstChildOfClass("Humanoid")
	if not hrp or not hum then return end

	flying = true
	hum.PlatformStand = true

	local bv = Instance.new("BodyVelocity")
	bv.MaxForce = Vector3.new(1e5,1e5,1e5)
	bv.P = 12500
	bv.Velocity = Vector3.new(0,0,0)
	bv.Parent = hrp

	inputConn = UserInputService.InputBegan:Connect(function(inp, gpe)
		if gpe then return end
		if inp.KeyCode == Enum.KeyCode.W then flyControls.F = true end
		if inp.KeyCode == Enum.KeyCode.S then flyControls.B = true end
		if inp.KeyCode == Enum.KeyCode.A then flyControls.L = true end
		if inp.KeyCode == Enum.KeyCode.D then flyControls.R = true end
		if inp.KeyCode == Enum.KeyCode.Space then flyControls.Up = true end
		if inp.KeyCode == Enum.KeyCode.LeftControl then flyControls.Down = true end
	end)
	inputEndConn = UserInputService.InputEnded:Connect(function(inp)
		if inp.KeyCode == Enum.KeyCode.W then flyControls.F = false end
		if inp.KeyCode == Enum.KeyCode.S then flyControls.B = false end
		if inp.KeyCode == Enum.KeyCode.A then flyControls.L = false end
		if inp.KeyCode == Enum.KeyCode.D then flyControls.R = false end
		if inp.KeyCode == Enum.KeyCode.Space then flyControls.Up = false end
		if inp.KeyCode == Enum.KeyCode.LeftControl then flyControls.Down = false end
	end)

	flyVelocityConnection = RunService.RenderStepped:Connect(function()
		if not flying or not hrp then return end
		local cam = workspace.CurrentCamera
		local dir = Vector3.new(0,0,0)
		local look = cam.CFrame
		if flyControls.F then dir = dir + look.LookVector end
		if flyControls.B then dir = dir - look.LookVector end
		if flyControls.L then dir = dir - look.RightVector end
		if flyControls.R then dir = dir + look.RightVector end
		if flyControls.Up then dir = dir + Vector3.new(0,1,0) end
		if flyControls.Down then dir = dir - Vector3.new(0,1,0) end
		if dir.Magnitude > 0 then dir = dir.Unit * flySpeed end
		bv.Velocity = dir
	end)

	flyVelocityConnection._bv = bv
	flyVelocityConnection._inputConn = inputConn
	flyVelocityConnection._inputEndConn = inputEndConn
end

local function stopFlyLocal()
	if not flying then return end
	local char = player.Character
	if char and char:FindFirstChildOfClass("Humanoid") then
		char:FindFirstChildOfClass("Humanoid").PlatformStand = false
	end
	if flyVelocityConnection then
		if flyVelocityConnection._inputConn then flyVelocityConnection._inputConn:Disconnect() end
		if flyVelocityConnection._inputEndConn then flyVelocityConnection._inputEndConn:Disconnect() end
		if flyVelocityConnection._bv and flyVelocityConnection._bv.Parent then flyVelocityConnection._bv:Destroy() end
		flyVelocityConnection:Disconnect()
		flyVelocityConnection = nil
	end
	flying = false
end

-- AIMBOT normal (team check)
local aimbotConn = nil
local function getClosestTarget()
	local closestTarget = nil
	local shortestDistance = 9999
	local myChar = player.Character
	local myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
	if not myRoot then return nil end

	for _, v in pairs(workspace:GetChildren()) do
		if v:FindFirstChild("Humanoid") and v ~= myChar then
			local plr = Players:GetPlayerFromCharacter(v)
			if plr and plr.Team and player.Team and plr.Team ~= player.Team then
				local rootPart = v:FindFirstChild("HumanoidRootPart")
				if rootPart then
					local dist = (myRoot.Position - rootPart.Position).Magnitude
					if dist < shortestDistance then
						shortestDistance = dist
						closestTarget = rootPart
					end
				end
			end
		end
	end
	return closestTarget
end

local function setAimbot(on)
	state.aimbot = on
	if on then
		aimbotConn = RunService.RenderStepped:Connect(function()
			local target = getClosestTarget()
			if target then
				workspace.CurrentCamera.CFrame = CFrame.new(
					workspace.CurrentCamera.CFrame.Position,
					target.Position
				)
			end
		end)
	else
		if aimbotConn then aimbotConn:Disconnect() aimbotConn = nil end
	end
end

-- ===== Build UI (right panel toggles) =====
local function buildMain()
	clearRight()
	local header = Instance.new("TextLabel", rightPanel)
	header.Size = UDim2.new(1,-20,0,30)
	header.Position = UDim2.new(0,10,0,6)
	header.BackgroundTransparency = 1
	header.Text = "Main"
	header.Font = Enum.Font.GothamBold
	header.TextScaled = true
	header.TextColor3 = Color3.fromRGB(240,240,240)

	local espRow = makeToggleRow("ESP")
	espRow.Frame.Parent = rightPanel
	espRow.Button.MouseButton1Click:Connect(function()
		local new = not state.esp
		if new then enableESP() else disableESP() end
		state.esp = new
		espRow.Button.Text = new and "ON" or "OFF"
		espRow.Button.BackgroundColor3 = new and Color3.fromRGB(120,0,255) or Color3.fromRGB(70,70,75)
	end)

	local flyRow = makeToggleRow("Fly")
	flyRow.Frame.Parent = rightPanel
	flyRow.Button.MouseButton1Click:Connect(function()
		local new = not state.fly
		if new then startFlyLocal() else stopFlyLocal() end
		state.fly = new
		flyRow.Button.Text = new and "ON" or "OFF"
		flyRow.Button.BackgroundColor3 = new and Color3.fromRGB(120,0,255) or Color3.fromRGB(70,70,75)
	end)

	local ijRow = makeToggleRow("Infinite Jump")
	ijRow.Frame.Parent = rightPanel
	ijRow.Button.MouseButton1Click:Connect(function()
		local new = not state.jump
		setInfiniteJump(new)
		state.jump = new
		ijRow.Button.Text = new and "ON" or "OFF"
		ijRow.Button.BackgroundColor3 = new and Color3.fromRGB(120,0,255) or Color3.fromRGB(70,70,75)
	end)

	local aimRow = makeToggleRow("Aimbot")
	aimRow.Frame.Parent = rightPanel
	aimRow.Button.MouseButton1Click:Connect(function()
		local new = not state.aimbot
		setAimbot(new)
		aimRow.Button.Text = new and "ON" or "OFF"
		aimRow.Button.BackgroundColor3 = new and Color3.fromRGB(120,0,255) or Color3.fromRGB(70,70,75)
	end)
end

mainTab.MouseButton1Click:Connect(function() buildMain() end)
buildMain()

closeBtn.MouseButton1Click:Connect(function()
	mainFrame.Visible = false
	openGui.Enabled = true
end)
openBtn.MouseButton1Click:Connect(function()
	mainFrame.Visible = true
	openGui.Enabled = false
end)

player.CharacterRemoving:Connect(function()
	setAimbot(false)
	if jumpConn then jumpConn:Disconnect() jumpConn = nil end
	disableESP()
	stopFlyLocal()
end)
screenGuiMain.Destroying:Connect(function()
	setAimbot(false)
	if jumpConn then jumpConn:Disconnect() end
	disableESP()
	stopFlyLocal()
end)

-- ===== Botões no canto direito para ESP e AIMBOT =====
local screenGuiCanto = Instance.new("ScreenGui")
screenGuiCanto.Name = "KGN_Indicators"
screenGuiCanto.ResetOnSpawn = false
screenGuiCanto.Parent = pg

local indicatorGui = Instance.new("Frame")
indicatorGui.Name = "KGN_Indicators_Frame"
indicatorGui.Parent = screenGuiCanto
indicatorGui.Size = UDim2.new(0,140,0,88)
indicatorGui.Position = UDim2.new(1,-150,0.2,0)
indicatorGui.BackgroundTransparency = 1

local espBtn = Instance.new("TextButton")
espBtn.Parent = indicatorGui
espBtn.Size = UDim2.new(1,0,0,36)
espBtn.Position = UDim2.new(0,0,0,0)
espBtn.BackgroundColor3 = Color3.fromRGB(30,30,35)
espBtn.BackgroundTransparency = 0.25
espBtn.Text = "ESP (OFF)"
espBtn.TextColor3 = Color3.fromRGB(220,220,220)
espBtn.Font = Enum.Font.GothamBold
espBtn.TextScaled = true
makeCorner(espBtn, 10)

local aimbotBtn = Instance.new("TextButton")
aimbotBtn.Parent = indicatorGui
aimbotBtn.Size = UDim2.new(1,0,0,36)
aimbotBtn.Position = UDim2.new(0,0,0,44)
aimbotBtn.BackgroundColor3 = Color3.fromRGB(30,30,35)
aimbotBtn.BackgroundTransparency = 0.25
aimbotBtn.Text = "Aimbot (OFF)"
aimbotBtn.TextColor3 = Color3.fromRGB(220,220,220)
aimbotBtn.Font = Enum.Font.GothamBold
aimbotBtn.TextScaled = true
makeCorner(aimbotBtn, 10)

local function updateESPBtn(on)
	espBtn.Text = on and "ESP (ON)" or "ESP (OFF)"
	espBtn.BackgroundColor3 = on and Color3.fromRGB(120,0,255) or Color3.fromRGB(30,30,35)
	espBtn.TextColor3 = on and Color3.fromRGB(255,255,255) or Color3.fromRGB(220,220,220)
end

local function updateAimbotBtn(on)
	aimbotBtn.Text = on and "Aimbot (ON)" or "Aimbot (OFF)"
	aimbotBtn.BackgroundColor3 = on and Color3.fromRGB(120,0,255) or Color3.fromRGB(30,30,35)
	aimbotBtn.TextColor3 = on and Color3.fromRGB(255,255,255) or Color3.fromRGB(220,220,220)
end

espBtn.MouseButton1Click:Connect(function()
	state.esp = not state.esp
	if state.esp then enableESP() else disableESP() end
	updateESPBtn(state.esp)
end)
aimbotBtn.MouseButton1Click:Connect(function()
	setAimbot(not state.aimbot)
	updateAimbotBtn(state.aimbot)
end)

print("KGN GUI completo: Interface principal + botões no canto direito para ESP e AIMBOT!")
