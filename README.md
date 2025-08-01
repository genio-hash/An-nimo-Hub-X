local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

local ScreenGui = Instance.new("ScreenGui", game:GetService("CoreGui"))
ScreenGui.Name = "AnonimosHub"

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 350, 0, 450)
Frame.Position = UDim2.new(0.5, -175, 0.5, -225)
Frame.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
Frame.Visible = false
Frame.ClipsDescendants = true
Frame.BorderSizePixel = 0

local Title = Instance.new("TextLabel", Frame)
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundTransparency = 1
Title.Text = "Anônimos Hub - Blox Fruits"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 22
Title.TextColor3 = Color3.new(1, 1, 1)

local CloseButton = Instance.new("TextButton", Frame)
CloseButton.Size = UDim2.new(0, 60, 0, 25)
CloseButton.Position = UDim2.new(1, -70, 0, 8)
CloseButton.Text = "Sair"
CloseButton.Font = Enum.Font.GothamBold
CloseButton.TextSize = 18
CloseButton.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
CloseButton.TextColor3 = Color3.new(1, 1, 1)

CloseButton.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
    getgenv().AnonimosHubRunning = false
end)

-- Toggle GUI com F4
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.F4 then
        Frame.Visible = not Frame.Visible
    end
end)

local function CreateToggle(text, default, callback)
    local btn = Instance.new("TextButton", Frame)
    btn.Size = UDim2.new(1, -20, 0, 30)
    btn.Position = UDim2.new(0, 10, 0, 50 + (#Frame:GetChildren() - 5) * 35)
    btn.BackgroundColor3 = default and Color3.fromRGB(50, 150, 50) or Color3.fromRGB(150, 50, 50)
    btn.Text = text .. (default and " [ON]" or " [OFF]")
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 18
    btn.TextColor3 = Color3.new(1, 1, 1)

    local toggled = default
    btn.MouseButton1Click:Connect(function()
        toggled = not toggled
        btn.BackgroundColor3 = toggled and Color3.fromRGB(50, 150, 50) or Color3.fromRGB(150, 50, 50)
        btn.Text = text .. (toggled and " [ON]" or " [OFF]")
        callback(toggled)
    end)
    return btn
end

getgenv().AnonimosHubRunning = true
local AutoFarm = false
local ESPEnabled = false
local FastAttack = false

local function AutoFarmFunc()
    spawn(function()
        while getgenv().AnonimosHubRunning and AutoFarm do
            local Character = LocalPlayer.Character
            if not Character or not Character:FindFirstChild("HumanoidRootPart") then wait(1) continue end

            local enemies = {}
            for _, npc in pairs(workspace.Enemies:GetChildren()) do
                if npc:FindFirstChild("Humanoid") and npc.Humanoid.Health > 0 then
                    table.insert(enemies, npc)
                end
            end

            if #enemies > 0 then
                for _, enemy in pairs(enemies) do
                    if enemy and enemy:FindFirstChild("HumanoidRootPart") then
                        Character.HumanoidRootPart.CFrame = enemy.HumanoidRootPart.CFrame * CFrame.new(0, 3, 3)
                        wait(0.3)
                        local CombatEvent = ReplicatedStorage:FindFirstChild("Combat") and ReplicatedStorage.Combat:FindFirstChild("Attack")
                        if CombatEvent then
                            CombatEvent:FireServer()
                        end
                        wait(0.5)
                        if not AutoFarm then break end
                    end
                end
            else
                wait(1)
            end
        end
    end)
end

local function FastAttackFunc(enabled)
    FastAttack = enabled
    if enabled then
        spawn(function()
            while getgenv().AnonimosHubRunning and FastAttack do
                local Character = LocalPlayer.Character
                local Humanoid = Character and Character:FindFirstChildOfClass("Humanoid")
                if Humanoid and Humanoid.Health > 0 then
                    local CombatEvent = ReplicatedStorage:FindFirstChild("Combat") and ReplicatedStorage.Combat:FindFirstChild("Attack")
                    if CombatEvent then
                        CombatEvent:FireServer()
                    end
                end
                wait(0.1)
            end
        end)
    end
end

local function ESPToggle(enabled)
    ESPEnabled = enabled
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local esp = player.Character:FindFirstChild("ESPBox")
                if esp then esp:Destroy() end
                if enabled then
                    local box = Instance.new("BoxHandleAdornment")
                    box.Name = "ESPBox"
                    box.Adornee = player.Character.HumanoidRootPart
                    box.Size = Vector3.new(4, 6, 4)
                    box.Color3 = Color3.new(1, 0, 0)
                    box.Transparency = 0.5
                    box.AlwaysOnTop = true
                    box.Parent = player.Character.HumanoidRootPart
                end
            end
        end
    end
end

CreateToggle("Auto Farm", false, function(val)
    AutoFarm = val
    if val then AutoFarmFunc() end
end)

CreateToggle("Fast Attack", false, FastAttackFunc)
CreateToggle("ESP Players", false, ESPToggle)

print("Anônimos Hub iniciado! Use F4 para abrir/fechar a interface.")
