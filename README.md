local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- 状態
local flying = false
local flySpeed = 16 -- 初期速度は通常歩行
local flyStep = 4
local guiExpanded = true

-- GUI作成（死んでも残る）
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "FlyGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- メインフレーム（丸みがあるポップデザイン）
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 200, 0, 100)
MainFrame.Position = UDim2.new(0.8,0,0.05,0)
MainFrame.BackgroundColor3 = Color3.fromRGB(30,30,30)
MainFrame.BackgroundTransparency = 0.1
MainFrame.BorderSizePixel = 0
MainFrame.AnchorPoint = Vector2.new(0.5,0)
MainFrame.Parent = ScreenGui
MainFrame.ClipsDescendants = true
MainFrame.RoundedCorner = UDim.new(0,15) -- 丸み追加

-- Flyボタン
local FlyButton = Instance.new("TextButton")
FlyButton.Size = UDim2.new(0, 80, 0, 40)
FlyButton.Position = UDim2.new(0.5, -40, 0.1, 0)
FlyButton.Text = "Fly"
FlyButton.BackgroundColor3 = Color3.fromRGB(255,255,255)
FlyButton.TextColor3 = Color3.fromRGB(0,0,0)
FlyButton.TextScaled = true
FlyButton.Parent = MainFrame

-- 速度調整ボタン
local SpeedUp = Instance.new("TextButton")
SpeedUp.Size = UDim2.new(0, 30, 0, 30)
SpeedUp.Position = UDim2.new(0.1,0,0.6,0)
SpeedUp.Text = "+"
SpeedUp.BackgroundColor3 = Color3.fromRGB(255,255,255)
SpeedUp.TextColor3 = Color3.fromRGB(0,0,0)
SpeedUp.TextScaled = true
SpeedUp.Parent = MainFrame

local SpeedDown = Instance.new("TextButton")
SpeedDown.Size = UDim2.new(0, 30, 0, 30)
SpeedDown.Position = UDim2.new(0.8,0,0.6,0)
SpeedDown.Text = "-"
SpeedDown.BackgroundColor3 = Color3.fromRGB(255,255,255)
SpeedDown.TextColor3 = Color3.fromRGB(0,0,0)
SpeedDown.TextScaled = true
SpeedDown.Parent = MainFrame

-- 縮小ボタン
local CollapseBtn = Instance.new("TextButton")
CollapseBtn.Size = UDim2.new(0,20,0,20)
CollapseBtn.Position = UDim2.new(0.85,0,0.02,0)
CollapseBtn.Text = "_"
CollapseBtn.BackgroundColor3 = Color3.fromRGB(255,255,255)
CollapseBtn.TextColor3 = Color3.fromRGB(0,0,0)
CollapseBtn.TextScaled = true
CollapseBtn.Parent = MainFrame

CollapseBtn.MouseButton1Click:Connect(function()
    guiExpanded = not guiExpanded
    MainFrame.Size = guiExpanded and UDim2.new(0,200,0,100) or UDim2.new(0,80,0,40)
end)

-- Flyボタン押下で飛行切替
FlyButton.MouseButton1Click:Connect(function()
    local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    local hum = char:WaitForChild("Humanoid")
    local root = char:WaitForChild("HumanoidRootPart")
    flying = not flying
    if flying then
        hum:ChangeState(Enum.HumanoidStateType.Physics)
        root.Anchored = false
    else
        hum:ChangeState(Enum.HumanoidStateType.GettingUp)
        root.Velocity = Vector3.new(0,0,0)
    end
end)

-- 速度調整
SpeedUp.MouseButton1Click:Connect(function()
    flySpeed += flyStep
end)
SpeedDown.MouseButton1Click:Connect(function()
    flySpeed = math.max(4, flySpeed - flyStep)
end)

-- 飛行処理
RunService.RenderStepped:Connect(function()
    if flying then
        local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
        local hum = char:WaitForChild("Humanoid")
        local root = char:WaitForChild("HumanoidRootPart")
        local moveDir = hum.MoveDirection
        local camLook = Camera.CFrame.LookVector

        if moveDir.Magnitude > 0 then
            local combined = (moveDir + Vector3.new(0, camLook.Y, 0)).Unit
            root.Velocity = combined * flySpeed
            -- 体を視点方向に向ける
            root.CFrame = CFrame.new(root.Position, root.Position + Vector3.new(camLook.X,0,camLook.Z))
        else
            root.Velocity = Vector3.new(0,0,0)
        end
    end
end)
