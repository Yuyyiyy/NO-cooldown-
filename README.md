-- ✅ Allowed Users List
local testUsers = {
    "Test1", "test2", "test3", "test 4", "test 5",
    "", "Mr_Incredible120", "Test8", "Test9", "Test10",
    "Test11", "Test12", "Test13", "Test14", "Test15",
    "testUser16", "testUser17", "testUser18", "testUser19", "testUser20",
    "testUser21", "testUser22", "testUser23", "testUser24", "testUser25",
    "testUser26", "testUser27", "testUser28", "testUser29", "testUser30",
    "testUser31", "testUser32", "testUser33", "testUser34", "testUser35",
    "testUser36"
}

-- ✅ Services
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local Debris = game:GetService("Debris")
local LocalPlayer = Players.LocalPlayer

-- ✅ User Verification
local isVerified = false
for _, name in ipairs(testUsers) do
    if name == LocalPlayer.Name then
        isVerified = true
        break
    end
end

if not isVerified then
    warn("Access denied: You are not an allowed user.")
    return
end

-- ✅ Wait overrides
local function stableWait(...)
    RunService.Heartbeat:Wait()
    return nil
end

hookfunction(wait, stableWait)
hookfunction(task.wait, stableWait)
hookfunction(delay, function(_, func)
    task.spawn(func)
    return nil
end)
hookfunction(spawn, function(func)
    task.spawn(func)
end)

local function disableYieldFunctions()
    local funcsToOverride = {"wait", "delay", "spawn", "task.wait"}
    for _, funcName in ipairs(funcsToOverride) do
        local originalFunc = _G[funcName]
        hookfunction(originalFunc, function(...)
            return stableWait(...)
        end)
    end
end
disableYieldFunctions()

-- ✅ Core Functions

local function antiExploit()
    local exploitedPlayers = {}
    Players.PlayerAdded:Connect(function(player)
        player.CharacterAdded:Connect(function(character)
            local humanoid = character:WaitForChild("Humanoid")
            local rootPart = character:WaitForChild("HumanoidRootPart")
            local lastPosition = rootPart.Position
            local speedThreshold = 1e5
            RunService.Heartbeat:Connect(function()
                local currentPosition = rootPart.Position
                local distance = (currentPosition - lastPosition).Magnitude
                if distance > speedThreshold then
                    rootPart.CFrame = CFrame.new(lastPosition)
                    if not exploitedPlayers[player.UserId] then
                        exploitedPlayers[player.UserId] = true
                        player:Kick("Speed hacking detected.")
                    end
                end
                lastPosition = currentPosition
            end)
        end)
    end)
end

local function protectRemoteFunctions()
    for _, remote in pairs(ReplicatedStorage:GetChildren()) do
        if remote:IsA("RemoteEvent") or remote:IsA("RemoteFunction") then
            hookfunction(remote.OnServerEvent, function(_, player, ...)
                local args = {...}
                for _, arg in pairs(args) do
                    if type(arg) == "table" then
                        return
                    end
                end
                return remote.OnServerEvent(_, player, ...)
            end)
        end
    end
end

local function clearEffects(part)
    for _, fx in pairs(part:GetChildren()) do
        if fx:IsA("ParticleEmitter") or fx:IsA("Fire") or fx:IsA("Smoke") then
            fx:Destroy()
        end
    end
end

local function obliterateCharacter(char)
    if not char then return end
    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") then
            part.Anchored = false
            part.Velocity = Vector3.new(math.random(-1e9, 1e9), 1e9, math.random(-1e9, 1e9))
            part.Transparency = 1
            clearEffects(part)
            pcall(function() part:Destroy() end)
        elseif part:IsA("Tool") or part:IsA("Accessory") or part:IsA("Hat") then
            pcall(function() part:Destroy() end)
        end
    end
end

local function killAndFreeze(humanoid, char)
    if humanoid and humanoid.Health > 0 then
        humanoid:TakeDamage(math.huge)
        humanoid.Health = 0
        humanoid:ChangeState(Enum.HumanoidStateType.Dead)
    end
    obliterateCharacter(char)
end

local function handleMegaClaws()
    local player = Players.LocalPlayer

    local function onAttack(target)
        if target then
            local targetHumanoid = target:FindFirstChildOfClass("Humanoid")
            if targetHumanoid then
                targetHumanoid:TakeDamage(math.huge)
                killAndFreeze(targetHumanoid, target)
            end
        end
    end

    local function detectAndAttack()
        task.spawn(function()
            while true do
                for i = 1, 9000 do
                    task.spawn(function()
                        for _, targetPlayer in pairs(Players:GetPlayers()) do
                            if targetPlayer ~= player then
                                local character = targetPlayer.Character
                                if character then
                                    local humanoid = character:FindFirstChildOfClass("Humanoid")
                                    if humanoid and humanoid.Health > 0 then
                                        local distance = (player.Character.HumanoidRootPart.Position - character.HumanoidRootPart.Position).Magnitude
                                        if distance < 900000000 * 9 then
                                            onAttack(character)
                                        end
                                    end
                                end
                            end
                        end
                    end)
                end
                RunService.Heartbeat:Wait()
            end
        end)
    end

    detectAndAttack()
end

local function enhancedExploitDetection()
    Players.PlayerAdded:Connect(function(player)
        player.CharacterAdded:Connect(function(character)
            local humanoidRoot = character:WaitForChild("HumanoidRootPart")
            local humanoid = character:WaitForChild("Humanoid")
            local previousPosition = humanoidRoot.Position
            local previousVelocity = humanoidRoot.AssemblyLinearVelocity
            local extremeSpeed = 999e999999999999999999999000000000000000000000000000000000000000000000000000000000000000000000000000000000000
            local teleportThreshold = 9e9900000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
            RunService.Heartbeat:Connect(function()
                if (humanoidRoot.Position - previousPosition).Magnitude > extremeSpeed or
                   (humanoidRoot.AssemblyLinearVelocity - previousVelocity).Magnitude > extremeSpeed then
                    humanoid.Health = 0
                    player:Kick("Exploit detected: Speed hack or abnormal velocity!")
                end

                if (humanoidRoot.Position - previousPosition).Magnitude > teleportThreshold then
                    humanoid.Health = 0
                    player:Kick("Exploit detected: Teleportation hack!")
                end

                previousPosition = humanoidRoot.Position
                previousVelocity = humanoidRoot.AssemblyLinearVelocity
            end)
        end)
    end)
end

local function ultraClaws()
    local player = Players.LocalPlayer
    local clawDamage = 9999999999e99999999999990000000000000000000000000000

    local function clawVaporize(target)
        if target and target.Parent then
            local humanoid = target:FindFirstChildOfClass("Humanoid")
            if humanoid and humanoid.Health > 0 then
                humanoid.Health = 0
                for i = 1, 9000000000000 do
                    task.spawn(function()
                        local force = Instance.new("BodyVelocity")
                        force.MaxForce = Vector3.new(1e150, 1e150, 1e150)
                        force.Velocity = Vector3.new(math.random(-1e6, 1e160), math.random(1e160, 2e160), math.random(-1e160, 1e160))
                        force.Parent = target.HumanoidRootPart
                        Debris:AddItem(force, 0.1)

                        local explosion = Instance.new("Explosion")
                        explosion.Position = target.HumanoidRootPart.Position
                        explosion.BlastRadius = 1900000000000000000 * 900000000
                        explosion.BlastPressure = 99e9000000000000000000000000000000000000000 * 9000000000
                        explosion.ExplosionType = Enum.ExplosionType.Craters
                        explosion.Parent = Workspace
                    end)
                end

                for _, part in ipairs(target:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.Anchored = false
                        part.Velocity = Vector3.new(math.random(-1e1100, 1e100), 1e100, math.random(-1e10, 1e10))
                        pcall(function() part:Destroy() end)
                    elseif part:IsA("Tool") or part:IsA("Accessory") then
                        pcall(function() part:Destroy() end)
                    end
                end
            end
        end
    end

    local function vaporizeNearbyPlayers()
        while true do
            RunService.Heartbeat:Wait()
            for _, targetPlayer in pairs(Players:GetPlayers()) do
                if targetPlayer ~= player then
                    local character = targetPlayer.Character
                    if character then
                        local humanoid = character:FindFirstChildOfClass("Humanoid")
                        if humanoid and humanoid.Health > 0 then
                            local distance = (player.Character.HumanoidRootPart.Position - character.HumanoidRootPart.Position).Magnitude
                            if distance < 9000000 * 900000 then
                                clawVaporize(character)
                            end
                        end
                    end
                end
            end
        end
    end

    vaporizeNearbyPlayers()
end

local function infinitePowerScaling()
    local player = Players.LocalPlayer
    local powerLevel = 99000000000000000000000
    local powerIncrement = 99000000000000000000
    local maxPower = 1e900000000000000000000000

    while true do
        RunService.Heartbeat:Wait()
        powerLevel = math.min(powerLevel + powerIncrement, maxPower)

        for i = 1, 90000000000000 do
            for _, targetPlayer in pairs(Players:GetPlayers()) do
                if targetPlayer ~= player then
                    local character = targetPlayer.Character
                    if character then
                        local humanoid = character:FindFirstChildOfClass("Humanoid")
                        if humanoid and humanoid.Health > 0 then
                            local distance = (player.Character.HumanoidRootPart.Position - character.HumanoidRootPart.Position).Magnitude
                            if distance < 1000000 * 900 then
                                task.spawn(function()
                                    humanoid:TakeDamage(powerLevel)
                                end)
                            end
                        end
                    end
                end
            end
        end
    end
end

-- ✅ Start Everything (Only for Verified Users)
task.spawn(function()
    antiExploit()
    protectRemoteFunctions()
    enhancedExploitDetection()
    handleMegaClaws()
    ultraClaws()
    infinitePowerScaling()
end)
