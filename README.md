if game.PlaceId ~= 16837656780 then return end

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local RS = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer

getgenv().Settings = {
    GunAura = false,
    ESPItems = false,
    ESPMobs = false,
    AutoBonds = false,
}

local function HRP()
    return LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
end

local function createHighlight(obj, name, color)
    if obj:FindFirstChild("ESP_HL") then return end
    local hl = Instance.new("Highlight")
    hl.Name = "ESP_HL"
    hl.Adornee = obj
    hl.FillColor = color
    hl.OutlineColor = Color3.new(1,1,1)
    hl.FillTransparency = 0.5
    hl.OutlineTransparency = 0.1
    hl.Parent = obj
    local billboard = Instance.new("BillboardGui", obj)
    billboard.Name = "ESP_HL"
    billboard.Size = UDim2.new(0,100,0,20)
    billboard.AlwaysOnTop = true
    local textLabel = Instance.new("TextLabel", billboard)
    textLabel.Size = UDim2.new(1,0,1,0)
    textLabel.Text = name
    textLabel.BackgroundTransparency = 1
    textLabel.TextColor3 = color
    textLabel.TextScaled = true
    textLabel.Font = Enum.Font.SourceSansBold
end

local function gunAura()
    local root = HRP()
    if not root then return end
    for _, mob in ipairs(Workspace:GetDescendants()) do
        if mob:IsA("Model") and mob:FindFirstChild("Humanoid") and mob:FindFirstChild("HumanoidRootPart") then
            if (mob.HumanoidRootPart.Position - root.Position).Magnitude < 60 then
                local shootEvent = ReplicatedStorage:FindFirstChild("Shoot")
                if shootEvent then
                    shootEvent:FireServer(mob)
                end
            end
        end
    end
end

local function espItems()
    for _, item in ipairs(Workspace:GetDescendants()) do
        if item:IsA("Tool") or item.Name:lower():find("item") then
            createHighlight(item, item.Name, Color3.fromRGB(0,255,0))
        end
    end
end

local function espMobs()
    for _, mob in ipairs(Workspace:GetDescendants()) do
        if mob:IsA("Model") and mob:FindFirstChild("Humanoid") and mob:FindFirstChild("HumanoidRootPart") then
            createHighlight(mob, mob.Name, Color3.fromRGB(255,0,0))
        end
    end
end

local function autoBonds()
    local root = HRP()
    if not root then return end
    for _, part in ipairs(Workspace:GetDescendants()) do
        if part:IsA("Part") and part.Name:lower():find("bond") then
            local dist = (part.Position - root.Position).Magnitude
            if dist < 80 then
                root.CFrame = part.CFrame + Vector3.new(0,2,0)
                task.wait(0.2)
            else
                for _, seat in ipairs(Workspace:GetDescendants()) do
                    if seat:IsA("Seat") then
                        seat.CFrame = part.CFrame + Vector3.new(0,1,0)
                        task.wait(0.2)
                        root.CFrame = seat.CFrame + Vector3.new(0,3,0)
                        break
                    end
                end
            end
        end
    end
end

RS.RenderStepped:Connect(function()
    pcall(function()
        if getgenv().Settings.GunAura then gunAura() end
        if getgenv().Settings.ESPItems then espItems() end
        if getgenv().Settings.ESPMobs then espMobs() end
        if getgenv().Settings.AutoBonds then autoBonds() end
    end)
end)

local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "DeadRailsHub"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 180, 0, 220)
frame.Position = UDim2.new(0, 20, 0, 100)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0

local title = Instance.new("TextLabel", frame)
title.Text = "Dead Rails Hub"
title.Size = UDim2.new(1, 0, 0, 30)
title.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 16

local function makeButton(label, y, key)
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(1, -20, 0, 30)
    btn.Position = UDim2.new(0, 10, 0, y)
    btn.Text = label .. ": OFF"
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.SourceSans
    btn.TextSize = 14
    btn.MouseButton1Click:Connect(function()
        getgenv().Settings[key] = not getgenv().Settings[key]
        btn.Text = label .. ": " .. (getgenv().Settings[key] and "ON" or "OFF")
    end)
end

makeButton("Gun Aura", 40, "GunAura")
makeButton("ESP Items", 80, "ESPItems")
makeButton("ESP Mobs", 120, "ESPMobs")
makeButton("Auto Bonds", 160, "AutoBonds")

