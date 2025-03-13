local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- Criando a UI (Interface Gráfica)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game.CoreGui

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 200, 0, 100)
Frame.Position = UDim2.new(0.5, -100, 0, 50)
Frame.BackgroundColor3 = Color3.new(0, 0, 0)
Frame.BackgroundTransparency = 0.5
Frame.Parent = ScreenGui
Frame.Active = true
Frame.Draggable = true

local Button = Instance.new("TextButton")
Button.Size = UDim2.new(1, 0, 0.5, 0)
Button.Position = UDim2.new(0, 0, 0, 0)
Button.Text = "Ativar ESP"
Button.BackgroundColor3 = Color3.new(1, 1, 1)
Button.Parent = Frame

local Status = Instance.new("TextLabel")
Status.Size = UDim2.new(1, 0, 0.5, 0)
Status.Position = UDim2.new(0, 0, 0.5, 0)
Status.Text = "ESP Desativado"
Status.BackgroundColor3 = Color3.new(1, 1, 1)
Status.Parent = Frame

local espAtivo = false

-- Função para criar ESP em um jogador
local function criarESP(jogador)
    if jogador == LocalPlayer then return end  -- Ignorar o próprio jogador

    local character = jogador.Character
    if not character then return end

    local head = character:FindFirstChild("Head")
    if not head then return end

    -- Criar BillboardGui para nome
    local billGui = Instance.new("BillboardGui")
    billGui.Size = UDim2.new(0, 200, 0, 50)
    billGui.Adornee = head
    billGui.StudsOffset = Vector3.new(0, 2, 0)
    billGui.Parent = head

    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(1, 0, 1, 0)
    textLabel.BackgroundTransparency = 1
    textLabel.TextScaled = true
    textLabel.TextStrokeTransparency = 0
    textLabel.Parent = billGui

    -- Criar linha de trace (Beam)
    local attachment1 = Instance.new("Attachment", head)
    local attachment2 = Instance.new("Attachment", LocalPlayer.Character.Head)

    local beam = Instance.new("Beam")
    beam.Attachment0 = attachment1
    beam.Attachment1 = attachment2
    beam.Width0 = 0.1
    beam.Width1 = 0.1
    beam.Parent = head

    -- Definir cores baseadas no time
    if jogador.Team == LocalPlayer.Team then
        textLabel.Text = "[Aliado] " .. jogador.Name
        textLabel.TextColor3 = Color3.new(0, 1, 0)
        beam.Color = ColorSequence.new(Color3.new(0, 1, 0))
    else
        textLabel.Text = "[Inimigo] " .. jogador.Name
        textLabel.TextColor3 = Color3.new(1, 0, 0)
        beam.Color = ColorSequence.new(Color3.new(1, 0, 0))
    end

    return billGui, beam
end

-- Ativar/desativar ESP
local function ativarESP()
    if espAtivo then
        -- Desativar ESP
        for _, player in pairs(Players:GetPlayers()) do
            if player.Character then
                local head = player.Character:FindFirstChild("Head")
                if head then
                    for _, obj in pairs(head:GetChildren()) do
                        if obj:IsA("BillboardGui") or obj:IsA("Beam") or obj:IsA("Attachment") then
                            obj:Destroy()
                        end
                    end
                end
            end
        end
        Status.Text = "ESP Desativado"
        Button.Text = "Ativar ESP"
        espAtivo = false
    else
        -- Ativar ESP
        for _, player in pairs(Players:GetPlayers()) do
            criarESP(player)
        end
        Status.Text = "ESP Ativado"
        Button.Text = "Desativar ESP"
        espAtivo = true
    end
end

Button.MouseButton1Click:Connect(ativarESP)

-- Atualizar ESP quando novos jogadores entrarem
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        if espAtivo then
            criarESP(player)
        end
    end)
end)
