local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local espEnabled = true
local aimlockEnabled = false
local maxDistance = 150

local espText = {}

local function criarESP(plr)
    if plr == LocalPlayer then return end
    if not plr.Character or not plr.Character:FindFirstChild("HumanoidRootPart") then return end

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "ESP_Text"
    billboard.AlwaysOnTop = true
    billboard.Adornee = plr.Character.HumanoidRootPart
    billboard.Size = UDim2.new(0, 200, 0, 30)
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.Parent = plr.Character

    local label = Instance.new("TextLabel", billboard)
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.TextStrokeTransparency = 0.5
    label.Font = Enum.Font.GothamBold
    label.TextSize = 14
    label.TextColor3 = plr.TeamColor.Color
    label.Text = ""
    espText[plr] = label
end

local function removerESP(plr)
    espText[plr] = nil
    if plr.Character and plr.Character:FindFirstChild("ESP_Text") then
        plr.Character.ESP_Text:Destroy()
    end
end

-- Interface simples
local screen = Instance.new("ScreenGui", game.CoreGui)
screen.ResetOnSpawn = false

local function criarBotao(txt, yPos)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 150, 0, 35)
    btn.Position = UDim2.new(0, 20, 0, yPos)
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 16
    btn.Text = txt
    Instance.new("UICorner", btn)
    btn.Parent = screen
    return btn
end

local espBtn = criarBotao("ESP: ON", 20)
local aimBtn = criarBotao("Aimlock: OFF", 65)

espBtn.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    espBtn.Text = espEnabled and "ESP: ON" or "ESP: OFF"
    if not espEnabled then
        for plr in pairs(espText) do
            removerESP(plr)
        end
    end
end)

aimBtn.MouseButton1Click:Connect(function()
    aimlockEnabled = not aimlockEnabled
    aimBtn.Text = aimlockEnabled and "Aimlock: ON" or "Aimlock: OFF"
end)

RunService.RenderStepped:Connect(function()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            if espEnabled and not espText[plr] then
                criarESP(plr)
            end

            if espEnabled and espText[plr] then
                local dist = (Camera.CFrame.Position - plr.Character.HumanoidRootPart.Position).Magnitude
                local hp = plr.Character:FindFirstChild("Humanoid") and math.floor(plr.Character.Humanoid.Health) or "?"
                espText[plr].TextColor3 = plr.TeamColor.Color
                espText[plr].Text = string.format("%s | HP: %s | %.0f studs", plr.Name, hp, dist)
            elseif espText[plr] then
                removerESP(plr)
            end
        elseif espText[plr] then
            removerESP(plr)
        end
    end

    if aimlockEnabled then
        local closest, shortest = nil, maxDistance
        for _, plr in ipairs(Players:GetPlayers()) do
            if
                plr ~= LocalPlayer
                and plr.Team ~= LocalPlayer.Team
                and plr.Character
                and plr.Character:FindFirstChild("HumanoidRootPart")
            then
                local dist = (Camera.CFrame.Position - plr.Character.HumanoidRootPart.Position).Magnitude
                if dist < shortest then
                    shortest = dist
                    closest = plr
                end
            end
        end

        if closest and closest.Character and closest.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = closest.Character.HumanoidRootPart
            local camPos = Camera.CFrame.Position
            local dir = (Vector3.new(hrp.Position.X, camPos.Y, hrp.Position.Z) - camPos).Unit
            Camera.CFrame = CFrame.new(camPos, camPos + dir)
        end
    end
end)

Players.PlayerRemoving:Connect(removerESP)
