local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local RootPart = Character:WaitForChild("HumanoidRootPart")
local Camera = workspace.CurrentCamera

-- 状態
local flying = false
local flySpeed = 16 -- 初期速度は通常歩行
local flySpeedStep = 4 -- + - で増減する量

-- GUI作成
local ScreenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))

local function createButton(name, text, pos)
    local btn = Instance.new("TextButton")
    btn.Name = name
    btn.Size = UDim2.new(0, 60, 0, 40)
    btn.Position = pos
    btn.Text = text
    btn.BackgroundColor3 = Color3.fromRGB(50,50,50)
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
