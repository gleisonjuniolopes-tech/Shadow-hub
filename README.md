-- Template de hub único e organizado (sem hacks)
-- Uso: basta colar em um LocalScript

local Players = game:GetService("Players")
local player = Players.LocalPlayer
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")

local SETTINGS_PATH = "MyHub/Settings/" .. player.Name .. ".json"
local DEFAULT_SETTINGS = {
    ["Feature A"] = false,
    ["Feature B"] = true,
    ["Speed"] = 16,
    ["SelectedTab"] = "Main"
}

local Settings = DEFAULT_SETTINGS

local function ensureFolder(path)
    if not isfolder(path) then
        makefolder(path)
    end
end

local function saveSettings()
    ensureFolder("MyHub")
    ensureFolder("MyHub/Settings")
    writefile(SETTINGS_PATH, HttpService:JSONEncode(Settings))
end

local function loadSettings()
    if isfile(SETTINGS_PATH) then
        local ok, data = pcall(function()
            return HttpService:JSONDecode(readfile(SETTINGS_PATH))
        end)

        if ok and type(data) == "table" then
            for k, v in pairs(DEFAULT_SETTINGS) do
                if data[k] == nil then
                    data[k] = v
                end
            end
            Settings = data
        end
    else
        saveSettings()
    end
end

loadSettings()

local function createScreenGui()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "MyHub"
    screenGui.ResetOnSpawn = false
    screenGui.IgnoreGuiInset = true
    screenGui.Parent = player:WaitForChild("PlayerGui")

    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 640, 0, 420)
    mainFrame.Position = UDim2.new(0.5, -320, 0.5, -210)
    mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
    mainFrame.BorderSizePixel = 0
    mainFrame.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = mainFrame

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 40)
    title.BackgroundTransparency = 1
    title.Text = "My Hub"
    title.Font = Enum.Font.GothamBold
    title.TextSize = 20
    title.TextColor3 = Color3.fromRGB(255,255,255)
    title.Parent = mainFrame

    return screenGui, mainFrame
end

local function createTab(parent, name)
    local tab = Instance.new("TextButton")
    tab.Name = name
    tab.Size = UDim2.new(0, 120, 0, 36)
    tab.Text = name
    tab.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
    tab.TextColor3 = Color3.fromRGB(255,255,255)
    tab.Font = Enum.Font.GothamSemibold
    tab.TextSize = 14
    tab.Parent = parent

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = tab

    return tab
end

local function createToggle(parent, text, defaultValue, callback)
    local toggle = Instance.new("TextButton")
    toggle.Size = UDim2.new(1, -20, 0, 32)
    toggle.BackgroundColor3 = Color3.fromRGB(48, 48, 58)
    toggle.Text = text
    toggle.TextColor3 = Color3.fromRGB(255,255,255)
    toggle.Font = Enum.Font.Gotham
    toggle.TextSize = 14
    toggle.Parent = parent

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = toggle

    local enabled = Instance.new("BoolValue")
    enabled.Value = defaultValue
    enabled.Parent = toggle

    local function updateVisual()
        if enabled.Value then
            toggle.BackgroundColor3 = Color3.fromRGB(70, 150, 90)
        else
            toggle.BackgroundColor3 = Color3.fromRGB(48, 48, 58)
        end
    end

    toggle.MouseButton1Click:Connect(function()
        enabled.Value = not enabled.Value
        updateVisual()
        if callback then
            callback(enabled.Value)
        end
    end)

    updateVisual()
    return toggle, enabled
end

local screenGui, mainFrame = createScreenGui()
local tabs = Instance.new("Frame")
tabs.Size = UDim2.new(1, -20, 0, 42)
tabs.Position = UDim2.new(0, 10, 0, 44)
tabs.BackgroundColor3 = Color3.fromRGB(35,35,40)
tabs.Parent = mainFrame

local content = Instance.new("Frame")
content.Size = UDim2.new(1, -20, 1, -70)
content.Position = UDim2.new(0, 10, 0, 70)
content.BackgroundColor3 = Color3.fromRGB(35,35,40)
content.Parent = mainFrame

local tabHolder = {}
local tabContent = {}

local function createTabPage(name)
    local page = Instance.new("ScrollingFrame")
    page.Name = name
    page.Size = UDim2.new(1, 0, 1, 0)
    page.CanvasSize = UDim2.new(0, 0, 0, 600)
    page.ScrollBarThickness = 4
    page.BackgroundTransparency = 1
    page.Visible = false
    page.Parent = content

    local layout = Instance.new("UIListLayout")
    layout.Padding = UDim.new(0, 10)
    layout.Parent = page

    return page
end

local mainTabButton = createTab(tabs, "Main")
local secondTabButton = createTab(tabs, "Tools")

local mainPage = createTabPage("Main")
local toolsPage = createTabPage("Tools")

tabHolder["Main"] = mainTabButton
tabHolder["Tools"] = secondTabButton
tabContent["Main"] = mainPage
tabContent["Tools"] = toolsPage

local function showTab(name)
    for k, v in pairs(tabContent) do
        v.Visible = (k == name)
    end
end

mainTabButton.MouseButton1Click:Connect(function()
    showTab("Main")
    Settings["SelectedTab"] = "Main"
    saveSettings()
end)

secondTabButton.MouseButton1Click:Connect(function()
    showTab("Tools")
    Settings["SelectedTab"] = "Tools"
    saveSettings()
end)

createToggle(mainPage, "Feature A", Settings["Feature A"], function(v)
    Settings["Feature A"] = v
    saveSettings()
end)

createToggle(mainPage, "Feature B", Settings["Feature B"], function(v)
    Settings["Feature B"] = v
    saveSettings()
end)

-- Exemplo de slider simples
local sliderContainer = Instance.new("Frame")
sliderContainer.Size = UDim2.new(1, -20, 0, 50)
sliderContainer.BackgroundTransparency = 1
sliderContainer.Parent = mainPage

local sliderText = Instance.new("TextLabel")
sliderText.Size = UDim2.new(0.5, 0, 1, 0)
sliderText.BackgroundTransparency = 1
sliderText.Text = "Speed: " .. Settings["Speed"]
sliderText.TextColor3 = Color3.fromRGB(255,255,255)
sliderText.Font = Enum.Font.Gotham
sliderText.TextSize = 14
sliderText.TextXAlignment = Enum.TextXAlignment.Left
sliderText.Parent = sliderContainer

local slider = Instance.new("TextBox")
slider.Size = UDim2.new(0.35, 0, 0, 28)
slider.Position = UDim2.new(0.6, 0, 0.5, -14)
slider.BackgroundColor3 = Color3.fromRGB(60,60,70)
slider.Text = tostring(Settings["Speed"])
slider.TextColor3 = Color3.fromRGB(255,255,255)
slider.Font = Enum.Font.Gotham
slider.TextSize = 14
slider.Parent = sliderContainer

slider.FocusLost:Connect(function()
    local value = tonumber(slider.Text) or Settings["Speed"]
    Settings["Speed"] = math.clamp(value, 1, 100)
    slider.Text = tostring(Settings["Speed"])
    sliderText.Text = "Speed: " .. Settings["Speed"]
    saveSettings()
end)

showTab(Settings["SelectedTab"] or "Main")

-- Exemplo de lógica funcional legítima
local function updatePlayerSpeed()
    if Settings["Feature A"] then
        player.Character.Humanoid.WalkSpeed = Settings["Speed"]
    else
        player.Character.Humanoid.WalkSpeed = 16
    end
end

player.CharacterAdded:Connect(function()
    task.wait(0.2)
    updatePlayerSpeed()
end)

while true do
    task.wait(0.2)
    if player.Character and player.Character:FindFirstChild("Humanoid") then
        updatePlayerSpeed()
    end
end
