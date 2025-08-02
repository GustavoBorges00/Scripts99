local ESP_ENABLED = true
local TEAM_CHECK = true
local ESP_RANGE = 300.0

local teamColors = {
    [1] = {r = 0, g = 255, b = 0},     -- Time 1: Verde
    [2] = {r = 255, g = 0, b = 0},     -- Time 2: Vermelho
    [0] = {r = 255, g = 255, b = 255}  -- Outro: Branco
}

-- Função de exemplo para pegar o time do jogador (modifique conforme seu sistema)
function GetPlayerTeam(playerId)
    return GetPedRelationshipGroupHash(GetPlayerPed(playerId)) % 3
end

-- Corrigida: desenha texto 2D baseado na posição 3D do jogador
function DrawText3D(x, y, z, text, color)
    local onScreen, _x, _y = World3dToScreen2d(x, y, z + 1.0)
    local px, py, pz = table.unpack(GetGameplayCamCoords())
    
    if onScreen then
        SetTextScale(0.35, 0.35)
        SetTextFont(4)
        SetTextProportional(1)
        SetTextColour(color.r, color.g, color.b, 255)
        SetTextEntry("STRING")
        SetTextCentre(true)
        AddTextComponentString(text)
        DrawText(_x, _y)
    end
end

-- ESP loop
Citizen.CreateThread(function()
    while true do
        Wait(0)
        if ESP_ENABLED then
            local myPed = PlayerPedId()
            local myCoords = GetEntityCoords(myPed)
            local myTeam = GetPlayerTeam(PlayerId())

            for i = 0, 255 do
                if NetworkIsPlayerActive(i) and i ~= PlayerId() then
                    local targetPed = GetPlayerPed(i)
                    if DoesEntityExist(targetPed) then
                        local targetCoords = GetEntityCoords(targetPed)
                        local dist = #(myCoords - targetCoords)
                        
                        if dist < ESP_RANGE then
                            local targetTeam = GetPlayerTeam(i)
                            if not TEAM_CHECK or targetTeam ~= myTeam then
                                local color = teamColors[targetTeam] or teamColors[0]
                                local name = GetPlayerName(i)
                                DrawText3D(targetCoords.x, targetCoords.y, targetCoords.z, name, color)
                            end
                        end
                    end
                end
            end
        end
    end
end)
