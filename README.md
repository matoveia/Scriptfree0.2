local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = game.Workspace.CurrentCamera
local AimbotEnabled = false
local UIS = game:GetService("UserInputService")

-- Criar botão de ativação/desativação
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local ToggleButton = Instance.new("TextButton", ScreenGui)

ToggleButton.Size = UDim2.new(0, 100, 0, 50)
ToggleButton.Position = UDim2.new(0.1, 0, 0.1, 0)
ToggleButton.Text = "Aimbot OFF"
ToggleButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
ToggleButton.TextColor3 = Color3.new(1, 1, 1)

ToggleButton.MouseButton1Click:Connect(function()
    AimbotEnabled = not AimbotEnabled
    if AimbotEnabled then
        ToggleButton.Text = "Aimbot ON"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
    else
        ToggleButton.Text = "Aimbot OFF"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    end
end)

-- Função para encontrar o inimigo mais próximo
local function GetClosestEnemy()
    local ClosestEnemy = nil
    local ShortestDistance = math.huge

    for _, Player in pairs(Players:GetPlayers()) do
        if Player ~= LocalPlayer and Player.Character and Player.Character:FindFirstChild("Head") then
            -- Verificar se o inimigo é de um time diferente
            if Player.Team ~= LocalPlayer.Team then
                local EnemyPos, OnScreen = Camera:WorldToViewportPoint(Player.Character.Head.Position)
                if OnScreen then
                    local Distance = (Vector2.new(EnemyPos.X, EnemyPos.Y) - UserInputService:GetMouseLocation()).Magnitude
                    if Distance < ShortestDistance then
                        ShortestDistance = Distance
                        ClosestEnemy = Player.Character.Head
                    end
                end
            end
        end
    end
    return ClosestEnemy
end

-- Função para fazer o tiro acertar automaticamente
local function Aimbot()
    while true do
        if AimbotEnabled then
            local Target = GetClosestEnemy()
            if Target then
                -- Mira suavizada para evitar movimentos abruptos
                local TargetPos = Target.Position
                local Direction = (TargetPos - Camera.CFrame.Position).unit
                Camera.CFrame = CFrame.new(Camera.CFrame.Position, Camera.CFrame.Position + Direction)
            end
        end
        task.wait(0.05)  -- Ajuste de performance, não precisa ser tão rápido
    end
end

-- Iniciar Aimbot
task.spawn(Aimbot)

-- Função de deslizar (slide)
local Sliding = false
local function Slide()
    if not Sliding and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        Sliding = true
        local Humanoid = LocalPlayer.Character:FindFirstChild("Humanoid")
        Humanoid.WalkSpeed = 50  -- Aumenta a velocidade durante o slide
        task.wait(0.5) -- Duração do slide
        Humanoid.WalkSpeed = 16 -- Volta ao normal
        Sliding = false
    end
end

-- Atalhos do teclado
UIS.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.LeftControl then -- Tecla para deslizar
        Slide()
    end
end)
