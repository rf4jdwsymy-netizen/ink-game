-- ライブラリとサービス
local GlobalEnv = getgenv()
local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/deividcomsono/Obsidian/main/Library.lua"))()
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer

-- ウィンドウ作成
local Window = Library:CreateWindow({
    AutoShow = true,
    Title = "イクイクハブ INK GAME",
    Footer ="イクイクハブの制作者:みどる",
    NotifySide = "Right",
    ShowCustomCursor = false
})

-----------------------------------
-- Player Tab (一番上) -----------
-----------------------------------
local PlayerTab = Window:AddTab("Player", "account_circle")

local GameBoostsBox = PlayerTab:AddLeftGroupbox("Game Boosts")
local MovementBox = PlayerTab:AddRightGroupbox("Movement & AntiFling")

GameBoostsBox:AddButton("Unlock Dash", function()
    local Boosts = LocalPlayer:FindFirstChild("Boosts")
    if Boosts then
        local FasterSprint = Boosts:FindFirstChild("Faster Sprint")
        if FasterSprint then
            FasterSprint.Value = 5
        end
    end
end)

GameBoostsBox:AddButton("Equip Phantom Power", function()
    LocalPlayer:SetAttribute("_EquippedPower", "PHANTOM STEP")
end)

GameBoostsBox:AddButton("Parkour Artist", function()
    LocalPlayer:SetAttribute("_EquippedPower", "PARKOUR ARTIST")
end)

local WalkSpeedConnection

MovementBox:AddToggle("WalkSpeedIncrease", {
    Text = "移動速度",
    Default = false,
    Callback = function(State)
        if not State then return end
        
        local Hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if Hum then
            Hum.WalkSpeed = Library.Options.WalkSpeedAmount.Value or 16
        end
        
        WalkSpeedConnection = LocalPlayer.CharacterAdded:Connect(function(Char)
            local NewHum = Char:WaitForChild("Humanoid")
            if NewHum and Library.Toggles.WalkSpeedIncrease.Value then
                NewHum.WalkSpeed = Library.Options.WalkSpeedAmount.Value or 16
            end
        end)
        
        task.spawn(function()
            while Library.Toggles.WalkSpeedIncrease.Value do
                local Hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
                if Hum then
                    Hum.WalkSpeed = Library.Options.WalkSpeedAmount.Value or 16
                end
                task.wait(0.4)
            end
        end)
    end
})

MovementBox:AddSlider("WalkSpeedAmount", {
    Min = 1,
    Default = 16,
    Max = 100,
    Text = "WalkSpeed Amount",
    Callback = function(Value)
        local Hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if Hum then
            Hum.WalkSpeed = Value
        end
    end
})

MovementBox:AddButton("Reset WalkSpeed to Normal", function()
    local Hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if Hum then
        Hum.WalkSpeed = 16
    end
    if WalkSpeedConnection then
        WalkSpeedConnection:Disconnect()
        WalkSpeedConnection = nil
    end
end)

MovementBox:AddToggle("NoclipToggle", {
    Text = "Noclip",
    Default = false,
    Callback = function(State)
        GlobalEnv.noclipEnabled = State
        
        if GlobalEnv.noclipConnection then
            GlobalEnv.noclipConnection:Disconnect()
            GlobalEnv.noclipConnection = nil
        end
        
        if State then
            GlobalEnv.noclipConnection = RunService.Stepped:Connect(function()
                if not Library.Toggles.NoclipToggle.Value then return end
                local Char = LocalPlayer.Character
                if Char then
                    for _, Part in pairs(Char:GetDescendants()) do
                        if Part:IsA("BasePart") then
                            Part.CanCollide = false
                        end
                    end
                end
            end)
        end
    end
})

local TeleportEmotesBox = PlayerTab:AddRightGroupbox("Teleport & Emotes")

local PlayerNames = {}
for _, Player in ipairs(Players:GetPlayers()) do
    if Player ~= LocalPlayer then
        table.insert(PlayerNames, Player.Name)
    end
end

local PlayerDropdown = TeleportEmotesBox:AddDropdown("SelectPlayerDropdown", {
    Multi = false,
    Values = PlayerNames,
    Callback = function(Value) end
})

TeleportEmotesBox:AddButton("Teleport to Selected Player", function()
    local SelectedName = Library.Options.SelectPlayerDropdown.Value
    local TargetPlayer = Players:FindFirstChild(SelectedName)
    
    if TargetPlayer and TargetPlayer.Character then
        local TargetRoot = TargetPlayer.Character:FindFirstChild("HumanoidRootPart")
        local MyRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        
        if TargetRoot and MyRoot then
            MyRoot.CFrame = TargetRoot.CFrame + Vector3.new(0, 2, 0)
        end
    end
end)

TeleportEmotesBox:AddButton("Refresh Player List", function()
    local NewNames = {}
    for _, Player in ipairs(Players:GetPlayers()) do
        if Player ~= LocalPlayer then
            table.insert(NewNames, Player.Name)
        end
    end
    PlayerDropdown:Refresh(NewNames)
end)

local EmoteNames = {}
local EmotesFolder = ReplicatedStorage:FindFirstChild("Animations")
if EmotesFolder then
    local EmotesList = EmotesFolder:FindFirstChild("Emotes")
    if EmotesList then
        for _, Emote in ipairs(EmotesList:GetChildren()) do
            if Emote:IsA("Animation") then
                table.insert(EmoteNames, Emote.Name)
            end
        end
    end
end

local EmotesDropdown = TeleportEmotesBox:AddDropdown("SelectEmotesDropdown", {
    Values = EmoteNames,
    AllowNone = true,
    Multi = true,
    Callback = function(Value) end,
    Default = {}
})

local EmoteTracks = {}

TeleportEmotesBox:AddButton("Play Selected Emotes", function()
    local Hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if not Hum then return end
    local Animator = Hum:FindFirstChildOfClass("Animator") or Hum:WaitForChild("Animator")
    
    for _, Track in pairs(EmoteTracks) do
        Track:Stop()
    end
    EmoteTracks = {}

    for _, EmoteName in ipairs(Library.Options.SelectEmotesDropdown.Value) do
        local Animation = EmotesFolder and EmotesFolder:FindFirstChild("Emotes") and EmotesFolder.Emotes:FindFirstChild(EmoteName)
        if Animation then
            local Track = Animator:LoadAnimation(Animation)
            Track:Play()
            table.insert(EmoteTracks, Track)
        end
    end
end)

TeleportEmotesBox:AddButton("Stop Emoting", function()
    for _, Track in pairs(EmoteTracks) do
        Track:Stop()
    end
    EmoteTracks = {}
end)

TeleportEmotesBox:AddToggle("AutoSkip", {
    Text = "Auto Skip",
    Default = false,
    Callback = function(State)
        if not State then return end
        
        local DialogueRemote = ReplicatedStorage:WaitForChild("Remotes"):WaitForChild("DialogueRemote")
        
        task.spawn(function()
            while Library.Toggles.AutoSkip.Value do
                DialogueRemote:FireServer("Skipped")
                task.wait(1.2)
            end
        end)
    end
})

-----------------------------------
-- RedLight Tab -------------------
-----------------------------------
local RedLightTab = Window:AddTab("だるまさんがころんだ", "lightbulb")
local RedLightMainBox = RedLightTab:AddLeftGroupbox("Main Features")
local RedLightUtilitiesBox = RedLightTab:AddRightGroupbox("Utilities")

RedLightMainBox:AddButton("Teleport to End", function()
    local HRP = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if HRP then
        HRP.CFrame = CFrame.new(Vector3.new(-45, 1026, 136.7))
    end
end)

RedLightMainBox:AddButton("Remove Injury", function()
    local Char = LocalPlayer.Character
    local Hum = Char and Char:FindFirstChild("Humanoid")
    local RootPart = Char and Char:FindFirstChild("HumanoidRootPart")
    
    if Hum then
        Hum.PlatformStand = false
        Hum:ChangeState(Enum.HumanoidStateType.GettingUp)
        Hum:SetStateEnabled(Enum.HumanoidStateType.Freefall, true)
        Hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
        Hum:SetStateEnabled(Enum.HumanoidStateType.Jumping, true)
        Hum:SetStateEnabled(Enum.HumanoidStateType.Seated, false)
        Hum:SetStateEnabled(Enum.HumanoidStateType.Swimming, false)
        Hum:SetStateEnabled(Enum.HumanoidStateType.Flying, false)
        Hum:SetStateEnabled(Enum.HumanoidStateType.StrafingNoPhysics, false)
        Hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
    end
    
    if RootPart then
        for _, Constraint in pairs(RootPart:GetChildren()) do
            if Constraint:IsA("BallSocketConstraint") then
                Constraint:Destroy()
            end
        end
    end
    
    if Char then
        for _, Part in pairs(Char:GetChildren()) do
            if Part:IsA("BasePart") and Part:FindFirstChild("BoneCustom") then
                Part.BoneCustom:Destroy()
            end
        end
        
        local tags = {"Ragdoll", "Stun", "RotateDisabled", "RagdollWakeupImmunity"}
        for _, tagName in ipairs(tags) do
            local tag = Char:FindFirstChild(tagName)
            if tag then tag:Destroy() end
        end
        
        local Effects = Workspace:FindFirstChild("Effects")
        local LocalRagdolls = Effects and Effects:FindFirstChild("LocalRagdolls")
        local LocalRagdoll = LocalRagdolls and LocalRagdolls:FindFirstChild(LocalPlayer.Name)
        if LocalRagdoll then LocalRagdoll:Destroy() end
    end
end)

RedLightUtilitiesBox:AddToggle("HelpPlayerLoop", {
    Text = "Help Player LOOP",
    Default = false,
    Callback = function(State)
        if not State then return end
        task.spawn(function()
            while Library.Toggles.HelpPlayerLoop.Value do
                -- ここにHelpPlayerLoop処理を書く（現状未実装）
                task.wait(0.1)
            end
        end)
    end
})

-----------------------------------
-- Dalgona Tab --------------------
-----------------------------------
local DalgonaTab = Window:AddTab("型抜き", "cake")
local DalgonaPlayerBox = DalgonaTab:AddLeftGroupbox("Player Features")
local DalgonaUtilitiesBox = DalgonaTab:AddRightGroupbox("Player Utilities")

DalgonaPlayerBox:AddButton("型抜き即クリア　(修正)", function()
    local DalgonaModule = ReplicatedStorage.Modules.Games.DalgonaClient
    for _, Registry in ipairs(getreg()) do
    end
end)

DalgonaPlayerBox:AddButton("無料ライター", function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/ergergq2/erge.github.io/refs/heads/main/free.lua"))()
end)

DalgonaUtilitiesBox:AddButton("アンチプッシュ", function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/eruiier/antipush.github.io/refs/heads/main/ringta.lua"))()
end)

-----------------------------------
-- Tug Of War Tab -----------------
-----------------------------------
local TugOfWarTab = Window:AddTab("綱引き", "sports_kabaddi")
local TugOfWarFeaturesBox = TugOfWarTab:AddLeftGroupbox("Tug Of War Features")
local TugOfWarRemote = ReplicatedStorage:WaitForChild("Remotes"):WaitForChild("TemporaryReachedBindable")

TugOfWarFeaturesBox:AddToggle("TugOfWarAuto", {
    Text = "オート綱引き",
    Default = false,
    Callback = function(State)
        if not State then return end
        task.spawn(function()
            while Library.Toggles.TugOfWarAuto.Value do
                TugOfWarRemote:FireServer({ IHateYou = true })
                task.wait(0.025)
            end
        end)
    end
})

-----------------------------------
-- Hide and Seek Tab --------------
-----------------------------------
local HideAndSeekTab = Window:AddTab("かくれんぼ", "code")

local HideSeekFeaturesBox = HideAndSeekTab:AddLeftGroupbox("Hide And Seek Features")
local HideSeekUtilitiesBox = HideAndSeekTab:AddRightGroupbox("Hide And Seek Utilities")

HideSeekFeaturesBox:AddToggle("SmartKillHidersToggle", {
    Text = "KILL HIDERS (修正)",
    Default = false,
    Callback = function(State)
        if not State then return end
        task.spawn(function()
            while Library.Toggles.SmartKillHidersToggle.Value do
                local KillingParts = workspace:FindFirstChild("HideAndSeekMap"):FindFirstChild("KillingParts")
                if KillingParts then
                    KillingParts:Destroy()
                end
                task.wait(2)
            end
        end)
    end
})

HideSeekUtilitiesBox:AddButton("Teleport 100 Blocks Up", function()
    local RootPart = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    local Position = RootPart.Position
    RootPart.CFrame = CFrame.new(Vector3.new(Position.X, Position.Y + 100, Position.Z))
end)

HideSeekUtilitiesBox:AddButton("Teleport 40 Blocks Down", function()
    local RootPart = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    local Position = RootPart.Position
    RootPart.CFrame = CFrame.new(Position.X, Position.Y - 40, Position.Z)
end)

HideSeekUtilitiesBox:AddButton("Delete the spikes", function()
    workspace:FindFirstChild("HideAndSeekMap"):WaitForChild("KillingParts"):Destroy()
end)

local HideSeekESPBox = HideAndSeekTab:AddLeftGroupbox("Hide And Seek ESP")
local ExitDoorESPBox = HideAndSeekTab:AddRightGroupbox("Exit Door ESP")

local SeekerColor = Color3.new(1, 0, 0)
local HiderColor = Color3.new(0, 0.5, 1)

local function CreatePlayerESP(TargetPlayer, IsSeeker)
    if TargetPlayer == LocalPlayer then return end
    if not TargetPlayer.Character then return end
    
    local Char = TargetPlayer.Character
    local Head = Char:FindFirstChild("Head")
    if not Head then return end
    
    local ExistingESP = Head:FindFirstChild("RoleESP")
    if ExistingESP then ExistingESP:Destroy() end
    
    for _, Part in ipairs(Char:GetChildren()) do
        if Part:IsA("BasePart") and Part:FindFirstChild("ESPBox") then
            Part.ESPBox:Destroy()
        end
    end
    
    local Color = IsSeeker and SeekerColor or HiderColor
    local RoleText = IsSeeker and "Seeker" or "Hider"
    
    local Billboard = Instance.new("BillboardGui")
    Billboard.Name = "RoleESP"
    Billboard.Adornee = Head
    Billboard.Size = UDim2.new(0, 60, 0, 20)
    Billboard.StudsOffset = Vector3.new(0, 1, 0)
    Billboard.AlwaysOnTop = true
    
    local Frame = Instance.new("Frame")
    Frame.Size = UDim2.new(1, 0, 1, 0)
    Frame.BackgroundColor3 = Color
    Frame.BackgroundTransparency = 0.3
    Frame.BorderSizePixel = 0
    Frame.Parent = Billboard
    
    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(1, 0, 1, 0)
    Label.BackgroundTransparency = 1
    Label.Text = RoleText
    Label.TextColor3 = Color
    Label.TextStrokeTransparency = 0.5
    Label.Font = Enum.Font.SourceSansBold
    Label.TextScaled = true
    Label.Parent = Billboard
    
    Billboard.Parent = Head
    
    for _, Part in ipairs(Char:GetChildren()) do
        if Part:IsA("BasePart") then
            local Box = Instance.new("BoxHandleAdornment")
            Box.Name = "ESPBox"
            Box.Size = Part.Size
            Box.Adornee = Part
            Box.AlwaysOnTop = true
            Box.ZIndex = 10
            Box.Transparency = 0.5
            Box.Color3 = Color
            Box.Parent = Part
        end
    end
end

HideSeekESPBox:AddToggle("EspHiderSeeker", {
    Text = "Esp Hider & Seeker",
    Default = false,
    Callback = function(State)
        if not State then return end
        
        for _, Player in ipairs(Players:GetPlayers()) do
            if Player ~= LocalPlayer and Player.Character then
                local HasKnife = (Player.Backpack:FindFirstChild("Knife") or Player.Character:FindFirstChild("Knife"))
                CreatePlayerESP(Player, HasKnife)
            end
            
            Player.CharacterAdded:Connect(function()
                task.wait(0.1)
                if Library.Toggles.EspHiderSeeker.Value then
                    local HasKnife = (Player.Backpack:FindFirstChild("Knife") or Player.Character:FindFirstChild("Knife"))
                    CreatePlayerESP(Player, HasKnife)
                end
            end)
        end
        
        Players.PlayerAdded:Connect(function(Player)
            Player.CharacterAdded:Connect(function()
                task.wait(0.1)
                if Library.Toggles.EspHiderSeeker.Value then
                    local HasKnife = (Player.Backpack:FindFirstChild("Knife") or Player.Character:FindFirstChild("Knife"))
                    CreatePlayerESP(Player, HasKnife)
                end
            end)
        end)
    end
})

local ExitDoorFillColor = Color3.fromRGB(0, 128, 255)
local ExitDoorOutlineColor = Color3.fromRGB(0, 128, 255)

local function CreateExitDoorESP()
    local HideAndSeekMap = workspace:FindFirstChild("HideAndSeekMap")
    if not HideAndSeekMap then return end
    
    local FixedDoors = HideAndSeekMap:FindFirstChild("NEWFIXEDDOORS")
    if not FixedDoors then return end
    
    for _, Floor in ipairs(FixedDoors:GetChildren()) do
        if string.find(string.lower(Floor.Name), "floor") then
            local ExitDoors = Floor:FindFirstChild("EXITDOORS")
            if ExitDoors then
                for _, Door in ipairs(ExitDoors:GetChildren()) do
                    if string.find(string.lower(Door.Name), "exitdoor") then
                        local BasePart = Door:FindFirstChildOfClass("BasePart")
                        if BasePart then
                            local Highlight = Instance.new("Highlight")
                            Highlight.Parent = BasePart
                            Highlight.FillColor = ExitDoorFillColor
                            Highlight.OutlineColor = ExitDoorOutlineColor
                            Highlight.FillTransparency = 0.3
                            Highlight.OutlineTransparency = 0
                            Highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                            
                            local Billboard = Instance.new("BillboardGui")
                            Billboard.Adornee = BasePart
                            Billboard.AlwaysOnTop = true
                            Billboard.Size = UDim2.new(0, 100, 0, 25)
                            Billboard.StudsOffset = Vector3.new(0, 2, 0)
                            Billboard.Parent = BasePart
                            
                            local Label = Instance.new("TextLabel")
                            Label.BackgroundTransparency = 1
                            Label.Font = Enum.Font.Oswald
                            Label.Size = UDim2.new(1, 0, 1, 0)
                            Label.Text = "Exit Door"
                            Label.TextColor3 = ExitDoorFillColor
                            Label.TextSize = 22
                            Label.TextStrokeColor3 = Color3.new(0, 0, 0)
                            Label.TextStrokeTransparency = 0.75
                            Label.Parent = Billboard
                        end
                    end
                end
            end
        end
    end
end

ExitDoorESPBox:AddToggle("ExitDoorESP", {
    Text = "ESP Exit Doors",
    Default = false,
    Callback = function(State)
        if not State then return end
        CreateExitDoorESP()
        
        GlobalEnv.exitDoorESPConnectionAdded = workspace.DescendantAdded:Connect(function(Descendant)
            if Descendant:IsA("Model") and string.find(string.lower(Descendant.Name), "exitdoor") then
                task.wait(0.5)
                CreateExitDoorESP()
            end
        end)
        
        GlobalEnv.exitDoorESPConnectionRemoved = workspace.DescendantRemoving:Connect(function(Descendant)
            if Descendant:IsA("Model") and string.find(string.lower(Descendant.Name), "exitdoor") then
                CreateExitDoorESP()
            end
        end)
    end
})

local KeyESPBox = HideAndSeekTab:AddLeftGroupbox("Key ESP & Utility")

KeyESPBox:AddToggle("ESPKeys", {
    Text = "ESP Keys",
    Default = false,
    Callback = function(State)
        if not State then return end
        
        local Effects = Workspace:FindFirstChild("Effects")
        if not Effects then return end
        
        for _, Object in pairs(Effects:GetChildren()) do
            if Object:IsA("Model") and Object.PrimaryPart and Object.Name:find("DroppedKey") then
                local Highlight = Instance.new("Highlight")
                Highlight.Name = "KeyESP"
                Highlight.Adornee = Object
                Highlight.FillColor = Color3.fromRGB(255, 255, 0)
                Highlight.OutlineColor = Color3.fromRGB(255, 215, 0)
                Highlight.FillTransparency = 0.3
                Highlight.OutlineTransparency = 0
                Highlight.Parent = Object
                
                local Billboard = Instance.new("BillboardGui")
                Billboard.Adornee = Object.PrimaryPart
                Billboard.Size = UDim2.new(0, 60, 0, 20)
                Billboard.StudsOffset = Vector3.new(0, 2, 0)
                Billboard.AlwaysOnTop = true
                
                local Label = Instance.new("TextLabel")
                Label.Size = UDim2.new(1, 0, 1, 0)
                Label.BackgroundTransparency = 1
                Label.Text = "KEY"
                Label.TextColor3 = Color3.fromRGB(255, 255, 0)
                Label.TextStrokeTransparency = 0.5
                Label.Font = Enum.Font.GothamSemibold
                Label.TextSize = 16
                Label.Parent = Billboard
                
                Billboard.Parent = Object
            end
        end
        
        Workspace.DescendantAdded:Connect(function(Descendant)
            if Descendant:IsA("Model") and Descendant.Parent == Effects then
            end
        end)
    end
})

KeyESPBox:AddButton("Teleport to Random Hider", function()
    for _, Player in ipairs(Players:GetPlayers()) do
        if Player ~= LocalPlayer and Player:FindFirstChild("Backpack") then
            if Player.Backpack:FindFirstChild("DODGE!") then
                local Live = Workspace:FindFirstChild("Live")
                if Live then
                    local TargetChar = Live:FindFirstChild(Player.Name)
                    if TargetChar then
                        local RootPart = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                        local TargetRoot = TargetChar:FindFirstChild("HumanoidRootPart")
                        if RootPart and TargetRoot then
                            RootPart.CFrame = TargetRoot.CFrame + Vector3.new(0, 2, 0)
                            break
                        end
                    end
                end
            end
        end
    end
end)

HideAndSeekTab:AddRightGroupbox("Kill Aura"):AddToggle("KillAuraSafe", {
    Text = "KILL AURA (EXTREMLY SAFE)",
    Default = false,
    Callback = function(State)
        if not State then return end
    end
})

-----------------------------------
-- Jump Rope / Glass Bridge Tab ---
-----------------------------------
local JumpRopeTab = Window:AddTab("大縄/ガラス橋", "sports_handball")
local JumpRopeBox = JumpRopeTab:AddLeftGroupbox("Jump Rope")

JumpRopeBox:AddButton("TP to End Jump Rope", function()
    if LocalPlayer.Character then
        LocalPlayer.Character:PivotTo(CFrame.new(Vector3.new(720.83, 202.7, 921.26)))
    end
end)

JumpRopeBox:AddButton("Delete The Rope", function()
    local Effects = Workspace:FindFirstChild("Effects")
    if Effects and Effects:FindFirstChild("rope") then
        Effects.rope:Destroy()
    end
    
    local SafePlatform = Instance.new("Part")
    SafePlatform.Size = Vector3.new(100, 1, 100)
    SafePlatform.Anchored = true
    SafePlatform.CanCollide = true
    SafePlatform.Position = Vector3.new(672.41, 190.24, 920.59)
    SafePlatform.Material = Enum.Material.SmoothPlastic
    SafePlatform.Color = Color3.fromRGB(120, 120, 120)
    SafePlatform.Parent = Workspace
end)

local GlassBridgeBox = JumpRopeTab:AddRightGroupbox("Glass Bridge Features")

GlassBridgeBox:AddButton("TP to End Glass bridge", function()
    if LocalPlayer.Character then
        LocalPlayer.Character:PivotTo(CFrame.new(Vector3.new(697.19, 118.02, 928.77)))
    end
end)

GlassBridgeBox:AddButton("Remove Glass", function()
    local GlassBridgeMap = Workspace:FindFirstChild("GlassBridgeMap")
    if GlassBridgeMap then
        local Glass = GlassBridgeMap:FindFirstChild("Glass")
        if Glass then
            Glass:Destroy()
        end
    end
end)

-----------------------------------
-- Mingle Tab ---------------------
-----------------------------------
local MingleTab = Window:AddTab("マッチゲーム", "group")

local MingleBox = MingleTab:AddLeftGroupbox("Mingle Utilities")

MingleBox:AddToggle("MingleNoclipToggle", {
    Text = "Mingle Noclip",
    Default = false,
    Callback = function(State)
        if GlobalEnv.MingleNoclipConnection then
            GlobalEnv.MingleNoclipConnection:Disconnect()
            GlobalEnv.MingleNoclipConnection = nil
        end
        if State then
            GlobalEnv.MingleNoclipConnection = RunService.Stepped:Connect(function()
                if not Library.Toggles.MingleNoclipToggle.Value then return end
                local Char = LocalPlayer.Character
                if Char then
                    for _, Part in pairs(Char:GetDescendants()) do
                        if Part:IsA("BasePart") then
                            Part.CanCollide = false
                        end
                    end
                end
            end)
        end
    end
})

local MingleAutoWinBox = MingleTab:AddLeftGroupbox("Auto Win Mingle")
MingleAutoWinBox:AddDivider()
MingleAutoWinBox:AddLabel("Auto Win Mingle")

MingleAutoWinBox:AddToggle("AutoWinMingle", {
    Text = "Auto Win Mingle",
    Default = false,
    Callback = function(State)
        if not State then return end
        task.spawn(function()
            local MingleMap = Workspace:FindFirstChild("MingleMap")
            if not MingleMap then return end
            
            local AllDoors = MingleMap:FindFirstChild("AllMingleDoors")
            if not AllDoors then return end
            
            for _, Door in ipairs(AllDoors:GetChildren()) do
                local HandleOther = Door:FindFirstChild("DoorHandleOtherSide")
                if HandleOther then
                    local ClosePrompt = HandleOther:FindFirstChild("CloseDoorPrompt")
                    if ClosePrompt then
                        local TeleportPart = Door:FindFirstChild("TeleportPart")
                        local RootPart = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                        if TeleportPart and RootPart then
                            RootPart.CFrame = CFrame.new(ClosePrompt.Parent.Position)
                            break
                        end
                    end
                end
            end
        end)
    end
})

MingleAutoWinBox:AddLabel({
    DoesWrap = true,
    Text = "機能するかわかりません"
})

-----------------------------------
-- 天空イカゲーム Tab --------------
-----------------------------------
local FinalTab = Window:AddTab("天空イカゲーム", "sports_esports")

local SkySquidBox = FinalTab:AddLeftGroupbox("Sky Squid Game & Poles")
local FlingBox = FinalTab:AddRightGroupbox("Fling Features")

SkySquidBox:AddToggle("SkySquidGameGodmode", {
    Text = "SKY SQUID GAME 無敵モード",
    Default = false,
    Callback = function(State)
        local ExistingPlatform = Workspace:FindFirstChild("SafePlatform")
        if State then
            if not ExistingPlatform then
                local SafePlatform = Instance.new("Part")
                SafePlatform.Name = "SafePlatform"
                SafePlatform.Size = Vector3.new(450, 5, 450)
                SafePlatform.Anchored = true
                SafePlatform.CanCollide = true
                SafePlatform.Position = Vector3.new(24.65, 958.13, 7.67)
                SafePlatform.Material = Enum.Material.SmoothPlastic
                SafePlatform.Color = Color3.fromRGB(120, 120, 120)
                SafePlatform.Parent = Workspace
            end
        else
            if ExistingPlatform then
                ExistingPlatform:Destroy()
            end
        end
    end
})

SkySquidBox:AddToggle("InstaGrabPoles", {
    Text = "INSTA GRAB Poles",
    Default = false,
    Callback = function(State)
        if State then
            task.spawn(function()
                while Library.Toggles.InstaGrabPoles.Value do
                    local Char = LocalPlayer.Character
                    local SkyMap = Workspace:FindFirstChild("SkySquidGamesMap")
                    if not SkyMap or not Char then
                        task.wait(1)
                    else
                        local PoleWeapons = SkyMap:FindFirstChild("PoleWeapons")
                        if not PoleWeapons then
                            task.wait(1)
                        else
                            local RootPart = Char:FindFirstChild("HumanoidRootPart")
                            if not RootPart then
                                task.wait(1)
                            else
                                for i = 1, 10 do
                                    if not Library.Toggles.InstaGrabPoles.Value then break end
                                    local Pole = PoleWeapons:FindFirstChild("InkPole" .. i)
                                    if Pole then
                                        local Prompt = Pole:FindFirstChildOfClass("ProximityPrompt")
                                        if Prompt then
                                            RootPart.CFrame = Pole.CFrame
                                            fireproximityprompt(Prompt)
                                            task.wait(0.3)
                                        end
                                    end
                                end
                                task.wait(1)
                            end
                        end
                    end
                end
            end)
        end
    end
})

FlingBox:AddDivider()
FlingBox:AddLabel("FLING ALL HAS RISK FOR BAN")

FlingBox:AddButton("Fling All Players", function()
    local Player = LocalPlayer
    local Char = Player.Character
    
    GlobalEnv.FPDH = Workspace.FallenPartsDestroyHeight
    GlobalEnv.FlingAllRunning = true
    
    task.spawn(function()
        while GlobalEnv.FlingAllRunning do
            for _, Target in ipairs(Players:GetPlayers()) do
                if Target ~= Player and Target.Character then
                    local TargetChar = Target.Character
                    local TargetRoot = TargetChar:FindFirstChild("HumanoidRootPart")
                    local MyRoot = Char and Char:FindFirstChild("HumanoidRootPart")
                    if TargetRoot and MyRoot then
                        MyRoot.CFrame = TargetRoot.CFrame + Vector3.new(0, 5, 0)
                        task.wait(0.1)
                    end
                end
            end
            task.wait(0.1)
        end
    end)
end)

FlingBox:AddButton("End Fling All Players Early", function()
    GlobalEnv.FlingAllRunning = false
end)

FlingBox:AddDivider()

-----------------------------------
-- 反乱 Tab -----------------------
-----------------------------------
local RebelTab = Window:AddTab("反乱", "military_tech")

local NPCKillBox = RebelTab:AddLeftGroupbox("NPC Kill & Aimbot")

NPCKillBox:AddToggle("AutoKillNPCGuards", {
    Text = "Auto Kill NPC Guards",
    Default = false,
    Callback = function(State)
        if not State then return end
        task.spawn(function()
            while Library.Toggles.AutoKillNPCGuards.Value do
                for _, Object in ipairs(Workspace:GetDescendants()) do
                    if Object:IsA("Model") and not Players:GetPlayerFromCharacter(Object) then
                        local TypeOfGuard = Object:FindFirstChild("TypeOfGuard")
                        if TypeOfGuard then
                            -- NPCに攻撃処理をここに実装する（例：ヒットする）
                            local Hum = Object:FindFirstChildOfClass("Humanoid")
                            if Hum and Hum.Health > 0 then
                                Hum.Health = 0 -- 即死
                            end
                        end
                    end
                end
                task.wait(0.5)
            end
        end)
    end
})

NPCKillBox:AddToggle("RebelAimbot", {
    Text = "Rebel Aimbot",
    Default = false,
    Callback = function(State)
        if not State then return end
        
        if GlobalEnv.RebelAimbotConnection then
            GlobalEnv.RebelAimbotConnection:Disconnect()
            GlobalEnv.RebelAimbotConnection = nil
        end
        
        GlobalEnv.RebelAimbotConnection = RunService.RenderStepped:Connect(function()
            if not Library.Toggles.RebelAimbot.Value then return end
            
            local Char = LocalPlayer.Character
            if not Char then return end
            
            local Live = Workspace:FindFirstChild("Live")
            if Live then
                local ClosestGuard = nil
                local ClosestDistance = math.huge
                local MyRoot = Char:FindFirstChild("HumanoidRootPart")
                if not MyRoot then return end
                
                for _, Guard in ipairs(Live:GetChildren()) do
                    if Guard:IsA("Model") and Guard:FindFirstChild("TypeOfGuard") and not Guard:FindFirstChild("Dead") then
                        local GuardRoot = Guard:FindFirstChild("HumanoidRootPart")
                        if GuardRoot then
                            local Dist = (GuardRoot.Position - MyRoot.Position).Magnitude
                            if Dist < ClosestDistance then
                                ClosestDistance = Dist
                                ClosestGuard = Guard
                            end
                        end
                    end
                end
                
                if ClosestGuard then
                    local Hum = ClosestGuard:FindFirstChildOfClass("Humanoid")
                    if Hum and Hum.Health > 0 then
                        -- 狙いを付ける処理（例えば向き変更など）
                        local MyHum = Char:FindFirstChildOfClass("Humanoid")
                        if MyHum then
                            local TargetRoot = ClosestGuard:FindFirstChild("HumanoidRootPart")
                            if TargetRoot then
                                -- 簡単なエイム処理例
                                MyHum.AutoRotate = false
                                Char.HumanoidRootPart.CFrame = CFrame.new(Char.HumanoidRootPart.Position, TargetRoot.Position)
                            end
                        end
                    end
                end
            end
        end)
    end
})

local HitboxBox = RebelTab:AddRightGroupbox("Expand Hitbox")
HitboxBox:AddLabel("BE CAREFUL USING THIS\nCOULD RISK IN BAN")

HitboxBox:AddToggle("ExpandRebelHitbox", {
    Text = "Expand Rebel Hitbox",
    Default = false,
    Callback = function(State)
        if not State then return end
        task.spawn(function()
            while Library.Toggles.ExpandRebelHitbox.Value do
                local Live = Workspace:FindFirstChild("Live")
                if Live then
                    for _, Object in ipairs(Live:GetChildren()) do
                        if Object:IsA("Model") and not Players:FindFirstChild(Object.Name) then
                            local Hum = Object:FindFirstChildOfClass("Humanoid")
                            if Hum then
                                local Head = Object:FindFirstChild("Head")
                                if Head then
                                    Head.Size = Vector3.new(10, 10, 10)
                                    Head.Transparency = 0.5
                                end
                            end
                        end
                    end
                end
                task.wait(3)
            end
        end)
    end
})
