--[[
    零域 (ZERO SECTOR) | 公開版啟動器
    功能：公開核心加載器
]]

local Players = game:GetService("Players")
local StarterGui = game:GetService("StarterGui")
local localPlayer = Players.LocalPlayer

-- 歡迎訊息函數
local function WelcomeUser()
    local name = localPlayer.Name
    
    -- 顯示歡迎通知
    StarterGui:SetCore("SendNotification", {
        Title = "零域 | 歡迎使用",
        Text = "歡迎: " .. name .. "\n核心模組加載中...",
        Duration = 5
    })
    
    print("[零域] 玩家 " .. name .. " 正在加載核心...")
end

-- 核心加載函數
local function LoadCore()
    -- 【可配置的核心路徑】你可以修改這裡的連結
    local coreUrls = {
        "https://raw.githubusercontent.com/xiaoxuan-77/-/refs/heads/main/零",
        "https://raw.githubusercontent.com/xiaoxuan-77/-/main/零",
        -- 可以添加更多備用連結
    }
    
    local success = false
    local lastError = ""
    
    -- 嘗試從多個來源加載
    for i, url in ipairs(coreUrls) do
        print("[零域] 嘗試從來源 " .. i .. " 加載: " .. url)
        
        local contentSuccess, content = pcall(function()
            return game:HttpGet(url, true) -- 使用 true 參數啟用緩存
        end)
        
        if contentSuccess and content and #content > 100 then -- 檢查內容長度
            print("[零域] 成功獲取核心腳本，長度: " .. #content .. " 字元")
            
            -- 嘗試執行腳本
            local func, err = loadstring(content)
            if func then
                print("[零域] 核心腳本語法檢查通過，正在執行...")
                local executeSuccess, executeError = pcall(func)
                if executeSuccess then
                    print("[零域] 核心腳本執行成功！")
                    success = true
                    break
                else
                    lastError = "執行錯誤: " .. tostring(executeError)
                    print("[零域] 錯誤: " .. lastError)
                end
            else
                lastError = "語法錯誤: " .. tostring(err)
                print("[零域] 錯誤: " .. lastError)
            end
        else
            if not contentSuccess then
                lastError = "HTTP 請求失敗: " .. tostring(content)
            elseif not content or #content <= 100 then
                lastError = "獲取內容過短或為空"
            end
            print("[零域] 從來源 " .. i .. " 加載失敗: " .. lastError)
        end
        
        -- 短暫延遲後嘗試下一個來源
        task.wait(0.5)
    end
    
    -- 如果所有來源都失敗
    if not success then
        warn("[零域] 所有核心加載嘗試均失敗")
        
        -- 顯示錯誤訊息
        StarterGui:SetCore("SendNotification", {
            Title = "零域 | 加載失敗",
            Text = "無法加載核心模組\n" .. lastError,
            Duration = 10,
            Icon = "rbxassetid://0"
        })
        
        -- 提供替代方案
        local choice = Instance.new("StringValue")
        choice.Name = "CoreLoadChoice"
        choice.Value = "retry"
        
        -- 可以添加重試邏輯或替代功能
    end
end

-- 初始化函數
local function Initialize()
    WelcomeUser()
    
    -- 短暫延遲後加載核心
    task.delay(1, function()
        LoadCore()
    end)
end

-- 安全初始化
local function SafeInitialize()
    local initSuccess, initError = pcall(Initialize)
    if not initSuccess then
        warn("[零域] 初始化錯誤: " .. tostring(initError))
        
        -- 顯示錯誤通知
        StarterGui:SetCore("SendNotification", {
            Title = "零域 | 啟動器錯誤",
            Text = "初始化失敗: " .. tostring(initError),
            Duration = 10,
            Icon = "rbxassetid://0"
        })
    end
end

-- 等待玩家載入完成
if localPlayer then
    SafeInitialize()
else
    Players.PlayerAdded:Wait()
    SafeInitialize()
end

-- 額外功能：簡單的 UI
task.spawn(function()
    task.wait(2)
    
    -- 創建簡單的控制台輸出
    print("====================================")
    print("      零域公開版啟動器 v1.0")
    print("====================================")
    print("玩家: " .. localPlayer.Name)
    print("用戶ID: " .. localPlayer.UserId)
    print("遊戲: " .. game:GetService("MarketplaceService"):GetProductInfo(game.PlaceId).Name)
    print("====================================")
end)
