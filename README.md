local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")

if not LocalPlayer.PlayerGui:FindFirstChild("HeadScalerUI") then
	local ScreenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
	ScreenGui.Name = "HeadScalerUI"

	-- 🔥 UIを死んでも消えないように
	ScreenGui.ResetOnSpawn = false

	-- 🔥 UIをどんなUIよりも最前線に固定
	ScreenGui.DisplayOrder = 999999

	local Frame = Instance.new("Frame", ScreenGui)
	Frame.AnchorPoint = Vector2.new(0.5, 0.5)
	Frame.Position = UDim2.new(0.5, 0, 0.85, 0)
	Frame.Size = UDim2.new(0, 240, 0, 110)
	Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
	Frame.BackgroundTransparency = 0.2
	Frame.BorderSizePixel = 0
	Frame.Active = true
	Frame.Draggable = true

	local Title = Instance.new("TextLabel", Frame)
	Title.Size = UDim2.new(1, 0, 0, 25)
	Title.BackgroundTransparency = 1
	Title.Text = "Hit box"
	Title.TextColor3 = Color3.fromRGB(255, 255, 255)
	Title.Font = Enum.Font.SourceSansBold
	Title.TextSize = 18

	local TextBox = Instance.new("TextBox", Frame)
	TextBox.PlaceholderText = "大きさを入力 (例: 3)"
	TextBox.Text = "2"
	TextBox.Size = UDim2.new(0, 100, 0, 30)
	TextBox.Position = UDim2.new(0.05, 0, 0.5, 0)
	TextBox.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
	TextBox.TextColor3 = Color3.fromRGB(255, 255, 255)
	TextBox.TextSize = 18
	TextBox.ClearTextOnFocus = true

	local Button = Instance.new("TextButton", Frame)
	Button.Size = UDim2.new(0, 100, 0, 30)
	Button.Position = UDim2.new(0.55, 0, 0.5, 0)
	Button.BackgroundColor3 = Color3.fromRGB(80, 0, 150)
	Button.Text = "オン"
	Button.TextColor3 = Color3.fromRGB(255, 255, 255)
	Button.Font = Enum.Font.SourceSansBold
	Button.TextSize = 18
	Button.AutoButtonColor = true

	-- ここから下はあなたのコードそのまま（変更なし）
	---------------------------------------------------------
	local Enabled = false
	local UpdateConnection
	local PlayerAddedConnection

	local function getHeadOrTopPart(character)
		local head = character:FindFirstChild("Head")
		if head then return head end
		local topPart
		local topY = -math.huge
		for _, part in pairs(character:GetChildren()) do
			if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
				if part.Position.Y > topY then
					topY = part.Position.Y
					topPart = part
				end
			end
		end
		return topPart
	end

	local function setHeadSize(player, size)
		local character = player.Character
		if not character then return end
		local part = getHeadOrTopPart(character)
		if not part then return end

		part.Transparency = 0.5
		part.Size = Vector3.new(size, size, size)
		part.CanCollide = true
		part.Massless = true
		part:SetAttribute("ScaledHead", true)

		local mesh = part:FindFirstChildOfClass("SpecialMesh")
		if mesh then
			mesh.Scale = Vector3.new(size, size, size)
		end

		if not part:FindFirstChild("HitboxPart") then
			local hitbox = Instance.new("Part")
			hitbox.Name = "HitboxPart"
			hitbox.Anchored = false
			hitbox.CanCollide = false
			hitbox.Transparency = 1
			hitbox.Size = Vector3.new(size * 1.8, size * 1.8, size * 1.8)
			hitbox.Massless = true
			hitbox.Parent = part
			local weld = Instance.new("WeldConstraint")
			weld.Part0 = part
			weld.Part1 = hitbox
			weld.Parent = part
		end

		if not part:FindFirstChild("Outline") then
			local outline = Instance.new("SelectionBox")
			outline.Name = "Outline"
			outline.Adornee = part
			outline.LineThickness = 0.08
			outline.Color3 = Color3.fromRGB(255, 0, 0)
			outline.SurfaceTransparency = 1
			outline.Parent = part
		end
	end

	local function resetHead(player)
		local character = player.Character
		if not character then return end
		local part = getHeadOrTopPart(character)
		if not part then return end

		part.Transparency = 0
		part.Size = Vector3.new(2, 1, 1)
		part.CanCollide = false
		part:SetAttribute("ScaledHead", false)

		local mesh = part:FindFirstChildOfClass("SpecialMesh")
		if mesh then
			mesh.Scale = Vector3.new(1, 1, 1)
		end

		local hitbox = part:FindFirstChild("HitboxPart")
		if hitbox then hitbox:Destroy() end

		local outline = part:FindFirstChild("Outline")
		if outline then outline:Destroy() end
	end

	local function toggle(enable)
		if enable then
			local size = tonumber(TextBox.Text) or 2

			UpdateConnection = RunService.RenderStepped:Connect(function()
				for _, player in pairs(Players:GetPlayers()) do
					if player ~= LocalPlayer then
						setHeadSize(player, size)
					end
				end
			end)

			PlayerAddedConnection = Players.PlayerAdded:Connect(function(player)
				player.CharacterAdded:Connect(function()
					if Enabled then
						task.wait(1)
						setHeadSize(player, tonumber(TextBox.Text) or 2)
					end
				end)
			end)
		else
			if UpdateConnection then UpdateConnection:Disconnect() end
			if PlayerAddedConnection then PlayerAddedConnection:Disconnect() end

			for _, player in pairs(Players:GetPlayers()) do
				if player ~= LocalPlayer then
					resetHead(player)
				end
			end
		end
	end

	Button.MouseButton1Click:Connect(function()
		Enabled = not Enabled
		Button.Text = Enabled and "オフ" or "オン"
		Button.BackgroundColor3 = Enabled and Color3.fromRGB(200, 50, 50) or Color3.fromRGB(80, 0, 150)
		toggle(Enabled)
	end)
end    btn.BackgroundColor3 = Color3.fromRGB(50,50,50)
    btn.TextColor3 = Color3.new(1,1,1)
    btn.TextScaled = true
    btn.Parent = ScreenGui
    return btn
end

local FlyButton = createButton("FlyButton", "Fly", UDim2.new(0.8,0,0.05,0))
local SpeedUpButton = createButton("SpeedUp", "+", UDim2.new(0.7,0,0.05,0))
local SpeedDownButton = createButton("SpeedDown", "-", UDim2.new(0.7,0,0.12,0))

-- Flyボタン押下で飛行切替
FlyButton.MouseButton1Click:Connect(function()
    flying = not flying
    if flying then
        Humanoid:ChangeState(Enum.HumanoidStateType.Physics)
        RootPart.Anchored = false
    else
        Humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
        RootPart.Velocity = Vector3.new(0,0,0)
    end
end)

-- 速度調整
SpeedUpButton.MouseButton1Click:Connect(function()
    flySpeed = flySpeed + flySpeedStep
end)
SpeedDownButton.MouseButton1Click:Connect(function()
    flySpeed = math.max(4, flySpeed - flySpeedStep)
end)

-- 飛行処理
RunService.RenderStepped:Connect(function(delta)
    if flying then
        local moveDir = Humanoid.MoveDirection -- 通常の移動方向
        if moveDir.Magnitude > 0 then
            -- 視点方向を加味して移動
            local camLook = Camera.CFrame.LookVector
            local finalDir = (moveDir + Vector3.new(0, camLook.Y, 0)).Unit
            RootPart.Velocity = finalDir * flySpeed
        else
            RootPart.Velocity = Vector3.new(0,0,0)
        end
    end
end)
