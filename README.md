-- SETTINGS
local flySpeed = 50
local guiSize = Vector2.new(140, 40)
local guiPos = Vector2.new(20, 100)

-- Fly state
local flying = false
local flyThread = nil

-- Player setup
local player = getlocalplayer()
local character = getcharacter(player)
local root = getprimarypart(character)
local camera = findservice(Game, "Workspace").CurrentCamera
local UIS = findservice(Game, "UserInputService")

-- Movement direction table
local dir = {
    forward = false,
    backward = false,
    left = false,
    right = false,
    up = false,
    down = false
}

-- Drawing GUI
local flyButton = Drawing.new("Square")
flyButton.Size = guiSize
flyButton.Position = guiPos
flyButton.Color = Color3.fromRGB(30, 30, 30)
flyButton.Filled = true
flyButton.Visible = true
flyButton.Thickness = 1

local flyText = Drawing.new("Text")
flyText.Text = "[FLY: OFF]"
flyText.Size = 20
flyText.Position = guiPos + Vector2.new(10, 10)
flyText.Color = Color3.fromRGB(255, 255, 255)
flyText.Center = false
flyText.Outline = true
flyText.Visible = true

-- Start/stop fly function
local function setFly(state)
    flying = state
    flyText.Text = flying and "[FLY: ON]" or "[FLY: OFF]"

    if flying then
        flyThread = thread.create("fly_gui_thread", function()
            while flying do
                local look = getlookvector(camera)
                local right = getrightvector(camera)
                local up = getupvector(camera)

                local vel = {x=0,y=0,z=0}
                if dir.forward then vel.x += look.x vel.y += look.y vel.z += look.z end
                if dir.backward then vel.x -= look.x vel.y -= look.y vel.z -= look.z end
                if dir.left then vel.x -= right.x vel.y -= right.y vel.z -= right.z end
                if dir.right then vel.x += right.x vel.y += right.y vel.z += right.z end
                if dir.up then vel.x += up.x vel.y += up.y vel.z += up.z end
                if dir.down then vel.x -= up.x vel.y -= up.y vel.z -= up.z end

                local mag = math.sqrt(vel.x^2 + vel.y^2 + vel.z^2)
                if mag > 0 then
                    vel.x = vel.x / mag * flySpeed
                    vel.y = vel.y / mag * flySpeed
                    vel.z = vel.z / mag * flySpeed
                end

                setvelocity(root, {vel.x, vel.y, vel.z})
                wait()
            end
            setvelocity(root, {0, 0, 0})
        end)
    else
        thread.terminate("fly_gui_thread")
        setvelocity(root, {0, 0, 0})
    end
end

-- Input handling
UIS.InputBegan:Connect(function(input)
    local code = input.KeyCode
    if code == Enum.KeyCode.W then dir.forward = true end
    if code == Enum.KeyCode.S then dir.backward = true end
    if code == Enum.KeyCode.A then dir.left = true end
    if code == Enum.KeyCode.D then dir.right = true end
    if code == Enum.KeyCode.Space then dir.up = true end
    if code == Enum.KeyCode.LeftControl then dir.down = true end
    if code == Enum.KeyCode.E then setFly(not flying) end
end)

UIS.InputEnded:Connect(function(input)
    local code = input.KeyCode
    if code == Enum.KeyCode.W then dir.forward = false end
    if code == Enum.KeyCode.S then dir.backward = false end
    if code == Enum.KeyCode.A then dir.left = false end
    if code == Enum.KeyCode.D then dir.right = false end
    if code == Enum.KeyCode.Space then dir.up = false end
    if code == Enum.KeyCode.LeftControl then dir.down = false end
end)

-- GUI click detection
local Mouse = findservice(Game, "Players").LocalPlayer:GetMouse()

Mouse.Button1Down:Connect(function()
    local mPos = Vector2.new(Mouse.X, Mouse.Y)
    if mPos.X >= guiPos.X and mPos.X <= guiPos.X + guiSize.X and
       mPos.Y >= guiPos.Y and mPos.Y <= guiPos.Y + guiSize.Y then
        setFly(not flying)
    end
end)
