--[[
    =====================================================
    FULL ATC SYSTEM — Roblox Flight Simulator
    =====================================================
    Place this Script inside ServerScriptService.
    
    REQUIREMENTS:
    - All aircraft models must be tagged "Aircraft"
    - Each aircraft model needs a Seat named "PilotSeat"
    - Create a Folder in Workspace named "Runways"
    - Each runway is a Part inside that folder
      with attributes: RunwayName (string), Heading (number)
    - (Optional) A RemoteEvent in ReplicatedStorage named
      "ATCMessage" for GUI-based ATC messages
    =====================================================
--]]

local Players         = game:GetService("Players")
local RunService      = game:GetService("RunService")
local CollectionService = game:GetService("CollectionService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- =====================================================
-- CONFIGURATION
-- =====================================================
local CONFIG = {
    AIRCRAFT_TAG        = "Aircraft",       -- CollectionService tag on all aircraft models
    PILOT_SEAT_NAME     = "PilotSeat",      -- Name of the VehicleSeat / Seat inside aircraft
    RUNWAY_FOLDER       = "Runways",        -- Folder in Workspace containing runway Parts
    SCAN_INTERVAL       = 2,               -- Seconds between each system loop tick
    APPROACH_ALTITUDE   = 500,             -- Studs — below this = approaching
    LANDING_ALTITUDE    = 20,              -- Studs — below this = landed
    LANDING_SPEED       = 30,             -- Studs/s — below this (with low alt) = landed
    APPROACH_RADIUS     = 2000,            -- Studs — within this of airport centre = approaching
    MESSAGE_COOLDOWN    = 8,              -- Seconds between ATC messages per flight
    AIRPORT_POSITION    = Vector3.new(0, 0, 0), -- Set to your airport's centre position
    USE_CHAT            = true,            -- Send ATC via in-game chat
    USE_REMOTE          = true,            -- Fire ATCMessage RemoteEvent for GUI panels
    
    -- Callsign generation
    AIRLINES = {"BA", "EZY", "RYR", "UAE", "SIA", "DLH", "AFR", "AAL", "UAL", "SWA"},
}

-- =====================================================
-- STATE
-- =====================================================
local flights   = {}       -- [aircraft] = flightData
local runways   = {}       -- array of runway info tables
local queue     = {}       -- array of aircraft waiting for a runway
local atcRemote = nil      -- optional RemoteEvent

-- Flight status constants
local STATUS = {
    APPROACH  = "Approach",
    LANDING   = "Landing",
    LANDED    = "Landed",
    TAXI      = "Taxi",
    PARKED    = "Parked",
    DEPARTING = "Departing",
    TAKEOFF   = "Takeoff",
    AIRBORNE  = "Airborne",
}

-- =====================================================
-- UTILITY FUNCTIONS
-- =====================================================

local function generateCallsign()
    local airline = CONFIG.AIRLINES[math.random(#CONFIG.AIRLINES)]
    local number  = math.random(100, 999)
    return airline .. tostring(number)
end

local function getAircraftPilot(aircraft)
    local seat = aircraft:FindFirstChild(CONFIG.PILOT_SEAT_NAME, true)
    if not seat then return nil end
    local occupant = seat.Occupant
    if not occupant then return nil end
    return Players:GetPlayerFromCharacter(occupant.Parent)
end

local function getAircraftAltitude(aircraft)
    local root = aircraft:FindFirstChild("PrimaryPart") or aircraft:FindFirstChildWhichIsA("BasePart")
    if not root then return 0 end
    -- Altitude above ground (raycast downward)
    local origin    = root.Position
    local direction = Vector3.new(0, -10000, 0)
    local rayParams = RaycastParams.new()
    rayParams.FilterDescendantsInstances = {aircraft}
    rayParams.FilterType = Enum.RaycastFilterType.Exclude
    local result = workspace:Raycast(origin, direction, rayParams)
    if result then
        return origin.Y - result.Position.Y
    end
    return origin.Y
end

local function getAircraftSpeed(aircraft)
    local root = aircraft:FindFirstChild("PrimaryPart") or aircraft:FindFirstChildWhichIsA("BasePart")
    if not root then return 0 end
    return root.AssemblyLinearVelocity.Magnitude
end

local function getAircraftPosition(aircraft)
    local root = aircraft:FindFirstChild("PrimaryPart") or aircraft:FindFirstChildWhichIsA("BasePart")
    if not root then return Vector3.new(0, 0, 0) end
    return root.Position
end

local function distanceToAirport(aircraft)
    return (getAircraftPosition(aircraft) - CONFIG.AIRPORT_POSITION).Magnitude
end

-- =====================================================
-- ATC MESSAGING
-- =====================================================

local function sendATCMessage(flight, message)
    local now = tick()
    if now - (flight.lastMessageTime or 0) < CONFIG.MESSAGE_COOLDOWN then return end
    flight.lastMessageTime = now

    local fullMessage = string.format("[ATC] %s, %s", flight.callsign, message)

    -- Chat method
    if CONFIG.USE_CHAT and flight.player then
        local success, err = pcall(function()
            -- Post to player's chat via a system message approach
            game:GetService("Chat"):Chat(
                flight.player.Character and flight.player.Character:FindFirstChild("Head") or workspace,
                fullMessage,
                Enum.ChatColor.Blue
            )
        end)
        if not success then
            -- Fallback: print to output (server-side logging)
            print(fullMessage)
        end
    end

    -- RemoteEvent method (fire to specific player for GUI panels)
    if CONFIG.USE_REMOTE and atcRemote and flight.player then
        pcall(function()
            atcRemote:FireClient(flight.player, flight.callsign, message)
        end)
    end

    print(fullMessage) -- Always log server-side
end

-- =====================================================
-- RUNWAY MANAGEMENT
-- =====================================================

local function loadRunways()
    runways = {}
    local folder = workspace:FindFirstChild(CONFIG.RUNWAY_FOLDER)
    if not folder then
        warn("[ATC] Runway folder '" .. CONFIG.RUNWAY_FOLDER .. "' not found in Workspace!")
        -- Create a dummy runway so the system still runs
        table.insert(runways, {
            name      = "27",
            part      = nil,
            position  = CONFIG.AIRPORT_POSITION,
            heading   = 270,
            occupied  = false,
            occupant  = nil,
        })
        return
    end

    for _, part in ipairs(folder:GetChildren()) do
        if part:IsA("BasePart") then
            local rwyName = part:GetAttribute("RunwayName") or part.Name
            local heading = part:GetAttribute("Heading") or 270
            table.insert(runways, {
                name     = tostring(rwyName),
                part     = part,
                position = part.Position,
                heading  = heading,
                occupied = false,
                occupant = nil,
            })
            print(string.format("[ATC] Loaded runway %s (hdg %d°)", rwyName, heading))
        end
    end

    if #runways == 0 then
        warn("[ATC] No runway Parts found in folder. Add BaseParts with RunwayName attribute.")
    end
end

local function getFreeRunway()
    for _, rwy in ipairs(runways) do
        if not rwy.occupied then
            return rwy
        end
    end
    return nil
end

local function assignRunway(flight)
    local rwy = getFreeRunway()
    if rwy then
        rwy.occupied = true
        rwy.occupant = flight
        flight.runway = rwy
        return rwy
    end
    -- All busy — add to queue if not already in it
    for _, f in ipairs(queue) do
        if f == flight then return nil end
    end
    table.insert(queue, flight)
    return nil
end

local function releaseRunway(flight)
    if flight.runway then
        flight.runway.occupied = false
        flight.runway.occupant = nil
        flight.runway = nil
        print(string.format("[ATC] Runway released by %s", flight.callsign))

        -- Process queue
        if #queue > 0 then
            local next = table.remove(queue, 1)
            local rwy  = assignRunway(next)
            if rwy then
                sendATCMessage(next, string.format(
                    "runway %s now available, cleared to land runway %s",
                    rwy.name, rwy.name
                ))
            end
        end
    end
end

-- =====================================================
-- FLIGHT LIFECYCLE
-- =====================================================

local function createFlight(aircraft)
    local player   = getAircraftPilot(aircraft)
    local callsign = generateCallsign()

    local flight = {
        aircraft       = aircraft,
        player         = player,
        callsign       = callsign,
        status         = STATUS.AIRBORNE,
        runway         = nil,
        lastMessageTime = 0,
        createdAt      = tick(),
    }

    flights[aircraft] = flight

    local pilotName = player and player.Name or "Unknown"
    print(string.format("[ATC] New flight: %s | Pilot: %s", callsign, pilotName))

    return flight
end

local function removeFlight(aircraft)
    local flight = flights[aircraft]
    if flight then
        releaseRunway(flight)
        -- Remove from queue
        for i, f in ipairs(queue) do
            if f == flight then
                table.remove(queue, i)
                break
            end
        end
        print(string.format("[ATC] Flight %s removed.", flight.callsign))
        flights[aircraft] = nil
    end
end

-- =====================================================
-- STATUS TRANSITIONS & ATC INSTRUCTIONS
-- =====================================================

local function handleApproach(flight)
    if flight.status == STATUS.AIRBORNE then
        flight.status = STATUS.APPROACH
        local rwy = assignRunway(flight)
        if rwy then
            sendATCMessage(flight, string.format(
                "identified on approach, runway %s in use, wind calm, cleared to land runway %s",
                rwy.name, rwy.name
            ))
        else
            sendATCMessage(flight,
                "all runways occupied, enter holding pattern, expect further clearance shortly"
            )
        end
    end
end

local function handleLanding(flight)
    if flight.status == STATUS.APPROACH or flight.status == STATUS.LANDING then
        flight.status = STATUS.LANDED
        local rwyName = flight.runway and flight.runway.name or "XX"
        sendATCMessage(flight, string.format(
            "landed runway %s, vacate when able and contact ground on taxiway Alpha",
            rwyName
        ))
    end
end

local function handleTaxi(flight)
    if flight.status == STATUS.LANDED then
        flight.status = STATUS.TAXI
        local rwyName = flight.runway and flight.runway.name or "XX"
        sendATCMessage(flight, string.format(
            "taxi to stand via Alpha, vacate runway %s to the right",
            rwyName
        ))
        -- Release runway once aircraft is taxiing clear
        task.delay(10, function()
            if flights[flight.aircraft] and flight.status == STATUS.TAXI then
                releaseRunway(flight)
            end
        end)
    end
end

local function handleParked(flight)
    if flight.status == STATUS.TAXI then
        flight.status = STATUS.PARKED
        sendATCMessage(flight, "stand confirmed, welcome, have a pleasant stay")
    end
end

local function handleDeparture(flight)
    -- Called when a pilot enters a plane that is on the ground
    if flight.status == STATUS.PARKED or flight.status == STATUS.TAXI then
        flight.status = STATUS.DEPARTING
        local rwy = assignRunway(flight)
        if rwy then
            sendATCMessage(flight, string.format(
                "pushback approved, taxi to holding point runway %s via Alpha",
                rwy.name
            ))
            -- Sequence takeoff clearance after a delay
            task.delay(15, function()
                if flights[flight.aircraft] and flight.status == STATUS.DEPARTING then
                    flight.status = STATUS.TAKEOFF
                    sendATCMessage(flight, string.format(
                        "line up and wait runway %s", rwy.name
                    ))
                    task.delay(5, function()
                        if flights[flight.aircraft] and flight.status == STATUS.TAKEOFF then
                            sendATCMessage(flight, string.format(
                                "wind calm, runway %s, cleared for takeoff", rwy.name
                            ))
                        end
                    end)
                end
            end)
        else
            sendATCMessage(flight,
                "departure delayed, runway occupied, standby for pushback clearance"
            )
        end
    end
end

local function handleAirborne(flight)
    if flight.status == STATUS.TAKEOFF then
        flight.status = STATUS.AIRBORNE
        releaseRunway(flight)
        sendATCMessage(flight,
            "airborne, fly runway heading, climb to flight level 80, good day"
        )
    end
end

-- =====================================================
-- MAIN TRACKING LOOP
-- =====================================================

local function updateFlight(aircraft, flight)
    -- Check pilot still present
    local pilot = getAircraftPilot(aircraft)
    if not pilot and flight.player then
        -- Pilot left — clean up
        removeFlight(aircraft)
        return
    end
    if pilot and not flight.player then
        flight.player = pilot
    end

    local altitude = getAircraftAltitude(aircraft)
    local speed    = getAircraftSpeed(aircraft)
    local dist     = distanceToAirport(aircraft)

    -- State machine transitions
    if flight.status == STATUS.AIRBORNE then
        -- Detect inbound approach
        if altitude < CONFIG.APPROACH_ALTITUDE and dist < CONFIG.APPROACH_RADIUS then
            handleApproach(flight)
        end

    elseif flight.status == STATUS.APPROACH then
        -- Refine to landing phase
        if altitude < CONFIG.LANDING_ALTITUDE * 3 then
            flight.status = STATUS.LANDING
        end

    elseif flight.status == STATUS.LANDING then
        -- Detect touchdown
        if altitude < CONFIG.LANDING_ALTITUDE and speed < CONFIG.LANDING_SPEED then
            handleLanding(flight)
        end

    elseif flight.status == STATUS.LANDED then
        -- Transition to taxi after brief pause on runway
        task.delay(5, function()
            if flights[aircraft] and flight.status == STATUS.LANDED then
                handleTaxi(flight)
            end
        end)

    elseif flight.status == STATUS.TAXI then
        -- Detect parked (very slow / stopped, low altitude)
        if speed < 2 and altitude < 5 then
            task.delay(8, function()
                if flights[aircraft] and flight.status == STATUS.TAXI and getAircraftSpeed(aircraft) < 2 then
                    handleParked(flight)
                end
            end)
        end

    elseif flight.status == STATUS.PARKED then
        -- Detect departure (speed picks up from parked)
        if speed > 5 then
            handleDeparture(flight)
        end

    elseif flight.status == STATUS.TAKEOFF then
        -- Detect airborne after takeoff roll
        if altitude > CONFIG.LANDING_ALTITUDE * 2 and speed > CONFIG.LANDING_SPEED then
            handleAirborne(flight)
        end
    end
end

-- =====================================================
-- AIRCRAFT DETECTION
-- =====================================================

local function onAircraftAdded(aircraft)
    if flights[aircraft] then return end -- Already tracked
    -- Small delay to let the aircraft fully load
    task.delay(1, function()
        if aircraft.Parent then
            createFlight(aircraft)
        end
    end)
end

local function onAircraftRemoved(aircraft)
    removeFlight(aircraft)
end

-- =====================================================
-- SYSTEM INITIALISATION
-- =====================================================

local function init()
    print("[ATC] Initialising ATC system...")

    -- Optional remote event for GUI panels
    if CONFIG.USE_REMOTE then
        local ok, remote = pcall(function()
            return ReplicatedStorage:WaitForChild("ATCMessage", 3)
        end)
        if ok and remote then
            atcRemote = remote
            print("[ATC] ATCMessage RemoteEvent found.")
        else
            warn("[ATC] ATCMessage RemoteEvent not found in ReplicatedStorage. GUI messages disabled.")
        end
    end

    -- Load runways
    loadRunways()

    -- Hook existing aircraft
    for _, aircraft in ipairs(CollectionService:GetTagged(CONFIG.AIRCRAFT_TAG)) do
        onAircraftAdded(aircraft)
    end

    -- Hook future aircraft
    CollectionService:GetInstanceAddedSignal(CONFIG.AIRCRAFT_TAG):Connect(onAircraftAdded)
    CollectionService:GetInstanceRemovedSignal(CONFIG.AIRCRAFT_TAG):Connect(onAircraftRemoved)

    print(string.format("[ATC] System online. %d runway(s) loaded.", #runways))
end

-- =====================================================
-- MAIN LOOP
-- =====================================================

init()

-- Heartbeat loop
local elapsed = 0
RunService.Heartbeat:Connect(function(dt)
    elapsed = elapsed + dt
    if elapsed < CONFIG.SCAN_INTERVAL then return end
    elapsed = 0

    -- Snapshot keys to avoid mutation during iteration
    local toUpdate = {}
    for aircraft, flight in pairs(flights) do
        table.insert(toUpdate, {aircraft, flight})
    end

    for _, pair in ipairs(toUpdate) do
        local aircraft, flight = pair[1], pair[2]
        if aircraft and aircraft.Parent then
            local ok, err = pcall(updateFlight, aircraft, flight)
            if not ok then
                warn(string.format("[ATC] Error updating flight %s: %s", flight.callsign, tostring(err)))
            end
        else
            -- Aircraft was destroyed
            removeFlight(aircraft)
        end
    end
end)

print("[ATC] System running.")
