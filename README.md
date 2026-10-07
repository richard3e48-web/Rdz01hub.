local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
local Window = Rayfield:CreateWindow({
Name = "Rdz01 Hub",
Icon = 0, -- Icon in Topbar. Can use Lucide Icons (string) or Roblox Image (number). 0 to use no icon (default).
LoadingTitle = " Carregando seu Hub..",
LoadingSubtitle = "by Rdz",
ShowText = "Open Rdz01 hub", -- for mobile users to unhide Rayfield, change if you'd like
Theme = "DarkBlue", -- Check https://docs.sirius.menu/rayfield/configuration/themes

ToggleUIKeybind = "Q", -- The keybind to toggle the UI visibility (string like "K" or Enum.KeyCode)

DisableRayfieldPrompts = false,
DisableBuildWarnings = false, -- Prevents Rayfield from emitting warnings when the script has a version mismatch with the interface.

-- Heartbeat = "https://www.sentivel.com/api/heartbeat/<your token>", -- Pings your Sentivel heartbeat while Rayfield is open, so you can see whether your script is running

-- ScriptID = "sid_xxxxxxxxxxxx", -- Your Script ID from developer.sirius.menu — enables analytics, managed keys, and script hosting

ConfigurationSaving = {
Enabled = true,
FolderName = nil, -- Create a custom folder for your hub/game
FileName = "Rdz01 Hub"
},

Discord = {
Enabled = false, -- Prompt the user to join your Discord server if their executor supports it
Invite = "noinvitelink", -- The Discord invite code, do not include Discord.gg/. E.g. Discord.gg/ABCD would be ABCD
RememberJoins = true -- Set this to false to make them join the Discord every time they load it up
},

KeySystem = true, -- Set this to true to use our key system
KeySettings = {
Title = "Rdz01 Key",
Subtitle = "My Key",
Note = "Key grátis: EM01", -- Use this to tell the user how to get a key
FileName = "Key", -- It is recommended to use something unique, as other scripts using Rayfield may overwrite your key file
SaveKey = true, -- The user's key will be saved, but if you change the key, they will be unable to use your script
GrabKeyFromSite = false, -- If this is true, set Key below to the RAW site you would like Rayfield to get the key from
Key = {"EM01"} -- List of keys that the system will accept, can be RAW file links (pastebin, github, etc.) or simple strings ("hello", "key22")
}
})
local Window = Rayfield:CreateWindow({
-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local Player = Players.LocalPlayer

-- =========================
-- PLAYER
-- =========================

local PlayerTab = Window:CreateTab("Player", 4483362458)

-- SPEED
local SpeedValue = 16

PlayerTab:CreateSlider({
    Name = "Speed",
    Range = {1, 100000000000000},
    Increment = 1,
    Suffix = " Speed",
    CurrentValue = 16,
    Flag = "PlayerSpeed",
    Callback = function(Value)
        SpeedValue = Value

        local Character = Player.Character
        local Humanoid = Character and Character:FindFirstChildOfClass("Humanoid")

        if Humanoid then
            Humanoid.WalkSpeed = Value
        end
    end,
})

-- Mantém a velocidade após respawn
Player.CharacterAdded:Connect(function(Character)
    local Humanoid = Character:WaitForChild("Humanoid")
    Humanoid.WalkSpeed = SpeedValue
end)

-- =========================
-- FLY
-- =========================

local Flying = false
local FlyConnection

local function StopFly()
    Flying = false

    if FlyConnection then
        FlyConnection:Disconnect()
        FlyConnection = nil
    end

    local Character = Player.Character
    local Root = Character and Character:FindFirstChild("HumanoidRootPart")

    if Root then
        Root.AssemblyLinearVelocity = Vector3.zero
    end
end

local function StartFly()
    if Flying then return end

    Flying = true

    FlyConnection = RunService.Heartbeat:Connect(function()
        if not Flying then return end

        local Character = Player.Character
        local Root = Character and Character:FindFirstChild("HumanoidRootPart")

        if not Root then return end

        local Camera = workspace.CurrentCamera
        local MoveDirection = Vector3.zero

        if UserInputService:IsKeyDown(Enum.KeyCode.W) then
            MoveDirection += Camera.CFrame.LookVector
        end

        if UserInputService:IsKeyDown(Enum.KeyCode.S) then
            MoveDirection -= Camera.CFrame.LookVector
        end

        if UserInputService:IsKeyDown(Enum.KeyCode.A) then
            MoveDirection -= Camera.CFrame.RightVector
        end

        if UserInputService:IsKeyDown(Enum.KeyCode.D) then
            MoveDirection += Camera.CFrame.RightVector
        end

        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
            MoveDirection += Vector3.new(0, 1, 0)
        end

        if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
            MoveDirection -= Vector3.new(0, 1, 0)
        end

        if MoveDirection.Magnitude > 0 then
            MoveDirection = MoveDirection.Unit
        end

        Root.AssemblyLinearVelocity = MoveDirection * 100
    end)
end

PlayerTab:CreateToggle({
    Name = "Fly",
    CurrentValue = false,
    Flag = "Fly",
    Callback = function(Value)
        if Value then
            StartFly()
        else
            StopFly()
        end
    end,
})

-- =========================
-- HIGHLIGHT
-- =========================

local HighlightEnabled = false
local Highlights = {}

local function AddHighlight(TargetPlayer)
    if TargetPlayer == Player then return end

    local Character = TargetPlayer.Character
    if not Character then return end

    local Humanoid = Character:FindFirstChildOfClass("Humanoid")
    if not Humanoid then return end

    if Highlights[TargetPlayer] then
        Highlights[TargetPlayer]:Destroy()
    end

    local Highlight = Instance.new("Highlight")
    Highlight.Name = "RDZ01_Highlight"
    Highlight.Adornee = Character
    Highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    Highlight.Parent = Character

    local Billboard = Instance.new("BillboardGui")
    Billboard.Name = "RDZ01_Name"
    Billboard.Adornee = Character:FindFirstChild("Head")
    Billboard.Size = UDim2.fromOffset(200, 40)
    Billboard.StudsOffset = Vector3.new(0, 3, 0)
    Billboard.AlwaysOnTop = true
    Billboard.Parent = Character

    local Text = Instance.new("TextLabel")
    Text.BackgroundTransparency = 1
    Text.Size = UDim2.fromScale(1, 1)
    Text.Text = TargetPlayer.DisplayName .. " (@" .. TargetPlayer.Name .. ")"
    Text.TextScaled = true
    Text.TextStrokeTransparency = 0
    Text.Parent = Billboard

    Highlights[TargetPlayer] = Highlight
    Highlights[TargetPlayer .. "_Billboard"] = Billboard
end

local function RemoveHighlight(TargetPlayer)
    if Highlights[TargetPlayer] then
        Highlights[TargetPlayer]:Destroy()
        Highlights[TargetPlayer] = nil
    end

    local Billboard = Highlights[TargetPlayer .. "_Billboard"]

    if Billboard then
        Billboard:Destroy()
        Highlights[TargetPlayer .. "_Billboard"] = nil
    end
end

local function UpdateHighlights()
    for _, TargetPlayer in ipairs(Players:GetPlayers()) do
        if TargetPlayer ~= Player then
            if HighlightEnabled then
                AddHighlight(TargetPlayer)
            else
                RemoveHighlight(TargetPlayer)
            end
        end
    end
end

PlayerTab:CreateToggle({
    Name = "Highlight",
    CurrentValue = false,
    Flag = "Highlight",
    Callback = function(Value)
        HighlightEnabled = Value
        UpdateHighlights()
    end,
})

Players.PlayerAdded:Connect(function(TargetPlayer)
    TargetPlayer.CharacterAdded:Connect(function()
        task.wait(0.5)

        if HighlightEnabled then
            AddHighlight(TargetPlayer)
        end
    end)
end)

Players.PlayerRemoving:Connect(function(TargetPlayer)
    RemoveHighlight(TargetPlayer)
end)
