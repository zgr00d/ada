-- إعدادات الخدمات المطلوبة
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local hrp = character:WaitForChild("HumanoidRootPart")
local folder = Instance.new("Folder", workspace)
folder.Name = "ZGR00DBlocks"
local blocks = {}
local colorList = {}
for i = 1, 100 do table.insert(colorList, Color3.fromHSV(i/100, 1, 1)) end
local rotating = true

-- إنشاء البلوكات
local function createBlocks()
    for i = 1, 130 do
        local part = Instance.new("Part")
        part.Name = "ZBlock_"..i
        part.Size = Vector3.new(1,1,1)
        part.Anchored = true
        part.CanCollide = false
        part.Color = colorList[i % #colorList + 1]
        part.Material = Enum.Material.Neon
        part.Parent = folder
        table.insert(blocks, part)
    end
end

-- دوران البلوكات حول اللاعب
local function rotateBlocks()
    RunService.Heartbeat:Connect(function()
        if rotating then
            local t = tick()
            for i, part in ipairs(blocks) do
                local angle = (i/#blocks) * math.pi * 2 + t
                local radius = 10
                local x = math.cos(angle) * radius
                local z = math.sin(angle) * radius
                local y = 2 + math.sin(t + i)
                part.Position = hrp.Position + Vector3.new(x, y, z)
            end
        end
    end)
end

-- تحديث ألوان البلوكات بشكل مستمر
local function updateColors()
    while true do
        for i, part in ipairs(blocks) do
            part.Color = colorList[(tick()*10 + i)%#colorList + 1]
        end
        task.wait(0.15)
    end
end

-- طرد اللاعبين الآخرين عند الاقتراب
local function repelOtherPlayers()
    RunService.Heartbeat:Connect(function()
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                local target = p.Character.HumanoidRootPart
                for _, b in ipairs(blocks) do
                    if (b.Position - target.Position).Magnitude < 5 then
                        target.Velocity = (target.Position - hrp.Position).Unit * 120
                    end
                end
            end
        end
    end)
end

-- عرض الاسم المتغير فوق اللاعب
local function createFloatingName()
    local gui = Instance.new("BillboardGui", hrp)
    gui.Size = UDim2.new(0, 200, 0, 50)
    gui.StudsOffset = Vector3.new(0, 3, 0)
    gui.AlwaysOnTop = true
    local label = Instance.new("TextLabel", gui)
    label.Size = UDim2.new(1,0,1,0)
    label.BackgroundTransparency = 1
    label.TextScaled = true
    label.Font = Enum.Font.GothamBlack
    label.Text = "ZGR00D"
    coroutine.wrap(function()
        while true do
            label.TextColor3 = colorList[math.random(1, #colorList)]
            task.wait(0.1)
        end
    end)()
end

-- تشكيل الأشكال المختلفة (صاروخ، حرف R، هرم، كرة، مكعب)
local function formRocket()
    rotating = false
    for i, part in ipairs(blocks) do
        TweenService:Create(part, TweenInfo.new(0.5), {
            Position = hrp.Position + Vector3.new(0, i*0.3, 0)
        }):Play()
    end
end

local function formLetterR()
    rotating = false
    for i, part in ipairs(blocks) do
        local offset = Vector3.new((i%10)-5, math.floor(i/10), 0)
        TweenService:Create(part, TweenInfo.new(0.5), {
            Position = hrp.Position + offset
        }):Play()
    end
end

local function formPyramid()
    rotating = false
    for i, part in ipairs(blocks) do
        local offset = Vector3.new(i % 10, math.floor(i / 10), math.abs(math.sin(i) * 5))
        TweenService:Create(part, TweenInfo.new(0.5), {
            Position = hrp.Position + offset
        }):Play()
    end
end

local function formSphere()
    rotating = false
    for i, part in ipairs(blocks) do
        local angle = math.pi * 2 * (i / #blocks)
        local radius = 5
        local x = radius * math.cos(angle)
        local z = radius * math.sin(angle)
        local y = math.sin(angle) * 2
        TweenService:Create(part, TweenInfo.new(0.5), {
            Position = hrp.Position + Vector3.new(x, y, z)
        }):Play()
    end
end

local function formCube()
    rotating = false
    for i, part in ipairs(blocks) do
        local offset = Vector3.new(i % 5, math.floor(i / 5), math.floor(i / 25))
        TweenService:Create(part, TweenInfo.new(0.5), {
            Position = hrp.Position + offset
        }):Play()
    end
end

-- التقاط اللاعب باستخدام البلوكات
local targetGrab = nil
local function setupGrab()
    UserInputService.InputBegan:Connect(function(input)
        if input.KeyCode == Enum.KeyCode.X then
            local mouse = player:GetMouse()
            mouse.Button1Down:Connect(function()
                local target = mouse.Target
                if target and target:IsA("BasePart") and target.Parent and Players:GetPlayerFromCharacter(target.Parent) then
                    targetGrab = target
                end
            end)
        end
    end)

    RunService.Heartbeat:Connect(function()
        if targetGrab then
            local dir = (hrp.Position - targetGrab.Position).Unit
            targetGrab.Velocity = dir * 60
        end
    end)
end

-- ربط المفاتيح المختلفة لتشكيل الأشكال
local forming = false
local function setupKeybinds()
    UserInputService.InputBegan:Connect(function(input, g)
        if g or forming then return end
        if input.KeyCode == Enum.KeyCode.E then
            forming = true
            formRocket()
            task.wait(1)
            forming = false
        elseif input.KeyCode == Enum.KeyCode.R then
            forming = true
            formLetterR()
            task.wait(1)
            forming = false
        elseif input.KeyCode == Enum.KeyCode.H then
            forming = true
            formPyramid()
            task.wait(1)
            forming = false
        elseif input.KeyCode == Enum.KeyCode.C then
            forming = true
            formSphere()
            task.wait(1)
            forming = false
        elseif input.KeyCode == Enum.KeyCode.B then
            forming = true
            formCube()
            task.wait(1)
            forming = false
        elseif input.KeyCode == Enum.KeyCode.Q then
            rotating = true
        end
    end)
end

-- تنفيذ السكربت عندما يتم تحميله
createBlocks()
rotateBlocks()
repelOtherPlayers()
createFloatingName()
setupGrab()
setupKeybinds()
task.spawn(updateColors)
for i = 1, 9000 do local x = i*math.random(); local y = x%7 end
