-- Script: Hop Los Brothers (com bordas arredondadas, fundo cinza, botões brancos e bolinha transparente)
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")
local player = Players.LocalPlayer

-- Verifica se já existe para não duplicar
if player.PlayerGui:FindFirstChild("HopLosBrothers") then
    player.PlayerGui:FindFirstChild("HopLosBrothers"):Destroy()
end

-- Criar ScreenGui
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "HopLosBrothers"
ScreenGui.Parent = player:WaitForChild("PlayerGui")
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- Criar Frame principal
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 300, 0, 200)
MainFrame.Position = UDim2.new(0.5, -150, 0.5, -100)
MainFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 50) -- cinza
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

-- Bordas arredondadas
local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 12)
MainCorner.Parent = MainFrame

-- Criar título
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -90, 0, 30)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.BackgroundTransparency = 1
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 18
Title.Text = "Hop Los Brothers"
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = MainFrame

-- Função para criar botões com estilo
local function CriarBotao(texto, posX)
    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(0, 60, 0, 28)
    Btn.Position = UDim2.new(0, posX, 0, 5)
    Btn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    Btn.TextColor3 = Color3.fromRGB(0, 0, 0)
    Btn.Font = Enum.Font.GothamBold
    Btn.TextSize = 14
    Btn.Text = texto
    Btn.Parent = MainFrame

    -- Bordas arredondadas
    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(0, 8)
    Corner.Parent = Btn

    return Btn
end

-- Criar botões
local FullscreenBtn = CriarBotao("Tela Cheia", 100)
local MinimizeBtn = CriarBotao("Minimizar", 170)
local CloseBtn = CriarBotao("Fechar", 240)

-- Criar botão principal
local Botao = Instance.new("TextButton")
Botao.Size = UDim2.new(0, 180, 0, 45)
Botao.Position = UDim2.new(0.5, -90, 0.5, -20)
Botao.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
Botao.TextColor3 = Color3.new(0, 0, 0)
Botao.Font = Enum.Font.GothamBold
Botao.TextSize = 22
Botao.Text = "Server Hop"
Botao.Parent = MainFrame

local BotaoCorner = Instance.new("UICorner")
BotaoCorner.CornerRadius = UDim.new(0, 10)
BotaoCorner.Parent = Botao

-- Criar texto TikTok
local TikTokLabel = Instance.new("TextLabel")
TikTokLabel.Size = UDim2.new(1, 0, 0, 20)
TikTokLabel.Position = UDim2.new(0, 0, 1, -25)
TikTokLabel.BackgroundTransparency = 1
TikTokLabel.TextColor3 = Color3.new(1, 1, 1)
TikTokLabel.Font = Enum.Font.Gotham
TikTokLabel.TextSize = 16
TikTokLabel.Text = "TikTok: LosBrothers.ofc"
TikTokLabel.Parent = MainFrame

-- Criar bolha para quando minimizar
local BubbleButton = Instance.new("TextButton")
BubbleButton.Size = UDim2.new(0, 40, 0, 40)
BubbleButton.Position = UDim2.new(0, 20, 1, -60)
BubbleButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
BubbleButton.BackgroundTransparency = 0.5 -- 50% transparente
BubbleButton.Text = "+"
BubbleButton.TextColor3 = Color3.fromRGB(0, 0, 0)
BubbleButton.Visible = false
BubbleButton.Parent = ScreenGui

local BubbleCorner = Instance.new("UICorner")
BubbleCorner.CornerRadius = UDim.new(1, 0)
BubbleCorner.Parent = BubbleButton

-- Função arrastar bolha
local UserInputService = game:GetService("UserInputService")
local dragging, dragInput, dragStart, startPos

local function updateInput(input, object)
    local delta = input.Position - dragStart
    object.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

local function makeDraggable(object)
    object.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = object.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    object.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            updateInput(input, object)
        end
    end)
end

makeDraggable(BubbleButton)

-- Função buscar servidor
local function BuscarServidorDiferente()
    local servidores = {}
    local cursor
    repeat
        local url = string.format("https://games.roblox.com/v1/games/%d/servers/Public?sortOrder=Asc&limit=100%s", game.PlaceId, cursor and "&cursor=" .. cursor or "")
        local sucesso, resultado = pcall(function()
            return HttpService:JSONDecode(game:HttpGet(url))
        end)
        if sucesso and resultado and resultado.data then
            for _, servidor in ipairs(resultado.data) do
                if servidor.playing < servidor.maxPlayers and servidor.id ~= game.JobId then
                    table.insert(servidores, servidor)
                end
            end
            cursor = resultado.nextPageCursor
        else
            break
        end
    until not cursor or #servidores > 0

    if #servidores > 0 then
        return servidores[math.random(1, #servidores)].id
    end
    return nil
end

-- Evento botão principal
Botao.MouseButton1Click:Connect(function()
    local servidorId = BuscarServidorDiferente()
    if servidorId then
        TeleportService:TeleportToPlaceInstance(game.PlaceId, servidorId, player)
    else
        Botao.Text = "Nenhum encontrado!"
        task.wait(2)
        Botao.Text = "Server Hop"
    end
end)

-- Evento minimizar
MinimizeBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
    BubbleButton.Visible = true
end)

-- Evento maximizar
BubbleButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = true
    BubbleButton.Visible = false
end)

-- Evento tela cheia
local isFullscreen = false
FullscreenBtn.MouseButton1Click:Connect(function()
    if not isFullscreen then
        MainFrame.Size = UDim2.new(1, 0, 1, 0)
        MainFrame.Position = UDim2.new(0, 0, 0, 0)
        isFullscreen = true
        FullscreenBtn.Text = "Normal"
    else
        MainFrame.Size = UDim2.new(0, 300, 0, 200)
        MainFrame.Position = UDim2.new(0.5, -150, 0.5, -100)
        isFullscreen = false
        FullscreenBtn.Text = "Tela Cheia"
    end
end)

-- Evento fechar
CloseBtn.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)
