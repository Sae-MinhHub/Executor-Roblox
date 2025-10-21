-- MM2 Sae | Minh Hub - WindUI Lite Edition (Fake WindUI, Blue, Centered)
-- Paste vào executor và chạy ở trong game Murder Mystery 2
-- Author: adapted for user (keeps original logic, optimized)

-- ====== Safety & env setup ======
if not game or not game:IsA("DataModel") then return end
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
if not LocalPlayer then return end

getgenv().EspAll = getgenv().EspAll or false
getgenv().autoFarmEnabled = getgenv().autoFarmEnabled or false
getgenv().noclipEnabled = getgenv().noclipEnabled or false
getgenv().killerAllActive = getgenv().killerAllActive or false

-- small helper
local function safe_pcall(fn, ...)
    local ok, res = pcall(fn, ...)
    return ok, res
end

-- ====== GUI (WindUI-like, centered, blue) ======
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "SaeMinhHub_GUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 420, 0, 420)
mainFrame.Position = UDim2.new(0.5, -210, 0.5, -210) -- centered
mainFrame.BackgroundColor3 = Color3.fromHex("#071428") -- dark bluish
mainFrame.BorderSizePixel = 0
mainFrame.AnchorPoint = Vector2.new(0,0)
mainFrame.Parent = screenGui
mainFrame.Active = true
mainFrame.ClipsDescendants = true
mainFrame.ZIndex = 10
mainFrame.BackgroundTransparency = 0

-- shadow / rounded look
local uicorner = Instance.new("UICorner", mainFrame)
uicorner.CornerRadius = UDim.new(0, 12)

-- Title
local title = Instance.new("TextLabel", mainFrame)
title.Name = "Title"
title.Size = UDim2.new(1, 0, 0, 48)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundTransparency = 1
title.Text = "Sae | Minh Hub"
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.TextColor3 = Color3.fromHex("#BFE9FF")
title.TextXAlignment = Enum.TextXAlignment.Left
title.TextYAlignment = Enum.TextYAlignment.Center
title.Padding = nil
title.TextStrokeTransparency = 0.8
title.TextWrapped = false
title.RichText = false
title.BorderSizePixel = 0
title.Position = UDim2.new(0, 16, 0, 10)

-- Sub title / small
local subtitle = Instance.new("TextLabel", mainFrame)
subtitle.Size = UDim2.new(1, -32, 0, 20)
subtitle.Position = UDim2.new(0, 16, 0, 34)
subtitle.BackgroundTransparency = 1
subtitle.Text = "MM2 - WindUI Lite"
subtitle.Font = Enum.Font.Gotham
subtitle.TextSize = 12
subtitle.TextColor3 = Color3.fromHex("#9FD7FF")
subtitle.TextXAlignment = Enum.TextXAlignment.Left

-- Container for buttons
local content = Instance.new("Frame", mainFrame)
content.Name = "Content"
content.Position = UDim2.new(0, 12, 0, 64)
content.Size = UDim2.new(1, -24, 1, -76)
content.BackgroundTransparency = 1

-- simple button factory with animation
local function makeButton(name, text, posY, width)
    width = width or 1
    local btn = Instance.new("TextButton")
    btn.Name = name
    btn.Size = UDim2.new(width, 0, 0, 36)
    btn.Position = UDim2.new(0, 0, 0, posY)
    btn.BackgroundColor3 = Color3.fromHex("#007BFF")
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Text = text
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 14
    btn.AutoButtonColor = false
    btn.Parent = content
    local cr = Instance.new("UICorner", btn)
    cr.CornerRadius = UDim.new(0, 8)
    -- hover / press effect
    btn.MouseEnter:Connect(function()
        pcall(function() TweenService:Create(btn, TweenInfo.new(0.12, Enum.EasingStyle.Quad), {Size = btn.Size + UDim2.new(0,6,0,6)}):Play() end)
    end)
    btn.MouseLeave:Connect(function()
        pcall(function() TweenService:Create(btn, TweenInfo.new(0.12, Enum.EasingStyle.Quad), {Size = UDim2.new(width, 0, 0, 36)}):Play() end)
    end)
    return btn
end

-- Toggle factory
local function makeToggle(name, text, posY, default, callback)
    local frame = Instance.new("Frame", content)
    frame.Name = name .. "_Frame"
    frame.Size = UDim2.new(1, 0, 0, 36)
    frame.Position = UDim2.new(0, 0, 0, posY)
    frame.BackgroundTransparency = 1

    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(0.75, 0, 1, 0)
    label.Position = UDim2.new(0, 0, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.Font = Enum.Font.Gotham
    label.TextSize = 14
    label.TextColor3 = Color3.fromHex("#CFCFCF")
    label.TextXAlignment = Enum.TextXAlignment.Left

    local toggleBtn = Instance.new("TextButton", frame)
    toggleBtn.Size = UDim2.new(0.24, 0, 0.8, 0)
    toggleBtn.Position = UDim2.new(0.76, 0, 0.1, 0)
    toggleBtn.Text = ""
    toggleBtn.BackgroundColor3 = default and Color3.fromHex("#00D1FF") or Color3.fromHex("#2A394B")
    toggleBtn.AutoButtonColor = false
    toggleBtn.Font = Enum.Font.Gotham
    toggleBtn.TextSize = 14
    local cr = Instance.new("UICorner", toggleBtn)
    cr.CornerRadius = UDim.new(0, 10)

    local state = default
    toggleBtn.MouseButton1Click:Connect(function()
        state = not state
        toggleBtn.BackgroundColor3 = state and Color3.fromHex("#00D1FF") or Color3.fromHex("#2A394B")
        pcall(callback, state)
        -- press effect
        pcall(function()
            TweenService:Create(toggleBtn, TweenInfo.new(0.08), {Size = UDim2.new(0.24, 6, 0.8, 6)}):Play()
            wait(0.08)
            TweenService:Create(toggleBtn, TweenInfo.new(0.08), {Size = UDim2.new(0.24, 0, 0.8, 0)}):Play()
        end)
    end)

    return frame, function() return state end
end

-- layout positions (simple)
local y = 0
local spacing = 42

-- ESP toggle
local espFrame, getEspState = makeToggle("ESP_Toggle", "ESP (All Roles)", y, getgenv().EspAll, function(state)
    getgenv().EspAll = state
    for _, bb in ipairs(game.CoreGui:GetDescendants()) do
        if bb:IsA("BillboardGui") and bb.Name:match("_ESP$") then
            bb.Enabled = state
        end
    end
end)
y = y + spacing

-- Auto Farm toggle
local farmFrame, getFarmState = makeToggle("Farm_Toggle", "Auto Farm Coin (1000 studs)", y, getgenv().autoFarmEnabled, function(state)
    getgenv().autoFarmEnabled = state
end)
y = y + spacing

-- NoClip toggle
local noclipFrame, getNoclipState = makeToggle("Noclip_Toggle", "NoClip", y, getgenv().noclipEnabled, function(state)
    getgenv().noclipEnabled = state
end)
y = y + spacing

-- Kill All toggle
local killAllFrame, getKillAllState = makeToggle("KillAll_Toggle", "Kill All (Auto Teleport+Shoot)", y, getgenv().killerAllActive, function(state)
    getgenv().killerAllActive = state
end)
y = y + spacing

-- Buttons: Fly to coin, Get Gun, Auto Shoot Murderer, Info
local btnFly = makeButton("FlyToCoin", "Fly to 3rd Nearest Coin", y); y = y + spacing
local btnGetGun = makeButton("GetGun", "Get Gun Drop (Tween)", y); y = y + spacing
local btnAutoShoot = makeButton("AutoShoot", "Auto Shoot Murderer", y); y = y + spacing
local btnDiscord = makeButton("Discord", "Copy Discord Link", y); y = y + spacing

-- info small
local credit = Instance.new("TextLabel", content)
credit.Size = UDim2.new(1, 0, 0, 24)
credit.Position = UDim2.new(0, 0, 1, -28)
credit.BackgroundTransparency = 1
credit.Text = "Sae | Minh Hub - Lite"
credit.Font = Enum.Font.Gotham
credit.TextSize = 12
credit.TextColor3 = Color3.fromHex("#7FBFEF")
credit.TextXAlignment = Enum.TextXAlignment.Left

-- ====== ESP Implementation (optimized) ======
local ESPFolder = Instance.new("Folder")
ESPFolder.Name = "ESP_Holder"
ESPFolder.Parent = game.CoreGui

local function createBillboardForPlayer(plr)
    if not plr or not plr.Name then return end
    local id = plr.Name .. "_ESP"
    if ESPFolder:FindFirstChild(id) then return end
    local billboard = Instance.new("BillboardGui")
    billboard.Name = id
    billboard.Size = UDim2.new(0, 120, 0, 28)
    billboard.AlwaysOnTop = true
    billboard.ExtentsOffset = Vector3.new(0, 2.5, 0)
    billboard.Parent = ESPFolder
    billboard.Enabled = getgenv().EspAll

    local lbl = Instance.new("TextLabel", billboard)
    lbl.Size = UDim2.new(1,0,1,0)
    lbl.BackgroundTransparency = 1
    lbl.Font = Enum.Font.GothamBold
    lbl.TextSize = 13
    lbl.TextStrokeTransparency = 0.7
    lbl.TextColor3 = Color3.new(1,1,1)
    lbl.Text = plr.Name

    -- update loop
    coroutine.wrap(function()
        while plr.Parent do
            task.wait(0.12)
            pcall(function()
                local char = plr.Character
                billboard.Adornee = char and char:FindFirstChild("Head")
                local bp = plr:FindFirstChild("Backpack")
                local hasKnife = (char and char:FindFirstChild("Knife")) or (bp and bp:FindFirstChild("Knife"))
                local hasGun = (char and char:FindFirstChild("Gun")) or (bp and bp:FindFirstChild("Gun"))
                if hasKnife then
                    lbl.TextColor3 = Color3.fromHex("#FF6B6B") -- red
                elseif hasGun then
                    lbl.TextColor3 = Color3.fromHex("#6BCBFF") -- blue
                else
                    lbl.TextColor3 = Color3.fromHex("#8DFF9F") -- greenish
                end
                billboard.Enabled = getgenv().EspAll
            end)
        end
        if billboard and billboard.Parent then billboard:Destroy() end
    end)()
end

-- init esp for existing players
for _, p in ipairs(Players:GetPlayers()) do
    if p ~= LocalPlayer then createBillboardForPlayer(p) end
end
Players.PlayerAdded:Connect(function(p) if p ~= LocalPlayer then createBillboardForPlayer(p) end end)
Players.PlayerRemoving:Connect(function(p) local node = ESPFolder:FindFirstChild(p.Name .. "_ESP") if node then node:Destroy() end end)

-- ====== Coin AutoFarm (optimized, radius 1000 studs, tween mượt) ======
local coinCache = {}
local lastCoinUpdate = 0
local CACHE_DURATION = 0.9 -- refresh cache under 1s

local function refreshCoinCache()
    local now = tick()
    if now - lastCoinUpdate < CACHE_DURATION and #coinCache > 0 then return end
    coinCache = {}
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj and (obj:IsA("BasePart") or obj:IsA("MeshPart")) then
            local n = obj.Name:lower()
            if n:find("coin") or n:find("money") or n:find("dollar") or n:find("cash") then
                table.insert(coinCache, obj)
            end
        end
    end
    lastCoinUpdate = now
end

local function findNearbyCoins(radius)
    radius = radius or 1000
    refreshCoinCache()
    local root = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not root then return {} end
    local list = {}
    for _, c in ipairs(coinCache) do
        if c and c:IsDescendantOf(Workspace) and c.Position then
            local dist = (root.Position - c.Position).Magnitude
            if dist <= radius then
                table.insert(list, {obj = c, dist = dist})
            end
        end
    end
    table.sort(list, function(a,b) return a.dist < b.dist end)
    return list
end

local isTeleporting = false

local function tweenToPosition(root, targetPos, speed)
    speed = speed or 60 -- studs per second
    if not root or not root:IsA("BasePart") then return false end
    local current = root.Position
    local dist = (targetPos - current).Magnitude
    if dist < 0.5 then return true end
    local travelTime = math.clamp(dist / speed, 0.04, 2)
    local tweenInfo = TweenInfo.new(travelTime, Enum.EasingStyle.Linear)
    local goal = {CFrame = CFrame.new(targetPos)}
    local tw = TweenService:Create(root, tweenInfo, goal)
    tw:Play()
    local ok, _ = pcall(function() tw.Completed:Wait() end)
    return ok
end

-- Move with noclip (disable collisions temporarily)
local function moveToCoin(coinPart)
    if not coinPart or not coinPart.Position then return false end
    if isTeleporting then return false end
    isTeleporting = true
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then isTeleporting = false return false end
    local root = char.HumanoidRootPart

    -- disable collisions
    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CanCollide = false
        end
    end

    -- attempt to approach third nearest if exists (mimic old behavior)
    local coins = findNearbyCoins(1000)
    local target = (coins[3] and coins[3].obj) or coins[1] and coins[1].obj or coinPart
    if not target then
        -- reenable collisions
        for _, part in ipairs(char:GetDescendants()) do
            if part:IsA("BasePart") then part.CanCollide = true end
        end
        isTeleporting = false
        return false
    end

    -- ensure target pos uses slightly above to pick coin safely
    local targetPos = target.Position + Vector3.new(0, 1.5, 0)
    local ok = tweenToPosition(root, targetPos, 90) -- speed tuned
    wait(0.12) -- small wait
    -- final adjustment small Y
    local diffY = math.abs(root.Position.Y - target.Position.Y)
    if diffY > 0.5 then
        pcall(function()
            tweenToPosition(root, Vector3.new(target.Position.X, target.Position.Y + 1.2, target.Position.Z), 40)
        end)
    end

    -- reenable collisions
    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") then part.CanCollide = true end
    end

    isTeleporting = false
    return ok
end

-- Heartbeat connection for auto farm (lightweight)
local farmConn = nil
local function startAutoFarm()
    if farmConn then farmConn:Disconnect() end
    farmConn = RunService.Heartbeat:Connect(function(dt)
        if not getgenv().autoFarmEnabled then return end
        if isTeleporting then return end
        local char = LocalPlayer.Character
        if not char or not char:FindFirstChild("HumanoidRootPart") then return end
        local coins = findNearbyCoins(1000)
        if #coins == 0 then return end
        local target = coins[3] and coins[3].obj or coins[1].obj
        if target and target:IsDescendantOf(Workspace) then
            moveToCoin(target)
            task.wait(0.9) -- small pause to prevent spamming and give time to collect
        end
    end)
end

local function stopAutoFarm()
    if farmConn then
        farmConn:Disconnect()
        farmConn = nil
    end
end

-- Wire toggle to start/stop
coroutine.wrap(function()
    while true do
        task.wait(0.4)
        if getgenv().autoFarmEnabled and not farmConn then
            startAutoFarm()
        elseif not getgenv().autoFarmEnabled and farmConn then
            stopAutoFarm()
        end
    end
end)()

-- FlyToCoin button action
btnFly.MouseButton1Click:Connect(function()
    -- find nearest coin from full cache
    refreshCoinCache()
    local root = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not root then return end
    local coins = findNearbyCoins(1000)
    local target = coins[3] and coins[3].obj or coins[1] and coins[1].obj
    if target then
        moveToCoin(target)
    else
        -- small notify: no coins
        pcall(function() print("No coin found within 1000 studs.") end)
    end
    -- simple click animation
    pcall(function()
        TweenService:Create(btnFly, TweenInfo.new(0.08), {Size = UDim2.new(1, 6, 0, 36)}):Play()
        task.wait(0.08)
        TweenService:Create(btnFly, TweenInfo.new(0.08), {Size = UDim2.new(1, 0, 0, 36)}):Play()
    end)
end)

-- ====== NoClip implementation (safe) ======
local noclipConn = nil
local function startNoclip()
    if noclipConn then noclipConn:Disconnect() end
    noclipConn = RunService.Stepped:Connect(function()
        if not getgenv().noclipEnabled then return end
        local char = LocalPlayer.Character
        if char then
            for _, part in ipairs(char:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end
    end)
end

local function stopNoclip()
    if noclipConn then
        noclipConn:Disconnect()
        noclipConn = nil
    end
end

-- Keep toggles in sync
coroutine.wrap(function()
    while true do
        task.wait(0.25)
        if getgenv().noclipEnabled and not noclipConn then
            startNoclip()
        elseif not getgenv().noclipEnabled and noclipConn then
            stopNoclip()
            -- reenable collisions once
            local char = LocalPlayer.Character
            if char then
                for _, p in ipairs(char:GetDescendants()) do
                    if p:IsA("BasePart") then p.CanCollide = true end
                end
            end
        end
    end
end)()

-- ====== Murderer detection & Auto Shoot ======
local function findMurderer()
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character then
            local bp = p:FindFirstChild("Backpack")
            local hasKnife = (bp and bp:FindFirstChild("Knife")) or p.Character:FindFirstChild("Knife")
            if hasKnife then return p end
        end
    end
    return nil
end

local function findGunTool()
    local char = LocalPlayer.Character
    if char then
        for _, t in ipairs(char:GetChildren()) do
            if t:IsA("Tool") and (t.Name:lower():find("gun") or t.Name:lower():find("revolver")) then
                return t, true
            end
        end
    end
    local bp = LocalPlayer:FindFirstChild("Backpack")
    if bp then
        for _, t in ipairs(bp:GetChildren()) do
            if t:IsA("Tool") and (t.Name:lower():find("gun") or t.Name:lower():find("revolver")) then
                return t, false
            end
        end
    end
    return nil, false
end

local function findClickDetectorInCharacter(targetChar)
    for _, part in ipairs(targetChar:GetDescendants()) do
        if part:IsA("ClickDetector") then
            return part
        end
    end
    return nil
end

local function autoShootMurdererOnce()
    local murderer = findMurderer()
    if not murderer or not murderer.Character or not murderer.Character:FindFirstChild("HumanoidRootPart") then
        pcall(function() print("No murderer found!") end)
        return
    end
    local gun, inChar = findGunTool()
    if not gun then
        pcall(function() print("No gun found!") end)
        return
    end
    -- equip if in backpack
    if not inChar then
        pcall(function() LocalPlayer.Character.Humanoid:EquipTool(gun) end)
        task.wait(0.18)
    end
    -- aim camera towards murderer (not all executors support manipulating camera)
    pcall(function()
        local cam = Workspace.CurrentCamera
        cam.CFrame = CFrame.new(cam.CFrame.Position, murderer.Character.HumanoidRootPart.Position)
    end)
    task.wait(0.05)
    local detector = findClickDetectorInCharacter(murderer.Character)
    if detector then
        pcall(function() fireclickdetector(detector, 10, "MouseClick") end)
    else
        pcall(function() gun:Activate() end)
    end
    task.wait(0.06)
end

btnAutoShoot.MouseButton1Click:Connect(function()
    autoShootMurdererOnce()
    -- small animation
    pcall(function()
        TweenService:Create(btnAutoShoot, TweenInfo.new(0.08), {Size = UDim2.new(1, 6, 0, 36)}):Play()
        task.wait(0.08)
        TweenService:Create(btnAutoShoot, TweenInfo.new(0.08), {Size = UDim2.new(1, 0, 0, 36)}):Play()
    end)
end)

-- ====== Kill All (auto teleport+shoot) ======
local killerConn = nil
local function startKillAll()
    if killerConn then killerConn:Disconnect() end
    killerConn = RunService.Heartbeat:Connect(function()
        if not getgenv().killerAllActive or isTeleporting then return end
        local myRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if not myRoot then return end
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character and p.Character:Fin
