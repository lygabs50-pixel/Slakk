--////////////////////////////////////////////////////////////--
--  LOADSCRIPT UNIFICADO V30.0 (XENO OPTIMIZED)               --
--////////////////////////////////////////////////////////////--

if not game:IsLoaded() then game.Loaded:Wait() end

local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

task.spawn(function()
    local character, humanoid, hrp
    local flickEnabled = true 
    local bodyFlickEnabled = true 
    local flickDuration, flickSmooth = 0.06, 8
    local b_yaw, b_flickActive = 0, false

    -- [SISTEMA CAMERA 360]
    local function doCamera360Flick()
        if not flickEnabled then return end
        local cam = workspace.CurrentCamera
        local startCF = cam.CFrame
        local startPos, pitch, yaw = startCF.Position, startCF:ToEulerAnglesYXZ()
        coroutine.wrap(function()
            for i=1, flickSmooth do
                cam.CFrame = CFrame.new(startPos) * CFrame.fromEulerAnglesYXZ(pitch, yaw + math.rad(360)*(i/flickSmooth), 0)
                task.wait(flickDuration / flickSmooth)
            end
        end)()
    end

    -- [SISTEMA BODY FLICK]
    local function executeBodyFlick()
        if not bodyFlickEnabled or b_flickActive then return end
        b_flickActive = true
        task.spawn(doCamera360Flick)
        -- Aumenta velocidade das animações durante o giro
        if humanoid then 
            for _, t in ipairs(humanoid:GetPlayingAnimationTracks()) do 
                t:AdjustSpeed(3) 
            end 
        end
        task.delay(0.12, function() b_flickActive = false end)
    end

    -- Hook para detectar quando o jogador ataca (Xeno Compatibility)
    local R = ReplicatedStorage:WaitForChild("LightsaberRemotes", 10)
    local Attack = R:WaitForChild("Attack", 10)
    
    local oldNameCall;
    oldNameCall = hookmetamethod(game, "__namecall", function(self, ...)
        local method = getnamecallmethod()
        if self == Attack and method == "FireServer" then 
            task.spawn(executeBodyFlick) 
        end
        return oldNameCall(self, ...)
    end)

    -- [GATILHO ADICIONAL: BOTÃO ESQUERDO DO MOUSE]
    UserInputService.InputBegan:Connect(function(input, p)
        if p then return end
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            task.spawn(executeBodyFlick)
        end
    end)

    -- Controle de rotação do personagem
    RunService.RenderStepped:Connect(function()
        if not hrp or not bodyFlickEnabled then 
            if humanoid then humanoid.AutoRotate = true end 
            return 
        end
        
        if b_flickActive then
            humanoid.AutoRotate = false
            b_yaw = (b_yaw + 180) % 360 
            hrp.CFrame = CFrame.new(hrp.Position) * CFrame.Angles(0, math.rad(b_yaw), 0)
        else
            humanoid.AutoRotate = true
            local _, y, _ = hrp.CFrame:ToEulerAnglesYXZ()
            b_yaw = math.deg(y) -- Corrigido de degrees para deg para rodar no Xeno
        end
    end)

    local function refreshCharacter()
        character = LP.Character or LP.CharacterAdded:Wait()
        humanoid  = character:WaitForChild("Humanoid")
        hrp       = character:WaitForChild("HumanoidRootPart")
    end
    LP.CharacterAdded:Connect(refreshCharacter)
    refreshCharacter()

    -- [INTERFACE GUI]
    local gui = Instance.new("ScreenGui", LP:WaitForChild("PlayerGui"))
    gui.Name = "SaberFlicker"
    gui.ResetOnSpawn = false
    
    local toggleBtn = Instance.new("TextButton", gui)
    toggleBtn.Size, toggleBtn.Position = UDim2.new(0, 70, 0, 30), UDim2.fromOffset(50, 120)
    toggleBtn.Text = "MENU"
    toggleBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    toggleBtn.TextColor3 = Color3.new(1,1,1)
    toggleBtn.Draggable = true
    toggleBtn.Active = true
    Instance.new("UICorner", toggleBtn)

    local win = Instance.new("Frame", gui)
    win.Size, win.Position = UDim2.new(0, 200, 0, 110), UDim2.fromOffset(50, 160)
    win.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
    win.Active = true
    win.Draggable = true
    win.Visible = true
    Instance.new("UICorner", win)

    local flickBtn = Instance.new("TextButton", win) 
    flickBtn.Size, flickBtn.Position = UDim2.new(1, -20, 0, 40), UDim2.new(0, 10, 0, 10)
    flickBtn.Text = "360 CAM: ON (F8)"
    flickBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
    flickBtn.TextColor3 = Color3.new(1,1,1)
    Instance.new("UICorner", flickBtn)

    local bodyBtn = Instance.new("TextButton", win)
    bodyBtn.Size, bodyBtn.Position = UDim2.new(1, -20, 0, 40), UDim2.new(0, 10, 0, 60)
    bodyBtn.Text = "BODY GIRO: ON (J)"
    bodyBtn.BackgroundColor3 = Color3.fromRGB(0, 180, 0)
    bodyBtn.TextColor3 = Color3.new(1,1,1)
    Instance.new("UICorner", bodyBtn)

    -- [FUNÇÕES DOS BOTÕES]
    toggleBtn.MouseButton1Click:Connect(function() win.Visible = not win.Visible end)

    local function toggleFlick()
        flickEnabled = not flickEnabled
        flickBtn.Text = "360 CAM: " .. (flickEnabled and "ON (F8)" or "OFF (F8)")
        flickBtn.BackgroundColor3 = flickEnabled and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(150, 0, 0)
    end

    local function toggleBody()
        bodyFlickEnabled = not bodyFlickEnabled
        bodyBtn.Text = "BODY GIRO: " .. (bodyFlickEnabled and "ON (J)" or "OFF (J)")
        bodyBtn.BackgroundColor3 = bodyFlickEnabled and Color3.fromRGB(0, 180, 0) or Color3.fromRGB(180, 0, 0)
    end

    flickBtn.MouseButton1Click:Connect(toggleFlick)
    bodyBtn.MouseButton1Click:Connect(toggleBody)

    -- Atalhos de Teclado
    UserInputService.InputBegan:Connect(function(input, p)
        if p then return end
        if input.KeyCode == Enum.KeyCode.F8 then toggleFlick()
        elseif input.KeyCode == Enum.KeyCode.J then toggleBody() end
    end)
end)
