-- [[ ZONE HUB - BIKE FOR EGG (SMART COSMIC EGG DETECTOR & AUTO HOP) ]] --

local WindUI
local success, err = pcall(function()
    WindUI = loadstring(game:HttpGet("https://raw.githubusercontent.com/Footagesus/WindUI/main/dist/main.lua"))()
end)

if not success or not WindUI then
    WindUI = loadstring(game:HttpGet("https://tree-hub.net/scripts/windui.lua"))()
end

local Window = WindUI:CreateWindow({
    Title = "Zone Hub 🥚",
    Icon = "egg", 
    Author = "Pro Player", 
    Theme = "Dark",
})

Window:EditOpenButton({
    Title = "Open Hub",
    Icon = "monitor",
    CornerRadius = UDim.new(0,16),
    StrokeThickness = 2,
    Color = ColorSequence.new(
        Color3.fromHex("00FF87"), 
        Color3.fromHex("60EFFF")
    ),
    OnlyMobile = false,
    Enabled = true,
    Draggable = true,
})

local player = game.Players.LocalPlayer
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local RunService = game:GetService("RunService")

local function getHrp()
    local char = player.Character or player.CharacterAdded:Wait()
    return char:FindFirstChild("HumanoidRootPart")
end

local function getHomeCFrame()
    if workspace:FindFirstChild("Plots") or workspace:FindFirstChild("Bases") then
        local plots = workspace:FindFirstChild("Plots") or workspace:FindFirstChild("Bases")
        for _, plot in pairs(plots:GetChildren()) do
            if plot:FindFirstChild("Owner") and tostring(plot.Owner.Value) == player.Name then
                return plot:GetPivot()
            elseif plot.Name:find(player.Name) then
                return plot:GetPivot()
            end
        end
    end
    if workspace:FindFirstChild("SpawnLocation") then
        return workspace.SpawnLocation.CFrame + Vector3.new(0, 5, 0)
    end
    return nil
end

local function tpTo(cf)
    local hrp = getHrp()
    if hrp and cf then
        hrp.CFrame = cf + Vector3.new(0, 3, 0)
    end
end

-- --- TAB CÀY TRỨNG ---
local MainTab = Window:Tab({
    Title = "Cày Trứng",
    Icon = "sparkles",
})

MainTab:Button({
    Title = "🏠 Về Nhà Ngay",
    Desc = "Teleport thẳng về khu đất / base của bạn để cất trứng",
    Callback = function()
        local home = getHomeCFrame()
        if home then tpTo(home) end
    end
})

MainTab:Button({
    Title = "🌌 TP Đến Map Cuối Cosmic",
    Desc = "Bay thẳng tới khu vực trứng Cosmic xịn nhất map cuối",
    Callback = function()
        pcall(function()
            local targetPart = nil
            local hrp = getHrp()
            if not hrp then return end
            
            for _, obj in pairs(workspace:GetDescendants()) do
                if obj.Name:lower():find("cosmic") then
                    if obj:IsA("BasePart") then
                        targetPart = obj
                        break
                    elseif obj:IsA("Model") then
                        targetPart = obj.PrimaryPart or obj:FindFirstChildWhichIsA("BasePart")
                        if targetPart then break end
                    end
                end
            end
            
            if targetPart then
                local cf = targetPart:IsA("Model") and targetPart:GetPivot() or targetPart.CFrame
                tpTo(cf)
            end
        end)
    end
})

MainTab:Input({
    Title = "⚡ Chỉnh Tốc Độ Chạy (WalkSpeed)",
    Desc = "Gõ số vào đây rồi bấm Enter (VD: 50, 100...)",
    Placeholder = "Nhập tốc độ...",
    Value = "",
    Callback = function(text)
        pcall(function()
            local speed = tonumber(text)
            if speed then
                local char = player.Character
                if char and char:FindFirstChild("Humanoid") then
                    char.Humanoid.WalkSpeed = speed
                end
            end
        end)
    end
})

local autoTPHome = false
MainTab:Toggle({
    Title = "🏃‍♂️ Cầm Trứng Tự Động TP Về Nhà",
    Desc = "Bật 1 lần: Ra ngoài khu an toàn lụm trứng là tự động Teleport về nhà ngay lập tức!",
    Value = false,
    Callback = function(state)
        autoTPHome = state
    end
})

-- LOGIC AUTO TP VỀ NHÀ KHI CẦM TRỨNG
local FIXED_RADIUS = 300 
local lastTeleportedEgg = nil

task.spawn(function()
    while true do
        task.wait(0.15)
        if autoTPHome then
            pcall(function()
                local char = player.Character
                local hrp = getHrp()
                local homeCF = getHomeCFrame()
                
                if char and hrp then
                    local centerPos = (homeCF and homeCF.Position) or hrp.Position
                    local currentPos = hrp.Position
                    
                    local distToCenter = (Vector3.new(currentPos.X, 0, currentPos.Z) - Vector3.new(centerPos.X, 0, centerPos.Z)).Magnitude
                    
                    if distToCenter <= FIXED_RADIUS then
                        lastTeleportedEgg = nil
                    else
                        local holdingEgg = nil
                        for _, item in pairs(char:GetChildren()) do
                            if item:IsA("Tool") or item:IsA("Model") then
                                local name = item.Name:lower()
                                if (name:find("egg") or name:find("trứng")) and not name:find("basic") then
                                    holdingEgg = item
                                    break
                                end
                            end
                        end
                        
                        if holdingEgg then
                            local eggName = holdingEgg.Name
                            if lastTeleportedEgg ~= eggName then
                                tpTo(homeCF or CFrame.new(centerPos))
                                lastTeleportedEgg = eggName
                            end
                        else
                            lastTeleportedEgg = nil
                        end
                    end
                end
            end)
        else
            lastTeleportedEgg = nil
        end
    end
end)

-- --- TAB CHỐNG BỊ ĐẬP VĂNG ---
local AntiTab = Window:Tab({
    Title = "Chống Đập Bay",
    Icon = "shield-alert",
})

local antiKnockbackEnabled = false

AntiTab:Toggle({
    Title = "🛡️ Bật Chống Gậy Đập Văng",
    Desc = "Chống bị văng khi người khác dùng gậy đập, chạy nhảy vẫn mượt đéo bị giảm tốc độ!",
    Value = false,
    Callback = function(state)
        antiKnockbackEnabled = state
    end
})

RunService.Stepped:connect(function()
    if antiKnockbackEnabled then
        pcall(function()
            local char = player.Character
            local hrp = getHrp()
            if char and hrp then
                local hum = char:FindFirstChildOfClass("Humanoid")
                
                for _, child in pairs(hrp:GetChildren()) do
                    if child:IsA("BodyVelocity") or child:IsA("BodyForce") or child:IsA("BodyThrust") or child:IsA("LinearVelocity") then
                        child:Destroy()
                    end
                end
                
                local vel = hrp.AssemblyLinearVelocity
                if math.abs(vel.X) > 80 or math.abs(vel.Z) > 80 then
                    hrp.AssemblyLinearVelocity = Vector3.new(0, vel.Y, 0)
                end
                
                if hum then
                    hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
                    hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
                end
            end
        end)
    end
end)

-- --- HÀM HOP SERVER THÔNG MINH (QUÉT TRỨNG COSMIC SỐNG) ---
local function smartHopCosmic()
    pcall(function()
        WindUI:Notify({
            Title = "Đang quét Server 🛰️",
            Content = "Đang lọc phòng 1-2 người...",
            Duration = 2
        })
        
        local servers = {}
        local reqUrl = "https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Asc&limit=100"
        local req = game:HttpGet(reqUrl)
        local body = HttpService:JSONDecode(req)
        
        if body and body.data then
            for _, s in pairs(body.data) do
                if type(s) == "table" and s.id and s.playing and s.maxPlayers then
                    -- Ưu tiên phòng từ 1 đến 2 người chơi
                    if s.id ~= game.JobId and s.playing >= 1 and s.playing <= 2 then
                        table.insert(servers, s)
                    end
                end
            end
            
            if #servers == 0 then
                for _, s in pairs(body.data) do
                    if type(s) == "table" and s.id and s.playing and s.maxPlayers then
                        if s.id ~= game.JobId and s.playing < s.maxPlayers then
                            table.insert(servers, s)
                        end
                    end
                end
                table.sort(servers, function(a, b)
                    return a.playing < b.playing
                end)
            end
        end
        
        if #servers > 0 then
            local targetServer = servers[math.random(1, math.min(3, #servers))]
            TeleportService:TeleportToPlaceInstance(game.PlaceId, targetServer.id, player)
        else
            TeleportService:Teleport(game.PlaceId, player)
        end
    end)
end

-- --- TAB SERVER HOP & SĂN TRỨNG ---
local HopTab = Window:Tab({
    Title = "Server Hop VIP",
    Icon = "globe",
})

HopTab:Button({
    Title = "🌌 Săn Server 1-2 Người (Check Trứng Cosmic)",
    Desc = "Bấm để nhảy vào phòng vắng, tự động check xem có trứng Cosmic hay không!",
    Callback = function()
        smartHopCosmic()
    end
})

-- TÍNH NĂNG TỰ ĐỘNG CHECK TRỨNG KHI VỪA VÀO MAP (NẾU KHÔNG CÓ TRỨNG SẼ TỰ ĐỘNG HOP TIẾP)
local autoCosmicHunt = false

HopTab:Toggle({
    Title = "🤖 Auto Tự Động Săn Trứng Cosmic (Hop Liên Tục)",
    Desc = "Vừa vào map nếu trong 4 giây không thấy trứng Cosmic -> Tự động nhảy server khác tiếp!",
    Value = false,
    Callback = function(state)
        autoCosmicHunt = state
        if autoCosmicHunt then
            WindUI:Notify({Title = "Đã bật Auto Săn Trứng! 🚀", Content = "Đang quét trứng Cosmic trong phòng...", Duration = 3})
            
            task.spawn(function()
                task.wait(4) -- Đợi map load trong 4 giây
                if not autoCosmicHunt then return end
                
                local hasCosmic = false
                for _, obj in pairs(workspace:GetDescendants()) do
                    local name = obj.Name:lower()
                    if name:find("cosmic") then
                        hasCosmic = true
                        break
                    end
                end
                
                if not hasCosmic then
                    WindUI:Notify({Title = "Phòng này không có trứng xịn! ❌", Content = "Đang tự động nhảy sang server khác...", Duration = 2})
                    task.wait(1)
                    smartHopCosmic()
                else
                    WindUI:Notify({Title = "Thấy trứng Cosmic rồi! 🎉", Content = "Húp ngay kẻo muộn màng mày ơi!", Duration = 3})
                end
            end)
        end
    end
})

local function hopServer()
    pcall(function()
        local servers = {}
        local req = game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Asc&limit=100")
        local body = HttpService:JSONDecode(req)
        if body and body.data then
            for _, s in pairs(body.data) do
                if type(s) == "table" and s.id and s.playing and s.maxPlayers and s.playing < s.maxPlayers then
                    if s.id ~= game.JobId then
                        table.insert(servers, s.id)
                    end
                end
            end
        end
        if #servers > 0 then
            TeleportService:TeleportToPlaceInstance(game.PlaceId, servers[math.random(1, #servers)], player)
        else
            TeleportService:Teleport(game.PlaceId, player)
        end
    end)
end

local savedVIPJobId = ""

HopTab:Button({
    Title = "💾 Lưu Server VIP Hiện Tại",
    Desc = "Bấm nút này khi đang ở trong server VIP để ghi nhớ mã server",
    Callback = function()
        savedVIPJobId = game.JobId
        WindUI:Notify({
            Title = "Đã lưu! 💾",
            Content = "Đã lưu lại mã Server VIP hiện tại thành công!",
            Duration = 3
        })
    end
})

HopTab:Button({
    Title = "👑 Vào Lại Server VIP Đã Lưu",
    Desc = "Bấm phát bay thẳng lại vào server VIP bằng mã đã lưu",
    Callback = function()
        if savedVIPJobId ~= "" then
            pcall(function()
                TeleportService:TeleportToPlaceInstance(game.PlaceId, savedVIPJobId, player)
            end)
        else
            WindUI:Notify({
                Title = "Chưa lưu!",
                Content = "Bạn chưa bấm lưu server VIP hiện tại!",
                Duration = 3
            })
        end
    end
})

HopTab:Button({
    Title = "🔄 Đổi Server Thường Ngẫu Nhiên",
    Desc = "Nhảy sang một server khác bất kỳ",
    Callback = function()
        hopServer()
    end
})

-- --- TAB SERVER INFO ---
local InfoTab = Window:Tab({
    Title = "Server Info",
    Icon = "info",
})

local Paragraph = InfoTab:Paragraph({
    Title = "Thông tin chi tiết",
    Desc = "Đang tải...",
})

task.spawn(function()
    while task.wait(1) do
        local ping = game:GetService("Stats").Network.ServerStatsItem["Data Ping"]:GetValueString()
        local memory = math.floor(game:GetService("Stats"):GetTotalMemoryUsageMb())
        Paragraph:SetDesc(string.format("📶 Ping: %s\n💾 Memory: %dMB\n👥 Players: %d/%d\n🆔 JobID: %s", ping, memory, #game.Players:GetPlayers(), game.Players.MaxPlayers, game.JobId))
    end
end)

