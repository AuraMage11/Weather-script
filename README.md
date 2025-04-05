-- DayNightCycleAndWeather.lua
-- This script creates a dynamic day-night cycle and weather system.
-- It updates Lighting properties to simulate time passing and
-- randomly triggers storms with rain and thunder effects.

-- SERVICES
local Lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")
local Debris = game:GetService("Debris")
local TweenService = game:GetService("TweenService")

-- SETTINGS
local DAY_LENGTH = 600        -- Duration of a day in seconds (example: 10 minutes)
local NIGHT_LENGTH = 300      -- Duration of a night in seconds (example: 5 minutes)
local TIME_STEP = 1           -- Time step update interval (in seconds)

local WEATHER_CHECK_INTERVAL = 60   -- Check weather conditions every 60 seconds
local STORM_PROBABILITY = 0.2         -- 20% chance of storm during the day
local THUNDER_INTERVAL_MIN = 5        -- Minimum seconds between thunder sounds
local THUNDER_INTERVAL_MAX = 15       -- Maximum seconds between thunder sounds

-- VARIABLES
local isDay = true       -- Start with day
local isStorm = false    -- Storm flag

-----------------------------------------------------------
-- FUNCTION: updateLighting
-- Updates Lighting properties based on timeOfDay (0-24 hours)
-----------------------------------------------------------
local function updateLighting(timeOfDay)
    local brightness = 0
    local ambient = Color3.new(0,0,0)
    local outdoorAmbient = Color3.new(0,0,0)
    
    if timeOfDay < 6 then
        -- Night early hours
        brightness = 1
        ambient = Color3.fromRGB(20, 20, 40)
        outdoorAmbient = Color3.fromRGB(20, 20, 40)
    elseif timeOfDay < 8 then
        -- Dawn transition
        brightness = math.clamp((timeOfDay - 6) / 2 * 4, 1, 4)
        ambient = Color3.fromRGB(40, 40, 60)
        outdoorAmbient = Color3.fromRGB(40, 40, 60)
    elseif timeOfDay < 18 then
        -- Day time
        brightness = 4
        ambient = Color3.fromRGB(100, 100, 120)
        outdoorAmbient = Color3.fromRGB(120, 120, 140)
    elseif timeOfDay < 20 then
        -- Dusk transition
        brightness = math.clamp(4 - ((timeOfDay - 18) / 2 * 3), 1, 4)
        ambient = Color3.fromRGB(80, 60, 80)
        outdoorAmbient = Color3.fromRGB(80, 60, 80)
    else
        -- Night time
        brightness = 1
        ambient = Color3.fromRGB(20, 20, 40)
        outdoorAmbient = Color3.fromRGB(20, 20, 40)
    end

    Lighting.Brightness = brightness
    Lighting.Ambient = ambient
    Lighting.OutdoorAmbient = outdoorAmbient
    -- Set the in-game clock (format HH:MM:SS)
    Lighting.TimeOfDay = string.format("%02d:%02d:00", math.floor(timeOfDay), math.floor((timeOfDay % 1) * 60))
end

-----------------------------------------------------------
-- FUNCTION: simulateDayNightCycle
-- Gradually simulates a day-night cycle.
-----------------------------------------------------------
local function simulateDayNightCycle()
    while true do
        local cycleDuration = isDay and DAY_LENGTH or NIGHT_LENGTH
        local timeIncrement = 24 / cycleDuration   -- How many hours per second?
        
        for elapsed = 0, cycleDuration, TIME_STEP do
            local newTime = isDay and (elapsed * timeIncrement) or (18 + (elapsed * timeIncrement))
            newTime = newTime % 24  -- Wrap-around at 24 hours
            updateLighting(newTime)
            wait(TIME_STEP)
        end

        isDay = not isDay
    end
end

-----------------------------------------------------------
-- FUNCTION: createRainEffect
-- Creates a rain effect using a ParticleEmitter.
-----------------------------------------------------------
local function createRainEffect(duration)
    -- Create a new Part for the rain emitter (invisible container)
    local rainPart = Instance.new("Part")
    rainPart.Name = "RainEffectPart"
    rainPart.Transparency = 1
    rainPart.Anchored = true
    rainPart.CanCollide = false
    rainPart.Size = Vector3.new(1,1,1)
    rainPart.Position = Vector3.new(0, 100, 0)  -- Position high above the map
    rainPart.Parent = workspace

    local rainEmitter = Instance.new("ParticleEmitter")
    rainEmitter.Texture = "rbxassetid://131070188"  -- Example rain texture
    rainEmitter.Rate = 300
    rainEmitter.Speed = NumberRange.new(20, 25)
    rainEmitter.Lifetime = NumberRange.new(1, 1.5)
    rainEmitter.VelocitySpread = 10
    rainEmitter.Parent = rainPart

    -- Clean up after duration
    delay(duration, function()
        rainEmitter.Enabled = false
        wait(3)
        rainPart:Destroy()
    end)
end

-----------------------------------------------------------
-- FUNCTION: playThunderSound
-- Plays a thunder sound effect.
-----------------------------------------------------------
local function playThunderSound()
    local thunderSound = Instance.new("Sound")
    thunderSound.SoundId = "rbxassetid://301964312"  -- Example thunder sound asset
    thunderSound.Volume = 1
    thunderSound.Parent = Lighting  -- Attach to Lighting so it persists
    thunderSound:Play()
    Debris:AddItem(thunderSound, 10)
end

-----------------------------------------------------------
-- FUNCTION: triggerStorm
-- Triggers a storm with rain and periodic thunder sounds.
-----------------------------------------------------------
local function triggerStorm()
    isStorm = true
    print("Storm started!")
    createRainEffect(120)  -- Rain lasts for 2 minutes

    local stormEndTime = tick() + 120
    while tick() < stormEndTime do
        local waitTime = math.random(THUNDER_INTERVAL_MIN, THUNDER_INTERVAL_MAX)
        wait(waitTime)
        playThunderSound()
        print("Thunder sound played!")
    end

    isStorm = false
    print("Storm ended!")
end

-----------------------------------------------------------
-- FUNCTION: weatherManager
-- Periodically checks and triggers weather events.
-----------------------------------------------------------
local function weatherManager()
    while true do
        wait(WEATHER_CHECK_INTERVAL)
        if isDay and math.random() < STORM_PROBABILITY and not isStorm then
            triggerStorm()
        end
    end
end

-----------------------------------------------------------
-- MAIN
-----------------------------------------------------------
spawn(simulateDayNightCycle)  -- Start the day-night cycle
spawn(weatherManager)         -- Start the weather manager

print("Day/Night and Weather system initialized.")
