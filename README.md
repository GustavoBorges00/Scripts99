-- ESP Configuração
local ESP_ENABLED = true
local TEAM_CHECK = true
local ESP_RANGE = 300.0 -- Distância máxima para mostrar

-- Cores por time (TeamID como exemplo)
local teamColors = {
    [1] = {r = 0, g = 255, b = 0},   -- Verde para time 1
    [2] = {r = 255, g = 0, b = 0},   -- Vermelho para time 2
    [0] = {r = 255, g = 255, b = 255} -- Branco para jogadores sem time
}

-- Simulação para pegar o Team ID (substitua com a função do seu jogo)
function GetPlayerTeam(playerId)
    -- Aqui você deveria usar a API do seu jogo para obter o time do jogador
    return GetPedRelationshipGroupHash(GetPlayerPed(playerId)) % 3 -- Exemplo genérico
end

-- Função para desenhar texto na tela
function DrawText3D(x, y, z, text, color)
    SetTextScale(0.35, 0.35)
    SetTextFont(4)
    SetTextProportional(1)
    SetTextColour(color.r, color.g, color.b, 215)
    SetTextEntry("STRING")
    SetTextCentre(true)
    AddTextComponentString(text)
    SetDrawOrigin(x, y, z + 1.0, 0)
    DrawText(0.0, 0.0)
    ClearDrawOrigin()
end

-- Loop principal
Citizen.CreateThread(function()
    while true do
        Wait(1)
        if ESP_ENABLED then
            local myPed = PlayerPedId()
            local myCoords = GetEntityCoords(myPed)
            local myTeam = GetPlayerTeam(PlayerId())

            for i = 0, 255 do
                if NetworkIsPlayerActive(i) and i ~= PlayerId() then
                    local ped = GetPlayerPed(i)
                    local pedCoords = GetEntityCoords(ped)
                    local dist = #(myCoords - pedCoords)

                    if dist < ESP_RANGE then
                        local team = GetPlayerTeam(i)
                        if not TEAM_CHECK or team ~= myTeam then
                            local color = teamColors[team] or teamColors[0]
                            local name = GetPlayerName(i)
                            DrawText3D(pedCoords.x, pedCoords.y, pedCoords.z, name, color)
                        end
                    end
                end
            end
        end
    end
end)
