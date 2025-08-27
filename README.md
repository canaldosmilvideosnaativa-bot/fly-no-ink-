-- LocalScript dentro do StarterPlayerScripts
local player = game.Players.LocalPlayer
local uis = game:GetService("UserInputService")
local runService = game:GetService("RunService")

local flying = false
local speed = 50

-- GUI
local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0,200,0,120)
frame.Position = UDim2.new(0.4,0,0.4,0)
frame.BackgroundColor3 = Color3.fromRGB(50,50,50)
frame.Active = true
frame.Draggable = true
frame.Parent = screenGui

local flyButton = Instance.new("TextButton", frame)
flyButton.Size = UDim2.new(0,180,0,50)
flyButton.Position = UDim2.new(0,10,0,10)
flyButton.Text = "VOAR"

local imgButton = Instance.new("ImageButton", frame)
imgButton.Size = UDim2.new(0,50,0,50)
imgButton.Position = UDim2.new(0,10,0,65)
imgButton.Image = "rbxassetid://YOUR_IMAGE_ID" -- substitua pelo ID da imagem

-- Teclas
local keysDown = {}

uis.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Keyboard then
        keysDown[input.KeyCode] = true
    end
end)

uis.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Keyboard then
        keysDown[input.KeyCode] = false
    end
end)

-- Ativa/desativa voo
flyButton.MouseButton1Click:Connect(function()
    flying = not flying
    local character = player.Character
    if character and character:FindFirstChild("Humanoid") then
        -- Desativa gravidade localmente
        character.Humanoid.PlatformStand = flying
        -- Cria uma força que cancela a gravidade
        if flying then
            if not character:FindFirstChild("AntiGravity") then
                local antiGrav = Instance.new("BodyVelocity")
                antiGrav.Name = "AntiGravity"
                antiGrav.MaxForce = Vector3.new(0, math.huge, 0)
                antiGrav.Velocity = Vector3.new(0,0,0)
                antiGrav.Parent = character.HumanoidRootPart
            end
        else
            local antiGrav = character:FindFirstChild("HumanoidRootPart"):FindFirstChild("AntiGravity")
            if antiGrav then antiGrav:Destroy() end
        end
    end
end)

-- Loop de voo LOCAL
runService.RenderStepped:Connect(function(delta)
    if flying then
        local character = player.Character
        if character and character:FindFirstChild("HumanoidRootPart") then
            local hrp = character.HumanoidRootPart
            local camCF = workspace.CurrentCamera.CFrame
            local direction = Vector3.new(0,0,0)

            if keysDown[Enum.KeyCode.W] then direction = direction + camCF.LookVector end
            if keysDown[Enum.KeyCode.S] then direction = direction - camCF.LookVector end
            if keysDown[Enum.KeyCode.A] then direction = direction - camCF.RightVector end
            if keysDown[Enum.KeyCode.D] then direction = direction + camCF.RightVector end
            if keysDown[Enum.KeyCode.E] then direction = direction + Vector3.new(0,1,0) end
            if keysDown[Enum.KeyCode.Q] then direction = direction + Vector3.new(0,-1,0) end

            if direction.Magnitude > 0 then
                hrp.CFrame = hrp.CFrame + direction.Unit * speed * delta
            end
        end
    end
end)
