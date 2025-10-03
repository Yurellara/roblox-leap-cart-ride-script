local pl = game:GetService("Players").LocalPlayer
local uis = game:GetService("UserInputService")
local rs = game:GetService("RunService")
local ws = game:GetService("Workspace")
local PlayerGui = pl:WaitForChild("PlayerGui")
local ragdolling = false

local gui = Instance.new("ScreenGui")
gui.ResetOnSpawn = false
gui.Parent = PlayerGui

local btn = Instance.new("ImageButton")
btn.Size = UDim2.new(0,160,0,120)
btn.Position = UDim2.new(1,-80,0.65,-60)
btn.AnchorPoint = Vector2.new(1,0.5)
btn.BackgroundTransparency = 1
btn.Image = "rbxassetid://71963367856281"
btn.ZIndex = 10
btn.Parent = gui

local chr, hum, hrp

local function setupChar(c)
	chr = c
	hum = chr:WaitForChild("Humanoid")
	hrp = chr:WaitForChild("HumanoidRootPart")
end

setupChar(pl.Character or pl.CharacterAdded:Wait())
pl.CharacterAdded:Connect(setupChar)

local savedAnimator
local function ragdoll(state)
	if not chr then return end
	for _,m in ipairs(chr:GetChildren()) do
		if m:IsA("Motor6D") then
			m.Enabled = not state
		end
	end
	hum:ChangeState(state and Enum.HumanoidStateType.Physics or Enum.HumanoidStateType.GettingUp)
	if state then
		local animator = hum:FindFirstChildOfClass("Animator")
		if animator then
			savedAnimator = animator:Clone()
			animator:Destroy()
		end
	else
		if savedAnimator then
			savedAnimator.Parent = hum
			savedAnimator = nil
		end
	end
end

local flightConn
local function stopFlight()
	if flightConn then
		flightConn:Disconnect()
		flightConn = nil
	end
	if hrp then hrp.AssemblyLinearVelocity = Vector3.zero end
	task.delay(1,function()
		if ragdolling then
			ragdoll(false)
			ragdolling = false
		end
	end)
end

local function fling()
	if ragdolling or not hrp then return end
	ragdolling = true
	ragdoll(true)
	local look = hrp.CFrame.LookVector.Unit
	local dist = 80
	local peak = 20
	local g = ws.Gravity
	local timeTotal = math.sqrt((8*peak)/g)*1.5
	local vx = dist/timeTotal
	local vy = g*(timeTotal/2)
	hrp.AssemblyLinearVelocity = look*vx + Vector3.new(0,vy,0)
	local startTime = tick()
	flightConn = rs.Heartbeat:Connect(function()
		if not hrp or not hrp.Parent then
			stopFlight()
			return
		end
		local t = tick()-startTime
		local alpha = math.clamp(t/timeTotal,0,1)
		local pos = hrp.Position
		local facing = CFrame.lookAt(pos,pos+look)
		local flip = CFrame.Angles(math.rad(-180*alpha),0,0)
		hrp.CFrame = facing*flip
		if #hrp:GetTouchingParts() > 0 then
			stopFlight()
			return
		end
		if alpha>=1 then
			stopFlight()
			return
		end
	end)
end

uis.InputBegan:Connect(function(i,gp)
	if gp then return end
	if i.KeyCode==Enum.KeyCode.P then
		fling()
	end
end)

btn.MouseButton1Click:Connect(fling)
