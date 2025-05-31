print("RLX BASIC v1 carregado com sucesso!")

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local TeleportService = game:GetService("TeleportService")

local player = Players.LocalPlayer
local mouse = player:GetMouse()

-- Controle de funções
local autoFarmEnabled = false
local fruitESPEnabled = false
local fastClickEnabled = false

-- Função para detectar mundo atual
local world = game.PlaceId
local currentWorld = "Desconhecido"

if world == 2753915549 then
    currentWorld = "Primeiro Mundo"
elseif world == 4442272183 then
    currentWorld = "Segundo Mundo"
elseif world == 7449423635 then
    currentWorld = "Terceiro Mundo"
end

print("Você está no: " .. currentWorld)

-- Função ESP Frutas
local function fruitESP()
    for _, fruit in pairs(Workspace:GetChildren()) do
        if fruit.Name:match("Fruit") and fruit:FindFirstChild("Handle") then
            if not fruit.Handle:FindFirstChild("Highlight") then
                local highlight = Instance.new("Highlight")
                highlight.Adornee = fruit.Handle
                highlight.FillColor = Color3.new(1, 0, 0)
                highlight.OutlineColor = Color3.new(1, 1, 0)
                highlight.Parent = fruit.Handle
            end
        end
    end
end

-- Função para pegar fruta automaticamente
local function pickupFruit()
    for _, fruit in pairs(Workspace:GetChildren()) do
        if fruit.Name:match("Fruit") and fruit:FindFirstChild("Handle") then
            local dist = (fruit.Handle.Position - player.Character.HumanoidRootPart.Position).Magnitude
            if dist < 15 then -- distância para pegar fruta
                firetouchinterest(player.Character.HumanoidRootPart, fruit.Handle, 0)
                firetouchinterest(player.Character.HumanoidRootPart, fruit.Handle, 1)
            end
        end
    end
end

-- Auto Farm simples (atacar inimigos próximos)
local function autoFarm()
    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end
    local hrp = player.Character.HumanoidRootPart
    for _, npc in pairs(Workspace:GetChildren()) do
        if npc:FindFirstChild("Humanoid") and npc.Humanoid.Health > 0 and npc.Name ~= player.Name then
            local npcRoot = npc:FindFirstChild("HumanoidRootPart")
            if npcRoot then
                local dist = (npcRoot.Position - hrp.Position).Magnitude
                if dist < 30 then -- distância para atacar
                    hrp.CFrame = CFrame.new(npcRoot.Position) * CFrame.new(0, 0, 3) -- se posiciona perto do npc
                    -- Atacar (exemplo de ataque básico: usar espada se tiver)
                    local tool = player.Character:FindFirstChildOfClass("Tool")
                    if tool then
                        tool:Activate()
                    end
                end
            end
        end
    end
end

-- Fast click (clicar rápido para atacar)
local function fastClick()
    if not player.Character then return end
    local tool = player.Character:FindFirstChildOfClass("Tool")
    if tool then
        tool:Activate()
    end
end

-- Teleporte rápido por clique no mapa (exemplo simples)
local teleportPoints = {
    ["FirstWorldSpawn"] = CFrame.new(100, 10, 100),
    ["SecondWorldSpawn"] = CFrame.new(200, 20, 200),
    ["ThirdWorldSpawn"] = CFrame.new(300, 30, 300),
}

local function teleportToPoint(name)
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local cf = teleportPoints[name]
        if cf then
            player.Character.HumanoidRootPart.CFrame = cf
        end
    end
end

-- GUI
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 230, 0, 250)
Frame.Position = UDim2.new(0, 10, 0.3, 0)
Frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Frame.BorderSizePixel = 0

local function createButton(name, posY, callback)
    local button = Instance.new("TextButton", Frame)
    button.Size = UDim2.new(1, -10, 0, 35)
    button.Position = UDim2.new(0, 5, 0, posY)
    button.Text = name
    button.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Font = Enum.Font.SourceSansBold
    button.TextSize = 18
    button.AutoButtonColor = true
    button.MouseButton1Click:Connect(callback)
    return button
end

-- Estado dos botões
local btnAutoFarm = createButton("Auto Farm: OFF", 10, function()
    autoFarmEnabled = not autoFarmEnabled
    btnAutoFarm.Text = "Auto Farm: " .. (autoFarmEnabled and "ON" or "OFF")
end)

local btnFruitESP = createButton("Fruit ESP: OFF", 55, function()
    fruitESPEnabled = not fruitESPEnabled
    btnFruitESP.Text = "Fruit ESP: " .. (fruitESPEnabled and "ON" or "OFF")
end)

local btnFastClick = createButton("Fast Click: OFF", 100, function()
    fastClickEnabled = not fastClickEnabled
    btnFastClick.Text = "Fast Click: " .. (fastClickEnabled and "ON" or "OFF")
end)

local btnTeleport = createButton("Teleport p/ Mundo", 145, function()
    if currentWorld == "Primeiro Mundo" then
        TeleportService:Teleport(4442272183, player) -- Para Segundo Mundo
    elseif currentWorld == "Segundo Mundo" then
        TeleportService:Teleport(7449423635, player) -- Para Terceiro Mundo
    elseif currentWorld == "Terceiro Mundo" then
        TeleportService:Teleport(2753915549, player) -- Para Primeiro Mundo
    else
        print("Mundo desconhecido para teleportar!")
    end
end)

local btnClose = createButton("Fechar GUI", 190, function()
    ScreenGui:Destroy()
end)

-- Loop principal
RunService.Heartbeat:Connect(function()
    if fruitESPEnabled then
        fruitESP()
        pickupFruit()
    end
    if autoFarmEnabled then
        autoFarm()
    end
    if fastClickEnabled then
        fastClick()
    end
end)
