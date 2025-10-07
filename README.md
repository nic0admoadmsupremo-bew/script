-- TeleportClick_Local.lua (LocalScript)
-- Coloque em StarterPlayer > StarterPlayerScripts

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local mouse = player:GetMouse()

-- CONFIG
local TELEPORT_HEIGHT_OFFSET = 4
local COOLDOWN = 0.25
local enabled = true
local lastTeleport = 0

-- UI
local gui = Instance.new("ScreenGui")
gui.Name = "TeleportClickGui"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0,220,0,80)
frame.Position = UDim2.new(0,10,0,10)
frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
frame.BorderSizePixel = 0

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,-16,0,24)
title.Position = UDim2.new(0,8,0,6)
title.BackgroundTransparency = 1
title.Text = "Teleport Click (Local)"
title.TextColor3 = Color3.fromRGB(230,230,230)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 18
title.TextXAlignment = Enum.TextXAlignment.Left

local status = Instance.new("TextLabel", frame)
status.Size = UDim2.new(0.5,-12,0,20)
status.Position = UDim2.new(0,8,0,34)
status.BackgroundTransparency = 1
status.Text = "Status: ON"
status.TextColor3 = Color3.fromRGB(200,200,200)
status.Font = Enum.Font.SourceSans
status.TextSize = 16

local toggle = Instance.new("TextButton", frame)
toggle.Size = UDim2.new(0.45,-12,0,28)
toggle.Position = UDim2.new(0.5,8,0,32)
toggle.BackgroundColor3 = Color3.fromRGB(35,125,70)
toggle.Text = "Desligar"
toggle.Font = Enum.Font.SourceSansBold
toggle.TextSize = 16
toggle.TextColor3 = Color3.fromRGB(230,230,230)

local hint = Instance.new("TextLabel", frame)
hint.Size = UDim2.new(1,-16,0,18)
hint.Position = UDim2.new(0,8,0,62)
hint.BackgroundTransparency = 1
hint.Text = "Clique no mundo para teleportar (quando ON)."
hint.TextColor3 = Color3.fromRGB(160,160,160)
hint.Font = Enum.Font.SourceSans
hint.TextSize = 12

local function updateUI()
    if enabled then
        status.Text = "Status: ON"
        toggle.Text = "Desligar"
        toggle.BackgroundColor3 = Color3.fromRGB(35,125,70)
    else
        status.Text = "Status: OFF"
        toggle.Text = "Ligar"
        toggle.BackgroundColor3 = Color3.fromRGB(125,35,35)
    end
end

toggle.MouseButton1Click:Connect(function()
    enabled = not enabled
    updateUI()
end)

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.T then
        enabled = not enabled
        updateUI()
    end
end)

local function teleportTo(pos)
    if not pos then return end
    local char = player.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not hrp then return end

    local now = tick()
    if now - lastTeleport < COOLDOWN then return end
    lastTeleport = now

    local target = CFrame.new(pos + Vector3.new(0, TELEPORT_HEIGHT_OFFSET, 0))
    -- Try safe teleport
    pcall(function()
        if humanoid then humanoid.PlatformStand = true end
        hrp.CFrame = target
        pcall(function() hrp.AssemblyLinearVelocity = Vector3.new(0,0,0) end)
        wait(0.03)
        if humanoid then humanoid.PlatformStand = false end
    end)
end

mouse.Button1Down:Connect(function()
    if not enabled then return end
    local hit = mouse.Hit
    if not hit then return end
    teleportTo(hit.p)
end)

updateUI()
