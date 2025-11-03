#-- SPEED CONTROL GUI | Type SPS | Toggle with L
-- Instant Speed + Instant Stop | Mobile + PC

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local MAX_SPEED = 200
local root, humanoid = nil, nil
local guiVisible = false

-- === CREATE GUI ===
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "SpeedGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 220, 0, 140)
mainFrame.Position = UDim2.new(0.5, -110, 0.5, -70)
mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
mainFrame.BorderSizePixel = 0
mainFrame.BackgroundTransparency = 0.1
mainFrame.Visible = false
mainFrame.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 14)
corner.Parent = mainFrame

-- Title
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 35)
title.BackgroundTransparency = 1
title.Text = "SPEED CONTROL"
title.TextColor3 = Color3.fromRGB(0, 255, 150)
title.Font = Enum.Font.GothamBold
title.TextSize = 16
title.Parent = mainFrame

-- Speed Input Box
local inputBox = Instance.new("TextBox")
inputBox.Size = UDim2.new(0, 180, 0, 40)
inputBox.Position = UDim2.new(0.5, -90, 0, 45)
inputBox.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
inputBox.PlaceholderText = "Enter SPS (e.g. 500)"
inputBox.Text = "200"
inputBox.TextColor3 = Color3.fromRGB(255, 255, 255)
inputBox.Font = Enum.Font.Gotham
inputBox.TextSize = 18
inputBox.ClearTextOnFocus = false
inputBox.Parent = mainFrame

local inputCorner = Instance.new("UICorner")
inputCorner.CornerRadius = UDim.new(0, 8)
inputCorner.Parent = inputBox

-- Apply Button
local applyBtn = Instance.new("TextButton")
applyBtn.Size = UDim2.new(0, 80, 0, 35)
applyBtn.Position = UDim2.new(0.5, -40, 0, 95)
applyBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
applyBtn.Text = "APPLY"
applyBtn.TextColor3 = Color3.new(1,1,1)
applyBtn.Font = Enum.Font.GothamBold
applyBtn.TextSize = 16
applyBtn.Parent = mainFrame

local applyCorner = Instance.new("UICorner")
applyCorner.CornerRadius = UDim.new(0, 8)
applyCorner.Parent = applyBtn

-- Hover effect
applyBtn.MouseEnter:Connect(function()
    TweenService:Create(applyBtn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(0, 200, 255)}):Play()
end)
applyBtn.MouseLeave:Connect(function()
    TweenService:Create(applyBtn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(0, 170, 255)}):Play()
end)

-- === APPLY SPEED ===
applyBtn.MouseButton1Click:Connect(function()
    local input = inputBox.Text:gsub("%D", "") -- Remove non-digits
    local speed = tonumber(input)
    if speed and speed >= 16 and speed <= 10000 then
        MAX_SPEED = speed
        inputBox.Text = tostring(MAX_SPEED)
        inputBox:ReleaseFocus()
        print("Speed set to: " .. MAX_SPEED)
    else
        inputBox.Text = tostring(MAX_SPEED)
        warn("Invalid speed! Must be 16–10000")
    end
end)

-- === TOGGLE GUI WITH L KEY ===
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.L then
        guiVisible = not guiVisible
        mainFrame.Visible = guiVisible
        
        if guiVisible then
            inputBox.Text = tostring(MAX_SPEED)
            TweenService:Create(mainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Back), {Position = UDim2.new(0.5, -110, 0.5, -70)}):Play()
        else
            TweenService:Create(mainFrame, TweenInfo.new(0.2), {Position = UDim2.new(0.5, -110, -1, 0)}):Play()
        end
    end

    -- === P / K / J KEYBINDS (Still work!) ===
    if input.KeyCode == Enum.KeyCode.P then
        MAX_SPEED = MAX_SPEED + 250
        if guiVisible then inputBox.Text = tostring(MAX_SPEED) end
        print("Speed: " .. MAX_SPEED)
    elseif input.KeyCode == Enum.KeyCode.K then
        MAX_SPEED = math.max(50, MAX_SPEED - 250)
        if guiVisible then inputBox.Text = tostring(MAX_SPEED) end
        print("Speed: " .. MAX_SPEED)
    elseif input.KeyCode == Enum.KeyCode.J then
        MAX_SPEED = 200
        if guiVisible then inputBox.Text = "200" end
        print("Speed Reset: 200")
    end
end)

-- === DRAG GUI (Mobile + PC) ===
local dragging = false
local dragStart, startPos

mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
    end
end)

mainFrame.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

mainFrame.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

-- === INSTANT SPEED LOGIC ===
local isMoving = false
local moveDir = Vector3.zero

local function updateMove()
    if not humanoid then return end
    local dir = humanoid.MoveDirection
    if dir.Magnitude > 0 then
        isMoving = true
        moveDir = dir.Unit
    else
        isMoving = false
    end
end

local function applySpeed()
    if not root or not humanoid then return end
    updateMove()

    if isMoving then
        root.Velocity = Vector3.new(moveDir.X * MAX_SPEED, root.Velocity.Y, moveDir.Z * MAX_SPEED)
    else
        root.Velocity = Vector3.new(0, root.Velocity.Y, 0)  -- Instant stop
    end
end

-- === ANTI-RESET WalkSpeed ===
task.spawn(function()
    while task.wait(0.05) do
        if humanoid and humanoid.WalkSpeed ~= MAX_SPEED then
            humanoid.WalkSpeed = MAX_SPEED
        end
    end
end)

-- === CHARACTER SETUP ===
local function setupCharacter(char)
    root = char:WaitForChild("HumanoidRootPart")
    humanoid = char:WaitForChild("Humanoid")

    humanoid:GetPropertyChangedSignal("WalkSpeed"):Connect(function()
        if humanoid.WalkSpeed ~= MAX_SPEED then
            humanoid.WalkSpeed = MAX_SPEED
        end
    end)

    isMoving = false
    moveDir = Vector3.zero

    if guiVisible then
        inputBox.Text = tostring(MAX_SPEED)
    end
end

player.CharacterAdded:Connect(setupCharacter)
if player.Character then setupCharacter(player.Character) end

-- === MAIN LOOP ===
RunService.Heartbeat:Connect(applySpeed)
