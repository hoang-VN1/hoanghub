# repeat task.wait() until game:IsLoaded()  
  
local Players = game:GetService("Players")  
local RunService = game:GetService("RunService")  
local ReplicatedStorage = game:GetService("ReplicatedStorage")  
local TweenService = game:GetService("TweenService")  
local HttpService = game:GetService("HttpService")  
local TeleportService = game:GetService("TeleportService")  
local Stats = game:GetService("Stats")  
local UIS = game:GetService("UserInputService")  
local CoreGui = game:GetService("CoreGui")  
local SoundService = game:GetService("SoundService")  
local VirtualInputManager = game:GetService("VirtualInputManager")  
  
local SELECTED_TEAM = "Pirates"  -- Options: "Pirates" or "Marines"  
local Selected_Region = "Brazil" -- Your Region  
local SELECTED_FRUIT = "T-Rex" -- "T-Rex", "Dragon", "Kitsune", "Empyrean", "Pain", "Control"  
  
local lp = Players.LocalPlayer or Players:GetPropertyChangedSignal("LocalPlayer"):Wait() and Players.LocalPlayer  
  
getgenv().ShuttingDown = false  
getgenv().IsServerHopping = false  
  
local CurrentJobId = game.JobId  
local CurrentPlaceId = game.PlaceId  
  
game:GetService("CoreGui").RobloxPromptGui.promptOverlay.ChildAdded:Connect(function(child)  
    if child.Name == 'ErrorPrompt' and child:FindFirstChild('MessageArea') and child.MessageArea:FindFirstChild("ErrorFrame") then  
        if not getgenv().IsServerHopping then  
            TeleportService:TeleportToPlaceInstance(CurrentPlaceId, CurrentJobId, lp)  
        end  
    end  
end)  
  
local playerGui  
repeat  
    task.wait()  
    playerGui = lp:FindFirstChild("PlayerGui")  
until playerGui  
  
local mainGui  
repeat  
    task.wait()  
    mainGui = playerGui:FindFirstChild("Main (minimal)")  
until mainGui  
  
local btn  
repeat  
    task.wait()  
    if SELECTED_TEAM == "Pirates" then  
        btn = mainGui:FindFirstChild("ChooseTeam")  
            and mainGui.ChooseTeam:FindFirstChild("Container")  
            and  mainGui.ChooseTeam.Container:FindFirstChild("Pirates")  
            and mainGui.ChooseTeam.Container.Pirates:FindFirstChild("Frame")  
            and mainGui.ChooseTeam.Container.Pirates.Frame:FindFirstChild("TextButton")  
    elseif SELECTED_TEAM == "Marines" then  
        btn = mainGui:FindFirstChild("ChooseTeam")  
            and mainGui.ChooseTeam:FindFirstChild("Container")  
            and mainGui.ChooseTeam.Container:FindFirstChild("Marines")  
            and mainGui.ChooseTeam.Container.Marines:FindFirstChild("Frame")  
            and mainGui.ChooseTeam.Container.Marines.Frame:FindFirstChild("TextButton")  
    end  
until btn  
  
local char, hum  
repeat  
    task.wait(0.25)  
    pcall(function()  
        firesignal(btn.Activated)  
    end)  
      
    if lp.Character then  
        char = lp.Character  
        hum = char:FindFirstChild("Humanoid")  
    end  
until char and hum  
  
local hrp  
repeat  
    task.wait()  
    hrp = char:FindFirstChild("HumanoidRootPart")  
until hrp  
  
repeat task.wait() until ReplicatedStorage:FindFirstChild("Util")  
repeat task.wait() until ReplicatedStorage.Util:FindFirstChild("CameraShaker")  
repeat task.wait() until ReplicatedStorage.Util.CameraShaker:FindFirstChild("Main")  
  
local CameraShaker = require(ReplicatedStorage.Util.CameraShaker.Main)  
  
local noop = function() end  
CameraShaker.StartShake = noop  
CameraShaker.ShakeOnce = noop  
CameraShaker.ShakeSustain = noop  
CameraShaker.CameraShakeInstance = noop  
CameraShaker.Shake = noop  
CameraShaker.Start = noop  
  
local AntiSeatConnection  
local AntiSeatConnection2  
  
local function StartAntiSeat()  
    if AntiSeatConnection then  
        AntiSeatConnection:Disconnect()  
    end  
    if AntiSeatConnection2 then  
        AntiSeatConnection2:Disconnect()  
    end  
  
    local char = lp.Character  
    if not char then return end  
  
    local humanoid = char:WaitForChild("Humanoid", 10)  
    if not humanoid then return end  
  
    AntiSeatConnection = RunService.Heartbeat:Connect(function()  
        if humanoid.Sit then  
            humanoid.Sit = false  
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)  
        end  
    end)  
  
    AntiSeatConnection2 = humanoid.StateChanged:Connect(function(_, new)  
        if new == Enum.HumanoidStateType.Seated then  
            humanoid.Sit = false  
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)  
        end  
    end)  
end  
  
StartAntiSeat()  
  
local MIN_PLAYER_LEVEL = 2300  
local PREDICTION_TIME = 0.25  
local YOffset = 1  
local FruitAttackRange = 100  
local ATTACK_DURATION = 20  
local LOW_HEALTH_THRESHOLD = 5000  
local SAFE_HEALTH_THRESHOLD = 9000  
local ESCAPE_HEIGHT = 273861  
local InstaTpConnection = nil  
local SelectedPlayer = nil  
local CurrentTargetPlayer = nil  
local FruitAttackEnabled = false  
local FruitAttackConnection = nil  
  
local SessionStartTime = tick()  
local InitialBounty = 0  
local TotalKills = 0  
local EliminatedPlayers = {}  
local TargetedPlayers = {}  
  
local V_KEY_DELAY = 1 -- dragon shi  
  
-- Forward declarations  
local StartInstaTeleport  
local StartFruitAttack  
local StopFruitAttack  
  
local function PerformHealthEscape()  
    print("[HEALTH ESCAPE] Starting health escape sequence")  
      
    -- Disconnect InstaTp first  
    if InstaTpConnection then  
        InstaTpConnection:Disconnect()  
        InstaTpConnection = nil  
        print("[HEALTH ESCAPE] Disconnected InstaTp")  
    end  
      
    -- Stop attacking  
    local wasAttacking = FruitAttackEnabled  
    local previousTarget = CurrentTargetPlayer  
    StopFruitAttack()  
    print("[HEALTH ESCAPE] Stopped fruit attack, was attacking:", wasAttacking)  
      
    -- Create escape flag  
    local escapeActive = true  
      
    -- Levitation loop  
    local escapeThread = task.spawn(function()  
        while escapeActive do  
            pcall(function()  
                local char = lp.Character  
                if char then  
                    local hrp = char:FindFirstChild("HumanoidRootPart")  
                    if hrp then  
                        local pos = hrp.Position  
                        hrp.CFrame = CFrame.new(pos.X, pos.Y + ESCAPE_HEIGHT, pos.Z)  
                    end  
                end  
            end)  
            task.wait(0.05)  
        end  
        print("[HEALTH ESCAPE] Escape levitation stopped")  
    end)  
      
    -- Wait for health to recover to 9000+  
    print("[HEALTH ESCAPE] Waiting for health to reach 9000+...")  
    while true do  
        task.wait(0.5)  
          
        local char = lp.Character  
        if char then  
            local humanoid = char:FindFirstChild("Humanoid")  
            if humanoid and humanoid.Health >= SAFE_HEALTH_THRESHOLD then  
                print("[HEALTH ESCAPE] Health recovered to:", humanoid.Health)  
                break  
            else  
                if humanoid then  
                    print("[HEALTH ESCAPE] Current health:", humanoid.Health)  
                end  
            end  
        end  
    end  
  
    -- Stop the escape levitation  
    escapeActive = false  
    task.wait(0.2)  
      
    print("[HEALTH ESCAPE] Health escape complete, resuming normal operation")  
  
    -- Resume normal operation  
    StartInstaTeleport()  
    print("[HEALTH ESCAPE] InstaTeleport restarted")  
      
    -- Resume attacking if we were attacking before  
    if wasAttacking and previousTarget and previousTarget.Parent then  
        CurrentTargetPlayer = previousTarget  
        SelectedPlayer = previousTarget.Name  
        FruitAttackEnabled = true  
        StartFruitAttack(previousTarget)  
        print("[HEALTH ESCAPE] Resumed attacking:", previousTarget.Name)  
    end  
end  
  
local function IsHealthLow()  
    local char = lp.Character  
    if not char then return false end  
      
    local humanoid = char:FindFirstChild("Humanoid")  
    if not humanoid then return false end  
      
    return humanoid.Health <= LOW_HEALTH_THRESHOLD  
end  
  
local function GetPlayerLevel(player)  
    local data = player:FindFirstChild("Data")  
    if data then  
        local level = data:FindFirstChild("Level")  
        if level and level.Value then  
            return tonumber(level.Value)  
        end  
    end  
    return 0  
end  
  
local function IsPlayerInSafeZone(player)  
    if not player.Character then return false end  
      
    local hrp = player.Character:FindFirstChild("HumanoidRootPart")  
    if not hrp then return false end  
      
    local inCombat = player.Character:GetAttribute("InCombat")  
    if inCombat == "0" or inCombat == "1" then  
        return false  
    end  
      
    local SafeZonesFolder = workspace._WorldOrigin:FindFirstChild("SafeZones")  
    if not SafeZonesFolder then  
        return false  
    end  
      
    local function getSafeZoneRadius(zone)  
        local mesh = zone:FindFirstChild("Mesh")  
        if mesh and mesh:IsA("SpecialMesh") then  
            local realDiameter = zone.Size.X * mesh.Scale.X  
            return realDiameter / 2  
        end  
        return nil  
    end  
      
    for _, zone in pairs(SafeZonesFolder:GetChildren()) do  
        local radius = getSafeZoneRadius(zone)  
        if radius then  
            local dist = (zone.Position - hrp.Position).Magnitude  
            if dist <= radius then  
                return true  
            end  
        end  
    end  
      
    return false  
end  
  
local function PvpEnable()  
    pcall(function()  
        local args = {"EnablePvp"}  
        game:GetService("ReplicatedStorage"):WaitForChild("Remotes"):WaitForChild("CommF_"):InvokeServer(unpack(args))  
    end)  
end  
  
local function IsPlayerValid(player)  
    if player == lp then return false end  
      
    if SELECTED_TEAM == "Marines" then  
        local team = player.Team  
        if team then  
            local teamName = string.lower(team.Name)  
            if teamName == "marines" or teamName == "marine" then  
                return false  
            end  
        end  
    elseif SELECTED_TEAM == "Pirates" then  
    end  
      
    local pvpDisabled = player:GetAttribute("PvpDisabled")  
    if pvpDisabled == true then return false end  
  
    local raiding = player:GetAttribute("IslandRaiding")  
    if raiding == true then return false end  
      
    local level = GetPlayerLevel(player)  
    if level < MIN_PLAYER_LEVEL then return false end  
      
    if IsPlayerInSafeZone(player) then return false end  
      
    return true  
end  
  
function GetCurrentBounty()  
    local leaderstats = lp:FindFirstChild("leaderstats")  
    if leaderstats then  
        local bounty = leaderstats:FindFirstChild("Bounty/Honor")  
        if bounty then  
            return tonumber(bounty.Value) or 0  
        end  
    end  
    return 0  
end  
  
local function IsInCombat()  
    local playerGui = lp:FindFirstChild("PlayerGui")  
    if not playerGui then return false end  
  
    local mainGui = playerGui:FindFirstChild("Main")  
    if not mainGui then return false end  
  
    local bottomHUD = mainGui:FindFirstChild("BottomHUDList")  
    if not bottomHUD then return false end  
  
    local inCombatUI = bottomHUD:FindFirstChild("InCombat")  
    if not inCombatUI then return false end  
  
    if not inCombatUI.Visible then return false end  
      
    if inCombatUI:IsA("TextLabel") and inCombatUI.Text and string.find(inCombatUI.Text, "risk") then  
        return true  
    end  
      
    return false  
end  
  
local function FakeIsInCombat()  
    local playerGui = lp:FindFirstChild("PlayerGui")  
    if not playerGui then return false end  
  
    local mainGui = playerGui:FindFirstChild("Main")  
    if not mainGui then return false end  
  
    local bottomHUD = mainGui:FindFirstChild("BottomHUDList")  
    if not bottomHUD then return false end  
  
    local inCombatUI = bottomHUD:FindFirstChild("InCombat")  
    if not inCombatUI then return false end  
  
    return inCombatUI.Visible == true  
end  
  
local function WaitForCombatEnd(timeout)  
    timeout = timeout or 30  
    local startTime = tick()  
      
    while IsInCombat() and (tick() - startTime) < timeout do  
        task.wait(1)  
    end  
      
    if IsInCombat() then  
        return false  
    end  
      
    return true  
end  
  
local function BusoKen()  
    pcall(function()  
        local args = {"Ken", true}  
        ReplicatedStorage:WaitForChild("Remotes"):WaitForChild("CommE"):FireServer(unpack(args))  
          
        local char = lp.Character  
        if char then  
            local hasBuso = char:FindFirstChild("HasBuso")  
            if not hasBuso then  
                local args2 = {"Buso"}  
                ReplicatedStorage:WaitForChild("Remotes"):WaitForChild("CommF_"):InvokeServer(unpack(args2))  
            end  
        end  
    end)  
end  
  
local function CheckDragonRage()  
    if SELECTED_FRUIT ~= "Dragon" then return end  
      
    pcall(function()  
        local char = lp.Character  
        if not char then return end  
  
        local dragonHybrid = char:FindFirstChild("DragonHybrid")  
        if dragonHybrid then  
            return  
        end  
          
        local rage = char:FindFirstChild("Rage")  
        if not rage or not rage:IsA("NumberValue") then return end  
          
        local rageValue = rage.Value  
          
        if rageValue > 50 and rageValue < 60 then  
            VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.V, false, game)  
            task.wait(0.05)  
            VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.V, false, game)  
        end  
    end)  
end  
  
task.spawn(function()  
    while not getgenv().ShuttingDown do  
        CheckDragonRage()  
        task.wait(V_KEY_DELAY)  
    end  
end)  
  
task.spawn(function()  
    while not getgenv().ShuttingDown do  
        pcall(function()  
            BusoKen()  
        end)  
        task.wait(5)  
    end  
end)  
  
local function ServerHop()  
  
    getgenv().IsServerHopping = true  
    getgenv().ShuttingDown = true  
  
    if InstaTpConnection then  
        InstaTpConnection:Disconnect()  
        InstaTpConnection = nil  
    end  
  
    if FakeIsInCombat() then  
          
        local flag = Instance.new("BoolValue")  
        flag.Name = "CombatEscape"  
        flag.Parent = workspace  
          
        task.spawn(function()  
            while flag.Parent do  
                local char = lp.Character  
                if char then  
                    local hrp = char:FindFirstChild("HumanoidRootPart")  
                    if hrp then  
                        local pos = hrp.Position  
                        hrp.CFrame = CFrame.new(pos.X, pos.Y + 273861, pos.Z)  
                    end  
                end  
                task.wait(0.05)  
            end  
        end)  
          
        WaitForCombatEnd(30)  
          
        flag:Destroy()  
    end  
  
    task.wait(1)  
  
    local PlayerGui = lp.PlayerGui  
  
    if not PlayerGui:FindFirstChild("ServerBrowser") then  
        getgenv().IsServerHopping = false  
        return  
    end  
  
    PlayerGui.ServerBrowser.Enabled = true  
    task.wait(0.1)  
  
    local Filters = PlayerGui.ServerBrowser.Frame:FindFirstChild("Filters")  
    local SearchRegion = Filters and Filters:FindFirstChild("SearchRegion")  
    local TextBox = SearchRegion and SearchRegion:FindFirstChild("TextBox")  
  
    if not TextBox then  
        getgenv().IsServerHopping = false  
        return  
    end  
  
    TextBox.Text = Selected_Region  
    task.wait(1.4)  
  
    local ScrollingFrame = PlayerGui.ServerBrowser.Frame.ScrollingFrame  
    local FakeScroll = PlayerGui.ServerBrowser.Frame.FakeScroll  
    local Inside = FakeScroll.Inside  
      
    ScrollingFrame.CanvasPosition = Vector2.new(0, math.random(100, 10000))  
    task.wait(0.1)  
  
    local currentJobId = game.JobId  
  
    while true do  
        task.wait(0.5)  
  
        for _, template in ipairs(Inside:GetChildren()) do  
            if template.Name == "Template" then  
                local joinButton = template:FindFirstChild("Join")  
                if joinButton then  
                    local job = joinButton:GetAttribute("Job")  
                    if job and tostring(job):find("-", 1, true) then  
                        job = tostring(job)  
                          
                        if job == currentJobId then  
                        else  
                            local success, err = pcall(function()  
                                TeleportService:TeleportToPlaceInstance(game.PlaceId, job)  
                            end)  
                            if not success then  
                                local success2, err2 = pcall(function()  
                                    TeleportService:TeleportToServer(job)  
                                end)  
                                if not success2 then  
                                end  
                            end  
                            task.wait(5)  
                        end  
                    end  
                end  
            end  
        end  
    end  
end  
  
local FruitConfigs = {  
    ["Dragon"] = {  
        ToolName = "Dragon-Dragon",  
        RemoteName = "LeftClickRemote",  
        Args = function(direction)  
            return {  
                Vector3.new(direction.X, direction.Y, direction.Z),  
                1  
            }  
        end  
    },  
    ["T-Rex"] = {  
        ToolName = "T-Rex-T-Rex",  
        RemoteName = "LeftClickRemote",  
        Args = function(direction)  
            return {  
                Vector3.new(direction.X, direction.Y, direction.Z),  
                3  
            }  
        end  
    },  
    ["Empyrean"] = {  
        ToolName = "Empyrean (Kitsune)-Empyrean (Kitsune)",  
        RemoteName = "LeftClickRemote",  
        Args = function(direction)  
            return {  
                Vector3.new(direction.X, direction.Y, direction.Z),  
                1  
            }  
        end  
    },  
        ["Kitsune"] = {  
        ToolName = "Kitsune-Kitsune",  
        RemoteName = "LeftClickRemote",  
        Args = function(direction)  
            return {  
                Vector3.new(direction.X, direction.Y, direction.Z),  
                1  
            }  
        end  
    },  
        ["Pain"] = {  
        ToolName = "Pain-Pain",  
        RemoteName = "LeftClickRemote",  
        Args = function(direction)  
            return {  
                Vector3.new(direction.X, direction.Y, direction.Z),  
                1  
            }  
        end  
    },  
        ["Control"] = {  
        ToolName = "Control-Control",  
        RemoteName = "LeftClickRemote",  
        Args = function(direction)  
            return {  
                Vector3.new(direction.X, direction.Y, direction.Z),  
                1  
            }  
        end  
    }  
}  
  
local function EquipFruit()  
    local config = FruitConfigs[SELECTED_FRUIT]  
    if not config then   
        return false  
    end  
      
    local character = lp.Character  
    local backpack = lp.Backpack  
      
    if not character then return false end  
      
    local equippedTool = character:FindFirstChild(config.ToolName)  
    if equippedTool then   
        return true   
    end  
      
    local tool = backpack:FindFirstChild(config.ToolName)  
    if tool then  
        character.Humanoid:EquipTool(tool)  
        task.wait(0.1)  
        return true  
    end  
      
    return false  
end  
  
local function FruitAttackPlayer(targetPlayer)  
    if not targetPlayer or not targetPlayer.Character then return end  
      
    local config = FruitConfigs[SELECTED_FRUIT]  
    if not config then return end  
      
    local myChar = lp.Character  
    if not myChar then return end  
      
    local myHRP = myChar:FindFirstChild("HumanoidRootPart")  
    local targetHRP = targetPlayer.Character:FindFirstChild("HumanoidRootPart")  
      
    if not myHRP or not targetHRP then return end  
      
    local distance = (targetHRP.Position - myHRP.Position).Magnitude  
    if distance > FruitAttackRange then return end  
      
    local direction = (targetHRP.Position - myHRP.Position).Unit  
      
    local tool = myChar:FindFirstChild(config.ToolName)  
    if not tool then  
        EquipFruit()  
        task.wait(0.1)  
        tool = myChar:FindFirstChild(config.ToolName)  
        if not tool then return end  
    end  
      
    local remote = tool:FindFirstChild(config.RemoteName)  
    if not remote then   
        return   
    end  
      
    pcall(function()  
        local args = config.Args(direction)  
        remote:FireServer(unpack(args))  
    end)  
end  
  
function StartFruitAttack(targetPlayer)  
    if FruitAttackConnection then  
        task.cancel(FruitAttackConnection)  
    end  
      
    FruitAttackConnection = task.spawn(function()  
        while FruitAttackEnabled and targetPlayer do  
            task.wait(0.01)  
              
            local myChar = lp.Character  
            local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")  
              
            if not myHRP then continue end  
              
            if not targetPlayer.Parent or not targetPlayer.Character then  
                break  
            end  
              
            local targetHRP = targetPlayer.Character:FindFirstChild("HumanoidRootPart")  
            local targetHumanoid = targetPlayer.Character:FindFirstChild("Humanoid")  
              
            if targetHRP and targetHumanoid and targetHumanoid.Health > 0 then  
                FruitAttackPlayer(targetPlayer)  
            end  
        end  
    end)  
end  
  
function StopFruitAttack()  
    FruitAttackEnabled = false  
    if FruitAttackConnection then  
        task.cancel(FruitAttackConnection)  
        FruitAttackConnection = nil  
    end  
end  
  
function StartInstaTeleport()  
    if InstaTpConnection then  
        InstaTpConnection:Disconnect()  
    end  
      
    InstaTpConnection = RunService.Stepped:Connect(function()  
        if not SelectedPlayer then return end  
          
        pcall(function()  
            local char = lp.Character  
            local target = Players:FindFirstChild(SelectedPlayer)  
              
            if char and target and target.Character then  
                local hrp = char:FindFirstChild("HumanoidRootPart")  
                local targetHRP = target.Character:FindFirstChild("HumanoidRootPart")  
                  
                if hrp and targetHRP then  
                    local predictedPos = targetHRP.Position + (targetHRP.AssemblyLinearVelocity * PREDICTION_TIME)  
                    hrp.CFrame = CFrame.new(predictedPos) * CFrame.new(0, YOffset, 0)  
                end  
            end  
        end)  
    end)  
      
end  
  
local function OnCharacterDeath()  
    print("[DEBUG] OnCharacterDeath triggered")  
      
    -- Set shutdown flag FIRST  
    getgenv().ShuttingDown = true  
      
    -- Disconnect all connections  
    if InstaTpConnection then  
        InstaTpConnection:Disconnect()  
        InstaTpConnection = nil  
        print("[DEBUG] InstaTp disconnected")  
    end  
      
    if AntiSeatConnection then  
        AntiSeatConnection:Disconnect()  
        AntiSeatConnection = nil  
        print("[DEBUG] AntiSeat disconnected")  
    end  
      
    if AntiSeatConnection2 then  
        AntiSeatConnection2:Disconnect()  
        AntiSeatConnection2 = nil  
        print("[DEBUG] AntiSeat2 disconnected")  
    end  
      
    -- Stop attacking  
    StopFruitAttack()  
    print("[DEBUG] Fruit attack stopped")  
      
    -- Clear targets  
    SelectedPlayer = nil  
    CurrentTargetPlayer = nil  
    FruitAttackEnabled = false  
      
    -- Wait for respawn  
    print("[DEBUG] Waiting for character respawn...")  
    local newChar = lp.Character or lp.CharacterAdded:Wait()  
    print("[DEBUG] New character detected:", newChar.Name)  
      
    -- Wait for essential parts  
    local newHRP = newChar:WaitForChild("HumanoidRootPart", 10)  
    local newHumanoid = newChar:WaitForChild("Humanoid", 10)  
      
    if newHRP and newHumanoid then  
        print("[DEBUG] HRP and Humanoid ready")  
          
        -- Wait a bit for character to fully load  
        task.wait(1)  
          
        -- Re-enable systems  
        getgenv().ShuttingDown = false  
        print("[DEBUG] ShuttingDown set to false")  
          
        -- Restart anti-seat  
        StartAntiSeat()  
        print("[DEBUG] AntiSeat restarted")  
          
        -- Restart InstaTeleport  
        StartInstaTeleport()  
        print("[DEBUG] InstaTeleport restarted")  
          
        -- Hook up the new humanoid's death event  
        newHumanoid.Died:Connect(function()  
            print("[DEBUG] New humanoid died, calling OnCharacterDeath again")  
            OnCharacterDeath()  
        end)  
          
        print("[DEBUG] OnCharacterDeath complete - systems restored")  
    else  
        print("[DEBUG] ERROR: Failed to get HRP or Humanoid")  
    end  
end  
  
-- Connect death handler to current character  
lp.CharacterAdded:Connect(function(character)  
    print("[DEBUG] CharacterAdded event fired for:", character.Name)  
    local humanoid = character:WaitForChild("Humanoid", 10)  
    if humanoid then  
        humanoid.Died:Connect(function()  
            print("[DEBUG] Humanoid.Died event fired")  
            OnCharacterDeath()  
        end)  
        print("[DEBUG] Death handler connected to new character")  
    else  
        print("[DEBUG] ERROR: Could not find Humanoid in new character")  
    end  
end)  
  
-- Also connect to existing character if it exists  
if lp.Character then  
    local humanoid = lp.Character:FindFirstChild("Humanoid")  
    if humanoid then  
        humanoid.Died:Connect(function()  
            print("[DEBUG] Initial humanoid died")  
            OnCharacterDeath()  
        end)  
        print("[DEBUG] Death handler connected to initial character")  
    end  
end  
  
StartInstaTeleport()  
  
if InitialBounty == 0 then  
    InitialBounty = GetCurrentBounty()  
end  
  
task.spawn(function()  
    while task.wait(1) do  
        if FruitAttackEnabled then  
            EquipFruit()  
        end  
    end  
end)  
  
local function GetNextValidTarget()  
    local validPlayers = {}  
      
    for _, player in pairs(Players:GetPlayers()) do  
        if IsPlayerValid(player) then  
            if not table.find(EliminatedPlayers, player.Name) and not table.find(TargetedPlayers, player.Name) then  
                table.insert(validPlayers, player)  
            end  
        end  
    end  
      
    if #validPlayers > 0 then  
        return validPlayers[math.random(1, #validPlayers)]  
    end  
      
    return nil  
end  
  
task.spawn(function()  
    task.wait(3)  
      
    while task.wait(0.5) do  
        -- Check if low health FIRST  
        if IsHealthLow() and CurrentTargetPlayer then  
            print("[DEBUG] Low health detected, performing escape")  
            PerformHealthEscape()  
            task.wait(1)  
        end  
          
        if not CurrentTargetPlayer or not CurrentTargetPlayer.Parent or not CurrentTargetPlayer.Character then  
            local nextTarget = GetNextValidTarget()  
              
            if nextTarget then  
                SelectedPlayer = nextTarget.Name  
                CurrentTargetPlayer = nextTarget  
                  
                if not table.find(TargetedPlayers, nextTarget.Name) then  
                    table.insert(TargetedPlayers, nextTarget.Name)  
                end  
                  
                StopFruitAttack()  
                  
                FruitAttackEnabled = true  
                StartFruitAttack(nextTarget)  
                  
                  
                local attackStartTime = tick()  
                  
                while tick() - attackStartTime < ATTACK_DURATION do  
                    task.wait(0.5)  
                      
                    -- Check health during attack  
                    if IsHealthLow() then  
                        print("[DEBUG] Low health during attack, performing escape")  
                        PerformHealthEscape()  
                        attackStartTime = tick()  
                    end  
                      
                    if not CurrentTargetPlayer or not CurrentTargetPlayer.Parent or not CurrentTargetPlayer.Character then  
                        break  
                    end  
                      
                    local pvpDisabled = CurrentTargetPlayer:GetAttribute("PvpDisabled")  
                    if pvpDisabled == true then  
                        CurrentTargetPlayer = nil  
                        break  
                    end  
                      
                    if IsPlayerInSafeZone(CurrentTargetPlayer) then  
                        CurrentTargetPlayer = nil  
                        break  
                    end  
                      
                    local humanoid = CurrentTargetPlayer.Character:FindFirstChild("Humanoid")  
                    if humanoid and humanoid.Health <= 0 then  
                          
                        local bountyGain = GetCurrentBounty() - InitialBounty  
                        TotalKills = TotalKills + 1  
                        table.insert(EliminatedPlayers, CurrentTargetPlayer.Name)  
                          
                        StopFruitAttack()  
                        CurrentTargetPlayer = nil  
                        break  
                    end  
                end  
                  
                if CurrentTargetPlayer and CurrentTargetPlayer.Parent then  
                    CurrentTargetPlayer = nil  
                end  
            else  
                ServerHop()  
            end  
        end  
    end  
end)  
  
local function antimover()  
    local character = Players.LocalPlayer.Character  
    if character and not character:FindFirstChild("AntiMover") then  
        Instance.new("Folder", character).Name = "AntiMover"  
    end  
end  
  
local function v4()  
    local args = {  
        true  
    }  
    game:GetService("Players").LocalPlayer:WaitForChild("Backpack"):WaitForChild("Awakening"):WaitForChild("RemoteFunction"):InvokeServer(unpack(args))  
    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.T, false, game)  
    task.wait(0.05)  
    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.T, false, game)  
end  
  
while wait(1) do  
    PvpEnable()  
    v4()  
    antimover()  
end  
