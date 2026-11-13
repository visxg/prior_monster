-- Pumpkin ESP + Pathfind | Splix-style menu
-- LocalScript -> StarterPlayer > StarterPlayerScripts

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local TextService = game:GetService("TextService")
local GuiService = game:GetService("GuiService")
local PathfindingService = game:GetService("PathfindingService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local folder = workspace:WaitForChild("CharacterIgnore")

local TARGET = "pumpkinmodel"            -- pumpkin model name (case-insensitive)
local CHARACTER_NAME = "nC0Zmil7fdGQJ2"  -- your model in workspace.Characters ("" = auto-detect)
local ORANGE = Color3.fromRGB(255, 140, 0)

local Settings = {
	Enabled = true,
	Fill = true,
	Outline = true,
	Labels = true,
	Distance = true,
	FillColor = ORANGE,
	OutlineColor = ORANGE,
	LabelColor = ORANGE,
	FillTransparency = 0.5,
	OutlineTransparency = 0,
	MaxDistance = 1000,
	MaxHighlights = 31,
	TextSize = 14,

	ShowPath = true,
	ShowJumps = true,
	PathColor = ORANGE,
	PathRate = 0.5,
	AgentRadius = 2,

	MenuKey = Enum.KeyCode.Insert,
	EspKey = nil,
	Watermark = true,
}

local Theme = {
	Accent = ORANGE,
	Light = Color3.fromRGB(30, 30, 30),
	Dark = Color3.fromRGB(20, 20, 20),
	Outline = Color3.fromRGB(0, 0, 0),
	Inline = Color3.fromRGB(50, 50, 50),
	Text = Color3.fromRGB(255, 255, 255),
	SubText = Color3.fromRGB(170, 170, 170),
	Font = Enum.Font.Code,
	TextSize = 13,
}

--==================== Core helpers ====================
local unloaded = false
local connections = {}
local function bind(signal, fn)
	local c = signal:Connect(fn)
	table.insert(connections, c)
	return c
end

-- mouse position in the same space as GuiObject.AbsolutePosition
local function mousePos()
	return UserInputService:GetMouseLocation() - GuiService:GetGuiInset()
end

local accentListeners = {}
local function onAccent(fn)
	table.insert(accentListeners, fn)
	fn(Theme.Accent)
end
local function setAccent(c)
	Theme.Accent = c
	for _, fn in ipairs(accentListeners) do fn(c) end
end

local function create(class, props, children)
	local obj = Instance.new(class)
	local parent
	for k, v in pairs(props or {}) do
		if k == "Parent" then parent = v else obj[k] = v end
	end
	for _, c in ipairs(children or {}) do c.Parent = obj end
	if parent then obj.Parent = parent end
	return obj
end

local function pad(px)
	return create("UIPadding", {
		PaddingTop = UDim.new(0, px), PaddingBottom = UDim.new(0, px),
		PaddingLeft = UDim.new(0, px), PaddingRight = UDim.new(0, px),
	})
end

local function gradient(parent)
	return create("UIGradient", {
		Rotation = 90,
		Color = ColorSequence.new(Color3.new(1, 1, 1), Color3.fromRGB(165, 165, 165)),
		Parent = parent,
	})
end

local function label(props)
	local t = create("TextLabel", {
		BackgroundTransparency = 1, Font = Theme.Font, TextSize = Theme.TextSize,
		TextColor3 = Theme.Text, TextStrokeColor3 = Theme.Outline, TextStrokeTransparency = 0,
		TextXAlignment = Enum.TextXAlignment.Left,
	})
	for k, v in pairs(props) do
		if k ~= "Parent" then t[k] = v end
	end
	t.Parent = props.Parent
	return t
end

-- Splix-style 3-layer box: outline (black) > inline (grey) > fill
local function box(class, props, fillColor, auto)
	local outer = Instance.new(class)
	outer.BackgroundColor3 = Theme.Outline
	outer.BorderSizePixel = 0
	if outer:IsA("TextButton") then
		outer.Text = ""
		outer.AutoButtonColor = false
	end
	for k, v in pairs(props) do
		if k ~= "Parent" then outer[k] = v end
	end
	local size = auto and UDim2.new(1, 0, 0, 0) or UDim2.fromScale(1, 1)
	local autoSize = auto and Enum.AutomaticSize.Y or Enum.AutomaticSize.None
	if auto then outer.AutomaticSize = Enum.AutomaticSize.Y end
	pad(1).Parent = outer
	local inline = create("Frame", {
		Size = size, AutomaticSize = autoSize, BackgroundColor3 = Theme.Inline, BorderSizePixel = 0, Parent = outer,
	}, { pad(1) })
	local fill = create("Frame", {
		Size = size, AutomaticSize = autoSize, BackgroundColor3 = fillColor or Theme.Light, BorderSizePixel = 0, Parent = inline,
	})
	outer.Parent = props.Parent
	return outer, fill
end

local order = 0
local function nextOrder()
	order += 1
	return order
end

local function textWidth(text)
	return TextService:GetTextSize(text, Theme.TextSize, Theme.Font, Vector2.new(1000, 100)).X
end

--==================== Window ====================
local gui = create("ScreenGui", {
	Name = "PumpkinESP", ResetOnSpawn = false, IgnoreGuiInset = true, DisplayOrder = 999,
	ZIndexBehavior = Enum.ZIndexBehavior.Sibling, Parent = playerGui,
})

local W, H = 520, 460
local main = create("Frame", {
	Size = UDim2.fromOffset(W, H), Position = UDim2.new(0.5, -W / 2, 0.5, -H / 2),
	BackgroundColor3 = Theme.Outline, BorderSizePixel = 0, Parent = gui,
}, { pad(1) })

local accentBorder = create("Frame", { Size = UDim2.fromScale(1, 1), BorderSizePixel = 0, Parent = main }, { pad(1) })
onAccent(function(c) accentBorder.BackgroundColor3 = c end)

local inner = create("Frame", { Size = UDim2.fromScale(1, 1), BackgroundColor3 = Theme.Light, BorderSizePixel = 0, Parent = accentBorder })
create("TextButton", { Modal = true, Size = UDim2.fromOffset(0, 0), BackgroundTransparency = 1, Text = "", Parent = inner })

local titleBar = create("Frame", { Size = UDim2.new(1, 0, 0, 20), BackgroundTransparency = 1, Active = true, Parent = inner })
local titleText = label({ Parent = titleBar, RichText = true, Position = UDim2.fromOffset(6, 0), Size = UDim2.new(1, -12, 1, 0) })
onAccent(function(c)
	titleText.Text = string.format('Pumpkin <font color="#%s">ESP</font>', c:ToHex())
end)
local keyHint = label({
	Parent = titleBar, Position = UDim2.fromOffset(6, 0), Size = UDim2.new(1, -12, 1, 0),
	TextColor3 = Theme.SubText, TextXAlignment = Enum.TextXAlignment.Right,
})

local _, body = box("Frame", { Position = UDim2.fromOffset(4, 20), Size = UDim2.new(1, -8, 1, -24), Parent = inner }, Theme.Dark)

local tabRow = create("Frame", {
	Position = UDim2.fromOffset(4, 4), Size = UDim2.new(1, -8, 0, 21), BackgroundTransparency = 1, Parent = body,
}, {
	create("UIListLayout", { FillDirection = Enum.FillDirection.Horizontal, Padding = UDim.new(0, 2), SortOrder = Enum.SortOrder.LayoutOrder }),
})

local _, pageArea = box("Frame", { Position = UDim2.fromOffset(4, 27), Size = UDim2.new(1, -8, 1, -31), Parent = body }, Theme.Light)

-- window dragging
do
	local dragging, startMouse, startPos = false, nil, nil
	titleBar.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
			startMouse = mousePos()
			startPos = main.Position
		end
	end)
	bind(UserInputService.InputChanged, function(input)
		if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
			local d = mousePos() - startMouse
			main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
		end
	end)
	bind(UserInputService.InputEnded, function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
	end)
end

--==================== Tabs & sections ====================
local tabs = {}

local function selectTab(tab)
	for _, t in ipairs(tabs) do
		local on = t == tab
		t.page.Visible = on
		t.fill.BackgroundColor3 = on and Theme.Light or Theme.Dark
		t.text.TextColor3 = on and Theme.Text or Theme.SubText
		t.line.Visible = on
	end
end

local function column(page, side)
	return create("ScrollingFrame", {
		Position = side == "Left" and UDim2.fromOffset(6, 6) or UDim2.new(0.5, 3, 0, 6),
		Size = UDim2.new(0.5, -9, 1, -12),
		BackgroundTransparency = 1, BorderSizePixel = 0,
		ScrollBarThickness = 2, ScrollBarImageColor3 = Theme.Inline,
		CanvasSize = UDim2.new(), AutomaticCanvasSize = Enum.AutomaticSize.Y,
		ScrollingDirection = Enum.ScrollingDirection.Y, Parent = page,
	}, {
		create("UIListLayout", { Padding = UDim.new(0, 8), SortOrder = Enum.SortOrder.LayoutOrder }),
		create("UIPadding", { PaddingRight = UDim.new(0, 4) }),
	})
end

local function Tab(name)
	local outer, fill = box("TextButton", {
		Size = UDim2.fromOffset(textWidth(name) + 24, 21), LayoutOrder = #tabs + 1, Parent = tabRow,
	}, Theme.Dark)
	gradient(fill)
	local line = create("Frame", { Size = UDim2.new(1, 0, 0, 1), BorderSizePixel = 0, Visible = false, Parent = fill })
	onAccent(function(c) line.BackgroundColor3 = c end)
	local text = label({ Parent = fill, Size = UDim2.fromScale(1, 1), Text = name, TextXAlignment = Enum.TextXAlignment.Center })
	local page = create("Frame", { Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, Visible = false, Parent = pageArea })

	local tab = { fill = fill, text = text, line = line, page = page, Left = column(page, "Left"), Right = column(page, "Right") }
	table.insert(tabs, tab)
	outer.MouseButton1Click:Connect(function() selectTab(tab) end)
	if #tabs == 1 then selectTab(tab) end
	return tab
end

local function Section(col, name)
	local _, fill = box("Frame", { Size = UDim2.new(1, 0, 0, 0), LayoutOrder = nextOrder(), Parent = col }, Theme.Dark, true)
	create("UIListLayout", { SortOrder = Enum.SortOrder.LayoutOrder, Parent = fill })
	local bar = create("Frame", { Size = UDim2.new(1, 0, 0, 2), BorderSizePixel = 0, LayoutOrder = 1, Parent = fill })
	onAccent(function(c) bar.BackgroundColor3 = c end)
	local content = create("Frame", {
		Size = UDim2.new(1, 0, 0, 0), AutomaticSize = Enum.AutomaticSize.Y,
		BackgroundTransparency = 1, LayoutOrder = 2, Parent = fill,
	}, {
		create("UIListLayout", { Padding = UDim.new(0, 5), SortOrder = Enum.SortOrder.LayoutOrder }),
		create("UIPadding", {
			PaddingTop = UDim.new(0, 4), PaddingBottom = UDim.new(0, 8),
			PaddingLeft = UDim.new(0, 6), PaddingRight = UDim.new(0, 6),
		}),
	})
	label({ Parent = content, Size = UDim2.new(1, 0, 0, 14), Text = name, LayoutOrder = 0 })
	return content
end

--==================== Elements ====================
local function Toggle(sec, name, default, callback)
	local state = default
	local row = create("TextButton", {
		Size = UDim2.new(1, 0, 0, 15), BackgroundTransparency = 1, Text = "",
		AutoButtonColor = false, LayoutOrder = nextOrder(), Parent = sec,
	})
	local _, fill = box("Frame", { Position = UDim2.fromOffset(0, 1), Size = UDim2.fromOffset(13, 13), Parent = row }, Theme.Light)
	gradient(fill)
	local text = label({ Parent = row, Position = UDim2.fromOffset(20, 0), Size = UDim2.new(1, -20, 1, 0), Text = name })

	local function render()
		fill.BackgroundColor3 = state and Theme.Accent or Theme.Light
		text.TextColor3 = state and Theme.Text or Theme.SubText
	end
	local function set(value)
		state = value
		render()
		callback(value)
	end
	onAccent(render)
	row.MouseButton1Click:Connect(function() set(not state) end)
	return { Set = set, Get = function() return state end }
end

local function Slider(sec, name, min, max, default, step, suffix, callback)
	local row = create("Frame", { Size = UDim2.new(1, 0, 0, 31), BackgroundTransparency = 1, LayoutOrder = nextOrder(), Parent = sec })
	label({ Parent = row, Size = UDim2.new(1, 0, 0, 14), Text = name })
	local track, fill = box("TextButton", { Position = UDim2.fromOffset(0, 17), Size = UDim2.new(1, 0, 0, 14), Parent = row }, Theme.Light)
	gradient(fill)
	local bar = create("Frame", { Size = UDim2.fromScale(0, 1), BorderSizePixel = 0, Parent = fill })
	gradient(bar)
	onAccent(function(c) bar.BackgroundColor3 = c end)
	local valueText = label({ Parent = fill, Size = UDim2.fromScale(1, 1), TextSize = 12, TextXAlignment = Enum.TextXAlignment.Center })

	local function fmt(x)
		if step < 1 then return string.format("%.1f", x) end
		return tostring(math.floor(x + 0.5))
	end
	local function set(x)
		x = math.clamp(math.floor(x / step + 0.5) * step, min, max)
		bar.Size = UDim2.fromScale((x - min) / (max - min), 1)
		valueText.Text = fmt(x) .. suffix .. " / " .. fmt(max) .. suffix
		callback(x)
	end
	local function fromMouse()
		local mx = mousePos().X
		set(min + (max - min) * math.clamp((mx - fill.AbsolutePosition.X) / fill.AbsoluteSize.X, 0, 1))
	end

	local dragging = false
	track.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
			fromMouse()
		end
	end)
	bind(UserInputService.InputChanged, function(input)
		if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then fromMouse() end
	end)
	bind(UserInputService.InputEnded, function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
	end)
	set(default)
end

local function Button(sec, name, callback)
	local outer, fill = box("TextButton", { Size = UDim2.new(1, 0, 0, 20), LayoutOrder = nextOrder(), Parent = sec }, Theme.Light)
	gradient(fill)
	local text = label({ Parent = fill, Size = UDim2.fromScale(1, 1), Text = name, TextXAlignment = Enum.TextXAlignment.Center })
	outer.MouseEnter:Connect(function() fill.BackgroundColor3 = Color3.fromRGB(40, 40, 40) end)
	outer.MouseLeave:Connect(function() fill.BackgroundColor3 = Theme.Light end)
	outer.MouseButton1Click:Connect(function()
		text.TextColor3 = Theme.Accent
		task.delay(0.15, function() text.TextColor3 = Theme.Text end)
		callback()
	end)
end

local function Label(sec, text, dim)
	return label({
		Parent = sec, Size = UDim2.new(1, 0, 0, 14), Text = text, TextTruncate = Enum.TextTruncate.AtEnd,
		TextColor3 = dim and Theme.SubText or Theme.Text, LayoutOrder = nextOrder(),
	})
end

local shortNames = {
	Insert = "Ins", Delete = "Del", Home = "Home", End = "End", PageUp = "PgUp", PageDown = "PgDn",
	LeftControl = "LCtrl", RightControl = "RCtrl", LeftShift = "LShift", RightShift = "RShift",
	LeftAlt = "LAlt", RightAlt = "RAlt", CapsLock = "Caps", Backspace = "Back", Return = "Enter",
}
local function keyName(k)
	if k == nil then return "None" end
	return shortNames[k.Name] or k.Name
end

local listening = nil

local function Keybind(sec, name, default, allowNone, callback)
	local kb = { key = default, allowNone = allowNone }
	local row = create("Frame", { Size = UDim2.new(1, 0, 0, 17), BackgroundTransparency = 1, LayoutOrder = nextOrder(), Parent = sec })
	label({ Parent = row, Size = UDim2.new(1, -60, 1, 0), Text = name })
	local outer, fill = box("TextButton", { Position = UDim2.new(1, -56, 0, 0), Size = UDim2.fromOffset(56, 17), Parent = row }, Theme.Light)
	gradient(fill)
	local text = label({ Parent = fill, Size = UDim2.fromScale(1, 1), TextSize = 12, TextXAlignment = Enum.TextXAlignment.Center })

	function kb.render()
		text.Text = "[" .. keyName(kb.key) .. "]"
		text.TextColor3 = Theme.Text
	end
	function kb.Set(k)
		kb.key = k
		kb.render()
		callback(k)
	end
	outer.MouseButton1Click:Connect(function()
		if listening and listening ~= kb then listening.render() end
		listening = kb
		text.Text = "[...]"
		text.TextColor3 = Theme.Accent
	end)
	kb.render()
	return kb
end

local function ColorPicker(sec, name, default, callback)
	local h, s, v = default:ToHSV()

	local holder = create("Frame", {
		Size = UDim2.new(1, 0, 0, 0), AutomaticSize = Enum.AutomaticSize.Y,
		BackgroundTransparency = 1, LayoutOrder = nextOrder(), Parent = sec,
	}, {
		create("UIListLayout", { Padding = UDim.new(0, 4), SortOrder = Enum.SortOrder.LayoutOrder }),
	})
	local row = create("Frame", { Size = UDim2.new(1, 0, 0, 15), BackgroundTransparency = 1, LayoutOrder = 1, Parent = holder })
	label({ Parent = row, Size = UDim2.new(1, -36, 1, 0), Text = name })
	local swatch, swatchFill = box("TextButton", { Position = UDim2.new(1, -30, 0, 0), Size = UDim2.fromOffset(30, 15), Parent = row }, default)
	gradient(swatchFill)

	local panel, panelFill = box("Frame", { Size = UDim2.new(1, 0, 0, 130), LayoutOrder = 2, Visible = false, Parent = holder }, Theme.Dark)
	local topBar = create("Frame", { Size = UDim2.new(1, 0, 0, 2), BorderSizePixel = 0, Parent = panelFill })
	onAccent(function(c) topBar.BackgroundColor3 = c end)

	local sv, svFill = box("TextButton", { Position = UDim2.fromOffset(4, 6), Size = UDim2.new(1, -30, 1, -10), Parent = panelFill }, Color3.fromHSV(h, 1, 1))
	create("Frame", { Size = UDim2.fromScale(1, 1), BackgroundColor3 = Color3.new(1, 1, 1), BorderSizePixel = 0, Parent = svFill }, {
		create("UIGradient", { Transparency = NumberSequence.new(0, 1) }),
	})
	create("Frame", { Size = UDim2.fromScale(1, 1), BackgroundColor3 = Color3.new(0, 0, 0), BorderSizePixel = 0, Parent = svFill }, {
		create("UIGradient", { Rotation = 90, Transparency = NumberSequence.new(1, 0) }),
	})
	local svCursor = create("Frame", {
		Size = UDim2.fromOffset(6, 6), AnchorPoint = Vector2.new(0.5, 0.5),
		BackgroundTransparency = 1, BorderSizePixel = 0, Parent = svFill,
	}, {
		create("UIStroke", { Color = Color3.new(1, 1, 1), Thickness = 1.5 }),
		create("Frame", { Size = UDim2.new(1, 4, 1, 4), Position = UDim2.fromOffset(-2, -2), BackgroundTransparency = 1 }, {
			create("UIStroke", { Color = Color3.new(0, 0, 0) }),
		}),
	})

	local hue, hueFill = box("TextButton", { Position = UDim2.new(1, -22, 0, 6), Size = UDim2.new(0, 18, 1, -10), Parent = panelFill }, Color3.new(1, 1, 1))
	local keys = {}
	for i = 0, 6 do
		table.insert(keys, ColorSequenceKeypoint.new(i / 6, Color3.fromHSV(i / 6, 1, 1)))
	end
	create("UIGradient", { Rotation = 90, Color = ColorSequence.new(keys), Parent = hueFill })
	local hueCursor = create("Frame", {
		Size = UDim2.new(1, 0, 0, 3), AnchorPoint = Vector2.new(0, 0.5),
		BackgroundColor3 = Color3.new(1, 1, 1), BorderSizePixel = 0, Parent = hueFill,
	}, { create("UIStroke", { Color = Color3.new(0, 0, 0) }) })

	local function update(fire)
		local c = Color3.fromHSV(h, s, v)
		svFill.BackgroundColor3 = Color3.fromHSV(h, 1, 1)
		svCursor.Position = UDim2.fromScale(s, 1 - v)
		hueCursor.Position = UDim2.fromScale(0, h)
		swatchFill.BackgroundColor3 = c
		if fire then callback(c) end
	end

	local dragSV, dragHue = false, false
	local function fromMouse()
		local m = mousePos()
		if dragSV then
			s = math.clamp((m.X - svFill.AbsolutePosition.X) / svFill.AbsoluteSize.X, 0, 1)
			v = 1 - math.clamp((m.Y - svFill.AbsolutePosition.Y) / svFill.AbsoluteSize.Y, 0, 1)
			update(true)
		elseif dragHue then
			h = math.clamp((m.Y - hueFill.AbsolutePosition.Y) / hueFill.AbsoluteSize.Y, 0, 0.999)
			update(true)
		end
	end

	sv.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragSV = true
			fromMouse()
		end
	end)
	hue.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragHue = true
			fromMouse()
		end
	end)
	bind(UserInputService.InputChanged, function(input)
		if (dragSV or dragHue) and input.UserInputType == Enum.UserInputType.MouseMovement then fromMouse() end
	end)
	bind(UserInputService.InputEnded, function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then dragSV, dragHue = false, false end
	end)

	swatch.MouseButton1Click:Connect(function() panel.Visible = not panel.Visible end)
	update(false)
end

--==================== Notifications & watermark ====================
local notifyList = create("Frame", {
	AnchorPoint = Vector2.new(0, 1), Position = UDim2.new(0, 12, 1, -12),
	Size = UDim2.fromOffset(400, 300), BackgroundTransparency = 1, Parent = gui,
}, {
	create("UIListLayout", { Padding = UDim.new(0, 4), VerticalAlignment = Enum.VerticalAlignment.Bottom, SortOrder = Enum.SortOrder.LayoutOrder }),
})

local function notify(msg, duration)
	duration = duration or 3
	local outer, fill = box("Frame", { Size = UDim2.fromOffset(textWidth(msg) + 18, 22), LayoutOrder = nextOrder(), Parent = notifyList }, Theme.Light)
	create("Frame", { Size = UDim2.new(0, 2, 1, 0), BorderSizePixel = 0, BackgroundColor3 = Theme.Accent, Parent = fill })
	label({ Parent = fill, Position = UDim2.fromOffset(8, 0), Size = UDim2.new(1, -10, 1, 0), Text = msg })
	local progress = create("Frame", {
		Position = UDim2.new(0, 0, 1, -1), Size = UDim2.new(1, 0, 0, 1),
		BorderSizePixel = 0, BackgroundColor3 = Theme.Accent, Parent = fill,
	})
	TweenService:Create(progress, TweenInfo.new(duration, Enum.EasingStyle.Linear), { Size = UDim2.new(0, 0, 0, 1) }):Play()
	task.delay(duration, function() outer:Destroy() end)
end

local wmOuter, wmFill = box("Frame", {
	AnchorPoint = Vector2.new(1, 0), Position = UDim2.new(1, -12, 0, 64),
	Size = UDim2.fromOffset(200, 22), Parent = gui,
}, Theme.Light)
local wmLine = create("Frame", { Size = UDim2.new(1, 0, 0, 1), BorderSizePixel = 0, Parent = wmFill })
onAccent(function(c) wmLine.BackgroundColor3 = c end)
local wmText = label({ Parent = wmFill, Position = UDim2.fromOffset(6, 1), Size = UDim2.new(1, -12, 1, -1) })

local function updateHint()
	keyHint.Text = "menu: [" .. keyName(Settings.MenuKey) .. "]"
end
updateHint()

--==================== Character ====================
local cachedRoot = nil

local function rootFromModel(model)
	local mesh = model and model:FindFirstChild("MeshModel")
	local rp = mesh and mesh:FindFirstChild("RootPart")
	if rp and rp:IsA("BasePart") then return rp end
	return nil
end

local function getRoot()
	local characters = workspace:FindFirstChild("Characters")

	-- 1) by name
	if CHARACTER_NAME ~= "" and characters then
		local rp = rootFromModel(characters:FindFirstChild(CHARACTER_NAME))
		if rp then return rp end
	end

	-- 2) default Roblox character
	local char = player.Character
	local hrp = char and char:FindFirstChild("HumanoidRootPart")
	if hrp then return hrp end

	-- 3) camera subject
	local cam = workspace.CurrentCamera
	local subject = cam and cam.CameraSubject
	if subject then
		if subject:IsA("BasePart") then return subject end
		if subject:IsA("Humanoid") and subject.RootPart then return subject.RootPart end
		if subject:IsA("Model") then
			local rp = rootFromModel(subject) or subject.PrimaryPart
			if rp then return rp end
		end
	end

	-- 4) cached / closest RootPart in workspace.Characters to the camera
	if cachedRoot and cachedRoot:IsDescendantOf(workspace) then return cachedRoot end
	if characters and cam then
		local best, bestDist = nil, math.huge
		for _, model in ipairs(characters:GetChildren()) do
			local rp = rootFromModel(model)
			if rp then
				local dist = (rp.Position - cam.CFrame.Position).Magnitude
				if dist < bestDist then best, bestDist = rp, dist end
			end
		end
		cachedRoot = best
		return best
	end
	return nil
end

--==================== ESP ====================
local tracked = {} -- [Model/Part] = { hl, bb, lbl, part, conn, dist }

local function hasTargetAncestor(inst)
	local p = inst.Parent
	while p and p ~= folder do
		if string.lower(p.Name) == TARGET then return true end
		p = p.Parent
	end
	return false
end

local function isTarget(inst)
	return (inst:IsA("Model") or inst:IsA("BasePart"))
		and string.lower(inst.Name) == TARGET
		and not hasTargetAncestor(inst)
end

local function getPart(inst)
	if inst:IsA("BasePart") then return inst end
	return inst.PrimaryPart or inst:FindFirstChildWhichIsA("BasePart", true)
end

local function removeESP(inst)
	local d = tracked[inst]
	if not d then return end
	tracked[inst] = nil
	d.conn:Disconnect()
	d.hl:Destroy()
	d.bb:Destroy()
end

local function addESP(inst)
	if tracked[inst] or not inst.Parent or not inst:IsDescendantOf(folder) or not isTarget(inst) then return end

	local hl = create("Highlight", {
		Name = "PumpkinESP", Adornee = inst, Enabled = false,
		DepthMode = Enum.HighlightDepthMode.AlwaysOnTop, Parent = inst,
	})
	local bb = create("BillboardGui", {
		Name = "PumpkinESPLabel", Size = UDim2.fromOffset(200, 20), StudsOffset = Vector3.new(0, 3, 0),
		AlwaysOnTop = true, Enabled = false, Parent = inst,
	})
	local lbl = create("TextLabel", {
		BackgroundTransparency = 1, Size = UDim2.fromScale(1, 1), Font = Theme.Font,
		TextSize = Settings.TextSize, TextStrokeTransparency = 0, Text = "Pumpkin", Parent = bb,
	})
	local conn = inst.AncestryChanged:Connect(function()
		if not inst:IsDescendantOf(folder) then removeESP(inst) end
	end)
	tracked[inst] = { hl = hl, bb = bb, lbl = lbl, conn = conn, dist = 0 }
end

local function scan()
	for _, inst in ipairs(folder:GetDescendants()) do addESP(inst) end
end

local function countTracked()
	local n = 0
	for _ in pairs(tracked) do n += 1 end
	return n
end

scan()
bind(folder.DescendantAdded, function(inst) task.defer(addESP, inst) end)
bind(folder.DescendantRemoving, removeESP)

--==================== Pathfind (visual guide) ====================
local pathFolder = create("Folder", { Name = "PumpkinPath", Parent = workspace.CurrentCamera })
local dots, segs = {}, {}
local pathStatus = "Idle"
local JUMP_COLOR = Color3.fromRGB(255, 255, 80)
local FAIL_COLOR = Color3.fromRGB(255, 60, 60)

local function newVisualPart(isBall)
	local p = Instance.new("Part")
	p.Anchored = true
	p.CanCollide = false
	p.CanQuery = false
	p.CanTouch = false
	p.CastShadow = false
	p.Material = Enum.Material.Neon
	p.Transparency = 1
	if isBall then
		p.Shape = Enum.PartType.Ball
		p.Size = Vector3.new(0.6, 0.6, 0.6)
	else
		p.Size = Vector3.new(0.15, 0.15, 1)
	end
	p.Parent = pathFolder
	return p
end

local function drawPath(points, jumps, color)
	for i = 1, #points do
		local dot = dots[i] or newVisualPart(true)
		dots[i] = dot
		dot.Position = points[i]
		dot.Color = (Settings.ShowJumps and jumps[i]) and JUMP_COLOR or color
		dot.Transparency = 0
	end
	for i = #points + 1, #dots do dots[i].Transparency = 1 end

	for i = 1, #points - 1 do
		local seg = segs[i] or newVisualPart(false)
		segs[i] = seg
		local a, b = points[i], points[i + 1]
		local len = (b - a).Magnitude
		if len > 0.05 then
			seg.Size = Vector3.new(0.15, 0.15, len)
			seg.CFrame = CFrame.lookAt((a + b) / 2, b)
			seg.Color = color
			seg.Transparency = 0.2
		else
			seg.Transparency = 1
		end
	end
	for i = math.max(#points, 1), #segs do segs[i].Transparency = 1 end
end

local function clearPath()
	drawPath({}, {}, Settings.PathColor)
end

local function nearestPumpkin(origin)
	local best, bestDist = nil, math.huge
	for _, d in pairs(tracked) do
		if d.part and d.part.Parent then
			local dist = (d.part.Position - origin).Magnitude
			if dist < bestDist and dist <= Settings.MaxDistance then
				best, bestDist = d, dist
			end
		end
	end
	return best, bestDist
end

local function computePath()
	local root = getRoot()
	if not root then
		pathStatus = "Character not found"
		clearPath()
		return
	end
	local target, dist = nearestPumpkin(root.Position)
	if not target then
		pathStatus = "No pumpkins in range"
		clearPath()
		return
	end

	local path = PathfindingService:CreatePath({
		AgentRadius = Settings.AgentRadius,
		AgentHeight = 5,
		AgentCanJump = true,
		WaypointSpacing = 4,
	})
	local ok = pcall(function()
		path:ComputeAsync(root.Position, target.part.Position)
	end)
	if unloaded or not Settings.ShowPath then return end

	if ok and path.Status == Enum.PathStatus.Success then
		local points, jumps = {}, {}
		for i, wp in ipairs(path:GetWaypoints()) do
			points[i] = wp.Position + Vector3.new(0, 0.5, 0)
			jumps[i] = wp.Action == Enum.PathWaypointAction.Jump
		end
		drawPath(points, jumps, Settings.PathColor)
		pathStatus = string.format("Path: %d pts | %dm", #points, math.floor(dist))
	else
		drawPath({ root.Position, target.part.Position }, {}, FAIL_COLOR)
		pathStatus = string.format("No path | direct %dm", math.floor(dist))
	end
end

--==================== Menu ====================
local espTab = Tab("ESP")
local settingsTab = Tab("Settings")

-- ESP tab: left column
local mainSec = Section(espTab.Left, "Main")
local espToggle = Toggle(mainSec, "Enabled", Settings.Enabled, function(v) Settings.Enabled = v end)
Toggle(mainSec, "Fill", Settings.Fill, function(v) Settings.Fill = v end)
Toggle(mainSec, "Outline", Settings.Outline, function(v) Settings.Outline = v end)
Toggle(mainSec, "Labels", Settings.Labels, function(v) Settings.Labels = v end)
Toggle(mainSec, "Show Distance", Settings.Distance, function(v) Settings.Distance = v end)

local hlSec = Section(espTab.Left, "Highlight")
ColorPicker(hlSec, "Fill Color", Settings.FillColor, function(c) Settings.FillColor = c end)
ColorPicker(hlSec, "Outline Color", Settings.OutlineColor, function(c) Settings.OutlineColor = c end)
Slider(hlSec, "Fill Transparency", 0, 1, Settings.FillTransparency, 0.05, "", function(v) Settings.FillTransparency = v end)
Slider(hlSec, "Outline Transparency", 0, 1, Settings.OutlineTransparency, 0.05, "", function(v) Settings.OutlineTransparency = v end)

-- ESP tab: right column
local pathSec = Section(espTab.Right, "Pathfind")
Toggle(pathSec, "Show Path", Settings.ShowPath, function(v)
	Settings.ShowPath = v
	if not v then clearPath() end
end)
Toggle(pathSec, "Jump Points", Settings.ShowJumps, function(v) Settings.ShowJumps = v end)
ColorPicker(pathSec, "Path Color", Settings.PathColor, function(c) Settings.PathColor = c end)
Slider(pathSec, "Update Rate", 0.2, 2, Settings.PathRate, 0.1, "s", function(v) Settings.PathRate = v end)
Slider(pathSec, "Agent Radius", 1, 6, Settings.AgentRadius, 0.5, "", function(v) Settings.AgentRadius = v end)
local pathStatusLabel = Label(pathSec, "Idle", true)

local filterSec = Section(espTab.Right, "Filter")
Slider(filterSec, "Max Distance", 50, 5000, Settings.MaxDistance, 50, "m", function(v) Settings.MaxDistance = v end)
Slider(filterSec, "Max Highlights", 1, 31, Settings.MaxHighlights, 1, "", function(v) Settings.MaxHighlights = v end)
Label(filterSec, "Roblox limit: 31 highlights", true)

local lblSec = Section(espTab.Right, "Labels")
ColorPicker(lblSec, "Text Color", Settings.LabelColor, function(c) Settings.LabelColor = c end)
Slider(lblSec, "Text Size", 10, 24, Settings.TextSize, 1, "px", function(v) Settings.TextSize = v end)

-- Settings tab
local menuSec = Section(settingsTab.Left, "Menu")
Keybind(menuSec, "Menu Key", Settings.MenuKey, false, function(k)
	Settings.MenuKey = k
	updateHint()
end)
Keybind(menuSec, "ESP Key", Settings.EspKey, true, function(k) Settings.EspKey = k end)
Toggle(menuSec, "Watermark", Settings.Watermark, function(v)
	Settings.Watermark = v
	wmOuter.Visible = v
end)
ColorPicker(menuSec, "Accent", Theme.Accent, setAccent)

local infoSec = Section(settingsTab.Right, "Info")
local foundLabel = Label(infoSec, "Found: 0")
local shownLabel = Label(infoSec, "Highlighted: 0")
local charLabel = Label(infoSec, "Character: -")
Button(infoSec, "Rescan", function()
	for inst in pairs(tracked) do removeESP(inst) end
	cachedRoot = nil
	scan()
	notify("Rescanned: " .. countTracked() .. " pumpkins found")
end)
Button(infoSec, "Unload", function()
	unloaded = true
	for _, c in ipairs(connections) do c:Disconnect() end
	for inst in pairs(tracked) do removeESP(inst) end
	pathFolder:Destroy()
	gui:Destroy()
end)

--==================== Input ====================
bind(UserInputService.InputBegan, function(input, processed)
	if listening then
		if input.UserInputType == Enum.UserInputType.Keyboard then
			local kb = listening
			listening = nil
			if input.KeyCode == Enum.KeyCode.Escape or input.KeyCode == Enum.KeyCode.Backspace then
				if kb.allowNone then kb.Set(nil) else kb.render() end
			else
				kb.Set(input.KeyCode)
			end
		end
		return
	end
	if processed then return end

	if input.KeyCode == Settings.MenuKey then
		main.Visible = not main.Visible
	elseif Settings.EspKey and input.KeyCode == Settings.EspKey then
		espToggle.Set(not espToggle.Get())
		notify("ESP " .. (espToggle.Get() and "enabled" or "disabled"), 2)
	end
end)

--==================== Pathfind loop ====================
task.spawn(function()
	while not unloaded do
		if Settings.ShowPath and Settings.Enabled then
			computePath()
		else
			clearPath()
			pathStatus = Settings.Enabled and "Path disabled" or "ESP disabled"
		end
		if unloaded then break end
		pathStatusLabel.Text = pathStatus
		task.wait(Settings.PathRate)
	end
end)

--==================== Update loop ====================
local fpsFrames, fpsTimer, espTimer = 0, 0, 0
local foundCount, shownCount = 0, 0

bind(RunService.Heartbeat, function(dt)
	-- watermark
	fpsFrames += 1
	fpsTimer += dt
	if fpsTimer >= 0.5 then
		local fps = math.floor(fpsFrames / fpsTimer + 0.5)
		fpsFrames, fpsTimer = 0, 0
		if Settings.Watermark then
			local t = string.format("Pumpkin ESP | %d fps | %d pumpkins", fps, foundCount)
			wmText.Text = t
			wmOuter.Size = UDim2.fromOffset(textWidth(t) + 16, 22)
		end
	end

	-- esp
	espTimer += dt
	if espTimer < 0.15 then return end
	espTimer = 0

	local root = getRoot()
	charLabel.Text = "Character: " .. (root and root.Parent and root.Parent.Parent and root.Parent.Parent.Name or "not found")
	local origin = root and root.Position or workspace.CurrentCamera.CFrame.Position

	local visible = {}
	foundCount = 0
	for inst, d in pairs(tracked) do
		foundCount += 1
		if not (d.part and (d.part == inst or d.part:IsDescendantOf(inst))) then
			d.part = getPart(inst)
			d.bb.Adornee = d.part
		end
		d.show = false
		d.inRange = false
		if Settings.Enabled and d.part then
			d.dist = (d.part.Position - origin).Magnitude
			if d.dist <= Settings.MaxDistance then
				d.inRange = true
				table.insert(visible, d)
			end
		end
	end

	table.sort(visible, function(a, b) return a.dist < b.dist end)
	for i = 1, math.min(#visible, Settings.MaxHighlights) do
		visible[i].show = true
	end

	shownCount = 0
	for _, d in pairs(tracked) do
		local on = d.show and (Settings.Fill or Settings.Outline)
		d.hl.Enabled = on
		if on then
			shownCount += 1
			d.hl.FillColor = Settings.FillColor
			d.hl.OutlineColor = Settings.OutlineColor
			d.hl.FillTransparency = Settings.Fill and Settings.FillTransparency or 1
			d.hl.OutlineTransparency = Settings.Outline and Settings.OutlineTransparency or 1
		end

		local labelOn = d.inRange and Settings.Labels
		d.bb.Enabled = labelOn
		if labelOn then
			d.bb.Size = UDim2.fromOffset(200, Settings.TextSize + 6)
			d.lbl.TextSize = Settings.TextSize
			d.lbl.TextColor3 = Settings.LabelColor
			d.lbl.Text = Settings.Distance and string.format("Pumpkin [%dm]", math.floor(d.dist)) or "Pumpkin"
		end
	end

	foundLabel.Text = "Found: " .. foundCount
	shownLabel.Text = "Highlighted: " .. shownCount
end)

notify("Pumpkin ESP loaded | press [" .. keyName(Settings.MenuKey) .. "] to toggle menu", 4)
