--[[  
    EA PRO PREMIUM HUB - Script Único
    Design inspirado em scripts pagos
    Inclui Biblioteca + Hub + Tela de Boas-vindas
]]

--============================--
--      Biblioteca EA PRO     --
--============================--

local Library = {}
Library.Tabs = {}
Library.ConfigFile = "EA_Config.txt"

Library.Theme = {
    BackgroundColor = Color3.fromRGB(25, 25, 25),
    PrimaryColor = Color3.fromRGB(0, 170, 255),
    TextColor = Color3.fromRGB(255, 255, 255)
}

Library.State = { Toggles = {}, Sliders = {} }

--// Salvar Config
function Library:SaveConfig()
    local config = {}
    config.Theme = self.Theme
    config.State = self.State
    writefile(self.ConfigFile, game:GetService("HttpService"):JSONEncode(config))
end

--// Carregar Config
function Library:LoadConfig()
    if isfile(self.ConfigFile) then
        local data = readfile(self.ConfigFile)
        local decoded = game:GetService("HttpService"):JSONDecode(data)
        self.Theme = decoded.Theme or self.Theme
        self.State = decoded.State or self.State
    end
end

--// Criar Janela
function Library:CreateWindow(Title)
    self:LoadConfig()

    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "EA_Library"
    ScreenGui.Parent = game:GetService("CoreGui")

    local MainFrame = Instance.new("Frame")
    MainFrame.Size = UDim2.new(0, 550, 0, 350)
    MainFrame.Position = UDim2.new(0.5, -275, 0.5, -175)
    MainFrame.BackgroundColor3 = self.Theme.BackgroundColor
    MainFrame.BorderSizePixel = 0
    MainFrame.Parent = ScreenGui

    local UICorner = Instance.new("UICorner", MainFrame)
    UICorner.CornerRadius = UDim.new(0, 8)

    local TabHolder = Instance.new("Frame")
    TabHolder.Size = UDim2.new(0, 140, 1, 0)
    TabHolder.BackgroundTransparency = 1
    TabHolder.Parent = MainFrame

    local UIList = Instance.new("UIListLayout", TabHolder)
    UIList.SortOrder = Enum.SortOrder.LayoutOrder

    local PageHolder = Instance.new("Frame")
    PageHolder.Size = UDim2.new(1, -140, 1, 0)
    PageHolder.Position = UDim2.new(0, 140, 0, 0)
    PageHolder.BackgroundTransparency = 1
    PageHolder.Parent = MainFrame

    local PageLayout = Instance.new("UIPageLayout", PageHolder)
    PageLayout.FillDirection = Enum.FillDirection.Horizontal
    PageLayout.SortOrder = Enum.SortOrder.LayoutOrder
    PageLayout.EasingDirection = Enum.EasingDirection.InOut
    PageLayout.EasingStyle = Enum.EasingStyle.Quad
    PageLayout.TweenTime = 0.3

    self.MainFrame = MainFrame
    self.PageLayout = PageLayout
    self.TabHolder = TabHolder
    self.PageHolder = PageHolder
    return self
end

--// Criar Aba
function Library:CreateTab(Name)
    local Button = Instance.new("TextButton")
    Button.Size = UDim2.new(1, 0, 0, 35)
    Button.Text = Name
    Button.BackgroundColor3 = self.Theme.PrimaryColor
    Button.TextColor3 = self.Theme.TextColor
    Button.Font = Enum.Font.GothamBold
    Button.TextSize = 14
    Button.Parent = self.TabHolder

    local UICorner = Instance.new("UICorner", Button)
    UICorner.CornerRadius = UDim.new(0, 6)

    local Page = Instance.new("Frame")
    Page.Size = UDim2.new(1, 0, 1, 0)
    Page.BackgroundTransparency = 1
    Page.Parent = self.PageHolder

    Button.MouseButton1Click:Connect(function()
        self.PageLayout:JumpTo(Page)
    end)

    local TabData = { Page = Page }
    table.insert(self.Tabs, TabData)
    return TabData
end

--// Criar Botão
function Library:CreateButton(Tab, Text, Callback)
    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(0, 200, 0, 30)
    Btn.Text = Text
    Btn.BackgroundColor3 = self.Theme.PrimaryColor
    Btn.TextColor3 = self.Theme.TextColor
    Btn.Font = Enum.Font.GothamBold
    Btn.TextSize = 14
    Btn.Parent = Tab.Page

    local UICorner = Instance.new("UICorner", Btn)
    UICorner.CornerRadius = UDim.new(0, 6)

    Btn.MouseButton1Click:Connect(function()
        pcall(Callback)
    end)
end

--// Criar Toggle
function Library:CreateToggle(Tab, Text, Default, Callback)
    local Toggle = Instance.new("TextButton")
    Toggle.Size = UDim2.new(0, 200, 0, 30)
    Toggle.BackgroundColor3 = self.Theme.PrimaryColor
    Toggle.TextColor3 = self.Theme.TextColor
    Toggle.Font = Enum.Font.GothamBold
    Toggle.TextSize = 14
    local state = self.State.Toggles[Text] ~= nil and self.State.Toggles[Text] or Default
    Toggle.Text = Text .. ": " .. (state and "ON" or "OFF")
    Toggle.Parent = Tab.Page

    Toggle.MouseButton1Click:Connect(function()
        state = not state
        self.State.Toggles[Text] = state
        Toggle.Text = Text .. ": " .. (state and "ON" or "OFF")
        pcall(Callback, state)
        self:SaveConfig()
    end)

    pcall(Callback, state)
end

--// Criar Slider Arrastável
function Library:CreateSlider(Tab, Text, Min, Max, Default, Callback)
    local Frame = Instance.new("Frame")
    Frame.Size = UDim2.new(0, 200, 0, 40)
    Frame.BackgroundColor3 = self.Theme.PrimaryColor
    Frame.Parent = Tab.Page

    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(1, 0, 0, 20)
    Label.BackgroundTransparency = 1
    Label.TextColor3 = self.Theme.TextColor
    Label.Font = Enum.Font.GothamBold
    Label.TextSize = 14
    local val = self.State.Sliders[Text] or Default
    Label.Text = Text .. ": " .. val
    Label.Parent = Frame

    local Slider = Instance.new("Frame")
    Slider.Size = UDim2.new(1, 0, 0, 20)
    Slider.Position = UDim2.new(0, 0, 0, 20)
    Slider.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    Slider.Parent = Frame

    local Fill = Instance.new("Frame")
    Fill.BackgroundColor3 = self.Theme.TextColor
    Fill.Size = UDim2.new((val - Min) / (Max - Min), 0, 1, 0)
    Fill.Parent = Slider

    local dragging = false

    Slider.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
        end
    end)

    Slider.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
            self.State.Sliders[Text] = val
            self:SaveConfig()
        end
    end)

    game:GetService("UserInputService").InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local relX = math.clamp((input.Position.X - Slider.AbsolutePosition.X) / Slider.AbsoluteSize.X, 0, 1)
            val = math.floor(Min + (Max - Min) * relX)
            Fill.Size = UDim2.new(relX, 0, 1, 0)
            Label.Text = Text .. ": " .. val
            pcall(Callback, val)
        end
    end)

    pcall(Callback, val)
end

--// Funções prontas
Library.Functions = {}

function Library.Functions.SpeedHack(Speed)
    local char = game.Players.LocalPlayer.Character
    if char and char:FindFirstChildOfClass("Humanoid") then
        char:FindFirstChildOfClass("Humanoid").WalkSpeed = Speed
    end
end

function Library.Functions.Fly()
    local char = game.Players.LocalPlayer.Character
    if char and char:FindFirstChildOfClass("Humanoid") then
        char:FindFirstChildOfClass("Humanoid"):ChangeState(Enum.HumanoidStateType.Physics)
    end
end

function Library.Functions.Noclip()
    game:GetService("RunService").Stepped:Connect(function()
        local char = game.Players.LocalPlayer.Character
        if char then
            for _, v in pairs(char:GetChildren()) do
                if v:IsA("BasePart") then
                    v.CanCollide = false
                end
            end
        end
    end)
end

--============================--
--         Hub Premium        --
--============================--

local Window = Library:CreateWindow("💎 EA PRO PREMIUM HUB")

-- Aba Principal
local MainTab = Library:CreateTab("🏠 Início")
Library:CreateButton(MainTab, "Ativar SpeedHack", function()
    Library.Functions.SpeedHack(100)
end)
Library:CreateToggle(MainTab, "Noclip", false, function(state)
    if state then
        Library.Functions.Noclip()
    end
end)
Library:CreateSlider(MainTab, "Velocidade", 16, 200, 16, function(val)
    Library.Functions.SpeedHack(val)
end)

-- Aba Visual
local VisualTab = Library:CreateTab("🎨 Aparência")
Library:CreateButton(VisualTab, "Tema Azul Neon", function()
    Library.Theme = { BackgroundColor = Color3.fromRGB(20, 20, 20), PrimaryColor = Color3.fromRGB(0, 255, 170), TextColor = Color3.fromRGB(255, 255, 255) }
    Library:SaveConfig()
    game:GetService("Players").LocalPlayer:Kick("Reabra o Hub para aplicar o novo tema.")
end)
Library:CreateButton(VisualTab, "Tema Roxo Royal", function()
    Library.Theme = { BackgroundColor = Color3.fromRGB(15, 15, 25), PrimaryColor = Color3.fromRGB(170, 0, 255), TextColor = Color3.fromRGB(255, 255, 255) }
    Library:SaveConfig()
    game:GetService("Players").LocalPlayer:Kick("Reabra o Hub para aplicar o novo tema.")
end)

-- Aba Funções
local FuncTab = Library:CreateTab("⚙ Funções Extras")
Library:CreateButton(FuncTab, "Ativar Fly", function()
    Library.Functions.Fly()
end)
Library:CreateButton(FuncTab, "Resetar Personagem", function()
    game.Players.LocalPlayer.Character:BreakJoints()
end)

-- Tela de boas-vindas
do
    local ScreenGui = Instance.new("ScreenGui", game:GetService("CoreGui"))
    ScreenGui.IgnoreGuiInset = true
    local Frame = Instance.new("Frame", ScreenGui)
    Frame.Size = UDim2.new(1, 0, 1, 0)
    Frame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    local Title = Instance.new("TextLabel", Frame)
    Title.AnchorPoint = Vector2.new(0.5, 0.5)
    Title.Position = UDim2.new(0.5, 0, 0.5, -30)
    Title.Size = UDim2.new(0, 500, 0, 50)
    Title.Text = "💎 EA PRO PREMIUM HUB"
    Title.Font = Enum.Font.GothamBlack
    Title.TextSize = 40
    Title.TextColor3 = Color3.fromRGB(0, 255, 170)
    Title.BackgroundTransparency = 1
    local Subtitle = Instance.new("TextLabel", Frame)
    Subtitle.AnchorPoint = Vector2.new(0.5, 0.5)
    Subtitle.Position = UDim2.new(0.5, 0, 0.5, 20)
    Subtitle.Size = UDim2.new(0, 500, 0, 30)
    Subtitle.Text = "By SeuNome - Design Premium"
    Subtitle.Font = Enum.Font.Gotham
    Subtitle.TextSize = 20
    Subtitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    Subtitle.BackgroundTransparency = 1
    task.wait(2)
    for i = 0, 1, 0.05 do
        Frame.BackgroundTransparency = i
        Title.TextTransparency = i
        Subtitle.TextTransparency = i
        task.wait(0.05)
    end
    Frame:Destroy()
end
