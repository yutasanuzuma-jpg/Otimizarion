local success, Rayfield = pcall(function()
    return loadstring(game:HttpGet('https://raw.githubusercontent.com/shlexware/Rayfield/main/source'))()
end)

if not success or not Rayfield then
    Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
end

local Window = Rayfield:CreateWindow({
   Name = "Hub de Otimização (Ultra Leve)",
   LoadingTitle = "Iniciando...",
   LoadingSubtitle = "Modo Anti-Lag",
   ConfigurationSaving = { Enabled = false },
   KeySystem = false
})

local Tab = Window:CreateTab("Otimização", 4483362458)

-- Serviços
local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

--------------------------------------------------------------------------------
-- 1. REMOVER TEXTURAS DO MAPA
--------------------------------------------------------------------------------
Tab:CreateSection("Otimização de Mapa")

local NoTexturesEnabled = false
local MapConn = nil

local function LimparTextura(v)
   pcall(function()
      if v:IsA("Decal") or v:IsA("Texture") then
         v:Destroy()
      elseif v:IsA("BasePart") then
         v.Material = Enum.Material.SmoothPlastic
         v.Reflectance = 0
      end
   end)
end

Tab:CreateToggle({
   Name = "Remover Texturas do Mapa",
   CurrentValue = false,
   Flag = "NoTexturesToggle",
   Callback = function(Value)
      NoTexturesEnabled = Value
      if Value then
         for _, obj in ipairs(workspace:GetDescendants()) do
            LimparTextura(obj)
         end
         MapConn = workspace.DescendantAdded:Connect(function(novoObj)
            if NoTexturesEnabled then LimparTextura(novoObj) end
         end)
      elseif MapConn then
         MapConn:Disconnect()
         MapConn = nil
      end
   end,
})

--------------------------------------------------------------------------------
-- 2. REMOVER EFEITOS GLOBAIS
--------------------------------------------------------------------------------
Tab:CreateSection("Efeitos Globais")

local NoEffectsEnabled = false
local EffMapConn = nil

local function LimparEfeito(v)
   pcall(function()
      if v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Smoke") or v:IsA("Fire") 
      or v:IsA("Sparkles") or v:IsA("Highlight") or v:IsA("Beam") or v:IsA("Explosion") then
         v:Destroy()
      elseif v:IsA("PostEffect") then
         v.Enabled = false
      end
   end)
end

Tab:CreateToggle({
   Name = "Remover Todos os Efeitos do Jogo",
   CurrentValue = false,
   Flag = "NoEffectsToggle",
   Callback = function(Value)
      NoEffectsEnabled = Value
      if Value then
         for _, obj in ipairs(workspace:GetDescendants()) do LimparEfeito(obj) end
         for _, obj in ipairs(Lighting:GetDescendants()) do LimparEfeito(obj) end

         EffMapConn = workspace.DescendantAdded:Connect(function(novoObj)
            if NoEffectsEnabled then LimparEfeito(novoObj) end
         end)
      elseif EffMapConn then
         EffMapConn:Disconnect()
         EffMapConn = nil
      end
   end,
})

--------------------------------------------------------------------------------
-- 3. FULLBRIGHT E NO FOG
--------------------------------------------------------------------------------
Tab:CreateSection("Iluminação e Visibilidade")

local FullbrightEnabled = false
local FullbrightConn = nil

Tab:CreateToggle({
   Name = "Ativar Fullbright (Sem Escuridão)",
   CurrentValue = false,
   Flag = "FullbrightToggle",
   Callback = function(Value)
      FullbrightEnabled = Value
      if Value then
         Lighting.Ambient = Color3.fromRGB(255, 255, 255)
         Lighting.OutdoorAmbient = Color3.fromRGB(255, 255, 255)
         Lighting.Brightness = 2
         Lighting.GlobalShadows = false

         FullbrightConn = Lighting.Changed:Connect(function()
            if FullbrightEnabled then
               Lighting.Ambient = Color3.fromRGB(255, 255, 255)
               Lighting.OutdoorAmbient = Color3.fromRGB(255, 255, 255)
               Lighting.Brightness = 2
               Lighting.GlobalShadows = false
            end
         end)
      elseif FullbrightConn then
         FullbrightConn:Disconnect()
         FullbrightConn = nil
      end
   end,
})

local NoFogEnabled = false
local NoFogConn = nil

local OldFogEnd = Lighting.FogEnd
local OldFogStart = Lighting.FogStart

local function AplicarNoFog()
   pcall(function()
      Lighting.FogStart = 9e9
      Lighting.FogEnd = 9e9
      for _, v in ipairs(Lighting:GetChildren()) do
         if v:IsA("Atmosphere") then
            v.Density = 0
         end
      end
   end)
end

Tab:CreateToggle({
   Name = "Remover Névoa (No Fog)",
   CurrentValue = false,
   Flag = "NoFogToggle",
   Callback = function(Value)
      NoFogEnabled = Value
      if Value then
         OldFogEnd = Lighting.FogEnd
         OldFogStart = Lighting.FogStart

         AplicarNoFog()

         NoFogConn = Lighting.Changed:Connect(function()
            if NoFogEnabled then
               AplicarNoFog()
            end
         end)

         Rayfield:Notify({
            Title = "Névoa Removida",
            Content = "A névoa e a neblina do mapa foram desativadas.",
            Duration = 3,
            Image = 4483362458,
         })
      else
         if NoFogConn then
            NoFogConn:Disconnect()
            NoFogConn = nil
         end

         pcall(function()
            Lighting.FogStart = OldFogStart
            Lighting.FogEnd = OldFogEnd
         end)
      end
   end,
})

--------------------------------------------------------------------------------
-- 4. CONTROLE DE CÂMERA (BALANÇO E FOV PERSONALIZADO)
--------------------------------------------------------------------------------
Tab:CreateSection("Controle de Câmera")

local NoCameraShakeEnabled = false
local CameraShakeConn = nil

local function EstabilizarCamera()
   pcall(function()
      local char = LocalPlayer.Character
      if char then
         local hum = char:FindFirstChildOfClass("Humanoid")
         if hum and hum.CameraOffset ~= Vector3.zero then
            hum.CameraOffset = Vector3.zero
         end
      end
   end)
end

Tab:CreateToggle({
   Name = "Remover Balanço de Tela (No Shake)",
   CurrentValue = false,
   Flag = "NoCameraShakeToggle",
   Callback = function(Value)
      NoCameraShakeEnabled = Value
      if Value then
         EstabilizarCamera()
         
         CameraShakeConn = RunService.RenderStepped:Connect(function()
            if NoCameraShakeEnabled then
               EstabilizarCamera()
            end
         end)

         Rayfield:Notify({
            Title = "Câmera Estabilizada",
            Content = "Os tremores e balanços de tela foram removidos.",
            Duration = 3,
            Image = 4483362458,
         })
      else
         if CameraShakeConn then
            CameraShakeConn:Disconnect()
            CameraShakeConn = nil
         end
      end
   end,
})

local CustomFOV = nil
local FOVConn = nil

Tab:CreateInput({
   Name = "Definir FOV (Digite o Número)",
   PlaceholderText = "Ex: 70, 90, 100, 120...",
   RemoveTextAfterFocusLost = false,
   Callback = function(Text)
      local num = tonumber(Text)
      if num and num > 0 and num <= 160 then
         CustomFOV = num
         workspace.CurrentCamera.FieldOfView = num

         if not FOVConn then
            FOVConn = RunService.RenderStepped:Connect(function()
               if CustomFOV and workspace.CurrentCamera and workspace.CurrentCamera.FieldOfView ~= CustomFOV then
                  workspace.CurrentCamera.FieldOfView = CustomFOV
               end
            end)
         end

         Rayfield:Notify({
            Title = "FOV Alterado",
            Content = "Campo de visão definido para: " .. tostring(num),
            Duration = 3,
            Image = 4483362458,
         })
      else
         Rayfield:Notify({
            Title = "Valor Inválido",
            Content = "Por favor, digite um número válido (ex: 70 até 120).",
            Duration = 3,
            Image = 4483362458,
         })
      end
   end,
})

--------------------------------------------------------------------------------
-- 5. PARAR ANIMAÇÕES (VOCÊ, PLAYERS E NPCS)
--------------------------------------------------------------------------------
Tab:CreateSection("Animações de Personagens e NPCs")

local FreezeAnimEnabled = false
local AnimConnections = {}

local function ProcessarHumanoid(humanoid)
   if not humanoid or not humanoid:IsA("Humanoid") then return end

   pcall(function()
      for _, track in ipairs(humanoid:GetPlayingAnimationTracks()) do
         track:Stop()
      end
   end)

   local conn = humanoid.AnimationPlayed:Connect(function(track)
      if FreezeAnimEnabled then
         track:Stop()
      end
   end)
   table.insert(AnimConnections, conn)
end

Tab:CreateToggle({
   Name = "Remover Animações (Você, Players e NPCs)",
   CurrentValue = false,
   Flag = "FreezeAnimToggle",
   Callback = function(Value)
      FreezeAnimEnabled = Value

      if Value then
         for _, obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("Humanoid") then
               ProcessarHumanoid(obj)
            end
         end

         local connAdd = workspace.DescendantAdded:Connect(function(novoObj)
            if FreezeAnimEnabled and novoObj:IsA("Humanoid") then
               task.wait(0.05)
               ProcessarHumanoid(novoObj)
            end
         end)
         table.insert(AnimConnections, connAdd)

         Rayfield:Notify({
            Title = "Animações Desativadas",
            Content = "Animações do seu personagem, players e NPCs foram paradas.",
            Duration = 3,
            Image = 4483362458,
         })
      else
         for _, conn in ipairs(AnimConnections) do
            if conn then conn:Disconnect() end
         end
         AnimConnections = {}
      end
   end,
})

--------------------------------------------------------------------------------
-- 6. OCULTAR OUTROS PLAYERS
--------------------------------------------------------------------------------
Tab:CreateSection("Ocultar Outros Jogadores")

local HidePlayersEnabled = false
local PlayerConn = nil

local function OcultarPersonagem(character)
   if not character then return end
   pcall(function()
      for _, v in ipairs(character:GetDescendants()) do
         if v:IsA("BasePart") then
            v.Transparency = 1
         elseif v:IsA("Accessory") or v:IsA("Shirt") or v:IsA("Pants") or v:IsA("ShirtGraphic") then
            v:Destroy()
         elseif v:IsA("BillboardGui") or v:IsA("SurfaceGui") then
            v.Enabled = false
         end
      end
   end)
end

Tab:CreateToggle({
   Name = "Invisibilizar Outros Players",
   CurrentValue = false,
   Flag = "HidePlayersToggle",
   Callback = function(Value)
      HidePlayersEnabled = Value
      if Value then
         for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character then
               OcultarPersonagem(p.Character)
            end
         end
         PlayerConn = Players.PlayerAdded:Connect(function(p)
            p.CharacterAdded:Connect(function(char)
               if HidePlayersEnabled then
                  task.wait(0.2)
                  OcultarPersonagem(char)
               end
            end)
         end)
      elseif PlayerConn then
         PlayerConn:Disconnect()
         PlayerConn = nil
      end
   end,
})
