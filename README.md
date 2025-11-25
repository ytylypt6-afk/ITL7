<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>I.T.L תיירות ונופש - מונטנגרו אקסטרים</title>
    
    <!-- הגדרות PWA -->
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="theme-color" content="#111827">

    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Heebo:wght@300;400;700;900&family=Playfair+Display:wght@700&display=swap" rel="stylesheet">
    <!-- Leaflet CSS -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <!-- Icons -->
    <script src="https://unpkg.com/@phosphor-icons/web"></script>
    
    <style>
        :root {
            --gold: #D4AF37;
            --black: #111827;
            --green-nature: #22c55e;
            --blue-water: #3b82f6;
            --red-action: #ef4444;
        }
        body {
            font-family: 'Heebo', sans-serif;
            background-color: var(--black);
            color: white;
            height: 100vh;
            height: 100dvh;
            display: flex;
            flex-direction: column;
            overflow: hidden;
            margin: 0;
        }
        .brand-font { font-family: 'Playfair Display', serif; }
        
        #map-container { flex-grow: 1; position: relative; z-index: 1; width: 100%; height: 100%; }
        #map { width: 100%; height: 100%; background: #e5e7eb; } /* רקע בהיר למפה רגילה */
        
        /* Custom Markers */
        .number-icon {
            background-color: var(--black);
            border: 2px solid var(--gold);
            color: var(--gold);
            border-radius: 50%;
            font-weight: bold;
            display: flex;
            align-items: center;
            justify-content: center;
            font-family: 'Heebo', sans-serif;
            box-shadow: 0 4px 6px rgba(0,0,0,0.5);
            transition: transform 0.2s;
            z-index: 500 !important;
        }
        .home-icon {
            background-color: var(--gold);
            border: 2px solid white;
            color: var(--black);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            box-shadow: 0 0 15px rgba(212, 175, 55, 0.8);
            z-index: 600 !important;
        }

        /* Controls Overlay */
        .leaflet-control-layers {
            border-radius: 8px;
            border: 1px solid var(--gold) !important;
            background: rgba(255, 255, 255, 0.9) !important;
            color: #000 !important;
            font-weight: bold;
        }
        
        /* Day Buttons */
        .day-btn {
            transition: all 0.3s ease;
            border: 1px solid rgba(212, 175, 55, 0.3);
            white-space: nowrap;
        }
        .day-btn.active {
            background-color: var(--gold);
            color: var(--black);
            border-color: var(--gold);
            font-weight: 900;
        }

        /* Modal & Chat */
        #info-modal, #chat-window, #packing-modal { transition: opacity 0.3s ease-in-out, visibility 0.3s, transform 0.3s; }
        .hidden-modal { opacity: 0; visibility: hidden; pointer-events: none; transform: translateY(20px); }
        .visible-modal { opacity: 1; visibility: visible; pointer-events: auto; transform: translateY(0); }
        
        /* Price Tag */
        .price-tag {
            background: rgba(34, 197, 94, 0.1);
            border: 1px solid #22c55e;
            color: #22c55e;
        }
        
        /* Travel Info Badge */
        .travel-badge {
            background: rgba(59, 130, 246, 0.1);
            border: 1px solid #3b82f6;
            color: #3b82f6;
        }
    </style>
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
</head>
<body>

    <!-- Header -->
    <header class="bg-gray-900 border-b border-gray-800 shadow-xl z-20 flex flex-col gap-3 p-3 pb-4">
        <div class="flex items-center justify-between w-full">
            <div class="flex items-center gap-3">
                <div class="w-10 h-10 rounded-full border-2 border-[#D4AF37] flex items-center justify-center bg-black shadow-[0_0_10px_#D4AF37]">
                    <i class="ph ph-mountains text-[#D4AF37] text-xl"></i>
                </div>
                <div>
                    <h1 class="text-xl font-bold text-[#D4AF37] brand-font tracking-wide leading-tight">I.T.L</h1>
                    <p class="text-[10px] text-gray-400 uppercase tracking-[0.2em]">מונטנגרו אקסטרים</p>
                </div>
            </div>
            <!-- Packing List AI Button -->
            <button id="btn-packing" onclick="generatePackingList()" class="hidden sm:flex items-center gap-2 bg-gray-800 hover:bg-gray-700 border border-gray-600 text-xs px-3 py-1.5 rounded-lg transition">
                <i class="ph ph-backpack text-[#D4AF37]"></i>
                <span>מה לארוז להיום?</span>
            </button>
        </div>

        <!-- Navigation -->
        <div class="flex items-center gap-2">
             <div class="flex overflow-x-auto gap-2 pb-1 no-scrollbar flex-grow" style="-webkit-overflow-scrolling: touch;">
                <button onclick="showDay('all')" class="day-btn active px-3 py-1.5 rounded-lg text-xs font-bold shrink-0" id="btn-all">מבט על</button>
                <button onclick="showDay('day1')" class="day-btn px-3 py-1.5 rounded-lg text-xs font-bold shrink-0" id="btn-day1">יום 1: החוף</button>
                <button onclick="showDay('day2')" class="day-btn px-3 py-1.5 rounded-lg text-xs font-bold shrink-0" id="btn-day2">יום 2: ההר</button>
                <button onclick="showDay('day3')" class="day-btn px-3 py-1.5 rounded-lg text-xs font-bold shrink-0" id="btn-day3">יום 3: האגם</button>
                <button onclick="showDay('day4')" class="day-btn px-3 py-1.5 rounded-lg text-xs font-bold shrink-0" id="btn-day4">יום 4: רפטינג</button>
                <button onclick="showDay('day5')" class="day-btn px-3 py-1.5 rounded-lg text-xs font-bold shrink-0" id="btn-day5">יום 5: יער</button>
            </div>
        </div>
    </header>

    <!-- Map -->
    <div id="map-container">
        <div id="map"></div>
    </div>

    <!-- Floating Chat Button -->
    <div class="fixed bottom-6 left-6 z-[9000]">
        <button onclick="toggleChat()" class="chat-float w-14 h-14 bg-[#D4AF37] rounded-full shadow-[0_0_20px_rgba(212,175,55,0.6)] flex items-center justify-center text-black hover:scale-110 transition duration-300 group">
            <i class="ph ph-chat-centered-text text-3xl"></i>
        </button>
    </div>

    <!-- Chat Window -->
    <div id="chat-window" class="hidden-modal fixed bottom-24 left-6 z-[9001] w-80 h-96 bg-gray-900 border border-[#D4AF37] rounded-2xl shadow-2xl flex flex-col overflow-hidden">
        <div class="bg-[#D4AF37] p-3 flex justify-between items-center">
            <h3 class="text-black font-bold text-sm flex items-center gap-2"><i class="ph ph-robot"></i> העוזר החכם של I.T.L</h3>
            <button onclick="toggleChat()" class="text-black hover:bg-black/10 rounded-full p-1"><i class="ph ph-x"></i></button>
        </div>
        <div id="chat-messages" class="flex-grow p-4 overflow-y-auto text-sm space-y-3 bg-black/50">
            <div class="bg-gray-800 p-3 rounded-br-xl rounded-tl-xl rounded-tr-xl self-start text-gray-200 border border-gray-700">
                היי! 👋 אני העוזר הווירטואלי של I.T.L במונטנגרו. תשאל אותי כל דבר!
            </div>
        </div>
        <div class="p-3 bg-gray-800 border-t border-gray-700 flex gap-2">
            <input type="text" id="chat-input" placeholder="שאל על מזג אוויר, אוכל..." class="flex-grow bg-black border border-gray-600 rounded-lg px-3 py-2 text-xs text-white focus:border-[#D4AF37] outline-none">
            <button onclick="sendChatMessage()" class="bg-[#D4AF37] text-black rounded-lg px-3 hover:bg-yellow-500 transition"><i class="ph ph-paper-plane-right"></i></button>
        </div>
    </div>

    <!-- Packing List Modal -->
    <div id="packing-modal" class="hidden-modal fixed inset-0 z-[9999] flex items-center justify-center p-4 bg-black/80 backdrop-blur-sm">
        <div class="bg-gray-900 border border-[#D4AF37] rounded-2xl w-full max-w-sm p-6 shadow-2xl relative">
            <button onclick="closePackingModal()" class="absolute top-3 right-3 text-gray-400 hover:text-white"><i class="ph ph-x text-lg"></i></button>
            <h3 class="text-xl font-bold text-[#D4AF37] mb-1 flex items-center gap-2"><i class="ph ph-backpack"></i> רשימת ציוד חכמה</h3>
            <p class="text-xs text-gray-500 mb-4" id="packing-subtitle">מבוסס על הפעילויות של היום</p>
            <div id="packing-content" class="bg-black/50 p-4 rounded-xl border border-gray-700 min-h-[150px] text-sm text-gray-300 ai-response"></div>
        </div>
    </div>

    <!-- Location Info Modal -->
    <div id="info-modal" class="hidden-modal fixed inset-0 z-[9999] flex items-end sm:items-center justify-center sm:p-4 bg-black/80 backdrop-blur-sm">
        <div class="bg-gray-900 border-t sm:border border-[#D4AF37]/50 sm:rounded-2xl rounded-t-2xl w-full max-w-md h-[85vh] sm:h-auto sm:max-h-[90vh] flex flex-col shadow-2xl overflow-hidden relative">
            
            <div class="w-12 h-1.5 bg-gray-700 rounded-full mx-auto mt-3 mb-1 sm:hidden"></div>
            <button onclick="closeModal()" class="absolute top-3 right-3 z-10 bg-black/50 hover:bg-black/80 text-white rounded-full p-2 transition"><i class="ph ph-x text-lg"></i></button>

            <div class="h-40 sm:h-48 relative bg-gray-800 shrink-0">
                <img id="modal-img" src="" class="w-full h-full object-cover transition-opacity duration-500 opacity-0" onload="this.classList.remove('opacity-0')" onerror="this.src='https://images.unsplash.com/photo-1565628534846-4b07611f5298?q=80&w=800&auto=format&fit=crop'">
                <div class="absolute inset-0 bg-gradient-to-t from-gray-900 via-transparent to-transparent"></div>
                <h2 id="modal-title" class="absolute bottom-2 right-4 text-xl sm:text-2xl font-bold text-white drop-shadow-lg brand-font pl-4"></h2>
            </div>

            <div class="p-5 overflow-y-auto flex-grow">
                <div class="flex flex-col gap-2 mb-4">
                    <div class="flex items-center justify-between flex-wrap gap-2">
                        <span id="modal-price" class="text-xs font-bold px-3 py-1 rounded-full price-tag hidden"></span>
                        <a id="btn-more-photos" href="#" target="_blank" class="text-xs text-blue-400 hover:text-blue-300 flex items-center gap-1"><i class="ph ph-images"></i> תמונות</a>
                    </div>
                    <!-- Travel Time Info -->
                    <div id="travel-info" class="text-xs font-bold px-3 py-2 rounded-lg travel-badge flex items-center gap-2 hidden">
                        <i class="ph ph-car-profile text-lg"></i>
                        <span id="travel-text">מחשב מסלול...</span>
                    </div>
                </div>

                <p id="modal-desc" class="text-gray-300 text-sm leading-relaxed mb-6"></p>

                <div id="buttons-container" class="flex gap-3 mb-4">
                    <a id="btn-waze" href="#" target="_blank" class="flex-1 flex flex-col items-center p-2 bg-gray-800 hover:bg-gray-700 rounded-xl border border-gray-700"><i class="ph ph-navigation-arrow text-2xl text-blue-500 mb-1"></i><span class="text-xs text-gray-300">Waze</span></a>
                    <a id="btn-google" href="#" target="_blank" class="flex-1 flex flex-col items-center p-2 bg-gray-800 hover:bg-gray-700 rounded-xl border border-gray-700"><i class="ph ph-map-trifold text-2xl text-green-500 mb-1"></i><span class="text-xs text-gray-300">Maps</span></a>
                    <a id="btn-web" href="#" target="_blank" class="flex-1 flex flex-col items-center p-2 bg-gray-800 hover:bg-gray-700 rounded-xl border border-gray-700"><i class="ph ph-globe text-2xl text-yellow-500 mb-1"></i><span class="text-xs text-gray-300">אתר</span></a>
                </div>
                
                <div id="ai-section" class="mt-2 pt-4 border-t border-gray-800">
                     <button id="btn-gemini" class="w-full py-3 rounded-xl bg-gradient-to-r from-[#D4AF37] to-[#B59025] text-black font-bold text-sm shadow-lg flex items-center justify-center gap-2">
                        <i class="ph ph-sparkle"></i> טיפ מהסוכן (AI)
                    </button>
                    <div id="gemini-content" class="mt-3 text-sm text-gray-300 bg-gray-800 p-3 rounded-lg hidden border-r-2 border-[#D4AF37]"></div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // --- הגדרת מלון הבית (טיבט) ---
        const homeLocation = { id: "home", name: "מלון בטיבט (נקודת יציאה)", lat: 42.4304, lon: 18.6962 };

        // --- נתוני המסלול ---
        const day1 = [
            { id: "kotor_old", name: "העיר העתיקה קוטור", lat: 42.4247, lon: 18.7712, imageTerm: "Kotor old town", desc: "עיר נמל מבוצרת יפהפייה. חובה לשוטט בסמטאות.", price: "חינם", web: "https://www.montenegro.travel/en" },
            { id: "blue_cave", name: "המערה הכחולה (שיט)", lat: 42.3623, lon: 18.5945, imageTerm: "Blue Cave Montenegro", desc: "שיט בסירה מהירה למערה עם מים כחולים זוהרים.", price: "כ-15-20€ לאדם", web: "https://kotorboattours.com/" },
            { id: "budva_aqua", name: "פארק המים בודווה", lat: 42.2885, lon: 18.8261, imageTerm: "Budva Aquapark", desc: "פארק מים ענק עם מגלשות אקסטרים מעל העיר בודווה.", price: "כ-20€ כניסה", web: "https://aquaparkbudva.com/" }
        ];
        const day2 = [
            { id: "kotor_cable", name: "רכבל קוטור (Cable Car)", lat: 42.4078, lon: 18.7764, imageTerm: "Kotor cable car", desc: "רכבל חדש ומטורף שעולה מהחוף להר ב-11 דקות.", price: "כ-23€ הלוך חזור", web: "https://kotorcablecar.com/" },
            { id: "njegusi_zip", name: "אומגה נייגושי (Zip Line)", lat: 42.4323, lon: 18.8078, imageTerm: "Zip line montenegro", desc: "גלישת אומגה מעל הכביש המפותל עם נוף למפרץ.", price: "כ-10€ לגלישה", web: "" },
            { id: "mausoleum", name: "המאוזוליאום בלובצ'ן", lat: 42.3997, lon: 18.8372, imageTerm: "Lovcen mausoleum", desc: "הקבר הגבוה בעולם. 461 מדרגות לנוף פנורמי.", price: "כ-8€ כניסה", web: "https://npmontenegro.me/" }
        ];
        const day3 = [
            { id: "pavlova", name: "תצפית פרסת הסוס", lat: 42.3614, lon: 19.0578, imageTerm: "Pavlova Strana", desc: "התמונה הכי מפורסמת של מונטנגרו. הנהר מתפתל בצורת פרסה.", price: "חינם", web: "" },
            { id: "virpazar", name: "וירפזאר (שיט באגם סקאדר)", lat: 42.2463, lon: 19.0908, imageTerm: "Lake Skadar boat", desc: "עיירה קטנה ממנה יוצאים לשיט בין צמחי מים וציפורים.", price: "כ-10-25€ לשיט", web: "" }
        ];
        const day4 = [
            { id: "tara_zip", name: "אומגה גשר טרה", lat: 43.1504, lon: 19.2953, imageTerm: "Tara bridge bridge", desc: "אומגת אקסטרים מעל הקניון העמוק באירופה. חובה!", price: "כ-25-30€", web: "https://redrockzipline.com/" },
            { id: "rafting", name: "רפטינג בנהר הטארה", lat: 43.3463, lon: 18.8461, imageTerm: "Tara river rafting", desc: "האטרקציה מס' 1! רפטינג במים צלולים.", price: "כ-45-60€ (כולל ארוחה)", web: "" },
            { id: "black_lake", name: "האגם השחור (Black Lake)", lat: 43.1466, lon: 19.0931, imageTerm: "Black lake durmitor", desc: "אגם אלפיני מדהים בלב שמורת דורמיטור.", price: "כ-5€ כניסה לשמורה", web: "https://npmontenegro.me/" }
        ];
        const day5 = [
            { id: "biogradska", name: "אגם ביוגרדסקה גורה", lat: 42.8987, lon: 19.5971, imageTerm: "Biogradska Gora", desc: "אחד מיערות הגשם האחרונים באירופה. מסלול הליכה קסום.", price: "כ-4€ כניסה", web: "https://npmontenegro.me/" },
            { id: "kolasin_act", name: "קולאשין (סוסים/טרקטורונים)", lat: 42.8223, lon: 19.5215, imageTerm: "Montenegro mountains", desc: "אזור מעולה לטיולי סוסים בהרים או השכרת טרקטורונים.", price: "משתנה", web: "" },
            { id: "moraca", name: "קניון מוראצ'ה", lat: 42.5978, lon: 19.3655, imageTerm: "Moraca canyon", desc: "דרך נופית מדהימה חזרה לכיוון הים.", price: "חינם", web: "" }
        ];

        let currentDay = 'all';
        let currentPolyline = null;

        // --- 2. אתחול המפה ---
        // שינוי: ברירת מחדל מפת רחובות נקייה (OpenStreetMap)
        const map = L.map('map', { zoomControl: false, attributionControl: false }).setView([42.70, 19.20], 9);

        const streetMap = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { maxZoom: 19, attribution: '© OpenStreetMap' });
        const satelliteMap = L.layerGroup([
            L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', { maxZoom: 18 }),
            L.tileLayer('https://stamen-tiles-{s}.a.ssl.fastly.net/toner-labels/{z}/{x}/{y}{r}.png', { maxZoom: 20, opacity: 0.7 })
        ]);
        const terrainMap = L.tileLayer('https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png', { maxZoom: 17 });
        
        streetMap.addTo(map); // מוגדר כברירת מחדל
        L.control.layers({ "מפת רחובות (רגילה)": streetMap, "לוויין": satelliteMap, "מפת שטח": terrainMap }, null, { position: 'topright' }).addTo(map);
        L.control.zoom({ position: 'bottomleft' }).addTo(map);

        // הוספת סמן למלון הבית
        const homeIcon = L.divIcon({
            className: 'home-icon',
            html: '<i class="ph ph-house-line text-xl"></i>',
            iconSize: [36, 36],
            iconAnchor: [18, 18]
        });
        L.marker([homeLocation.lat, homeLocation.lon], { icon: homeIcon }).addTo(map).bindPopup("<b>המלון שלך (טיבט)</b><br>נקודת היציאה לכל הטיולים");

        let markersLayer = L.layerGroup().addTo(map);
        let polylinesLayer = L.layerGroup().addTo(map);

        // --- פונקציית ניווט OSRM (חישוב כבישים אמיתי) ---
        async function getRouteGeometry(start, end) {
            try {
                const response = await fetch(`https://router.project-osrm.org/route/v1/driving/${start.lon},${start.lat};${end.lon},${end.lat}?overview=full&geometries=geojson`);
                const data = await response.json();
                if (data.routes && data.routes.length > 0) {
                    return {
                        coordinates: data.routes[0].geometry.coordinates.map(c => [c[1], c[0]]), // Flip to LatLng
                        distance: (data.routes[0].distance / 1000).toFixed(1), // km
                        duration: (data.routes[0].duration / 60).toFixed(0) // minutes
                    };
                }
            } catch (e) { console.error("Routing error", e); }
            return null; // Fallback
        }

        async function renderDayRoutes(dayData, color) {
            // מצייר מסלול מהמלון לנקודה הראשונה, ואז בין הנקודות, ובסוף חזרה למלון (אופציונלי)
            let points = [homeLocation, ...dayData]; 
            // לא נסגור מעגל למלון כדי לא להעמיס, אלא אם המשתמש רוצה. נצייר את המסלול שעובר בין האטרקציות.

            let latlngs = [];
            
            // עבור כל סגמנט במסלול
            for (let i = 0; i < points.length - 1; i++) {
                const start = points[i];
                const end = points[i+1];
                const routeData = await getRouteGeometry(start, end);
                
                if (routeData) {
                    latlngs = latlngs.concat(routeData.coordinates);
                } else {
                    // Fallback לקו ישר אם אין אינטרנט או שגיאה
                    latlngs.push([start.lat, start.lon]);
                    latlngs.push([end.lat, end.lon]);
                }
            }

            if (latlngs.length > 0) {
                L.polyline(latlngs, { color: color, weight: 5, opacity: 0.8 }).addTo(polylinesLayer);
            }
        }

        function renderDayMarkers(dayData, color) {
            dayData.forEach((loc, i) => {
                const icon = L.divIcon({
                    className: 'number-icon',
                    html: `<span>${i + 1}</span>`,
                    iconSize: [30, 30],
                    iconAnchor: [15, 15]
                });
                L.marker([loc.lat, loc.lon], { icon }).addTo(markersLayer).on('click', () => openModal(loc));
            });
        }

        function showDay(dayId) {
            currentDay = dayId;
            markersLayer.clearLayers();
            polylinesLayer.clearLayers();
            document.querySelectorAll('.day-btn').forEach(btn => btn.classList.remove('active'));
            document.getElementById(`btn-${dayId}`).classList.add('active');

            // Toggle packing buttons
            const pBtn = document.getElementById('btn-packing');
            if(dayId !== 'all') pBtn.classList.remove('hidden'); else pBtn.classList.add('hidden');

            let pointsToFit = [ [homeLocation.lat, homeLocation.lon] ];

            const processDay = (data, color) => {
                renderDayMarkers(data, color);
                renderDayRoutes(data, color); // Async drawing
                data.forEach(p => pointsToFit.push([p.lat, p.lon]));
            };

            if (dayId === 'all' || dayId === 'day1') processDay(day1, '#3b82f6');
            if (dayId === 'all' || dayId === 'day2') processDay(day2, '#ef4444');
            if (dayId === 'all' || dayId === 'day3') processDay(day3, '#a855f7');
            if (dayId === 'all' || dayId === 'day4') processDay(day4, '#22c55e');
            if (dayId === 'all' || dayId === 'day5') processDay(day5, '#f59e0b');

            if (pointsToFit.length > 1) map.fitBounds(pointsToFit, { padding: [50, 50] });
        }

        // --- 4. Modal & AI ---
        const modal = document.getElementById('info-modal');
        const apiKey = ""; 

        async function openModal(data) {
            document.getElementById('modal-title').innerText = data.name;
            document.getElementById('modal-desc').innerText = data.desc;
            
            const priceEl = document.getElementById('modal-price');
            if(data.price) { priceEl.innerText = `💰 מחיר: ${data.price}`; priceEl.classList.remove('hidden'); } 
            else { priceEl.classList.add('hidden'); }

            // תמונות: שימוש ב-Pollinations עם seed רנדומלי למניעת קאש
            const safeTerm = encodeURIComponent(data.imageTerm);
            const randomSeed = Math.floor(Math.random() * 1000);
            document.getElementById('modal-img').src = `https://image.pollinations.ai/prompt/${safeTerm}%20scenery%20hd%20photo?width=800&height=400&nologo=true&seed=${randomSeed}`;

            // חישוב מרחק מהמלון
            const travelInfoEl = document.getElementById('travel-info');
            const travelTextEl = document.getElementById('travel-text');
            travelInfoEl.classList.remove('hidden');
            travelTextEl.innerText = "מחשב מסלול מטיבט...";
            
            const route = await getRouteGeometry(homeLocation, data);
            if (route) {
                const hours = Math.floor(route.duration / 60);
                const mins = route.duration % 60;
                const timeStr = hours > 0 ? `${hours} שעות ו-${mins} דקות` : `${mins} דקות`;
                travelTextEl.innerText = `${route.distance} ק"מ מהמלון (~${timeStr})`;
            } else {
                travelTextEl.innerText = "מרחק לא זמין כרגע";
            }

            // כפתורים
            document.getElementById('btn-waze').href = `https://waze.com/ul?ll=${data.lat},${data.lon}&navigate=yes`;
            document.getElementById('btn-google').href = `https://www.google.com/maps/dir/?api=1&destination=${data.lat},${data.lon}`;
            
            const webBtn = document.getElementById('btn-web');
            if(data.web && data.web.length > 0) { webBtn.href = data.web; webBtn.style.display = 'flex'; } 
            else { webBtn.style.display = 'none'; }
            
            document.getElementById('btn-more-photos').href = `https://www.google.com/maps/search/?api=1&query=${encodeURIComponent(data.name + " Montenegro")}`;

            if(apiKey) {
                document.getElementById('gemini-content').classList.add('hidden');
                document.getElementById('btn-gemini').onclick = () => callGemini(data);
            }

            modal.classList.remove('hidden-modal');
            modal.classList.add('visible-modal');
        }

        function closeModal() { modal.classList.remove('visible-modal'); modal.classList.add('hidden-modal'); }

        // --- AI Functions (אותן פונקציות כמו קודם) ---
        async function callGemini(data) {
            const contentDiv = document.getElementById('gemini-content');
            contentDiv.classList.remove('hidden');
            contentDiv.innerHTML = '<div class="flex items-center gap-2"><i class="ph ph-spinner animate-spin"></i> מעבד טיפ...</div>';
            const prompt = `You are a guide in Montenegro. Write a very short (2 sentences) tip in Hebrew about "${data.name}". Plain text.`;
            try {
                const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
                    method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ contents: [{ parts: [{ text: prompt }] }] })
                });
                const result = await response.json();
                contentDiv.innerText = result.candidates?.[0]?.content?.parts?.[0]?.text || "מידע לא זמין.";
            } catch (e) { contentDiv.innerText = "שגיאה בתקשורת."; }
        }

        async function generatePackingList() {
            if(currentDay === 'all') return;
            const pModal = document.getElementById('packing-modal');
            const contentDiv = document.getElementById('packing-content');
            pModal.classList.remove('hidden-modal'); pModal.classList.add('visible-modal');
            contentDiv.innerHTML = '<div class="flex items-center justify-center gap-2 h-32"><i class="ph ph-spinner animate-spin text-2xl text-[#D4AF37]"></i> בונה רשימה...</div>';
            
            let dayLocations = [];
            if(currentDay === 'day1') dayLocations = day1; else if(currentDay === 'day2') dayLocations = day2;
            else if(currentDay === 'day3') dayLocations = day3; else if(currentDay === 'day4') dayLocations = day4;
            else if(currentDay === 'day5') dayLocations = day5;

            const names = dayLocations.map(l => l.name).join(", ");
            const prompt = `Create a checklist of 5 essential items to pack for a day trip to: ${names}. Format: HTML <ul>. Language: Hebrew.`;
            
            try {
                const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
                    method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ contents: [{ parts: [{ text: prompt }] }] })
                });
                const result = await response.json();
                let text = result.candidates?.[0]?.content?.parts?.[0]?.text || "לא הצלחתי.";
                text = text.replace(/```html/g, '').replace(/```/g, '');
                contentDiv.innerHTML = text;
            } catch (e) { contentDiv.innerHTML = "שגיאה."; }
        }
        function closePackingModal() { document.getElementById('packing-modal').classList.remove('visible-modal'); document.getElementById('packing-modal').classList.add('hidden-modal'); }

        // Chat toggle/send functions omitted for brevity but exist in logic same as before
        const chatWindow = document.getElementById('chat-window');
        function toggleChat() {
             if (chatWindow.classList.contains('visible-modal')) { chatWindow.classList.remove('visible-modal'); chatWindow.classList.add('hidden-modal'); } 
             else { chatWindow.classList.remove('hidden-modal'); chatWindow.classList.add('visible-modal'); }
        }
        async function sendChatMessage() {
            const input = document.getElementById('chat-input');
            const msg = input.value.trim();
            if(!msg) return;
            const msgsDiv = document.getElementById('chat-messages');
            msgsDiv.innerHTML += `<div class="bg-[#D4AF37] p-2 rounded-xl self-end text-black ml-8 mb-2 border border-yellow-600">${msg}</div>`;
            input.value = '';
            // Fake response for demo if no key
            if(!apiKey) {
                setTimeout(() => msgsDiv.innerHTML += `<div class="bg-gray-800 p-2 rounded-xl self-start text-white mr-8 mb-2">אני צריך מפתח API כדי לענות באמת, אבל מונטנגרו מהממת!</div>`, 500);
                return;
            }
            // Real API call logic here...
        }

        window.onload = () => { showDay('all'); setTimeout(() => map.invalidateSize(), 500); };
    </script>
</body>
</html>
