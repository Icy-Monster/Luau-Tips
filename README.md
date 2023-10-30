
## Readability and Reliability for Luau

# Table of Contents
### [Introduction](#10-Introduction) 1.0  

### [Deprecations and Misconceptions](#20-Deprecations-and-Misconceptions) 2.0  
[The Task Library](#21-The-Task-Library) **----------------------------------------**  2.1  
[Parts and MeshParts (BaseParts)](#22-parts-and-meshparts-baseparts) **----------------------** 2.2  
[The Invisible Variables](#23-The-Invisible-Variables) **---------------------------------** 2.3  
[The more lines the better... right?](#24-the-more-lines-the-better-right) **---------------------** 2.4  

### [The Art Side of Coding (Syntax)](#30-the-art-side-of-coding-syntax) 3.0  
[Operator Syntax](#31-Operator-Syntax)
**---------------------------------------** 3.1  
[Variables](#32-Variables)
**-----------------------------------------------** 3.2  
[Strongly Strict Typing](#33-Strongly-Strict-Typing)
**----------------------------------** 3.3  
[How_To WriteVariable NAMES](#34-How_To-WriteVariable-NAMES)
**------------------------** 3.4  
[Paragraphs](#35-Paragraphs)
**--------------------------------------------** 3.5  
[Nesting](#36-Nesting)
**-----------------------------------------------** 3.6  

# 1.0 Introduction
This is a simple guide to writing reliable and quickly readable code that is universaly known to all programmers. It includes many aspects of coding and many practices of formatting and writing code.

This guide requires basic knowledge of Lua. The guide does not explain programming in Lua, but common bugs and problems with readability.

# 2.0 Deprecations and Misconceptions
## 2.1 The Task Library
https://create.roblox.com/docs/reference/engine/libraries/task
Roblox introduced the Task library in august of 2021, with the Task library being used to directly talk to the game engines task schedule of tasks to be ran.

It brang and deprecated many functions such as:

```lua
wait(x: Number): Number -> task.wait(x: Number): Number

spawn(function) -> task.defer(function)
```
While function of wait() is fundamentally the same it still differs from task.wait().
The difference is in the precision of the yield, coming from 1/30 of a second to 1/60 of a second with the Task Library

## 2.2 Parts and MeshParts (BaseParts)
While Parts and MeshParts may look like completely different instances, they are fundamentally the same with just the MeshPart having a different, editable topology

To create a BasePart you can use 
```lua
Instance.new(ClassName, Parent)
```
While this may look like the best way to create an instance it can sometimes be less efficient if you do not know what you are doing. The primary optimization here will be the 2nd arguement when using Instance.new().

```lua
local Part = Instance.new("Part", workspace)
-- Once the parent is set the Physics engine will check if its anchored.
-- Since it isnt it will start tracking it
Part.Anchored = true
-- The part is anchored so the Physics engine will stop tracking it.
-- This will create an overhead
```

### ->

```lua
local Part = Instance.new("Part")
Part.Anchored = true

Part.Parent = workspace
-- Once the parent is set the Physics engine will check if its anchored and since it isn't it will not start tracking it.
-- This will not create any overhead
```
When you set the parent with the 2nd arguement of Instance.new it sets its parent immediately after the instance is made, meaning that it will also render and fire off RBXLSignalEvents for when the part is created. If you edit properties after setting the parent of an instance it can cause performance drops since you are editing the properties while it has running events and is being rendered.

The proper sequence that you should set up properties/connections is:
```lua
-- Instance.new
local Part = Instance.new("Part")

-- Assign Properties
Part.Anchored = true

-- Assign Parent
Part.Parent = workspace

-- Connect Signals
Part.Touched:Connect(function(TouchedPart: BasePart)
    print(TouchedPart.Name)
end)
```

## 2.3 The Invisible Variables
Sometimes you may just not see a built-in function in Roblox and then you try making it yourself. All Roblox API (Application Programming Interface) is written in C++ so it is marginaly faster than recreating it yourself in Luau.

You might just overlook some clear features that the API offers to you such as:

```lua
local Index = 0

for i, v in {} do
    Index += 1
    print(Index)
end
```

### ->

```lua
for i, v in {} do
    print(i)
end
```

While you are looking at what variables you should use, you should also look at what variables to hide. You should also not be scared of naming variables to something that describes them instead of using single letters like i or v

```lua
for i, v in {} do
    Part.Position += Vector3.new(1,0,0)
end
```

### ->
```lua
for _, Part: BasePart in {} do
    Part.Position += Vector3.new(1,0,0)
end
```


## 2.4 The more lines the better... right?
No. Just no, the exact opposite applies. The less lines there are the more readable it is (Do not take this to the extreme). A lot of the times when we are making functions we think about the task logically in our heads, so we take it one step at a time but sometimes it may be better to just take all steps at the same time. Here are some common examples on where you can improve:

```lua
    local function PointInRectangle(Point: Vector2, Rectangle: Vector2, RectangleSize: Vector2): boolean
	if point.x >= rectPos.x and point.x <= (rectPos.x + rectSize.x) then
		if point.y >= rectPos.y and point.y <= (rectPos.y + rectSize.y)  then
			return true
		end
	end
	
	return false
end
```

### ->

```lua
local function PointInRectangle(Point: Vector2, Rectangle: Vector2, RectangleSize: Vector2): boolean
	return point.x >= rectPos.x and point.x <= (rectPos.x + rectSize.x) and 
		point.y >= rectPos.y and point.y <= (rectPos.y + rectSize.y)
end

```
Look at how cleaner that is, and more readable. 


# 3.0 The Art Side of Coding (Syntax)
https://roblox.github.io/lua-style-guide/

While one may think that code is purely written to be as most efficient as possible, it may be better to value readability over performance in some cases
** **
## 3.1 Operator Syntax
All operators have their own syntax for shortening and simplifying your code to make it more readable such as:
```lua
Math Operators
X = X + 1  -> X += 1
X = X - 1  -> X -= 1
X = X * X  -> X *= X
X = X / X  -> X /= X

String Operators
X = X.."JoinStrings" -> X ..= "JoinStrings"

Boolean Operators:
if X == true then -> if X then
if X == false then -> if not X then

if not X == 1 then -> if X ~= 1 then
```
This helps to shorten the code while increasing its readability, there is no performance change between the two ways of writing the syntax. Since it is compiled into the same bytecode

## 3.2 Variables
There are 2 types of variables: **Local Variables** and **Global Variables**,
the difference between these 2 types is the scope of their declaration. Global variables have their scopes outside all functions while Local variables are scoped inside a function Block

Between the 2, Local variables should be used the most. Due to their performance benefits when compared to Global variables and their memory costs

Variables can have many values inside of them, some can have predetermined values such as when you are using **strict typing**

## 3.3 Strongly Strict Typing
Strict typing or otherwise known as strong typing is a method of programming with strictly defined types for each value.

To use strict typing in Luau you have to declare it at the **top** of your script:
```lua
-- !strict
Rest of the code
```

The syntax for defining a variables type is:
```lua
local MyNumber: Number = 0
```

The syntax for defining types in a function is:
```lua
local function RandomDecimal(Min: Number, Max: Number): Number
    return math.random(Min * 100, Max * 100)/100
end
```
## 3.4 How_To WriteVariable NAMES
There are many ways of writing variable names such as: PascalCase, camelCase, snake_case and many more variations

All of them are completely fine to use but should be used consistently with your coding accent

You can use Robloxs lua styling guide:
* PascalCase for class and enum-like objects
* PascalCase for Roblox API (camelCase versions are deprecated)
* camelCase for Local variables
* Spell out words fully, abbreviations may save time but may also cause confusion to the reader
- SCREAMING_SNAKE_CASE for Constant (Non-Changing) variables
- Acronyms that  represent a set (XYZ, RGB, ...) should be fully capitalized, but acronyms that do not respesent a set should only have their starting letter be capitalized (Http, Json, ...)

## 3.5 Paragraphs
While programming languages aren't structured as normal writings, they still contain aspects of them. When scripting you would want to increase the readability by seperating your code into chunks. These chunks would each share a simiral task such as:

```lua
local Part = Instance.new("Part")
Part.Anchored = true
Part.CanCollide = false
Part.Size = Vector3.new(10,10,10)
Part.Position = Vector3.new(0,10,0)
Part.Color = Color3.fromRGB(255,255,0)
Part.Parent = workspace
Part.Touched:Connect(function(TouchedPart: BasePart)
	print(TouchedPart.Name)
end)
```

### ->

```lua
local Part = Instance.new("Part")
Part.Anchored = true
Part.CanCollide = false

Part.Size = Vector3.new(10,10,10)
Part.Position = Vector3.new(0,10,0)
Part.Color = Color3.fromRGB(255,255,0)

Part.Parent = workspace


Part.Touched:Connect(function(TouchedPart: BasePart)
	print(TouchedPart.Name)
end)
```

You can see how much the readability increases, just by seperating it into small chunks that share a common task. While there are no clear rules of how you should seperate your code, you should always focus on each group doing a simiral task.

## 3.6 Nesting
Closely related to paragraphs, nesting is how many "levels" that your code has.
If not quickly handled nesting can become a nightmare to look at.

Here's a small example that displays nesting and how to fix it:

```lua
Part.Touched:Connect(function(TouchedPart: BasePart)
	if TouchedPart.Parent:FindFirstChild("Humanoid") then
		if not TouchedPart.Parent:FindFirstChild("Shield") then
			TouchedPart.Parent:Destroy()
		end
	end
end)
```

### ->

```lua
Part.Touched:Connect(function(TouchedPart: BasePart)
	if not TouchedPart.Parent:FindFirstChild("Humanoid") then return end
	if TouchedPart.Parent:FindFirstChild("Shield") then return end

	TouchedPart.Parent:Destroy()
end)
```

Nesting should always be avoided, since it makes the code harder to read.