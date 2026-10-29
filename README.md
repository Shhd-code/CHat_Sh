-- // SH Admin Ultimate Color Palette, Fonts & Movable UI 🐣⚙️🎨
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- حذف أي واجهة قديمة
if PlayerGui:FindFirstChild("SH_AdminChatUI") then
    PlayerGui.SH_AdminChatUI:Destroy()
end

-- متغيرات الإعدادات
local currentPrefix = ";to" -- البادئة الافتراضية
local isColoredChatEnabled = false -- حالة تفعيل ملون الشات
local selectedHexColor = "#FFFFFF" -- اللون الافتراضي (أبيض)
local currentTextSize = "18" -- حجم الخط الافتراضي
local selectedFont = "GothamBold" -- الخط الافتراضي

-- دالة للحصول على ريموت الأدمين بسرعة
local function GetAdminRemote()
    local success, remote = pcall(function()
        return ReplicatedStorage.HDAdminHDClient.Signals.RequestCommand
    end)
    if success and remote then return remote end

    for _, obj in ipairs(ReplicatedStorage:GetDescendants()) do
        if (obj:IsA("RemoteFunction") or obj:IsA("RemoteEvent")) and (obj.Name == "RequestCommand" or obj.Name == "Command") then
            return obj
        end
    end
    return nil
end

-- دالة عامة لجعل أي فريم قابل للسحب (Draggable)
local function makeDraggable(frame)
    local dragging, dragInput, dragStart, startPos
    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    frame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

-- إنشاء الواجهة الرئيسية
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SH_AdminChatUI"
ScreenGui.Parent = PlayerGui
ScreenGui.ResetOnSpawn = false

-- الإطار الأساسي (مستطيل الكتابة)
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 260, 0, 45)
MainFrame.Position = UDim2.new(1, -270, 0.35, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 50)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 10)
UICorner.Parent = MainFrame

local UIStroke = Instance.new("UIStroke")
UIStroke.Color = Color3.fromRGB(200, 50, 50) -- لمسة حمراء ستايل SH
UIStroke.Thickness = 1.5
UIStroke.Parent = MainFrame

-- خانة الكتابة الرئيسية
local TextBox = Instance.new("TextBox")
TextBox.Name = "CommandInput"
TextBox.Size = UDim2.new(1, -40, 1, 0)
TextBox.Position = UDim2.new(0, 10, 0, 0)
TextBox.BackgroundTransparency = 1
TextBox.TextColor3 = Color3.fromRGB(255, 255, 255)
TextBox.PlaceholderText = "اكتب رسالتك هنا..."
TextBox.PlaceholderColor3 = Color3.fromRGB(150, 150, 150)
TextBox.Text = ""
TextBox.TextSize = 14
TextBox.Font = Enum.Font.GothamBold
TextBox.TextXAlignment = Enum.TextXAlignment.Left
TextBox.Parent = MainFrame

-- زر الإعدادات (⚙️)
local SettingsButton = Instance.new("TextButton")
SettingsButton.Name = "SettingsButton"
SettingsButton.Size = UDim2.new(0, 30, 0, 30)
SettingsButton.Position = UDim2.new(1, -35, 0.5, -15)
SettingsButton.BackgroundColor3 = Color3.fromRGB(60, 60, 65)
SettingsButton.Text = "⚙️"
SettingsButton.TextSize = 14
SettingsButton.TextColor3 = Color3.fromRGB(255, 255, 255)
SettingsButton.BorderSizePixel = 0
SettingsButton.Parent = MainFrame

local ButtonCorner = Instance.new("UICorner")
ButtonCorner.CornerRadius = UDim.new(0, 8)
ButtonCorner.Parent = SettingsButton

-- ==================== قائمة الإعدادات الموسعة (صغّرنا طولها ورتبناها) ====================
local SettingsMenu = Instance.new("Frame")
SettingsMenu.Name = "SettingsMenu"
SettingsMenu.Size = UDim2.new(0, 280, 0, 235) -- صغّرنا الطول لتصير مرتبة أكثر
SettingsMenu.Position = UDim2.new(1, -280, 0.35, 50)
SettingsMenu.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
SettingsMenu.BorderSizePixel = 0
SettingsMenu.Visible = false
SettingsMenu.Parent = ScreenGui

-- تفعيل حركة سحب قائمة الإعدادات من أي مكان بها
makeDraggable(SettingsMenu)

local MenuCorner = Instance.new("UICorner")
MenuCorner.CornerRadius = UDim.new(0, 10)
MenuCorner.Parent = SettingsMenu

local MenuStroke = Instance.new("UIStroke")
MenuStroke.Color = Color3.fromRGB(200, 50, 50)
MenuStroke.Thickness = 1.5
MenuStroke.Parent = SettingsMenu

-- 1. زر تفعيل تحريك شريط الكتابة
local MoveToggleBtn = Instance.new("TextButton")
MoveToggleBtn.Name = "MoveToggleBtn"
MoveToggleBtn.Size = UDim2.new(1, -20, 0, 26)
MoveToggleBtn.Position = UDim2.new(0, 10, 0, 8)
MoveToggleBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 65)
MoveToggleBtn.Text = "تحريك شريط الكتابة: 🔓 (معطل)"
MoveToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
MoveToggleBtn.TextSize, MoveToggleBtn.Font = 11, Enum.Font.GothamBold
MoveToggleBtn.Parent = SettingsMenu

local MoveCorner = Instance.new("UICorner")
MoveCorner.CornerRadius = UDim.new(0, 6)
MoveCorner.Parent = MoveToggleBtn

-- منطق تحريك شريط الكتابة (من كل الجهات والمستطيل بالكامل)
local isMainFrameMovable = false
local draggingMain, dragInputMain, dragStartMain, startPosMain

MoveToggleBtn.MouseButton1Click:Connect(function()
    isMainFrameMovable = not isMainFrameMovable
    if isMainFrameMovable then
        MoveToggleBtn.Text = "تحريك شريط الكتابة: 🔒 (مفعل - اسحب من أي مكان)"
        MoveToggleBtn.BackgroundColor3 = Color3.fromRGB(50, 120, 50)
    else
        MoveToggleBtn.Text = "تحريك شريط الكتابة: 🔓 (معطل)"
        MoveToggleBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 65)
    end
end)

-- ربط حدث السحب لجميع أجزاء MainFrame (حتى يتحرك من كل جهة ومن داخل خانة الكتابة عند تفعيله)
local function setupFullFrameDrag(obj)
    obj.InputBegan:Connect(function(input)
        if isMainFrameMovable and (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) then
            draggingMain = true
            dragStartMain = input.Position
            startPosMain = MainFrame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    draggingMain = false
                end
            end)
        end
    end)
    obj.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInputMain = input
        end
    end)
end

setupFullFrameDrag(MainFrame)
setupFullFrameDrag(TextBox)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInputMain and draggingMain and isMainFrameMovable then
        local delta = input.Position - dragStartMain
        MainFrame.Position = UDim2.new(startPosMain.X.Scale, startPosMain.X.Offset + delta.X, startPosMain.Y.Scale, startPosMain.Y.Offset + delta.Y)
    end
end)

-- دالة إطفاء حركة شريط الكتابة تلقائياً عند الكتابة أو الضغط
local function disableMovementAfterAction()
    if isMainFrameMovable then
        isMainFrameMovable = false
        draggingMain = false
        MoveToggleBtn.Text = "تحريك شريط الكتابة: 🔓 (معطل)"
        MoveToggleBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 65)
    end
end

-- 2. مربع تغيير علامة الأمر (Prefix)
local PrefixBox = Instance.new("TextBox")
PrefixBox.Name = "PrefixBox"
PrefixBox.Size = UDim2.new(1, -20, 0, 26)
PrefixBox.Position = UDim2.new(0, 10, 0, 39)
PrefixBox.BackgroundColor3 = Color3.fromRGB(50, 50, 55)
PrefixBox.TextColor3 = Color3.fromRGB(255, 255, 255)
PrefixBox.Text = currentPrefix
PrefixBox.PlaceholderText = "تغيير علامة الأمر (مثال: ;to)"
PrefixBox.TextSize, PrefixBox.Font = 12, Enum.Font.GothamBold
PrefixBox.Parent = SettingsMenu

local PrefixCorner = Instance.new("UICorner")
PrefixCorner.CornerRadius = UDim.new(0, 6)
PrefixCorner.Parent = PrefixBox

PrefixBox.FocusLost:Connect(function()
    if PrefixBox.Text ~= "" then
        currentPrefix = PrefixBox.Text
    end
    disableMovementAfterAction()
end)

-- 3. زر تفعيل ملون الشات + مربع كتابة الحجم (Size)
local ToggleColoredChat = Instance.new("TextButton")
ToggleColoredChat.Name = "ToggleColoredChat"
ToggleColoredChat.Size = UDim2.new(0, 150, 0, 26)
ToggleColoredChat.Position = UDim2.new(0, 10, 0, 70)
ToggleColoredChat.BackgroundColor3 = Color3.fromRGB(70, 50, 50)
ToggleColoredChat.Text = "تفعيل ملون الشات🖌️: OFF"
ToggleColoredChat.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleColoredChat.TextSize, ToggleColoredChat.Font = 10, Enum.Font.GothamBold
ToggleColoredChat.Parent = SettingsMenu

local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(0, 6)
ToggleCorner.Parent = ToggleColoredChat

-- مربع كتابة حجم الخط (Size)
local SizeBox = Instance.new("TextBox")
SizeBox.Name = "SizeBox"
SizeBox.Size = UDim2.new(0, 98, 0, 26)
SizeBox.Position = UDim2.new(0, 168, 0, 70)
SizeBox.BackgroundColor3 = Color3.fromRGB(50, 50, 55)
SizeBox.TextColor3 = Color3.fromRGB(255, 255, 255)
SizeBox.Text = currentTextSize
SizeBox.PlaceholderText = "الحجم (مثال: 24)"
SizeBox.TextSize, SizeBox.Font = 11, Enum.Font.GothamBold
SizeBox.Parent = SettingsMenu

local SizeCorner = Instance.new("UICorner")
SizeCorner.CornerRadius = UDim.new(0, 6)
SizeCorner.Parent = SizeBox

SizeBox.FocusLost:Connect(function()
    if SizeBox.Text ~= "" then
        currentTextSize = SizeBox.Text
    end
    disableMovementAfterAction()
end)

ToggleColoredChat.MouseButton1Click:Connect(function()
    isColoredChatEnabled = not isColoredChatEnabled
    if isColoredChatEnabled then
        ToggleColoredChat.Text = "تفعيل ملون الشات🖌️: ON"
        ToggleColoredChat.BackgroundColor3 = Color3.fromRGB(50, 120, 50)
    else
        ToggleColoredChat.Text = "تفعيل ملون الشات🖌️: OFF"
        ToggleColoredChat.BackgroundColor3 = Color3.fromRGB(70, 50, 50)
    end
end)

-- ==================== شريط الألوان الأفقي (Horizontal Colors) ====================
local ColorContainer = Instance.new("ScrollingFrame")
ColorContainer.Name = "ColorContainer"
ColorContainer.Size = UDim2.new(1, -20, 0, 32)
ColorContainer.Position = UDim2.new(0, 10, 0, 101)
ColorContainer.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
ColorContainer.BorderSizePixel = 0
ColorContainer.CanvasSize = UDim2.new(0, 0, 0, 0)
ColorContainer.ScrollBarThickness = 2
ColorContainer.Parent = SettingsMenu

Instance.new("UICorner", ColorContainer).CornerRadius = UDim.new(0, 6)

local colorsList = {
    {Color3.fromRGB(255, 255, 255), "#FFFFFF"},
    {Color3.fromRGB(0, 255, 0), "#00FF00"},
    {Color3.fromRGB(255, 0, 127), "#FF007F"},
    {Color3.fromRGB(0, 255, 255), "#00FFFF"},
    {Color3.fromRGB(255, 255, 0), "#FFFF00"},
    {Color3.fromRGB(255, 128, 0), "#FF8000"},
    {Color3.fromRGB(128, 0, 255), "#8000FF"},
    {Color3.fromRGB(255, 100, 100), "#FF6464"},
    {Color3.fromRGB(100, 200, 255), "#64C8FF"},
    {Color3.fromRGB(255, 180, 100), "#FFB464"},
    {Color3.fromRGB(200, 100, 255), "#C864FF"},
    {Color3.fromRGB(100, 255, 180), "#64FFB4"},
    {Color3.fromRGB(150, 0, 0), "#960000"},
    {Color3.fromRGB(0, 0, 150), "#000096"},
    {Color3.fromRGB(0, 100, 0), "#006400"},
    {Color3.fromRGB(100, 0, 100), "#640064"},
    {Color3.fromRGB(80, 80, 80), "#505050"}
}

for i = 1, 30 do
    local r, g, b = math.random(50,255), math.random(50,255), math.random(50,255)
    local hex = string.format("#%02X%02X%02X", r, g, b)
    table.insert(colorsList, {Color3.fromRGB(r,g,b), hex})
end

for i, data in ipairs(colorsList) do
    local col = data[1]
    local hex = data[2]
    
    local colorBtn = Instance.new("TextButton")
    colorBtn.Size = UDim2.new(0, 26, 0, 22)
    colorBtn.Position = UDim2.new(0, (i - 1) * 30 + 5, 0, 5)
    colorBtn.BackgroundColor3 = col
    colorBtn.Text = ""
    colorBtn.Parent = ColorContainer
    
    local cCorner = Instance.new("UICorner")
    cCorner.CornerRadius = UDim.new(0, 4)
    cCorner.Parent = colorBtn
    
    colorBtn.MouseButton1Click:Connect(function()
        selectedHexColor = hex
        UIStroke.Color = col
    end)
end
ColorContainer.CanvasSize = UDim2.new(0, #colorsList * 30 + 10, 0, 0)

-- ==================== شريط الخطوط الأفقي (Horizontal Fonts) ====================
local FontContainer = Instance.new("ScrollingFrame")
FontContainer.Name = "FontContainer"
FontContainer.Size = UDim2.new(1, -20, 0, 32)
FontContainer.Position = UDim2.new(0, 10, 0, 138)
FontContainer.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
FontContainer.BorderSizePixel = 0
FontContainer.CanvasSize = UDim2.new(0, 0, 0, 0)
FontContainer.ScrollBarThickness = 2
FontContainer.Parent = SettingsMenu

Instance.new("UICorner", FontContainer).CornerRadius = UDim.new(0, 6)

local fonts = {}
for _, font in ipairs(Enum.Font:GetEnumItems()) do
    table.insert(fonts, font.Name)
end
table.sort(fonts)

for i, fName in ipairs(fonts) do
    local fBtn = Instance.new("TextButton", FontContainer)
    fBtn.Size = UDim2.new(0, 95, 0, 22)
    fBtn.Position = UDim2.new(0, (i - 1) * 102 + 5, 0, 5)
    fBtn.BackgroundColor3 = (fName == selectedFont) and Color3.fromRGB(200, 50, 50) or Color3.fromRGB(50, 50, 55)
    fBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    fBtn.Text = fName
    fBtn.TextSize = 11
    pcall(function() fBtn.Font = Enum.Font[fName] end)
    Instance.new("UICorner", fBtn).CornerRadius = UDim.new(0, 4)
    
    fBtn.MouseButton1Click:Connect(function()
        selectedFont = fName
        for _, child in ipairs(FontContainer:GetChildren()) do
            if child:IsA("TextButton") then
                child.BackgroundColor3 = Color3.fromRGB(50, 50, 55)
            end
        end
        fBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
    end)
end
FontContainer.CanvasSize = UDim2.new(0, #fonts * 102 + 10, 0, 0)

-- زر إغلاق / فتح الإعدادات بضغط ⚙️
SettingsButton.MouseButton1Click:Connect(function()
    SettingsMenu.Visible = not SettingsMenu.Visible
end)

-- ==================== معالجة الإرسال الفوري وإنشاء الكود ====================
local function SendCommand()
    local text = TextBox.Text
    text = string.gsub(text, "^%s*(.-)%s*$", "%1")
    
    if text and text ~= "" then
        local finalCommand = ""
        local baseText = ""
        if string.sub(text, 1, 1) == ";" then
            baseText = text
        else
            baseText = currentPrefix .. " " .. text
        end
        
        if isColoredChatEnabled then
            local actualMessage = text
            if string.sub(text, 1, 1) == ";" then
                actualMessage = text
            end
            finalCommand = currentPrefix .. ' <font face="' .. selectedFont .. '" color="' .. selectedHexColor .. '" size="' .. currentTextSize .. '"><b><i>' .. actualMessage .. '</i></b></font>'
        else
            finalCommand = baseText
        end
        
        local remote = GetAdminRemote()
        if remote then
            local args = { [1] = finalCommand }
            TextBox.Text = ""
            
            task.spawn(function()
                pcall(function()
                    if remote:IsA("RemoteFunction") then
                        remote:InvokeServer(unpack(args))
                    elseif remote:IsA("RemoteEvent") then
                        remote:FireServer(unpack(args))
                    end
                end)
            end)
        else
            warn("[SH Error] لم يتم العثور على ريموت الأدمين!")
        end
    end
    
    disableMovementAfterAction()
end

-- الإرسال الفوري بضغط زر Enter مع إطفاء الحركة
TextBox.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        SendCommand()
    end
    disableMovementAfterAction()
end)

TextBox.Focused:Connect(function()
    disableMovementAfterAction()
end)

print("--- [SH] Ultimate Movable UI Loaded Successfully! ---")
