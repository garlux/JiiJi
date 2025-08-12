-- // Flash Speedster Combo | Stylish + Color List
-- R6 + R15 Compatible | Mobile Friendly | Executor Ready

pcall(function() game.CoreGui:FindFirstChild("FlashGUI"):Destroy() end)

local Players = game:GetService("Players")
local Debris = game:GetService("Debris")
local TweenService = game:GetService("TweenService")
local player = Players.LocalPlayer
local cam = workspace.CurrentCamera
local char = player.Character or player.CharacterAdded:Wait()
local humanoid = char:WaitForChild("Humanoid")
local hrp = char:WaitForChild("HumanoidRootPart")

-- ====== CONFIG ======
local normalSpeed = 16
local currentSpeed = 70
local maxSpeed = 3000
local accelerationRate = 5
local isFlash = false

-- ====== COLORS ======
local colorList = {
    Color3.fromRGB(0, 162, 255),   -- Blue
    Color3.fromRGB(255, 0, 0),     -- Red
    Color3.fromRGB(0, 255, 0),     -- Green
    Color3.fromRGB(192, 192, 192), -- Gray
    Color3.fromRGB(139, 69, 19),   -- Brown
    Color3.fromRGB(255, 105, 180), -- Pink
    Color3.fromRGB(255, 255, 0), -- Yellow
	Color3.fromRGB(255, 255, 255) -- White
}
local currentColorIndex = 1
local flashColor = colorList[currentColorIndex]
local bodyColor = flashColor

-- ====== SOUNDS ======
local rumble = Instance.new("Sound", hrp)
rumble.SoundId = "rbxassetid://9112795583"
rumble.Looped = true
rumble.Volume = 10

local zap = Instance.new("Sound", hrp)
zap.SoundId = "rbxassetid://9116277062"
zap.Volume = 1

-- ====== TRAILS ======
local electricTrails = {}

local function lightningTrail()
    local a0 = Instance.new("Attachment", hrp)
    local a1 = Instance.new("Attachment", hrp)
    a0.Position = Vector3.new(0, 2, 0)
    a1.Position = Vector3.new(0, -2, 0)

    local trail = Instance.new("Trail", hrp)
    trail.Attachment0 = a0
    trail.Attachment1 = a1
    trail.Color = ColorSequence.new(flashColor)
    trail.Lifetime = 0.3
    trail.WidthScale = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 1.2),
        NumberSequenceKeypoint.new(1, 0)
    })
    trail.Transparency = NumberSequence.new(0.2)
    trail.LightEmission = 1
	trail.Texture = "rbxassetid://5699911443"

    Debris:AddItem(trail, 1)
    Debris:AddItem(a0, 1)
    Debris:AddItem(a1, 1)
end

local function createElectricTrails()
    local parts = {
        {char:FindFirstChild("Head"), Vector3.new(0, 0.4, 0)},
        {char:FindFirstChild("Left Arm") or char:FindFirstChild("LeftUpperArm"), Vector3.new(0, -0.5, 0)},
        {char:FindFirstChild("Right Arm") or char:FindFirstChild("RightUpperArm"), Vector3.new(0, -0.5, 0)},
        {char:FindFirstChild("Left Leg") or char:FindFirstChild("LeftUpperLeg"), Vector3.new(0, -0.5, 0)},
        {char:FindFirstChild("Right Leg") or char:FindFirstChild("RightUpperLeg"), Vector3.new(0, -0.5, 0)},
        {hrp, Vector3.new(0.5, 1, 0)}
    }
    for _, info in ipairs(parts) do
        if info[1] then
            local att0 = Instance.new("Attachment", info[1])
            local att1 = Instance.new("Attachment", info[1])
            att0.Position = info[2]
            att1.Position = info[2] + Vector3.new(0, -0.3, 0)
            local trail = Instance.new("Trail", info[1])
            trail.Attachment0 = att0
            trail.Attachment1 = att1
            trail.Color = ColorSequence.new(bodyColor)
            trail.Lifetime = 0.2
            trail.Transparency = NumberSequence.new(0.15)
            trail.WidthScale = NumberSequence.new(0.35)
            trail.LightEmission = 1
            table.insert(electricTrails, trail)
            table.insert(electricTrails, att0)
            table.insert(electricTrails, att1)
	        trail.Texture = "rbxassetid://7151778311"
        end
    end
end

local function destroyElectricTrails()
    for _, obj in pairs(electricTrails) do
        if obj and obj.Parent then obj:Destroy() end
    end
    electricTrails = {}
end

local function startShaking()
    while isFlash do
        if humanoid.MoveDirection.Magnitude < 0.1 then
            local offset = Vector3.new(
                math.random(-8, 8) * 0.1,
                math.random(-8, 8) * 0.1,
                math.random(-8, 8) * 0.1
            )
            pcall(function()
                hrp.CFrame = hrp.CFrame * CFrame.new(offset)
            end)
        end
        task.wait(0.03)
    end
end

local function toggleFlash()
    isFlash = not isFlash
    if isFlash then
        currentSpeed = 70
        rumble:Play()
        TweenService:Create(cam, TweenInfo.new(0.3), {FieldOfView = 100}):Play()
        createElectricTrails()
        task.spawn(function()
            while isFlash do
                local moving = humanoid.MoveDirection.Magnitude > 0.1
                if moving then
                    currentSpeed = math.min(currentSpeed + accelerationRate, maxSpeed)
                    lightningTrail()
                    if not zap.IsPlaying and math.random() < 0.15 then zap:Play() end
                else
                    currentSpeed = normalSpeed
                end
                humanoid.WalkSpeed = currentSpeed
                task.wait(0.08)
            end
        end)
        task.spawn(startShaking)
    else
        humanoid.WalkSpeed = normalSpeed
        rumble:Stop()
        currentSpeed = normalSpeed
        TweenService:Create(cam, TweenInfo.new(0.3), {FieldOfView = 70}):Play()
        destroyElectricTrails()
    end
end

-- ====== GUI ======
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "FlashGUI"
gui.ResetOnSpawn = false

local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0, 10)
local uiStroke = Instance.new("UIStroke")
uiStroke.Thickness = 2
uiStroke.Color = Color3.fromRGB(255,255,255)

-- Toggle Button
local btnToggle = Instance.new("TextButton", gui)
btnToggle.Size = UDim2.new(0, 140, 0, 50)
btnToggle.Position = UDim2.new(0, 20, 1, -110)
btnToggle.BackgroundColor3 = flashColor
btnToggle.Text = "⚡ Toggle"
btnToggle.TextSize = 18
btnToggle.Font = Enum.Font.GothamBold
btnToggle.TextColor3 = Color3.new(1, 1, 1)
btnToggle.Draggable = false
btnToggle.Active = true
uiCorner:Clone().Parent = btnToggle
uiStroke:Clone().Parent = btnToggle
btnToggle.MouseButton1Click:Connect(toggleFlash)

-- Color Button
local btnColor = Instance.new("TextButton", gui)
btnColor.Size = UDim2.new(0, 140, 0, 50)
btnColor.Position = UDim2.new(0, 20, 1, -50)
btnColor.BackgroundColor3 = Color3.fromRGB(200, 200, 200)
btnColor.Text = "🎨 Color"
btnColor.TextSize = 18
btnColor.Font = Enum.Font.GothamBold
btnColor.TextColor3 = Color3.new(1, 1, 1)
btnColor.Draggable = false
btnColor.Active = true
uiCorner:Clone().Parent = btnColor
uiStroke:Clone().Parent = btnColor
btnColor.MouseButton1Click:Connect(function()
    currentColorIndex = currentColorIndex + 1
    if currentColorIndex > #colorList then
        currentColorIndex = 1
    end
    flashColor = colorList[currentColorIndex]
    bodyColor = flashColor
    btnToggle.BackgroundColor3 = flashColor
end)
