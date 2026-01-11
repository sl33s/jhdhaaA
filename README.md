--[[
    jhdh aimbot - LOCA VERSION
    Made for: XZS
    Loader: GitHub Raw (Kuak.loca)
]]

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local SETTINGS = {
    FOV_RADIUS = 80,          -- فوف صغير (سنايبر)
    TARGET_PART = "Head",     -- الرأس
    TEAM_CHECK = false,       -- عطّل في المابات الخاصة
    MAX_DISTANCE = 3000,      -- مدى بعيد
    ENABLED = true,           -- تشغيل/إيقاف
}

-- حماية بسيطة من التكرار
if getgenv and getgenv().JHDH_AIMBOT_LOADED then
    return
end

if getgenv then
    getgenv().JHDH_AIMBOT_LOADED = true
end

-- GUI بسيط للتحكم
local screen = Instance.new("ScreenGui")
screen.Name = "jhdh_Trainer"
screen.ResetOnSpawn = false
screen.Parent = LocalPlayer:WaitForChild("PlayerGui")

local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.fromOffset(160, 40)
toggleBtn.Position = UDim2.new(0, 20, 0, 20)
toggleBtn.Text = "jhdh aimbot: ON"
toggleBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
toggleBtn.TextColor3 = Color3.fromRGB(0, 255, 140)
toggleBtn.Font = Enum.Font.SourceSansBold
toggleBtn.TextSize = 18
toggleBtn.Parent = screen

toggleBtn.MouseButton1Click:Connect(function()
    SETTINGS.ENABLED = not SETTINGS.ENABLED
    toggleBtn.Text = SETTINGS.ENABLED and "jhdh aimbot: ON" or "jhdh aimbot: OFF"
    toggleBtn.TextColor3 = SETTINGS.ENABLED and Color3.fromRGB(0,255,140) or Color3.fromRGB(255,80,80)
end)

local aimingHold = false

UIS.InputBegan:Connect(function(inp)
    if inp.UserInputType == Enum.UserInputType.MouseButton2 then
        aimingHold = true
    end
end)

UIS.InputEnded:Connect(function(inp)
    if inp.UserInputType == Enum.UserInputType.MouseButton2 then
        aimingHold = false
    end
end)

local function isAlive(plr)
    local char = plr.Character
    if not char then return false end
    local hum = char:FindFirstChildOfClass("Humanoid")
    return hum and hum.Health > 0
end

local function teamCheck(plr)
    if not SETTINGS.TEAM_CHECK then return true end
    if plr.Team == nil or LocalPlayer.Team == nil then return true end
    return plr.Team ~= LocalPlayer.Team
end

local function screenPos(worldPos)
    local vec, onScreen = Camera:WorldToViewportPoint(worldPos)
    return Vector2.new(vec.X, vec.Y), onScreen, vec.Z
end

local function getTargetPart(model, partName)
    if not model then return nil end
    return model:FindFirstChild(partName) or model:FindFirstChild("Head")
end

local function getBestTarget(mousePos)
    local bestPart, bestDist = nil, math.huge
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and isAlive(plr) and teamCheck(plr) then
            local part = getTargetPart(plr.Character, SETTINGS.TARGET_PART)
            if part then
                local sp, onScreen, depth = screenPos(part.Position)
                if onScreen and depth > 0 and depth < SETTINGS.MAX_DISTANCE then
                    local d = (sp - mousePos).Magnitude
                    if d <= SETTINGS.FOV_RADIUS and d < bestDist then
                        bestDist = d
                        bestPart = part
                    end
                end
            end
        end
    end
    return bestPart
end

RunService.RenderStepped:Connect(function()
    if not SETTINGS.ENABLED then return end
    if aimingHold then
        local mp = UIS:GetMouseLocation()
        local targetPart = getBestTarget(mp)
        if targetPart then
            Camera.CFrame = CFrame.new(
                Camera.CFrame.Position,
                targetPart.Position + Vector3.new(0, 0.05, 0)
            )
        end
    end
end)
