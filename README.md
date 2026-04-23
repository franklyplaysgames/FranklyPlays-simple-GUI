
local PlayersService = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local TeleportService = game:GetService("TeleportService")
local Stats = game:GetService("Stats")

local LocalPlayer = PlayersService.LocalPlayer
local Character, Humanoid, HumanoidRootPart
local Camera = Workspace.CurrentCamera

local function UpdateCharacter()
    Character = LocalPlayer.Character
    if Character then
        Humanoid = Character:FindFirstChildOfClass("Humanoid")
        HumanoidRootPart = Character:FindFirstChild("HumanoidRootPart")
    end
end
UpdateCharacter()

LocalPlayer.CharacterAdded:Connect(function(newChar)
    Character = newChar
    task.spawn(function()
        Humanoid = newChar:WaitForChild("Humanoid", 5)
        HumanoidRootPart = newChar:WaitForChild("HumanoidRootPart", 5)
    end)
end)

local Toggles = {
    Fly = false,
    Noclip = false,
    InfiniteJump = false,
    LoopWalkSpeed = false,
    LoopJumpPower = false,
    LoopGravity = false,
    LoopVehicleBoost = false,
    VehicleFly = false,
    ESP = false,
    NameAndHealth = false,
    Highlight = false,
    Fullbright = false,
    AntiAFK = false,
    ClickTeleport = false,
    GodMode = false,
    AntiVoid = false,
    Spectating = false
}

local Settings = {
    WalkSpeed = 16,
    JumpPower = 50,
    Gravity = 196.2,
    FlySpeed = 50,
    VehicleBoost = 1,
    VehicleFlySpeed = 50
}

local SpectatingTarget, OriginalCameraSubject = nil, nil
local ESPFolder = Instance.new("Folder")
ESPFolder.Name = "ESPF"
ESPFolder.Parent = game:GetService("CoreGui")

local Library = loadstring(game:HttpGet("https://sirius.menu/rayfield"))()
local Window = Library:CreateWindow({
    Name = "FranklyPlays simple GUI",
    LoadingTitle = "FranklyPlays Simple GUI",
    LoadingSubtitle = "its Univesal if the game is normal",
    Theme = "Ocean",
    ConfigurationSaving = { Enabled = false },
    KeySystem = false
})

local function GetPlayerList()
    local list = {}
    for _, player in PlayersService:GetPlayers() do
        if player ~= LocalPlayer then
            list[#list + 1] = player.Name
        end
    end
    if #list == 0 then
        list[1] = "No Players"
    end
    return list
end

local function StopSpectating()
    Toggles.Spectating = false
    SpectatingTarget = nil
    if OriginalCameraSubject then
        pcall(function()
            Camera.CameraSubject = OriginalCameraSubject
        end)
    elseif Humanoid then
        pcall(function()
            Camera.CameraSubject = Humanoid
        end)
    end
    pcall(function()
        Camera.CameraType = Enum.CameraType.Custom
    end)
end

local function StartSpectating(playerName)
    if playerName == "No Players" then
        return false
    end
    local target = PlayersService:FindFirstChild(playerName)
    if target and target.Character then
        local targetHumanoid = target.Character:FindFirstChildOfClass("Humanoid")
        if targetHumanoid then
            if not Toggles.Spectating then
                OriginalCameraSubject = Camera.CameraSubject
            end
            Toggles.Spectating = true
            SpectatingTarget = target
            pcall(function()
                Camera.CameraSubject = targetHumanoid
                Camera.CameraType = Enum.CameraType.Custom
            end)
            return true
        end
    end
    return false
end

local SpectateTab = Window:CreateTab("Spectate")
SpectateTab:CreateSection("Player Spectate")

local SelectedSpectatePlayer = nil
local SpectateDropdown

SpectateTab:CreateInput({
    Name = "Search Player to Spectate",
    PlaceholderText = "Type username...",
    RemoveTextAfterFocusLost = false,
    Callback = function(Text)
        if Text == "" then return end
        local query = Text:lower()
        local playerList = game:GetService("Players"):GetPlayers()
        
        for _, p in pairs(playerList) do
            if p.Name:lower():find(query) or (p.DisplayName and p.DisplayName:lower():find(query)) then
                SelectedSpectatePlayer = p.Name
                
                if SpectateDropdown then
                    SpectateDropdown:Set({p.Name})
                end
                
                Rayfield:Notify({
                    Title = "Spectate Target Found",
                    Content = "Target set to: " .. p.Name,
                    Duration = 2
                })
                break
            end
        end
    end,
})

SpectateDropdown = SpectateTab:CreateDropdown({
    Name = "Select Player",
    Options = GetPlayerList(),
    CurrentOption = {},
    MultipleOptions = false,
    Flag = "SPD",
    Callback = function(option)
        if option and option[1] then
            SelectedSpectatePlayer = option[1]
        end
    end
})

SpectateTab:CreateButton({
    Name = "Start Spectating",
    Callback = function()
        if SelectedSpectatePlayer and SelectedSpectatePlayer ~= "No Players" and StartSpectating(SelectedSpectatePlayer) then
            Library:Notify({
                Title = "Spectate",
                Content = "Spectating: " .. SelectedSpectatePlayer,
                Duration = 2
            })
        end
    end
})

SpectateTab:CreateButton({
    Name = "Stop Spectating",
    Callback = function()
        StopSpectating()
        Library:Notify({
            Title = "Spectate",
            Content = "Stopped",
            Duration = 2
        })
    end
})

SpectateTab:CreateButton({
    Name = "Refresh Player List",
    Callback = function()
        SpectateDropdown:Refresh(GetPlayerList())
    end
})

SpectateTab:CreateSection("Quick")

SpectateTab:CreateButton({
    Name = "Next Player",
    Callback = function()
        local players = game:GetService("Players"):GetPlayers()
        local currentIndex = 0
        for i, player in players do
            if SpectatingTarget and player == SpectatingTarget then
                currentIndex = i
                break
            end
        end
        for offset = 1, #players do
            local index = ((currentIndex + offset - 1) % #players) + 1
            local target = players[index]
            if target ~= game:GetService("Players").LocalPlayer and target.Character then
                StartSpectating(target.Name)
                SelectedSpectatePlayer = target.Name
                if SpectateDropdown then SpectateDropdown:Set({target.Name}) end
                break
            end
        end
    end
})

SpectateTab:CreateButton({
    Name = "Previous Player",
    Callback = function()
        local players = game:GetService("Players"):GetPlayers()
        local currentIndex = #players + 1
        for i, player in players do
            if SpectatingTarget and player == SpectatingTarget then
                currentIndex = i
                break
            end
        end
        for offset = 1, #players do
            local index = ((currentIndex - offset - 2) % #players) + 1
            local target = players[index]
            if target ~= game:GetService("Players").LocalPlayer and target.Character then
                StartSpectating(target.Name)
                SelectedSpectatePlayer = target.Name
                if SpectateDropdown then SpectateDropdown:Set({target.Name}) end
                break
            end
        end
    end
})

local SpectateStatusLabel = SpectateTab:CreateLabel("Not spectating")

local InfoTab = Window:CreateTab("Player Info")
InfoTab:CreateSection("Select")

local SelectedInfoPlayer = nil
local InfoDropdown

InfoTab:CreateInput({
    Name = "Search Player Info",
    PlaceholderText = "Type username...",
    RemoveTextAfterFocusLost = false,
    Callback = function(Text)
        if Text == "" then return end
        local query = Text:lower()
        local playerList = game:GetService("Players"):GetPlayers()
        
        for _, p in pairs(playerList) do
            if p.Name:lower():find(query) or (p.DisplayName and p.DisplayName:lower():find(query)) then
                SelectedInfoPlayer = p.Name
                
                if InfoDropdown then
                    InfoDropdown:Set({p.Name})
                end
                
                Rayfield:Notify({
                    Title = "Player Selected",
                    Content = "Target set to: " .. p.Name,
                    Duration = 2
                })
                break
            end
        end
    end,
})

InfoDropdown = InfoTab:CreateDropdown({
    Name = "Select Player",
    Options = GetPlayerList(),
    CurrentOption = {},
    MultipleOptions = false,
    Flag = "PID",
    Callback = function(option)
        if option and option[1] then
            SelectedInfoPlayer = option[1]
        end
    end
})

InfoTab:CreateSection("Info")

local InfoLabels = {}
for i = 1, 19 do
    InfoLabels[i] = InfoTab:CreateLabel("---")
end

InfoTab:CreateButton({
    Name = "Refresh Info",
    Callback = function()
        if not SelectedInfoPlayer or SelectedInfoPlayer == "No Players" then
            return
        end
        local target = game:GetService("Players"):FindFirstChild(SelectedInfoPlayer)
        if not target then
            return
        end
        pcall(function()
            InfoLabels[1]:Set("Name: " .. target.Name)
            InfoLabels[2]:Set("Display: " .. target.DisplayName)
            InfoLabels[3]:Set("UserID: " .. target.UserId)
            InfoLabels[4]:Set("Age: " .. target.AccountAge .. " days")
            InfoLabels[5]:Set("Team: " .. (target.Team and target.Team.Name or "None"))
            
            if target.Character then
                local targetHumanoid = target.Character:FindFirstChildOfClass("Humanoid")
                local targetRootPart = target.Character:FindFirstChild("HumanoidRootPart")
                
                if targetHumanoid then
                    InfoLabels[6]:Set("Health: " .. math.floor(targetHumanoid.Health) .. "/" .. math.floor(targetHumanoid.MaxHealth))
                    InfoLabels[7]:Set("Speed: " .. targetHumanoid.WalkSpeed)
                    InfoLabels[8]:Set("Jump: " .. targetHumanoid.JumpPower)
                end
                
                if targetRootPart then
                    local pos = targetRootPart.Position
                    InfoLabels[9]:Set("Pos: " .. math.floor(pos.X) .. "," .. math.floor(pos.Y) .. "," .. math.floor(pos.Z))
                end
                
                local parts, accessories, tools = 0, 0, 0
                for _, child in target.Character:GetChildren() do
                    if child:IsA("BasePart") then
                        parts = parts + 1
                    elseif child:IsA("Accessory") then
                        accessories = accessories + 1
                    elseif child:IsA("Tool") then
                        tools = tools + 1
                    end
                end
                InfoLabels[10]:Set("Parts: " .. parts)
                InfoLabels[11]:Set("Acc: " .. accessories)
                InfoLabels[12]:Set("Tools: " .. tools)
            end
            
            local backpack = target:FindFirstChild("Backpack")
            InfoLabels[13]:Set("Backpack: " .. (backpack and #backpack:GetChildren() or 0))
            
            if target == game:GetService("Players").LocalPlayer then
                local playerGui = game:GetService("Players").LocalPlayer:FindFirstChild("PlayerGui")
                if playerGui then
                    local guiCount, scriptCount = 0, 0
                    for _, descendant in playerGui:GetDescendants() do
                        if descendant:IsA("GuiObject") then
                            guiCount = guiCount + 1
                        elseif descendant:IsA("LocalScript") then
                            scriptCount = scriptCount + 1
                        end
                    end
                    InfoLabels[14]:Set("GUI: " .. guiCount)
                    InfoLabels[15]:Set("Scripts: " .. scriptCount)
                end
                InfoLabels[16]:Set("Ping: " .. math.floor(game:GetService("Players").LocalPlayer:GetNetworkPing() * 1000) .. "ms")
            else
                InfoLabels[14]:Set("GUI: N/A")
                InfoLabels[15]:Set("Scripts: N/A")
                InfoLabels[16]:Set("Ping: N/A")
            end
            
            InfoLabels[17]:Set("Member: " .. (target.AccountAge > 365 and math.floor(target.AccountAge / 365) .. "y" or target.AccountAge .. "d"))
            InfoLabels[18]:Set("Follow: " .. (target.FollowUserId > 0 and target.FollowUserId or "No"))
            InfoLabels[19]:Set("Char: " .. (target.Character and "Yes" or "No"))
        end)
    end
})

InfoTab:CreateButton({
    Name = "Refresh Player List",
    Callback = function()
        InfoDropdown:Refresh(GetPlayerList())
    end
})

InfoTab:CreateButton({
    Name = "Copy UserID",
    Callback = function()
        if SelectedInfoPlayer and SelectedInfoPlayer ~= "No Players" then
            local target = PlayersService:FindFirstChild(SelectedInfoPlayer)
            if target then
                setclipboard(tostring(target.UserId))
            end
        end
    end
})

local HumanoidTab = Window:CreateTab("Humanoid")
HumanoidTab:CreateSection("Speed")

HumanoidTab:CreateSlider({
    Name = "WalkSpeed",
    Range = {0, 1000},
    Increment = 5,
    Suffix = "",
    CurrentValue = 20,
    Flag = "WS",
    Callback = function(value)
        Settings.WalkSpeed = value
        if Humanoid then
            pcall(function()
                Humanoid.WalkSpeed = value
            end)
        end
    end
})

HumanoidTab:CreateToggle({
    Name = "Loop WalkSpeed",
    CurrentValue = false,
    Flag = "LW",
    Callback = function(value)
        Toggles.LoopWalkSpeed = value
    end
})

HumanoidTab:CreateSlider({
    Name = "JumpPower",
    Range = {0, 1000},
    Increment = 5,
    Suffix = "",
    CurrentValue = 50,
    Flag = "JP",
    Callback = function(value)
        Settings.JumpPower = value
        if Humanoid then
            pcall(function()
                Humanoid.JumpPower = value
                Humanoid.UseJumpPower = true
            end)
        end
    end
})

HumanoidTab:CreateToggle({
    Name = "Loop JumpPower",
    CurrentValue = false,
    Flag = "LJ",
    Callback = function(value)
        Toggles.LoopJumpPower = value
    end
})

HumanoidTab:CreateSection("Gravity")

HumanoidTab:CreateSlider({
    Name = "Gravity",
    Range = {0, 1000},
    Increment = 5,
    Suffix = "",
    CurrentValue = 195,
    Flag = "GR",
    Callback = function(value)
        Settings.Gravity = value
        Workspace.Gravity = value
    end
})

HumanoidTab:CreateToggle({
    Name = "Loop Gravity",
    CurrentValue = false,
    Flag = "LG",
    Callback = function(value)
        Toggles.LoopGravity = value
    end
})

HumanoidTab:CreateButton({
    Name = "Reset Gravity",
    Callback = function()
        Settings.Gravity = 196.2
        Workspace.Gravity = 196.2
    end
})

HumanoidTab:CreateSection("Fly")

HumanoidTab:CreateToggle({
    Name = "Fly",
    CurrentValue = false,
    Flag = "FL",
    Callback = function(value)
        Toggles.Fly = value
        if not HumanoidRootPart then
            UpdateCharacter()
        end
        if not HumanoidRootPart then
            return
        end
        if value then
            local bodyVelocity = Instance.new("BodyVelocity")
            bodyVelocity.Name = "FlyV"
            bodyVelocity.MaxForce = Vector3.one * 9e9
            bodyVelocity.Velocity = Vector3.zero
            bodyVelocity.Parent = HumanoidRootPart
            
            local bodyGyro = Instance.new("BodyGyro")
            bodyGyro.Name = "FlyG"
            bodyGyro.MaxTorque = Vector3.one * 9e9
            bodyGyro.P = 9e4
            bodyGyro.Parent = HumanoidRootPart
        else
            local flyV = HumanoidRootPart:FindFirstChild("FlyV")
            local flyG = HumanoidRootPart:FindFirstChild("FlyG")
            if flyV then
                flyV:Destroy()
            end
            if flyG then
                flyG:Destroy()
            end
        end
    end
})

HumanoidTab:CreateSlider({
    Name = "Fly Speed",
    Range = {10, 1000},
    Increment = 5,
    Suffix = "",
    CurrentValue = 50,
    Flag = "FS",
    Callback = function(value)
        Settings.FlySpeed = value
    end
})

HumanoidTab:CreateSection("Noclip")

HumanoidTab:CreateToggle({
    Name = "Noclip",
    CurrentValue = false,
    Flag = "NC",
    Callback = function(value)
        Toggles.Noclip = value
    end
})

HumanoidTab:CreateToggle({
    Name = "Infinite Jump",
    CurrentValue = false,
    Flag = "IJ",
    Callback = function(value)
        Toggles.InfiniteJump = value
    end
})

HumanoidTab:CreateSection("Protection")

HumanoidTab:CreateToggle({
    Name = "God Mode (only in a few games)",
    CurrentValue = false,
    Flag = "GM",
    Callback = function(value)
        Toggles.GodMode = value
    end
})

HumanoidTab:CreateToggle({
    Name = "Anti Void",
    CurrentValue = false,
    Flag = "AV",
    Callback = function(value)
        Toggles.AntiVoid = value
    end
})

HumanoidTab:CreateSection("Quick")

HumanoidTab:CreateButton({
    Name = "Reset (for games that you can't reset)",
    Callback = function()
        if Humanoid then
            Humanoid.Health = 0
        end
    end
})

HumanoidTab:CreateButton({
    Name = "Full Heal (barely works)",
    Callback = function()
        if Humanoid then
            Humanoid.Health = Humanoid.MaxHealth
        end
    end
})

local EmoteTab = Window:CreateTab("Emotes")

local currentTrack = nil
local listEmoteEnabled = false
local selectedID = ""
local emoteSpeed = 1

local char = game.Players.LocalPlayer.Character or game.Players.LocalPlayer.CharacterAdded:Wait()
local hum = char:WaitForChild("Humanoid")
local animator = hum:WaitForChild("Animator")

local emotes = {
    ['Fashion'] = 3333331310; 
    ["Baby Dance"] = 4265725525; 
    ["Cha-Cha"] = 6862001787; 
    ['Monkey'] = 3333499508; 
    ['Shuffle'] = 4349242221; 
    ["Top Rock"] = 3361276673; 
    ["Around Town"] = 3303391864; 
    ["Fancy Feet"] = 3333432454; 
    ["Hype Dance"] = 3695333486; 
    ['Bodybuilder'] = 3333387824; 
    ['Idol'] = 4101966434;
    ['Curtsy'] = 4555816777;
    ['Happy'] = 4841405708;
    ["Quiet Waves"] = 7465981288;
    ['Sleep'] = 4686925579;
    ["Floss Dance"] = 5917459365;
    ['Shy'] = 3337978742;
    ['Godlike'] = 3337994105;
    ["Hero Landing"] = 5104344710;
    ["High Wave"] = 5915690960;
    ['Cower'] = 4940563117;
    ['Bored'] = 5230599789;
    ["Show Dem Wrists -KSI"] = 7198989668;
    ['Celebrate'] = 3338097973;
    ['Dash'] = 582855105;
    ['Beckon'] = 5230598276;
    ['Haha'] = 3337966527;
    ["Lasso Turn - Tai Verdes"] = 7942896991;
    ["Line Dance"] = 4049037604;
    ['Shrug'] = 3334392772;
    ['Point2'] = 3344585679;
    ['Stadium'] = 3338055167;
    ['Confused'] = 4940561610;
    ['Side to Side'] = 3333136415;
    ['Old Town Road Dance - Lil Nas X'] = 5937560570;
    ['Hello'] = 3344650532;
    ['Dolphin Dance'] = 5918726674;
    ['Samba'] = 6869766175;
    ['Break Dance'] = 5915648917;
    ["Hips Poppin' - Zara Larsson"] = 6797888062;
    ['Wake Up Call - KSI'] = 7199000883;
    ['Greatest'] = 3338042785;
    ['On The Outside - Twenty One'] = 7422779536;
    ['Boxing Punch - KSI'] = 7202863182;
    ['Sad'] = 4841407203;
    ['Flowing Breeze'] = 7465946930;
    ['Twirl'] = 3334968680;
    ['Jumping Wave'] = 4940564896;
    ['HOLIDAY Dance - Lil Nas X'] = 5937558680;
    ['Take Me Under - Zara Larsson'] = 6797890377;
    ['Dizzy'] = 3361426436;
    ["Dancing' Shoes - Twenty One"] = 7404878500;
    ['Fashionable'] = 3333331310;
    ['Fast Hands'] = 4265701731;
    ['Tree'] = 4049551434;
    ['Agree'] = 4841397952;
    ['Power Blast'] = 4841403964;
    ['Swoosh'] = 3361481910;
    ['Jumping Cheer'] = 5895324424;
    ['Disagree'] = 4841401869;
    ['Rodeo Dance - Lil Nas X'] = 5918728267;
    ["It Ain't My Fault - Zara Larsson"] = 6797891807;
    ['Rock On'] = 5915714366;
    ['Block Partier'] = 6862022283;
    ['Dorky Dance'] = 4212455378;
    ['Zombie'] = 4210116953;
    ['AOK - Tai Verdes'] = 7942885103;
    ['T-Pose'] = 3338010159;
    ['Cobra Arms - Tai Verdes'] = 7942890105;
    ['Panini Dance - Lil Nas X'] = 5915713518;
    ['Fishing'] = 3334832150;
    ['Robot'] = 3338025566;
    ['Saturday Dance - Twenty One'] = 7422807549;
    ['Keeping Time'] = 4555808220;
    ['Air Dance'] = 4555782893;
    ['Rock Guitar - Royal Blood'] = 6532134724;
    ["Borock's Rage"] = 3236842542;
    ["Ud'zal's Summoning"] = 3303161675;
    ['Y-Pose'] = 4349285876;
    ['Swan Dance'] = 7465997989;
    ['Louder'] = 3338083565;
    ['Up and Down - Twenty One'] = 7422797678;
    ['Swish'] = 3361481910;
    ['Drummer Moves - Twenty One'] = 7422527690;
    ['Sneaky'] = 3334424322;
    ['Heisman Pose'] = 3695263073;
    ['Jacks'] = 3338066331;
    ['Cha-Cha 2'] = 3695322025;
    ['BURBERRY LOLA ATTITUDE - NIMBUS'] = 10147821284;
    ['BURBERRY LOLA ATTITUDE - GEM'] = 10147815602;
    ['BURBERRY LOLA ATTITUDE - HYDRO'] = 10147823318;
    ['BURBERRY LOLA ATTITUDE - BLOOM'] = 10147817997;
    ['Superhero Reveal'] = 3695373233;
    ['Air Guitar'] = 3695300085;
    ['Dismissive Wave'] = 3333272779;
    ['Country Line Dance - Lil Nas X'] = 5915712534;
    ['Salute'] = 3333474484;
    ['Applaud'] = 5915693819;
    ['Get Out'] = 3333272779;
    ['Hwaiting (화이팅)'] = 9527885267;
    ['Annyeong (안녕)'] = 9527883498;
    ['Bunny Hop'] = 4641985101;
    ['Sandwich Dance'] = 4406555273;
    ['Hyperfast 5G'] = 9408617181;
    ['Victory - 24kGoldn'] = 9178377686;
    ['Tantrum'] = 5104341999;
    ['Rock Star - Royal Blood'] = 10714400171;
    ['Drum Solo - Royal Blood'] = 6532839007;
    ['Drum Master - Royal Blood'] = 6531483720;
    ['High Hands'] = 9710985298;
    ['Tilt'] = 3334538554;
    ['Gashina - SUNMI'] = 9527886709;
    ['Chicken Dance'] = 4841399916;
    ["You can't sit with us - Sunmi"] = 9983520970;
    ["Frosty Flair - Tommy Hilfiger"] = 10214311282;
    ["Floor Rock Freeze - Tommy Hilfiger"] = 10214314957;
    ['Boom Boom Clap - George Ezra'] = 10370346995;
    ['Cartwheel - George Ezra'] = 10370351535;
    ['Chill Vibes - George Ezra'] = 10370353969; 
    ['Sidekicks - George Ezra'] = 10370362157;
    ['The Conductor - George Ezra'] = 10370359115;
    ['Super Charge'] = 10478338114;
    ['Swag Walk'] = 10478341260;
    ['Mean Mug - Tommy Hilfiger'] = 10214317325;
    ['V Pose - Tommy Hilfiger'] = 10214319518;
    ['Uprise - Tommy Hilfiger'] = 10275008655;
    ['2 Baddies Dance Move - NCT 127'] = 12259828678; 
    ['Kick It Dance Move - NCT 127'] = 12259826609;
    ['Sticker Dance Move - NCT 127'] = 12259825026;
    ['Elton John - Rock Out'] = 11753474067;
    ['Elton John - Heart Skip'] = 11309255148;
    ['Elton John - Still Standing'] = 11444443576;
    ['Elton John - Elevate'] = 11394033602;
    ['Elton John - Cat Man'] = 11444441914;
    ['Elton John - Piano Jump'] = 11453082181;
    ['Alo Yoga Pose - Triangle'] = 12507084541;
    ['Alo Yoga Pose - Warrior II'] = 12507083048;
    ['Alo Yoga Pose - Lotus Position'] = 12507085924;
    ['TWICE-Moonlight-Sunrise'] = 12714233242;
    ['TWICE-Set-Me-Free-Dance-1'] = 12714228341;
    ['TWICE-Set-Me-Free-Dance-2'] = 12714231087;
    ['Ay-Yo-Dance-Move-NCT-127'] = 12804157977;
    ['TWICE-The-Feels'] = 12874447851;
    ['Rise-Above-The-Chainsmokers'] = 12992262118;
    ['TWICE-What-Is-Love'] = 13327655243;
    ['Man-City-Bicycle-Kick'] = 13421057998;
    ['TWICE-Fancy'] = 13520524517;
    ['TWICE Pop by Nayeon'] = 13768941455;
    ['Tommy - Archer'] = 13823324057;
    ['Man City Backflip'] = 13694100677;
    ['Man-City-Scorpion-Kick'] = 13694096724;
    ['Arm Twist'] = 10713968716;
    ['YUNGBLUD – HIGH KICK'] = 14022936101;
    ['TWICE Like Ooh-Ahh'] = 14123781004;
    ['Baby Queen - Air Guitar'] = 14352335202;
    ['Baby Queen - Dramatic Bow'] = 14352337694;
    ['Baby Queen - Face Frame'] = 14352340648;
    ['Baby Queen - Bouncy Twirl'] = 14352343065;
    ['Baby Queen - Strut'] = 14352362059;
    ['BLACKPINK Pink Venom 1'] = 14548619594;
    ['BLACKPINK Pink Venom 2'] = 14548620495;
    ['BLACKPINK Pink Venom 3'] = 14548621256;
    ['TWICE LIKEY'] = 14899979575;
    ['TWICE Feel Special'] = 14899980745;
    ['BLACKPINK Shut Down 1'] = 14901306096;
    ['BLACKPINK Shut Down 2'] = 14901308987;
    ["Bone Chillin' Bop"] = 15122972413;
    ['Paris Hilton - Sliving'] = 15392759696;
    ['Paris Hilton - Iconic'] = 15392756794;
    ['Paris Hilton - Angles'] = 15392752812;
    ['BLACKPINK JISOO Flower'] = 15439354020;
    ['BLACKPINK JENNIE You and Me'] = 15439356296;
    ['Rock n Roll'] = 15505458452;
    ['Victory Dance'] = 15505456446;
    ['Flex Walk'] = 15505459811;
    ['Olivia Rodrigo Head Bop'] = 15517864808;
    ['Olivia Rodrigo good 4 u'] = 15517862739;
    ['Olivia Rodrigo Fall Back'] = 15549124879;
    ["Nicki Minaj Super Bass"] = 15571446961;
    ['Nicki Minaj Boom Boom'] = 15571448688;
    ['Nicki Minaj Anaconda'] = 15571450952;
    ['Nicki Minaj Starships'] = 15571453761;
    ['Yungblud Happier Jump'] = 15609995579;
    ['Festive Dance'] = 15679621440;
    ['BLACKPINK LISA Money'] = 15679623052;
    ['BLACKPINK ROSÉ On The Ground'] = 15679624464;
    ['Imagine Dragons Bones'] = 15689279687;
    ['GloRilla Tomorrow'] = 15689278184;
    ['d4vd Backflip'] = 15693621070;
    ['ericdoa dance'] = 15698402762;
    ['Cuco Levitate'] = 15698404340;
    ['Mean Girls Dance Break'] = 15963314052;
    ['Paris Hilton Sanasa'] = 16126469463;
    ['BLACKPINK Ice Cream'] = 16181797368;
    ['BLACKPINK Kill This Love'] = 16181798319;
    ['TWICE I GOT YOU 1'] = 16215030041;
    ['TWICE I GOT YOU 2'] = 16256203246;
    ["Dave's Spin Move"] = 16272432203;
    ['Sol de Janeiro Samba'] = 16270690701;
    ['Beauty Touchdown'] = 16302968986;
    ['Skadoosh - Kung Fu Panda'] = 16371217304;
    ['Jawny Stomp'] = 16392075853;
    ['Mae Stephens Piano'] = 16553163212;
    ['BLACKPINK Boombayah'] = 16553164850;
    ['BLACKPINK DDU-DU DDU-DU'] = 16553170471;
    ['HIPMOTION Amaarae'] = 16572740012;
    ['Mae Stephens Arm Wave'] = 16584481352;
    ['Wanna play?'] = 16646423316;
    ['BLACKPINK How You Like That'] = 16874470507;
    ['BLACKPINK Lovesick Girls'] = 16874472321;
    ['Mini Kong'] = 17000021306;
    ["HUGO Let's Drive!"] = 17360699557;
    ['Wisp air guitar'] = 17370775305;
    ['Vans Ollie'] = 18305395285;
    ['Sturdy - Ice Spice'] = 17746180844;
    ['Rolling Stones Strum'] = 18148804340;
    ['Rock Out - Bebe Rexha'] = 18225053113;
    ['SpongeBob Imaginaaation'] = 18443237526;
    ['SpongeBob Dance'] = 18443245017;
    ['Shrek Roar'] = 18524313628;
    ['Team USA Breaking'] = 18526288497;
    ['NBA WNBA Fadeaway'] = 18526362841;
    ['Vroom Vroom'] = 18526397037;
    ['TMNT Dance'] = 18665811005;
    ['Olympic Dismount'] = 18665825805;
    ["BLACKPINK As If It's Your Last"] = 18855536648;
    ["BLACKPINK Don't know what to do"] = 18855531354;
    ['TWICE ABCD by Nayeon'] = 18933706381;
    ['Charli xcx Apple Dance'] = 18946844622;
    ['The Zabb'] = 129470135909814;
    ['Fashion Runway'] = 80995190624232;
    ['ALTEGO Care Less'] = 107875941017127;
    ['Fashion Roadkill'] = 136831243854748;
    ['Skibidi Titan Speakerman'] = 134283166482394;
    ['Chappell Roan HOT TO GO!'] = 85267023718407;
    ['Secret Handshake'] = 71243990877913;
    ['KATSEYE Touch'] = 135876612109535;
    ['Fashion Spin'] = 131669256082047;
    ['TWICE Strategy'] = 97311229290836;
    ['NBA Monster Dunk'] = 132748833449150;
    ['DearALICE Ariana'] = 134318425949290;
    ['The Weeknd Starboy'] = 71105746210464;
    ['The Weeknd Opening Night'] = 133110725387025;
    ['Robot M3GAN'] = 125803725853577; 
    ["M3GAN's Dance"] = 99649534578309;
    ['Rasputin – Boney M.'] = 114872820353992;
    ['Thanos Happy Jump'] = 97611664803614;
    ['Young-hee Head Spin'] = 112011282168475;
    ['TWICE Takedown'] = 140182843839424;
    ['Stray Kids Walkin On Water'] = 125064469983655;
    ['TWICE TAKEDOWN DANCE 2'] = 127104635954695;
}

local function stopEmote()
    if currentTrack then
        currentTrack:Stop(0.1)
        currentTrack:Destroy()
        currentTrack = nil
    end
    if char:FindFirstChild("Animate") then
        char.Animate.Disabled = false
    end
end

local function playEmote(id)
    stopEmote()
    if id == "" or id == nil then return end
    
    local cleanID = tostring(id):gsub("%D", "")
    
    local success, err = pcall(function()
        if char:FindFirstChild("Animate") then char.Animate.Disabled = true end
        
        local anim = Instance.new("Animation")
        anim.AnimationId = "rbxassetid://" .. cleanID
        anim.Parent = animator
        
        currentTrack = animator:LoadAnimation(anim)
        currentTrack.Priority = Enum.AnimationPriority.Action4
        currentTrack.Looped = true
        currentTrack:Play(0.1)
        currentTrack:AdjustSpeed(emoteSpeed)
    end)
    
    if not success then warn("Emote Error: " .. tostring(err)) end
end

EmoteTab:CreateToggle({
    Name = "Enable List Emote",
    CurrentValue = false,
    Callback = function(v)
        listEmoteEnabled = v
        if v then playEmote(selectedID) else stopEmote() end
    end,
})

local EmoteDropdown 

EmoteTab:CreateInput({
    Name = "Quick Search Emote",
    PlaceholderText = "Type name (e.g. 'Skibidi')",
    RemoveTextAfterFocusLost = false,
    Callback = function(Text)
        if Text == "" then return end
        local query = Text:lower()
        for name, id in pairs(emotes) do
            if name:lower():find(query) then
                selectedID = id
                
                if EmoteDropdown then
                    EmoteDropdown:Set({name}) 
                end

                if listEmoteEnabled then playEmote(selectedID) end
                break
            end
        end
    end,
})

local emoteNames = {}
for name, _ in pairs(emotes) do table.insert(emoteNames, name) end
table.sort(emoteNames)

-- Assigning the dropdown to the variable EmoteDropdown
EmoteDropdown = EmoteTab:CreateDropdown({
    Name = "Select Emote from List",
    Options = emoteNames,
    CurrentOption = {"None"},
    Callback = function(opt)
        selectedID = emotes[opt[1]]
        if listEmoteEnabled then playEmote(selectedID) end
    end,
})

EmoteTab:CreateSlider({
    Name = "Emote Speed",
    Range = {0, 5},
    Increment = 0.1,
    Suffix = "x",
    CurrentValue = 1,
    Callback = function(v)
        emoteSpeed = v
        if currentTrack then currentTrack:AdjustSpeed(v) end
    end,
})

EmoteTab:CreateButton({
    Name = "Refresh Character",
    Callback = function()
        char = game.Players.LocalPlayer.Character
        hum = char:FindFirstChildOfClass("Humanoid")
        animator = hum:FindFirstChildOfClass("Animator")
        Rayfield:Notify({Title = "Updated", Content = "Character re-linked.", Duration = 2})
    end,
})

EmoteTab:CreateSection("Custom Animations")

local autoSwitch = false
local currentTrack = nil
local currentState = "None" 
local idleTimer = 0

local char = game.Players.LocalPlayer.Character or game.Players.LocalPlayer.CharacterAdded:Wait()
local hum = char:WaitForChild("Humanoid")
local animator = hum:WaitForChild("Animator")

local ID_IDLE = 110211186840347
local ID_RUN = 118320322718866
local ID_HERO = 114191137265065
local ID_JUMP = 109996626521204
local ID_SWIM = 656121399
local ID_SIT  = 2506281703

local function stopCurrent()
    if currentTrack then
        currentTrack:Stop(0.1)
        currentTrack:Destroy()
        currentTrack = nil
    end
end

local function playNew(id, looped)
    stopCurrent()
    local anim = Instance.new("Animation")
    anim.AnimationId = "rbxassetid://" .. tostring(id)
    
    currentTrack = animator:LoadAnimation(anim)
    currentTrack.Priority = Enum.AnimationPriority.Action4
    currentTrack.Looped = looped
    currentTrack:Play(0.1)
    
    if char:FindFirstChild("Animate") then
        char.Animate.Disabled = true
    end
    return currentTrack
end

EmoteTab:CreateToggle({
    Name = "Enable All Auto-Animations",
    CurrentValue = false,
    Callback = function(v)
        autoSwitch = v
        if not v then
            stopCurrent()
            currentState = "None"
            idleTimer = 0
            if char:FindFirstChild("Animate") then
                char.Animate.Disabled = false
            end
        end
    end,
})

task.spawn(function()
    while task.wait(0.1) do
        if autoSwitch then
            local state = hum:GetState()
            local moving = hum.MoveDirection.Magnitude > 0
            
            if state == Enum.HumanoidStateType.Seated then
                idleTimer = 0
                if currentState ~= "Sitting" then
                    currentState = "Sitting"
                    playNew(ID_SIT, true)
                end

            elseif state == Enum.HumanoidStateType.Swimming then
                idleTimer = 0
                if currentState ~= "Swimming" then
                    currentState = "Swimming"
                    playNew(ID_SWIM, true)
                end

            elseif state == Enum.HumanoidStateType.Jumping or state == Enum.HumanoidStateType.Freefall then
                idleTimer = 0
                if currentState ~= "Jumping" then
                    currentState = "Jumping"
                    playNew(ID_JUMP, false)
                end

            elseif moving then
                idleTimer = 0
                if currentState ~= "Running" then
                    currentState = "Running"
                    playNew(ID_RUN, true)
                end

            else
                if currentState ~= "Idle" and currentState ~= "Hero" then
                    currentState = "Idle"
                    playNew(ID_IDLE, true)
                end
                
                idleTimer = idleTimer + 0.1
                
                if idleTimer >= 20 then
                    idleTimer = 0
                    currentState = "Hero"
                    local heroTrack = playNew(ID_HERO, false)
                    
                    task.spawn(function()
                        heroTrack.Stopped:Wait()
                        if hum.MoveDirection.Magnitude == 0 and autoSwitch and currentState == "Hero" then
                            currentState = "Idle"
                            playNew(ID_IDLE, true)
                        end
                    end)
                end
            end
        end
    end
end)

EmoteTab:CreateButton({
    Name = "Force Reset",
    Callback = function()
        stopCurrent()
        currentState = "None"
        idleTimer = 0
        if char:FindFirstChild("Animate") then
            char.Animate.Disabled = false
        end
        Rayfield:Notify({Title = "Reset", Content = "Animations cleared."})
    end,
})

local TeleportTab = Window:CreateTab("Teleport")
TeleportTab:CreateSection("Player")

local SelectedTeleportPlayer = nil
local TeleportDropdown

TeleportTab:CreateInput({
    Name = "Search Player",
    PlaceholderText = "Type username...",
    RemoveTextAfterFocusLost = false,
    Callback = function(Text)
        if Text == "" then return end
        local query = Text:lower()
        local playerList = game:GetService("Players"):GetPlayers()
        
        for _, p in pairs(playerList) do
            if p.Name:lower():find(query) or (p.DisplayName and p.DisplayName:lower():find(query)) then
                SelectedTeleportPlayer = p.Name
                
                if TeleportDropdown then
                    TeleportDropdown:Set({p.Name})
                end
                
                Rayfield:Notify({
                    Title = "Player Found",
                    Content = "Target set to: " .. p.Name,
                    Duration = 2
                })
                break
            end
        end
    end,
})

TeleportDropdown = TeleportTab:CreateDropdown({
    Name = "Select Player",
    Options = GetPlayerList(),
    CurrentOption = {},
    MultipleOptions = false,
    Flag = "TPD",
    Callback = function(option)
        if option and option[1] then
            SelectedTeleportPlayer = option[1]
        end
    end
})

TeleportTab:CreateButton({
    Name = "Teleport to Player",
    Callback = function()
        if not SelectedTeleportPlayer or SelectedTeleportPlayer == "No Players" or not HumanoidRootPart then
            return
        end
        local target = game:GetService("Players"):FindFirstChild(SelectedTeleportPlayer)
        if target and target.Character then
            local targetRoot = target.Character:FindFirstChild("HumanoidRootPart")
            if targetRoot then
                HumanoidRootPart.CFrame = targetRoot.CFrame * CFrame.new(0, 0, 3)
            end
        end
    end
})

TeleportTab:CreateButton({
    Name = "Refresh Player List",
    Callback = function()
        TeleportDropdown:Refresh(GetPlayerList())
    end
})

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local SavedLocations = {} 
local LocationNames = {}
local CurrentlySelected = nil
local TempName = "Waypoint 1"
local TPList

TeleportTab:CreateSection("Waypoints")

TeleportTab:CreateInput({
   Name = "New Waypoint Name",
   PlaceholderText = "E.g. Base, Shop",
   RemoveTextAfterFocusLost = false,
   Callback = function(Value)
       TempName = Value
   end,
})

TeleportTab:CreateButton({
   Name = "Save My Current Position",
   Callback = function()
       local character = LocalPlayer.Character
       local hrp = character and character:FindFirstChild("HumanoidRootPart")
       
       if hrp then
           SavedLocations[TempName] = hrp.CFrame
           
           local exists = false
           for _, n in pairs(LocationNames) do 
               if n == TempName then exists = true break end 
           end
           
           if not exists then 
               table.insert(LocationNames, TempName) 
           end
           
           if TPList then
               TPList:Refresh(LocationNames)
           end
           
           Rayfield:Notify({
               Title = "Position Saved",
               Content = "Added '" .. TempName .. "' to your list.",
               Duration = 3
           })
       end
   end,
})

TPList = TeleportTab:CreateDropdown({
   Name = "Select Destination",
   Options = {"No Waypoints Saved"},
   CurrentOption = {"No Waypoints Saved"},
   MultipleOptions = false,
   Callback = function(Option)
       CurrentlySelected = (type(Option) == "table" and Option[1]) or Option
   end,
})

TeleportTab:CreateButton({
   Name = "Teleport to Selected",
   Callback = function()
       if CurrentlySelected and SavedLocations[CurrentlySelected] then
           local character = LocalPlayer.Character
           local hrp = character and character:FindFirstChild("HumanoidRootPart")
           
           if hrp then
               hrp.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
               hrp.CFrame = SavedLocations[CurrentlySelected]
           end
       else
           Rayfield:Notify({
               Title = "Selection Error",
               Content = "Please select a waypoint from the list first!",
               Duration = 3
           })
       end
   end,
})

TeleportTab:CreateSection("Click")

TeleportTab:CreateToggle({
    Name = "Click Teleport",
    CurrentValue = false,
    Flag = "CT",
    Callback = function(value)
        Toggles.ClickTeleport = value
    end
})

local Tab = Window:CreateTab("Mirroring")

local targetPlayer = nil
local mirrorEnabled = false
local mirrorDelay = 0
local currentIndex = 1
local offsetMode = "Center"

local offsets = {
    ["Center"] = CFrame.new(0, 0, 0),
    ["Left"]   = CFrame.new(-3, 0, 0),
    ["Right"]  = CFrame.new(3, 0, 0),
    ["Front"]  = CFrame.new(0, 0, -3),
    ["Back"]   = CFrame.new(0, 0, 3),
    ["Up"]     = CFrame.new(0, 5, 0),
    ["Stand"]   = CFrame.new(2, 1, 1)
}

local function getPlayers()
    local pTable = {}
    for _, p in pairs(game.Players:GetPlayers()) do
        if p ~= game.Players.LocalPlayer then table.insert(pTable, p) end
    end
    return pTable
end

local function getNames()
    local nTable = {}
    for _, p in pairs(getPlayers()) do table.insert(nTable, p.Name) end
    return nTable
end

local TargetLabel = Tab:CreateLabel("Current Target: None")

Tab:CreateToggle({
   Name = "Enable Mirroring",
   CurrentValue = false,
   Callback = function(v) 
       mirrorEnabled = v 
       local char = game.Players.LocalPlayer.Character
       if char then
           if not v then
               if char:FindFirstChild("Animate") then char.Animate.Disabled = false end
               local hum = char:FindFirstChildOfClass("Humanoid")
               if hum then for _, t in pairs(hum:GetPlayingAnimationTracks()) do t:Stop() end end
               local root = char:FindFirstChild("HumanoidRootPart")
               if root then root.Velocity = Vector3.new(0,0,0) end
           end
       end
   end,
})

local PlayerDropdown

Tab:CreateInput({
   Name = "Search Player",
   PlaceholderText = "Type username...",
   RemoveTextAfterFocusLost = false,
   Callback = function(Text)
       if Text == "" then return end
       local query = Text:lower()
       local playerList = game.Players:GetPlayers()
       
       for _, p in pairs(playerList) do
           if p.Name:lower():find(query) or (p.DisplayName and p.DisplayName:lower():find(query)) then
               targetPlayer = p
               
               TargetLabel:Set("Current Target: " .. p.Name)
               
               if PlayerDropdown then
                   PlayerDropdown:Set({p.Name})
               end

               Rayfield:Notify({
                   Title = "Target Found",
                   Content = "Set target to: " .. p.Name,
                   Duration = 2
               })
               break
           end
       end
   end,
})

PlayerDropdown = Tab:CreateDropdown({
   Name = "Select Player",
   Options = getNames(),
   CurrentOption = {"None"},
   Callback = function(opt) 
       targetPlayer = game.Players:FindFirstChild(opt[1])
       TargetLabel:Set("Current Target: " .. (targetPlayer and targetPlayer.Name or "None"))
   end,
})

Tab:CreateDropdown({
   Name = "Mirror Position",
   Options = {"Center", "Left", "Right", "Front", "Back", "Up", "Stand"},
   CurrentOption = {"Center"},
   Callback = function(opt) 
       offsetMode = opt[1]
   end,
})

Tab:CreateButton({
   Name = "Next Player",
   Callback = function()
       local players = getPlayers()
       if #players > 0 then
           currentIndex = (currentIndex % #players) + 1
           targetPlayer = players[currentIndex]
           PlayerDropdown:Set({targetPlayer.Name})
           TargetLabel:Set("Current Target: " .. targetPlayer.Name)
       end
   end,
})

Tab:CreateButton({
   Name = "Previous Player",
   Callback = function()
       local players = getPlayers()
       if #players > 0 then
           currentIndex = (currentIndex - 2 + #players) % #players + 1
           targetPlayer = players[currentIndex]
           PlayerDropdown:Set({targetPlayer.Name})
           TargetLabel:Set("Current Target: " .. targetPlayer.Name)
       end
   end,
})

Tab:CreateSlider({
   Name = "Mirror Delay",
   Range = {0, 5},
   Increment = 0.1,
   Suffix = "s",
   CurrentValue = 0,
   Callback = function(v) mirrorDelay = v end,
})

Tab:CreateButton({
   Name = "Refresh Player List",
   Callback = function() PlayerDropdown:Refresh(getNames(), true) end,
})

game:GetService("RunService").RenderStepped:Connect(function()
    if mirrorEnabled and targetPlayer and targetPlayer.Character then
        local myChar = game.Players.LocalPlayer.Character
        local tChar = targetPlayer.Character
        
        if myChar and tChar then
            local myHum = myChar:FindFirstChildOfClass("Humanoid")
            local tHum = tChar:FindFirstChildOfClass("Humanoid")
            local myRoot = myChar:FindFirstChild("HumanoidRootPart")
            local tRoot = tChar:FindFirstChild("HumanoidRootPart")

            if myRoot and tRoot and myHum and tHum then
                if myChar:FindFirstChild("Animate") then myChar.Animate.Disabled = true end

                task.delay(mirrorDelay, function()
                    if not mirrorEnabled then return end
                    
                    local chosenOffset = offsets[offsetMode] or offsets["Center"]
                    myRoot.CFrame = tRoot.CFrame * chosenOffset
                    
                    local tTracks = tHum:GetPlayingAnimationTracks()
                    local myTracks = myHum:GetPlayingAnimationTracks()

                    for _, tTrack in pairs(tTracks) do
                        local isPlaying = false
                        for _, myTrack in pairs(myTracks) do
                            if myTrack.Animation.AnimationId == tTrack.Animation.AnimationId then
                                isPlaying = true
                                myTrack.TimePosition = tTrack.TimePosition
                                myTrack:AdjustSpeed(tTrack.Speed)
                                break
                            end
                        end
                        if not isPlaying then
                            local new = myHum:LoadAnimation(tTrack.Animation)
                            new:Play(0)
                        end
                    end
                    
                    for _, myTrack in pairs(myTracks) do
                        local found = false
                        for _, tTrack in pairs(tTracks) do
                            if tTrack.Animation.AnimationId == myTrack.Animation.AnimationId then
                                found = true
                                break
                            end
                        end
                        if not found then myTrack:Stop(0) end
                    end
                end)
            end
        end
    end
end)

local WorldTab = Window:CreateTab("World")
WorldTab:CreateSection("Lighting")

WorldTab:CreateToggle({
    Name = "Fullbright",
    CurrentValue = false,
    Flag = "FB",
    Callback = function(value)
        Toggles.Fullbright = value
        if value then
            Lighting.Brightness = 2
            Lighting.ClockTime = 14
            Lighting.FogEnd = 1e6
            Lighting.GlobalShadows = false
        else
            Lighting.Brightness = 1
            Lighting.GlobalShadows = true
            Lighting.FogEnd = 1e3
        end
    end
})

WorldTab:CreateSlider({
    Name = "Time",
    Range = {0, 24},
    Increment = 0.5,
    Suffix = "h",
    CurrentValue = 14,
    Flag = "TM",
    Callback = function(value)
        Lighting.ClockTime = value
    end
})

WorldTab:CreateSection("X-Ray")

WorldTab:CreateToggle({
		Name = "Xray",
		CurrentValue = false,
		Flag = "Xray",
		Callback = function(Value)
			if Value == true then
				for i,v in pairs(workspace:GetDescendants()) do
					if v:IsA("BasePart") then
						v.LocalTransparencyModifier = 0.5
					end
				end
			else
				for i,v in pairs(workspace:GetDescendants()) do
					if v:IsA("BasePart") then
						v.LocalTransparencyModifier = 0
					end
				end
			end
		end,
})

local InstantInteractEnabled = false

WorldTab:CreateToggle({
   Name = "Instant Proximity Prompt",
   CurrentValue = false,
   Flag = "InstantInteract",
   Callback = function(Value)
      InstantInteractEnabled = Value
      
      task.spawn(function()
         while InstantInteractEnabled do
            for _, obj in ipairs(game:GetDescendants()) do
               if obj:IsA("ProximityPrompt") then
                  obj.HoldDuration = 0
               end
            end
            task.wait(2)
         end
      end)
   end,
})

local VehicleTab = Window:CreateTab("Vehicle")
VehicleTab:CreateSection("Only certain vehicles")

local function GetCurrentVehicle()
    if not Character then
        return nil
    end
    local humanoid = Character:FindFirstChildOfClass("Humanoid")
    if humanoid and humanoid.SeatPart then
        local vehicle = humanoid.SeatPart.Parent
        while vehicle and vehicle.Parent ~= Workspace do
            if vehicle:FindFirstChildOfClass("VehicleSeat") then
                return vehicle
            end
            vehicle = vehicle.Parent
        end
        return humanoid.SeatPart.Parent
    end
    return nil
end

VehicleTab:CreateSlider({
    Name = "Vehicle Boost",
    Range = {1, 10},
    Increment = 0.5,
    Suffix = "x",
    CurrentValue = 1,
    Flag = "VB",
    Callback = function(value)
        Settings.VehicleBoost = value
        local vehicle = GetCurrentVehicle()
        if vehicle then
            for _, descendant in vehicle:GetDescendants() do
                if descendant:IsA("VehicleSeat") then
                    descendant.MaxSpeed = 100 * value
                    descendant.Torque = 50 * value
                end
            end
        end
    end
})

VehicleTab:CreateToggle({
    Name = "Loop Boost",
    CurrentValue = false,
    Flag = "LV",
    Callback = function(value)
        Toggles.LoopVehicleBoost = value
    end
})

VehicleTab:CreateSection("Fly (more vehicles than the obove one)")

VehicleTab:CreateToggle({
    Name = "Vehicle Fly",
    CurrentValue = false,
    Flag = "VF",
    Callback = function(value)
        Toggles.VehicleFly = value
        local vehicle = GetCurrentVehicle()
        if not vehicle then
            Toggles.VehicleFly = false
            return
        end
        local primaryPart = vehicle.PrimaryPart or vehicle:FindFirstChildWhichIsA("BasePart")
        if not primaryPart then
            return
        end
        if value then
            local bodyVelocity = Instance.new("BodyVelocity")
            bodyVelocity.Name = "VehFlyV"
            bodyVelocity.MaxForce = Vector3.one * 9e9
            bodyVelocity.Velocity = Vector3.zero
            bodyVelocity.Parent = primaryPart
            
            local bodyGyro = Instance.new("BodyGyro")
            bodyGyro.Name = "VehFlyG"
            bodyGyro.MaxTorque = Vector3.one * 9e9
            bodyGyro.P = 9e4
            bodyGyro.Parent = primaryPart
        else
            local flyV = primaryPart:FindFirstChild("VehFlyV")
            local flyG = primaryPart:FindFirstChild("VehFlyG")
            if flyV then
                flyV:Destroy()
            end
            if flyG then
                flyG:Destroy()
            end
        end
    end
})

VehicleTab:CreateSlider({
    Name = "Vehicle Fly Speed",
    Range = {10, 1000},
    Increment = 10,
    Suffix = "",
    CurrentValue = 50,
    Flag = "VFS",
    Callback = function(value)
        Settings.VehicleFlySpeed = value
    end
})

local ESPTab = Window:CreateTab("ESP")
ESPTab:CreateSection("Player ESP")

local function UpdateESP()
    ESPFolder:ClearAllChildren()
    if not Toggles.ESP then
        return
    end
    for _, player in PlayersService:GetPlayers() do
        if player ~= LocalPlayer and player.Character then
            local char = player.Character
            local head = char:FindFirstChild("Head")
            local rootPart = char:FindFirstChild("HumanoidRootPart")
            local humanoid = char:FindFirstChildOfClass("Humanoid")
            
            if head and rootPart and humanoid then
                if Toggles.NameAndHealth then
                    local billboard = Instance.new("BillboardGui")
                    billboard.Name = player.Name .. "_Name"
                    billboard.Adornee = head
                    billboard.Size = UDim2.new(0, 100, 0, 40)
                    billboard.StudsOffset = Vector3.new(0, 2, 0)
                    billboard.AlwaysOnTop = true
                    billboard.Parent = ESPFolder
                    
                    local nameLabel = Instance.new("TextLabel")
                    nameLabel.Size = UDim2.new(1, 0, 0.5, 0)
                    nameLabel.BackgroundTransparency = 1
                    nameLabel.Text = player.Name
                    nameLabel.TextColor3 = Color3.new(1, 1, 1)
                    nameLabel.TextScaled = true
                    nameLabel.Font = Enum.Font.SourceSansBold
                    nameLabel.Parent = billboard
                    
                    local healthLabel = Instance.new("TextLabel")
                    healthLabel.Size = UDim2.new(1, 0, 0.5, 0)
                    healthLabel.Position = UDim2.new(0, 0, 0.5, 0)
                    healthLabel.BackgroundTransparency = 1
                    healthLabel.Text = math.floor(humanoid.Health) .. " HP"
                    healthLabel.TextColor3 = Color3.new(0, 1, 0)
                    healthLabel.TextScaled = true
                    healthLabel.Font = Enum.Font.SourceSansBold
                    healthLabel.Parent = billboard
                end
                
                if Toggles.Highlight then
                    local highlight = Instance.new("Highlight")
                    highlight.Name = player.Name .. "_High"
                    highlight.Adornee = char
                    highlight.FillColor = Color3.new(1, 0, 0)
                    highlight.OutlineColor = Color3.new(1, 1, 1)
                    highlight.FillTransparency = 0.7
                    highlight.OutlineTransparency = 0
                    highlight.Parent = ESPFolder
                end
            end
        end
    end
end

ESPTab:CreateToggle({
    Name = "Enable ESP",
    CurrentValue = false,
    Flag = "ES",
    Callback = function(value)
        Toggles.ESP = value
        UpdateESP()
    end
})

ESPTab:CreateToggle({
    Name = "Name and Health",
    CurrentValue = false,
    Flag = "NE",
    Callback = function(value)
        Toggles.NameAndHealth = value
        UpdateESP()
    end
})

ESPTab:CreateToggle({
    Name = "Highlight",
    CurrentValue = false,
    Flag = "BE",
    Callback = function(value)
        Toggles.Highlight = value
        UpdateESP()
    end
})

ESPTab:CreateButton({
    Name = "Refresh ESP",
    Callback = function()
        UpdateESP()
    end
})

local MiscTab = Window:CreateTab("Misc")
MiscTab:CreateSection("Utility")

MiscTab:CreateToggle({
    Name = "Anti AFK",
    CurrentValue = false,
    Flag = "AA",
    Callback = function(value)
        Toggles.AntiAFK = value
    end
})

MiscTab:CreateButton({
    Name = "Rejoin",
    Callback = function()
        TeleportService:Teleport(game.PlaceId, LocalPlayer)
    end
})

MiscTab:CreateButton({
    Name = "Small server                                                                             (works 50% of times, can use as serverhop)",
    Callback = function()
    local module = loadstring(game:HttpGet"https://raw.githubusercontent.com/LeoKholYt/roblox/main/lk_serverhop.lua")()

module:Teleport(game.PlaceId)
    end
})

MiscTab:CreateButton({
    Name = "Copy Game ID",
    Callback = function()
        setclipboard(tostring(game.PlaceId))
    end
})

MiscTab:CreateSection("GUI")

MiscTab:CreateButton({
    Name = "Destroy GUI",
    Callback = function()
        StopSpectating()
        ESPFolder:Destroy()
        Library:Destroy()
    end
})

RunService.Heartbeat:Connect(function()
    UpdateCharacter()
    if not Humanoid then
        return
    end
    
    if Toggles.LoopWalkSpeed then
        pcall(function()
            Humanoid.WalkSpeed = Settings.WalkSpeed
        end)
    end
    
    if Toggles.LoopJumpPower then
        pcall(function()
            Humanoid.JumpPower = Settings.JumpPower
            Humanoid.UseJumpPower = true
        end)
    end
    
    if Toggles.LoopGravity then
        Workspace.Gravity = Settings.Gravity
    end
    
    if Toggles.LoopVehicleBoost then
        local vehicle = GetCurrentVehicle()
        if vehicle then
            for _, descendant in vehicle:GetDescendants() do
                if descendant:IsA("VehicleSeat") then
                    descendant.MaxSpeed = 200 * Settings.VehicleBoost
                    descendant.Torque = 100 * Settings.VehicleBoost
                end
            end
        end
    end
    
    if Toggles.GodMode then
        pcall(function()
            Humanoid.Health = Humanoid.MaxHealth
        end)
    end
    
    if Toggles.AntiVoid and HumanoidRootPart then
        if HumanoidRootPart.Position.Y < -50 then
            HumanoidRootPart.CFrame = CFrame.new(0, 50, 0)
        end
    end
    
    if Toggles.Spectating and SpectatingTarget then
        if SpectatingTarget.Parent and SpectatingTarget.Character then
            local targetHumanoid = SpectatingTarget.Character:FindFirstChildOfClass("Humanoid")
            local targetRoot = SpectatingTarget.Character:FindFirstChild("HumanoidRootPart")
            local healthText = targetHumanoid and math.floor(targetHumanoid.Health) .. "/" .. math.floor(targetHumanoid.MaxHealth) or "N/A"
            local posText = targetRoot and string.format("%.0f,%.0f,%.0f", targetRoot.Position.X, targetRoot.Position.Y, targetRoot.Position.Z) or "N/A"
            pcall(function()
                SpectateStatusLabel:Set("Spectating: " .. SpectatingTarget.Name .. " | " .. healthText .. " | " .. posText)
            end)
        else
            StopSpectating()
        end
    else
        pcall(function()
            SpectateStatusLabel:Set("Not spectating")
        end)
    end
end)

RunService.RenderStepped:Connect(function()
    if Toggles.Fly and HumanoidRootPart then
        local flyV = HumanoidRootPart:FindFirstChild("FlyV")
        local flyG = HumanoidRootPart:FindFirstChild("FlyG")
        if flyV and flyG then
            flyG.CFrame = Camera.CFrame
            local moveDirection = Vector3.zero
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then
                moveDirection = moveDirection + Camera.CFrame.LookVector
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then
                moveDirection = moveDirection - Camera.CFrame.LookVector
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then
                moveDirection = moveDirection - Camera.CFrame.RightVector
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then
                moveDirection = moveDirection + Camera.CFrame.RightVector
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
                moveDirection = moveDirection + Vector3.yAxis
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
                moveDirection = moveDirection - Vector3.yAxis
            end
            flyV.Velocity = moveDirection.Magnitude > 0 and moveDirection.Unit * Settings.FlySpeed or Vector3.zero
        end
    end
    
    if Toggles.VehicleFly then
        local vehicle = GetCurrentVehicle()
        if vehicle then
            local primaryPart = vehicle.PrimaryPart or vehicle:FindFirstChildWhichIsA("BasePart")
            if primaryPart then
                local flyV = primaryPart:FindFirstChild("VehFlyV")
                local flyG = primaryPart:FindFirstChild("VehFlyG")
                if flyV and flyG then
                    flyG.CFrame = Camera.CFrame
                    local moveDirection = Vector3.zero
                    if UserInputService:IsKeyDown(Enum.KeyCode.W) then
                        moveDirection = moveDirection + Camera.CFrame.LookVector
                    end
                    if UserInputService:IsKeyDown(Enum.KeyCode.S) then
                        moveDirection = moveDirection - Camera.CFrame.LookVector
                    end
                    if UserInputService:IsKeyDown(Enum.KeyCode.A) then
                        moveDirection = moveDirection - Camera.CFrame.RightVector
                    end
                    if UserInputService:IsKeyDown(Enum.KeyCode.D) then
                        moveDirection = moveDirection + Camera.CFrame.RightVector
                    end
                    if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
                        moveDirection = moveDirection + Vector3.yAxis
                    end
                    if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
                        moveDirection = moveDirection - Vector3.yAxis
                    end
                    flyV.Velocity = moveDirection.Magnitude > 0 and moveDirection.Unit * Settings.VehicleFlySpeed or Vector3.zero
                end
            end
        end
    end
end)

RunService.Stepped:Connect(function()
    if Toggles.Noclip and Character then
        for _, part in Character:GetDescendants() do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
end)

UserInputService.JumpRequest:Connect(function()
    if Toggles.InfiniteJump and Humanoid then
        pcall(function()
            Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        end)
    end
end)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then
        return
    end
    if Toggles.ClickTeleport and input.UserInputType == Enum.UserInputType.MouseButton1 and HumanoidRootPart then
        local mouse = LocalPlayer:GetMouse()
        if mouse.Target then
            HumanoidRootPart.CFrame = CFrame.new(mouse.Hit.Position + Vector3.new(0, 3, 0))
        end
    end
end)

task.spawn(function()
    while task.wait(60) do
        if Toggles.AntiAFK then
            pcall(function()
                local virtualUser = game:GetService("VirtualUser")
                virtualUser:CaptureController()
                virtualUser:ClickButton2(Vector2.zero)
            end)
        end
    end
end)

PlayersService.PlayerAdded:Connect(function()
    task.delay(1, function()
        local list = GetPlayerList()
        pcall(function()
            SpectateDropdown:Refresh(list)
        end)
        pcall(function()
            InfoDropdown:Refresh(list)
        end)
        pcall(function()
            TeleportDropdown:Refresh(list)
        end)
        UpdateESP()
    end)
end)

PlayersService.PlayerRemoving:Connect(function(player)
    task.delay(0.5, function()
        local list = GetPlayerList()
        pcall(function()
            SpectateDropdown:Refresh(list)
        end)
        pcall(function()
            InfoDropdown:Refresh(list)
        end)
        pcall(function()
            TeleportDropdown:Refresh(list)
        end)
        UpdateESP()
        if SpectatingTarget and SpectatingTarget == player then
            StopSpectating()
        end
    end)
end)

Library:Notify({
    Title = "Franks Simple GUI",
    Content = "Just fused many common universal script thank for trying it out",
    Duration = 5
})
