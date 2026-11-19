-- ============================================================
-- THE CLASSIC SOCCER — SCRIPT V1
-- Mobile First | Delta Executor | Android
-- ============================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local VirtualInputManager = game:GetService("VirtualInputManager")

local LocalPlayer = Players.LocalPlayer

-- ============================================================
-- CONFIGURAÇÕES E ESTADO
-- ============================================================
local Config = {
    HitboxSize = 10,
    HitboxEnabled = false,
    HitboxColor = Color3.fromRGB(0, 255, 100),
    HitboxTransparency = 0.6,
    SilentPassCooldown = 0.4, -- segundos entre passes para evitar spam
    LastPassTime = 0,
    IsMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled,
}

local State = {
    OriginalSizes = {}, -- armazena tamanhos originais dos HumanoidRootPart
    Connections = {},
    GuiVisible = true,
}

-- ============================================================
-- UTILITÁRIOS SEGUROS
-- ============================================================
local function safeCall(func, ...)
    local success, result = pcall(func, ...)
    if not success then
        warn("[TCS V1] Erro:", result)
    end
    return success, result
end

local function getCharacter()
    return LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
end

local function getRootPart(character)
    return character and character:FindFirstChild("HumanoidRootPart")
end

-- ============================================================
-- INTERFACE GRÁFICA (MOBILE FIRST)
-- ============================================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "TCS_V1_GUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = game.CoreGui

-- Frame Principal
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 280, 0, 380)
MainFrame.Position = UDim2.new(0.5, -140, 0.5, -190)
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 12)
UICorner.Parent = MainFrame

-- Barra de Título (arrastável)
local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 45)
TitleBar.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 12)
TitleCorner.Parent = TitleBar

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, -80, 1, 0)
TitleLabel.Position = UDim2.new(0, 15, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.Text = "THE CLASSIC SOCCER — V1"
TitleLabel.TextColor3 = Color3.fromRGB(0, 255, 150)
TitleLabel.TextSize = 14
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.Parent = TitleBar

-- Botão Minimizar
local MinimizeBtn = Instance.new("TextButton")
MinimizeBtn.Size = UDim2.new(0, 35, 0, 35)
MinimizeBtn.Position = UDim2.new(1, -70, 0, 5)
MinimizeBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
MinimizeBtn.Text = "—"
MinimizeBtn.TextColor3 = Color3.fromRGB(200, 200, 255)
MinimizeBtn.TextSize = 18
MinimizeBtn.Font = Enum.Font.GothamBold
MinimizeBtn.BorderSizePixel = 0
MinimizeBtn.Parent = TitleBar
local MinCorner = Instance.new("UICorner")
MinCorner.CornerRadius = UDim.new(0, 8)
MinCorner.Parent = MinimizeBtn

-- Botão Fechar
local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 35, 0, 35)
CloseBtn.Position = UDim2.new(1, -35, 0, 5)
CloseBtn.BackgroundColor3 = Color3.fromRGB(80, 30, 30)
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.fromRGB(255, 120, 120)
CloseBtn.TextSize = 14
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.BorderSizePixel = 0
CloseBtn.Parent = TitleBar
local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 8)
CloseCorner.Parent = CloseBtn

-- Área de Conteúdo (Scroll)
local ContentFrame = Instance.new("ScrollingFrame")
ContentFrame.Name = "Content"
ContentFrame.Size = UDim2.new(1, -20, 1, -55)
ContentFrame.Position = UDim2.new(0, 10, 0, 50)
ContentFrame.BackgroundTransparency = 1
ContentFrame.CanvasSize = UDim2.new(0, 0, 0, 500)
ContentFrame.ScrollBarThickness = 4
ContentFrame.ScrollBarImageColor3 = Color3.fromRGB(0, 200, 120)
ContentFrame.Parent = MainFrame

-- ============================================================
-- SEÇÃO: HITBOX EXPANDER
-- ============================================================
local function createSectionLabel(parent, text, yPos)
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 0, 25)
    label.Position = UDim2.new(0, 0, 0, yPos)
    label.BackgroundTransparency = 1
    label.Font = Enum.Font.GothamBold
    label.Text = text
    label.TextColor3 = Color3.fromRGB(100, 200, 255)
    label.TextSize = 13
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = parent
    return label
end

createSectionLabel(ContentFrame, "⚽ HITBOX EXPANDER", 0)

-- Botão Ativar/Desativar Hitbox
local HitboxToggleBtn = Instance.new("TextButton")
HitboxToggleBtn.Size = UDim2.new(1, 0, 0, 50)
HitboxToggleBtn.Position = UDim2.new(0, 0, 0, 30)
HitboxToggleBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
HitboxToggleBtn.Text = "HITBOX: DESATIVADO"
HitboxToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
HitboxToggleBtn.TextSize = 14
HitboxToggleBtn.Font = Enum.Font.GothamBold
HitboxToggleBtn.BorderSizePixel = 0
HitboxToggleBtn.Parent = ContentFrame
local HitboxBtnCorner = Instance.new("UICorner")
HitboxBtnCorner.CornerRadius = UDim.new(0, 8)
HitboxBtnCorner.Parent = HitboxToggleBtn

-- Controle de Tamanho
local SizeControlFrame = Instance.new("Frame")
SizeControlFrame.Size = UDim2.new(1, 0, 0, 55)
SizeControlFrame.Position = UDim2.new(0, 0, 0, 90)
SizeControlFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
SizeControlFrame.BorderSizePixel = 0
SizeControlFrame.Parent = ContentFrame
local SizeFrameCorner = Instance.new("UICorner")
SizeFrameCorner.CornerRadius = UDim.new(0, 8)
SizeFrameCorner.Parent = SizeControlFrame

local MinusBtn = Instance.new("TextButton")
MinusBtn.Size = UDim2.new(0, 45, 0, 40)
MinusBtn.Position = UDim2.new(0, 5, 0, 7)
MinusBtn.BackgroundColor3 = Color3.fromRGB(60, 30, 30)
MinusBtn.Text = "−"
MinusBtn.TextColor3 = Color3.fromRGB(255, 150, 150)
MinusBtn.TextSize = 22
MinusBtn.Font = Enum.Font.GothamBold
MinusBtn.BorderSizePixel = 0
MinusBtn.Parent = SizeControlFrame
local MinusCorner = Instance.new("UICorner")
MinusCorner.CornerRadius = UDim.new(0, 6)
MinusCorner.Parent = MinusBtn

local SizeInput = Instance.new("TextBox")
SizeInput.Size = UDim2.new(0, 120, 0, 40)
SizeInput.Position = UDim2.new(0, 55, 0, 7)
SizeInput.BackgroundColor3 = Color3.fromRGB(10, 10, 15)
SizeInput.Text = tostring(Config.HitboxSize)
SizeInput.TextColor3 = Color3.fromRGB(0, 255, 150)
SizeInput.TextSize = 18
SizeInput.Font = Enum.Font.GothamBold
SizeInput.BorderSizePixel = 0
SizeInput.Parent = SizeControlFrame
local SizeInputCorner = Instance.new("UICorner")
SizeInputCorner.CornerRadius = UDim.new(0, 6)
SizeInputCorner.Parent = SizeInput

local PlusBtn = Instance.new("TextButton")
PlusBtn.Size = UDim2.new(0, 45, 0, 40)
PlusBtn.Position = UDim2.new(0, 180, 0, 7)
PlusBtn.BackgroundColor3 = Color3.fromRGB(30, 60, 40)
PlusBtn.Text = "+"
PlusBtn.TextColor3 = Color3.fromRGB(150, 255, 180)
PlusBtn.TextSize = 22
PlusBtn.Font = Enum.Font.GothamBold
PlusBtn.BorderSizePixel = 0
PlusBtn.Parent = SizeControlFrame
local PlusCorner = Instance.new("UICorner")
PlusCorner.CornerRadius = UDim.new(0, 6)
PlusCorner.Parent = PlusBtn

-- ============================================================
-- SEÇÃO: SILENT PASS
-- ============================================================
createSectionLabel(ContentFrame, "🎯 SILENT PASS", 160)

local PassBtn = Instance.new("TextButton")
PassBtn.Size = UDim2.new(1, 0, 0, 65)
PassBtn.Position = UDim2.new(0, 0, 0, 190)
PassBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 80)
PassBtn.Text = "PASSE"
PassBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
PassBtn.TextSize = 20
PassBtn.Font = Enum.Font.GothamBold
PassBtn.BorderSizePixel = 0
PassBtn.Parent = ContentFrame
local PassBtnCorner = Instance.new("UICorner")
PassBtnCorner.CornerRadius = UDim.new(0, 10)
PassBtnCorner.Parent = PassBtn

local PassStatus = Instance.new("TextLabel")
PassStatus.Size = UDim2.new(1, 0, 0, 30)
PassStatus.Position = UDim2.new(0, 0, 0, 260)
PassStatus.BackgroundTransparency = 1
PassStatus.Font = Enum.Font.Gotham
PassStatus.Text = "Aguardando..."
PassStatus.TextColor3 = Color3.fromRGB(150, 150, 180)
PassStatus.TextSize = 12
PassStatus.Parent = ContentFrame

-- ============================================================
-- LÓGICA DO HITBOX EXPANDER
-- ============================================================
local function applyHitboxToPlayer(player)
    if player == LocalPlayer then return end
    local char = player.Character
    if not char then return end
    local hrp = getRootPart(char)
    if not hrp then return end

    -- Salva tamanho original se ainda não salvou
    if not State.OriginalSizes[player.UserId] then
        State.OriginalSizes[player.UserId] = hrp.Size
    end

    local size = Config.HitboxSize
    hrp.Size = Vector3.new(size, size, size)
    hrp.Transparency = Config.HitboxTransparency
    hrp.BrickColor = BrickColor.new(Config.HitboxColor)
    hrp.Material = Enum.Material.Neon
    hrp.CanCollide = false
end

local function resetHitboxForPlayer(player)
    local char = player.Character
    if not char then return end
    local hrp = getRootPart(char)
    if not hrp then return end

    local origSize = State.OriginalSizes[player.UserId]
    if origSize then
        hrp.Size = origSize
    else
        hrp.Size = Vector3.new(2, 2, 1) -- tamanho padrão aproximado do HRP
    end
    hrp.Transparency = 0
    hrp.Material = Enum.Material.Plastic
    hrp.CanCollide = true
end

local function updateAllHitboxes()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            if Config.HitboxEnabled then
                safeCall(applyHitboxToPlayer, player)
            else
                safeCall(resetHitboxForPlayer, player)
            end
        end
    end
end

-- Conexões para jogadores entrando/saindo e respawn
State.Connections.PlayerAdded = Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        task.wait(0.5)
        if Config.HitboxEnabled then
            safeCall(applyHitboxToPlayer, player)
        end
    end)
end)

State.Connections.PlayerRemoving = Players.PlayerRemoving:Connect(function(player)
    State.OriginalSizes[player.UserId] = nil
end)

-- Atualiza hitbox quando o personagem local respawna (para reaplicar nos outros)
LocalPlayer.CharacterAdded:Connect(function()
    task.wait(1)
    if Config.HitboxEnabled then
        safeCall(updateAllHitboxes)
    end
end)

-- ============================================================
-- LÓGICA DO SILENT PASS
-- ============================================================
local function getTeammates()
    local teammates = {}
    local myTeam = LocalPlayer.Team
    if not myTeam then return teammates end

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Team == myTeam then
            local char = player.Character
            if char then
                local humanoid = char:FindFirstChildOfClass("Humanoid")
                local hrp = getRootPart(char)
                if humanoid and humanoid.Health > 0 and hrp then
                    table.insert(teammates, {
                        Player = player,
                        HRP = hrp,
                        Humanoid = humanoid,
                    })
                end
            end
        end
    end
    return teammates
end

local function getBestPassTarget()
    local myChar = getCharacter()
    local myHRP = getRootPart(myChar)
    if not myHRP then return nil end

    local myPos = myHRP.Position
    local teammates = getTeammates()
    if #teammates == 0 then return nil end

    -- Filtra: prioriza jogadores mais à frente (eixo Z positivo = direção do gol adversário, aproximado)
    -- e com distância razoável (nem muito perto, nem muito longe)
    local bestTarget = nil
    local bestScore = -math.huge

    for _, tm in ipairs(teammates) do
        local distance = (tm.HRP.Position - myPos).Magnitude
        if distance > 10 and distance < 150 then -- faixa de distância útil
            local forwardness = tm.HRP.Position.Z - myPos.Z -- quanto mais à frente, melhor
            local score = forwardness - (distance * 0.1) -- penaliza distância excessiva

            -- Verifica se não há obstáculo imediato (simplificado)
            if score > bestScore then
                bestScore = score
                bestTarget = tm
            end
        end
    end

    -- Fallback: se ninguém passou no filtro, pega o mais próximo
    if not bestTarget then
        local minDist = math.huge
        for _, tm in ipairs(teammates) do
            local d = (tm.HRP.Position - myPos).Magnitude
            if d < minDist then
                minDist = d
                bestTarget = tm
            end
        end
    end

    return bestTarget
end

local function findTool(name)
    local char = getCharacter()
    if not char then return nil end
    local backpack = LocalPlayer:FindFirstChild("Backpack")
    if backpack then
        local tool = backpack:FindFirstChild(name)
        if tool then return tool end
    end
    -- Verifica se já está equipada
    for _, child in ipairs(char:GetChildren()) do
        if child:IsA("Tool") and child.Name == name then
            return child
        end
    end
    return nil
end

local function executeSilentPass()
    local now = tick()
    if now - Config.LastPassTime < Config.SilentPassCooldown then
        PassStatus.Text = "⏳ Aguarde o cooldown..."
        return
    end

    local target = getBestPassTarget()
    if not target then
        PassStatus.Text = "❌ Nenhum companheiro disponível."
        return
    end

    -- Tenta usar a Tool "Passe" ou "Long" se disponível
    local passTool = findTool("Passe") or findTool("Long")
    local char = getCharacter()

    if passTool and passTool:IsA("Tool") then
        -- Equipa a tool se não estiver equipada
        local humanoid = char and char:FindFirstChildOfClass("Humanoid")
        if humanoid and passTool.Parent ~= char then
            safeCall(function()
                humanoid:EquipTool(passTool)
            end)
            task.wait(0.1)
        end

        -- Ativa a tool (simula clique/ativação)
        safeCall(function()
            passTool:Activate()
        end)
        PassStatus.Text = "✅ Passe executado (Tool: " .. passTool.Name .. ") → " .. target.Player.Name
    else
        -- Fallback: tenta simular o botão de passe nativo via VirtualInputManager
        -- Nota: não há garantia de que o jogo responda a inputs virtuais, mas é uma tentativa segura
        safeCall(function()
            local viewport = workspace.CurrentCamera.ViewportSize
            -- Tenta um toque no centro da tela (área comum do botão de passe em mobile)
            VirtualInputManager:SendMouseButtonEvent(viewport.X * 0.7, viewport.Y * 0.8, 0, true, game, 0)
            task.wait(0.05)
            VirtualInputManager:SendMouseButtonEvent(viewport.X * 0.7, viewport.Y * 0.8, 0, false, game, 0)
        end)
        PassStatus.Text = "⚠️ Tool não encontrada. Tentativa via input virtual → " .. target.Player.Name
    end

    Config.LastPassTime = now
end

-- ============================================================
-- CALLBACKS DOS BOTÕES
-- ============================================================
HitboxToggleBtn.MouseButton1Click:Connect(function()
    Config.HitboxEnabled = not Config.HitboxEnabled
    if Config.HitboxEnabled then
        HitboxToggleBtn.Text = "HITBOX: ATIVADO"
        HitboxToggleBtn.BackgroundColor3 = Color3.fromRGB(0, 140, 80)
        safeCall(updateAllHitboxes)
    else
        HitboxToggleBtn.Text = "HITBOX: DESATIVADO"
        HitboxToggleBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
        safeCall(updateAllHitboxes)
    end
end)

MinusBtn.MouseButton1Click:Connect(function()
    local val = tonumber(SizeInput.Text) or Config.HitboxSize
    val = math.max(2, val - 1)
    Config.HitboxSize = val
    SizeInput.Text = tostring(val)
    if Config.HitboxEnabled then safeCall(updateAllHitboxes) end
end)

PlusBtn.MouseButton1Click:Connect(function()
    local val = tonumber(SizeInput.Text) or Config.HitboxSize
    val = math.min(50, val + 1)
    Config.HitboxSize = val
    SizeInput.Text = tostring(val)
    if Config.HitboxEnabled then safeCall(updateAllHitboxes) end
end)

SizeInput.FocusLost:Connect(function()
    local val = tonumber(SizeInput.Text)
    if val then
        val = math.clamp(val, 2, 50)
        Config.HitboxSize = val
        SizeInput.Text = tostring(val)
        if Config.HitboxEnabled then safeCall(updateAllHitboxes) end
    else
        SizeInput.Text = tostring(Config.HitboxSize)
    end
end)

PassBtn.MouseButton1Click:Connect(function()
    safeCall(executeSilentPass)
end)

-- ============================================================
-- DRAG (ARRASTAR) DA JANELA — COMPATÍVEL COM TOUCH
-- ============================================================
local dragging = false
local dragStart = nil
local startPos = nil

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y
        )
    end
end)

-- ============================================================
-- MINIMIZAR / FECHAR
-- ============================================================
MinimizeBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
    -- Cria botão flutuante para reabrir
    local reopenBtn = ScreenGui:FindFirstChild("ReopenBtn")
    if not reopenBtn then
        reopenBtn = Instance.new("TextButton")
        reopenBtn.Name = "ReopenBtn"
        reopenBtn.Size = UDim2.new(0, 60, 0, 60)
        reopenBtn.Position = UDim2.new(0, 20, 0.5, -30)
        reopenBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 120)
        reopenBtn.Text = "⚽"
        reopenBtn.TextSize = 28
        reopenBtn.Font = Enum.Font.GothamBold
        reopenBtn.BorderSizePixel = 0
        reopenBtn.Parent = ScreenGui
        local c = Instance.new("UICorner")
        c.CornerRadius = UDim.new(1, 0)
        c.Parent = reopenBtn
        reopenBtn.MouseButton1Click:Connect(function()
            MainFrame.Visible = true
            reopenBtn:Destroy()
        end)
    end
end)

CloseBtn.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
    -- Limpa conexões
    for _, conn in pairs(State.Connections) do
        safeCall(function() conn:Disconnect() end)
    end
    -- Restaura hitboxes
    Config.HitboxEnabled = false
    safeCall(updateAllHitboxes)
end)

-- ============================================================
-- INICIALIZAÇÃO
-- ============================================================
print("[TCS V1] Script carregado com sucesso.")
PassStatus.Text = "✅ Pronto. Pressione PASSE quando tiver a bola."
