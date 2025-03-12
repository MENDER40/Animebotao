-- Icon personalizado
local ScreenGui = Instance.new("ScreenGui")
local ImageButton = Instance.new("ImageButton")
local UICorner = Instance.new("UICorner")

-- Propriedades da GUI
ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Enabled = true -- Garante que a GUI está ativada

ImageButton.Parent = ScreenGui
ImageButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
ImageButton.BorderSizePixel = 0
ImageButton.Position = UDim2.new(0.005, 0, 0.010, 0)
ImageButton.Size = UDim2.new(0, 50, 0, 50) -- Tamanho aumentado para facilitar a visualização
ImageButton.Image = "rbxassetid://139166819013498" -- ID do ícone

UICorner.Parent = ImageButton

-- Função para tornar o botão arrastável
local UIS = game:GetService("UserInputService")
local dragging, dragInput, startPos, startMousePos
local hasMoved = false

ImageButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        hasMoved = false
        startPos = ImageButton.Position
        startMousePos = UIS:GetMouseLocation()

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

ImageButton.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UIS.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = UIS:GetMouseLocation() - startMousePos
        if math.abs(delta.X) > 5 or math.abs(delta.Y) > 5 then 
            hasMoved = true
        end
        ImageButton.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y
        )
    end
end)

ImageButton.MouseButton1Down:Connect(function()
    task.wait(0.1)
    if not hasMoved then
        game:GetService("VirtualInputManager"):SendKeyEvent(true, Enum.KeyCode.LeftControl, false, game)
    end
end)

-- Carregar Fluent Library
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

if not Fluent then
    warn("Erro ao carregar Fluent Library")
    return
end

local MarketplaceService = game:GetService("MarketplaceService")
local PlaceId = game.PlaceId
local ProductInfo = MarketplaceService:GetProductInfo(PlaceId)
local GameName = ProductInfo.Name

Fluent:Notify({ 
    Title = "Script executado com sucesso", 
    Content = "Você está usando Mender_hub",
    Duration = 4 
})

local Window = Fluent:CreateWindow({
    Title = "Mender_hub",
    SubTitle = "-- " .. GameName,
    TabWidth = 102,
    Size = UDim2.fromOffset(450, 320),
    Acrylic = false,
    Theme = "Amethyst",
    MinimizeKey = Enum.KeyCode.LeftControl
})

local Tabs = {
    Info = Window:AddTab({ Title = "Info", Icon = "scroll" }),
    Farm = Window:AddTab({ Title = "Farm", Icon = "rbxassetid://18831424669" }),
    Eggs = Window:AddTab({ Title = "Hatch", Icon = "egg" }),
    Settings = Window:AddTab({ Title = "Settings", Icon = "settings" }),
}

Tabs.Info:AddParagraph({
    Title = "This script is updated daily",
    Content = "Correções de bugs"
})

-- Função para executar o ataque
local function AttackNPC(npcName)
    local args = { [1] = "Damage", [2] = npcName }
    game:GetService("ReplicatedStorage"):WaitForChild("Assets"):WaitForChild("Remotes"):WaitForChild("doMobs"):InvokeServer(unpack(args))
end

-- Lista de ilhas e NPCs correspondentes
local World_NPCs = {
    ["World 1"] = { "Dragon_1", "Dragon_2", "Dragon_3", "Dragon_4", "Dragon_5" },
    ["World 2"] = { "Leaf_1", "Leaf_2", "Leaf_3", "Leaf_4", "Leaf_5" },
    ["World 7"] = { "Piece_1", "Piece_2", "Piece_3", "Piece_4", "Piece_5" },
    ["World 8"] = { "Assassin_1", "Assassin_2", "Assassin_3", "Assassin_4", "Assassin_5" }, 
    ["World 9"] = { "Empty_1", "Empty_2", "Empty_3", "Empty_4", "Empty_5" }
}

local Selected_World = "World 1" -- Ilha inicial padrão
local Selected_NPC = World_NPCs[Selected_World][1] -- Primeiro NPC da ilha inicial

-- Criando Dropdown para escolher a Ilha
local WorldDropdown = Tabs.Farm:AddDropdown("Selecionar Ilha", {
    Title = "Escolha uma Ilha",
    Values = { "World 1", "World 2", "World 7", "World 8", "World 9" },
    Default = 1
})

-- Criando Dropdown para escolher o NPC
local NPCDropdown = Tabs.Farm:AddDropdown("Selecionar NPC", {
    Title = "Escolha um NPC",
    Values = World_NPCs[Selected_World], -- NPCs da ilha inicial
    Default = 1
})

-- Atualizar o Dropdown de NPCs ao mudar a Ilha
WorldDropdown:OnChanged(function(value)
    Selected_World = value
    NPCDropdown:SetValues(World_NPCs[Selected_World]) -- Atualiza a lista de NPCs com base na ilha escolhida
    Selected_NPC = World_NPCs[Selected_World][1] -- Define o primeiro NPC como padrão
end)

-- Atualizar o NPC selecionado quando o jogador escolher um no Dropdown
NPCDropdown:OnChanged(function(value)
    Selected_NPC = value
end)

-- Criando botão de AutoFarm
local AutoFarming = false

local AutoFarmToggle = Tabs.Farm:AddToggle("AutoFarm", { Title = "Ativar AutoFarm", Default = false })
AutoFarmToggle:OnChanged(function(value)
    AutoFarming = value -- Ativa ou desativa o AutoFarm
    while AutoFarming do
        AttackNPC(Selected_NPC) -- Ataca o NPC selecionado
        task.wait(0.1) -- Pequeno delay para evitar spam excessivo
    end
end)

-- Criando botões individuais para ataque manual
Tabs.Farm:AddButton({ Title = "Atacar NPC Selecionado", Callback = function() AttackNPC(Selected_NPC) end })


--AUTO HATH

local AutoHatch = Tabs.Eggs:AddToggle("AutoHatch", {Title = "SoloWorld", Default = false})
AutoHatch:OnChanged(function()
    while AutoHatch.Value do
        --remote
        local args = {
    [1] = "DungeonWorld",
    [2] = "Multi"
}

game:GetService("ReplicatedStorage"):WaitForChild("Assets"):WaitForChild("Remotes"):WaitForChild("openEssence"):InvokeServer(unpack(args))
        task.wait(0.5)
    end
end)


local AutoHatch = Tabs.Eggs:AddToggle("AutoHatch", {Title = "Piece Docks", Default = false})
AutoHatch:OnChanged(function()
    while AutoHatch.Value do
        --remote
        local args = {
    [1] = "PieceWorld",
    [2] = "Multi"
}

game:GetService("ReplicatedStorage"):WaitForChild("Assets"):WaitForChild("Remotes"):WaitForChild("openEssence"):InvokeServer(unpack(args))

        task.wait(0.5)
    end
end)


local AutoHatch = Tabs.Eggs:AddToggle("AutoHatch", {Title = "Assasin park", Default = false})
AutoHatch:OnChanged(function()
    while AutoHatch.Value do
        --remote
        local args = {
    [1] = "AssassinWorld",
    [2] = "Multi"
}

game:GetService("ReplicatedStorage"):WaitForChild("Assets"):WaitForChild("Remotes"):WaitForChild("openEssence"):InvokeServer(unpack(args))

        task.wait(0.5)
    end
end)

local AutoHatch = Tabs.Eggs:AddToggle("AutoHatch", {Title = "Empty Dimension", Default = false})
AutoHatch:OnChanged(function()
    while AutoHatch.Value do
        --remote
        local args = {
    [1] = "EmptyWorld",
    [2] = "Multi"
}

game:GetService("ReplicatedStorage"):WaitForChild("Assets"):WaitForChild("Remotes"):WaitForChild("openEssence"):InvokeServer(unpack(args))
        task.wait(0.5)
    end
end)

    
local Toggle = Tabs.Settings:AddToggle("AntiAFK", { Title = "Anti AFK", Default = true})

Toggle:OnChanged(function(state)
    _G.antiAFK = state

    if _G.antiAFK then
        task.spawn(function()
            while _G.antiAFK do
                game:GetService("Players").LocalPlayer.Idled:Connect(function()
                    game:GetService("VirtualUser"):CaptureController()
                    game:GetService("VirtualUser"):ClickButton2(Vector2.new())
                end)
                task.wait(1)
            end
        end)
    end
end)

