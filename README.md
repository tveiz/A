--// Serviços
local Players = game:GetService("Players")
local RS = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

local lp = Players.LocalPlayer
local PlayerGui = lp:WaitForChild("PlayerGui")

-- Criar ScreenGui
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "CustomMenu"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = CoreGui

-- Função de arrastar
local function makeDraggable(frame, dragHandle)
	local dragging, dragInput, mousePos, framePos
	dragHandle = dragHandle or frame

	dragHandle.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			mousePos = input.Position
			framePos = frame.Position

			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)

	dragHandle.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
			dragInput = input
		end
	end)

	UIS.InputChanged:Connect(function(input)
		if input == dragInput and dragging then
			local delta = input.Position - mousePos
			frame.Position = UDim2.new(framePos.X.Scale, framePos.X.Offset + delta.X, framePos.Y.Scale, framePos.Y.Offset + delta.Y)
		end
	end)
end

-- Criar frame principal
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 350, 0, 250)
MainFrame.Position = UDim2.new(0.5, -175, 0.5, -125)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Parent = ScreenGui

makeDraggable(MainFrame)

-- Botões fechar e minimizar
local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 30, 0, 30)
CloseBtn.Position = UDim2.new(1, -35, 0, 5)
CloseBtn.Text = "X"
CloseBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
CloseBtn.Parent = MainFrame

local MinBtn = Instance.new("TextButton")
MinBtn.Size = UDim2.new(0, 30, 0, 30)
MinBtn.Position = UDim2.new(1, -70, 0, 5)
MinBtn.Text = "_"
MinBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
MinBtn.Parent = MainFrame

-- Minimizado
local MiniFrame = Instance.new("TextButton")
MiniFrame.Size = UDim2.new(0, 120, 0, 40)
MiniFrame.Position = UDim2.new(0.5, -60, 0.5, -20)
MiniFrame.Text = "🔳 Menu"
MiniFrame.Visible = false
MiniFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
MiniFrame.Parent = ScreenGui
makeDraggable(MiniFrame)

-- Funções fechar/minimizar
CloseBtn.MouseButton1Click:Connect(function()
	ScreenGui:Destroy()
end)

MinBtn.MouseButton1Click:Connect(function()
	MainFrame.Visible = false
	MiniFrame.Visible = true
end)

MiniFrame.MouseButton1Click:Connect(function()
	MainFrame.Visible = true
	MiniFrame.Visible = false
end)

-- Layout interno
local UIListLayout = Instance.new("UIListLayout")
UIListLayout.Padding = UDim.new(0, 8)
UIListLayout.FillDirection = Enum.FillDirection.Vertical
UIListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
UIListLayout.Parent = MainFrame

-- Função criar toggle
local function createToggle(name, callback)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(0, 300, 0, 30)
	btn.Text = name .. " [OFF]"
	btn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
	btn.Parent = MainFrame

	local state = false
	btn.MouseButton1Click:Connect(function()
		state = not state
		btn.Text = name .. (state and " [ON]" or " [OFF]")
		callback(state)
	end)
end

-- Função criar slider
local function createSlider(name, min, max, default, callback)
	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(0, 300, 0, 40)
	frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
	frame.Parent = MainFrame

	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(1, 0, 0.5, 0)
	label.Text = name .. ": " .. default
	label.TextColor3 = Color3.new(1,1,1)
	label.BackgroundTransparency = 1
	label.Parent = frame

	local slider = Instance.new("TextButton")
	slider.Size = UDim2.new(1, 0, 0.5, 0)
	slider.Position = UDim2.new(0, 0, 0.5, 0)
	slider.Text = ""
	slider.BackgroundColor3 = Color3.fromRGB(80,80,80)
	slider.Parent = frame

	local dragging = false
	local value = default

	local function update(inputX)
		local relative = math.clamp((inputX - slider.AbsolutePosition.X) / slider.AbsoluteSize.X, 0, 1)
		value = math.floor(min + (max - min) * relative)
		label.Text = name .. ": " .. value
		callback(value)
	end

	slider.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			update(input.Position.X)
		end
	end)

	slider.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = false
		end
	end)

	UIS.InputChanged:Connect(function(input)
		if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
			update(input.Position.X)
		end
	end)
end

-- ESP Toggle
local espEnabled = false
local espDistance = 100
local espConnections = {}

local function toggleESP(state)
	espEnabled = state
	if not state then
		for _, v in pairs(espConnections) do v:Disconnect() end
		espConnections = {}
	end
end

createToggle("ESP Nomes", toggleESP)

createSlider("ESP Distância", 50, 500, 100, function(val)
	espDistance = val
end)

RS.RenderStepped:Connect(function()
	if espEnabled then
		for _, p in pairs(Players:GetPlayers()) do
			if p ~= lp and p.Character and p.Character:FindFirstChild("Head") then
				local head = p.Character.Head
				if not head:FindFirstChild("BillboardGui") then
					local bb = Instance.new("BillboardGui", head)
					bb.Size = UDim2.new(0, 200, 0, 50)
					bb.Adornee = head
					bb.AlwaysOnTop = true
					local txt = Instance.new("TextLabel", bb)
					txt.Size = UDim2.new(1,0,1,0)
					txt.Text = p.Name
					txt.BackgroundTransparency = 1
					txt.TextColor3 = Color3.new(1,1,1)
				end
				local dist = (lp.Character.HumanoidRootPart.Position - head.Position).Magnitude
				head.BillboardGui.Enabled = dist <= espDistance
			end
		end
	end
end)

-- Auto Kick Toggle
createToggle("Auto Kick Vida <= 3", function(state)
	if state then
		espConnections["Kick"] = RS.Heartbeat:Connect(function()
			if lp.Character and lp.Character:FindFirstChild("Humanoid") then
				if lp.Character.Humanoid.Health <= 3 then
					lp:Kick("Kickado por baixa vida (<=3)")
				end
			end
		end)
	else
		if espConnections["Kick"] then
			espConnections["Kick"]:Disconnect()
			espConnections["Kick"] = nil
		end
	end
end)

-- Script externo toggle
local externalConnection
createToggle("Script Externo", function(state)
	if state then
		externalConnection = loadstring(game:HttpGet("https://gist.githubusercontent.com/Aimboter477387/582af6aec49782899d5d375ab239039e/raw/51b6ddf5dc74731a24f912134061f150b6f6b316/gistfile1.txt"))()
	else
		if externalConnection and type(externalConnection) == "function" then
			pcall(externalConnection, false)
		end
	end
end)

-- Button executar outro script
local btnScript = Instance.new("TextButton")
btnScript.Size = UDim2.new(0, 300, 0, 30)
btnScript.Text = "Executar Script Itens"
btnScript.BackgroundColor3 = Color3.fromRGB(60, 60, 120)
btnScript.Parent = MainFrame

btnScript.MouseButton1Click:Connect(function()
	pcall(function()
		PlayerGui:FindFirstChild("NotifyGui"):Destroy()
	end)

	local NotifyBackup = PlayerGui:FindFirstChild("NotifyGui") and PlayerGui:FindFirstChild("NotifyGui"):Clone()

	RS.Heartbeat:Connect(function()
		for _, Item in ipairs({
			"Glock 17", "Hi Power", "AK47", "PARAFAL", "Uzi", "G3", "IA2",
			string.char(65, 82, 45, 49, 53),
			"Faca", "Natalina", "Lockpick", "Escudo", "Skate", "Planta Suja",
			"Planta Limpa", "Tratamento"
		}) do
			game:GetService("ReplicatedStorage").Modules.InvRemotes.InvRequest:InvokeServer("mudaInv", "3", Item, "1")
			task.wait(0.1)
		end
	end)
end)
