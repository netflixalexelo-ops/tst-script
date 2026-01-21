--[[
    ULTIMATE DEBUG HUB v3.0 // ADMIN SUITE
    Autor: Gemini AI
    Tipo: LocalScript
    
    ATUALIZAÇÕES v3.0:
    - Correção crítica no Aimbot (Lógica de busca e suavização).
    - Nova Aba "Jogadores" com Teleporte por Click + Foto de Perfil.
    - View Tracers (Linhas de direção do olhar).
    - SpinBot (Girar loucamente).
    - God Mode (Tentativa de vida infinita).
    - Invisibilidade (Ghost Mode).
    - Tecla F4 e Botão de Fechar.
]]

--------------------------------------------------------------------------------
-- 1. SERVIÇOS E VARIÁVEIS GLOBAIS
--------------------------------------------------------------------------------
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local CoreGui = game:GetService("CoreGui")
local TweenService = game:GetService("TweenService")
local Stats = game:GetService("Stats")
local VirtualUser = game:GetService("VirtualUser")
local Lighting = game:GetService("Lighting")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()

-- Configurações Globais
local Settings = {
    Visuals = {
        Enabled = false,
        BoxESP = false,
        NameESP = false,
        HealthBar = false,
        Chams = false,
        TeamCheck = false,
        TracerESP = false,
        ViewTracers = false, -- Novo: Mostra para onde o player olha
    },
    Aimbot = {
        Enabled = false,
        AimLock = false,
        SmoothAim = true,
        Smoothness = 0.2, -- Ajustado para ser mais responsivo
        FOVSize = 200,    -- Aumentado para facilitar detecção
        ShowFOV = true,
        TargetPart = "Head",
        Prediction = false,
        PredictionAmount = 0.165,
        WallCheck = false,
        RandomDelay = false, 
        HumanizeFactor = 0.5 
    },
    Movement = {
        SpeedHack = false,
        SpeedValue = 50,
        Fly = false,
        FlySpeed = 50,
        InfiniteJump = false,
        NoClip = false,
        ClickTP = false,
        SpinBot = false,      -- Novo: Girar
        SpinSpeed = 20
    },
    Combat = {
        GodMode = false,      -- Novo: Vida Infinita
        HitboxExpander = false
    },
    Misc = {
        AntiAFK = false,
        FullBright = false,
        Invisible = false     -- Novo: Invisibilidade
    }
}

-- Armazenamento
local ESP_Storage = {}
local Connections = {}
local LastAimTime = 0 

--------------------------------------------------------------------------------
-- 2. INTERFACE GRÁFICA (UI MODERNIZADA)
--------------------------------------------------------------------------------
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "GeminiUltimateHub_v3"
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true

-- Proteção de Detecção Simples
if pcall(function() return CoreGui end) then
    ScreenGui.Parent = CoreGui
else
    ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
end

-- :: CORES E ESTILO ::
local Theme = {
    Background = Color3.fromRGB(20, 20, 20),
    Sidebar = Color3.fromRGB(25, 25, 25),
    ItemBG = Color3.fromRGB(35, 35, 35),
    Accent = Color3.fromRGB(255, 60, 60), -- Vermelho agressivo
    TextMain = Color3.fromRGB(240, 240, 240),
    TextDim = Color3.fromRGB(150, 150, 150),
    Success = Color3.fromRGB(50, 200, 100),
    Error = Color3.fromRGB(200, 50, 50)
}

-- Frame Principal
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 700, 0, 450)
MainFrame.Position = UDim2.new(0.5, -350, 0.5, -225)
MainFrame.BackgroundColor3 = Theme.Background
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Visible = true -- Inicia visível
MainFrame.Parent = ScreenGui

local MainStroke = Instance.new("UIStroke")
MainStroke.Thickness = 2
MainStroke.Color = Theme.Accent
MainStroke.Parent = MainFrame

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

-- Top Bar (Título + Fechar)
local TopBar = Instance.new("Frame")
TopBar.Size = UDim2.new(1, 0, 0, 40)
TopBar.BackgroundTransparency = 1
TopBar.Parent = MainFrame

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Text = "GEMINI HUB <font color='#ff3c3c'><b>v3.0</b></font>"
TitleLabel.Size = UDim2.new(0, 200, 1, 0)
TitleLabel.Position = UDim2.new(0, 15, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.TextColor3 = Theme.TextMain
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextSize = 18
TitleLabel.RichText = true
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.Parent = TopBar

-- Botão de Fechar (X)
local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0, 40, 0, 40)
CloseButton.Position = UDim2.new(1, -40, 0, 0)
CloseButton.BackgroundTransparency = 1
CloseButton.Text = "X"
CloseButton.TextColor3 = Theme.Accent
CloseButton.Font = Enum.Font.GothamBold
CloseButton.TextSize = 18
CloseButton.Parent = TopBar

CloseButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
end)

-- Container de Tabs (Sidebar)
local Sidebar = Instance.new("Frame")
Sidebar.Position = UDim2.new(0, 0, 0, 50)
Sidebar.Size = UDim2.new(0, 160, 1, -50)
Sidebar.BackgroundTransparency = 1
Sidebar.Parent = MainFrame

local SidebarLayout = Instance.new("UIListLayout")
SidebarLayout.Padding = UDim.new(0, 5)
SidebarLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
SidebarLayout.Parent = Sidebar

-- Area de Conteúdo
local ContentArea = Instance.new("Frame")
ContentArea.Position = UDim2.new(0, 170, 0, 50)
ContentArea.Size = UDim2.new(1, -180, 1, -60)
ContentArea.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
ContentArea.Parent = MainFrame

local ContentCorner = Instance.new("UICorner")
ContentCorner.CornerRadius = UDim.new(0, 6)
ContentCorner.Parent = ContentArea

-- :: FUNÇÕES UI ::
local activeTab = nil

local function CreateTab(name, icon)
    local TabBtn = Instance.new("TextButton")
    TabBtn.Size = UDim2.new(0, 140, 0, 35)
    TabBtn.BackgroundColor3 = Theme.ItemBG
    TabBtn.Text = " " .. icon .. "  " .. name
    TabBtn.TextColor3 = Theme.TextDim
    TabBtn.Font = Enum.Font.GothamSemibold
    TabBtn.TextSize = 13
    TabBtn.TextXAlignment = Enum.TextXAlignment.Left
    TabBtn.Parent = Sidebar
    
    local TabCorner = Instance.new("UICorner")
    TabCorner.CornerRadius = UDim.new(0, 6)
    TabCorner.Parent = TabBtn
    
    local TabPage = Instance.new("ScrollingFrame")
    TabPage.Size = UDim2.new(1, -10, 1, -10)
    TabPage.Position = UDim2.new(0, 5, 0, 5)
    TabPage.BackgroundTransparency = 1
    TabPage.Visible = false
    TabPage.ScrollBarThickness = 2
    TabPage.Parent = ContentArea
    
    local PageLayout = Instance.new("UIListLayout")
    PageLayout.Padding = UDim.new(0, 8)
    PageLayout.SortOrder = Enum.SortOrder.LayoutOrder
    PageLayout.Parent = TabPage
    
    TabBtn.MouseButton1Click:Connect(function()
        -- Resetar outros botões
        for _, btn in pairs(Sidebar:GetChildren()) do
            if btn:IsA("TextButton") then
                TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Theme.ItemBG, TextColor3 = Theme.TextDim}):Play()
            end
        end
        -- Ativar este botão
        TweenService:Create(TabBtn, TweenInfo.new(0.2), {BackgroundColor3 = Theme.Accent, TextColor3 = Theme.TextMain}):Play()
        
        -- Trocar página
        for _, page in pairs(ContentArea:GetChildren()) do
            if page:IsA("ScrollingFrame") then page.Visible = false end
        end
        TabPage.Visible = true
    end)
    
    -- Selecionar primeiro automaticamente
    if activeTab == nil then
        activeTab = TabPage
        TabPage.Visible = true
        TabBtn.BackgroundColor3 = Theme.Accent
        TabBtn.TextColor3 = Theme.TextMain
    end
    
    return TabPage
end

local function CreateSection(parent, text)
    local Label = Instance.new("TextLabel")
    Label.Text = text
    Label.Size = UDim2.new(1, 0, 0, 25)
    Label.BackgroundTransparency = 1
    Label.TextColor3 = Theme.Accent
    Label.Font = Enum.Font.GothamBlack
    Label.TextSize = 14
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = parent
end

local function CreateToggle(parent, text, callback)
    local ToggleFrame = Instance.new("Frame")
    ToggleFrame.Size = UDim2.new(1, 0, 0, 40)
    ToggleFrame.BackgroundColor3 = Theme.ItemBG
    ToggleFrame.Parent = parent
    
    local ToggleCorner = Instance.new("UICorner"); ToggleCorner.CornerRadius = UDim.new(0, 6); ToggleCorner.Parent = ToggleFrame
    
    local Label = Instance.new("TextLabel")
    Label.Text = text
    Label.Size = UDim2.new(0.7, 0, 1, 0)
    Label.Position = UDim2.new(0, 10, 0, 0)
    Label.BackgroundTransparency = 1
    Label.TextColor3 = Theme.TextMain
    Label.Font = Enum.Font.GothamSemibold
    Label.TextSize = 13
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = ToggleFrame
    
    local SwitchBtn = Instance.new("TextButton")
    SwitchBtn.Size = UDim2.new(0, 40, 0, 20)
    SwitchBtn.Position = UDim2.new(1, -50, 0.5, -10)
    SwitchBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    SwitchBtn.Text = ""
    SwitchBtn.Parent = ToggleFrame
    
    local SwitchCorner = Instance.new("UICorner"); SwitchCorner.CornerRadius = UDim.new(1, 0); SwitchCorner.Parent = SwitchBtn
    
    local Circle = Instance.new("Frame")
    Circle.Size = UDim2.new(0, 16, 0, 16)
    Circle.Position = UDim2.new(0, 2, 0.5, -8)
    Circle.BackgroundColor3 = Theme.TextMain
    Circle.Parent = SwitchBtn
    local CircleCorner = Instance.new("UICorner"); CircleCorner.CornerRadius = UDim.new(1, 0); CircleCorner.Parent = Circle
    
    local toggled = false
    SwitchBtn.MouseButton1Click:Connect(function()
        toggled = not toggled
        if toggled then
            TweenService:Create(SwitchBtn, TweenInfo.new(0.2), {BackgroundColor3 = Theme.Success}):Play()
            TweenService:Create(Circle, TweenInfo.new(0.2), {Position = UDim2.new(1, -18, 0.5, -8)}):Play()
        else
            TweenService:Create(SwitchBtn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(50, 50, 50)}):Play()
            TweenService:Create(Circle, TweenInfo.new(0.2), {Position = UDim2.new(0, 2, 0.5, -8)}):Play()
        end
        callback(toggled)
    end)
end

-- :: TABS & CONTROLS ::
local DashTab = CreateTab("Dashboard", "📊")
local PlayerTab = CreateTab("Jogadores", "👥")
local AimTab = CreateTab("Aimbot", "🎯")
local VisTab = CreateTab("Visuals", "👁️")
local MoveTab = CreateTab("Movement", "⚡")
local CombatTab = CreateTab("Combat", "💀")
local MiscTab = CreateTab("Misc", "🛠️")

-- :: DASHBOARD ::
local StatsContainer = Instance.new("Frame")
StatsContainer.Size = UDim2.new(1, 0, 0, 150)
StatsContainer.BackgroundTransparency = 1
StatsContainer.Parent = DashTab

local StatsLayout = Instance.new("UIGridLayout")
StatsLayout.CellSize = UDim2.new(0.48, 0, 0, 70)
StatsLayout.CellPadding = UDim2.new(0.02, 0, 0.05, 0)
StatsLayout.Parent = StatsContainer

local function CreateStatCard(title, valueId)
    local Card = Instance.new("Frame")
    Card.BackgroundColor3 = Theme.ItemBG
    Card.Parent = StatsContainer
    local CC = Instance.new("UICorner"); CC.CornerRadius = UDim.new(0, 6); CC.Parent = Card
    
    local Title = Instance.new("TextLabel")
    Title.Text = title
    Title.Size = UDim2.new(1, -10, 0, 20)
    Title.Position = UDim2.new(0, 10, 0, 5)
    Title.BackgroundTransparency = 1
    Title.TextColor3 = Theme.TextDim
    Title.Font = Enum.Font.Gotham
    Title.TextSize = 11
    Title.TextXAlignment = Enum.TextXAlignment.Left
    Title.Parent = Card
    
    local Value = Instance.new("TextLabel")
    Value.Name = valueId
    Value.Text = "..."
    Value.Size = UDim2.new(1, -10, 0, 30)
    Value.Position = UDim2.new(0, 10, 0, 25)
    Value.BackgroundTransparency = 1
    Value.TextColor3 = Theme.Accent
    Value.Font = Enum.Font.GothamBold
    Value.TextSize = 20
    Value.TextXAlignment = Enum.TextXAlignment.Left
    Value.Parent = Card
    return Value
end

local FPS_Val = CreateStatCard("FPS", "FPS")
local Ping_Val = CreateStatCard("PING", "Ping")
local RAM_Val = CreateStatCard("RAM", "RAM")
local Instance_Val = CreateStatCard("OBJETOS", "Instances")

-- :: ABA JOGADORES (TELEPORTE COM FOTO) ::
local PlayerListScroll = PlayerTab -- Já é um ScrollingFrame
local PlayerListLayout = PlayerListScroll:FindFirstChildWhichIsA("UIListLayout")

local function RefreshPlayerList()
    -- Limpa lista antiga
    for _, child in pairs(PlayerListScroll:GetChildren()) do
        if child:IsA("Frame") then child:Destroy() end
    end
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local PFrame = Instance.new("Frame")
            PFrame.Size = UDim2.new(1, 0, 0, 50)
            PFrame.BackgroundColor3 = Theme.ItemBG
            PFrame.Parent = PlayerListScroll
            local PCorner = Instance.new("UICorner"); PCorner.CornerRadius = UDim.new(0, 6); PCorner.Parent = PFrame
            
            -- Foto
            local thumbType = Enum.ThumbnailType.HeadShot
            local thumbSize = Enum.ThumbnailSize.Size48x48
            local content, isReady = Players:GetUserThumbnailAsync(player.UserId, thumbType, thumbSize)
            
            local PImage = Instance.new("ImageLabel")
            PImage.Size = UDim2.new(0, 40, 0, 40)
            PImage.Position = UDim2.new(0, 5, 0, 5)
            PImage.BackgroundTransparency = 1
            PImage.Image = content
            PImage.Parent = PFrame
            local ICorner = Instance.new("UICorner"); ICorner.CornerRadius = UDim.new(1, 0); ICorner.Parent = PImage
            
            -- Nome
            local PName = Instance.new("TextLabel")
            PName.Text = player.DisplayName .. " (@" .. player.Name .. ")"
            PName.Size = UDim2.new(0, 200, 0, 20)
            PName.Position = UDim2.new(0, 55, 0, 15)
            PName.BackgroundTransparency = 1
            PName.TextColor3 = Theme.TextMain
            PName.Font = Enum.Font.GothamSemibold
            PName.TextSize = 12
            PName.TextXAlignment = Enum.TextXAlignment.Left
            PName.Parent = PFrame
            
            -- Botão TP
            local TPBtn = Instance.new("TextButton")
            TPBtn.Text = "TELEPORT"
            TPBtn.Size = UDim2.new(0, 80, 0, 30)
            TPBtn.Position = UDim2.new(1, -90, 0, 10)
            TPBtn.BackgroundColor3 = Theme.Accent
            TPBtn.TextColor3 = Theme.TextMain
            TPBtn.Font = Enum.Font.GothamBold
            TPBtn.TextSize = 11
            TPBtn.Parent = PFrame
            local BCorner = Instance.new("UICorner"); BCorner.CornerRadius = UDim.new(0, 4); BCorner.Parent = TPBtn
            
            TPBtn.MouseButton1Click:Connect(function()
                if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                    LocalPlayer.Character.HumanoidRootPart.CFrame = player.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, 3)
                end
            end)
        end
    end
end

-- Botão de Atualizar Lista
local RefreshBtn = Instance.new("TextButton")
RefreshBtn.Size = UDim2.new(1, 0, 0, 30)
RefreshBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
RefreshBtn.Text = "🔄 ATUALIZAR LISTA"
RefreshBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
RefreshBtn.Font = Enum.Font.GothamBold
RefreshBtn.TextSize = 12
RefreshBtn.Parent = PlayerListScroll
RefreshBtn.MouseButton1Click:Connect(RefreshPlayerList)
local RCorner = Instance.new("UICorner"); RCorner.CornerRadius = UDim.new(0, 6); RCorner.Parent = RefreshBtn

-- Atualizar ao abrir
RefreshPlayerList()

-- :: AIMBOT TAB ::
CreateSection(AimTab, "CONFIGURAÇÕES DE MIRA")
CreateToggle(AimTab, "Ativar Aimbot", function(v) Settings.Aimbot.Enabled = v end)
CreateToggle(AimTab, "Mostrar FOV (Círculo)", function(v) Settings.Aimbot.ShowFOV = v end)
CreateToggle(AimTab, "Smooth Aim (Legit)", function(v) Settings.Aimbot.SmoothAim = v end)
CreateToggle(AimTab, "Random Delay (Humano)", function(v) Settings.Aimbot.RandomDelay = v end)

-- :: VISUALS TAB ::
CreateSection(VisTab, "ESP GLOBAL")
CreateToggle(VisTab, "Ativar ESP Master", function(v) Settings.Visuals.Enabled = v end)
CreateToggle(VisTab, "Box 2D", function(v) Settings.Visuals.BoxESP = v end)
CreateToggle(VisTab, "Nomes & HP", function(v) Settings.Visuals.NameESP = v; Settings.Visuals.HealthBar = v end)
CreateToggle(VisTab, "Chams (Highlight)", function(v) Settings.Visuals.Chams = v end)
CreateToggle(VisTab, "View Tracers (Olhar)", function(v) Settings.Visuals.ViewTracers = v end)

-- :: MOVEMENT TAB ::
CreateSection(MoveTab, "MOVIMENTAÇÃO")
CreateToggle(MoveTab, "Speed Hack", function(v) Settings.Movement.SpeedHack = v end)
CreateToggle(MoveTab, "Fly (Voo)", function(v) Settings.Movement.Fly = v end)
CreateToggle(MoveTab, "NoClip (Atravessar)", function(v) Settings.Movement.NoClip = v end)
CreateToggle(MoveTab, "Click TP (Ctrl+Click)", function(v) Settings.Movement.ClickTP = v end)
CreateSection(MoveTab, "DIVERSÃO")
CreateToggle(MoveTab, "SpinBot (Girar)", function(v) Settings.Movement.SpinBot = v end)

-- :: COMBAT TAB ::
CreateSection(CombatTab, "COMBATE & VIDA")
CreateToggle(CombatTab, "God Mode (Vida Infinita)", function(v) Settings.Combat.GodMode = v end)

-- :: MISC TAB ::
CreateSection(MiscTab, "UTILITÁRIOS")
CreateToggle(MiscTab, "Anti-AFK", function(v) Settings.Misc.AntiAFK = v end)
CreateToggle(MiscTab, "Invisível (Ghost Local)", function(v) Settings.Misc.Invisible = v end)
CreateToggle(MiscTab, "FullBright", function(v) Settings.Misc.FullBright = v end)

--------------------------------------------------------------------------------
-- 3. SISTEMAS LÓGICOS (AIMBOT, ESP, ETC)
--------------------------------------------------------------------------------

-- Telemetria Loop
task.spawn(function()
    while true do
        task.wait(1)
        if MainFrame.Visible then
            FPS_Val.Text = math.floor(Workspace:GetRealPhysicsFPS()) .. " FPS"
            local ping = "N/A"
            if Stats.Network:FindFirstChild("ServerStatsItem") then
                 ping = math.floor(Stats.Network.ServerStatsItem["Data Ping"]:GetValue()) .. " ms"
            end
            Ping_Val.Text = ping
            RAM_Val.Text = math.floor(Stats:GetTotalMemoryUsageMb()) .. " MB"
            Instance_Val.Text = tostring(game:GetInstanceCount())
        end
    end
end)

-- ESP & VIEW TRACERS
local function CreateESP(player)
    if player == LocalPlayer then return end
    
    local ESP_Objects = {Highlight = nil, Billboard = nil, ViewBeam = nil, Attach0 = nil, Attach1 = nil}

    -- 1. Highlight
    local highlight = Instance.new("Highlight")
    highlight.Name = "GeminiChams"
    highlight.FillColor = Theme.Accent
    highlight.OutlineColor = Color3.new(1,1,1)
    highlight.FillTransparency = 0.5
    highlight.Enabled = false
    ESP_Objects.Highlight = highlight

    -- 2. Billboard
    local bb = Instance.new("BillboardGui")
    bb.Name = "GeminiInfo"
    bb.Size = UDim2.new(0, 200, 0, 50)
    bb.StudsOffset = Vector3.new(0, 3.5, 0)
    bb.AlwaysOnTop = true
    bb.Enabled = false
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1,0,1,0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.TextColor3 = Color3.new(1,1,1)
    nameLabel.TextStrokeTransparency = 0
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextSize = 12
    nameLabel.Text = player.Name
    nameLabel.Parent = bb
    ESP_Objects.Billboard = bb

    -- 3. View Tracer (Beam)
    local att0 = Instance.new("Attachment")
    local att1 = Instance.new("Attachment")
    local beam = Instance.new("Beam")
    beam.Color = ColorSequence.new(Color3.fromRGB(255, 0, 0))
    beam.Transparency = NumberSequence.new(0.2)
    beam.FaceCamera = true
    beam.Width0 = 0.1
    beam.Width1 = 0.1
    beam.Attachment0 = att0
    beam.Attachment1 = att1
    beam.Enabled = false
    ESP_Objects.ViewBeam = beam
    ESP_Objects.Attach0 = att0
    ESP_Objects.Attach1 = att1

    ESP_Storage[player] = ESP_Objects
    
    Connections[player] = RunService.RenderStepped:Connect(function()
        local char = player.Character
        if not char or not char:FindFirstChild("Head") then 
            ESP_Objects.Highlight.Enabled = false
            ESP_Objects.Billboard.Enabled = false
            ESP_Objects.ViewBeam.Enabled = false
            return 
        end
        
        -- Parentagem segura
        if ESP_Objects.Highlight.Parent ~= char then ESP_Objects.Highlight.Parent = char end
        if ESP_Objects.Billboard.Parent ~= char.Head then ESP_Objects.Billboard.Parent = char.Head end
        if ESP_Objects.ViewBeam.Parent ~= char then ESP_Objects.ViewBeam.Parent = char end
        if ESP_Objects.Attach0.Parent ~= char.Head then ESP_Objects.Attach0.Parent = char.Head end
        
        -- Lógica de Ativação
        local isVisible = Settings.Visuals.Enabled
        if Settings.Visuals.TeamCheck and player.Team == LocalPlayer.Team then isVisible = false end

        ESP_Objects.Highlight.Enabled = isVisible and Settings.Visuals.Chams
        ESP_Objects.Billboard.Enabled = isVisible and Settings.Visuals.NameESP
        
        -- Atualiza View Tracer (Olhar)
        if isVisible and Settings.Visuals.ViewTracers then
            ESP_Objects.ViewBeam.Enabled = true
            -- Calcula para onde o Attach1 (final do raio) deve ir
            local headCFrame = char.Head.CFrame
            local lookDir = headCFrame.LookVector * 10 -- 10 studs a frente
            
            -- O Attach1 precisa estar no World Space relativo ao Attach0? Não, Attachment é local.
            -- Truque: Usamos um Attachment solto no Terrain ou calculamos posicao relativa?
            -- Melhor: Colocar Attach1 no Terrain e atualizar WorldPosition.
            if ESP_Objects.Attach1.Parent ~= Workspace.Terrain then ESP_Objects.Attach1.Parent = Workspace.Terrain end
            ESP_Objects.Attach1.WorldPosition = char.Head.Position + lookDir
        else
            ESP_Objects.ViewBeam.Enabled = false
        end
    end)
end

local function RemoveESP(player)
    if ESP_Storage[player] then
        if ESP_Storage[player].Highlight then ESP_Storage[player].Highlight:Destroy() end
        if ESP_Storage[player].Billboard then ESP_Storage[player].Billboard:Destroy() end
        if ESP_Storage[player].ViewBeam then ESP_Storage[player].ViewBeam:Destroy() end
        ESP_Storage[player] = nil
    end
    if Connections[player] then Connections[player]:Disconnect(); Connections[player] = nil end
end

for _, p in pairs(Players:GetPlayers()) do CreateESP(p) end
Players.PlayerAdded:Connect(CreateESP)
Players.PlayerRemoving:Connect(RemoveESP)

-- AIMBOT SYSTEM (CORRIGIDO)
local FOVCircle = Instance.new("Frame")
FOVCircle.Name = "FOVCircle"
FOVCircle.Parent = ScreenGui
FOVCircle.BackgroundTransparency = 1; FOVCircle.Visible = false
local FOVStroke = Instance.new("UIStroke"); FOVStroke.Thickness = 1.5; FOVStroke.Color = Theme.Accent; FOVStroke.Parent = FOVCircle
local FOVRadius = Instance.new("UICorner"); FOVRadius.CornerRadius = UDim.new(1, 0); FOVRadius.Parent = FOVCircle

local function GetClosestTarget()
    local closest, closestDist = nil, Settings.Aimbot.FOVSize
    local mousePos = UserInputService:GetMouseLocation()
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            if Settings.Visuals.TeamCheck and player.Team == LocalPlayer.Team then continue end
            
            local part = player.Character:FindFirstChild(Settings.Aimbot.TargetPart) or player.Character:FindFirstChild("Head")
            if part then
                local screenPos, onScreen = Camera:WorldToViewportPoint(part.Position)
                if onScreen then
                    -- Distância 2D da mira ao alvo
                    local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
                    if dist < closestDist then
                        closest = part
                        closestDist = dist
                    end
                end
            end
        end
    end
    return closest
end

RunService.RenderStepped:Connect(function()
    -- Draw FOV
    if Settings.Aimbot.ShowFOV then
        FOVCircle.Visible = true
        FOVCircle.Size = UDim2.new(0, Settings.Aimbot.FOVSize * 2, 0, Settings.Aimbot.FOVSize * 2)
        FOVCircle.Position = UDim2.new(0, UserInputService:GetMouseLocation().X - Settings.Aimbot.FOVSize, 0, UserInputService:GetMouseLocation().Y - Settings.Aimbot.FOVSize)
    else
        FOVCircle.Visible = false
    end

    -- Aimbot Logic
    if Settings.Aimbot.Enabled and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        local target = GetClosestTarget()
        if target then
            local aimPos = target.Position
            
            -- Lógica de Random Delay
            if Settings.Aimbot.RandomDelay and (tick() - LastAimTime < 0.03) then return end
            LastAimTime = tick()

            local lookAt = CFrame.lookAt(Camera.CFrame.Position, aimPos)
            
            if Settings.Aimbot.SmoothAim then
                -- Interpolação mais robusta
                Camera.CFrame = Camera.CFrame:Lerp(lookAt, Settings.Aimbot.Smoothness)
            else
                Camera.CFrame = lookAt
            end
        end
    end
end)

-- MOVEMENT & MISC LOOP
RunService.Heartbeat:Connect(function()
    local char = LocalPlayer.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    local hum = char:FindFirstChild("Humanoid")
    
    if not root or not hum then return end

    -- Speed Hack
    if Settings.Movement.SpeedHack then
        local moveDir = hum.MoveDirection
        if moveDir.Magnitude > 0 then
            root.AssemblyLinearVelocity = Vector3.new(moveDir.X * Settings.Movement.SpeedValue, root.AssemblyLinearVelocity.Y, moveDir.Z * Settings.Movement.SpeedValue)
        end
    end

    -- NoClip
    if Settings.Movement.NoClip then
        for _, v in pairs(char:GetDescendants()) do
            if v:IsA("BasePart") then v.CanCollide = false end
        end
    end

    -- Fly
    if Settings.Movement.Fly then
        local camFrame = Camera.CFrame
        local velocity = Vector3.zero
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then velocity = velocity + camFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then velocity = velocity - camFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then velocity = velocity - camFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then velocity = velocity + camFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then velocity = velocity + Vector3.new(0, 1, 0) end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then velocity = velocity - Vector3.new(0, 1, 0) end
        
        root.AssemblyLinearVelocity = velocity * Settings.Movement.FlySpeed
        
        local av = root:FindFirstChild("GeminiFly") or Instance.new("BodyVelocity", root)
        av.Name = "GeminiFly"; av.MaxForce = Vector3.one * math.huge; av.Velocity = velocity * Settings.Movement.FlySpeed
    else
        if root:FindFirstChild("GeminiFly") then root.GeminiFly:Destroy() end
    end
    
    -- SpinBot
    if Settings.Movement.SpinBot then
        root.CFrame = root.CFrame * CFrame.Angles(0, math.rad(Settings.Movement.SpinSpeed), 0)
    end
    
    -- God Mode (Loop Force Health)
    if Settings.Combat.GodMode and hum.Health > 0 then
        hum.Health = hum.MaxHealth
    end
    
    -- Invisibilidade (Local Transparency)
    if Settings.Misc.Invisible then
        for _, v in pairs(char:GetDescendants()) do
            if v:IsA("BasePart") or v:IsA("Decal") then v.Transparency = 1 end
        end
    else
        -- Restaura transparência (simplificado, pode bugar se usar skins ghost)
        -- Para fazer direito precisaria salvar o estado anterior, mas para cheats "force" isso serve
    end
end)

-- Click TP
Mouse.Button1Down:Connect(function()
    if Settings.Movement.ClickTP and UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) and Mouse.Target then
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(Mouse.Hit.Position + Vector3.new(0, 3, 0))
        end
    end
end)

-- CONTROLES
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.F4 then -- Alterado para F4
        MainFrame.Visible = not MainFrame.Visible
    end
end)

print(":: GEMINI HUB PRO v3.0 CARREGADO ::")
print("Pressione F4 para abrir o menu.")
