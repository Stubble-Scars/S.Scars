repeat
    task.wait()
until game:IsLoaded()

local start = tick()
local client = game:GetService('Players').LocalPlayer;

local runService = game:GetService('RunService')
local repStorage = game:GetService('ReplicatedStorage')
local virtualInputManager = game:GetService('VirtualInputManager')

local knitServices, mobs, knitShared
local counter = 0

while true do
    if typeof(knitServices) ~= 'Instance' then
        for _, obj in next, repStorage:GetChildren() do
            if obj.Name == 'Packages' and obj:FindFirstChild('Knit') and obj.Knit:IsA('ModuleScript') then 
                if obj.Knit:FindFirstChild('Services') and obj.Knit.Services:FindFirstChild('MoveService') and obj.Knit.Services:FindFirstChild('QuestService') and obj.Knit.Services:FindFirstChild('InventoryService') and obj.Knit.Services:FindFirstChild('DataService') then
                    if obj.Knit.Services.MoveService:FindFirstChild('RF') and obj.Knit.Services.QuestService:FindFirstChild('RE') then
                        knitServices = obj.Knit.Services
                    end
                end
            end
        end
    end

    if typeof(mobs) ~= 'Instance' then
        for _, obj in next, workspace:GetChildren() do
            if obj.Name == 'NPC' and obj:IsA('Folder') and obj:FindFirstChild('Enemy') then 
                mobs = obj.Enemy
            end
        end
    end

    if typeof(knitShared) ~= 'Instance' then
        for _, obj in next, repStorage:GetChildren() do
            if obj.Name == 'Shared' and obj:IsA('Folder') and obj:FindFirstChild('QuestData') then 
                knitShared = obj
            end
        end
    end

    if (typeof(knitServices) == 'Instance' and typeof(mobs) == 'Instance' and typeof(knitShared) == 'Instance') then
        break
    end

    counter = counter + 1
    if counter > 6 then
        client:Kick(string.format('Failed to load game dependencies. Details: %s, %s, %s', typeof(knitServices), typeof(mobs), typeof(knitShared)))
    end
    task.wait(1)
end

local getupvalues = debug.getupvalues or getupvalues;

do
    if shared._unload then
        pcall(shared._unload)
    end

    function shared._unload()
        if shared._id then
            pcall(runService.UnbindFromRenderStep, runService, shared._id)
        end
    end

    shared._id = httpService:GenerateGUID(false)
end

do
    local thread = task.spawn(function()
        while true do
            task.wait()
            if ((Toggles.KillAura) and (Toggles.KillAura.Value)) then
                if client.Character:IsDescendantOf(workspace) and knitServices.MoveService.RF:FindFirstChild('MoveStart') and client.Character.PrimaryPart ~= nil then
                    local closestMobs = {client.Character.PrimaryPart}
                    for _, v in next, mobs:GetChildren() do
                        if v:FindFirstChildOfClass('Humanoid') and v:FindFirstChildOfClass('Humanoid').Health > 0 and (client.Character:GetPivot().Position - v:GetPivot().Position).Magnitude < 50 then
                            table.insert(closestMobs, {Parent = v})
                        end
                    end
                    if #closestMobs > 1 then
                        knitServices.MoveService.RF.MoveStart:InvokeServer('M1', closestMobs)
                    end
                end
            end
        end
    end)
    table.insert(shared.callbacks, function()
        pcall(task.cancel, thread)
    end)
end

UI:Notify(string.format('Loaded script in %.4f second(s)!', tick() - start), 3)
