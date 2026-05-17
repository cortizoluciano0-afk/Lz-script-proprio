getgenv().SencerSettings = {
    KillAura = false,
    ESP = false,
    FarmLevel = false,
    Range = 50
}

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

-- INTERFACE
local ScreenGui = Instance.new("ScreenGui")
local Frame = Instance.new("Frame")
local Title = Instance.new("TextLabel")

ScreenGui.Parent = game.CoreGui
ScreenGui.Name = "SencerHubBF"

Frame.Parent = ScreenGui
Frame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Frame.BorderColor3 = Color3.fromRGB(0, 170, 255)
Frame.BorderSizePixel = 2
Frame.Position = UDim2.new(0.05, 0, 0.3, 0)
Frame.Size = UDim2.new(0, 220, 0, 200)
Frame.Active = true
Frame.Draggable = true

Title.Parent = Frame
Title.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
Title.Size = UDim2.new(1, 0, 0, 30)
Title.Text = "SENCER BF HUB"
Title.TextColor3 = Color3.fromRGB(0, 0, 0)
Title.TextScaled = true
Title.Font = Enum.Font.GothamBold

local function CreateBtn(name, pos, callback)
    local btn = Instance.new("TextButton")
    btn.Parent = Frame
    btn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    btn.Position = UDim2.new(0.05, 0, pos, 0)
    btn.Size = UDim2.new(0.9, 0, 0, 30)
    btn.Text = name .. ": OFF"
    btn.TextColor3 = Color3.fromRGB(255, 80, 80)
    btn.TextScaled = true
    btn.Font = Enum.Font.Gotham
    btn.MouseButton1Click:Connect(callback)
    return btn
end

local AuraBtn = CreateBtn("Kill Aura", 0.2, function()
    getgenv().SencerSettings.KillAura = not getgenv().SencerSettings.KillAura
    AuraBtn.Text = "Kill Aura: " .. (getgenv().SencerSettings.KillAura and "ON" or "OFF")
    AuraBtn.TextColor3 = getgenv().SencerSettings.KillAura and Color3.fromRGB(80, 255, 80) or Color3.fromRGB(255, 80, 80)
end)

local ESPBtn = CreateBtn("ESP Player", 0.38, function()
    getgenv().SencerSettings.ESP = not getgenv().SencerSettings.ESP
    ESPBtn.Text = "ESP Player: " .. (getgenv().SencerSettings.ESP and "ON" or "OFF")
    ESPBtn.TextColor3 = getgenv().SencerSettings.ESP and Color3.fromRGB(80, 255, 80) or Color3.fromRGB(255, 80, 80)
end)

local FarmBtn = CreateBtn("Auto Farm LVL", 0.56, function()
    getgenv().SencerSettings.FarmLevel = not getgenv().SencerSettings.FarmLevel
    FarmBtn.Text = "Auto Farm LVL: " .. (getgenv().SencerSettings.FarmLevel and "ON" or "OFF")
    FarmBtn.TextColor3 = getgenv().SencerSettings.FarmLevel and Color3.fromRGB(80, 255, 80) or Color3.fromRGB(255, 80, 80)
end)

local CloseBtn = CreateBtn("FECHAR", 0.74, function()
    getgenv().SencerSettings.KillAura = false
    getgenv().SencerSettings.ESP = false
    getgenv().SencerSettings.FarmLevel = false
    ScreenGui:Destroy()
end)
CloseBtn.BackgroundColor3 = Color3.fromRGB(80, 0, 0)

-- KILL AURA
task.spawn(function()
    while task.wait() do
        if getgenv().SencerSettings.KillAura then
            pcall(function()
                for i,v in pairs(workspace.Enemies:GetChildren()) do
                    if v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v.Humanoid.Health > 0 then
                        if (v.HumanoidRootPart.Position - player.Character.HumanoidRootPart.Position).Magnitude <= getgenv().SencerSettings.Range then
                            v.Humanoid.Health = 0
                        end
                    end
                end
            end)
        end
    end
end)

-- ESP ESQUELETO + BOX
local function CreateESP(plr)
    local Box = Drawing.new("Square")
    Box.Visible = false
    Box.Color = Color3.fromRGB(0, 170, 255)
    Box.Thickness = 2
    Box.Transparency = 1
    Box.Filled = false

    local Line = Drawing.new("Line")
    Line.Visible = false
    Line.Color = Color3.fromRGB(255, 255, 255)
    Line.Thickness = 1
    Line.Transparency = 1

    local Name = Drawing.new("Text")
    Name.Visible = false
    Name.Color = Color3.fromRGB(255, 255, 255)
    Name.Size = 16
    Name.Center = true
    Name.Outline = true

    RunService.RenderStepped:Connect(function()
        if getgenv().SencerSettings.ESP and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and plr.Character:FindFirstChild("Humanoid") and plr.Character.Humanoid.Health > 0 then
            local pos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(plr.Character.HumanoidRootPart.Position)
            if onScreen then
                local size = 2000 / pos.Z
                Box.Size = Vector2.new(size, size * 1.5)
                Box.Position = Vector2.new(pos.X - size/2, pos.Y - size * 0.75)
                Box.Visible = true

                Line.From = Vector2.new(workspace.CurrentCamera.ViewportSize.X/2, workspace.CurrentCamera.ViewportSize.Y)
                Line.To = Vector2.new(pos.X, pos.Y)
                Line.Visible = true

                Name.Position = Vector2.new(pos.X, pos.Y - size * 0.75 - 15)
                Name.Text = plr.Name .. " [" .. math.floor((player.Character.HumanoidRootPart.Position - plr.Character.HumanoidRootPart.Position).Magnitude) .. "]"
                Name.Visible = true
            else
                Box.Visible = false
                Line.Visible = false
                Name.Visible = false
            end
        else
            Box.Visible = false
            Line.Visible = false
            Name.Visible = false
        end
    end)
end

for _,plr in pairs(Players:GetPlayers()) do
    if plr ~= player then
        CreateESP(plr)
    end
end

Players.PlayerAdded:Connect(function(plr)
    CreateESP(plr)
end)

-- AUTO FARM LVL BÁSICO
task.spawn(function()
    while task.wait(0.5) do
        if getgenv().SencerSettings.FarmLevel then
            pcall(function()
                local quest = workspace:FindFirstChild("QuestGivers")
                if quest then
                    player.Character.HumanoidRootPart.CFrame = quest.CFrame * CFrame.new(0,3,0)
                end
            end)
        end
    end
end)
