local Players = game:GetService("Players") 
local ReplicatedStorage = game:GetService("ReplicatedStorage") 
local RunService = game:GetService("RunService") 
local lp = Players.LocalPlayer

-- GUI Setup 
local screenGui = Instance.new("ScreenGui") 
screenGui.Name = "BackstabToggleGui" 
screenGui.ResetOnSpawn = false 
screenGui.Parent = lp:WaitForChild("PlayerGui")

-- Toggle Button 
local toggleButton = Instance.new("TextButton") 
toggleButton.Size = UDim2.new(0, 150, 0, 40) 
toggleButton.Position = UDim2.new(0, 10, 0, 10) 
toggleButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30) 
toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255) 
toggleButton.Font = Enum.Font.SourceSansBold 
toggleButton.TextSize = 20 
toggleButton.Text = "Backstab: OFF" 
toggleButton.Parent = screenGui

-- Range Label 
local rangeLabel = Instance.new("TextLabel") 
rangeLabel.Size = UDim2.new(0, 150, 0, 20) 
rangeLabel.Position = UDim2.new(0, 10, 0, 55) 
rangeLabel.BackgroundTransparency = 1 
rangeLabel.TextColor3 = Color3.fromRGB(255, 255, 255) 
rangeLabel.Font = Enum.Font.SourceSans 
rangeLabel.TextSize = 16 
rangeLabel.Text = "Range:" 
rangeLabel.Parent = screenGui

-- TextBox for Range Input 
local rangeBox = Instance.new("TextBox") 
rangeBox.Size = UDim2.new(0, 150, 0, 25) 
rangeBox.Position = UDim2.new(0, 10, 0, 75) 
rangeBox.BackgroundColor3 = Color3.fromRGB(50, 50, 50) 
rangeBox.TextColor3 = Color3.fromRGB(255, 255, 255) 
rangeBox.Font = Enum.Font.SourceSans 
rangeBox.TextSize = 16 
rangeBox.PlaceholderText = "Enter range (number)" 
rangeBox.Text = "4" 
rangeBox.ClearTextOnFocus = false 
rangeBox.Parent = screenGui

-- Vars 
local enabled = false 
local cooldown = false 
local lastTarget = nil 
local range = 4 
local daggerRemote = ReplicatedStorage:WaitForChild("Modules"):WaitForChild("Network"):WaitForChild("RemoteEvent") 
local killerNames = { "Jason", "c00lkidd", "JohnDoe", "1x1x1x1", "Noli" } 
local killersFolder = workspace:WaitForChild("Players"):WaitForChild("Killers")

-- GUI toggle 
toggleButton.MouseButton1Click:Connect(function() 
    enabled = not enabled 
    toggleButton.Text = "Backstab: " .. (enabled and "ON" or "OFF") 
    toggleButton.BackgroundColor3 = enabled and Color3.fromRGB(50, 150, 50) or Color3.fromRGB(30, 30, 30) 
end)

-- TextBox Range Handling 
rangeBox.FocusLost:Connect(function() 
    local input = tonumber(rangeBox.Text) 
    if input and input >= 1 then 
        range = input 
    else 
        rangeBox.Text = tostring(range) 
    end 
end)

-- Mode Toggle 
local mode = "Behind" 
local modeButton = Instance.new("TextButton") 
modeButton.Size = UDim2.new(0, 150, 0, 25) 
modeButton.Position = UDim2.new(0, 10, 0, 105) 
modeButton.BackgroundColor3 = Color3.fromRGB(70, 70, 70) 
modeButton.TextColor3 = Color3.fromRGB(255, 255, 255) 
modeButton.Font = Enum.Font.SourceSans 
modeButton.TextSize = 16 
modeButton.Text = "Mode: Behind" 
modeButton.Parent = screenGui

modeButton.MouseButton1Click:Connect(function() 
    if mode == "Behind" then 
        mode = "Around" 
    else 
        mode = "Behind" 
    end 
    modeButton.Text = "Mode: " .. mode 
end)

-- Helper function 
local function isBehindTarget(hrp, targetHRP) 
    local distance = (hrp.Position - targetHRP.Position).Magnitude 
    if distance > range then return false end
    if mode == "Around" then 
        return true 
    else 
        local direction = -targetHRP.CFrame.LookVector 
        local toPlayer = (hrp.Position - targetHRP.Position) 
        return toPlayer:Dot(direction) > 0.5 
    end
end

-- Main loop
RunService.RenderStepped:Connect(function()
	if not enabled or cooldown then return end

	local char = lp.Character
	if not (char and char:FindFirstChild("HumanoidRootPart")) then return end
	local hrp = char.HumanoidRootPart
	local stats = game:GetService("Stats")

	for _, name in ipairs(killerNames) do
		local killer = killersFolder:FindFirstChild(name)
		if killer and killer:FindFirstChild("HumanoidRootPart") then
			local kHRP = killer.HumanoidRootPart

			if isBehindTarget(hrp, kHRP) and killer ~= lastTarget then
				cooldown = true
				lastTarget = killer

				local start = tick()
				local didDagger = false
				local connection
				connection = RunService.Heartbeat:Connect(function()
					if not (char and char.Parent and kHRP and kHRP.Parent) then
						if connection then connection:Disconnect() end
						return
					end

					local elapsed = tick() - start
					if elapsed >= 0.5 then
						if connection then connection:Disconnect() end
						return
					end

					-- LIVE Ping + velocity prediction
					local ping = tonumber(stats.Network.ServerStatsItem["Data Ping"]:GetValueString():match("%d+")) or 50
					local pingSeconds = ping / 1000
					local killerVelocity = kHRP.Velocity
					local moveDir = killerVelocity.Magnitude > 0.1 and killerVelocity.Unit or Vector3.new()
					local pingOffset = moveDir * (pingSeconds * killerVelocity.Magnitude)
					local predictedPos = kHRP.Position + pingOffset

					-- Apply mode logic with improved "Around" handling
					local targetPos
					if mode == "Behind" then
						targetPos = predictedPos - (kHRP.CFrame.LookVector * 0.3)
					elseif mode == "Around" then
						local lookVec = kHRP.CFrame.LookVector
						local rightVec = kHRP.CFrame.RightVector
						local rel = (hrp.Position - kHRP.Position)
						local lateralSpeed = killerVelocity:Dot(rightVec)
						
						local baseOffset = (rel.Magnitude > 0.1) and rel.Unit * 0.3 or Vector3.new()
						local lateralOffset = rightVec * lateralSpeed * 0.3

						targetPos = predictedPos + baseOffset + lateralOffset
					end

					-- Constant live TP
					hrp.CFrame = CFrame.new(targetPos, targetPos + kHRP.CFrame.LookVector)

					-- Only dagger once
					if not didDagger then
						didDagger = true

						-- Keep aligning for 0.7s
						local faceStart = tick()
						local faceConn
						faceConn = RunService.Heartbeat:Connect(function()
							if tick() - faceStart >= 0.7 or not kHRP or not kHRP.Parent then
								if faceConn then faceConn:Disconnect() end
								return
							end
							-- Live align during window
							local livePing = tonumber(stats.Network.ServerStatsItem["Data Ping"]:GetValueString():match("%d+")) or 50
							local livePingSeconds = livePing / 1000
							local liveVelocity = kHRP.Velocity
							local liveMoveDir = liveVelocity.Magnitude > 0.1 and liveVelocity.Unit or Vector3.new()
							local livePingOffset = liveMoveDir * (livePingSeconds * liveVelocity.Magnitude)
							local livePredictedPos = kHRP.Position + livePingOffset

							local liveTargetPos
							if mode == "Behind" then
								liveTargetPos = livePredictedPos - (kHRP.CFrame.LookVector * 0.3)
							elseif mode == "Around" then
								local lookVec = kHRP.CFrame.LookVector
								local rightVec = kHRP.CFrame.RightVector
								local liveRel = (hrp.Position - kHRP.Position)
								local liveLateralSpeed = liveVelocity:Dot(rightVec)
								
								local baseOffset = (liveRel.Magnitude > 0.1) and liveRel.Unit * 0.3 or Vector3.new()
								local lateralOffset = rightVec * liveLateralSpeed * 0.3

								liveTargetPos = livePredictedPos + baseOffset + lateralOffset
							end
							hrp.CFrame = CFrame.new(liveTargetPos, liveTargetPos + kHRP.CFrame.LookVector)
						end)

						daggerRemote:FireServer("UseActorAbility", "Dagger")
					end
				end)

				-- Reset cooldown when out of range
				task.delay(2, function()
					RunService.Heartbeat:Wait()
					while isBehindTarget(hrp, kHRP) do
						RunService.Heartbeat:Wait()
					end
					lastTarget = nil
					cooldown = false
				end)

				break
			end
		end
	end
end)
