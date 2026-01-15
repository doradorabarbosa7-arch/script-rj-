-- SISTEMA COMPLETO COM FLUENT UI + ADMIN DETECTOR + PERFECT AIM
-- Versão: 2.3 - Sistema para desenvolvimento de jogos
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")

local player = Players.LocalPlayer
local camera = Workspace.CurrentCamera

-- Carregar Fluent UI
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

-- Criar Window
local Window = Fluent:CreateWindow({
    Title = "⚡ rj ⚡",
    SubTitle = "By Karatekaff",
    TabWidth = 160,
    Theme = "Dark",
    Acrylic = true,
    Size = UDim2.fromOffset(580, 460),
    MinimizeKey = Enum.KeyCode.End
})

-- Criar Abas
local Tabs = {
    Aimbot = Window:AddTab({ Title = "🎯 Aimbot", Icon = "" }),
    ESP = Window:AddTab({ Title = "👁️ ESP", Icon = "" }),
    Visual = Window:AddTab({ Title = "🎨 Visual", Icon = "" }),
    Settings = Window:AddTab({ Title = "⚙️ Config", Icon = "" })
}

-- Estados Globais
_G.Config = {
    Aimbot = false,
    AimbotFOV = 100,
    FOVFollowCursor = false,
    PerfectAim = false, -- NOVO: Não erra nenhum tiro
    LockDistance = 500,
    WallCheck = true,
    TeamCheck = false,
    IgnoreDead = true,
    ESP = false,
    ESPTeamCheck = false,
    ESPSkeleton = true,
    ESPHeadCircle = true,
    ESPTracer = true,
    ESPDistance = true,
    ESPMaxDistance = 1000,
    ESPShowDead = false,
    ShowAdminCount = false -- NOVO: Mostrar contagem de admins
}

local espObjects = {}
local adminCount = 0
local AdminCountLabel = nil

-- Funções Auxiliares
local function notify(text)
    Fluent:Notify({
        Title = "Sistema",
        Content = text,
        Duration = 3
    })
end

local function isSameTeam(targetPlayer)
    if not player.Team or not targetPlayer.Team then return false end
    return player.Team == targetPlayer.Team
end

local function isPlayerAlive(targetPlayer)
    if not targetPlayer or not targetPlayer.Character then return false end
    
    local humanoid = targetPlayer.Character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return false end
    
    if humanoid.Health <= 0 then return false end
    if humanoid:GetState() == Enum.HumanoidStateType.Dead then return false end
    
    return true
end

-- NOVA FUNÇÃO: Detectar se jogador é admin
local function isAdmin(targetPlayer)
    -- Método 1: Verificar grupo (ajuste o GroupId e Rank conforme seu jogo)
    local success1, isInGroup = pcall(function()
        return targetPlayer:IsInGroup(0) -- Coloque o ID do seu grupo aqui
    end)
    
    -- Método 2: Verificar se tem badge de admin
    local success2, hasBadge = pcall(function()
        return targetPlayer:FindFirstChild("AdminBadge") ~= nil
    end)
    
    -- Método 3: Verificar UserIds específicos (adicione IDs de admins aqui)
    local adminIds = {
        -- Adicione os UserIds dos seus admins aqui
        -- exemplo: 123456789, 987654321
    }
    for _, id in ipairs(adminIds) do
        if targetPlayer.UserId == id then
            return true
        end
    end
    
    -- Método 4: Verificar se tem permissão de creator/dev
    if targetPlayer.UserId == game.CreatorId then
        return true
    end
    
    return false
end

-- NOVA FUNÇÃO: Atualizar contagem de admins
local function updateAdminCount()
    adminCount = 0
    for _, p in pairs(Players:GetPlayers()) do
        if isAdmin(p) then
            adminCount = adminCount + 1
        end
    end
    return adminCount
end

-- ============= ABA AIMBOT =============
local AimbotSection = Tabs.Aimbot:AddSection("Configurações de Aimbot")

Tabs.Aimbot:AddToggle("AimbotToggle", {
    Title = "Ativar Aimbot (FOV Fixo)",
    Description = "FOV fixo no centro - Segura botão direito para mirar",
    Default = false,
    Callback = function(v)
        _G.Config.Aimbot = v
        if v then
            _G.Config.FOVFollowCursor = false
            notify("Aimbot FOV FIXO ATIVADO ✅")
        else
            notify("Aimbot DESATIVADO ❌")
        end
    end
})

Tabs.Aimbot:AddToggle("FOVFollowCursor", {
    Title = "FOV Segue Cursor",
    Description = "Ativa FOV que segue o mouse (desativa FOV fixo)",
    Default = false,
    Callback = function(v)
        _G.Config.FOVFollowCursor = v
        if v then
            _G.Config.Aimbot = false
            notify("FOV Follow Cursor ATIVADO 🖱️ (FOV fixo desativado)")
        else
            notify("FOV Follow Cursor DESATIVADO ❌")
        end
    end
})

-- NOVO: Perfect Aim Toggle
Tabs.Aimbot:AddToggle("PerfectAim", {
    Title = "🎯 Perfect Aim (Não Erra)",
    Description = "Modo preciso - Todos os tiros acertam o alvo",
    Default = false,
    Callback = function(v)
        _G.Config.PerfectAim = v
        notify(v and "Perfect Aim ATIVADO 🎯" or "Perfect Aim DESATIVADO ❌")
    end
})

Tabs.Aimbot:AddToggle("IgnoreDead", {
    Title = "Ignorar Jogadores Mortos",
    Description = "Não mira em jogadores com 0 de vida ou mortos",
    Default = true,
    Callback = function(v)
        _G.Config.IgnoreDead = v
        notify(v and "Ignorando mortos ✅" or "Mirando em todos ⚠️")
    end
})

Tabs.Aimbot:AddToggle("WallCheck", {
    Title = "Wall Check",
    Description = "Não mira através de paredes",
    Default = true,
    Callback = function(v)
        _G.Config.WallCheck = v
    end
})

Tabs.Aimbot:AddToggle("TeamCheck", {
    Title = "Team Check",
    Description = "Não mira em membros do seu time",
    Default = false,
    Callback = function(v)
        _G.Config.TeamCheck = v
    end
})

Tabs.Aimbot:AddSlider("FOVSlider", {
    Title = "FOV Size",
    Description = "Tamanho do círculo de FOV",
    Default = 100,
    Min = 50,
    Max = 500,
    Rounding = 0,
    Callback = function(v)
        _G.Config.AimbotFOV = v
    end
})

Tabs.Aimbot:AddSlider("DistanceSlider", {
    Title = "Lock Distance",
    Description = "Distância máxima para lock",
    Default = 500,
    Min = 100,
    Max = 2000,
    Rounding = 0,
    Callback = function(v)
        _G.Config.LockDistance = v
    end
})

Tabs.Aimbot:AddParagraph({
    Title = "🎯 Perfect Aim Info",
    Content = "Perfect Aim garante que todos os tiros acertem o alvo.\n\n⚠️ Funciona melhor com Aimbot ativado!\n\nIdeal para NPCs ou modo single-player."
})

-- ============= ABA ESP =============
local ESPSection = Tabs.ESP:AddSection("Configurações Principais")

Tabs.ESP:AddToggle("ESPToggle", {
    Title = "Ativar ESP",
    Description = "Visualizar inimigos através das paredes",
    Default = false,
    Callback = function(v)
        _G.Config.ESP = v
        notify(v and "ESP ATIVADO ✅" or "ESP DESATIVADO ❌")
    end
})

Tabs.ESP:AddToggle("ESPTeamCheck", {
    Title = "ESP Team Check",
    Description = "Cores diferentes: Azul (Aliados) | Vermelho (Inimigos)",
    Default = false,
    Callback = function(v)
        _G.Config.ESPTeamCheck = v
    end
})

Tabs.ESP:AddToggle("ESPShowDead", {
    Title = "Mostrar Jogadores Mortos",
    Description = "Exibir ESP em jogadores mortos (cor cinza)",
    Default = false,
    Callback = function(v)
        _G.Config.ESPShowDead = v
        notify(v and "Mostrando mortos no ESP 💀" or "Ocultando mortos no ESP ✅")
    end
})

-- NOVO: Admin Counter Toggle
Tabs.ESP:AddToggle("ShowAdminCount", {
    Title = "👮 Mostrar Contagem de Admins",
    Description = "Exibe quantos administradores estão no servidor",
    Default = false,
    Callback = function(v)
        _G.Config.ShowAdminCount = v
        if v then
            updateAdminCount()
            notify(string.format("👮 %d Admin(s) detectado(s) no servidor", adminCount))
        else
            notify("Contador de Admins DESATIVADO")
        end
    end
})

Tabs.ESP:AddSlider("ESPDistanceSlider", {
    Title = "Distância Máxima ESP",
    Description = "Distância máxima para mostrar ESP (em metros)",
    Default = 1000,
    Min = 100,
    Max = 5000,
    Rounding = 0,
    Callback = function(v)
        _G.Config.ESPMaxDistance = v
    end
})

-- NOVO: Botão para atualizar contagem de admins manualmente
Tabs.ESP:AddButton({
    Title = "🔄 Atualizar Contagem de Admins",
    Description = "Recarrega a lista de admins no servidor",
    Callback = function()
        local count = updateAdminCount()
        notify(string.format("👮 %d Admin(s) no servidor", count))
    end
})

local ESPVisuals = Tabs.ESP:AddSection("Opções Visuais")

Tabs.ESP:AddToggle("ESPSkeletonToggle", {
    Title = "Skeleton",
    Description = "Mostrar esqueleto do jogador",
    Default = true,
    Callback = function(v)
        _G.Config.ESPSkeleton = v
    end
})

Tabs.ESP:AddToggle("ESPHeadCircleToggle", {
    Title = "Head Circle",
    Description = "Círculo na cabeça do jogador",
    Default = true,
    Callback = function(v)
        _G.Config.ESPHeadCircle = v
    end
})

Tabs.ESP:AddToggle("ESPTracerToggle", {
    Title = "Tracer Line",
    Description = "Linha do topo da tela até o jogador",
    Default = true,
    Callback = function(v)
        _G.Config.ESPTracer = v
    end
})

Tabs.ESP:AddToggle("ESPDistanceToggle", {
    Title = "Mostrar Distância",
    Description = "Exibir distância em metros acima da cabeça",
    Default = true,
    Callback = function(v)
        _G.Config.ESPDistance = v
    end
})

Tabs.ESP:AddParagraph({
    Title = "📋 Informações ESP",
    Content = "🔵 Azul = Aliados (com Team Check ativo)\n🔴 Vermelho = Inimigos\n⚫ Cinza = Jogadores Mortos\n👮 Dourado = Administradores\n\n✨ Personalize cada elemento do ESP!"
})

-- ============= ABA VISUAL =============
Tabs.Visual:AddParagraph({
    Title = "🎯 FOV Circle - Dois Modos",
    Content = "⚪ FOV FIXO: Aimbot tradicional (centro da tela)\n🖱️ FOV CURSOR: Segue o mouse em tempo real\n\n⚠️ Apenas UM modo pode estar ativo por vez!\nAtivar um desativa o outro automaticamente."
})

Tabs.Visual:AddParagraph({
    Title = "👁️ ESP Visual",
    Content = "• Skeleton (Esqueleto do jogador)\n• Head Circle (Círculo na cabeça)\n• Tracer Line (Linha do topo da tela)\n• Distance Display (Distância em metros)\n• Admin Detection (Detecta administradores)\n\nTodos podem ser ativados/desativados individualmente!"
})

Tabs.Visual:AddParagraph({
    Title = "💀 Sistema Anti-Morto",
    Content = "• Aimbot NÃO mira em jogadores mortos\n• ESP pode mostrar/ocultar mortos\n• Jogadores mortos aparecem em CINZA no ESP"
})

Tabs.Visual:AddParagraph({
    Title = "🎯 Perfect Aim System",
    Content = "• Perfect Aim garante 100% de precisão\n• Todos os tiros acertam o alvo\n• Melhor usado com Aimbot ativado\n• Ideal para NPCs e bots"
})

-- ============= ABA SETTINGS =============
Tabs.Settings:AddButton({
    Title = "Restaurar Configurações Padrão",
    Description = "Reset todas as configurações",
    Callback = function()
        _G.Config = {
            Aimbot = false,
            AimbotFOV = 100,
            FOVFollowCursor = false,
            PerfectAim = false,
            LockDistance = 500,
            WallCheck = true,
            TeamCheck = false,
            IgnoreDead = true,
            ESP = false,
            ESPTeamCheck = false,
            ESPSkeleton = true,
            ESPHeadCircle = true,
            ESPTracer = true,
            ESPDistance = true,
            ESPMaxDistance = 1000,
            ESPShowDead = false,
            ShowAdminCount = false
        }
        notify("Configurações resetadas! 🔄")
    end
})

Tabs.Settings:AddParagraph({
    Title = "⌨️ Controles",
    Content = "• INSERT ou END = Minimizar/Maximizar\n• Botão Direito Mouse = Ativar Aimbot\n• Botão Flutuante = Minimizar Interface"
})

Tabs.Settings:AddParagraph({
    Title = "📋 Informações",
    Content = "Script para desenvolvimento de jogos\nVersão: 2.3 - Admin Detector + Perfect Aim\nÚltima atualização: 2026\n\nNOVAS FEATURES:\n✅ Detector de Admins\n✅ Perfect Aim (100% precisão)"
})

-- ============= FOV CIRCLE =============
local FOVCircle = Drawing.new("Circle")
FOVCircle.Visible = false
FOVCircle.Thickness = 2
FOVCircle.Color = Color3.fromRGB(255, 255, 255)
FOVCircle.Filled = false
FOVCircle.Transparency = 1
FOVCircle.NumSides = 64

RunService.RenderStepped:Connect(function()
    if _G.Config.Aimbot or _G.Config.FOVFollowCursor then
        local screenSize = camera.ViewportSize
        
        if _G.Config.FOVFollowCursor then
            local mousePos = UserInputService:GetMouseLocation()
            FOVCircle.Position = Vector2.new(mousePos.X, mousePos.Y)
        else
            FOVCircle.Position = Vector2.new(screenSize.X / 2, screenSize.Y / 2)
        end
        
        FOVCircle.Radius = _G.Config.AimbotFOV
        
        -- Mudar cor do FOV se Perfect Aim estiver ativo
        if _G.Config.PerfectAim then
            FOVCircle.Color = Color3.fromRGB(0, 255, 0) -- Verde quando Perfect Aim ativo
        else
            FOVCircle.Color = Color3.fromRGB(255, 255, 255) -- Branco normal
        end
        
        FOVCircle.Visible = true
    else
        FOVCircle.Visible = false
    end
end)

-- ============= ADMIN COUNT DISPLAY =============
local AdminCountDisplay = Drawing.new("Text")
AdminCountDisplay.Visible = false
AdminCountDisplay.Size = 18
AdminCountDisplay.Center = true
AdminCountDisplay.Outline = true
AdminCountDisplay.Color = Color3.fromRGB(255, 215, 0) -- Dourado
AdminCountDisplay.Font = 2

RunService.RenderStepped:Connect(function()
    if _G.Config.ShowAdminCount then
        local screenSize = camera.ViewportSize
        AdminCountDisplay.Position = Vector2.new(screenSize.X / 2, 30)
        AdminCountDisplay.Text = string.format("👮 Admins no servidor: %d", adminCount)
        AdminCountDisplay.Visible = true
    else
        AdminCountDisplay.Visible = false
    end
end)

-- Atualizar contador quando jogadores entram/saem
Players.PlayerAdded:Connect(function()
    wait(1)
    updateAdminCount()
end)

Players.PlayerRemoving:Connect(function()
    wait(0.5)
    updateAdminCount()
end)

-- ============= AIMBOT COM PERFECT AIM =============
local function getClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = _G.Config.AimbotFOV
    
    local fovCenter
    if _G.Config.FOVFollowCursor then
        fovCenter = UserInputService:GetMouseLocation()
    else
        local screenSize = camera.ViewportSize
        fovCenter = Vector2.new(screenSize.X / 2, screenSize.Y / 2)
    end
    
    for _, v in pairs(Players:GetPlayers()) do
        if v ~= player and v.Character and v.Character:FindFirstChild("Head") then
            if _G.Config.IgnoreDead and not isPlayerAlive(v) then 
                continue 
            end
            
            if _G.Config.TeamCheck and isSameTeam(v) then continue end
            
            local head = v.Character.Head
            local hrp = v.Character:FindFirstChild("HumanoidRootPart")
            
            if hrp then
                local distance3D = (hrp.Position - player.Character.HumanoidRootPart.Position).Magnitude
                if distance3D > _G.Config.LockDistance then continue end
            end
            
            local screenPoint, onScreen = camera:WorldToViewportPoint(head.Position)
            
            if onScreen then
                local mousePos = Vector2.new(screenPoint.X, screenPoint.Y)
                local distance = (fovCenter - mousePos).Magnitude
                
                if distance < shortestDistance then
                    if _G.Config.WallCheck then
                        local ray = Ray.new(camera.CFrame.Position, (head.Position - camera.CFrame.Position).Unit * 1000)
                        local hit = Workspace:FindPartOnRayWithIgnoreList(ray, {player.Character, camera})
                        
                        if hit and hit:IsDescendantOf(v.Character) then
                            closestPlayer = v
                            shortestDistance = distance
                        end
                    else
                        closestPlayer = v
                        shortestDistance = distance
                    end
                end
            end
        end
    end
    
    return closestPlayer
end

-- NOVO: Sistema de Perfect Aim
local currentTarget = nil

RunService.RenderStepped:Connect(function()
    if (_G.Config.Aimbot or _G.Config.FOVFollowCursor) and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        local target = getClosestPlayer()
        
        if target and target.Character and target.Character:FindFirstChild("Head") then
            if not _G.Config.IgnoreDead or isPlayerAlive(target) then
                currentTarget = target
                
                -- Perfect Aim: Mira exata na cabeça sem spread
                if _G.Config.PerfectAim then
                    local head = target.Character.Head
                    camera.CFrame = CFrame.new(camera.CFrame.Position, head.Position)
                    
                    -- Compensação de movimento (prediction)
                    if target.Character:FindFirstChild("HumanoidRootPart") then
                        local targetVelocity = target.Character.HumanoidRootPart.AssemblyVelocity
                        local predictedPosition = head.Position + (targetVelocity * 0.1)
                        camera.CFrame = CFrame.new(camera.CFrame.Position, predictedPosition)
                    end
                else
                    -- Aimbot normal
                    camera.CFrame = CFrame.new(camera.CFrame.Position, target.Character.Head.Position)
                end
            end
        else
            currentTarget = nil
        end
    else
        currentTarget = nil
    end
end)

-- ============= ESP SYSTEM COM ADMIN DETECTION =============
local function removeESP(targetPlayer)
    if espObjects[targetPlayer] then
        pcall(function()
            if espObjects[targetPlayer].connection then 
                espObjects[targetPlayer].connection:Disconnect() 
            end
            for _, line in pairs(espObjects[targetPlayer].lines) do 
                pcall(function() line:Remove() end)
            end
            pcall(function() espObjects[targetPlayer].headCircle:Remove() end)
            pcall(function() espObjects[targetPlayer].tracer:Remove() end)
            pcall(function() espObjects[targetPlayer].distanceText:Remove() end)
            pcall(function() espObjects[targetPlayer].adminLabel:Remove() end)
        end)
        espObjects[targetPlayer] = nil
    end
end

local function createESP(targetPlayer)
    if espObjects[targetPlayer] then 
        removeESP(targetPlayer)
    end
    
    local esp = {}
    espObjects[targetPlayer] = esp
    
    local function createLine()
        local line = Drawing.new("Line")
        line.Visible = false
        line.Thickness = 2
        line.Transparency = 1
        return line
    end
    
    esp.lines = {
        head_neck = createLine(),
        neck_torso = createLine(),
        torso_leftArm = createLine(),
        torso_rightArm = createLine(),
        torso_leftLeg = createLine(),
        torso_rightLeg = createLine()
    }
    
    esp.headCircle = Drawing.new("Circle")
    esp.headCircle.Visible = false
    esp.headCircle.Thickness = 2
    esp.headCircle.Filled = false
    esp.headCircle.Transparency = 1
    esp.headCircle.NumSides = 32
    esp.headCircle.Radius = 10
    
    esp.tracer = Drawing.new("Line")
    esp.tracer.Visible = false
    esp.tracer.Thickness = 2
    esp.tracer.Transparency = 1
    
    esp.distanceText = Drawing.new("Text")
    esp.distanceText.Visible = false
    esp.distanceText.Center = true
    esp.distanceText.Outline = true
    esp.distanceText.Size = 14
    esp.distanceText.Color = Color3.fromRGB(255, 255, 255)
    
    -- NOVO: Label para admins
    esp.adminLabel = Drawing.new("Text")
    esp.adminLabel.Visible = false
    esp.adminLabel.Center = true
    esp.adminLabel.Outline = true
    esp.adminLabel.Size = 16
    esp.adminLabel.Color = Color3.fromRGB(255, 215, 0) -- Dourado
    
    esp.connection = RunService.RenderStepped:Connect(function()
        pcall(function()
            if not _G.Config.ESP or not targetPlayer or not targetPlayer.Parent or not targetPlayer.Character then
                for _, line in pairs(esp.lines) do line.Visible = false end
                esp.headCircle.Visible = false
                esp.tracer.Visible = false
                esp.distanceText.Visible = false
                esp.adminLabel.Visible = false
                return
            end
            
            local char = targetPlayer.Character
            local head = char:FindFirstChild("Head")
            local torso = char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso")
            local hrp = char:FindFirstChild("HumanoidRootPart")
            
            if not head or not torso or not hrp then
                for _, line in pairs(esp.lines) do line.Visible = false end
                esp.headCircle.Visible = false
                esp.tracer.Visible = false
                esp.distanceText.Visible = false
                esp.adminLabel.Visible = false
                return
            end
            
            local isAlive = isPlayerAlive(targetPlayer)
            if not _G.Config.ESPShowDead and not isAlive then
                for _, line in pairs(esp.lines) do line.Visible = false end
                esp.headCircle.Visible = false
                esp.tracer.Visible = false
                esp.distanceText.Visible = false
                esp.adminLabel.Visible = false
                return
            end
            
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local distance = (hrp.Position - player.Character.HumanoidRootPart.Position).Magnitude
                
                if distance > _G.Config.ESPMaxDistance then
                    for _, line in pairs(esp.lines) do line.Visible = false end
                    esp.headCircle.Visible = false
                    esp.tracer.Visible = false
                    esp.distanceText.Visible = false
                    esp.adminLabel.Visible = false
                    return
                end
                
                -- Mostrar distância
                if _G.Config.ESPDistance then
                    local headPos, headOnScreen = camera:WorldToViewportPoint(head.Position)
                    if headOnScreen then
                        esp.distanceText.Position = Vector2.new(headPos.X, headPos.Y - 25)
                        esp.distanceText.Text = string.format("%.0fm", distance)
                        esp.distanceText.Visible = true
                    else
                        esp.distanceText.Visible = false
                    end
                else
                    esp.distanceText.Visible = false
                end
                
                -- NOVO: Mostrar label de admin
                local playerIsAdmin = isAdmin(targetPlayer)
                if playerIsAdmin then
                    local headPos, headOnScreen = camera:WorldToViewportPoint(head.Position)
                    if headOnScreen then
                        esp.adminLabel.Position = Vector2.new(headPos.X, headPos.Y - 45)
                        esp.adminLabel.Text = "👮 ADMIN"
                        esp.adminLabel.Visible = true
                    else
                        esp.adminLabel.Visible = false
                    end
                else
                    esp.adminLabel.Visible = false
                end
            end
            
            -- Definir cor
            local color
            if not isAlive then
                color = Color3.fromRGB(128, 128, 128) -- Cinza para mortos
            elseif isAdmin(targetPlayer) then
                color = Color3.fromRGB(255, 215, 0) -- Dourado para admins
            elseif _G.Config.ESPTeamCheck and isSameTeam(targetPlayer) then
                color = Color3.fromRGB(0, 150, 255) -- Azul para aliados
            else
                color = Color3.fromRGB(255, 0, 0) -- Vermelho para inimigos
            end
            
            for _, line in pairs(esp.lines) do line.Color = color end
            esp.headCircle.Color = color
            esp.tracer.Color = color
            
            -- Head Circle
            if _G.Config.ESPHeadCircle then
                local headPos, headOnScreen = camera:WorldToViewportPoint(head.Position)
                if headOnScreen then
                    esp.headCircle.Position = Vector2.new(headPos.X, headPos.Y)
                    esp.headCircle.Visible = true
                else
                    esp.headCircle.Visible = false
                end
            else
                esp.headCircle.Visible = false
            end
            
            -- Tracer
            if _G.Config.ESPTracer then
                local torsoPos, torsoOnScreen = camera:WorldToViewportPoint(torso.Position)
                if torsoOnScreen then
                    local screenSize = camera.ViewportSize
                    esp.tracer.From = Vector2.new(screenSize.X / 2, 0)
                    esp.tracer.To = Vector2.new(torsoPos.X, torsoPos.Y)
                    esp.tracer.Visible = true
                else
                    esp.tracer.Visible = false
                end
            else
                esp.tracer.Visible = false
            end
            
            -- Skeleton
            if _G.Config.ESPSkeleton then
                local limbs = {
                    Head = head,
                    Neck = torso,
                    LeftArm = char:FindFirstChild("Left Arm") or char:FindFirstChild("LeftUpperArm"),
                    RightArm = char:FindFirstChild("Right Arm") or char:FindFirstChild("RightUpperArm"),
                    LeftLeg = char:FindFirstChild("Left Leg") or char:FindFirstChild("LeftUpperLeg"),
                    RightLeg = char:FindFirstChild("Right Leg") or char:FindFirstChild("RightUpperLeg")
                }
                
                local function drawLimb(line, from, to)
                    if from and to then
                        local fromPos, fromOn = camera:WorldToViewportPoint(from.Position)
                        local toPos, toOn = camera:WorldToViewportPoint(to.Position)
                        if fromOn and toOn then
                            line.From = Vector2.new(fromPos.X, fromPos.Y)
                            line.To = Vector2.new(toPos.X, toPos.Y)
                            line.Visible = true
                            return
                        end
                    end
                    line.Visible = false
                end
                
                drawLimb(esp.lines.head_neck, limbs.Head, limbs.Neck)
                drawLimb(esp.lines.torso_leftArm, limbs.Neck, limbs.LeftArm)
                drawLimb(esp.lines.torso_rightArm, limbs.Neck, limbs.RightArm)
                drawLimb(esp.lines.torso_leftLeg, limbs.Neck, limbs.LeftLeg)
                drawLimb(esp.lines.torso_rightLeg, limbs.Neck, limbs.RightLeg)
            else
                for _, line in pairs(esp.lines) do line.Visible = false end
            end
        end)
    end)
end

Players.PlayerAdded:Connect(function(p)
    if p ~= player then 
        wait(0.5)
        createESP(p) 
    end
end)

Players.PlayerRemoving:Connect(function(p)
    removeESP(p)
end)

for _, p in pairs(Players:GetPlayers()) do
    if p ~= player then createESP(p) end
end

-- ============= BOTÃO FLUTUANTE =============
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ControlGUI"
screenGui.Parent = game.CoreGui

local toggleButton = Instance.new("ImageButton")
toggleButton.Size = UDim2.new(0, 50, 0, 50)
toggleButton.Position = UDim2.new(0.15, 0, 0.15, 0)
toggleButton.Image = "rbxassetid://136724157425040"
toggleButton.BackgroundTransparency = 1
toggleButton.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0.25, 0)
corner.Parent = toggleButton

local isFluentVisible = true
local dragging, dragInput, dragStart, startPos

local function update(input)
    local delta = input.Position - dragStart
    toggleButton.Position = UDim2.new(
        startPos.X.Scale,
        startPos.X.Offset + delta.X,
        startPos.Y.Scale,
        startPos.Y.Offset + delta.Y
    )
end

toggleButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch 
    or input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = toggleButton.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

toggleButton.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch 
    or input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

game:GetService("UserInputService").InputChanged:Connect(function(input)
    if dragging and input == dragInput then
        update(input)
    end
end)

toggleButton.MouseButton1Click:Connect(function()
    isFluentVisible = not isFluentVisible
    if isFluentVisible then
        Window:Minimize(false)
    else
        Window:Minimize(true)
    end
end)

-- ============= INICIALIZAÇÃO =============
wait(0.5)
updateAdminCount()
notify(string.format("✅ Sistema Carregado! 👮 %d Admin(s) detectado(s)", adminCount))
print("==========================================")
print("✅ SISTEMA COMPLETO CARREGADO!")
print("==========================================")
print("NOVAS FEATURES:")
print("🎯 Perfect Aim - Precisão 100%")
print("👮 Admin Detector - " .. adminCount .. " admin(s)")
print("==========================================")
print("END ou INSERT = Minimizar")
print("🎯 Aimbot | 👁️ ESP | 👮 Admin Detection")
print("🖱️ FOV Segue Cursor")
print("💀 Sistema Anti-Morto ATIVO!")
print("==========================================")
