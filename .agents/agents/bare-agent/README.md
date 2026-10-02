# Bare Agent

## When to use
Use the bare agent when you want a clean slate without any pre-configured skills or customized personas. It has access to the installed MCP servers but relies entirely on your prompts to guide its behavior. This is ideal for general-purpose tasks where you don't want any specialized instructions or toolset overhead to pollute the context.
// TIGERSPOT - CodeAI (App Lab) Implementation
// "Know Before You Park."

// --- GLOBAL STATE ---
var isVehRegistered = false;
var activeSpot = null;
var spaceA104 = "AVAILABLE";
var spaceA105 = "OCCUPIED";
var spaceA106 = "AVAILABLE";

var allElements = []; // Tracks UI elements for easy view switching

// ==========================================
// 1. UI GENERATION
// ==========================================
// This creates the UI programmatically so you don't need Design Mode.

function createUI() {
  // AUTH VIEW
  textLabel("title", "TIGERSPOT"); setPosition("title", 90, 50, 200, 40); 
  setProperty("title", "font-size", 28); setProperty("title", "text-color", "#4c1d95"); allElements.push("title");
  textLabel("tagline", "Know Before You Park."); setPosition("tagline", 90, 90, 200, 20); allElements.push("tagline");
  textInput("email", "student@school.edu"); setPosition("email", 60, 150, 200, 30); allElements.push("email");
  button("loginBtn", "LOG IN"); setPosition("loginBtn", 60, 220, 200, 40); 
  setProperty("loginBtn", "background-color", "#d97706"); allElements.push("loginBtn");

  // DASHBOARD VIEW
  textLabel("dashTitle", "Dashboard"); setPosition("dashTitle", 20, 20, 200, 30); 
  setProperty("dashTitle", "font-size", 24); allElements.push("dashTitle");
  textLabel("stats", "Welcome! Please register your vehicle."); setPosition("stats", 20, 60, 280, 40); 
  setProperty("stats", "text-color", "#991b1b"); allElements.push("stats");
  button("navMap", "Find a Parking Spot"); setPosition("navMap", 20, 110, 280, 40); allElements.push("navMap");
  button("navScan", "Scan QR Code"); setPosition("navScan", 20, 160, 280, 40); allElements.push("navScan");
  button("navVeh", "My Vehicle"); setPosition("navVeh", 20, 210, 280, 40); allElements.push("navVeh");
  button("navLogout", "Log Out"); setPosition("navLogout", 20, 350, 280, 40); 
  setProperty("navLogout", "background-color", "#991b1b"); allElements.push("navLogout");
  
  // VEHICLE VIEW
  textLabel("vehTitle", "Register Vehicle"); setPosition("vehTitle", 20, 20, 200, 30); 
  setProperty("vehTitle", "font-size", 24); allElements.push("vehTitle");
  textLabel("vehStatus", "Status: Not Registered"); setPosition("vehStatus", 20, 60, 280, 20); 
  setProperty("vehStatus", "text-color", "#991b1b"); allElements.push("vehStatus");
  textInput("plate", "License Plate (e.g. ABC-123)"); setPosition("plate", 20, 100, 280, 30); allElements.push("plate");
  textInput("make", "Make & Model (e.g. Toyota)"); setPosition("make", 20, 150, 280, 30); allElements.push("make");
  button("regBtn", "Submit Registration"); setPosition("regBtn", 20, 200, 280, 40); 
  setProperty("regBtn", "background-color", "#4c1d95"); allElements.push("regBtn");
  button("vehBack", "Back to Home"); setPosition("vehBack", 20, 350, 280, 40); allElements.push("vehBack");

  // MAP VIEW
  textLabel("mapTitle", "Select a Space"); setPosition("mapTitle", 20, 20, 200, 30); 
  setProperty("mapTitle", "font-size", 24); allElements.push("mapTitle");
  button("btnA104", "A-104 (AVAILABLE)"); setPosition("btnA104", 20, 80, 280, 40); allElements.push("btnA104");
  button("btnA105", "A-105 (OCCUPIED)"); setPosition("btnA105", 20, 130, 280, 40); allElements.push("btnA105");
  button("btnA106", "A-106 (AVAILABLE)"); setPosition("btnA106", 20, 180, 280, 40); allElements.push("btnA106");
  button("mapBack", "Back to Home"); setPosition("mapBack", 20, 350, 280, 40); allElements.push("mapBack");

  // SCAN VIEW (Simulated QR Scanner)
  textLabel("scanTitle", "Simulate QR Scan"); setPosition("scanTitle", 20, 20, 250, 30); 
  setProperty("scanTitle", "font-size", 24); allElements.push("scanTitle");
  textInput("scanInput", "Enter Space ID (e.g. A-104)"); setPosition("scanInput", 20, 100, 280, 30); allElements.push("scanInput");
  button("scanBtn", "Simulate Scan"); setPosition("scanBtn", 20, 150, 280, 40); 
  setProperty("scanBtn", "background-color", "#d97706"); allElements.push("scanBtn");
  button("scanBack", "Back to Home"); setPosition("scanBack", 20, 350, 280, 40); allElements.push("scanBack");

  // ACTIVE SESSION VIEW
  textLabel("sessTitle", "ACTIVE SESSION"); setPosition("sessTitle", 20, 20, 280, 30); 
  setProperty("sessTitle", "font-size", 24); setProperty("sessTitle", "text-color", "#15803d"); allElements.push("sessTitle");
  textLabel("sessInfo", "Spot: --"); setPosition("sessInfo", 20, 70, 280, 60); 
  setProperty("sessInfo", "font-size", 16); allElements.push("sessInfo");
  button("endSessBtn", "SIGN OUT OF PARKING"); setPosition("endSessBtn", 20, 150, 280, 50); 
  setProperty("endSessBtn", "background-color", "#991b1b"); allElements.push("endSessBtn");
  button("geoBtn", "[DEMO] Trigger Geofence Exit"); setPosition("geoBtn", 20, 250, 280, 40); 
  setProperty("geoBtn", "background-color", "#4b5563"); allElements.push("geoBtn");
}

// ==========================================
// 2. VIEW NAVIGATION LOGIC
// ==========================================
// Defines which elements belong on which "Screen"

var authView = ["title", "tagline", "email", "loginBtn"];
var dashView = ["dashTitle", "stats", "navMap", "navScan", "navVeh", "navLogout"];
var vehView = ["vehTitle", "vehStatus", "plate", "make", "regBtn", "vehBack"];
var mapView = ["mapTitle", "btnA104", "btnA105", "btnA106", "mapBack"];
var scanView = ["scanTitle", "scanInput", "scanBtn", "scanBack"];
var sessView = ["sessTitle", "sessInfo", "endSessBtn", "geoBtn"];

function setView(viewArray) {
  // Hide everything first
  for (var i = 0; i < allElements.length; i++) {
    hideElement(allElements[i]);
  }
  // Show only the requested elements
  for (var j = 0; j < viewArray.length; j++) {
    showElement(viewArray[j]);
  }
}

// ==========================================
// 3. PARKING LOGIC & EVENTS
// ==========================================

function updateMapColors() {
  setText("btnA104", "A-104 (" + spaceA104 + ")");
  setProperty("btnA104", "background-color", spaceA104 == "AVAILABLE" ? "#15803d" : "#991b1b");
  
  setText("btnA105", "A-105 (" + spaceA105 + ")");
  setProperty("btnA105", "background-color", spaceA105 == "AVAILABLE" ? "#15803d" : "#991b1b");
  
  setText("btnA106", "A-106 (" + spaceA106 + ")");
  setProperty("btnA106", "background-color", spaceA106 == "AVAILABLE" ? "#15803d" : "#991b1b");
}

function startSession(spotId) {
  if (!isVehRegistered) {
    setText("stats", "ACTION REQUIRED: Register vehicle first!");
    setProperty("stats", "text-color", "#991b1b");
    setView(dashView);
    return;
  }
  
  var success = false;
  if (spotId == "A-104" && spaceA104 == "AVAILABLE") {
    spaceA104 = "OCCUPIED"; success = true;
  } else if (spotId == "A-106" && spaceA106 == "AVAILABLE") {
    spaceA106 = "OCCUPIED"; success = true;
  }

  if (success) {
    activeSpot = spotId;
    setText("sessInfo", "Spot: " + activeSpot + "\nStatus: ACTIVE\nVehicle Verified");
    setView(sessView);
  } else {
    setText("stats", "Sorry, space " + spotId + " is occupied or invalid.");
    setProperty("stats", "text-color", "#991b1b");
    setView(dashView);
  }
}

function endSession() {
  if (activeSpot == "A-104") spaceA104 = "AVAILABLE";
  if (activeSpot == "A-106") spaceA106 = "AVAILABLE";
  activeSpot = null;
  setText("stats", "You are signed out. Space is available.");
  setProperty("stats", "text-color", "#15803d");
  setView(dashView);
}

// --- BUTTON CLICKS ---

onEvent("loginBtn", "click", function() {
  setView(dashView);
});
onEvent("navLogout", "click", function() {
  setView(authView);
});
onEvent("navMap", "click", function() {
  updateMapColors();
  setView(mapView);
});
onEvent("navScan", "click", function() {
  setView(scanView);
});
onEvent("navVeh", "click", function() {
  setView(vehView);
});
onEvent("vehBack", "click", function() { setView(dashView); });
onEvent("mapBack", "click", function() { setView(dashView); });
onEvent("scanBack", "click", function() { setView(dashView); });

// Vehicle Registration
onEvent("regBtn", "click", function() {
  isVehRegistered = true;
  setText("vehStatus", "Status: VERIFIED");
  setProperty("vehStatus", "text-color", "#15803d");
  setText("stats", "Vehicle registered! You can now park.");
  setProperty("stats", "text-color", "#15803d");
});

// Map Clicks
onEvent("btnA104", "click", function() { startSession("A-104"); });
onEvent("btnA105", "click", function() { startSession("A-105"); });
onEvent("btnA106", "click", function() { startSession("A-106"); });

// QR Simulation
onEvent("scanBtn", "click", function() {
  var scannedId = getText("scanInput");
  startSession(scannedId);
});

// Session Management
onEvent("endSessBtn", "click", function() {
  endSession();
});

// Geofencing Simulation
onEvent("geoBtn", "click", function() {
  setText("sessInfo", "⚠️ LOCATION ALERT:\nLooks like you left the parking area.\nDid you leave spot " + activeSpot + "?");
});

// ==========================================
// 4. BOOT UP APP
// ==========================================
createUI();
setView(authView);
