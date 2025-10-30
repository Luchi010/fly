local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- GUI作成（死んでも残る）
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "PopFlyGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- 状態
local flying = false
local flySpeed = 16 -- 初期速度
local flyStep = 4

-- ボタン作成関数（ポップ＆チート感）
local function createButton(name, text, pos, bg)
    local btn = Instance.new("TextButton")
    btn.Name = name
    btn.Text = text
    btn.Size = UDim2.new(0, 80, 0, 40)
    btn.Position = pos
    btn.TextScaled = true
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.Font = Enum.Font.GothamBold
    if bg then
        btn.BackgroundColor3 = Color3.fromRGB(255,0,128)
        btn.BackgroundTransparency = 0
        btn.AutoButtonColor = true
        btn.BorderSizePixel = 2
    else
        btn.BackgroundTransparency = 1
    end
    btn.Parent = ScreenGui
    return btn
end

-- ボタン作成
local FlyButton = createButton("FlyButton", "Fly", UDim2.new(0.8,0,0.05,0), true)
local SpeedUp = createButton("SpeedUp", "+", UDim2.new(0.7,0,0.05,0), false)
local SpeedDown = createButton("SpeedDown", "-", UDim2.new(0.7,0,0.12,0), false)

-- 移動速度表示ラベル
local SpeedLabel = Instance.new("TextLabel")
SpeedLabel.Size = UDim2.new(0,120,0,30)
SpeedLabel.Position = UDim2.new(0.05,0,0.05,0)
SpeedLabel.TextColor3 = Color3.fromRGB(255,255,0)
SpeedLabel.Font = Enum.Font.GothamBold
SpeedLabel.TextScaled = true
SpeedLabel.BackgroundTransparency = 0.5
SpeedLabel.BackgroundColor3 = Color3.fromRGB(50,0,100)
SpeedLabel.Text = "Speed: "..flySpeed
SpeedLabel.Parent = ScreenGui

-- キャラ取得関数
local function getCharacter()
    local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    local hum = char:WaitForChild("Humanoid")
    local root = char:WaitForChild("HumanoidRootPart")
    return char, hum, root
end

-- Flyボタン押下で飛行切替
FlyButton.MouseButton1Click:Connect(function()
    local _, hum = getCharacter()
    flying = not flying
    if flying then
        hum:ChangeState(Enum.HumanoidStateType.Physics)
    else
        hum:ChangeState(Enum.HumanoidStateType.GettingUp)
        local _, _, root = getCharacter()
        root.Velocity = Vector3.new(0,0,0)
    end
end)

-- 速度調整
SpeedUp.MouseButton1Click:Connect(function()
    flySpeed += flyStep
    SpeedLabel.Text = "Speed: "..flySpeed
end)
SpeedDown.MouseButton1Click:Connect(function()
    flySpeed = math.max(4, flySpeed - flyStep)
    SpeedLabel.Text = "Speed: "..flySpeed
end)

-- 飛行処理
RunService.RenderStepped:Connect(function(delta)
    if flying then
        local char, hum, root = getCharacter()
        local moveDir = hum.MoveDirection
        local camLook = Camera.CFrame.LookVector

        if moveDir.Magnitude > 0 then
            local combined = (moveDir + Vector3.new(0, camLook.Y, 0)).Unit
            root.Velocity = combined * flySpeed
            -- 体を視点方向に向ける
            root.CFrame = CFrame.new(root.Position, root.Position + combined)
        else
            root.Velocity = Vector3.new(0,0,0)
        end
    end
end)
