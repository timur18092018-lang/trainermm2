-- Fix: Removed jump shooting trigger so it only fires when you actually want it to
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer
local CurrentGun = nil
getgenv().ShootOffset = getgenv().ShootOffset or 1

-- Gun Detector Loop
task.spawn(function()
    while task.wait(0.5) do
        local equipped = LocalPlayer:FindFirstChild("EquippedGun")
        if equipped and equipped.Value ~= CurrentGun then
            CurrentGun = equipped.Value
        elseif (not equipped or equipped.Value == nil) and CurrentGun ~= nil then
            CurrentGun = nil
        end
    end
end)

local function GetClosestTarget()
    local character = LocalPlayer.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return nil end

    local root = character.HumanoidRootPart
    local closest = nil
    local closestDist = math.huge

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and (player.Team == nil or player.Team ~= LocalPlayer.Team) then
            local enemyChar = player.Character
            if enemyChar and enemyChar:FindFirstChild("HumanoidRootPart") then
                local dist = (root.Position - enemyChar.HumanoidRootPart.Position).Magnitude
                if dist < closestDist then
                    closestDist = dist
                    closest = enemyChar
                end
            end
        end
    end

    if not closest then
        local rigsFolder = Workspace:FindFirstChild("Rigs")
        if rigsFolder then
            for _, obj in ipairs(rigsFolder:GetDescendants()) do
                if obj:IsA("BasePart") and obj.Name == "HumanoidRootPart" then
                    local dist = (root.Position - obj.Position).Magnitude
                    if dist < closestDist then
                        closestDist = dist
                        closest = obj.Parent
                    end
                end
            end
        end
    end

    return closest
end

local function PredictPosition(target, offset)
    if not target or not target:FindFirstChild("HumanoidRootPart") then return Vector3.zero end
    local hrp = target.HumanoidRootPart
    local velocity = hrp.AssemblyLinearVelocity
    local ping = LocalPlayer:GetNetworkPing() or 0
    local pingAdjust = velocity * math.max(0, ping)
    return hrp.Position + velocity * (offset / 15) + pingAdjust
end

local function Shoot()
    if not CurrentGun then return end
    local target = GetClosestTarget()
    if not target then return end

    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end

    -- Equip gun
    local gunInBackpack = LocalPlayer.Backpack:FindFirstChild(CurrentGun)
    if gunInBackpack then gunInBackpack.Parent = char end

    local predicted = PredictPosition(target, getgenv().ShootOffset)
    local gun = char:FindFirstChild(CurrentGun)
    
    if gun and gun:FindFirstChild("GunServer") and gun.GunServer:FindFirstChild("ShootStart") then
        gun.GunServer.ShootStart:FireServer(1, predicted)
        task.wait(0.14)
    end

    -- Unequip gun
    if gun then gun.Parent = LocalPlayer.Backpack end
end

-- Кнопка для стрельбы на экране (без прыжка)
if CoreGui:FindFirstChild("ApexShootGui") then
    CoreGui.ApexShootGui:Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ApexShootGui"
ScreenGui.Parent = CoreGui
ScreenGui.ResetOnSpawn = false

local ShootBtn = Instance.new("ImageButton")
ShootBtn.Size = UDim2.new(0, 65, 0, 65)
ShootBtn.Position = UDim2.new(1, -145, 1, -140)
ShootBtn.Image = "rbxassetid://16688482749"
ShootBtn.BackgroundTransparency = 1
ShootBtn.Parent = ScreenGui

ShootBtn.MouseButton1Click:Connect(Shoot)
