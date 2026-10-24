-- =====================================================================
-- SPAM & FARM HUB (SEA 1) - BY ZTK (DIRECT VIEWPORT CLICK FIX)
-- =====================================================================

-- Proteção de Inicialização da UI
local success, Fluent = pcall(function()
    return loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
end)

if not success or not Fluent then
    warn("[ZTK HUB] Erro ao carregar a biblioteca Fluent UI!")
    return
end

local SaveManager, InterfaceManager
pcall(function()
    SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
end)
pcall(function()
    InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()
end)

-- Serviços do Roblox
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local VirtualInputManager = game:GetService("VirtualInputManager")

local player = Players.LocalPlayer

-- Janela do Hub
local Window = Fluent:CreateWindow({
    Title = "Spam & Farm Hub (Sea 1)",
    SubTitle = "by ZTK",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 440),
    Acrylic = true,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.RightControl
})

-- Abas
local Tabs = {
    Main = Window:AddTab({ Title = "Principal", Icon = "gamepad-2" }),
    Player = Window:AddTab({ Title = "Jogador", Icon = "user" }),
    Visuals = Window:AddTab({ Title = "Visuais", Icon = "eye" }),
    Settings = Window:AddTab({ Title = "Configurações", Icon = "settings" })
}

---------------------------------------------------------------------
-- VARIÁVEIS E ESTADOS
---------------------------------------------------------------------
local autoFarmCompleto = false
local antiFogAtivo = false
local espPlayerAtivo = false
local noClipAtivo = false
local walkSpeedVal = 16
local jumpPowerVal = 50

local targetNPCName = ""
local selectedSlotKey = Enum.KeyCode.Two
local isTakingQuest = false
local forcedOptionMode = "Auto (ID Calculado)"

local slotKeyMap = {
    ["Melee"] = Enum.KeyCode.One,
    ["Fruit"] = Enum.KeyCode.Two,
    ["Sword"] = Enum.KeyCode.Three,
    ["Weapon"] = Enum.KeyCode.Four
}

local originalFog = {
    FogStart = Lighting.FogStart,
    FogEnd = Lighting.FogEnd,
    Atmosphere = Lighting:FindFirstChildOfClass("Atmosphere") and Lighting:FindFirstChildOfClass("Atmosphere"):Clone() or nil
}

---------------------------------------------------------------------
-- TABELA DE QUESTS FIXA
---------------------------------------------------------------------
local FixedSea1Quests = {
    { LevelReq = 0,   QuestName = "BanditQuest1",   QuestID = 1, MobName = "Bandit",          CFrame = CFrame.new(1059.2, 16.9, 1550.4) },
    { LevelReq = 10,  QuestName = "JungleQuest",    QuestID = 1, MobName = "Monkey",          CFrame = CFrame.new(-1598.1, 36.8, 153.7) },
    { LevelReq = 15,  QuestName = "JungleQuest",    QuestID = 2, MobName = "Gorilla",         CFrame = CFrame.new(-1598.1, 36.8, 153.7) },
    { LevelReq = 30,  QuestName = "BuggyQuest1",    QuestID = 1, MobName = "Pirate",          CFrame = CFrame.new(-1140.1, 4.7, 3827.1) },
    { LevelReq = 40,  QuestName = "BuggyQuest1",    QuestID = 2, MobName = "Brute",           CFrame = CFrame.new(-1140.1, 4.7, 3827.1) },
    { LevelReq = 60,  QuestName = "DesertQuest",    QuestID = 1, MobName = "Desert Bandit",   CFrame = CFrame.new(895.9, 6.5, 4388.5) },
    { LevelReq = 75,  QuestName = "DesertQuest",    QuestID = 2, MobName = "Desert Officer",  CFrame = CFrame.new(895.9, 6.5, 4388.5) },
    { LevelReq = 90,  QuestName = "SnowQuest",      QuestID = 1, MobName = "Snow Bandit",     CFrame = CFrame.new(1385.8, 87.2, -1299.2) },
    { LevelReq = 100, QuestName = "SnowQuest",      QuestID = 2, MobName = "Snowman",         CFrame = CFrame.new(1385.8, 87.2, -1299.2) },
    { LevelReq = 120, QuestName = "MarineQuest2",   QuestID = 1, MobName = "Chief Petty Officer", CFrame = CFrame.new(-2443.8, 7.3, 3300.2) },
    { LevelReq = 150, QuestName = "SkyQuest",       QuestID = 1, MobName = "Sky Bandit",      CFrame = CFrame.new(-4839.5, 717.6, -2619.4) },
    { LevelReq = 175, QuestName = "SkyQuest",       QuestID = 2, MobName = "Dark Master",     CFrame = CFrame.new(-4839.5, 717.6, -2619.4) },
    { LevelReq = 190, QuestName = "PrisonerQuest",  QuestID = 1, MobName = "Prisoner",        CFrame = CFrame.new(4875.1, 5.7, 734.5) },
    { LevelReq = 210, QuestName = "PrisonerQuest",  QuestID = 2, MobName = "Dangerous Prisoner", CFrame = CFrame.new(4875.1, 5.7, 734.5) },
    { LevelReq = 225, QuestName = "ColosseumQuest", QuestID = 1, MobName = "Toga Warrior",    CFrame = CFrame.new(-1255.4, 6.3, -7452.2) },
    { LevelReq = 250, QuestName = "ColosseumQuest", QuestID = 2, MobName = "Gladiator",       CFrame = CFrame.new(-1255.4, 6.3, -7452.2) },
    { LevelReq = 275, QuestName = "MagmaQuest",     QuestID = 1, MobName = "Military Soldier", CFrame = CFrame.new(-5316.8, 12.3, 8515.1) },
    { LevelReq = 300, QuestName = "MagmaQuest",     QuestID = 2, MobName = "Military Spy",    CFrame = CFrame.new(-5316.8, 12.3, 8515.1) },
    { LevelReq = 375, QuestName = "FishmanQuest",   QuestID = 1, MobName = "Fishman Warrior", CFrame = CFrame.new(61122.3, 18.5, 1569.3) },
    { LevelReq = 400, QuestName = "FishmanQuest",   QuestID = 2, MobName = "Fishman Commando",CFrame = CFrame.new(61122.3, 18.5, 1569.3) },
    { LevelReq = 450, QuestName = "SkyExp1Quest",   QuestID = 1, MobName = "God's Guard",     CFrame = CFrame.new(-7862.1, 5545.4, -381.4) },
    { LevelReq = 475, QuestName = "SkyExp1Quest",   QuestID = 2, MobName = "Shandora Warrior",CFrame = CFrame.new(-7862.1, 5545.4, -381.4) },
    { LevelReq = 525, QuestName = "SkyExp2Quest",   QuestID = 1, MobName = "Royal Squad",     CFrame = CFrame.new(-7903.1, 5635.6, -1412.8) },
    { LevelReq = 550, QuestName = "SkyExp2Quest",   QuestID = 2, MobName = "Royal Soldier",   CFrame = CFrame.new(-7903.1, 5635.6, -1412.8) },
    { LevelReq = 625, QuestName = "FountainQuest",  QuestID = 1, MobName = "Galley Pirate",   CFrame = CFrame.new(5257, 38, 4049) },
    { LevelReq = 650, QuestName = "FountainQuest",  QuestID = 2, MobName = "Galley Captain",  CFrame = CFrame.new(5257, 38, 4049) }
}

---------------------------------------------------------------------
-- ESP PLAYER SYSTEM (NOME + DISTÂNCIA)
---------------------------------------------------------------------
local function applyESP(targetPlayer)
    if targetPlayer == player then return end

    local function setupESP(char)
        if not char then return end
        local head = char:WaitForChild("Head", 5)
        if not head then return end

        if head:FindFirstChild("ZTK_NameESP") then return end

        local billboard = Instance.new("BillboardGui")
        billboard.Name = "ZTK_NameESP"
        billboard.Adornee = head
        billboard.Size = UDim2.new(0, 200, 0, 50)
        billboard.StudsOffset = Vector3.new(0, 2.5, 0)
        billboard.AlwaysOnTop = true

        local textLabel = Instance.new("TextLabel")
        textLabel.Name = "Text"
        textLabel.Size = UDim2.new(1, 0, 1, 0)
        textLabel.BackgroundTransparency = 1
        textLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
        textLabel.TextStrokeTransparency = 0
        textLabel.TextSize = 14
        textLabel.Font = Enum.Font.SourceSansBold
        textLabel.Parent = billboard

        billboard.Parent = head

        local connection
        connection = RunService.RenderStepped:Connect(function()
            if not espPlayerAtivo or not char or not char.Parent or not head.Parent then
                billboard:Destroy()
                if connection then connection:Disconnect() end
                return
            end

            local localChar = player.Character
            if localChar and localChar:FindFirstChild("HumanoidRootPart") then
                local dist = (localChar.HumanoidRootPart.Position - head.Position).Magnitude
                textLabel.Text = string.format("%s [%d studs]", targetPlayer.Name, math.floor(dist))
            else
                textLabel.Text = targetPlayer.Name
            end
        end)
    end

    if targetPlayer.Character then
        task.spawn(function() setupESP(targetPlayer.Character) end)
    end
    targetPlayer.CharacterAdded:Connect(function(char)
        task.wait(1)
        if espPlayerAtivo then setupESP(char) end
    end)
end

local function removeESP(targetPlayer)
    if targetPlayer.Character then
        local head = targetPlayer.Character:FindFirstChild("Head")
        if head and head:FindFirstChild("ZTK_NameESP") then
            head.ZTK_NameESP:Destroy()
        end
    end
end

for _, p in ipairs(Players:GetPlayers()) do applyESP(p) end
Players.PlayerAdded:Connect(applyESP)

---------------------------------------------------------------------
-- LÓGICA DE SELEÇÃO DE QUEST
---------------------------------------------------------------------
local function getBestQuestFromTable()
    local playerLevel = 1
    local levelObj = player:FindFirstChild("Level") or (player:FindFirstChild("Data") and player.Data:FindFirstChild("Level"))
    if levelObj and levelObj:IsA("IntValue") then
        playerLevel = levelObj.Value
    end

    local bestQuest = nil
    local bestReqLevel = -1

    for _, q in ipairs(FixedSea1Quests) do
        if playerLevel >= q.LevelReq and q.LevelReq > bestReqLevel then
            bestReqLevel = q.LevelReq
            bestQuest = q
        end
    end

    return bestQuest
end

local function hasActiveQuest()
    local pGui = player:FindFirstChild("PlayerGui")
    if not pGui then return false end

    local mainGui = pGui:FindFirstChild("Main")
    local questGui = mainGui and mainGui:FindFirstChild("Quest")
    return questGui and questGui.Visible or false
end

local function getValidNPC()
    if targetNPCName == "" then return nil, nil end

    local function checkModel(model)
        if model and model:IsA("Model") and string.lower(model.Name) == string.lower(targetNPCName) then
            local hum = model:FindFirstChildOfClass("Humanoid")
            local root = model:FindFirstChild("HumanoidRootPart")
            if hum and root and hum.Health > 0 then
                return root, hum
            end
        end
        return nil, nil
    end

    local root, hum = checkModel(workspace:FindFirstChild(targetNPCName))
    if root then return root, hum end

    for _, obj in ipairs(workspace:GetDescendants()) do
        root, hum = checkModel(obj)
        if root then return root, hum end
    end

    return nil, nil
end

local function triggerGuiButton(guiObj)
    if not guiObj or not guiObj:IsA("GuiObject") then return end
    
    pcall(function()
        if guiObj:IsA("TextButton") or guiObj:IsA("ImageButton") then
            for _, signal in ipairs({guiObj.MouseButton1Click, guiObj.MouseButton1Down, guiObj.Activated}) do
                for _, connection in ipairs(getconnections(signal)) do
                    connection:Fire()
                end
            end
        end
    end)

    local absPos = guiObj.AbsolutePosition
    local absSize = guiObj.AbsoluteSize
    local clickX = absPos.X + (absSize.X / 2)
    local clickY = absPos.Y + (absSize.Y / 2) + 36

    VirtualInputManager:SendMouseButtonEvent(clickX, clickY, 0, true, game, 1)
    VirtualInputManager:SendMouseButtonEvent(clickX, clickY, 0, false, game, 1)
end

-- CLIQUE DE MOUSE NO CENTRO DO ECRÃ (ATÉ 5 VEZES A CADA 0.1s ATÉ O DIÁLOGO SUMIR)
local function spamClickToCloseDialogue()
    local camera = workspace.CurrentCamera
    local centerX = camera.ViewportSize.X / 2
    local centerY = camera.ViewportSize.Y / 2

    for i = 1, 5 do
        local pGui = player:FindFirstChild("PlayerGui")
        local currentDialogue = pGui and pGui:FindFirstChild("Dialogue", true)
        if not currentDialogue or not currentDialogue.Visible then
            break -- Para assim que o diálogo desaparecer
        end

        VirtualInputManager:SendMouseButtonEvent(centerX, centerY, 0, true, game, 1)
        task.wait(0.02)
        VirtualInputManager:SendMouseButtonEvent(centerX, centerY, 0, false, game, 1)

        task.wait(0.1)
    end
end

local function processExactQuestSelection(targetID)
    local pGui = player:FindFirstChild("PlayerGui")
    if not pGui then return false end

    local selectOptionNumber = targetID
    if forcedOptionMode == "Option1 Apenas" then
        selectOptionNumber = 1
    elseif forcedOptionMode == "Option2 Apenas" then
        selectOptionNumber = 2
    end

    local dialogueFrame = nil
    for _ = 1, 20 do
        dialogueFrame = pGui:FindFirstChild("Dialogue", true)
        if dialogueFrame and dialogueFrame.Visible then break end
        task.wait(0.05)
    end

    if not (dialogueFrame and dialogueFrame.Visible) then return false end
    local initialPos = dialogueFrame.AbsolutePosition

    local targetName = "Option" .. tostring(selectOptionNumber)
    local questBtn = nil
    
    for _ = 1, 10 do
        questBtn = dialogueFrame:FindFirstChild(targetName, true)
        if questBtn and questBtn.Visible then break end
        task.wait(0.05)
    end

    if questBtn and questBtn.Visible then
        triggerGuiButton(questBtn)
    else
        return false
    end

    local positionChanged = false
    local timeout = os.clock() + 2.0

    while os.clock() < timeout do
        if dialogueFrame and dialogueFrame.Parent and dialogueFrame.Visible then
            local currentPos = dialogueFrame.AbsolutePosition
            if math.abs(currentPos.X - initialPos.X) >= 1 or math.abs(currentPos.Y - initialPos.Y) >= 1 then
                positionChanged = true
                break
            end
        end
        task.wait(0.02)
    end

    -- Se mudou de posição, confirma a missão clicando no Option1
    if positionChanged then
        task.wait(0.5)
        dialogueFrame = pGui:FindFirstChild("Dialogue", true)
        if dialogueFrame and dialogueFrame.Visible then
            local confirmBtn = dialogueFrame:FindFirstChild("Option1", true)
            if confirmBtn and confirmBtn.Visible then
                triggerGuiButton(confirmBtn)
            end
        end
    end

    -- ESPERA 0.5s APÓS A CONFIRMAÇÃO E EXECUTA O SPAM DE CLIQUES NO CENTRO DO ECRÃ
    task.wait(0.5)
    spamClickToCloseDialogue()

    return true
end

---------------------------------------------------------------------
-- LOOPS DO SCRIPT
---------------------------------------------------------------------
RunService.RenderStepped:Connect(function()
    local char = player.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            hum.UseJumpPower = true
            hum.WalkSpeed = walkSpeedVal
            hum.JumpPower = jumpPowerVal
        end
        if noClipAtivo then
            for _, part in ipairs(char:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end
    end
end)

-- Loop de Pegar Quest
task.spawn(function()
    while true do
        if autoFarmCompleto then
            pcall(function()
                if not hasActiveQuest() and not isTakingQuest then
                    isTakingQuest = true
                    
                    local questData = getBestQuestFromTable()
                    if questData then
                        targetNPCName = questData.MobName
                        local char = player.Character
                        
                        if char and char:FindFirstChild("HumanoidRootPart") then
                            char.HumanoidRootPart.CFrame = questData.CFrame + Vector3.new(0, 3, 0)
                            task.wait(0.3)

                            for _, v in ipairs(workspace:GetDescendants()) do
                                if v:IsA("ProximityPrompt") and (v.Parent.Position - char.HumanoidRootPart.Position).Magnitude < 15 then
                                    fireproximityprompt(v)
                                    break
                                end
                            end

                            local camera = workspace.CurrentCamera
                            local screenPos, onScreen = camera:WorldToViewportPoint(questData.CFrame.Position)
                            if onScreen then
                                VirtualInputManager:SendMouseButtonEvent(screenPos.X, screenPos.Y, 0, true, game, 1)
                                VirtualInputManager:SendMouseButtonEvent(screenPos.X, screenPos.Y, 0, false, game, 1)
                            end

                            processExactQuestSelection(questData.QuestID)
                        end
                    end
                    isTakingQuest = false
                end
            end)
        end
        task.wait(0.5)
    end
end)

-- Loop de Posicionamento nas Costas do Mob
task.spawn(function()
    while true do
        if autoFarmCompleto and hasActiveQuest() and not isTakingQuest then
            pcall(function()
                local char = player.Character
                if char and char:FindFirstChild("HumanoidRootPart") then
                    local npcRoot = getValidNPC()
                    if npcRoot then
                        local backPos = npcRoot.Position - (npcRoot.CFrame.LookVector * 5) + Vector3.new(0, 1, 0)
                        char.HumanoidRootPart.CFrame = CFrame.lookAt(backPos, npcRoot.Position)
                    end
                end
            end)
        end
        RunService.RenderStepped:Wait()
    end
end)

-- Loop de Ataque Ultra Spam por Tool:Activate() (SÓ COM QUEST ATIVA)
task.spawn(function()
    while true do
        if autoFarmCompleto and hasActiveQuest() and not isTakingQuest then
            pcall(function()
                local char = player.Character
                local tool = char and char:FindFirstChildOfClass("Tool")
                if tool then
                    for _ = 1, 3 do
                        tool:Activate()
                    end
                end
            end)
        end
        RunService.RenderStepped:Wait()
    end
end)

-- Loop de Equipar Slot da Hotbar
task.spawn(function()
    while true do
        if autoFarmCompleto and hasActiveQuest() and not isTakingQuest then
            pcall(function()
                VirtualInputManager:SendKeyEvent(true, selectedSlotKey, false, game)
                VirtualInputManager:SendKeyEvent(false, selectedSlotKey, false, game)
            end)
        end
        task.wait(0.1)
    end
end)

---------------------------------------------------------------------
-- INTERFACE (FLUENT UI)
---------------------------------------------------------------------

-- ABA PRINCIPAL
local ToggleAutoFarmCompleto = Tabs.Main:AddToggle("AutoFarmCompleto", {
    Title = "Auto Farm Completo",
    Default = false
})

ToggleAutoFarmCompleto:OnChanged(function(Value)
    autoFarmCompleto = Value
    isTakingQuest = false
end)

local DropdownForcedOption = Tabs.Main:AddDropdown("ForcedOptionSelect", {
    Title = "Forçar Botão da Missão do NPC",
    Values = {"Auto (ID Calculado)", "Option2 Apenas", "Option1 Apenas"},
    Multi = false,
    Default = "Auto (ID Calculado)",
    Callback = function(Value)
        forcedOptionMode = Value
    end
})

local DropdownSlot = Tabs.Main:AddDropdown("SlotSelect", {
    Title = "Selecione o Slot do Ataque",
    Values = {"Melee", "Fruit", "Sword", "Weapon"},
    Multi = false,
    Default = "Fruit",
    Callback = function(Value)
        if slotKeyMap[Value] then
            selectedSlotKey = slotKeyMap[Value]
        end
    end
})

-- ABA JOGADOR
local SliderSpeed = Tabs.Player:AddSlider("WalkSpeedSlider", {
    Title = "Velocidade de Movimento (Speed)",
    Default = 16,
    Min = 16,
    Max = 200,
    Rounding = 0,
    Callback = function(Value)
        walkSpeedVal = Value
    end
})

local SliderJump = Tabs.Player:AddSlider("JumpPowerSlider", {
    Title = "Força do Pulo (Jump)",
    Default = 50,
    Min = 50,
    Max = 300,
    Rounding = 0,
    Callback = function(Value)
        jumpPowerVal = Value
    end
})

local ToggleNoClip = Tabs.Player:AddToggle("NoClipToggle", {
    Title = "Ativar NoClip (Atravessar Paredes)",
    Default = false
})

ToggleNoClip:OnChanged(function(Value)
    noClipAtivo = Value
end)

-- ABA VISUAIS
local ToggleESPPlayer = Tabs.Visuals:AddToggle("ESPPlayerToggle", {
    Title = "Ativar ESP (Nome + Distância)",
    Default = false
})

ToggleESPPlayer:OnChanged(function(Value)
    espPlayerAtivo = Value
    for _, p in ipairs(Players:GetPlayers()) do
        if espPlayerAtivo then
            applyESP(p)
        else
            removeESP(p)
        end
    end
end)

local ToggleAntiFog = Tabs.Visuals:AddToggle("AntiFog", {
    Title = "Remover Fog / Atmosfera (Anti-Fog)",
    Default = false
})

ToggleAntiFog:OnChanged(function(Value)
    antiFogAtivo = Value
    if antiFogAtivo then
        Lighting.FogStart = 9e9
        Lighting.FogEnd = 9e9
        for _, v in ipairs(Lighting:GetChildren()) do
            if v:IsA("Atmosphere") or v:IsA("Clouds") then
                v.Parent = nil
            end
        end
    else
        Lighting.FogStart = originalFog.FogStart
        Lighting.FogEnd = originalFog.FogEnd
        if originalFog.Atmosphere then
            originalFog.Atmosphere.Parent = Lighting
        end
    end
end)

-- Keybind ALT
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.LeftAlt or input.KeyCode == Enum.KeyCode.RightAlt then
        ToggleAutoFarmCompleto:SetValue(not autoFarmCompleto)
    end
end)

if InterfaceManager and SaveManager then
    InterfaceManager:SetLibrary(Fluent)
    SaveManager:SetLibrary(Fluent)
    SaveManager:IgnoreThemeSettings()
    SaveManager:SetIgnoreIndexes({})
    SaveManager:BuildConfigSection(Tabs.Settings)
end

Window:SelectTab(1)

-- Notificação
Fluent:Notify({
    Title = "Hub ZTK Atualizado!",
    Content = "Cliques diretos no centro do ecrã para fechar o diálogo aplicados.",
    Duration = 5
})
