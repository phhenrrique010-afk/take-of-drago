-- Script educativo para Roblox Studio (LocalScript em StarterPlayerScripts)
-- Use apenas em seu próprio lugar para aprender. Não use para trapacear em jogos alheios.
-- Funcionalidades demonstradas: UI com toggles, auto-mover, "auto-kill" em NPCs de teste,
-- safe-kill (atrás do NPC), simulação de espada com durabilidade/quebra/recarga, e "auto-start" de uma rodada local.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local StarterGui = game:GetService("StarterGui")

local player = Players.LocalPlayer
local char = nil

local config = {
    AutoExecute = false,   -- exemplo de toggle
    AutoKill = false,
    SafeKill = true,
    AutoGoto = false,
    GrowSwordOnBreak = true,
    AutoRefill = false,
    AutoStartMatch = false,
    KillRange = 40,        -- distância máxima para "matar" (apenas em ambiente local)
}

-- UTIL: atualiza referência ao personagem
local function updateCharacter()
    char = player.Character or player.CharacterAdded:Wait()
end
updateCharacter()
player.CharacterAdded:Connect(function() updateCharacter() end)

-- UTIL: cria uma espada simples na mão para demonstração (Server-authoritative em jogo real, aqui é apenas local)
local function ensureLocalSword()
    if not char then return end
    local tool = char:FindFirstChild("DemoSword")
    if tool and tool:IsA("Tool") then
        return tool
    end
    -- criar ferramenta simples
    local sword = Instance.new("Tool")
    sword.Name = "DemoSword"
    sword.RequiresHandle = true
    sword.Parent = char

    local handle = Instance.new("Part")
    handle.Name = "Handle"
    handle.Size = Vector3.new(1, 4, 0.3)
    handle.Color = Color3.fromRGB(120, 120, 120)
    handle.CanCollide = false
    handle.Parent = sword

    local mesh = Instance.new("SpecialMesh", handle)
    mesh.MeshType = Enum.MeshType.FileMesh
    mesh.Scale = Vector3.new(1, 1, 1)

    -- durabilidade como atributo local
    sword:SetAttribute("Durability", 100)
    sword:SetAttribute("MaxDurability", 100)
    sword:SetAttribute("Broken", false)
    return sword
end

-- UTIL: "usar" a espada: reduz durabilidade; if breaks, simulação de quebra
local function useSword(tool, amount)
    if not tool or not tool:IsA("Tool") then return end
    local d = tool:GetAttribute("Durability") or 0
    local broken = tool:GetAttribute("Broken")
    if broken then return end
    d = d - (amount or 10)
    if d <= 0 then
        tool:SetAttribute("Durability", 0)
        tool:SetAttribute("Broken", true)
        -- mostrar efeito visual de quebra (local)
        if tool.Handle then
            tool.Handle.Transparency = 0.6
            tool.Handle.Color = Color3.fromRGB(80,80,80)
            tool.Handle.Size = tool.Handle.Size
