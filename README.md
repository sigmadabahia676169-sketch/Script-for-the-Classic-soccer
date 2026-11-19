-- ============================================================
-- THE CLASSIC SOCCER — V2 (Reach + Silent Pass Corrigidos)
-- Mobile First | Delta Executor | Android
-- ============================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local VirtualInputManager = game:GetService("VirtualInputManager")

local LocalPlayer = Players.LocalPlayer

-- ============================================================
-- CONFIGURAÇÕES
-- ============================================================
local Config = {
    ReachSize = 8,
    ReachEnabled = false,
    ReachColor = Color3.fromRGB(0, 255, 100),
    PassCooldown = 0.5,
    LastPassTime = 0,
    PassTargetDistanceMin = 8,
    PassTargetDistanceMax = 200,
    ForwardBias = 1.5, -- peso para priorizar jogadores à frente
}

local State = {
    Connections = {},
    ReachSphere = nil,
    ReachConn = nil,
    PassCooldownActive = false,
}

-- ============================================================
-- UTILITÁRIOS
-- ============================================================
local function safeCall(func, ...)
    local ok, err = pcall(func, ...)
    if not ok then warn("[TCS V2]", err) end
    return ok, err
end

local function getChar()
    return LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
end

local function getHRP(char)
    return char and char:FindFirstChild("HumanoidRootPart")
end

local function getHumanoid(char)
    return char and char:FindFirstChildOfClass("Humanoid")
end

-- ============================================================
-- INTERFACE PRINCIPAL (MENU)
-- ============================================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "TCS_V2_GUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = game.CoreGui

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 270, 0, 340)
MainFrame.Position = UDim2.new(0.5, -135, 0.5, -170)
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 22)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Parent = ScreenGui
local mc = Instance.new("UICorner", MainFrame)
mc.CornerRadius = UDim.new(0, 12)

local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 45)
TitleBar.BackgroundColor3 = Color3.fromRGB(25, 25, 38)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame
local tc = Instance.new("UICorner", TitleBar)
tc.CornerRadius = UDim.new(0, 12)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -80, 1, 0)
Title.Position = UDim2.new(0, 12, 0, 0)
Title.BackgroundTransparency = 1
Title.Font = Enum.Font.GothamBold
Title.Text = "TCS V2 — REACH + PASSE"
Title.TextColor3 = Color3.fromRGB(0, 255, 150)
Title.TextSize = 13
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TitleBar

local MinBtn = Instance.new("TextButton")
MinBtn.Size = UDim2.new(0, 32, 0, 32)
MinBtn.Position = UDim2.new(1, -68, 0, 6)
MinBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 75)
MinBtn.Text = "—"
MinBtn.TextColor3 = Color3.fromRGB(200, 200, 255)
MinBtn.Font = Enum.Font.GothamBold
MinBtn.TextSize = 16
MinBtn.BorderSizePixel = 0
MinBtn.Parent = TitleBar
local mbc = Instance.new("UICorner", MinBtn)
mbc.CornerRadius = UDim.new(0, 8)

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 32, 0, 32)
CloseBtn.Position = UDim2.new(1, -34, 0, 6)
CloseBtn.BackgroundColor3 = Color3.fromRGB(80, 30, 30)
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.fromRGB(255, 120, 120)
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 12
CloseBtn.BorderSizePixel = 0
CloseBtn.Parent = TitleBar
local cbc = Instance.new("UICorner", CloseBtn)
cbc.CornerRadius = UDim.new(0, 8)

local Content = Instance.new("ScrollingFrame")
Content.Size = UDim2.new(1, -20, 1, -55)
Content.Position = UDim2.new(0, 10, 0, 50)
Content.BackgroundTransparency = 1
Content.CanvasSize = UDim2.new(0, 0, 0, 400)
Content.ScrollBarThickness = 3
Content.ScrollBarImageColor3 = Color3.fromRGB(0, 200, 120)
Content.Parent = MainFrame

-- ============================================================
-- SEÇÃO: REACH
-- ============================================================
local ReachLabel = Instance.new("TextLabel")
ReachLabel.Size = UDim2.new(1, 0, 0, 25)
ReachLabel.Position = UDim2.new(0, 0, 0, 0)
ReachLabel.BackgroundTransparency = 1
ReachLabel.Font = Enum.Font.GothamBold
ReachLabel.Text = "⚽ REACH (ALCANCE DE CHUTE)"
ReachLabel.TextColor3 = Color3.fromRGB(100, 200, 255)
ReachLabel.TextSize = 12
ReachLabel.TextXAlignment = Enum.TextXAlignment.Left
ReachLabel.Parent = Content

local ReachToggle = Instance.new("TextButton")
ReachToggle.Size = UDim2.new(1, 0, 0, 48)
ReachToggle.Position = UDim2.new(0, 0, 0, 28)
ReachToggle.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
ReachToggle.Text = "REACH: DESATIVADO"
ReachToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
ReachToggle.Font = Enum.Font.GothamBold
ReachToggle.TextSize = 14
ReachToggle.BorderSizePixel = 0
ReachToggle.Parent = Content
local rtc = Instance.new("UICorner", ReachToggle)
rtc.CornerRadius = UDim.new(0, 8)

local SizeRow = Instance.new("Frame")
SizeRow.Size = UDim2.new(1, 0, 0, 50)
SizeRow.Position = UDim2.new(0, 0, 0, 82)
SizeRow.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
SizeRow.BorderSizePixel = 0
SizeRow.Parent = Content
local src = Instance.new("UICorner", SizeRow)
src.CornerRadius = UDim.new(0, 8)

local MinusBtn = Instance.new("TextButton")
MinusBtn.Size = UDim2.new(0, 42, 0, 38)
MinusBtn.Position = UDim2.new(0, 5, 0, 6)
MinusBtn.BackgroundColor3 = Color3.fromRGB(60, 30, 30)
MinusBtn.Text = "−"
MinusBtn.TextColor3 = Color3.fromRGB(255, 150, 150)
MinusBtn.TextSize = 20
MinusBtn.Font = Enum.Font.GothamBold
MinusBtn.BorderSizePixel = 0
MinusBtn.Parent = SizeRow
local mbc2 = Instance.new("UICorner", MinusBtn)
mbc2.CornerRadius = UDim.new(0, 6)

local SizeInput = Instance.new("TextBox")
SizeInput.Size = UDim2.new(0, 110, 0, 38)
SizeInput.Position = UDim2.new(0, 52, 0, 6)
SizeInput.BackgroundColor3 = Color3.fromRGB(10, 10, 15)
SizeInput.Text = tostring(Config.ReachSize)
SizeInput.TextColor3 = Color3.fromRGB(0, 255, 150)
SizeInput.TextSize = 16
SizeInput.Font = Enum.Font.GothamBold
SizeInput.BorderSizePixel = 0
SizeInput.Parent = SizeRow
local sic = Instance.new("UICorner", SizeInput)
sic.CornerRadius = UDim.new(0, 6)

local PlusBtn = Instance.new("TextButton")
PlusBtn.Size = UDim2.new(0, 42, 0, 38)
PlusBtn.Position = UDim2.new(0, 170, 0, 6)
PlusBtn.BackgroundColor3 = Color3.fromRGB(30, 60, 40)
PlusBtn.Text = "+"
PlusBtn.TextColor3 = Color3.fromRGB(150, 255, 180)
PlusBtn.TextSize = 20
PlusBtn.Font = Enum.Font.GothamBold
PlusBtn.BorderSizePixel = 0
PlusBtn.Parent = SizeRow
local pbc = Instance.new("UICorner", PlusBtn)
pbc.CornerRadius = UDim.new(0, 6)

local ReachStatus = Instance.new("TextLabel")
ReachStatus.Size = UDim2.new(1, 0, 0, 20)
ReachStatus.Position = UDim2.new(0, 0, 0, 136)
ReachStatus.BackgroundTransparency = 1
ReachStatus.Font = Enum.Font.Gotham
ReachStatus.Text = "Ative para visualizar a esfera de alcance."
ReachStatus.TextColor3 = Color3.fromRGB(150, 150, 180)
ReachStatus.TextSize = 10
ReachStatus.Parent = Content

-- ============================================================
-- BOTÃO FLUTUANTE DE PASSE (SEPARADO E ARRASTÁVEL)
-- ============================================================
local PassFloat = Instance.new("TextButton")
PassFloat.Name = "PassFloat"
PassFloat.Size = UDim2.new(0, 75, 0, 75)
PassFloat.Position = UDim2.new(0.85, -75, 0.75, -75)
PassFloat.BackgroundColor3 = Color3.fromRGB(0, 150, 90)
PassFloat.Text = "PASSE"
PassFloat.TextColor3 = Color3.fromRGB(255, 255, 255)
PassFloat.TextSize = 15
PassFloat.Font = Enum.Font.GothamBold
PassFloat.BorderSizePixel = 0
PassFloat.Active = true
PassFloat.Parent = ScreenGui
local pfc = Instance.new("UICorner", PassFloat)
pfc.CornerRadius = UDim.new(1, 0)

local PassStatusLabel = Instance.new("TextLabel")
PassStatusLabel.Size = UDim2.new(0, 120, 0, 22)
PassStatusLabel.Position = UDim2.new(0.5, -60, 1, 3)
PassStatusLabel.BackgroundTransparency = 1
PassStatusLabel.Font = Enum.Font.Gotham
PassStatusLabel.Text = ""
PassStatusLabel.TextColor3 = Color3.fromRGB(200, 255, 200)
PassStatusLabel.TextSize = 9
PassStatusLabel.Parent = PassFloat

-- ============================================================
-- LÓGICA DO REACH (ALCANCE DE CHUTE)
-- Estratégia: aumenta o tamanho da perna/pé + esfera visual de referência
-- O HumanoidRootPart NÃO afeta o alcance de chute no jogo
-- ============================================================
local function findLegParts(char)
    local parts = {}
    for _, name in ipairs({"Right Leg", "Left Leg", "RightFoot", "LeftFoot", "RightUpperLeg", "LeftUpperLeg"}) do
        local p = char:FindFirstChild(name)
        if p and p:IsA("BasePart") then
            table.insert(parts, p)
        end
    end
    return parts
end

local function updateReachSphere()
    if State.ReachSphere then
        State.ReachSphere:Destroy()
        State.ReachSphere = nil
    end
    if not Config.ReachEnabled then return end

    local char = getChar()
    local hrp = getHRP(char)
    if not hrp then return end

    local sphere = Instance.new("Part")
    sphere.Name = "TCS_ReachSphere"
    sphere.Shape = Enum.PartType.Ball
    sphere.Size = Vector3.new(Config.ReachSize * 2, Config.ReachSize * 2, Config.ReachSize * 2)
    sphere.Transparency = 0.75
    sphere.CanCollide = false
    sphere.Anchored = true
    sphere.CastShadow = false
    sphere.Material = Enum.Material.Neon
    sphere.Color = Config.ReachColor
    sphere.Parent = workspace
    State.ReachSphere = sphere

    -- Atualiza posição continuamente
    if State.ReachConn then State.ReachConn:Disconnect() end
    State.ReachConn = RunService.Heartbeat:Connect(function()
        if not Config.ReachEnabled or not State.ReachSphere then return end
        local c = getChar()
        local h = getHRP(c)
        if h then
            State.ReachSphere.CFrame = h.CFrame
        end
    end)
end

local function applyReach(enable)
    local char = getChar()
    if not char then return end
    local legParts = findLegParts(char)

    for _, part in ipairs(legParts) do
        if enable then
            if not part:GetAttribute("TCS_OrigSize") then
                part:SetAttribute("TCS_OrigSize", part.Size)
            end
            local orig = part:GetAttribute("TCS_OrigSize")
            if orig then
                part.Size = Vector3.new(orig.X * 1.5, orig.Y, orig.Z * 1.5)
            end
            part.CanCollide = false
        else
            local orig = part:GetAttribute("TCS_OrigSize")
            if orig then
                part.Size = orig
                part:SetAttribute("TCS_OrigSize", nil)
            end
            part.CanCollide = true
        end
    end

    updateReachSphere()
end

ReachToggle.MouseButton1Click:Connect(function()
    Config.ReachEnabled = not Config.ReachEnabled
    if Config.ReachEnabled then
        ReachToggle.Text = "REACH: ATIVADO (" .. Config.ReachSize .. ")"
        ReachToggle.BackgroundColor3 = Color3.fromRGB(0, 140, 80)
        ReachStatus.Text = "Esfera verde = alcance visual. O alcance real depende da hitbox da perna."
    else
        ReachToggle.Text = "REACH: DESATIVADO"
        ReachToggle.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
        ReachStatus.Text = "Reach desativado."
    end
    safeCall(applyReach, Config.ReachEnabled)
end)

MinusBtn.MouseButton1Click:Connect(function()
    local v = tonumber(SizeInput.Text) or Config.ReachSize
    v = math.max(2, v - 1)
    Config.ReachSize = v
    SizeInput.Text = tostring(v)
    if Config.ReachEnabled then
        ReachToggle.Text = "REACH: ATIVADO (" .. v .. ")"
        safeCall(updateReachSphere)
    end
end)

PlusBtn.MouseButton1Click:Connect(function()
    local v = tonumber(SizeInput.Text) or Config.ReachSize
    v = math.min(25, v + 1)
    Config.ReachSize = v
    SizeInput.Text = tostring(v)
    if Config.ReachEnabled then
        ReachToggle.Text = "REACH: ATIVADO (" .. v .. ")"
        safeCall(updateReachSphere)
    end
end)

SizeInput.FocusLost:Connect(function()
    local v = tonumber(SizeInput.Text)
    if v then
        v = math.clamp(v, 2, 25)
        Config.ReachSize = v
        SizeInput.Text = tostring(v)
        if Config.ReachEnabled then
            ReachToggle.Text = "REACH: ATIVADO (" .. v .. ")"
            safeCall(updateReachSphere)
        end
    else
        SizeInput.Text = tostring(Config.ReachSize)
    end
end)

-- ============================================================
-- LÓGICA DO SILENT PASS (MAPEAR COMPANHEIROS + TOOL LONG)
-- ============================================================

-- Retorna todos os companheiros de equipe vivos com HumanoidRootPart
local function getTeammates()
    local list = {}
    local myTeam = LocalPlayer.Team
    if not myTeam then return list end

    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Team == myTeam then
            local c = p.Character
            if c then
                local hum = getHumanoid(c)
                local hrp = getHRP(c)
                if hum and hum.Health > 0 and hrp then
                    table.insert(list, {
                        Player = p,
                        HRP = hrp,
                        Humanoid = hum,
                    })
                end
            end
        end
    end
    return list
end

-- Seleciona o melhor alvo: prioriza quem está mais à frente no campo
-- Considerando distância útil e evitando jogadores muito próximos
local function getBestPassTarget()
    local myChar = getChar()
    local myHRP = getHRP(myChar)
    if not myHRP then return nil end

    local myPos = myHRP.Position
    local candidates = getTeammates()
    if #candidates == 0 then return nil end

    local best = nil
    local bestScore = -math.huge

    for _, tm in ipairs(candidates) do
        local dist = (tm.HRP.Position - myPos).Magnitude
        -- Filtro de distância: não muito perto (evita passe inútil), não muito longe
        if dist >= Config.PassTargetDistanceMin and dist <= Config.PassTargetDistanceMax then
            -- Score: quanto mais à frente (Z maior) e mais próximo, melhor
            -- O eixo Z no futebol do Roblox geralmente aponta para o gol adversário
            local forwardness = tm.HRP.Position.Z - myPos.Z
            local score = (forwardness * Config.ForwardBias) - (dist * 0.5)

            -- Bônus se o companheiro estiver mais perto do gol adversário
            -- (assumindo que o gol fica no Z positivo)
            if tm.HRP.Position.Z > myPos.Z then
                score = score + 50
            end

            if score > bestScore then
                bestScore = score
                best = tm
            end
        end
    end

    -- Fallback: se ninguém passou no filtro, pega o mais à frente disponível
    if not best then
        for _, tm in ipairs(candidates) do
            local fwd = tm.HRP.Position.Z - myPos.Z
            if fwd > (best and (best.HRP.Position.Z - myPos.Z) or -math.huge) then
                best = tm
            end
        end
    end

    return best
end

-- Encontra e usa a Tool "Long" (ou "Passe" como alternativa)
local function findPassTool()
    local backpack = LocalPlayer:FindFirstChild("Backpack")
    local char = getChar()

    -- Primeiro procura "Long"
    for _, container in ipairs({backpack, char}) do
        if container then
            local tool = container:FindFirstChild("Long")
            if tool and tool:IsA("Tool") then return tool end
        end
    end
    -- Fallback: "Passe"
    for _, container in ipairs({backpack, char}) do
        if container then
            local tool = container:FindFirstChild("Passe")
            if tool and tool:IsA("Tool") then return tool end
        end
    end
    return nil
end

local function executePass()
    local now = tick()
    if now - Config.LastPassTime < Config.PassCooldown then
        PassStatusLabel.Text = "⏳ Aguarde..."
        return
    end

    local target = getBestPassTarget()
    if not target then
        PassStatusLabel.Text = "❌ Sem companheiro"
        return
    end

    local char = getChar()
    local hum = getHumanoid(char)
    if not hum then
        PassStatusLabel.Text = "❌ Sem personagem"
        return
    end

    -- Tenta equipar e ativar a Tool "Long"
    local tool = findPassTool()
    if tool then
        -- Equipa se necessário
        if tool.Parent ~= char then
            safeCall(function()
                hum:EquipTool(tool)
            end)
            task.wait(0.15)
        end

        -- Ativa a tool (simula o clique de passe)
        if tool.Parent == char then
            safeCall(function()
                tool:Activate()
            end)
            PassStatusLabel.Text = "✅ Long → " .. target.Player.Name
        else
            PassStatusLabel.Text = "⚠️ Falha ao equipar"
        end
    else
        -- Fallback: toque virtual na região do botão de passe (canto inferior direito)
        safeCall(function()
            local vp = workspace.CurrentCamera.ViewportSize
            VirtualInputManager:SendMouseButtonEvent(vp.X * 0.8, vp.Y * 0.8, 0, true, game, 0)
            task.wait(0.05)
            VirtualInputManager:SendMouseButtonEvent(vp.X * 0.8, vp.Y * 0.8, 0, false, game, 0)
        end)
        PassStatusLabel.Text = "⚠️ Tool não achada. Toque virtual → " .. target.Player.Name
    end

    Config.LastPassTime = now
end

PassFloat.MouseButton1Click:Connect(function()
    safeCall(executePass)
end)

-- ============================================================
-- DRAG DO MENU (TITLEBAR)
-- ============================================================
local dragMenu, dragStart, startPos
TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragMenu = true
        dragStart = input.Position
        startPos = MainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragMenu = false end
        end)
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragMenu and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
        local d = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
    end
end)

-- ============================================================
-- DRAG DO BOTÃO FLUTUANTE DE PASSE
-- ============================================================
local dragPass, passDragStart, passStartPos
PassFloat.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragPass = true
        passDragStart = input.Position
        passStartPos = PassFloat.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragPass = false end
        end)
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragPass and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
        local d = input.Position - passDragStart
        PassFloat.Position = UDim2.new(passStartPos.X.Scale, passStartPos.X.Offset + d.X, passStartPos.Y.Scale, passStartPos.Y.Offset + d.Y)
    end
end)

-- ============================================================
-- MINIMIZAR / FECHAR
-- ============================================================
local ReopenBtn = nil
MinBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
    if not ReopenBtn then
        ReopenBtn = Instance.new("TextButton")
        ReopenBtn.Size = UDim2.new(0, 55, 0, 55)
        ReopenBtn.Position = UDim2.new(0, 15, 0.5, -27)
        ReopenBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 120)
        ReopenBtn.Text = "⚽"
        ReopenBtn.TextSize = 24
        ReopenBtn.Font = Enum.Font.GothamBold
        ReopenBtn.BorderSizePixel = 0
        ReopenBtn.Parent = ScreenGui
        local c = Instance.new("UICorner", ReopenBtn)
        c.CornerRadius = UDim.new(1, 0)
        ReopenBtn.MouseButton1Click:Connect(function()
            MainFrame.Visible = true
            ReopenBtn:Destroy()
            ReopenBtn = nil
        end)
    end
end)

CloseBtn.MouseButton1Click:Connect(function()
    Config.ReachEnabled = false
    safeCall(applyReach, false)
    if State.ReachSphere then State.ReachSphere:Destroy() end
    if State.ReachConn then State.ReachConn:Disconnect() end
    ScreenGui:Destroy()
end)

-- ============================================================
-- INICIALIZAÇÃO
-- ============================================================
print("[TCS V2] Carregado. Reach: pernas. Pass: botão flutuante arrastável.")
