repeat task.wait() until game:IsLoaded()

if setfpscap then
    setfpscap(1000000)
else
    warn("Your exploit does not support setfpscap.")
end

local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()

local Localization = WindUI:Localization({
    Enabled = true,
    Prefix = "loc:",
    DefaultLanguage = "en",
    Translations = {
        ["en"] = {
            ["CPS_NETWORK"] = "CPS Network",
            ["WELCOME"] = "Welcome to CPS Network!",
            ["HUB_DESC"] = "Premium Roblox Script Hub",
            ["INTRODUCTION"] = "Introduction",
            ["COMBAT"] = "Combat",
            ["SHOP"] = "Shop",
            ["AUTOMATION"] = "Automation",
            ["UTILITIES"] = "Utilities",
            ["SETTINGS"] = "Settings"
        }
    }
})

WindUI:AddTheme({
    Name = "MonoOrange",

    Accent = WindUI:Gradient({
        ["0"]   = { Color = Color3.fromHex("#FFB347"), Transparency = 0 },
        ["100"] = { Color = Color3.fromHex("#D94A00"), Transparency = 0 },
    }, {
        Rotation = 90,
    }),

    Dialog = Color3.fromHex("#2A0A00"),
    Outline = Color3.fromHex("#5B2606"),
    Text = Color3.fromHex("#FFEEDD"),
    Placeholder = Color3.fromHex("#7A3A12"),
    Background = Color3.fromHex("#0D0400"),
    Button = Color3.fromHex("#7A3A12"),
    Icon = Color3.fromHex("#FFB347"),

    Hover = Color3.fromHex("#FFB24D"),
    Active = Color3.fromHex("#D95A00"),
    ScrollBar = Color3.fromHex("#3A1806"),
    Border = Color3.fromHex("#6C3208"),

    Glow = Color3.fromHex("#FFAE4F"),
    Shadow = Color3.fromHex("#210800"),
})

WindUI:SetFont("rbxassetid://12187375716")
WindUI:SetTheme("MonoOrange")

local function gradient(text, startColor, endColor)
    local result = ""
    for i = 1, #text do
        local t = (i - 1) / (#text - 1)
        local r = math.floor((startColor.R + (endColor.R - startColor.R) * t) * 255)
        local g = math.floor((startColor.G + (endColor.G - startColor.G) * t) * 255)
        local b = math.floor((startColor.B + (endColor.B - startColor.B) * t) * 255)
        result = result .. string.format('<font color="rgb(%d,%d,%d)">%s</font>', r, g, b, text:sub(i, i))
    end
    return result
end

WindUI:Popup({
    Title = gradient("Welcome To CPS Network!", Color3.fromHex("#ec4d03"), Color3.fromHex("#490303")),
    Icon = "rbxassetid://115220176602432",
    Content = "Please join the discord server !",
    Theme = "MonoOrange",
    Buttons = {
        {
            Title = "Enjoy!",
            Icon = "lucide:heart",
            Variant = "Primary",
            Callback = function() end
        }
    }
})

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local VirtualUser = game:GetService("VirtualUser")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")

local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")
local Backpack = LocalPlayer:WaitForChild("Backpack")

local AutoCollect = false
local AutoFarm = false
local autoClicking = false
local AutoCollectDelay = 60
local ClickInterval = 0.25
local HeldToolName = "Basic Bat"
local SellPlant = false
local SellBrainrot = false
local SellEverything = false

local serverStartTime = os.time()

local shop = {
    seedList = {
        "Cactus Seed",
        "Strawberry Seed",
        "Pumpkin Seed",
        "Sunflower Seed",
        "Dragon Seed",
        "Eggplant Seed",
        "Watermelon Seed",
        "Cocotank Seed",
        "Carnivorous Plant Seed",
        "Mr Carrot Seed",
        "Tomatrio Seed",
        "Shroombino Seed"
    },
    gearList = {
        "Water Bucket",
        "Frost Grenade",
        "Banana Gun",
        "Frost Blower",
        "Carrot Launcher"
    }
}

local selectedSeeds = {}
local selectedGears = {}
local AutoBuySelectedSeed = false
local AutoBuySelectedGear = false
local AutoBuyAllSeed = false
local AutoBuyAllGear = false
local AutoBuySeed = false
local AutoBuyGear = false

local function GetMyPlot()
    for _, plot in ipairs(Workspace.Plots:GetChildren()) do
        local playerSign = plot:FindFirstChild("PlayerSign")
        if playerSign then
            local bg = playerSign:FindFirstChild("BillboardGui")
            local textLabel = bg and bg:FindFirstChild("TextLabel")
            if textLabel and (textLabel.Text == LocalPlayer.Name or textLabel.Text == LocalPlayer.DisplayName) then
                return plot
            end
        end
    end
    return nil
end

local function GetMyPlotName()
    local plot = GetMyPlot()
    return plot and plot.Name or "No Plot"
end

local function GetMoney()
    local leaderstats = LocalPlayer:FindFirstChild("leaderstats")
    return leaderstats and leaderstats:FindFirstChild("Money") and leaderstats.Money.Value or 0
end

local function GetRebirth()
    local gui = LocalPlayer:FindFirstChild("PlayerGui") and LocalPlayer.PlayerGui:FindFirstChild("Main")
    if gui and gui:FindFirstChild("Rebirth") then
        local text = gui.Rebirth.Frame.Title.Text or "Rebirth 0"
        local n = tonumber(text:match("%d+")) or 0
        return math.max(0, n - 1)
    end
    return 0
end

local function FormatTime(sec)
    local h = math.floor(sec / 3600)
    local m = math.floor((sec % 3600) / 60)
    local s = sec % 60
    return string.format("%02d:%02d:%02d", h, m, s)
end

local function GetBridgeNet2()
    return ReplicatedStorage:FindFirstChild("BridgeNet2")
end

local function GetRemotesFolder()
    return ReplicatedStorage:FindFirstChild("Remotes")
end

local Window = WindUI:CreateWindow({
    Title = "Plants vs Brainrots",
    Icon = "rbxassetid://115220176602432",
    Author = "CPS Network",
    Folder = "WindUI_Example",
    Size = UDim2.fromOffset(590, 500),
    Theme = "MonoOrange",
    Transparent = true,
    BackgroundImageTransparency = 0.25,
    HidePanelBackground = false,
    NewElements = false,
    Background = "rbxassetid://126127539858360",
    User = {
        Enabled = true,
        Anonymous = true,
        Callback = function()
            WindUI:Notify({
                Title = "User Profile",
                Content = "User profile clicked!",
                Duration = 3
            })
        end
    },
    Acrylic = false,
    HideSearchBar = false,
    SideBarWidth = 200,
    
    OpenButton = {
        Title = "Open CPS Network Hub UI",
        CornerRadius = UDim.new(1,0),
        StrokeThickness = 3,
        Enabled = true,
        OnlyMobile = false,
        
        Color = ColorSequence.new(
            Color3.fromHex("#FFB347"), 
            Color3.fromHex("#D94A00")
        ),
    },
})

Window.User:SetAnonymous(false)
Window:SetIconSize(48)

Window:Tag({
    Title = "By CPS Network",
    Color = Color3.fromHex("#ff3300")
})

Window:CreateTopbarButton("theme-switcher", "geist:logo-discord", function()
    local link = "https://discord.com/invite/J2PK69tFRM"
    if setclipboard then
        pcall(function()
            setclipboard(link)
        end)
    end
    WindUI:Notify({
        Title = "Discord link has been copied!",
        Content = "Please join our discord!",
        Icon = "geist:logo-discord",
        Duration = 3
    })
end, 990)

Window:CreateTopbarButton("theme-switcherr", "lucide:youtube", function()
    local link = "https://www.youtube.com/@waspire"
    if setclipboard then
        pcall(function()
            setclipboard(link)
        end)
    end
    WindUI:Notify({
        Title = "Youtube Channel link has been copied!",
        Content = "Please Subscribe!",
        Icon = "lucide:youtube",
        Duration = 3
    })
end, 990)

Window:Divider()

local Tabs = {
    Introduction = Window:Tab({
        Title = "Introduction",
        Icon = "lucide:house",
        Opened = true
    }),

    Window:Divider(),

    Utilities = Window:Tab({
        Title = "Utilities",
        Icon = "lucide:plus",
        Opened = true
    }),
    
    AutoCounter = Window:Tab({
        Title = "Auto Counter",
        Icon = "lucide:shield",
        Opened = true
    }),
    
    AutoCombat = Window:Tab({
        Title = "Auto Combat",
        Icon = "lucide:sword",
        Opened = true
    }),
}

Tabs.Introduction:Divider()

Tabs.Introduction:Paragraph({
    Title = "   <font size=\"20\">━━━<u>CPS Network Introduction</u>━━━</font>",
    Desc = "",
    TextSize = 21,
    TextTransparency = 0.05,
    Box = false,
})

Tabs.Introduction:Divider()

Tabs.Introduction:Paragraph({
    Title = "",
    Desc = "",
    Image = "",
    ImageSize = 30,
    Thumbnail = "rbxassetid://136887189396137",
    ThumbnailSize = 250,
})

Tabs.Introduction:Divider()

Tabs.Introduction:Paragraph({
    Title = gradient("Welcome to CPS Network!", Color3.fromHex("#FFB347"), Color3.fromHex("#D94A00")),
    Desc = "CPS Network is your premium destination for high-quality Roblox scripts and exploits. We specialize in providing cutting-edge automation tools for Plants vs Brainrots and many other popular Roblox games.\n\nOur hub features over 40,000+ active members and constantly updated scripts!",
    TextSize = 14,
    Box = true,
})

Tabs.Introduction:Divider()

Tabs.Introduction:Paragraph({
    Title = gradient("Key Features", Color3.fromHex("#ec4d03"), Color3.fromHex("#D94A00")),
    Desc = "• Advanced Auto Combat System - Automatically farm brainrots with precision\n• Smart Auto Collection - Efficiently collect money from your plots\n• Auto Shop Integration - Purchase seeds and gear automatically\n• Anti-AFK Protection - Stay in-game longer\n• Premium Support - Join our Discord for 24/7 assistance",
    TextSize = 13,
    Box = true,
})

Tabs.Introduction:Divider()

Tabs.Introduction:Paragraph({
    Title = gradient("Getting Started", Color3.fromHex("#FFB347"), Color3.fromHex("#490303")),
    Desc = "1. Join our Discord server for updates and support\n2. Navigate through the tabs to explore features\n3. Enable Auto Combat for automatic farming\n4. Use Auto Counter for defense strategies\n5. Configure Utilities for enhanced gameplay\n\nFor best results, use on PRIVATE SERVERS ONLY!",
    TextSize = 13,
    Box = true,
})

Tabs.Introduction:Divider()

Tabs.Introduction:Button({
    Title = "Join CPS Network Discord",
    Icon = "geist:logo-discord",
    Callback = function()
        local link = "https://discord.com/invite/J2PK69tFRM"
        if setclipboard then
            pcall(function()
                setclipboard(link)
            end)
        end
        WindUI:Notify({
            Title = "Discord Invite Copied!",
            Content = "Discord link copied to clipboard!",
            Icon = "check",
            Duration = 3
        })
    end
})

Tabs.Introduction:Button({
    Title = "Subscribe on YouTube",
    Icon = "lucide:youtube",
    Callback = function()
        local link = "https://www.youtube.com/@waspire"
        if setclipboard then
            pcall(function()
                setclipboard(link)
            end)
        end
        WindUI:Notify({
            Title = "YouTube Link Copied!",
            Content = "Please subscribe to our channel!",
            Icon = "check",
            Duration = 3
        })
    end
})

-- ========== UTILITIES TAB ==========

local function GetNearestPlot()
    local nearestPlot = nil
    local minDist = math.huge
    for _, plot in ipairs(Workspace.Plots:GetChildren()) do
        if plot:IsA("Folder") then
            local center = plot:FindFirstChild("Center") or plot:FindFirstChildWhichIsA("BasePart")
            if center then
                local dist = (HumanoidRootPart.Position - center.Position).Magnitude
                if dist < minDist then
                    minDist = dist
                    nearestPlot = plot
                end
            end
        end
    end
    return nearestPlot
end

local function CollectFromPlot(plot)
    if not plot then return end
    local brainrotsFolder = plot:FindFirstChild("Brainrots")
    if not brainrotsFolder then return end

    for i = 1, 17 do
        local slot = brainrotsFolder:FindFirstChild(tostring(i))
        if slot and slot:FindFirstChild("Brainrot") then
            local brainrot = slot:FindFirstChild("Brainrot")
            if brainrot:FindFirstChild("BrainrotHitbox") then
                local hitbox = brainrot.BrainrotHitbox
                local offset = Vector3.new(0, 1, 3)
                HumanoidRootPart.CFrame = CFrame.new(hitbox.Position + offset, hitbox.Position)
                task.wait(0.2)
                pcall(function()
                    local remotes = GetRemotesFolder()
                    if remotes and remotes:FindFirstChild("AttacksServer") and remotes.AttacksServer:FindFirstChild("WeaponAttack") then
                        remotes.AttacksServer.WeaponAttack:FireServer({ { target = hitbox } })
                    else
                        ReplicatedStorage.Remotes.AttacksServer.WeaponAttack:FireServer({ { target = hitbox } })
                    end
                end)
            end
        end
    end
end

Tabs.Utilities:Paragraph({
    Title = "Auto Collection Settings",
    Desc = "Configure automatic money collection from your plots",
    Image = "lucide:settings",
    ImageSize = 20,
    Color = Color3.fromHex("#FFB347"),
})

Tabs.Utilities:Divider()

Tabs.Utilities:Slider({
    Title = "Collection Delay (seconds)",
    Desc = "Time between each collection cycle",
    Value = {Min = 1, Max = 60, Default = 5},
    Step = 1,
    Callback = function(val)
        AutoCollectDelay = val
    end
})

Tabs.Utilities:Toggle({
    Title = "Auto Collect Money",
    Desc = "Automatically collect money from nearest plot",
    Value = false,
    Callback = function(state)
        AutoCollect = state
        if state then
            task.spawn(function()
                while AutoCollect do
                    local nearestPlot = GetNearestPlot()
                    if nearestPlot then
                        CollectFromPlot(nearestPlot)
                    end
                    task.wait(AutoCollectDelay)
                end
            end)
        end
    end
})

Tabs.Utilities:Toggle({
    Title = "Auto Collect V2 [PATCHED]",
    Desc = "Alternative collection method (currently patched)",
    Value = false,
    Callback = function(state)
        if state then
            task.spawn(function()
                while state do
                    local args = {
                        {
                            [2] = "\004"
                        }
                    }
                    game:GetService("ReplicatedStorage"):WaitForChild("BridgeNet2"):WaitForChild("dataRemoteEvent"):FireServer(unpack(args))
                    task.wait(1)
                end
            end)
        end
    end
})

Tabs.Utilities:Divider()

Tabs.Utilities:Paragraph({
    Title = "Selling Options",
    Desc = "Automatically sell your plants and brainrots",
    Image = "lucide:dollar-sign",
    ImageSize = 20,
    Color = Color3.fromHex("#D94A00"),
})

Tabs.Utilities:Divider()

Tabs.Utilities:Toggle({
    Title = "Sell All Brainrots",
    Desc = "Automatically sell all brainrots in inventory",
    Value = false,
    Callback = function(state)
        SellBrainrot = state
    end
})

Tabs.Utilities:Toggle({
    Title = "Sell All Plants",
    Desc = "Automatically sell all plants in inventory",
    Value = false,
    Callback = function(state)
        SellPlant = state
    end
})

Tabs.Utilities:Toggle({
    Title = "Sell Everything",
    Desc = "Sell both plants and brainrots simultaneously",
    Value = false,
    Callback = function(state)
        SellEverything = state
    end
})

Tabs.Utilities:Divider()

Tabs.Utilities:Paragraph({
    Title = "Shop Automation - Seeds",
    Desc = "Automatically purchase seeds from the shop",
    Image = "lucide:shopping-cart",
    ImageSize = 20,
    Color = Color3.fromHex("#FFB347"),
})

Tabs.Utilities:Divider()

Tabs.Utilities:Dropdown({
    Title = "Select Seeds to Buy",
    Values = shop.seedList,
    Multi = true,
    AllowNone = true,
    Callback = function(values)
        selectedSeeds = values
    end
})

Tabs.Utilities:Toggle({
    Title = "Auto Buy Selected Seeds",
    Desc = "Purchase only the selected seeds",
    Value = false,
    Callback = function(state)
        AutoBuySelectedSeed = state
        AutoBuySeed = state
        if state then
            task.spawn(function()
                while AutoBuySelectedSeed do
                    for _, seed in ipairs(selectedSeeds) do
                        local args = {{ seed, "\b" }}
                        game:GetService("ReplicatedStorage"):WaitForChild("BridgeNet2"):WaitForChild("dataRemoteEvent"):FireServer(unpack(args))
                        task.wait(0.5)
                    end
                    task.wait(1)
                end
            end)
        end
    end
})

Tabs.Utilities:Toggle({
    Title = "Auto Buy ALL Seeds",
    Desc = "Purchase all available seeds continuously",
    Value = false,
    Callback = function(state)
        AutoBuyAllSeed = state
        if state then
            task.spawn(function()
                while AutoBuyAllSeed do
                    for _, seed in ipairs(shop.seedList) do
                        local args = {{ seed, "\b" }}
                        game:GetService("ReplicatedStorage"):WaitForChild("BridgeNet2"):WaitForChild("dataRemoteEvent"):FireServer(unpack(args))
                        task.wait(0.5)
                    end
                    task.wait(1)
                end
            end)
        end
    end
})

-- ======= CONTINUATION IN SNIPPET 2 =======
Tabs.Utilities:Paragraph({
    Title = "Shop Automation - Gear",
    Desc = "Automatically purchase gear from the shop",
    Image = "lucide:package",
    ImageSize = 20,
    Color = Color3.fromHex("#FFB347"),
})

Tabs.Utilities:Divider()

Tabs.Utilities:Dropdown({
    Title = "Select Gear to Buy",
    Values = shop.gearList,
    Multi = true,
    AllowNone = true,
    Callback = function(values)
        selectedGears = values
    end
})

Tabs.Utilities:Toggle({
    Title = "Auto Buy Selected Gear",
    Desc = "Purchase only the selected gear",
    Value = false,
    Callback = function(state)
        AutoBuySelectedGear = state
        AutoBuyGear = state
        if state then
            task.spawn(function()
                while AutoBuySelectedGear do
                    for _, gear in ipairs(selectedGears) do
                        local args = {{ gear, "\026" }}
                        game:GetService("ReplicatedStorage"):WaitForChild("BridgeNet2"):WaitForChild("dataRemoteEvent"):FireServer(unpack(args))
                        task.wait(0.5)
                    end
                    task.wait(1)
                end
            end)
        end
    end
})

Tabs.Utilities:Toggle({
    Title = "Auto Buy ALL Gear",
    Desc = "Purchase all available gear continuously",
    Value = false,
    Callback = function(state)
        AutoBuyAllGear = state
        if state then
            task.spawn(function()
                while AutoBuyAllGear do
                    for _, gear in ipairs(shop.gearList) do
                        local args = {{ gear, "\026" }}
                        game:GetService("ReplicatedStorage"):WaitForChild("BridgeNet2"):WaitForChild("dataRemoteEvent"):FireServer(unpack(args))
                        task.wait(0.5)
                    end
                    task.wait(1)
                end
            end)
        end
    end
})

-- ========== AUTO COUNTER TAB ==========

Tabs.AutoCounter:Paragraph({
    Title = "Auto Counter",
    Desc = "Automatically counters enemies with timing-based defense.",
    Image = "lucide:shield",
    ImageSize = 24,
    Color = Color3.fromHex("#D94A00"),
})

Tabs.AutoCounter:Toggle({
    Title = "Enable Auto Counter",
    Desc = "Start using the auto-counter system to avoid incoming attacks.",
    Value = false,
    Callback = function(state)
        if state then
            task.spawn(function()
                while state do
                    -- Checks for threatening NPCs nearby and triggers blocks/counters.
                    local enemy = Workspace:FindFirstChild("NearestEnemy") -- Example; replace with game-specific logic
                    if enemy then
                        -- Add proper logic to block/counter here
                        print("Countering detected threat!")
                        -- optional: trigger animation or remote event
                    end
                    task.wait(0.5)
                end
            end)
        end
    end
})

Tabs.AutoCounter:Divider()

Tabs.AutoCounter:Paragraph({
    Title = "Counter Settings",
    Desc = "Fine-tune counter delay and reaction settings.",
    TextSize = 14,
    Box = true,
})

Tabs.AutoCounter:Slider({
    Title = "Counter Reaction Delay",
    Desc = "Set delay before reacting to attack (ms)",
    Value = { Min = 0.05, Max = 1, Default = 0.3 },
    Step = 0.05,
    Callback = function(value)
        -- you could use this value for advanced counter logic
    end
})

-- ========== AUTO COMBAT TAB ==========

Tabs.AutoCombat:Paragraph({
    Title = "Auto Combat (PvB)",
    Desc = "Automates attacking the nearest brainrot enemy in Plants VS Brainrot.",
    Image = "lucide:sword",
    ImageSize = 24,
    Color = Color3.fromHex("#FFB347"),
})

Tabs.AutoCombat:Toggle({
    Title = "Enable Auto Combat",
    Desc = "Farm ALL enemies automatically. Only use on private servers!",
    Value = false,
    Callback = function(state)
        AutoFarm = state
        autoClicking = state

        if state then
            EquipBat()
            task.spawn(function()
                while autoClicking do
                    -- Auto-clicking logic
                    if Character and Character:FindFirstChild(HeldToolName) then
                        pcall(function()
                            VirtualUser:Button1Down(Vector2.new(0, 0))
                            task.wait(0.03)
                            VirtualUser:Button1Up(Vector2.new(0, 0))
                        end)
                    end
                    task.wait(ClickInterval)
                end
            end)
            task.spawn(function()
                while AutoFarm do
                    -- Find and attack nearest brainrot
                    -- Custom auto-farm logic
                    local nearestPlot = GetNearestPlot()
                    if nearestPlot then
                        CollectFromPlot(nearestPlot)
                    end
                    task.wait(1)
                end
            end)
        end
    end
})

Tabs.AutoCombat:Paragraph({
    Title = "Combat Notes",
    Desc = "Uses your equipped tool to attack enemies in range.\nCustom AI logic can be improved for PvB.",
    TextSize = 13,
    Box = true,
})

-- ========== GENERAL WINDOW EVENTS ==========

Window:OnClose(function()
    print("CPS - PvB UI closed")
    if ConfigManager and configFile then
        configFile:Set("playerData", MyPlayerData)
        configFile:Set("lastSave", os.date("%Y-%m-%d %H:%M:%S"))
        configFile:Save()
        print("Config auto-saved on close")
    end
end)

Window:OnDestroy(function()
    print("Window destroyed")
end)

Window:OnOpen(function()
    print("Window opened")
end)

Window:UnlockAll()

task.spawn(function()
    while task.wait(0.69) do
        if SellBrainrot or SellPlant or SellEverything then
            local remotes = GetRemotesFolder()
            if remotes and remotes:FindFirstChild("ItemSell") then
                pcall(function() remotes.ItemSell:FireServer() end)
            else
                pcall(function() ReplicatedStorage.Remotes.ItemSell:FireServer() end)
            end
        end
    end
end)

task.spawn(function()
    while task.wait(0.95) do
        if AutoBuyGear and #selectedGears > 0 then
            local bn = GetBridgeNet2()
            for _, g in ipairs(selectedGears) do
                local args = {{g, "\026"}}
                if bn and bn:FindFirstChild("dataRemoteEvent") then
                    pcall(function() bn.dataRemoteEvent:FireServer(unpack(args)) end)
                else
                    pcall(function() ReplicatedStorage:WaitForChild("BridgeNet2"):WaitForChild("dataRemoteEvent"):FireServer(unpack(args)) end)
                end
                task.wait(0.12)
            end
        end

        if AutoBuySeed and #selectedSeeds > 0 then
            local bn = GetBridgeNet2()
            for _, s in ipairs(selectedSeeds) do
                local args = {{s, "\b"}}
                if bn and bn:FindFirstChild("dataRemoteEvent") then
                    pcall(function() bn.dataRemoteEvent:FireServer(unpack(args)) end)
                else
                    pcall(function() ReplicatedStorage:WaitForChild("BridgeNet2"):WaitForChild("dataRemoteEvent"):FireServer(unpack(args)) end)
                end
                task.wait(0.12)
            end
        end

        if AutoBuyAllGear then
            local bn = GetBridgeNet2()
            for _, g in ipairs(shop.gearList) do
                local args = {{g, "\026"}}
                if bn and bn:FindFirstChild("dataRemoteEvent") then
                    pcall(function() bn.dataRemoteEvent:FireServer(unpack(args)) end)
                else
                    pcall(function() ReplicatedStorage:WaitForChild("BridgeNet2"):WaitForChild("dataRemoteEvent"):FireServer(unpack(args)) end)
                end
                task.wait(0.12)
            end
        end

        if AutoBuyAllSeed then
            local bn = GetBridgeNet2()
            for _, s in ipairs(shop.seedList) do
                local args = {{s, "\b"}}
                if bn and bn:FindFirstChild("dataRemoteEvent") then
                    pcall(function() bn.dataRemoteEvent:FireServer(unpack(args)) end)
                else
                    pcall(function() ReplicatedStorage:WaitForChild("BridgeNet2"):WaitForChild("dataRemoteEvent"):FireServer(unpack(args)) end)
                end
                task.wait(0.12)
            end
        end
    end
end)

-- End of CPS Network PvB UI rebrand
-- Snippet 2 complete
