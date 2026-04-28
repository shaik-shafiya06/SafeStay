<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes">
  <title>SafeStay • Voice & Multi‑Lang Crisis Bridge</title>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <style>
    * { margin:0; padding:0; box-sizing:border-box; }
    body { font-family: 'Inter', sans-serif; background:#f8fafc; color:#1e293b; height:100vh; display:flex; }
    .app-nav { width:220px; background:white; border-right:1px solid #e2e8f0; padding:2rem 1rem; display:flex; flex-direction:column; gap:0.8rem; }
    .logo { font-size:1.4rem; font-weight:700; color:#0f172a; margin-bottom:1.5rem; display:flex; align-items:center; gap:0.5rem; }
    .nav-item { padding:0.8rem 1.2rem; border-radius:12px; cursor:pointer; font-weight:500; color:#475569; transition:0.2s; display:flex; align-items:center; gap:0.7rem; }
    .nav-item:hover { background:#f1f5f9; color:#0f172a; }
    .nav-item.active { background:#0f172a; color:white; font-weight:600; }
    .main-panel { flex:1; overflow-y:auto; padding:2rem; }
    .card { background:white; border-radius:24px; padding:2rem; box-shadow:0 4px 20px rgba(0,0,0,0.03); border:1px solid #f1f5f9; }
    button { background:#0f172a; color:white; border:none; border-radius:14px; padding:0.75rem 1.5rem; font-weight:600; cursor:pointer; transition:0.2s; display:inline-flex; align-items:center; gap:0.5rem; }
    button:hover { background:#1e293b; transform:translateY(-1px); }
    button.outline { background:transparent; border:1px solid #cbd5e1; color:#334155; }
    button.danger { background:#ef4444; }
    button.call-btn { background:#16a34a; }
    button.voice-btn { background:#8b5cf6; }
    input, select { width:100%; padding:0.8rem 1rem; border:1px solid #e2e8f0; border-radius:14px; background:#ffffff; font-size:0.95rem; margin:0.5rem 0; }
    .badge { display:inline-block; padding:0.2rem 0.8rem; border-radius:20px; font-size:0.7rem; font-weight:600; }
    .badge-critical { background:#fee2e2; color:#b91c1c; }
    .badge-urgent { background:#fef3c7; color:#b45309; }
    .badge-routine { background:#dcfce7; color:#166534; }
    .map-container { height:350px; border-radius:20px; overflow:hidden; border:1px solid #e2e8f0; }
    .chat-window { background:#f8fafc; border-radius:16px; padding:1rem; height:180px; overflow-y:auto; font-size:0.9rem; border:1px solid #e2e8f0; }
    .incident-row { display:flex; align-items:center; justify-content:space-between; padding:0.8rem 1rem; margin-bottom:0.5rem; background:white; border-left:4px solid; border-radius:12px; cursor:pointer; }
    .incident-row:hover { background:#f8fafc; }
    .incident-row.critical { border-color:#ef4444; }
    .incident-row.urgent { border-color:#f59e0b; }
    .incident-row.routine { border-color:#10b981; }
    .grid-2col { display:grid; grid-template-columns:1fr 1fr; gap:1.2rem; }
    .hidden { display:none !important; }
    @keyframes pulse { 0% { box-shadow: 0 0 0 0 rgba(239,68,68,0.4); } 70% { box-shadow: 0 0 0 18px rgba(239,68,68,0); } 100% { box-shadow: 0 0 0 0 rgba(239,68,68,0); } }
    .panic-btn { background:#ef4444; border:none; color:white; width:120px; height:120px; border-radius:50%; font-size:1.1rem; font-weight:700; cursor:pointer; animation:pulse 2s infinite; box-shadow:0 10px 25px rgba(239,68,68,0.3); }
    .panic-btn:active { transform:scale(0.92); }
    .live-dot { display:inline-block; width:10px; height:10px; border-radius:50%; background:#10b981; animation: blink 1s infinite; }
    @keyframes blink { 50% { opacity:0.3; } }
    .location-badge { font-size:0.8rem; color:#10b981; display:flex; align-items:center; gap:0.3rem; margin:0.3rem 0; }
    .voice-recording { background:#fef3c7; padding:0.5rem; border-radius:12px; display:inline-block; margin:0.5rem 0; }
  </style>
</head>
<body>
<div class="app-nav">
  <div class="logo"><i class="fas fa-shield-haltered" style="color:#2563eb;"></i> SafeStay</div>
  <div class="nav-item active" data-panel="guest">🚨 Guest</div>
  <div class="nav-item" data-panel="staff">👨‍⚕️ Staff</div>
  <div class="nav-item" data-panel="command">🏢 Command</div>
  <div class="nav-item" data-panel="police">🚔 Dispatch</div>
</div>

<div class="main-panel">
  <!-- GUEST PANEL -->
  <div id="guestPanel">
    <div class="card" style="max-width:500px; margin:0 auto; text-align:center;">
      <i class="fas fa-hotel" style="font-size:2rem; color:#2563eb;"></i>
      <h2 style="font-weight:600;">Emergency Portal</h2>
      <p style="color:#64748b;" id="langSubtitle">Verified by PIN • Live location sent</p>
      <div style="margin:0.5rem 0;">
        <button class="outline" onclick="toggleLanguage()" style="font-size:0.8rem; padding:0.3rem 1rem;">🌐 <span id="langBtn">हिन्दी</span></button>
      </div>
      <div id="pinSection">
        <input type="text" id="pinInput" placeholder="Booking PIN (e.g. 12345)" value="12345">
        <button onclick="verifyPin()" style="width:100%;">Verify PIN</button>
      </div>
      <div id="emergencySection" class="hidden">
        <div style="background:#f1f5f9; border-radius:12px; padding:1rem; margin-bottom:1rem;">
          <span style="font-weight:600;" id="guestName"></span> · Room <strong id="guestRoom"></strong>
          <div id="medicalNote" class="badge badge-urgent" style="display:none; margin-top:0.3rem;"></div>
          <div class="location-badge"><i class="fas fa-map-marker-alt"></i> <span id="liveLocationLabel">Location not shared</span></div>
        </div>
        <select id="emergencyType">
          <option value="Medical">⚕️ Medical</option>
          <option value="Fire">🔥 Fire</option>
          <option value="Suspicious">🕵️ Suspicious</option>
          <option value="Fall">🦽 Fall / Immobile</option>
        </select>
        <div style="display:flex; gap:0.5rem; justify-content:center; margin:1rem 0;">
          <button class="call-btn" onclick="simulateCall('108')"><i class="fas fa-phone"></i> Call 108</button>
          <button class="call-btn" onclick="simulateCall('100')"><i class="fas fa-phone"></i> Call Police</button>
        </div>
        <button class="panic-btn" onclick="triggerEmergency()">PANIC</button>
        <p style="margin-top:0.5rem; font-size:0.8rem; color:#64748b;">or shake device (simulated fall)</p>
        <button class="outline" onclick="simulateFall()" style="margin-top:0.5rem;">Simulate Fall</button>
        <!-- Voice Message -->
        <div style="margin-top:1.5rem; background:#f5f3ff; border-radius:16px; padding:1rem;">
          <p style="font-weight:600;"><i class="fas fa-microphone"></i> Voice Help</p>
          <button id="recordBtn" class="voice-btn" onclick="toggleRecording()"><i class="fas fa-microphone"></i> Record Message</button>
          <div id="recordingStatus" class="hidden voice-recording">🔴 Recording... <span id="recordTimer">0</span>s</div>
          <audio id="audioPlayback" controls class="hidden" style="margin-top:0.5rem; width:100%;"></audio>
          <p style="font-size:0.8rem; margin-top:0.3rem;">Your recorded message will be sent automatically.</p>
        </div>
        <div id="guestStatus" class="hidden" style="margin-top:2rem; text-align:left;">
          <div style="display:flex; align-items:center; gap:0.5rem; margin-bottom:0.5rem;">
            <span class="live-dot"></span><strong id="statusText">Alert sent</strong>
          </div>
          <div id="autoDispatchInfo"></div>
          <div class="chat-window" id="guestChat" style="margin-top:1rem;"></div>
          <div style="display:flex; gap:0.5rem; margin-top:0.8rem;">
            <input type="text" id="guestChatInput" placeholder="Message..."><button onclick="sendGuestChat()">Send</button>
          </div>
          <div class="map-container" id="guestMiniMap" style="height:180px; margin-top:1rem;"></div>
          <div style="margin-top:1rem;"><label><input type="checkbox" id="doorUnlock" onchange="toggleDoorUnlock()"> Allow responder to unlock door</label></div>
          <button onclick="speakStatus()" style="margin-top:0.5rem; background:#64748b;">🔊 Read Status Aloud</button>
        </div>
        <div id="callStatus" class="hidden" style="margin-top:1rem; background:#fef3c7; padding:0.8rem; border-radius:12px;">
          <i class="fas fa-phone-alt"></i> <span id="callText"></span>
        </div>
      </div>
    </div>
  </div>

  <!-- STAFF PANEL -->
  <div id="staffPanel" class="hidden">
    <div class="grid-2col">
      <div class="map-container" id="staffMap"></div>
      <div class="card">
        <div style="display:flex; align-items:center; gap:1rem;">
          <i class="fas fa-user-circle" style="font-size:2.5rem; color:#2563eb;"></i>
          <div><h3 id="staffNameDisplay">Sarah (Medic)</h3><span style="color:#64748b;">First Responder</span></div>
        </div>
        <div id="staffAssignment" class="hidden">
          <div style="background:#fef3c7; border-radius:12px; padding:1rem; margin:1rem 0;">
            <i class="fas fa-bell"></i> <strong>New assignment</strong>
            <p id="incidentDetails"></p>
          </div>
          <button id="acceptBtn" onclick="staffAction('accept')" style="background:#10b981;">Accept</button>
          <button id="arriveBtn" onclick="staffAction('arrive')" class="hidden outline">Arrived</button>
          <button id="resolveBtn" onclick="staffAction('resolve')" class="hidden danger">Resolve</button>
        </div>
        <div id="noAssignment" style="color:#64748b; text-align:center; margin-top:2rem;">
          <i class="fas fa-check-circle" style="font-size:2rem;"></i><p>No active assignment</p>
        </div>
      </div>
    </div>
  </div>

  <!-- COMMAND PANEL -->
  <div id="commandPanel" class="hidden">
    <div class="grid-2col">
      <div class="card" style="overflow-y:auto;">
        <h3><i class="fas fa-clipboard-list" style="color:#2563eb;"></i> Incidents</h3>
        <div id="incidentList"></div>
        <h3 style="margin-top:2rem;"><i class="fas fa-users" style="color:#2563eb;"></i> Staff</h3>
        <div id="staffList"></div>
        <div id="commandManage" class="hidden" style="margin-top:2rem;">
          <h4>Manage Incident</h4>
          <select id="staffSelect"></select>
          <button onclick="dispatchStaff()">Dispatch Staff</button>
          <div class="chat-window" id="cmdChat" style="margin-top:1rem;"></div>
          <div style="display:flex; gap:0.5rem; margin-top:0.5rem;">
            <input type="text" id="cmdChatInput" placeholder="Message..."><button onclick="sendCmdChat()">Send</button>
          </div>
        </div>
      </div>
      <div>
        <div class="map-container" id="cmdMap" style="margin-bottom:1rem;"></div>
        <div class="card">
          <h4><i class="fas fa-clock-rotate-left" style="color:#2563eb;"></i> Event Log</h4>
          <div id="eventLog" style="max-height:150px; overflow-y:auto; font-size:0.85rem; color:#475569;"></div>
        </div>
      </div>
    </div>
  </div>

  <!-- POLICE VIEW -->
  <div id="policePanel" class="hidden">
    <div class="card">
      <h3><i class="fas fa-shield" style="color:#2563eb;"></i> Emergency Dispatch</h3>
      <div class="grid-2col" style="margin-top:1rem;">
        <div>
          <h4>Active Incidents</h4>
          <div id="policeIncidentList"></div>
        </div>
        <div class="map-container" id="policeMap"></div>
      </div>
    </div>
  </div>
</div>

<script>
  // ---------- DATA ----------
  const guestsDB = {
    '12345': { name:'John Doe', room:'304', medical:'Allergy: Penicillin' },
    '67890': { name:'Jane Smith', room:'512', medical:'None' },
    '11111': { name:'Bob Brown', room:'201', medical:'Diabetic' }
  };
  const staffDB = [
    { id:'staff1', name:'Sarah (Medic)', skill:'First Aid', available:true },
    { id:'staff2', name:'Mike (Security)', skill:'Security', available:true },
    { id:'staff3', name:'Emma (General)', skill:'General', available:true }
  ];
  const policeStations = [
    { name:'West End Police', lat:51.5072, lng:-0.0915, phone:'999-111' },
    { name:'City Center Police', lat:51.5038, lng:-0.0872, phone:'999-222' }
  ];
  const hospitals = [
    { name:'St. Thomas Hospital', lat:51.5069, lng:-0.0895, phone:'911-333', ambulanceETA:'4 min' },
    { name:'Royal London Clinic', lat:51.5025, lng:-0.0885, phone:'911-444', ambulanceETA:'7 min' }
  ];
  const aedLocations = [ { name:'AED Lobby', lat:51.5055, lng:-0.0905 }, { name:'AED 3rd Floor', lat:51.5048, lng:-0.091 } ];
  const roomCoords = { '304':[51.505,-0.09], '512':[51.506,-0.091], '201':[51.504,-0.089] };

  let currentGuest=null, currentPin=null, incidents=[], selectedIncidentId=null, assignedIncident=null;
  let guestLivePosition=null, liveTrackingInterval=null;
  let mediaRecorder=null, audioChunks=[], recordingStartTime=null, recordingTimerId=null;
  let currentLanguage = 'en'; // 'en','hi','te'

  // Language content
  const lang = {
    en: {
      subtitle: 'Verified by PIN • Live location sent',
      panic: 'PANIC',
      verify: 'Verify PIN',
      recording: 'Recording...',
      fallBtn: 'Simulate Fall',
      call108: 'Call 108',
      call100: 'Call Police',
      recordBtn: 'Record Message',
      send: 'Send',
      speakBtn: 'Read Status Aloud',
      doorLabel: 'Allow responder to unlock door'
    },
    hi: {
      subtitle: 'पिन से सत्यापित • लाइव लोकेशन भेजी गई',
      panic: 'आपातकाल',
      verify: 'पिन सत्यापित करें',
      recording: 'रिकॉर्डिंग...',
      fallBtn: 'गिरने का अनुकरण करें',
      call108: '108 पर कॉल करें',
      call100: 'पुलिस को कॉल करें',
      recordBtn: 'संदेश रिकॉर्ड करें',
      send: 'भेजें',
      speakBtn: 'स्थिति सुनाएं',
      doorLabel: 'प्रतिसादकर्ता को दरवाजा खोलने दें'
    },
    te: {
      subtitle: 'PIN ద్వారా ధృవీకరించబడింది • లైవ్ లొకేషన్ పంపబడింది',
      panic: 'అత్యవసరం',
      verify: 'PIN ధృవీకరించండి',
      recording: 'రికార్డింగ్...',
      fallBtn: 'పడిపోవడాన్ని అనుకరించండి',
      call108: '108కి కాల్ చేయండి',
      call100: 'పోలీసులకు కాల్ చేయండి',
      recordBtn: 'సందేశాన్ని రికార్డ్ చేయండి',
      send: 'పంపండి',
      speakBtn: 'స్థితిని చదవండి',
      doorLabel: 'ప్రతిస్పందకుడు తలుపు తీయడానికి అనుమతించండి'
    }
  };

  // Map icons
  const policeIcon = L.icon({iconUrl:'https://cdn-icons-png.flaticon.com/512/149/149060.png', iconSize:[32,32]});
  const hospitalIcon = L.icon({iconUrl:'https://cdn-icons-png.flaticon.com/512/2966/2966327.png', iconSize:[32,32]});
  const guestIcon = L.icon({iconUrl:'https://cdn-icons-png.flaticon.com/512/684/684908.png', iconSize:[30,30]});

  // Maps
  let staffMap, cmdMap, policeMap, guestMiniMap;
  function initMaps() {
    staffMap = L.map('staffMap').setView([51.505,-0.09],17);
    L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png').addTo(staffMap);
    cmdMap = L.map('cmdMap').setView([51.505,-0.09],17);
    L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png').addTo(cmdMap);
    policeMap = L.map('policeMap').setView([51.505,-0.09],17);
    L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png').addTo(policeMap);
    guestMiniMap = L.map('guestMiniMap').setView([51.505,-0.09],17);
    L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png').addTo(guestMiniMap);
  }

  window.onload = () => { initMaps(); updateAllMaps(); };

  // Navigation
  document.querySelectorAll('.nav-item').forEach(item=>{
    item.addEventListener('click', ()=>{
      document.querySelectorAll('.nav-item').forEach(i=>i.classList.remove('active'));
      item.classList.add('active');
      document.querySelectorAll('.main-panel > div').forEach(p=>p.classList.add('hidden'));
      document.getElementById(item.dataset.panel+'Panel').classList.remove('hidden');
      if(item.dataset.panel==='staff') updateStaffMap();
      if(item.dataset.panel==='command'){ updateCommandMap(); renderCommand(); }
      if(item.dataset.panel==='police'){ updatePoliceMap(); renderPoliceView(); }
    });
  });

  // ---------- LANGUAGE TOGGLE (en -> hi -> te -> en) ----------
  function toggleLanguage() {
    const langs = ['en','hi','te'];
    const currentIdx = langs.indexOf(currentLanguage);
    const nextIdx = (currentIdx + 1) % langs.length;
    currentLanguage = langs[nextIdx];
    const l = lang[currentLanguage];
    document.getElementById('langSubtitle').textContent = l.subtitle;
    document.querySelector('.panic-btn').textContent = l.panic;
    document.querySelector('#pinSection button').textContent = l.verify;
    document.getElementById('recordBtn').innerHTML = `<i class="fas fa-microphone"></i> ${l.recordBtn}`;
    document.querySelector('.outline[onclick="simulateFall()"]').textContent = l.fallBtn;
    document.querySelectorAll('.call-btn')[0].innerHTML = `<i class="fas fa-phone"></i> ${l.call108}`;
    document.querySelectorAll('.call-btn')[1].innerHTML = `<i class="fas fa-phone"></i> ${l.call100}`;
    document.querySelector('#guestStatus button[onclick="speakStatus()"]').textContent = `🔊 ${l.speakBtn}`;
    document.querySelector('label[for="doorUnlock"]').childNodes[1].textContent = l.doorLabel;
    document.getElementById('langBtn').textContent = currentLanguage==='en'?'हिन्दी / తెలుగు':(currentLanguage==='hi'?'తెలుగు / English':'English / हिन्दी');
  }

  // Verify PIN
  function verifyPin() {
    const pin = document.getElementById('pinInput').value.trim();
    if(guestsDB[pin]){
      currentGuest=guestsDB[pin]; currentPin=pin;
      document.getElementById('pinSection').classList.add('hidden');
      document.getElementById('emergencySection').classList.remove('hidden');
      document.getElementById('guestName').textContent=currentGuest.name;
      document.getElementById('guestRoom').textContent=currentGuest.room;
      if(currentGuest.medical!=='None'){
        document.getElementById('medicalNote').style.display='inline-block';
        document.getElementById('medicalNote').textContent='⚠️ '+currentGuest.medical;
      }
      requestLocation();
    } else alert('Invalid PIN');
  }

  // ---------- LOCATION ----------
  function requestLocation(cb){
    if(navigator.geolocation){
      navigator.geolocation.getCurrentPosition(pos=>{
        guestLivePosition = { lat:pos.coords.latitude, lng:pos.coords.longitude };
        document.getElementById('liveLocationLabel').textContent = `Live (${guestLivePosition.lat.toFixed(4)}, ${guestLivePosition.lng.toFixed(4)})`;
        updateGuestMiniMap();
        if(liveTrackingInterval) clearInterval(liveTrackingInterval);
        liveTrackingInterval = setInterval(()=>{
          navigator.geolocation.getCurrentPosition(p=>{
            guestLivePosition={lat:p.coords.latitude,lng:p.coords.longitude};
            updateGuestMiniMap(); updateAllMaps();
          });
        },5000);
        if(cb) cb(true);
      }, err=>{
        alert('Location access denied. Using room approximate.');
        const rc = roomCoords[currentGuest.room]||[51.505,-0.09];
        guestLivePosition={lat:rc[0],lng:rc[1]};
        if(cb) cb(false);
      },{enableHighAccuracy:true});
    } else if(cb) cb(false);
  }

  function updateGuestMiniMap() {
    if(!guestMiniMap||!guestLivePosition) return;
    guestMiniMap.eachLayer(layer=>{if(layer instanceof L.Marker) guestMiniMap.removeLayer(layer);});
    L.marker([guestLivePosition.lat,guestLivePosition.lng], {icon: guestIcon}).addTo(guestMiniMap).bindPopup('You');
    policeStations.forEach(ps=> L.marker([ps.lat,ps.lng],{icon:policeIcon}).addTo(guestMiniMap).bindPopup(ps.name));
    hospitals.forEach(h=> L.marker([h.lat,h.lng],{icon:hospitalIcon}).addTo(guestMiniMap).bindPopup(h.name));
    aedLocations.forEach(a=> L.circleMarker([a.lat,a.lng],{radius:6,color:'#2563eb'}).addTo(guestMiniMap).bindPopup(a.name));
    guestMiniMap.setView([guestLivePosition.lat,guestLivePosition.lng],17);
  }

  // ---------- VOICE RECORDING ----------
  async function toggleRecording() {
    if(mediaRecorder && mediaRecorder.state==='recording') {
      mediaRecorder.stop();
      return;
    }
    try {
      const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
      mediaRecorder = new MediaRecorder(stream);
      audioChunks = [];
      mediaRecorder.ondataavailable = e => audioChunks.push(e.data);
      mediaRecorder.onstop = () => {
        const audioBlob = new Blob(audioChunks, { type:'audio/webm' });
        const audioUrl = URL.createObjectURL(audioBlob);
        document.getElementById('audioPlayback').src = audioUrl;
        document.getElementById('audioPlayback').classList.remove('hidden');
        document.getElementById('recordingStatus').classList.add('hidden');
        clearInterval(recordingTimerId);
        sendVoiceMessage();
      };
      mediaRecorder.start();
      document.getElementById('recordBtn').classList.add('hidden');
      document.getElementById('recordingStatus').classList.remove('hidden');
      recordingStartTime = Date.now();
      recordingTimerId = setInterval(()=>{
        document.getElementById('recordTimer').textContent = Math.floor((Date.now()-recordingStartTime)/1000);
      },1000);
    } catch(e) {
      alert('Microphone access denied. Please use text messaging.');
    }
  }

  function sendVoiceMessage() {
    if(!audioChunks.length || !currentPin) return;
    const inc = incidents.find(i=>i.pin===currentPin && i.status!=='resolved');
    if(inc){
      const audioBlob = new Blob(audioChunks, { type:'audio/webm' });
      inc.chatLog.push({from:'Guest (voice)', msg:'[Voice message]', audio: URL.createObjectURL(audioBlob)});
      refreshChats(inc);
    }
    audioChunks=[];
    document.getElementById('audioPlayback').classList.add('hidden');
    document.getElementById('recordBtn').classList.remove('hidden');
  }

  // ---------- EMERGENCY ----------
  function triggerEmergency(typeOverride) {
    if(!currentGuest) return;
    requestLocation(()=>{
      const type = typeOverride||document.getElementById('emergencyType').value;
      const severity = (type==='Medical'||type==='Fire'||type==='Fall')?'critical':(type==='Suspicious'?'urgent':'routine');
      const inc = {
        id:'INC-'+Date.now(), pin:currentPin, guestName:currentGuest.name, room:currentGuest.room,
        type, severity, status:'new', assignedStaff:null,
        chatLog:[{from:'system', msg:'Emergency triggered.'}],
        liveLocation: guestLivePosition
      };
      incidents.push(inc);
      document.getElementById('guestStatus').classList.remove('hidden');
      document.getElementById('statusText').textContent = 'Alert sent – help dispatched';
      let nearPolice = policeStations[0], nearHospital = hospitals[0];
      if(guestLivePosition){
        nearPolice = policeStations.reduce((a,b)=> Math.hypot(a.lat-guestLivePosition.lat,a.lng-guestLivePosition.lng) < Math.hypot(b.lat-guestLivePosition.lat,b.lng-guestLivePosition.lng) ? a : b);
        nearHospital = hospitals.reduce((a,b)=> Math.hypot(a.lat-guestLivePosition.lat,a.lng-guestLivePosition.lng) < Math.hypot(b.lat-guestLivePosition.lat,b.lng-guestLivePosition.lng) ? a : b);
      }
      inc.nearestPolice=nearPolice; inc.nearestHospital=nearHospital;
      inc.chatLog.push({from:'system', msg:`${nearPolice.name} notified.`});
      inc.chatLog.push({from:'system', msg:`${nearHospital.name} ambulance ETA ${nearHospital.ambulanceETA}.`});
      document.getElementById('autoDispatchInfo').innerHTML = `
        <div><i class="fas fa-shield-haltered"></i> ${nearPolice.name} alerted</div>
        <div><i class="fas fa-ambulance"></i> ${nearHospital.name} en route (${nearHospital.ambulanceETA})</div>
      `;
      playAlarm();
      updateAll();
      addLog(`🚨 ${inc.id} at location. Police & ambulance dispatched.`);
      speakStatus();
    });
  }

  function simulateFall() { triggerEmergency('Fall'); }
  function simulateCall(num) {
    document.getElementById('callStatus').classList.remove('hidden');
    document.getElementById('callText').textContent = `Calling ${num}... (simulated)`;
    setTimeout(()=>document.getElementById('callStatus').classList.add('hidden'),5000);
  }
  function toggleDoorUnlock() {
    addLog('Door unlock permission '+(document.getElementById('doorUnlock').checked?'granted':'revoked'));
  }
  function playAlarm() {
    try{ const ctx=new (window.AudioContext||window.webkitAudioContext)(); const o=ctx.createOscillator(); o.type='sine'; o.frequency.setValueAtTime(800,ctx.currentTime); o.connect(ctx.destination); o.start(); o.stop(ctx.currentTime+0.3); }catch(e){}
  }

  function speakStatus() {
    if(!('speechSynthesis' in window)) return;
    const l = lang[currentLanguage];
    let text = currentLanguage==='en'?'Emergency alert sent. ':(currentLanguage==='hi'?'आपातकालीन चेतावनी भेजी गई। ':'ఎమర్జెన్సీ హెచ్చరిక పంపబడింది. ');
    const inc = incidents.find(i=>i.pin===currentPin && i.status!=='resolved');
    if(inc){
      text += `Status: ${inc.status}. `;
      if(inc.nearestPolice) text += (currentLanguage==='en'?`Police from ${inc.nearestPolice.name} notified. `:(currentLanguage==='hi'?`${inc.nearestPolice.name} पुलिस को सूचित किया गया। `:`${inc.nearestPolice.name} పోలీసులకు సమాచారం ఇవ్వబడింది. `));
      if(inc.nearestHospital) text += (currentLanguage==='en'?`Ambulance from ${inc.nearestHospital.name} ETA ${inc.nearestHospital.ambulanceETA}.`:(currentLanguage==='hi'?`${inc.nearestHospital.name} से एम्बुलेंस आ रही है, समय ${inc.nearestHospital.ambulanceETA}`:`${inc.nearestHospital.name} నుండి అంబులెన్స్ వస్తోంది, సమయం ${inc.nearestHospital.ambulanceETA}`));
    }
    window.speechSynthesis.speak(new SpeechSynthesisUtterance(text));
  }

  function sendGuestChat() {
    const msg = document.getElementById('guestChatInput').value.trim();
    const inc = incidents.find(i=>i.pin===currentPin && i.status!=='resolved');
    if(msg && inc){ inc.chatLog.push({from:'Guest', msg}); document.getElementById('guestChatInput').value=''; refreshChats(inc); }
    sendVoiceMessage();
  }

  function staffAction(action) {
    if(!assignedIncident) return;
    const staffMember = staffDB.find(s=>s.id==='staff1');
    if(action==='accept'){
      assignedIncident.status='accepted';
      assignedIncident.chatLog.push({from:'system', msg:`${staffMember.name} accepted.`});
      document.getElementById('acceptBtn').classList.add('hidden');
      document.getElementById('arriveBtn').classList.remove('hidden');
    } else if(action==='arrive'){
      assignedIncident.status='onScene';
      assignedIncident.chatLog.push({from:'system', msg:`${staffMember.name} arrived.`});
      document.getElementById('arriveBtn').classList.add('hidden');
      document.getElementById('resolveBtn').classList.remove('hidden');
    } else if(action==='resolve'){
      assignedIncident.status='resolved';
      staffMember.available=true;
      assignedIncident.chatLog.push({from:'system', msg:`Resolved by ${staffMember.name}.`});
      document.getElementById('staffAssignment').classList.add('hidden');
      document.getElementById('noAssignment').classList.remove('hidden');
      assignedIncident=null;
      if(liveTrackingInterval) clearInterval(liveTrackingInterval);
    }
    updateAll();
  }

  function renderCommand() {
    document.getElementById('incidentList').innerHTML = incidents.map(inc=>`
      <div class="incident-row ${inc.severity}" onclick="selectIncident('${inc.id}')">
        <div><strong>${inc.id}</strong> – Room ${inc.room}<br><small>${inc.type} | ${inc.status}</small></div>
        <span class="badge badge-${inc.severity}">${inc.severity}</span>
      </div>`).join('');
    document.getElementById('staffList').innerHTML = staffDB.map(s=>
      `<div style="display:flex;justify-content:space-between;"><span>${s.name} (${s.skill})</span><span style="color:${s.available?'#10b981':'#ef4444'};">${s.available?'Available':'Busy'}</span></div>`).join('');
    updateCommandMap();
  }

  function updateCommandMap() {
    if(!cmdMap) return;
    cmdMap.eachLayer(layer=>{if(layer instanceof L.Marker) cmdMap.removeLayer(layer);});
    Object.entries(roomCoords).forEach(([room,coord])=>L.marker(coord).addTo(cmdMap).bindPopup(room));
    policeStations.forEach(ps=> L.marker([ps.lat,ps.lng],{icon:policeIcon}).addTo(cmdMap).bindPopup(ps.name));
    hospitals.forEach(h=> L.marker([h.lat,h.lng],{icon:hospitalIcon}).addTo(cmdMap).bindPopup(h.name));
    incidents.forEach(inc=>{ if(inc.liveLocation) L.marker([inc.liveLocation.lat,inc.liveLocation.lng],{icon:guestIcon}).addTo(cmdMap).bindPopup(inc.id); });
  }

  function updateStaffMap() {
    if(!staffMap) return;
    staffMap.eachLayer(layer=>{if(layer instanceof L.Marker) staffMap.removeLayer(layer);});
    Object.entries(roomCoords).forEach(([room,coord])=>L.marker(coord).addTo(staffMap).bindPopup(room));
    policeStations.forEach(ps=> L.marker([ps.lat,ps.lng],{icon:policeIcon}).addTo(staffMap).bindPopup(ps.name));
    hospitals.forEach(h=> L.marker([h.lat,h.lng],{icon:hospitalIcon}).addTo(staffMap).bindPopup(h.name));
    if(assignedIncident && assignedIncident.liveLocation) {
      L.marker([assignedIncident.liveLocation.lat,assignedIncident.liveLocation.lng],{icon:guestIcon}).addTo(staffMap).bindPopup('Guest');
      staffMap.setView([assignedIncident.liveLocation.lat,assignedIncident.liveLocation.lng],17);
    }
  }

  function updatePoliceMap() {
    if(!policeMap) return;
    policeMap.eachLayer(layer=>{if(layer instanceof L.Marker) policeMap.removeLayer(layer);});
    policeStations.forEach(ps=> L.marker([ps.lat,ps.lng],{icon:policeIcon}).addTo(policeMap).bindPopup(ps.name));
    hospitals.forEach(h=> L.marker([h.lat,h.lng],{icon:hospitalIcon}).addTo(policeMap).bindPopup(h.name));
    incidents.forEach(inc=>{ if(inc.liveLocation) L.marker([inc.liveLocation.lat,inc.liveLocation.lng]).addTo(policeMap).bindPopup(inc.id); });
  }

  function renderPoliceView() {
    document.getElementById('policeIncidentList').innerHTML = incidents.map(inc=>`
      <div style="padding:0.5rem; margin-bottom:0.5rem; background:white; border-left:3px solid #ef4444;">
        <strong>${inc.id}</strong> – ${inc.guestName}, Room ${inc.room}<br>
        Location: ${inc.liveLocation ? inc.liveLocation.lat.toFixed(4)+','+inc.liveLocation.lng.toFixed(4) : 'N/A'}<br>
        <small>Status: ${inc.status}</small>
      </div>`).join('');
    updatePoliceMap();
  }

  window.selectIncident = (id) => {
    selectedIncidentId = id;
    const inc = incidents.find(i=>i.id===id);
    if(!inc) return;
    document.getElementById('commandManage').classList.remove('hidden');
    document.getElementById('staffSelect').innerHTML = staffDB.filter(s=>s.available).map(s=>`<option value="${s.id}">${s.name}</option>`).join('');
    document.getElementById('cmdChat').innerHTML = inc.chatLog.map(m=>`<div><b>${m.from}:</b> ${m.msg} ${m.audio ? `<audio controls src="${m.audio}"></audio>` : ''}</div>`).join('');
  };

  function dispatchStaff() {
    if(!selectedIncidentId) return;
    const staffId = document.getElementById('staffSelect').value;
    const inc = incidents.find(i=>i.id===selectedIncidentId);
    const staffMember = staffDB.find(s=>s.id===staffId);
    if(inc && staffMember) {
      inc.assignedStaff = staffId; inc.status = 'dispatched'; staffMember.available = false;
      inc.chatLog.push({from:'system', msg:`Dispatched ${staffMember.name}.`});
      if(staffId==='staff1') {
        assignedIncident = inc;
        document.getElementById('staffAssignment').classList.remove('hidden');
        document.getElementById('noAssignment').classList.add('hidden');
        document.getElementById('incidentDetails').innerHTML = `Room ${inc.room} – ${inc.type}<br>Guest: ${inc.guestName}`;
        document.getElementById('acceptBtn').classList.remove('hidden');
        document.getElementById('arriveBtn').classList.add('hidden');
        document.getElementById('resolveBtn').classList.add('hidden');
      }
      updateAll();
      addLog(`📟 ${staffMember.name} dispatched to ${inc.id}`);
    }
  }

  function sendCmdChat() {
    const msg = document.getElementById('cmdChatInput').value.trim();
    if(!msg || !selectedIncidentId) return;
    const inc = incidents.find(i=>i.id===selectedIncidentId);
    if(inc) { inc.chatLog.push({from:'Command', msg}); refreshCmdChat(inc); }
    document.getElementById('cmdChatInput').value='';
  }

  function refreshChats(inc) {
    document.getElementById('guestChat').innerHTML = inc.chatLog.map(m=>`<div><b>${m.from}:</b> ${m.msg} ${m.audio ? `<audio controls src="${m.audio}"></audio>` : ''}</div>`).join('');
    if(inc.id===selectedIncidentId) refreshCmdChat(inc);
  }
  function refreshCmdChat(inc) {
    document.getElementById('cmdChat').innerHTML = inc.chatLog.map(m=>`<div><b>${m.from}:</b> ${m.msg} ${m.audio ? `<audio controls src="${m.audio}"></audio>` : ''}</div>`).join('');
  }

  function addLog(msg) {
    const log = document.getElementById('eventLog');
    const div = document.createElement('div');
    div.textContent = `[${new Date().toLocaleTimeString()}] ${msg}`;
    log.prepend(div);
  }

  function updateAllMaps() { updateCommandMap(); updateStaffMap(); updatePoliceMap(); updateGuestMiniMap(); }
  function updateAll() {
    renderCommand();
    updateStaffMap();
    updatePoliceMap();
    if(currentPin) {
      const inc = incidents.find(i=>i.pin===currentPin && i.status!=='resolved');
      if(inc) {
        document.getElementById('statusText').textContent = `Status: ${inc.status} – ${inc.assignedStaff||'Awaiting dispatch'}`;
        document.getElementById('guestChat').innerHTML = inc.chatLog.map(m=>`<div><b>${m.from}:</b> ${m.msg} ${m.audio ? `<audio controls src="${m.audio}"></audio>` : ''}</div>`).join('');
      }
    }
  }
</script>
</body>
</html>
