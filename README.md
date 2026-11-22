<!DOCTYPE html> lang="th" class="h-full dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="ระบบวิเคราะห์ความน่าอยู่และความปลอดภัย โรงเรียนบุญเหลือวิทยานุสรณ์ ต.โคกสูง อ.เมืองนครราชสีมา จ.นครราชสีมา ประเมินจุดเสี่ยง สภาพอากาศ ค่าฝุ่น PM2.5 สด และจุดบริการฉุกเฉินรอบโรงเรียน">
    <title>GeoSafety & Livability | โรงเรียนบุญเหลือวิทยานุสรณ์</title>

    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>

    <!-- Leaflet Map CSS & JS (Cloudflare CDN) -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.min.css" crossorigin="anonymous" referrerpolicy="no-referrer" />
    <script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.min.js" crossorigin="anonymous" referrerpolicy="no-referrer"></script>

    <!-- Lucide Icons (Cloudflare CDN) -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/lucide/0.468.0/umd/lucide.min.js" crossorigin="anonymous" referrerpolicy="no-referrer"></script>

    <!-- QRCode.js Generator Library (Cloudflare CDN) -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js" crossorigin="anonymous" referrerpolicy="no-referrer"></script>

    <!-- Google Fonts: Kanit & Inter -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;600;700&family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">

    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Kanit', 'Inter', 'sans-serif'],
                    },
                    colors: {
                        brand: {
                            50: '#f0f9ff',
                            500: '#0284c7',
                            600: '#0369a1',
                            700: '#075985',
                        }
                    }
                }
            }
        };

        // Global theme toggle (Light / Dark Mode) while preserving Satellite view
        window.toggleTheme = function () {
            const html = document.documentElement;
            const isDark = html.classList.toggle('dark');
            const iconSun = document.getElementById('iconSun');
            const iconMoon = document.getElementById('iconMoon');
            const themeText = document.getElementById('themeText');

            if (iconSun) iconSun.classList.toggle('hidden', !isDark);
            if (iconMoon) iconMoon.classList.toggle('hidden', isDark);
            if (themeText) themeText.textContent = isDark ? 'โหมดสว่าง' : 'โหมดมืด';

            if (typeof window.setMapLayer === 'function' && window.currentMapMode !== 'satellite') {
                window.setMapLayer(isDark ? 'dark' : 'street', true);
            }
        };
    </script>

    <style>
        /* Custom Scrollbar */
        ::-webkit-scrollbar {
            width: 8px;
            height: 8px;
        }
        .dark ::-webkit-scrollbar-track {
            background: #0f172a;
        }
        .dark ::-webkit-scrollbar-thumb {
            background: #334155;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f5f9;
        }
        ::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #94a3b8;
        }

        /* Map container & full-screen styling */
        #map {
            width: 100%;
            height: 100%;
            z-index: 10;
        }
        .map-fullscreen-active {
            position: fixed !important;
            inset: 0 !important;
            width: 100vw !important;
            height: 100vh !important;
            z-index: 9990 !important;
            border-radius: 0 !important;
            margin: 0 !important;
        }
        .leaflet-control-attribution {
            font-size: 10px !important;
            opacity: 0.75;
        }
        @keyframes pulseRing {
            0% { transform: scale(0.85); opacity: 0.85; }
            70% { transform: scale(2.1); opacity: 0; }
            100% { transform: scale(2.1); opacity: 0; }
        }
        .gps-pulse-ring {
            animation: pulseRing 2s cubic-bezier(0.24, 0, 0.38, 1) infinite;
        }
    </style>
</head>
<body class="h-full flex flex-col font-sans antialiased bg-slate-50 text-slate-800 dark:bg-slate-900 dark:text-slate-100 transition-colors duration-300 selection:bg-sky-500 selection:text-white">

    <!-- Top Navigation Bar -->
    <header class="bg-white/85 dark:bg-slate-800/80 backdrop-blur-md border-b border-slate-200 dark:border-slate-700/60 sticky top-0 z-50 transition-colors duration-300">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 h-16 flex items-center justify-between gap-2">
            <div class="flex items-center space-x-3 min-w-0">
                <div class="p-2 bg-gradient-to-tr from-sky-500 to-indigo-600 rounded-xl shadow-lg shadow-sky-500/20 shrink-0">
                    <i data-lucide="shield-check" class="w-6 h-6 text-white"></i>
                </div>
                <div class="min-w-0">
                    <div class="flex items-center gap-2">
                        <h1 class="font-bold text-base sm:text-lg leading-tight bg-gradient-to-r from-slate-900 via-sky-700 to-indigo-600 dark:from-white dark:via-slate-200 dark:to-sky-400 bg-clip-text text-transparent truncate">
                            GeoSafety • รร.บุญเหลือวิทยานุสรณ์
                        </h1>
                        <!-- Real-Time LIVE Badge -->
                        <span class="inline-flex items-center gap-1 px-2 py-0.5 rounded-full text-[10px] font-bold bg-emerald-500/15 text-emerald-700 dark:text-emerald-400 border border-emerald-500/30 shrink-0">
                            <span class="w-1.5 h-1.5 rounded-full bg-emerald-500 animate-ping"></span>
                            LIVE
                        </span>
                    </div>
                    <p class="text-xs text-slate-500 dark:text-slate-400 truncate">
                        ต.โคกสูง อ.เมืองนครราชสีมา จ.นครราชสีมา • <span id="liveClockText" class="font-mono">--:--:--</span>
                    </p>
                </div>
            </div>

            <!-- Header Quick Actions -->
            <div class="flex items-center gap-1.5 sm:gap-2 shrink-0">
                <!-- QR Code Generator Button -->
                <button type="button" onclick="openQrCodeModal()" title="สร้างและดาวน์โหลด QR Code ของเว็บไซต์" class="inline-flex items-center gap-1.5 px-2.5 sm:px-3 py-1.5 text-xs sm:text-sm font-medium bg-sky-600 hover:bg-sky-500 text-white rounded-lg transition-all shadow-sm active:scale-95">
                    <i data-lucide="qr-code" class="w-4 h-4"></i>
                    <span class="hidden sm:inline">QR Code เว็บ</span>
                </button>

                <!-- Share School Zone Link Button -->
                <button type="button" onclick="shareCurrentArea()" title="คัดลอกลิงก์แชร์จุดตรวจสอบนี้" class="inline-flex items-center gap-1.5 px-2.5 sm:px-3 py-1.5 text-xs sm:text-sm font-medium bg-slate-100 hover:bg-slate-200 dark:bg-slate-700 dark:hover:bg-slate-600 text-slate-700 dark:text-slate-200 rounded-lg transition-all border border-slate-300/70 dark:border-slate-600/50 active:scale-95">
                    <i data-lucide="share-2" class="w-4 h-4 text-indigo-500 dark:text-indigo-400"></i>
                    <span class="hidden md:inline">แชร์จุดตรวจสอบ</span>
                </button>

                <!-- Theme Toggle Button (Light / Dark Mode) -->
                <button type="button" onclick="toggleTheme()" id="themeToggleBtn" title="สลับโหมดสว่าง / โหมดมืด" class="inline-flex items-center gap-1.5 px-2.5 sm:px-3 py-1.5 text-xs sm:text-sm font-medium bg-slate-100 hover:bg-slate-200 dark:bg-slate-700 dark:hover:bg-slate-600 text-slate-700 dark:text-slate-200 rounded-lg transition-all border border-slate-300/70 dark:border-slate-600/50 active:scale-95">
                    <i data-lucide="sun" id="iconSun" class="w-4 h-4 text-amber-500 hidden"></i>
                    <i data-lucide="moon" id="iconMoon" class="w-4 h-4 text-sky-400"></i>
                    <span id="themeText" class="hidden md:inline">โหมดสว่าง</span>
                </button>

                <!-- Live GPS Tracking Toggle -->
                <button type="button" onclick="toggleLiveGpsTracking()" id="liveTrackBtn" title="ติดตามตำแหน่งสดขณะเดินในโรงเรียน" class="inline-flex items-center gap-1.5 px-2.5 sm:px-3 py-1.5 text-xs sm:text-sm font-medium bg-slate-100 hover:bg-slate-200 dark:bg-slate-700 dark:hover:bg-slate-600 text-slate-700 dark:text-slate-200 rounded-lg transition-all border border-slate-300/70 dark:border-slate-600/50 active:scale-95">
                    <span id="liveTrackDot" class="w-2 h-2 rounded-full bg-slate-400"></span>
                    <span id="liveTrackLabel" class="hidden sm:inline">ติดตามสด</span>
                </button>

                <!-- Fast 1-Click GPS Location Button -->
                <button type="button" onclick="useCurrentLocation()" id="gpsHeaderBtn" class="inline-flex items-center gap-1.5 sm:gap-2 px-3 py-1.5 text-xs sm:text-sm font-medium bg-sky-50 hover:bg-sky-100 dark:bg-slate-700 dark:hover:bg-slate-600 text-sky-700 dark:text-slate-200 rounded-lg transition-all border border-sky-200 dark:border-slate-600/50 active:scale-95">
                    <span id="gpsPermStatusDot" class="w-2 h-2 rounded-full bg-amber-400" title="สถานะสิทธิ์ตำแหน่ง"></span>
                    <i data-lucide="crosshair" id="gpsHeaderIcon" class="w-4 h-4 text-sky-600 dark:text-sky-400"></i>
                    <span class="hidden sm:inline">ตำแหน่งปัจจุบัน</span>
                </button>
            </div>
        </div>
    </header>

    <!-- Main Container -->
    <main class="flex-1 max-w-7xl w-full mx-auto px-4 sm:px-6 lg:px-8 py-6 space-y-6">

        <!-- One-Click Location Permission / In-App Browser Helper Banner -->
        <div id="quickPermissionBanner" class="hidden bg-gradient-to-r from-sky-500/10 via-indigo-500/10 to-sky-500/10 border border-sky-300/60 dark:border-sky-500/30 rounded-2xl p-3.5 sm:px-5 flex flex-col sm:flex-row items-start sm:items-center justify-between gap-3">
            <div class="flex items-start sm:items-center gap-3">
                <div class="p-2 rounded-xl bg-sky-500/15 text-sky-600 dark:text-sky-400 shrink-0">
                    <i data-lucide="navigation" class="w-5 h-5"></i>
                </div>
                <div class="text-xs sm:text-sm">
                    <div class="flex flex-wrap items-center gap-2">
                        <p id="permBannerTitle" class="font-semibold text-slate-900 dark:text-white">เปิดใช้งาน GPS เพื่อระบุจุดที่คุณอยู่ภายในโรงเรียนบุญเหลือวิทยานุสรณ์</p>
                        <span id="inAppBadge" class="hidden px-2 py-0.5 rounded-full text-[10px] font-bold bg-amber-500/15 text-amber-700 dark:text-amber-300 border border-amber-500/30">
                            In-App Browser
                        </span>
                    </div>
                    <p id="permBannerDesc" class="text-slate-600 dark:text-slate-300 text-xs mt-0.5">กดยืนยันสิทธิ์ 1 ครั้งเพื่อปักหมุดพิกัดจริงของคุณและประเมินความปลอดภัยรอบตัวแบบเรียลไทม์</p>
                </div>
            </div>
            <div class="flex flex-wrap items-center gap-2 shrink-0 self-end sm:self-center">
                <button type="button" id="inAppCopyBtn" onclick="copyAppUrlForExternalBrowser()" class="hidden px-3.5 py-2 text-xs sm:text-sm font-medium bg-amber-500 hover:bg-amber-400 text-slate-950 rounded-xl shadow-sm transition active:scale-95 flex items-center gap-1.5">
                    <i data-lucide="copy" class="w-3.5 h-3.5"></i>
                    <span>คัดลอกลิงก์เปิดใน Chrome / Safari</span>
                </button>
                <button type="button" onclick="requestQuickLocationPermission()" class="px-4 py-2 text-xs sm:text-sm font-medium bg-sky-600 hover:bg-sky-500 text-white rounded-xl shadow-sm transition active:scale-95 flex items-center gap-1.5">
                    <i data-lucide="locate-fixed" class="w-4 h-4"></i>
                    <span>ยืนยันตำแหน่งทันที</span>
                </button>
            </div>
        </div>

        <!-- Campus Zone Selection & Local Search Section -->
        <section class="bg-white dark:bg-slate-800/50 border border-slate-200 dark:border-slate-700/50 rounded-2xl p-4 sm:p-6 shadow-sm dark:shadow-xl backdrop-blur-sm transition-colors duration-300 relative z-30">
            <div class="max-w-4xl mx-auto text-center space-y-4">
                <div class="inline-flex items-center gap-1.5 px-3 py-1 rounded-full bg-sky-500/10 text-sky-700 dark:text-sky-300 border border-sky-500/20 text-xs font-medium">
                    <i data-lucide="graduation-cap" class="w-3.5 h-3.5"></i>
                    <span>โรงเรียนบุญเหลือวิทยานุสรณ์ • เขตปลอดภัยรอบสถานศึกษา 300 เมตร</span>
                </div>
                <h2 class="text-xl sm:text-2xl font-semibold text-slate-900 dark:text-slate-100">ตรวจสอบความปลอดภัยและสภาพแวดล้อมภายในโรงเรียน</h2>
                <p class="text-slate-500 dark:text-slate-400 text-sm">เลือกจุดภายในโรงเรียนบุญเหลือวิทยานุสรณ์ ค้นหาอาคาร/ประตู/จุดรับ-ส่ง หรือคลิกบนแผนที่ดาวเทียมเพื่อประเมินทันที</p>

                <!-- School Zone Selector & Local Search Input -->
                <div class="grid grid-cols-1 sm:grid-cols-3 gap-3 text-left">
                    <!-- 7 School Zones Dropdown -->
                    <div class="relative">
                        <select id="schoolZoneSelect" onchange="selectZoneByDropdown()" aria-label="เลือกโซนภายในโรงเรียนบุญเหลือวิทยานุสรณ์" class="w-full h-full px-4 py-3.5 bg-slate-50 dark:bg-slate-900/90 border border-slate-300 dark:border-slate-700 rounded-xl text-slate-800 dark:text-slate-100 focus:outline-none focus:ring-2 focus:ring-sky-500 text-sm appearance-none cursor-pointer transition-colors pr-9">
                            <option value="">-- เลือกจุดภายในโรงเรียน --</option>
                        </select>
                        <i data-lucide="chevron-down" class="w-4 h-4 text-slate-500 dark:text-slate-400 absolute right-3.5 top-1/2 -translate-y-1/2 pointer-events-none"></i>
                    </div>

                    <!-- School Campus Autocomplete Search Input -->
                    <div class="relative flex items-center sm:col-span-2">
                        <i data-lucide="map-pin" class="absolute left-4 w-5 h-5 text-slate-400 pointer-events-none"></i>
                        <input type="text" id="searchInput" oninput="handleSearchInput(this.value)" onkeypress="if(event.key === 'Enter') performSearch()" autocomplete="off"
                               placeholder="ค้นหาจุดในโรงเรียน เช่น ประตูหน้า, อาคารเรียน, สนามฟุตบอล, โรงอาหาร, ทางม้าลาย..." 
                               class="w-full pl-12 pr-28 py-3.5 bg-slate-50 dark:bg-slate-900/90 border border-slate-300 dark:border-slate-700 rounded-xl text-slate-800 dark:text-slate-100 placeholder-slate-400 dark:placeholder-slate-500 focus:outline-none focus:ring-2 focus:ring-sky-500 focus:border-transparent transition-all text-sm">
                        <button type="button" id="clearSearchBtn" onclick="clearSearchInput()" title="ล้างคำค้นหา" class="hidden absolute right-20 p-1.5 text-slate-400 hover:text-slate-600 dark:hover:text-slate-200 rounded-lg">
                            <i data-lucide="x" class="w-4 h-4"></i>
                        </button>
                        <button type="button" id="searchBtn" onclick="performSearch()" class="absolute right-2 px-4 py-2 bg-sky-600 hover:bg-sky-500 text-white font-medium text-sm rounded-lg transition-all shadow-md active:scale-95 flex items-center gap-1.5">
                            <span>ค้นหา</span>
                        </button>

                        <!-- Autocomplete Suggestions Dropdown -->
                        <div id="autocompleteDropdown" class="hidden absolute left-0 right-0 top-full mt-1.5 bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-700 rounded-xl shadow-2xl overflow-hidden z-50 max-h-72 overflow-y-auto divide-y divide-slate-100 dark:divide-slate-800">
                        </div>
                    </div>
                </div>

                <!-- Quick Campus Preset Chips -->
                <div class="flex flex-wrap items-center justify-center gap-2 pt-1 text-xs">
                    <span class="text-slate-500 dark:text-slate-400">จุดสำคัญในโรงเรียน:</span>
                    <button type="button" onclick="loadZoneData('overview')" class="px-3 py-1 bg-slate-100 hover:bg-slate-200 dark:bg-slate-700/60 dark:hover:bg-slate-700 text-slate-700 dark:text-slate-300 rounded-full border border-slate-200 dark:border-slate-600/40 transition">ศูนย์กลางโรงเรียน</button>
                    <button type="button" onclick="loadZoneData('front_gate')" class="px-3 py-1 bg-slate-100 hover:bg-slate-200 dark:bg-slate-700/60 dark:hover:bg-slate-700 text-slate-700 dark:text-slate-300 rounded-full border border-slate-200 dark:border-slate-600/40 transition">ประตูหน้า & จุดรับ-ส่ง</button>
                    <button type="button" onclick="loadZoneData('main_buildings')" class="px-3 py-1 bg-slate-100 hover:bg-slate-200 dark:bg-slate-700/60 dark:hover:bg-slate-700 text-slate-700 dark:text-slate-300 rounded-full border border-slate-200 dark:border-slate-600/40 transition">อาคารเรียน & ห้องสมุด</button>
                    <button type="button" onclick="loadZoneData('sports_field')" class="px-3 py-1 bg-slate-100 hover:bg-slate-200 dark:bg-slate-700/60 dark:hover:bg-slate-700 text-slate-700 dark:text-slate-300 rounded-full border border-slate-200 dark:border-slate-600/40 transition">สนามกีฬา & หน้าเสาธง</button>
                    <button type="button" onclick="loadZoneData('canteen')" class="px-3 py-1 bg-slate-100 hover:bg-slate-200 dark:bg-slate-700/60 dark:hover:bg-slate-700 text-slate-700 dark:text-slate-300 rounded-full border border-slate-200 dark:border-slate-600/40 transition">โรงอาหาร</button>
                    <button type="button" onclick="loadZoneData('auditorium')" class="px-3 py-1 bg-slate-100 hover:bg-slate-200 dark:bg-slate-700/60 dark:hover:bg-slate-700 text-slate-700 dark:text-slate-300 rounded-full border border-slate-200 dark:border-slate-600/40 transition">หอประชุม</button>
                    <button type="button" onclick="loadZoneData('crosswalk_zone')" class="px-3 py-1 bg-slate-100 hover:bg-slate-200 dark:bg-slate-700/60 dark:hover:bg-slate-700 text-slate-700 dark:text-slate-300 rounded-full border border-slate-200 dark:border-slate-600/40 transition">ทางม้าลายหน้าโรงเรียน</button>
                </div>
            </div>
        </section>

        <!-- Emergency Hotlines & School Incident Coordinate Dispatch Strip -->
        <section class="bg-white dark:bg-slate-800/60 border border-slate-200 dark:border-slate-700/50 rounded-2xl px-4 py-3 shadow-sm flex flex-col lg:flex-row items-start lg:items-center justify-between gap-3">
            <div class="flex flex-wrap items-center gap-2 text-xs">
                <span class="inline-flex items-center gap-1.5 font-semibold text-rose-600 dark:text-rose-400 mr-1">
                    <i data-lucide="phone-call" class="w-4 h-4"></i>
                    <span>สายด่วนฉุกเฉิน:</span>
                </span>
                <a href="tel:191" class="px-2.5 py-1 rounded-lg bg-rose-500/10 hover:bg-rose-500/20 text-rose-700 dark:text-rose-300 border border-rose-500/30 font-semibold transition">
                    191 เหตุด่วนตำรวจ
                </a>
                <a href="tel:1669" class="px-2.5 py-1 rounded-lg bg-emerald-500/10 hover:bg-emerald-500/20 text-emerald-700 dark:text-emerald-300 border border-emerald-500/30 font-semibold transition">
                    1669 เจ็บป่วยฉุกเฉิน
                </a>
                <a href="tel:199" class="px-2.5 py-1 rounded-lg bg-amber-500/10 hover:bg-amber-500/20 text-amber-700 dark:text-amber-300 border border-amber-500/30 font-semibold transition">
                    199 ดับเพลิง/กู้ภัย
                </a>
                <a href="tel:1784" class="px-2.5 py-1 rounded-lg bg-sky-500/10 hover:bg-sky-500/20 text-sky-700 dark:text-sky-300 border border-sky-500/30 font-semibold transition">
                    1784 ปภ. ภัยพิบัติ
                </a>
            </div>

            <div class="flex flex-wrap items-center gap-2 text-xs w-full lg:w-auto justify-end">
                <button type="button" onclick="copyEmergencyCoordinates()" class="inline-flex items-center gap-1.5 px-3 py-1.5 rounded-xl bg-rose-600 hover:bg-rose-500 text-white font-medium shadow-sm transition active:scale-95">
                    <i data-lucide="siren" class="w-3.5 h-3.5"></i>
                    <span>คัดลอกพิกัดจุดเกิดเหตุในโรงเรียน</span>
                </button>
                <button type="button" onclick="copyAreaSummaryReport()" class="inline-flex items-center gap-1.5 px-3 py-1.5 rounded-xl bg-slate-100 hover:bg-slate-200 dark:bg-slate-700 dark:hover:bg-slate-600 text-slate-700 dark:text-slate-200 font-medium border border-slate-300/60 dark:border-slate-600/50 transition active:scale-95">
                    <i data-lucide="clipboard-check" class="w-3.5 h-3.5 text-sky-500"></i>
                    <span>คัดลอกรายงานสรุป</span>
                </button>
            </div>
        </section>

        <!-- Real-Time Satellite Map Card (Zoom 17 & 300m School Safety Zone) -->
        <section id="mapCardSection" class="bg-white dark:bg-slate-800/70 border border-slate-200 dark:border-slate-700/60 rounded-2xl overflow-hidden shadow-sm dark:shadow-lg transition-colors duration-300">
            <!-- Map Toolbar (School POI Chips + Live Radar + Reset + Google Maps Links) -->
            <div class="px-4 py-2.5 border-b border-slate-200 dark:border-slate-700/60 flex flex-wrap items-center justify-between gap-2 bg-slate-50/70 dark:bg-slate-900/40">
                <!-- School-Zone POI Filter Chips -->
                <div class="flex flex-wrap items-center gap-1.5 text-xs">
                    <span class="text-slate-500 dark:text-slate-400 font-medium mr-1 hidden sm:inline">จุดสำคัญรอบโรงเรียน:</span>
                    <button type="button" id="chip-firstaid" onclick="togglePOICategory('firstaid')" class="px-2.5 py-1 rounded-full bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-slate-700 dark:text-slate-300 hover:border-rose-400 transition flex items-center gap-1">
                        <span class="w-2 h-2 rounded-full bg-rose-500"></span>
                        <span>จุดปฐมพยาบาล/รพ.</span>
                    </button>
                    <button type="button" id="chip-police" onclick="togglePOICategory('police')" class="px-2.5 py-1 rounded-full bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-slate-700 dark:text-slate-300 hover:border-blue-400 transition flex items-center gap-1">
                        <span class="w-2 h-2 rounded-full bg-blue-500"></span>
                        <span>ป้อมตำรวจโคกสูง</span>
                    </button>
                    <button type="button" id="chip-green" onclick="togglePOICategory('green')" class="px-2.5 py-1 rounded-full bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-slate-700 dark:text-slate-300 hover:border-emerald-400 transition flex items-center gap-1">
                        <span class="w-2 h-2 rounded-full bg-emerald-500"></span>
                        <span>พื้นที่สีเขียว/สนามกีฬา</span>
                    </button>
                    <button type="button" id="chip-transit" onclick="togglePOICategory('transit')" class="px-2.5 py-1 rounded-full bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-slate-700 dark:text-slate-300 hover:border-amber-400 transition flex items-center gap-1">
                        <span class="w-2 h-2 rounded-full bg-amber-500"></span>
                        <span>จุดรับ-ส่ง/ป้ายรถโดยสาร</span>
                    </button>
                </div>

                <!-- Live Radar + Quick Reset + External Google Maps & Street View Links -->
                <div class="flex flex-wrap items-center gap-2 text-xs">
                    <button type="button" id="liveRadarBtn" onclick="toggleLiveRadar()" title="เปิด/ปิดชั้นข้อมูลเรดาร์กลุ่มเมฆฝนเวลาปัจจุบัน" class="inline-flex items-center gap-1.5 px-3 py-1.5 rounded-lg bg-slate-100 hover:bg-slate-200 dark:bg-slate-800 dark:hover:bg-slate-700 text-slate-700 dark:text-slate-200 border border-slate-300/80 dark:border-slate-600/70 font-medium transition active:scale-95">
                        <span id="liveRadarDot" class="w-2 h-2 rounded-full bg-slate-400"></span>
                        <i data-lucide="cloud-rain-wind" class="w-3.5 h-3.5 text-sky-500"></i>
                        <span>เรดาร์เมฆฝนสด</span>
                    </button>
                    <button type="button" onclick="resetToSchoolCenter()" class="inline-flex items-center gap-1.5 px-3 py-1.5 rounded-lg bg-indigo-500/15 hover:bg-indigo-500/25 text-indigo-700 dark:text-indigo-300 border border-indigo-400/40 font-medium transition active:scale-95">
                        <i data-lucide="school" class="w-3.5 h-3.5"></i>
                        <span>กลับสู่โรงเรียนบุญเหลือวิทยานุสรณ์</span>
                    </button>
                    <a id="streetViewLink" href="https://www.google.com/maps/@?api=1&map_action=pano&viewpoint=15.0538,102.1389" target="_blank" rel="noopener noreferrer" class="inline-flex items-center gap-1.5 px-3 py-1.5 rounded-lg bg-amber-500/15 hover:bg-amber-500/25 text-amber-800 dark:text-amber-300 border border-amber-400/40 font-medium transition">
                        <i data-lucide="person-standing" class="w-3.5 h-3.5 text-amber-600 dark:text-amber-400"></i>
                        <span>Street View</span>
                    </a>
                    <a id="googleMapsLink" href="https://www.google.com/maps/search/?api=1&query=15.0538,102.1389" target="_blank" rel="noopener noreferrer" class="inline-flex items-center gap-1.5 px-3 py-1.5 rounded-lg bg-sky-600 hover:bg-sky-500 text-white font-medium shadow-sm transition">
                        <i data-lucide="external-link" class="w-3.5 h-3.5"></i>
                        <span>Google Maps</span>
                    </a>
                </div>
            </div>

            <!-- Map Viewport -->
            <div id="mapWrapper" class="relative w-full h-80 sm:h-96 lg:h-[420px] bg-slate-200 dark:bg-slate-900">
                <div id="map"></div>

                <!-- Top-Left Real-Time Satellite Telemetry HUD -->
                <div class="absolute top-3 left-3 z-20 max-w-[calc(100%-6.5rem)] sm:max-w-sm bg-slate-950/85 text-white border border-sky-400/40 rounded-xl px-3 py-2 shadow-lg backdrop-blur-md text-[11px] space-y-1">
                    <div class="flex flex-wrap items-center gap-1.5">
                        <span class="inline-flex items-center gap-1 px-1.5 py-0.5 rounded bg-emerald-500/20 text-emerald-300 border border-emerald-400/30 font-bold text-[10px] tracking-wide">
                            <span class="w-1.5 h-1.5 rounded-full bg-emerald-400 animate-ping"></span>
                            SATELLITE LIVE
                        </span>
                        <span class="font-semibold text-sky-200 truncate">ภาพถ่ายดาวเทียมเวลาปัจจุบัน</span>
                    </div>
                    <div class="flex flex-wrap items-center gap-x-2 gap-y-0.5 text-[10px] text-slate-300">
                        <span id="satLiveClock" class="font-mono text-white font-medium">--/--/---- --:--:-- น.</span>
                        <span>•</span>
                        <span id="satDayNightBadge" class="inline-flex items-center gap-1 text-amber-300 font-medium">กำลังตรวจสอบแสง...</span>
                    </div>
                    <div class="flex flex-wrap items-center gap-x-2 gap-y-0.5 text-[10px] text-slate-400">
                        <span>พิกัด: <span id="satLiveCoords" class="font-mono text-slate-200">15.05380, 102.13890</span></span>
                        <span>•</span>
                        <span class="text-sky-300">เขตปลอดภัย 300 ม.</span>
                        <span id="satRadarTimeTag" class="hidden text-emerald-300 font-medium">• เรดาร์สดทำงาน</span>
                    </div>
                </div>

                <!-- Top-Right Fullscreen & GPS Recalibrate Pill -->
                <div class="absolute top-3 right-3 z-20 flex items-center gap-1.5">
                    <div id="liveGpsTelemetryStrip" class="hidden items-center gap-1.5 px-2.5 py-1.5 rounded-xl bg-white/95 dark:bg-slate-900/95 border border-slate-200/90 dark:border-slate-700/80 shadow-md backdrop-blur-md text-[11px]">
                        <span class="w-2 h-2 rounded-full bg-blue-500 animate-ping"></span>
                        <span id="gpsSignalQualityBadge" class="font-semibold text-emerald-600 dark:text-emerald-400">±-- ม.</span>
                        <span id="liveTelemetrySpeed" class="font-mono text-slate-500 dark:text-slate-400">0 km/h</span>
                        <button type="button" onclick="recalibrateGpsNow()" class="px-1.5 py-0.5 rounded bg-sky-500/15 text-sky-700 dark:text-sky-300 font-medium hover:bg-sky-500/25 transition">
                            ปรับเทียบ
                        </button>
                    </div>
                    <button type="button" onclick="toggleMapFullscreen()" id="fullscreenMapBtn" title="ขยายแผนที่เต็มจอ" class="p-2 bg-white/95 dark:bg-slate-900/95 hover:bg-slate-50 dark:hover:bg-slate-800 text-slate-700 dark:text-slate-200 border border-slate-200/90 dark:border-slate-700/80 rounded-xl shadow-md backdrop-blur-md transition active:scale-95">
                        <i data-lucide="maximize-2" id="fullscreenIcon" class="w-4 h-4"></i>
                    </button>
                </div>

                <!-- Bottom-Left Map Layer Switcher Pills (Street / Satellite / Dark) -->
                <div class="absolute bottom-3 left-3 z-20 inline-flex items-center gap-1 p-1 bg-white/95 dark:bg-slate-900/95 backdrop-blur-md border border-slate-200/90 dark:border-slate-700/80 rounded-xl shadow-md text-xs font-medium">
                    <button type="button" id="layer-street" onclick="setMapLayer('street')" class="px-2.5 py-1 rounded-lg transition-colors text-slate-600 dark:text-slate-300 hover:bg-slate-100 dark:hover:bg-slate-800 flex items-center gap-1">
                        <i data-lucide="map" class="w-3.5 h-3.5"></i>
                        <span>ถนน</span>
                    </button>
                    <button type="button" id="layer-satellite" onclick="setMapLayer('satellite')" class="px-2.5 py-1 rounded-lg transition-colors bg-sky-600 text-white flex items-center gap-1">
                        <i data-lucide="globe" class="w-3.5 h-3.5"></i>
                        <span>ดาวเทียม</span>
                    </button>
                    <button type="button" id="layer-dark" onclick="setMapLayer('dark')" class="px-2.5 py-1 rounded-lg transition-colors text-slate-600 dark:text-slate-300 hover:bg-slate-100 dark:hover:bg-slate-800 flex items-center gap-1">
                        <i data-lucide="moon" class="w-3.5 h-3.5"></i>
                        <span>โหมดมืด</span>
                    </button>
                </div>

                <!-- Floating Google Maps & Street View Paired Pill on Map Card -->
                <div class="hidden sm:inline-flex items-center gap-1.5 p-1 bg-white/95 dark:bg-slate-900/95 backdrop-blur-md border border-slate-200/90 dark:border-slate-700/80 rounded-xl shadow-md text-xs font-medium absolute bottom-3 left-1/2 -translate-x-1/2 z-20">
                    <a id="mapOverlayStreetView" href="https://www.google.com/maps/@?api=1&map_action=pano&viewpoint=15.0538,102.1389" target="_blank" rel="noopener noreferrer" class="inline-flex items-center gap-1.5 px-2.5 py-1 rounded-lg bg-amber-500/15 hover:bg-amber-500/25 text-amber-800 dark:text-amber-300 transition">
                        <i data-lucide="person-standing" class="w-3.5 h-3.5 text-amber-600 dark:text-amber-400"></i>
                        <span>มุมมองถนนจริง (Street View)</span>
                    </a>
                    <span class="text-slate-300 dark:text-slate-700">|</span>
                    <a id="mapOverlayGoogleMaps" href="https://www.google.com/maps/search/?api=1&query=15.0538,102.1389" target="_blank" rel="noopener noreferrer" class="inline-flex items-center gap-1.5 px-2.5 py-1 rounded-lg bg-sky-600 hover:bg-sky-500 text-white transition">
                        <i data-lucide="map-pinned" class="w-3.5 h-3.5"></i>
                        <span>เปิดใน Google Maps</span>
                    </a>
                </div>

                <!-- Bottom-Right Zoom +/- Controls -->
                <div class="absolute bottom-3 right-3 z-20 flex flex-col bg-white/95 dark:bg-slate-900/95 backdrop-blur-md border border-slate-200/90 dark:border-slate-700/80 rounded-xl shadow-md overflow-hidden">
                    <button type="button" onclick="if(map) map.zoomIn()" title="ซูมเข้า (+)" class="p-2 text-slate-700 dark:text-slate-200 hover:bg-slate-100 dark:hover:bg-slate-800 border-b border-slate-200/80 dark:border-slate-700/80 transition active:scale-95">
                        <i data-lucide="plus" class="w-4 h-4"></i>
                    </button>
                    <button type="button" onclick="if(map) map.zoomOut()" title="ซูมออก (-)" class="p-2 text-slate-700 dark:text-slate-200 hover:bg-slate-100 dark:hover:bg-slate-800 transition active:scale-95">
                        <i data-lucide="minus" class="w-4 h-4"></i>
                    </button>
                </div>
            </div>

            <!-- Subtle Map Hint Strip -->
            <div class="px-4 py-2 bg-slate-50 dark:bg-slate-900/60 border-t border-slate-200/80 dark:border-slate-700/50 flex flex-wrap items-center justify-between gap-2 text-[11px] text-slate-500 dark:text-slate-400">
                <span class="flex items-center gap-1.5">
                    <i data-lucide="mouse-pointer-click" class="w-3.5 h-3.5 text-sky-500"></i>
                    <span>คลิกจุดใดก็ได้บนแผนที่ดาวเทียม หรือลากหมุดสีแดงเพื่อระบุตำแหน่งอาคาร/จุดเกิดเหตุ</span>
                </span>
                <span id="mapCardPm" class="font-medium text-slate-700 dark:text-slate-300">PM2.5: กำลังโหลด... • --°C</span>
            </div>
        </section>

        <!-- Dynamic Results Dashboard -->
        <div id="dashboard" class="space-y-6">

            <!-- Location Header & Main Score Summary -->
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">

                <!-- Overall Score Card -->
                <div class="bg-white dark:bg-slate-800/80 border border-slate-200 dark:border-slate-700/60 rounded-2xl p-6 flex flex-col justify-between relative overflow-hidden shadow-sm dark:shadow-lg transition-colors duration-300">
                    <div class="absolute -right-10 -bottom-10 w-40 h-40 bg-sky-500/10 rounded-full blur-3xl pointer-events-none"></div>
                    <div>
                        <div class="flex justify-between items-start mb-4 gap-2">
                            <div>
                                <span class="text-xs font-semibold tracking-wider uppercase text-sky-600 dark:text-sky-400">คะแนนความปลอดภัย & ความน่าอยู่</span>
                                <h3 id="locTitle" class="text-xl sm:text-2xl font-bold text-slate-900 dark:text-white mt-1">ภาพรวมโรงเรียนบุญเหลือวิทยานุสรณ์ (จุดศูนย์กลาง)</h3>
                                <p id="locSub" class="text-xs text-slate-500 dark:text-slate-400 mt-1">ต.โคกสูง อ.เมืองนครราชสีมา จ.นครราชสีมา • พิกัด: 15.0538, 102.1389</p>
                            </div>
                            <span id="riskBadge" class="px-3 py-1 rounded-full text-xs font-bold bg-emerald-500/15 text-emerald-700 dark:text-emerald-400 border border-emerald-500/30 whitespace-nowrap">
                                เสี่ยงต่ำ
                            </span>
                        </div>

                        <!-- Big Rating Display -->
                        <div class="flex items-baseline space-x-3 my-6">
                            <span id="overallScore" class="text-5xl font-extrabold tracking-tight text-slate-900 dark:text-white">88</span>
                            <span class="text-slate-400 text-lg">/ 100</span>
                            <span id="livabilityGrade" class="ml-auto text-xs sm:text-sm font-medium px-3 py-1 bg-slate-100 dark:bg-slate-700/70 text-slate-700 dark:text-slate-200 rounded-lg">
                                น่าเรียนและปลอดภัยดีเยี่ยม (A+)
                            </span>
                        </div>
                    </div>

                    <!-- Mini Summary Indicators -->
                    <div class="grid grid-cols-3 gap-2 pt-4 border-t border-slate-200 dark:border-slate-700/50 text-center">
                        <div class="p-2 bg-slate-50 dark:bg-slate-900/40 rounded-xl border border-slate-100 dark:border-transparent">
                            <p class="text-[11px] text-slate-500 dark:text-slate-400">ความปลอดภัย</p>
                            <p id="quickSafety" class="text-xs sm:text-sm font-semibold text-emerald-600 dark:text-emerald-400">ดีมาก (90%)</p>
                        </div>
                        <div class="p-2 bg-slate-50 dark:bg-slate-900/40 rounded-xl border border-slate-100 dark:border-transparent">
                            <p class="text-[11px] text-slate-500 dark:text-slate-400">สภาพแวดล้อม</p>
                            <p id="quickEnv" class="text-xs sm:text-sm font-semibold text-emerald-600 dark:text-emerald-400">ดีมาก (86%)</p>
                        </div>
                        <div class="p-2 bg-slate-50 dark:bg-slate-900/40 rounded-xl border border-slate-100 dark:border-transparent">
                            <p class="text-[11px] text-slate-500 dark:text-slate-400">การเข้าถึง</p>
                            <p id="quickConvenience" class="text-xs sm:text-sm font-semibold text-sky-600 dark:text-sky-400">ดีเยี่ยม (92%)</p>
                        </div>
                    </div>
                </div>

                <!-- Quick Alert & School Zone Highlights Card -->
                <div class="lg:col-span-2 bg-white dark:bg-slate-800/80 border border-slate-200 dark:border-slate-700/60 rounded-2xl p-6 shadow-sm dark:shadow-lg flex flex-col justify-between transition-colors duration-300">
                    <div>
                        <h4 class="text-sm font-semibold text-slate-700 dark:text-slate-300 uppercase tracking-wider mb-3 flex items-center gap-2">
                            <i data-lucide="bell" class="w-4 h-4 text-amber-500 dark:text-amber-400"></i>
                            ข้อควรระวัง & คำแนะนำความปลอดภัยสำหรับนักเรียนและผู้ปกครอง
                        </h4>

                        <div id="alertBoxList" class="space-y-3">
                            <!-- Dynamic Alerts injected here -->
                        </div>
                    </div>

                    <div class="mt-4 pt-4 border-t border-slate-200 dark:border-slate-700/50 flex flex-wrap items-center justify-between text-xs text-slate-500 dark:text-slate-400 gap-2">
                        <div class="flex flex-wrap items-center gap-2">
                            <span class="flex items-center gap-1.5">
                                <i data-lucide="clock" class="w-3.5 h-3.5 text-sky-500"></i>
                                <span id="lastUpdatedText">อัปเดตข้อมูลล่าสุด: กำลังซิงค์...</span>
                            </span>
                            <span id="autoSyncCountdown" class="px-2 py-0.5 rounded-md bg-emerald-500/10 text-emerald-700 dark:text-emerald-300 font-medium">
                                ซิงค์อัตโนมัติใน 30 วิ
                            </span>
                            <button type="button" onclick="manualRefreshNow()" class="inline-flex items-center gap-1 px-2 py-0.5 rounded-md bg-slate-100 hover:bg-slate-200 dark:bg-slate-700 dark:hover:bg-slate-600 text-slate-700 dark:text-slate-200 transition active:scale-95">
                                <i data-lucide="refresh-cw" id="manualRefreshIcon" class="w-3 h-3"></i>
                                <span>อัปเดตทันที</span>
                            </button>
                        </div>
                        <span class="flex items-center gap-1.5">
                            <i data-lucide="database" class="w-3.5 h-3.5 text-emerald-500"></i>
                            <span>พิกัด รร.บุญเหลือวิทยานุสรณ์ • Open-Meteo Real-Time</span>
                        </span>
                    </div>
                </div>
            </div>

            <!-- Detailed Metrics Grid (5 School-Tailored Dimensions) -->
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">

                <!-- Dimension 1: School Safety & Road Crossing -->
                <div class="bg-white dark:bg-slate-800/60 border border-slate-200 dark:border-slate-700/50 rounded-2xl p-5 hover:border-slate-300 dark:hover:border-slate-600 transition shadow-sm dark:shadow-md">
                    <div class="flex items-center justify-between mb-4">
                        <div class="flex items-center space-x-3">
                            <div class="p-2.5 bg-indigo-50 dark:bg-indigo-500/10 text-indigo-600 dark:text-indigo-400 rounded-xl border border-indigo-200 dark:border-indigo-500/20">
                                <i data-lucide="shield-alert" class="w-5 h-5"></i>
                            </div>
                            <div>
                                <h4 class="font-semibold text-slate-900 dark:text-white">ความปลอดภัยในโรงเรียนและจุดข้ามถนน</h4>
                                <p class="text-xs text-slate-500 dark:text-slate-400">กล้อง CCTV & ทางม้าลาย ถ.สุรนารายณ์</p>
                            </div>
                        </div>
                        <span id="scoreSafety" class="text-lg font-bold text-indigo-600 dark:text-indigo-400">90/100</span>
                    </div>

                    <div class="space-y-3 text-xs sm:text-sm">
                        <div>
                            <div class="flex justify-between text-slate-600 dark:text-slate-300 mb-1">
                                <span>ความครอบคลุมกล้อง CCTV ภายในโรงเรียน</span>
                                <span id="cctvStatus" class="font-medium text-emerald-600 dark:text-emerald-400">ครอบคลุมจุดหลัก</span>
                            </div>
                            <div class="w-full bg-slate-200 dark:bg-slate-700 h-1.5 rounded-full overflow-hidden">
                                <div id="cctvBar" class="bg-emerald-500 dark:bg-emerald-400 h-full rounded-full" style="width: 90%"></div>
                            </div>
                        </div>
                        <div>
                            <div class="flex justify-between text-slate-600 dark:text-slate-300 mb-1">
                                <span>ความปลอดภัยจุดข้ามถนน ถ.สุรนารายณ์</span>
                                <span id="crossingStatus" class="font-medium text-emerald-600 dark:text-emerald-400">มีทางม้าลาย & ครูเวร</span>
                            </div>
                            <div class="w-full bg-slate-200 dark:bg-slate-700 h-1.5 rounded-full overflow-hidden">
                                <div id="crossingBar" class="bg-emerald-500 dark:bg-emerald-400 h-full rounded-full" style="width: 86%"></div>
                            </div>
                        </div>
                        <div>
                            <div class="flex justify-between text-slate-600 dark:text-slate-300 mb-1">
                                <span>แสงสว่างและเวรยามรักษาความปลอดภัย</span>
                                <span id="lightStatus" class="font-medium text-sky-600 dark:text-sky-400">พร้อมดูแลตลอดวัน</span>
                            </div>
                            <div class="w-full bg-slate-200 dark:bg-slate-700 h-1.5 rounded-full overflow-hidden">
                                <div id="lightBar" class="bg-sky-500 dark:bg-sky-400 h-full rounded-full" style="width: 88%"></div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Dimension 2: Weather, PM2.5 & Campus Drainage -->
                <div class="bg-white dark:bg-slate-800/60 border border-slate-200 dark:border-slate-700/50 rounded-2xl p-5 hover:border-slate-300 dark:hover:border-slate-600 transition shadow-sm dark:shadow-md">
                    <div class="flex items-center justify-between mb-4">
                        <div class="flex items-center space-x-3">
                            <div class="p-2.5 bg-amber-50 dark:bg-amber-500/10 text-amber-600 dark:text-amber-400 rounded-xl border border-amber-200 dark:border-amber-500/20">
                                <i data-lucide="cloud-rain" class="w-5 h-5"></i>
                            </div>
                            <div>
                                <h4 class="font-semibold text-slate-900 dark:text-white">สภาพอากาศ ฝุ่น PM2.5 และจุดระบายน้ำ</h4>
                                <p class="text-xs text-slate-500 dark:text-slate-400">ประเมินกิจกรรมกลางแจ้ง & น้ำขัง</p>
                            </div>
                        </div>
                        <span id="scoreEnv" class="text-lg font-bold text-amber-600 dark:text-amber-400">86/100</span>
                    </div>

                    <div class="space-y-3 text-xs sm:text-sm">
                        <div>
                            <div class="flex justify-between text-slate-600 dark:text-slate-300 mb-1">
                                <span>จุดระบายน้ำและน้ำขังหลังฝนตก</span>
                                <span id="floodStatus" class="font-medium text-emerald-600 dark:text-emerald-400">ระบายน้ำได้ดี</span>
                            </div>
                            <div class="w-full bg-slate-200 dark:bg-slate-700 h-1.5 rounded-full overflow-hidden">
                                <div id="floodBar" class="bg-emerald-500 dark:bg-emerald-400 h-full rounded-full" style="width: 84%"></div>
                            </div>
                        </div>
                        <div>
                            <div class="flex justify-between text-slate-600 dark:text-slate-300 mb-1">
                                <span>ดรรชนีคุณภาพอากาศ (PM2.5 สด)</span>
                                <span id="pmStatus" class="font-medium text-emerald-600 dark:text-emerald-400">กำลังวัดค่า...</span>
                            </div>
                            <div class="w-full bg-slate-200 dark:bg-slate-700 h-1.5 rounded-full overflow-hidden">
                                <div id="pmBar" class="bg-emerald-500 dark:bg-emerald-400 h-full rounded-full" style="width: 82%"></div>
                            </div>
                        </div>
                        <div>
                            <div class="flex justify-between text-slate-600 dark:text-slate-300 mb-1">
                                <span>กิจกรรมหน้าเสาธง / วิชาพละ</span>
                                <span id="outdoorActivityStatus" class="font-medium text-emerald-600 dark:text-emerald-400">ทำกิจกรรมกลางแจ้งได้ปกติ</span>
                            </div>
                            <div class="w-full bg-slate-200 dark:bg-slate-700 h-1.5 rounded-full overflow-hidden">
                                <div id="outdoorActivityBar" class="bg-emerald-500 dark:bg-emerald-400 h-full rounded-full" style="width: 90%"></div>
                            </div>
                        </div>
                        <div class="pt-1 border-t border-slate-100 dark:border-slate-700/40 flex justify-between text-[11px] text-slate-500 dark:text-slate-400">
                            <span>ระดับความสูงพื้นที่จริง (รทก.)</span>
                            <span id="elevationInfoText" class="font-semibold text-slate-700 dark:text-slate-200">กำลังวัดระดับความสูง...</span>
                        </div>
                    </div>
                </div>

                <!-- Dimension 3: First Aid, Medical & Police Response -->
                <div class="bg-white dark:bg-slate-800/60 border border-slate-200 dark:border-slate-700/50 rounded-2xl p-5 hover:border-slate-300 dark:hover:border-slate-600 transition shadow-sm dark:shadow-md">
                    <div class="flex items-center justify-between mb-4">
                        <div class="flex items-center space-x-3">
                            <div class="p-2.5 bg-rose-50 dark:bg-rose-500/10 text-rose-600 dark:text-rose-400 rounded-xl border border-rose-200 dark:border-rose-500/20">
                                <i data-lucide="hospital" class="w-5 h-5"></i>
                            </div>
                            <div>
                                <h4 class="font-semibold text-slate-900 dark:text-white">ห้องพยาบาลและหน่วยกู้ชีพใกล้โรงเรียน</h4>
                                <p class="text-xs text-slate-500 dark:text-slate-400">ห้องพยาบาล, รพ.สต.โคกสูง & ป้อมตำรวจ</p>
                            </div>
                        </div>
                        <span id="scoreInfra" class="text-lg font-bold text-rose-600 dark:text-rose-400">92/100</span>
                    </div>

                    <div class="space-y-3 text-xs sm:text-sm">
                        <div>
                            <div class="flex justify-between text-slate-600 dark:text-slate-300 mb-1">
                                <span>ห้องพยาบาลโรงเรียน / รพ.สต.โคกสูง</span>
                                <span id="hospitalDist" class="font-medium text-emerald-600 dark:text-emerald-400">ห้องพยาบาลใน รร. / รพ.สต. 1.2 กม.</span>
                            </div>
                            <div class="w-full bg-slate-200 dark:bg-slate-700 h-1.5 rounded-full overflow-hidden">
                                <div id="hospitalBar" class="bg-emerald-500 dark:bg-emerald-400 h-full rounded-full" style="width: 92%"></div>
                            </div>
                        </div>
                        <div>
                            <div class="flex justify-between text-slate-600 dark:text-slate-300 mb-1">
                                <span>ป้อมตำรวจโคกสูง / สภ.จอหอ</span>
                                <span id="policeDist" class="font-medium text-emerald-600 dark:text-emerald-400">สายตรวจโคกสูง (~850 ม.)</span>
                            </div>
                            <div class="w-full bg-slate-200 dark:bg-slate-700 h-1.5 rounded-full overflow-hidden">
                                <div id="policeBar" class="bg-emerald-500 dark:bg-emerald-400 h-full rounded-full" style="width: 90%"></div>
                            </div>
                        </div>
                        <div>
                            <div class="flex justify-between text-slate-600 dark:text-slate-300 mb-1">
                                <span>ระบบไฟฟ้า น้ำดื่มสะอาด และถังดับเพลิง</span>
                                <span class="font-medium text-emerald-600 dark:text-emerald-400">พร้อมใช้งานครบจุด</span>
                            </div>
                            <div class="w-full bg-slate-200 dark:bg-slate-700 h-1.5 rounded-full overflow-hidden">
                                <div class="bg-emerald-500 dark:bg-emerald-400 h-full rounded-full" style="width: 95%"></div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Dimension 4: School Drop-off, Transit & Rush Hour Traffic -->
                <div class="bg-white dark:bg-slate-800/60 border border-slate-200 dark:border-slate-700/50 rounded-2xl p-5 hover:border-slate-300 dark:hover:border-slate-600 transition shadow-sm dark:shadow-md">
                    <div class="flex items-center justify-between mb-4">
                        <div class="flex items-center space-x-3">
                            <div class="p-2.5 bg-sky-50 dark:bg-sky-500/10 text-sky-600 dark:text-sky-400 rounded-xl border border-sky-200 dark:border-sky-500/20">
                                <i data-lucide="bus" class="w-5 h-5"></i>
                            </div>
                            <div>
                                <h4 class="font-semibold text-slate-900 dark:text-white">การเดินทาง จุดรับ-ส่ง และรถโดยสาร</h4>
                                <p class="text-xs text-slate-500 dark:text-slate-400">ถ.สุรนารายณ์ ช่วงเช้า-เย็น</p>
                            </div>
                        </div>
                        <span id="scoreTransit" class="text-lg font-bold text-sky-600 dark:text-sky-400">85/100</span>
                    </div>

                    <div class="space-y-3 text-xs sm:text-sm">
                        <div>
                            <div class="flex justify-between text-slate-600 dark:text-slate-300 mb-1">
                                <span>จุดจอดรับ-ส่งนักเรียน & รถโดยสาร</span>
                                <span id="transitAccess" class="font-medium text-emerald-600 dark:text-emerald-400">ติด ถ.สุรนารายณ์ เดินทางสะดวก</span>
                            </div>
                            <div class="w-full bg-slate-200 dark:bg-slate-700 h-1.5 rounded-full overflow-hidden">
                                <div id="transitBar" class="bg-emerald-500 dark:bg-emerald-400 h-full rounded-full" style="width: 88%"></div>
                            </div>
                        </div>
                        <div>
                            <div class="flex justify-between text-slate-600 dark:text-slate-300 mb-1">
                                <span>จราจรช่วงเร่งด่วน (07:00-08:15 / 15:30-17:00)</span>
                                <span id="trafficStatus" class="font-medium text-amber-600 dark:text-amber-400">คล่องตัวปานกลาง</span>
                            </div>
                            <div class="w-full bg-slate-200 dark:bg-slate-700 h-1.5 rounded-full overflow-hidden">
                                <div id="trafficBar" class="bg-amber-500 dark:bg-amber-400 h-full rounded-full" style="width: 75%"></div>
                            </div>
                        </div>
                        <div>
                            <div class="flex justify-between text-slate-600 dark:text-slate-300 mb-1">
                                <span>ทางเดินเท้าและทางม้าลายหน้าโรงเรียน</span>
                                <span id="walkStatus" class="font-medium text-sky-600 dark:text-sky-400">มีทางเท้าและป้ายเขตโรงเรียน</span>
                            </div>
                            <div class="w-full bg-slate-200 dark:bg-slate-700 h-1.5 rounded-full overflow-hidden">
                                <div id="walkBar" class="bg-sky-500 dark:bg-sky-400 h-full rounded-full" style="width: 84%"></div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Dimension 5: Campus Green Spaces, Sports & Live Weather (2-Column Wide Card) -->
                <div class="bg-white dark:bg-slate-800/60 border border-slate-200 dark:border-slate-700/50 rounded-2xl p-5 hover:border-slate-300 dark:hover:border-slate-600 transition shadow-sm dark:shadow-md md:col-span-2 lg:col-span-2">
                    <div class="flex items-center justify-between mb-4">
                        <div class="flex items-center space-x-3">
                            <div class="p-2.5 bg-emerald-50 dark:bg-emerald-500/10 text-emerald-600 dark:text-emerald-400 rounded-xl border border-emerald-200 dark:border-emerald-500/20">
                                <i data-lucide="trees" class="w-5 h-5"></i>
                            </div>
                            <div>
                                <h4 class="font-semibold text-slate-900 dark:text-white">พื้นที่สีเขียว สนามกีฬา & สภาพอากาศสดในโรงเรียน</h4>
                                <p class="text-xs text-slate-500 dark:text-slate-400">สวนหย่อม ลานกิจกรรม และสภาพอากาศในโรงเรียนบุญเหลือวิทยานุสรณ์</p>
                            </div>
                        </div>
                        <span id="scoreGreen" class="text-lg font-bold text-emerald-600 dark:text-emerald-400">89/100</span>
                    </div>

                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-4 text-xs sm:text-sm">
                        <div class="bg-slate-50 dark:bg-slate-900/50 p-3.5 rounded-xl border border-slate-200/80 dark:border-slate-700/30">
                            <div class="flex items-center gap-2 mb-1.5 text-slate-700 dark:text-slate-300 font-medium">
                                <i data-lucide="tree-pine" class="w-4 h-4 text-emerald-600 dark:text-emerald-400"></i>
                                <span>บรรยากาศและพื้นที่พักผ่อนในโซนนี้</span>
                            </div>
                            <p id="greenSpaceDesc" class="text-xs text-slate-500 dark:text-slate-400 leading-relaxed">
                                ภายในโรงเรียนบุญเหลือวิทยานุสรณ์มีแนวต้นไม้ร่มรื่น สนามฟุตบอลมาตรฐาน และลานกิจกรรมกว้างขวาง เหมาะกับการเรียนรู้และออกกำลังกาย
                            </p>
                        </div>

                        <div class="bg-slate-50 dark:bg-slate-900/50 p-3.5 rounded-xl border border-slate-200/80 dark:border-slate-700/30">
                            <div class="flex items-center gap-2 mb-1.5 text-slate-700 dark:text-slate-300 font-medium">
                                <i data-lucide="thermometer-sun" class="w-4 h-4 text-sky-600 dark:text-sky-400"></i>
                                <span>สภาพอากาศในโรงเรียนปัจจุบัน (Real-Time)</span>
                            </div>
                            <div class="flex flex-wrap items-center gap-2 text-xs text-slate-600 dark:text-slate-300 mt-1">
                                <span id="liveWeatherTemp" class="px-2 py-0.5 rounded bg-sky-500/10 text-sky-700 dark:text-sky-300 font-semibold">--.-°C</span>
                                <span id="liveWeatherRain" class="px-2 py-0.5 rounded bg-indigo-500/10 text-indigo-700 dark:text-indigo-300">กำลังวัดปริมาณฝน</span>
                                <span id="liveWeatherHumidity" class="px-2 py-0.5 rounded bg-emerald-500/10 text-emerald-700 dark:text-emerald-300">ความชื้น --%</span>
                                <span id="liveWeatherWind" class="px-2 py-0.5 rounded bg-amber-500/10 text-amber-700 dark:text-amber-300">ลม -- กม./ชม.</span>
                            </div>
                        </div>
                    </div>
                </div>

            </div>

            <!-- Community Voting, Rating & Reviews Section (Persisted via localStorage) -->
            <section id="communityFeedbackSection" class="bg-white dark:bg-slate-800/70 border border-slate-200 dark:border-slate-700/60 rounded-2xl p-5 sm:p-6 shadow-sm dark:shadow-lg transition-colors duration-300 space-y-6">
                <!-- Header & Quick Sentiment Voting -->
                <div class="flex flex-col lg:flex-row lg:items-center justify-between gap-4 pb-5 border-b border-slate-200 dark:border-slate-700/60">
                    <div>
                        <div class="flex items-center gap-2">
                            <span class="p-2 rounded-xl bg-sky-500/10 text-sky-600 dark:text-sky-400 border border-sky-500/20">
                                <i data-lucide="message-square-heart" class="w-5 h-5"></i>
                            </span>
                            <div>
                                <div class="flex flex-wrap items-center gap-2">
                                    <h3 class="text-base sm:text-lg font-bold text-slate-900 dark:text-white">
                                        โหวตให้คะแนนและความคิดเห็นจากนักเรียน ครู และผู้ปกครอง
                                    </h3>
                                    <span class="inline-flex items-center gap-1 px-2 py-0.5 rounded-full text-[10px] font-semibold bg-emerald-500/10 text-emerald-700 dark:text-emerald-300 border border-emerald-500/25">
                                        <span class="w-1.5 h-1.5 rounded-full bg-emerald-500"></span>
                                        <span>บันทึกข้อมูลอัตโนมัติ</span>
                                    </span>
                                </div>
                                <p class="text-xs text-slate-500 dark:text-slate-400">
                                    จุดตรวจสอบ: <span id="communityAreaName" class="font-medium text-sky-600 dark:text-sky-400">ภาพรวมโรงเรียนบุญเหลือวิทยานุสรณ์ (จุดศูนย์กลาง)</span>
                                </p>
                            </div>
                        </div>
                    </div>

                    <!-- Summary Rating & Quick Vote Buttons -->
                    <div class="flex flex-col sm:flex-row sm:items-center gap-3">
                        <div class="flex items-center gap-2.5 px-3.5 py-2 rounded-xl bg-amber-500/10 border border-amber-400/30">
                            <svg viewBox="0 0 24 24" class="w-5 h-5 shrink-0" fill="#f59e0b" stroke="#f59e0b" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                                <polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"></polygon>
                            </svg>
                            <div>
                                <div class="flex items-baseline gap-1">
                                    <span id="communityAvgRating" class="text-lg font-extrabold text-slate-900 dark:text-white">4.7</span>
                                    <span class="text-xs text-slate-500 dark:text-slate-400">/ 5.0</span>
                                </div>
                                <p id="communityTotalCount" class="text-[11px] text-slate-500 dark:text-slate-400">จาก 34 โหวต</p>
                            </div>
                        </div>

                        <div class="space-y-1.5">
                            <div class="flex flex-wrap items-center gap-2">
                                <button type="button" id="voteUpBtn" onclick="voteAreaSentiment('up')" class="inline-flex items-center gap-1.5 px-3.5 py-2 rounded-xl text-xs sm:text-sm font-medium bg-emerald-500/10 hover:bg-emerald-500/20 text-emerald-700 dark:text-emerald-300 border border-emerald-500/30 transition active:scale-95">
                                    <i data-lucide="thumbs-up" class="w-4 h-4"></i>
                                    <span>ปลอดภัย / น่าเรียน</span>
                                    <span id="voteUpCount" class="px-1.5 py-0.5 rounded-md bg-emerald-500/20 font-bold text-xs">31</span>
                                </button>

                                <button type="button" id="voteDownBtn" onclick="voteAreaSentiment('down')" class="inline-flex items-center gap-1.5 px-3.5 py-2 rounded-xl text-xs sm:text-sm font-medium bg-rose-500/10 hover:bg-rose-500/20 text-rose-700 dark:text-rose-300 border border-rose-500/30 transition active:scale-95">
                                    <i data-lucide="thumbs-down" class="w-4 h-4"></i>
                                    <span>ควรปรับปรุง/เฝ้าระวัง</span>
                                    <span id="voteDownCount" class="px-1.5 py-0.5 rounded-md bg-rose-500/20 font-bold text-xs">3</span>
                                </button>
                            </div>

                            <!-- Sentiment Percentage Progress Bar -->
                            <div class="w-full space-y-0.5">
                                <div class="flex justify-between text-[10px] font-medium">
                                    <span id="sentimentUpPct" class="text-emerald-600 dark:text-emerald-400">ปลอดภัย/น่าเรียน 91%</span>
                                    <span id="sentimentDownPct" class="text-rose-600 dark:text-rose-400">เฝ้าระวัง 9%</span>
                                </div>
                                <div class="w-full h-1.5 rounded-full bg-rose-500/25 overflow-hidden flex">
                                    <div id="sentimentProgressBar" class="h-full bg-emerald-500 transition-all duration-500" style="width: 91%"></div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Two-Column Form + Community Comments Feed -->
                <div class="grid grid-cols-1 lg:grid-cols-12 gap-6">
                    <!-- Left: Submit Rating & Comment Form -->
                    <div class="lg:col-span-5 bg-slate-50 dark:bg-slate-900/50 border border-slate-200/80 dark:border-slate-700/50 rounded-2xl p-4 sm:p-5 space-y-4">
                        <h4 class="text-sm font-semibold text-slate-800 dark:text-slate-100 flex items-center gap-2">
                            <i data-lucide="pen-square" class="w-4 h-4 text-sky-500"></i>
                            <span>ร่วมประเมินและแจ้งข้อเสนอแนะในจุดนี้</span>
                        </h4>

                        <!-- Interactive Star Rating Selector -->
                        <div>
                            <div class="flex flex-wrap items-center justify-between gap-1 mb-1.5">
                                <span class="text-xs text-slate-500 dark:text-slate-400">ให้คะแนนความปลอดภัย/ความน่าอยู่:</span>
                                <span id="starRatingLabel" class="text-xs font-semibold text-amber-600 dark:text-amber-400">5 ดาว - ดีเยี่ยม/ปลอดภัยมาก</span>
                            </div>
                            <div id="starRatingContainer" class="relative z-10 inline-flex items-center gap-1.5 select-none" role="radiogroup" aria-label="เลือกคะแนนดาว 1 ถึง 5 ดาว">
                                <button type="button" data-rating="1" aria-label="1 ดาว" onclick="window.setReviewStar(1)" onmouseenter="window.hoverReviewStar(1)" onmouseleave="window.resetReviewStarHover()" class="review-star-btn relative z-10 cursor-pointer p-1.5 rounded-xl hover:bg-amber-500/15 focus:outline-none focus:ring-2 focus:ring-amber-400 transition active:scale-90">
                                    <svg viewBox="0 0 24 24" class="w-7 h-7 pointer-events-none transition-transform duration-150" fill="#f59e0b" stroke="#f59e0b" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                                        <polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"></polygon>
                                    </svg>
                                </button>
                                <button type="button" data-rating="2" aria-label="2 ดาว" onclick="window.setReviewStar(2)" onmouseenter="window.hoverReviewStar(2)" onmouseleave="window.resetReviewStarHover()" class="review-star-btn relative z-10 cursor-pointer p-1.5 rounded-xl hover:bg-amber-500/15 focus:outline-none focus:ring-2 focus:ring-amber-400 transition active:scale-90">
                                    <svg viewBox="0 0 24 24" class="w-7 h-7 pointer-events-none transition-transform duration-150" fill="#f59e0b" stroke="#f59e0b" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                                        <polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"></polygon>
                                    </svg>
                                </button>
                                <button type="button" data-rating="3" aria-label="3 ดาว" onclick="window.setReviewStar(3)" onmouseenter="window.hoverReviewStar(3)" onmouseleave="window.resetReviewStarHover()" class="review-star-btn relative z-10 cursor-pointer p-1.5 rounded-xl hover:bg-amber-500/15 focus:outline-none focus:ring-2 focus:ring-amber-400 transition active:scale-90">
                                    <svg viewBox="0 0 24 24" class="w-7 h-7 pointer-events-none transition-transform duration-150" fill="#f59e0b" stroke="#f59e0b" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                                        <polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"></polygon>
                                    </svg>
                                </button>
                                <button type="button" data-rating="4" aria-label="4 ดาว" onclick="window.setReviewStar(4)" onmouseenter="window.hoverReviewStar(4)" onmouseleave="window.resetReviewStarHover()" class="review-star-btn relative z-10 cursor-pointer p-1.5 rounded-xl hover:bg-amber-500/15 focus:outline-none focus:ring-2 focus:ring-amber-400 transition active:scale-90">
                                    <svg viewBox="0 0 24 24" class="w-7 h-7 pointer-events-none transition-transform duration-150" fill="#f59e0b" stroke="#f59e0b" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                                        <polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"></polygon>
                                    </svg>
                                </button>
                                <button type="button" data-rating="5" aria-label="5 ดาว" onclick="window.setReviewStar(5)" onmouseenter="window.hoverReviewStar(5)" onmouseleave="window.resetReviewStarHover()" class="review-star-btn relative z-10 cursor-pointer p-1.5 rounded-xl hover:bg-amber-500/15 focus:outline-none focus:ring-2 focus:ring-amber-400 transition active:scale-90">
                                    <svg viewBox="0 0 24 24" class="w-7 h-7 pointer-events-none transition-transform duration-150" fill="#f59e0b" stroke="#f59e0b" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                                        <polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"></polygon>
                                    </svg>
                                </button>
                            </div>
                        </div>

                        <!-- School-Specific Quick Tags -->
                        <div>
                            <p class="text-xs text-slate-500 dark:text-slate-400 mb-1.5">เลือกจุดเด่นหรือข้อสังเกต (เลือกได้หลายข้อ):</p>
                            <div id="reviewTagChips" class="flex flex-wrap gap-1.5 text-[11px]">
                                <button type="button" onclick="toggleReviewTag(this, 'จุดรับ-ส่งปลอดภัย')" class="review-tag-chip px-2.5 py-1 rounded-full bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-slate-600 dark:text-slate-300 transition">จุดรับ-ส่งปลอดภัย</button>
                                <button type="button" onclick="toggleReviewTag(this, 'ทางม้าลายชัดเจน')" class="review-tag-chip px-2.5 py-1 rounded-full bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-slate-600 dark:text-slate-300 transition">ทางม้าลายชัดเจน</button>
                                <button type="button" onclick="toggleReviewTag(this, 'ร่มรื่นน่าเรียน')" class="review-tag-chip px-2.5 py-1 rounded-full bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-slate-600 dark:text-slate-300 transition">ร่มรื่นน่าเรียน</button>
                                <button type="button" onclick="toggleReviewTag(this, 'โรงอาหารสะอาด')" class="review-tag-chip px-2.5 py-1 rounded-full bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-slate-600 dark:text-slate-300 transition">โรงอาหารสะอาด</button>
                                <button type="button" onclick="toggleReviewTag(this, 'ระวังรถเร็วหน้าโรงเรียน')" class="review-tag-chip px-2.5 py-1 rounded-full bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-slate-600 dark:text-slate-300 transition">ระวังรถเร็วหน้าโรงเรียน</button>
                                <button type="button" onclick="toggleReviewTag(this, 'ระวังน้ำขังช่วงฝนตก')" class="review-tag-chip px-2.5 py-1 rounded-full bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-slate-600 dark:text-slate-300 transition">ระวังน้ำขังช่วงฝนตก</button>
                            </div>
                        </div>

                        <!-- Reviewer Name & Comment -->
                        <div class="space-y-2.5">
                            <input type="text" id="reviewerNameInput" maxlength="60" placeholder="ชื่อผู้แสดงความคิดเห็น (เช่น นักเรียน ม.5, ผู้ปกครอง, ครูเวร)" class="w-full px-3.5 py-2 bg-white dark:bg-slate-900 border border-slate-300 dark:border-slate-700 rounded-xl text-xs sm:text-sm text-slate-800 dark:text-slate-100 placeholder-slate-400 focus:outline-none focus:ring-2 focus:ring-sky-500">
                            <textarea id="reviewCommentInput" rows="3" maxlength="500" placeholder="เล่าสภาพแวดล้อม ความปลอดภัย จุดรับ-ส่ง หรือสิ่งที่ควรระวังในโซนนี้..." class="w-full px-3.5 py-2 bg-white dark:bg-slate-900 border border-slate-300 dark:border-slate-700 rounded-xl text-xs sm:text-sm text-slate-800 dark:text-slate-100 placeholder-slate-400 focus:outline-none focus:ring-2 focus:ring-sky-500 resize-none"></textarea>
                        </div>

                        <button type="button" onclick="submitAreaReview()" class="w-full py-2.5 px-4 bg-sky-600 hover:bg-sky-500 text-white font-medium text-xs sm:text-sm rounded-xl shadow-sm transition active:scale-95 flex items-center justify-center gap-2">
                            <i data-lucide="send" class="w-4 h-4"></i>
                            <span>ส่งคะแนนและความคิดเห็น</span>
                        </button>
                    </div>

                    <!-- Right: Community Comments List -->
                    <div class="lg:col-span-7 flex flex-col justify-between space-y-3">
                        <div class="flex items-center justify-between">
                            <h4 class="text-sm font-semibold text-slate-800 dark:text-slate-100 flex items-center gap-2">
                                <i data-lucide="messages-square" class="w-4 h-4 text-emerald-500"></i>
                                <span>ความคิดเห็นล่าสุดของโซนนี้</span>
                            </h4>
                            <span id="communitySentimentRatio" class="text-xs font-medium text-emerald-600 dark:text-emerald-400">
                                91% แนะนำว่าปลอดภัย/น่าเรียน
                            </span>
                        </div>

                        <div id="communityReviewsList" class="space-y-3 max-h-96 overflow-y-auto pr-1">
                            <!-- Dynamic Community Reviews Injected Here -->
                        </div>
                    </div>
                </div>
            </section>
        </div>
    </main>

    <!-- Footer -->
    <footer class="border-t border-slate-200 dark:border-slate-800 bg-white/70 dark:bg-slate-900/70 py-4 px-4 sm:px-6 lg:px-8 mt-6">
        <div class="max-w-7xl mx-auto flex flex-col sm:flex-row items-center justify-between gap-2 text-xs text-slate-500 dark:text-slate-400">
            <p>GeoSafety & Livability • ระบบประเมินความปลอดภัยและสภาพแวดล้อม <strong class="text-slate-700 dark:text-slate-200">โรงเรียนบุญเหลือวิทยานุสรณ์</strong> (ต.โคกสูง อ.เมืองนครราชสีมา)</p>
            <p>พิกัดอ้างอิง: 15.0538, 102.1389 • ข้อมูลสภาพอากาศเรียลไทม์จาก Open-Meteo</p>
        </div>
    </footer>

    <!-- QR Code Generator & Download Modal -->
    <div id="qrCodeModal" onclick="if(event.target === this) closeQrCodeModal()" class="hidden fixed inset-0 z-[9996] bg-slate-950/65 backdrop-blur-sm flex items-center justify-center p-4">
        <div class="bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-2xl max-w-md w-full p-5 sm:p-6 shadow-2xl space-y-4">
            <!-- Modal Header -->
            <div class="flex items-start justify-between gap-3">
                <div class="flex items-center gap-3">
                    <div class="p-2.5 rounded-xl bg-sky-500/15 text-sky-600 dark:text-sky-400">
                        <i data-lucide="qr-code" class="w-6 h-6"></i>
                    </div>
                    <div>
                        <h3 class="font-bold text-base text-slate-900 dark:text-white">QR Code เข้าใช้งานเว็บไซต์</h3>
                        <p class="text-xs text-slate-500 dark:text-slate-400">GeoSafety • โรงเรียนบุญเหลือวิทยานุสรณ์</p>
                    </div>
                </div>
                <button type="button" onclick="closeQrCodeModal()" class="p-1.5 text-slate-400 hover:text-slate-600 dark:hover:text-slate-200 rounded-lg transition">
                    <i data-lucide="x" class="w-5 h-5"></i>
                </button>
            </div>

            <!-- Target Link Mode Switcher -->
            <div class="grid grid-cols-2 gap-1.5 p-1 bg-slate-100 dark:bg-slate-900/80 rounded-xl text-xs font-medium">
                <button type="button" id="qrModeHomeBtn" onclick="setQrTargetMode('home')" class="py-2 px-3 rounded-lg bg-sky-600 text-white shadow-sm transition">
                    หน้าหลักโรงเรียน
                </button>
                <button type="button" id="qrModeZoneBtn" onclick="setQrTargetMode('zone')" class="py-2 px-3 rounded-lg text-slate-600 dark:text-slate-300 hover:bg-white/60 dark:hover:bg-slate-800 transition">
                    เฉพาะจุดตรวจสอบนี้
                </button>
            </div>

            <!-- QR Code Visual Card -->
            <div class="bg-slate-50 dark:bg-slate-900/70 border border-slate-200 dark:border-slate-700/70 rounded-2xl p-4 flex flex-col items-center text-center space-y-3">
                <div class="bg-white p-3.5 rounded-2xl shadow-md border border-slate-200/80">
                    <div id="qrCodeRenderBox" class="w-48 h-48 flex items-center justify-center"></div>
                </div>
                <div class="space-y-0.5">
                    <p id="qrZoneCaption" class="text-xs font-semibold text-slate-800 dark:text-slate-100">ภาพรวมโรงเรียนบุญเหลือวิทยานุสรณ์</p>
                    <p class="text-[11px] text-slate-500 dark:text-slate-400">สแกนด้วยกล้องมือถือเพื่อตรวจสอบความปลอดภัยและค่าฝุ่น PM2.5 สด</p>
                </div>
            </div>

            <!-- Editable Website URL Box -->
            <div class="space-y-1.5">
                <label for="qrUrlInput" class="block text-xs font-medium text-slate-600 dark:text-slate-300">
                    ลิงก์ปลายทาง (สามารถแก้ไขหรือวางลิงก์เว็บจริงที่อัปโหลดแล้วได้):
                </label>
                <input type="text" id="qrUrlInput" oninput="generateQrFromInput()" placeholder="https://..." class="w-full px-3 py-2 bg-slate-50 dark:bg-slate-900 border border-slate-300 dark:border-slate-700 rounded-xl text-xs font-mono text-slate-800 dark:text-slate-100 focus:outline-none focus:ring-2 focus:ring-sky-500">
                <p id="qrLocalHostNotice" class="hidden text-[11px] text-amber-600 dark:text-amber-400">
                    * หากเปิดจากไฟล์ในเครื่องหรือหน้าต่างพรีวิว กรุณาวางลิงก์เว็บจริง (https://...) ด้านบนก่อนดาวน์โหลดเพื่อให้มือถือเครื่องอื่นสแกนเข้าได้ครับ
                </p>
            </div>

            <!-- Action Buttons -->
            <div class="grid grid-cols-2 gap-2 pt-1">
                <button type="button" onclick="copyQrUrl()" class="py-2.5 px-3 text-xs sm:text-sm font-medium bg-slate-100 hover:bg-slate-200 dark:bg-slate-700 dark:hover:bg-slate-600 text-slate-700 dark:text-slate-200 rounded-xl transition active:scale-95 flex items-center justify-center gap-1.5">
                    <i data-lucide="copy" class="w-4 h-4 text-sky-500"></i>
                    <span>คัดลอกลิงก์</span>
                </button>
                <button type="button" onclick="downloadQrCodePng()" class="py-2.5 px-3 text-xs sm:text-sm font-medium bg-sky-600 hover:bg-sky-500 text-white rounded-xl shadow-sm transition active:scale-95 flex items-center justify-center gap-1.5">
                    <i data-lucide="download" class="w-4 h-4"></i>
                    <span>ดาวน์โหลด PNG</span>
                </button>
            </div>
        </div>
    </div>

    <!-- GPS Permission Help Modal -->
    <div id="gpsHelpModal" class="hidden fixed inset-0 z-[9995] bg-slate-950/60 backdrop-blur-sm flex items-center justify-center p-4">
        <div class="bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-2xl max-w-md w-full p-6 shadow-2xl space-y-4">
            <div class="flex items-start justify-between gap-3">
                <div class="flex items-center gap-3">
                    <div class="p-2.5 rounded-xl bg-amber-500/15 text-amber-600 dark:text-amber-400">
                        <i data-lucide="map-pin-off" class="w-6 h-6"></i>
                    </div>
                    <div>
                        <h3 class="font-bold text-base text-slate-900 dark:text-white">ไม่สามารถเข้าถึงตำแหน่ง GPS อัตโนมัติ</h3>
                        <p class="text-xs text-slate-500 dark:text-slate-400">เบราว์เซอร์หรือหน้าต่างพรีวิวจำกัดสิทธิ์ตำแหน่ง</p>
                    </div>
                </div>
                <button type="button" onclick="closeGpsHelpModal()" class="p-1 text-slate-400 hover:text-slate-600 dark:hover:text-slate-200 rounded-lg">
                    <i data-lucide="x" class="w-5 h-5"></i>
                </button>
            </div>

            <div class="space-y-2.5 text-xs text-slate-600 dark:text-slate-300 bg-slate-50 dark:bg-slate-900/60 p-3.5 rounded-xl border border-slate-200/80 dark:border-slate-700/60">
                <p class="font-semibold text-slate-800 dark:text-slate-100">วิธีแก้ไขอย่างรวดเร็ว:</p>
                <p>1. <strong>บนพีซี / มือถือ:</strong> กดไอคอนรูป <strong>กุญแจ หรือ ตัวปรับตั้งค่า</strong> ข้างช่อง URL ด้านบน แล้วเปลี่ยนสิทธิ์ <em>ตำแหน่ง (Location)</em> เป็น <strong>อนุญาต (Allow)</strong></p>
                <p>2. <strong>หากเปิดผ่านแอป Google / LINE / Facebook:</strong> กดปุ่มคัดลอกลิงก์ด้านล่างแล้วนำไปเปิดใน <strong>Chrome</strong> หรือ <strong>Safari</strong></p>
                <p>3. <strong>หรือเลือกจุดบนแผนที่โรงเรียนโดยตรง:</strong> คลิกจุดที่คุณอยู่ภายในโรงเรียนบนแผนที่ ระบบจะวิเคราะห์พื้นที่ให้ทันที</p>
            </div>

            <div class="flex flex-wrap items-center justify-end gap-2">
                <button type="button" onclick="copyAppUrlForExternalBrowser()" class="px-3.5 py-2 text-xs font-medium bg-slate-100 hover:bg-slate-200 dark:bg-slate-700 dark:hover:bg-slate-600 text-slate-700 dark:text-slate-200 rounded-xl transition">
                    คัดลอกลิงก์เว็บ
                </button>
                <button type="button" onclick="closeGpsHelpModal()" class="px-4 py-2 text-xs font-medium bg-sky-600 hover:bg-sky-500 text-white rounded-xl transition">
                    เข้าใจแล้ว
                </button>
            </div>
        </div>
    </div>

    <!-- Application Logic: Boonluea Wittayanusorn School GeoSafety Engine -->
    <script>
        // =========================================================================
        // 1. SCHOOL CONSTANTS, 7 CAMPUS ZONES & LOCAL LANDMARKS
        // =========================================================================
        const SCHOOL_CENTER = {
            lat: 15.0538,
            lon: 102.1389,
            name: 'โรงเรียนบุญเหลือวิทยานุสรณ์',
            subdistrict: 'ต.โคกสูง อ.เมืองนครราชสีมา จ.นครราชสีมา 30310',
            safetyRadiusMeters: 300
        };

        const STORAGE_KEY = 'bw_school_geosafety_v1';

        const SCHOOL_ZONES = [
            {
                id: 'overview',
                title: 'ภาพรวมโรงเรียนบุญเหลือวิทยานุสรณ์ (จุดศูนย์กลาง)',
                shortName: 'ศูนย์กลางโรงเรียนบุญเหลือวิทยานุสรณ์',
                keywords: ['บุญเหลือวิทยานุสรณ์', 'บุญเหลือวิทยานุสร์', 'ศูนย์กลาง', 'ภาพรวม', 'โรงเรียน', 'โคกสูง'],
                lat: 15.0538,
                lon: 102.1389,
                zoneType: 'campus_center',
                scores: { safety: 90, env: 86, infra: 92, transit: 85, green: 89 },
                cctv: 'ครอบคลุมจุดหลักทุกอาคาร',
                crossing: 'มีทางม้าลาย & ครูเวรหน้าโรงเรียน',
                flood: 'ระบายน้ำได้ดี (พื้นที่ดอน)',
                hospital: 'ห้องพยาบาลใน รร. / รพ.สต.โคกสูง 1.2 กม.',
                police: 'สายตรวจโคกสูง (~850 ม.)',
                transit: 'ติด ถ.สุรนารายณ์ เดินทางสะดวก',
                walk: 'มีทางเท้าและป้ายเขตโรงเรียนชัดเจน',
                greenDesc: 'ภายในโรงเรียนบุญเหลือวิทยานุสรณ์มีแนวต้นไม้ร่มรื่น สนามฟุตบอลมาตรฐาน และลานกิจกรรมกว้างขวาง เหมาะกับการเรียนรู้และออกกำลังกาย',
                tip: 'เขตปลอดภัยรอบสถานศึกษารัศมี 300 เมตร ครอบคลุมแนว ถ.สุรนารายณ์ มีครูเวรและระบบกล้อง CCTV ดูแลจุดสำคัญ'
            },
            {
                id: 'front_gate',
                title: 'ประตูหน้าโรงเรียน & จุดรับ-ส่งนักเรียน (ถ.สุรนารายณ์)',
                shortName: 'ประตูหน้าโรงเรียน & จุดรับ-ส่ง',
                keywords: ['ประตูหน้า', 'จุดรับส่ง', 'รับส่งนักเรียน', 'ถ.สุรนารายณ์', 'สุรนารายณ์', 'ทางเข้า'],
                lat: 15.0526,
                lon: 102.1396,
                zoneType: 'gate_road',
                scores: { safety: 85, env: 80, infra: 90, transit: 92, green: 78 },
                cctv: 'กล้องความละเอียดสูงส่องป้ายทะเบียน',
                crossing: 'มีครูเวร & ป้ายชะลอความเร็วเขตโรงเรียน',
                flood: 'ระบายน้ำลงรางริม ถ.สุรนารายณ์',
                hospital: 'ห้องพยาบาล รร. 140 ม. / รพ.สต.โคกสูง 1.1 กม.',
                police: 'จุดตรวจจราจรหน้าโรงเรียนช่วงเช้า-เย็น',
                transit: 'จุดจอดรับ-ส่ง & ป้ายรถโดยสารหน้าประตู',
                walk: 'ทางเท้าหน้าประตู ควรระวังรถเข้า-ออก',
                greenDesc: 'บริเวณหน้าประตูทางเข้าติดถนนสุรนารายณ์ มีซุ้มประตูและแนวไม้ประดับ โปร่งโล่งมองเห็นรถสัญจรได้ชัดเจน',
                tip: 'ช่วงเวลา 07:00-08:15 น. และ 15:30-17:00 น. การจราจรหน้าประตูหนาแน่น แนะนำให้จอดรับ-ส่งในจุดที่กำหนดและข้ามทางม้าลายทุกครั้ง'
            },
            {
                id: 'main_buildings',
                title: 'โซนอาคารเรียนหลัก & ห้องสมุด',
                shortName: 'อาคารเรียนหลัก & ห้องสมุด',
                keywords: ['อาคารเรียน', 'ห้องสมุด', 'ห้องเรียน', 'ตึกเรียน', 'ห้องพักครู'],
                lat: 15.0541,
                lon: 102.1386,
                zoneType: 'academic',
                scores: { safety: 93, env: 88, infra: 94, transit: 84, green: 88 },
                cctv: 'ติดตั้งตามโถงทางเดินและบันไดอาคาร',
                crossing: 'ทางเดินภายในปลอดรถยนต์ 100%',
                flood: 'พื้นอาคารยกสูง ปลอดภัยจากน้ำขัง',
                hospital: 'ห้องพยาบาลอาคาร 1 (เดิน < 2 นาที)',
                police: 'พื้นที่ควบคุมบุคคลภายนอกภายในโรงเรียน',
                transit: 'เชื่อมต่อทางเดินมีหลังคาคลุมระหว่างอาคาร',
                walk: 'ทางเดินในอาคารกว้างขวางและปลอดภัยสูง',
                greenDesc: 'รอบอาคารเรียนมีสวนหย่อมและม้านั่งใต้ร่มไม้สำหรับอ่านหนังสือ บรรยากาศเงียบสงบเหมาะแก่การเรียนการสอน',
                tip: 'โซนอาคารเรียนเป็นพื้นที่ปลอดยานพาหนะ มีถังดับเพลิงประจำชั้นและแสงสว่างเพียงพอตลอดวัน'
            },
            {
                id: 'sports_field',
                title: 'สนามกีฬา สนามฟุตบอล & ลานกิจกรรมหน้าเสาธง',
                shortName: 'สนามกีฬา & ลานหน้าเสาธง',
                keywords: ['สนามกีฬา', 'สนามฟุตบอล', 'หน้าเสาธง', 'ลานกิจกรรม', 'พละ', 'ออกกำลังกาย'],
                lat: 15.0548,
                lon: 102.1379,
                zoneType: 'outdoor_sports',
                scores: { safety: 88, env: 85, infra: 88, transit: 82, green: 95 },
                cctv: 'ครอบคลุมขอบสนามและลานหน้าเสาธง',
                crossing: 'พื้นที่เปิดโล่งในรั้วโรงเรียน',
                flood: 'สนามหญ้าระบายน้ำเร็วหลังฝนหยุด',
                hospital: 'ชุดปฐมพยาบาลประจำหมวดพละ & ห้องพยาบาล 160 ม.',
                police: 'มีครูผู้สอนและเจ้าหน้าที่ดูแลตลอดกิจกรรม',
                transit: 'เข้าถึงได้จากทางเดินรอบสนามกีฬา',
                walk: 'ลู่วิ่งและขอบสนามเรียบ ปลอดภัยสำหรับวิ่ง',
                greenDesc: 'พื้นที่สีเขียวขนาดใหญ่ที่สุดของโรงเรียน ประกอบด้วยสนามฟุตบอล ลานอเนกประสงค์ และต้นไม้ใหญ่รอบขอบสนาม',
                tip: 'ก่อนทำกิจกรรมกลางแจ้งหรือวิชาพละ แนะนำตรวจสอบค่าฝุ่น PM2.5 และสภาพอากาศสดด้านบนเพื่อความปลอดภัยของนักเรียน'
            },
            {
                id: 'canteen',
                title: 'โรงอาหาร & ร้านค้าสวัสดิการนักเรียน',
                shortName: 'โรงอาหาร & ร้านค้าสวัสดิการ',
                keywords: ['โรงอาหาร', 'อาหารกลางวัน', 'ร้านค้า', 'สวัสดิการ', 'น้ำดื่ม'],
                lat: 15.0534,
                lon: 102.1382,
                zoneType: 'canteen',
                scores: { safety: 91, env: 87, infra: 95, transit: 86, green: 85 },
                cctv: 'ติดตั้งรอบทางเข้า-ออกโรงอาหาร',
                crossing: 'ทางเดินเชื่อมต่ออาคารเรียนสะดวก',
                flood: 'พื้นกระเบื้องกันลื่นและรางระบายน้ำรอบอาคาร',
                hospital: 'จุดล้างมือ น้ำดื่มสะอาด และห้องพยาบาล 90 ม.',
                police: 'มีครูเวรตรวจความเรียบร้อยช่วงพักกลางวัน',
                transit: 'อยู่กึ่งกลางระหว่างอาคารเรียนและประตูหลัก',
                walk: 'ระวังพื้นเปียกชื้นบริเวณจุดล้างภาชนะ',
                greenDesc: 'โรงอาหารอากาศถ่ายเทสะดวก มีร่มไม้และที่นั่งพักผ่อนรอบบริเวณ พร้อมจุดคัดแยกขยะและจุดบริการน้ำดื่มสะอาด',
                tip: 'ช่วงพักกลางวัน (11:30-13:00 น.) มีนักเรียนใช้งานหนาแน่น ควรเดินด้วยความระมัดระวังบริเวณจุดเก็บภาชนะ'
            },
            {
                id: 'auditorium',
                title: 'หอประชุม & อาคารอเนกประสงค์',
                shortName: 'หอประชุม & อาคารอเนกประสงค์',
                keywords: ['หอประชุม', 'อเนกประสงค์', 'ประชุม', 'กิจกรรมในร่ม'],
                lat: 15.0544,
                lon: 102.1392,
                zoneType: 'auditorium',
                scores: { safety: 92, env: 89, infra: 93, transit: 85, green: 86 },
                cctv: 'ครอบคลุมประตูทางเข้า-ออกหอประชุม',
                crossing: 'ทางเดินในร่มเชื่อมต่อโซนอาคารเรียน',
                flood: 'อาคารในร่ม ปลอดภัยจากฝนและแดดจัด',
                hospital: 'เข้าถึงหน่วยปฐมพยาบาลได้ภายใน 2 นาที',
                police: 'ประตูทางออกฉุกเฉินเปิดโล่งพร้อมป้ายนำทาง',
                transit: 'ใกล้ลานจอดรถผู้มาติดต่อและครู-บุคลากร',
                walk: 'ทางลาดและทางเดินกว้างขวางรองรับคนจำนวนมาก',
                greenDesc: 'รายล้อมด้วยแนวไม้ประดับและลานจอดรถที่เป็นระเบียบ เป็นพื้นที่จัดกิจกรรมในร่มที่ปลอดภัยแม้ในวันที่ฝนตกหรือค่าฝุ่นสูง',
                tip: 'เป็นจุดรวมพลในร่มที่เหมาะสมเมื่อเกิดฝนตกหนักหรือค่าฝุ่น PM2.5 สูงเกินเกณฑ์'
            },
            {
                id: 'crosswalk_zone',
                title: 'โซนทางม้าลายและชุมชนหน้าโรงเรียน (รัศมี 300 ม.)',
                shortName: 'ทางม้าลาย & ชุมชนหน้าโรงเรียน',
                keywords: ['ทางม้าลาย', 'ชุมชนหน้าโรงเรียน', '300 ม.', 'ร้านค้าหน้าโรงเรียน', 'ข้ามถนน', 'ถนนมิตรภาพ'],
                lat: 15.0519,
                lon: 102.1401,
                zoneType: 'perimeter_road',
                scores: { safety: 80, env: 79, infra: 88, transit: 90, green: 76 },
                cctv: 'กล้องจราจรและกล้องร้านค้ารอบแนวถนน',
                crossing: 'ทางม้าลายและสัญญาณไฟกระพริบเขตโรงเรียน',
                flood: 'เฝ้าระวังน้ำรอการระบายริมขอบทางช่วงฝนตกหนัก',
                hospital: 'รพ.สต.โคกสูง ~1.1 กม. / รพ.มหาราชนครราชสีมา 11 กม.',
                police: 'ป้อมตำรวจชุมชนโคกสูง (~850 ม.)',
                transit: 'รถสองแถวและรถโดยสารสายโคกสูง-ในเมืองผ่านตลอด',
                walk: 'ควรเดินบนทางเท้าและข้ามถนนเฉพาะทางม้าลาย',
                greenDesc: 'แนวชุมชนและร้านค้าหน้าโรงเรียนบุญเหลือวิทยานุสรณ์ คึกคักช่วงเช้าและหลังเลิกเรียน มีร้านอาหารและจุดพักรอผู้ปกครอง',
                tip: 'รถที่วิ่งบนถนนสุรนารายณ์ใช้ความเร็วค่อนข้างสูง นักเรียนควรข้ามถนนเป็นกลุ่มและสังเกตสัญญาณมือจากครูเวร'
            }
        ];

        // Curated School-Area Landmarks for Accurate Local Plotting
        const LOCAL_SCHOOL_POIS = {
            firstaid: [
                { name: 'ห้องพยาบาล โรงเรียนบุญเหลือวิทยานุสรณ์', detail: 'จุดปฐมพยาบาลเบื้องต้นภายในโรงเรียน (อาคาร 1)', lat: 15.0536, lon: 102.1387 },
                { name: 'รพ.สต.โคกสูง (โรงพยาบาลส่งเสริมสุขภาพตำบล)', detail: 'สถานพยาบาลปฐมภูมิใกล้โรงเรียน (~1.2 กม.)', lat: 15.0585, lon: 102.1332 }
            ],
            police: [
                { name: 'ป้อมตำรวจชุมชนโคกสูง (สภ.จอหอ)', detail: 'จุดบริการประชาชนและสายตรวจตำบลโคกสูง (~850 ม.)', lat: 15.0502, lon: 102.1438 },
                { name: 'จุดอำนวยความสะดวกจราจรหน้าโรงเรียน', detail: 'จุดดูแลความปลอดภัยช่วงเช้า 07:00-08:15 น. และเย็น 15:30-17:00 น.', lat: 15.0525, lon: 102.1397 }
            ],
            green: [
                { name: 'สนามฟุตบอลและลู่วิ่ง รร.บุญเหลือวิทยานุสรณ์', detail: 'พื้นที่ออกกำลังกายและสนามกีฬากลางแจ้ง', lat: 15.0548, lon: 102.1379 },
                { name: 'สวนหย่อมและลานร่มไม้อาคารเรียน', detail: 'พื้นที่สีเขียวพักผ่อนและอ่านหนังสือของนักเรียน', lat: 15.0539, lon: 102.1384 }
            ],
            transit: [
                { name: 'จุดรับ-ส่งนักเรียน ประตูหน้า รร.บุญเหลือวิทยานุสรณ์', detail: 'จุดจอดส่ง-รับนักเรียนริม ถ.สุรนารายณ์', lat: 15.0526, lon: 102.1396 },
                { name: 'ศาลาที่พักผู้โดยสาร หน้าโรงเรียนบุญเหลือวิทยานุสรณ์', detail: 'จุดรอรถสองแถวและรถประจำทางสายโคกสูง-ตัวเมืองโคราช', lat: 15.0523, lon: 102.1399 }
            ]
        };

        const POI_META = {
            firstaid: { label: 'จุดปฐมพยาบาล/สถานพยาบาล', color: '#f43f5e', query: '["amenity"~"hospital|clinic"]' },
            police: { label: 'ป้อมตำรวจ/จุดบริการ', color: '#3b82f6', query: '["amenity"="police"]' },
            green: { label: 'พื้นที่สีเขียว/สนามกีฬา', color: '#10b981', query: '["leisure"~"park|garden|pitch"]' },
            transit: { label: 'จุดรับ-ส่ง/ป้ายรถโดยสาร', color: '#f59e0b', query: '["highway"="bus_stop"]' }
        };

        const TILE_URLS = {
            street: 'https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png',
            dark: 'https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png',
            satellite: 'https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}',
            satelliteLabels: 'https://{s}.basemaps.cartocdn.com/dark_only_labels/{z}/{x}/{y}{r}.png'
        };

        // =========================================================================
        // 2. APPLICATION STATE & SAFE HTML ESCAPING
        // =========================================================================
        let map = null;
        let marker = null;
        let zoneRadiusCircle = null;
        let schoolSafetyBoundaryCircle = null;
        let userLocationMarker = null;
        let userAccuracyCircle = null;
        let currentBaseLayer = null;
        let currentLabelLayer = null;
        let liveRadarLayer = null;
        let isLiveRadarActive = false;
        let currentMapMode = 'satellite';
        window.currentMapMode = 'satellite';
        let isMapFullscreen = false;
        let isLocatingGPS = false;
        let isLiveTracking = false;
        let gpsWatchId = null;
        let lastTrackedCoords = null;
        let lastGoodGpsFix = null;
        let currentZone = null;
        let autoRefreshEnabled = true;
        let nextRefreshSeconds = 30;
        let liveClockTimer = null;
        let autoSyncTimer = null;
        let toastTimeout = null;
        let currentSuggestions = [];
        const activePOICategories = new Set();
        const poiLayerGroups = {
            firstaid: null,
            police: null,
            green: null,
            transit: null
        };

        function escapeHtml(str) {
            return String(str ?? '')
                .replace(/&/g, '&amp;')
                .replace(/</g, '&lt;')
                .replace(/>/g, '&gt;')
                .replace(/"/g, '&quot;')
                .replace(/'/g, '&#39;');
        }

        function refreshIcons() {
            if (typeof lucide !== 'undefined' && typeof lucide.createIcons === 'function') {
                try {
                    lucide.createIcons();
                } catch (e) {}
            }
        }

        function showToast(message, icon = 'info', duration = 3000) {
            let toastEl = document.getElementById('appToast');
            if (!toastEl) {
                toastEl = document.createElement('div');
                toastEl.id = 'appToast';
                toastEl.className = 'fixed bottom-5 left-1/2 -translate-x-1/2 z-[9999] px-4 py-2.5 rounded-xl bg-slate-900/95 dark:bg-slate-800/95 text-white text-xs sm:text-sm font-medium shadow-2xl border border-slate-700/80 backdrop-blur-md flex items-center gap-2 transition-all duration-300 opacity-0 pointer-events-none max-w-[92vw]';
                document.body.appendChild(toastEl);
            }

            toastEl.innerHTML = `<i data-lucide="${escapeHtml(icon)}" class="w-4 h-4 text-sky-400 shrink-0"></i><span>${escapeHtml(message)}</span>`;
            refreshIcons();

            toastEl.classList.remove('opacity-0', 'translate-y-2');
            toastEl.classList.add('opacity-100', 'translate-y-0');

            if (toastTimeout) clearTimeout(toastTimeout);
            toastTimeout = setTimeout(() => {
                toastEl.classList.remove('opacity-100', 'translate-y-0');
                toastEl.classList.add('opacity-0', 'translate-y-2');
            }, duration);
        }

        // =========================================================================
        // 3. CAMPUS-SCALE REAL-TIME SATELLITE MAP & LIVE RADAR
        // =========================================================================
        function initMap(lat = SCHOOL_CENTER.lat, lon = SCHOOL_CENTER.lon) {
            if (typeof L === 'undefined') return;
            const mapEl = document.getElementById('map');
            if (!mapEl) return;

            const numLat = Number.isFinite(Number(lat)) ? Number(lat) : SCHOOL_CENTER.lat;
            const numLon = Number.isFinite(Number(lon)) ? Number(lon) : SCHOOL_CENTER.lon;

            map = L.map('map', {
                center: [numLat, numLon],
                zoom: 17,
                zoomControl: false
            });

            Object.keys(poiLayerGroups).forEach(key => {
                poiLayerGroups[key] = L.layerGroup().addTo(map);
            });

            // Default to Real-Time Satellite View with transparent road & place labels
            setMapLayer('satellite', true);

            // Fixed 300-meter School Safety Zone Circle centered on Boonluea Wittayanusorn School
            schoolSafetyBoundaryCircle = L.circle([SCHOOL_CENTER.lat, SCHOOL_CENTER.lon], {
                radius: SCHOOL_CENTER.safetyRadiusMeters,
                color: '#38bdf8',
                fillColor: '#0ea5e9',
                fillOpacity: 0.08,
                weight: 2,
                dashArray: '6, 6'
            }).addTo(map).bindTooltip('เขตปลอดภัยรอบโรงเรียนบุญเหลือวิทยานุสรณ์ (รัศมี 300 ม.)', { direction: 'top' });

            // Plot clickable campus zone pins for fast visual navigation
            SCHOOL_ZONES.forEach(z => {
                const campusDot = L.circleMarker([z.lat, z.lon], {
                    radius: 6,
                    color: '#ffffff',
                    weight: 2,
                    fillColor: '#6366f1',
                    fillOpacity: 0.92
                }).addTo(map);

                campusDot.bindTooltip(escapeHtml(z.shortName), { direction: 'top', offset: [0, -4] });
                campusDot.on('click', (e) => {
                    if (e && e.originalEvent) L.DomEvent.stopPropagation(e.originalEvent);
                    loadZoneData(z.id);
                });
            });

            // Draggable Red Target Pin for exact building/incident inspection
            const pinIcon = L.divIcon({
                className: 'custom-target-pin',
                html: `<div style="position:relative;width:32px;height:32px;display:flex;align-items:center;justify-content:center;">
                        <span style="width:26px;height:26px;border-radius:9999px;background:#ef4444;border:3px solid #ffffff;box-shadow:0 4px 12px rgba(239,68,68,0.6);display:flex;align-items:center;justify-content:center;">
                            <span style="width:6px;height:6px;border-radius:9999px;background:#ffffff;"></span>
                        </span>
                       </div>`,
                iconSize: [32, 32],
                iconAnchor: [16, 16]
            });

            marker = L.marker([numLat, numLon], { icon: pinIcon, draggable: true, zIndexOffset: 800 }).addTo(map);
            marker.on('dragend', function () {
                const pos = marker.getLatLng();
                if (pos) {
                    showToast('กำลังประเมินจุดที่ลากหมุดไปวาง...', 'map-pin', 2000);
                    inspectCustomSchoolCoordinate(pos.lat, pos.lng);
                }
            });

            // Active 60m local micro-zone ring around selected pin
            zoneRadiusCircle = L.circle([numLat, numLon], {
                radius: 60,
                color: '#ef4444',
                fillColor: '#ef4444',
                fillOpacity: 0.1,
                weight: 1.5
            }).addTo(map);

            // Click anywhere on the school map to inspect that exact spot
            map.on('click', function (e) {
                if (e && e.latlng) {
                    showToast('กำลังประเมินจุดที่เลือกภายในบริเวณโรงเรียน...', 'map-pin', 2000);
                    inspectCustomSchoolCoordinate(e.latlng.lat, e.latlng.lng);
                }
            });

            if (typeof ResizeObserver !== 'undefined') {
                const ro = new ResizeObserver(() => {
                    if (map) map.invalidateSize(false);
                });
                ro.observe(mapEl);
            }
        }

        function setMapLayer(mode, silent = false) {
            currentMapMode = mode;
            window.currentMapMode = mode;

            ['street', 'satellite', 'dark'].forEach(m => {
                const btn = document.getElementById(`layer-${m}`);
                if (!btn) return;
                if (m === mode) {
                    btn.className = 'px-2.5 py-1 rounded-lg transition-colors bg-sky-600 text-white flex items-center gap-1';
                } else {
                    btn.className = 'px-2.5 py-1 rounded-lg transition-colors text-slate-600 dark:text-slate-300 hover:bg-slate-100 dark:hover:bg-slate-800 flex items-center gap-1';
                }
            });

            if (!map || typeof L === 'undefined') return;

            if (currentBaseLayer) map.removeLayer(currentBaseLayer);
            if (currentLabelLayer) {
                map.removeLayer(currentLabelLayer);
                currentLabelLayer = null;
            }

            const tileUrl = TILE_URLS[mode] || TILE_URLS.satellite;
            currentBaseLayer = L.tileLayer(tileUrl, { maxZoom: 19 }).addTo(map);

            if (mode === 'satellite') {
                currentLabelLayer = L.tileLayer(TILE_URLS.satelliteLabels, { maxZoom: 19, zIndex: 500 }).addTo(map);
            }

            if (!silent) {
                const labels = { street: 'แผนที่ถนน', satellite: 'ภาพถ่ายดาวเทียมเวลาปัจจุบัน', dark: 'แผนที่โหมดมืด' };
                showToast(`เปลี่ยนมุมมองเป็น ${labels[mode] || mode}`, 'layers', 1800);
            }
        }

        async function toggleLiveRadar() {
            const btn = document.getElementById('liveRadarBtn');
            const dot = document.getElementById('liveRadarDot');
            const tag = document.getElementById('satRadarTimeTag');

            if (isLiveRadarActive) {
                isLiveRadarActive = false;
                if (liveRadarLayer && map) {
                    map.removeLayer(liveRadarLayer);
                    liveRadarLayer = null;
                }
                if (btn) btn.classList.remove('ring-2', 'ring-sky-500', 'bg-sky-500/15', 'text-sky-700', 'dark:text-sky-300');
                if (dot) dot.className = 'w-2 h-2 rounded-full bg-slate-400';
                if (tag) tag.classList.add('hidden');
                showToast('ปิดชั้นข้อมูลเรดาร์เมฆฝนสดแล้ว', 'info', 1800);
                return;
            }

            isLiveRadarActive = true;
            if (btn) btn.classList.add('ring-2', 'ring-sky-500', 'bg-sky-500/15', 'text-sky-700', 'dark:text-sky-300');
            if (dot) dot.className = 'w-2 h-2 rounded-full bg-emerald-500 animate-ping';
            await refreshLiveRadarLayer(false);
        }

        async function refreshLiveRadarLayer(silent = false) {
            if (!isLiveRadarActive || !map || typeof L === 'undefined') return;
            const tag = document.getElementById('satRadarTimeTag');

            try {
                const res = await fetch('https://api.rainviewer.com/public/weather-maps.json');
                if (!res.ok) throw new Error('Radar API unavailable');
                const data = await res.json();
                const pastFrames = Array.isArray(data?.radar?.past) ? data.radar.past : [];
                const latestFrame = pastFrames.length > 0 ? pastFrames[pastFrames.length - 1] : null;
                const host = data?.host || 'https://tilecache.rainviewer.com';

                if (!latestFrame || !latestFrame.path) throw new Error('No radar frame');

                const radarUrl = `${host}${latestFrame.path}/256/{z}/{x}/{y}/2/1_1.png`;
                if (liveRadarLayer) {
                    map.removeLayer(liveRadarLayer);
                }
                liveRadarLayer = L.tileLayer(radarUrl, {
                    opacity: 0.65,
                    zIndex: 450,
                    maxNativeZoom: 10,
                    maxZoom: 19
                }).addTo(map);

                const frameDate = new Date((latestFrame.time || Math.floor(Date.now() / 1000)) * 1000);
                const radarTimeStr = frameDate.toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' });
                if (tag) {
                    tag.textContent = `• เรดาร์ฝนสด ${radarTimeStr} น.`;
                    tag.classList.remove('hidden');
                }
                if (!silent) {
                    showToast(`ซ้อนทับเรดาร์กลุ่มเมฆฝนสด (รอบเวลา ${radarTimeStr} น.) เรียบร้อยแล้ว`, 'cloud-rain-wind', 2500);
                }
            } catch (err) {
                if (!silent) {
                    showToast('ไม่สามารถดึงภาพเรดาร์ฝนสดได้ชั่วคราว กรุณาลองใหม่', 'alert-circle', 2500);
                }
            }
        }

        async function togglePOICategory(cat) {
            const chip = document.getElementById(`chip-${cat}`);
            const meta = POI_META[cat];
            if (!meta || !poiLayerGroups[cat]) return;

            if (activePOICategories.has(cat)) {
                activePOICategories.delete(cat);
                poiLayerGroups[cat].clearLayers();
                if (chip) {
                    chip.classList.remove('ring-2', 'ring-sky-500', 'bg-sky-50', 'dark:bg-sky-500/20');
                }
                return;
            }

            activePOICategories.add(cat);
            if (chip) {
                chip.classList.add('ring-2', 'ring-sky-500', 'bg-sky-50', 'dark:bg-sky-500/20');
            }
            await loadCategoryPOIs(cat);
        }

        async function loadCategoryPOIs(cat) {
            if (!poiLayerGroups[cat] || typeof L === 'undefined') return;
            const meta = POI_META[cat];
            poiLayerGroups[cat].clearLayers();

            // 1. Always plot curated local school-zone landmarks first
            const curated = LOCAL_SCHOOL_POIS[cat] || [];
            curated.forEach(item => {
                const dot = L.circleMarker([item.lat, item.lon], {
                    radius: 8,
                    color: '#ffffff',
                    weight: 2,
                    fillColor: meta.color,
                    fillOpacity: 0.95
                }).bindPopup(`<strong>${escapeHtml(item.name)}</strong><br><span style="font-size:11px;color:#64748b;">${escapeHtml(item.detail)}</span>`);
                poiLayerGroups[cat].addLayer(dot);
            });

            // 2. Also enrich with live OpenStreetMap Overpass POIs within 1.5 km of the school
            try {
                const overpassQuery = `[out:json][timeout:8];(node${meta.query}(around:1500,${SCHOOL_CENTER.lat},${SCHOOL_CENTER.lon});way${meta.query}(around:1500,${SCHOOL_CENTER.lat},${SCHOOL_CENTER.lon}););out center 12;`;
                const res = await fetch('https://overpass-api.de/api/interpreter', {
                    method: 'POST',
                    body: overpassQuery
                });
                if (res.ok) {
                    const data = await res.json();
                    const elements = Array.isArray(data.elements) ? data.elements : [];
                    elements.forEach(el => {
                        const pLat = el.lat ?? el.center?.lat;
                        const pLon = el.lon ?? el.center?.lon;
                        if (!Number.isFinite(pLat) || !Number.isFinite(pLon)) return;
                        const name = el.tags?.['name:th'] || el.tags?.name || meta.label;
                        const dot = L.circleMarker([pLat, pLon], {
                            radius: 7,
                            color: '#ffffff',
                            weight: 2,
                            fillColor: meta.color,
                            fillOpacity: 0.9
                        }).bindPopup(`<strong>${escapeHtml(name)}</strong><br><span style="font-size:11px;color:#64748b;">${escapeHtml(meta.label)} (OpenStreetMap)</span>`);
                        poiLayerGroups[cat].addLayer(dot);
                    });
                }
            } catch (e) {}

            showToast(`แสดงหมุด${meta.label}รอบโรงเรียนบุญเหลือวิทยานุสรณ์แล้ว`, 'check-circle', 2200);
        }

        function toggleMapFullscreen() {
            const wrapper = document.getElementById('mapWrapper');
            const icon = document.getElementById('fullscreenIcon');
            if (!wrapper) return;

            isMapFullscreen = !isMapFullscreen;
            wrapper.classList.toggle('map-fullscreen-active', isMapFullscreen);
            if (icon) {
                icon.setAttribute('data-lucide', isMapFullscreen ? 'minimize-2' : 'maximize-2');
                refreshIcons();
            }
            if (map) {
                setTimeout(() => map.invalidateSize(false), 120);
            }
        }

        function resetToSchoolCenter() {
            const dropdown = document.getElementById('schoolZoneSelect');
            const searchInput = document.getElementById('searchInput');
            if (dropdown) dropdown.value = 'overview';
            if (searchInput) {
                searchInput.value = '';
                updateClearBtnVisibility();
            }
            loadZoneData('overview');
            showToast('กลับสู่จุดศูนย์กลางโรงเรียนบุญเหลือวิทยานุสรณ์แล้ว', 'school', 2000);
        }

        // =========================================================================
        // 4. FAST 1-CLICK GPS ENGINE, ANTI-DRIFT FILTER & IN-APP BROWSER HELPER
        // =========================================================================
        function detectInAppBrowserName() {
            const ua = navigator.userAgent || navigator.vendor || '';
            if (/GSA\/|GoogleApp/i.test(ua)) return 'แอป Google (Google App)';
            if (/Line\//i.test(ua)) return 'แอป LINE';
            if (/FBAN|FBAV|FB_IAB/i.test(ua)) return 'แอป Facebook';
            if (/Instagram/i.test(ua)) return 'แอป Instagram';
            if (/BytedanceWebview|musical_ly/i.test(ua)) return 'แอป TikTok';
            if (/; wv\)|WebView/i.test(ua)) return 'แอปเบราว์เซอร์ (Android WebView)';
            if (/(iPhone|iPod|iPad).*AppleWebKit(?!.*Safari)/i.test(ua)) return 'แอปเบราว์เซอร์ (iOS WebView)';
            return null;
        }

        function isInAppBrowser() {
            return detectInAppBrowserName() !== null;
        }

        async function initGpsPermissionMonitor() {
            const banner = document.getElementById('quickPermissionBanner');
            const statusDot = document.getElementById('gpsPermStatusDot');
            const inAppCopyBtn = document.getElementById('inAppCopyBtn');
            const inAppBadge = document.getElementById('inAppBadge');
            const appName = detectInAppBrowserName();

            if (appName && banner) {
                banner.classList.remove('hidden');
                const titleEl = document.getElementById('permBannerTitle');
                const descEl = document.getElementById('permBannerDesc');
                if (inAppBadge) {
                    inAppBadge.textContent = appName;
                    inAppBadge.classList.remove('hidden');
                }
                if (titleEl) titleEl.textContent = `กำลังเปิดผ่าน ${appName}`;
                if (descEl) descEl.textContent = 'กดคัดลอกลิงก์เพื่อนำไปเปิดใน Chrome หรือ Safari สำหรับใช้งาน GPS เต็มรูปแบบ หรือกดยืนยันตำแหน่งได้ทันที';
                if (inAppCopyBtn) inAppCopyBtn.classList.remove('hidden');
            }

            if (navigator.permissions && typeof navigator.permissions.query === 'function') {
                try {
                    const status = await navigator.permissions.query({ name: 'geolocation' });
                    const applyState = (state) => {
                        if (statusDot) {
                            statusDot.className = state === 'granted'
                                ? 'w-2 h-2 rounded-full bg-emerald-500'
                                : state === 'denied'
                                ? 'w-2 h-2 rounded-full bg-rose-500'
                                : 'w-2 h-2 rounded-full bg-amber-400';
                        }
                        if (banner && !appName) {
                            banner.classList.toggle('hidden', state === 'granted');
                        }
                    };
                    applyState(status.state);
                    status.onchange = () => applyState(status.state);
                } catch (e) {
                    if (banner) banner.classList.remove('hidden');
                }
            } else if (banner) {
                banner.classList.remove('hidden');
            }
        }

        function requestQuickLocationPermission() {
            useCurrentLocation(false);
        }

        function copyAppUrlForExternalBrowser() {
            copyTextSafe(window.location.href, 'คัดลอกลิงก์เว็บแล้ว สามารถนำไปวางเปิดใน Chrome หรือ Safari ได้เลย');
        }

        function haversineMeters(lat1, lon1, lat2, lon2) {
            const R = 6371000;
            const dLat = (lat2 - lat1) * Math.PI / 180;
            const dLon = (lon2 - lon1) * Math.PI / 180;
            const a = Math.sin(dLat / 2) * Math.sin(dLat / 2) +
                Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
                Math.sin(dLon / 2) * Math.sin(dLon / 2);
            return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
        }

        function filterGpsCoordinates(coords, timestamp) {
            const lat = Number(coords.latitude);
            const lon = Number(coords.longitude);
            const acc = Number(coords.accuracy) || 35;
            const ageMs = timestamp ? (Date.now() - timestamp) : 0;

            if (ageMs > 20000 && lastGoodGpsFix) return lastGoodGpsFix;
            if (!lastGoodGpsFix) {
                lastGoodGpsFix = { lat, lon, accuracy: acc, time: Date.now() };
                return lastGoodGpsFix;
            }

            const distM = haversineMeters(lastGoodGpsFix.lat, lastGoodGpsFix.lon, lat, lon);
            if (distM > 400 && acc > Math.max(100, lastGoodGpsFix.accuracy * 2.2)) return lastGoodGpsFix;
            if (distM < 6 && acc >= lastGoodGpsFix.accuracy) return lastGoodGpsFix;

            const wPrev = 1 / Math.pow(Math.max(5, lastGoodGpsFix.accuracy), 2);
            const wCurr = 1 / Math.pow(Math.max(5, acc), 2);
            const totalW = wPrev + wCurr;

            lastGoodGpsFix = {
                lat: (lastGoodGpsFix.lat * wPrev + lat * wCurr) / totalW,
                lon: (lastGoodGpsFix.lon * wPrev + lon * wCurr) / totalW,
                accuracy: Math.min(acc, (lastGoodGpsFix.accuracy + acc) / 2),
                time: Date.now()
            };
            return lastGoodGpsFix;
        }

        function useCurrentLocation(silent = false) {
            if (!navigator.geolocation) {
                if (!silent) showToast('เบราว์เซอร์นี้ไม่รองรับการระบุตำแหน่ง GPS', 'alert-circle', 3000);
                return;
            }
            if (isLocatingGPS) return;
            isLocatingGPS = true;

            const headerIcon = document.getElementById('gpsHeaderIcon');
            if (headerIcon) headerIcon.classList.add('animate-spin');
            if (!silent) showToast('กำลังจับพิกัด GPS ของคุณ...', 'crosshair', 2500);

            let gotFirstFix = false;

            const handleFix = (pos, isHighAcc) => {
                const filtered = filterGpsCoordinates(pos.coords, pos.timestamp);
                const statusDot = document.getElementById('gpsPermStatusDot');
                const banner = document.getElementById('quickPermissionBanner');
                const strip = document.getElementById('liveGpsTelemetryStrip');

                if (statusDot) statusDot.className = 'w-2 h-2 rounded-full bg-emerald-500';
                if (banner && !isInAppBrowser()) banner.classList.add('hidden');
                if (strip) {
                    strip.classList.remove('hidden');
                    strip.classList.add('flex');
                }

                updateUserGpsDot(filtered.lat, filtered.lon, filtered.accuracy, pos.coords.speed || 0);

                if (!gotFirstFix || isHighAcc) {
                    gotFirstFix = true;
                    inspectCustomSchoolCoordinate(filtered.lat, filtered.lon, true);
                    if (!silent && isHighAcc) {
                        const distFromSchool = Math.round(haversineMeters(SCHOOL_CENTER.lat, SCHOOL_CENTER.lon, filtered.lat, filtered.lon));
                        const distNote = distFromSchool <= 350
                            ? `อยู่ในบริเวณโรงเรียน (ห่างศูนย์กลาง ${distFromSchool} ม.)`
                            : `ห่างจากโรงเรียน ${(distFromSchool / 1000).toFixed(2)} กม.`;
                        showToast(`ปักหมุดตำแหน่งแล้ว • ${distNote}`, 'check-circle', 3000);
                    }
                }

                if (isHighAcc) {
                    isLocatingGPS = false;
                    if (headerIcon) headerIcon.classList.remove('animate-spin');
                }
            };

            const handleError = (err, isHighAcc) => {
                if (!isHighAcc) return;
                isLocatingGPS = false;
                if (headerIcon) headerIcon.classList.remove('animate-spin');
                if (gotFirstFix) return;

                const statusDot = document.getElementById('gpsPermStatusDot');
                if (err && err.code === 1) {
                    if (statusDot) statusDot.className = 'w-2 h-2 rounded-full bg-rose-500';
                    if (!silent) {
                        const modal = document.getElementById('gpsHelpModal');
                        if (modal) modal.classList.remove('hidden');
                    }
                } else if (!silent) {
                    showToast('สัญญาณ GPS อ่อน กรุณาลองใหม่หรือคลิกเลือกจุดบนแผนที่โรงเรียน', 'alert-circle', 3200);
                }
            };

            navigator.geolocation.getCurrentPosition(
                (pos) => handleFix(pos, false),
                (err) => handleError(err, false),
                { enableHighAccuracy: false, timeout: 3500, maximumAge: 15000 }
            );

            navigator.geolocation.getCurrentPosition(
                (pos) => handleFix(pos, true),
                (err) => handleError(err, true),
                { enableHighAccuracy: true, timeout: 12000, maximumAge: 0 }
            );
        }

        function recalibrateGpsNow() {
            lastGoodGpsFix = null;
            showToast('กำลังปรับเทียบพิกัด GPS ใหม่...', 'refresh-cw', 2000);
            useCurrentLocation(false);
        }

        function toggleLiveGpsTracking() {
            if (!navigator.geolocation) {
                showToast('อุปกรณ์นี้ไม่รองรับโหมดติดตามพิกัดสด', 'alert-circle', 2800);
                return;
            }

            const btn = document.getElementById('liveTrackBtn');
            const dot = document.getElementById('liveTrackDot');
            const label = document.getElementById('liveTrackLabel');
            const strip = document.getElementById('liveGpsTelemetryStrip');

            if (isLiveTracking) {
                isLiveTracking = false;
                if (gpsWatchId !== null) {
                    navigator.geolocation.clearWatch(gpsWatchId);
                    gpsWatchId = null;
                }
                if (btn) btn.classList.remove('ring-2', 'ring-sky-500', 'bg-sky-50', 'dark:bg-sky-500/20');
                if (dot) dot.className = 'w-2 h-2 rounded-full bg-slate-400';
                if (label) label.textContent = 'ติดตามสด';
                showToast('ปิดโหมดติดตามตำแหน่งสดแล้ว', 'info', 2000);
                return;
            }

            isLiveTracking = true;
            if (btn) btn.classList.add('ring-2', 'ring-sky-500', 'bg-sky-50', 'dark:bg-sky-500/20');
            if (dot) dot.className = 'w-2 h-2 rounded-full bg-emerald-500 animate-ping';
            if (label) label.textContent = 'กำลังติดตาม';
            if (strip) {
                strip.classList.remove('hidden');
                strip.classList.add('flex');
            }

            showToast('เปิดโหมดติดตามพิกัดสดแล้ว', 'navigation', 2400);

            gpsWatchId = navigator.geolocation.watchPosition(
                (pos) => {
                    const filtered = filterGpsCoordinates(pos.coords, pos.timestamp);
                    updateUserGpsDot(filtered.lat, filtered.lon, filtered.accuracy, pos.coords.speed || 0);

                    if (!lastTrackedCoords || haversineMeters(lastTrackedCoords.lat, lastTrackedCoords.lon, filtered.lat, filtered.lon) > 25) {
                        lastTrackedCoords = { lat: filtered.lat, lon: filtered.lon };
                        inspectCustomSchoolCoordinate(filtered.lat, filtered.lon, true);
                    }
                },
                (err) => {
                    if (err && err.code === 1) {
                        toggleLiveGpsTracking();
                        const modal = document.getElementById('gpsHelpModal');
                        if (modal) modal.classList.remove('hidden');
                    }
                },
                { enableHighAccuracy: true, maximumAge: 2000, timeout: 10000 }
            );
        }

        function updateUserGpsDot(lat, lon, accuracy = 20, speedMps = 0) {
            if (!map || typeof L === 'undefined') return;
            const numLat = Number(lat);
            const numLon = Number(lon);
            if (!Number.isFinite(numLat) || !Number.isFinite(numLon)) return;

            const safeAccuracy = Math.max(5, Math.min(Number(accuracy) || 20, 1000));

            if (!userLocationMarker) {
                const blueDotIcon = L.divIcon({
                    className: 'gps-blue-dot-wrapper',
                    html: `<div style="position:relative;width:22px;height:22px;display:flex;align-items:center;justify-content:center;">
                            <span class="gps-pulse-ring" style="position:absolute;width:20px;height:20px;border-radius:9999px;background:rgba(59,130,246,0.4);"></span>
                            <span style="position:relative;width:13px;height:13px;border-radius:9999px;background:#2563eb;border:2.5px solid #ffffff;box-shadow:0 2px 6px rgba(0,0,0,0.35);"></span>
                           </div>`,
                    iconSize: [22, 22],
                    iconAnchor: [11, 11]
                });
                userLocationMarker = L.marker([numLat, numLon], { icon: blueDotIcon, zIndexOffset: 1000 }).addTo(map);
            } else {
                userLocationMarker.setLatLng([numLat, numLon]);
            }

            if (!userAccuracyCircle) {
                userAccuracyCircle = L.circle([numLat, numLon], {
                    radius: safeAccuracy,
                    color: '#3b82f6',
                    fillColor: '#3b82f6',
                    fillOpacity: 0.14,
                    weight: 1.5
                }).addTo(map);
            } else {
                userAccuracyCircle.setLatLng([numLat, numLon]);
                userAccuracyCircle.setRadius(safeAccuracy);
            }

            const qualityBadge = document.getElementById('gpsSignalQualityBadge');
            if (qualityBadge) qualityBadge.textContent = `±${Math.round(safeAccuracy)} ม.`;

            const speedEl = document.getElementById('liveTelemetrySpeed');
            if (speedEl) {
                const kmh = Number.isFinite(speedMps) && speedMps > 0 ? Math.round(speedMps * 3.6) : 0;
                speedEl.textContent = `${kmh} km/h`;
            }
        }

        function closeGpsHelpModal() {
            const modal = document.getElementById('gpsHelpModal');
            if (modal) modal.classList.add('hidden');
        }

        // =========================================================================
        // 5. SCHOOL ZONE DROPDOWN & LOCAL AUTOCOMPLETE SEARCH
        // =========================================================================
        function populateSchoolZoneDropdown() {
            const select = document.getElementById('schoolZoneSelect');
            if (!select) return;

            select.innerHTML = '';
            SCHOOL_ZONES.forEach(zone => {
                const opt = document.createElement('option');
                opt.value = zone.id;
                opt.textContent = zone.shortName;
                select.appendChild(opt);
            });
            select.value = 'overview';
        }

        function selectZoneByDropdown() {
            const select = document.getElementById('schoolZoneSelect');
            if (!select || !select.value) return;
            loadZoneData(select.value);
        }

        function updateClearBtnVisibility() {
            const input = document.getElementById('searchInput');
            const clearBtn = document.getElementById('clearSearchBtn');
            if (input && clearBtn) {
                clearBtn.classList.toggle('hidden', !input.value.trim());
            }
        }

        function clearSearchInput() {
            const input = document.getElementById('searchInput');
            const dropdown = document.getElementById('autocompleteDropdown');
            if (input) {
                input.value = '';
                input.focus();
            }
            if (dropdown) dropdown.classList.add('hidden');
            updateClearBtnVisibility();
        }

        function handleSearchInput(val) {
            updateClearBtnVisibility();
            const dropdown = document.getElementById('autocompleteDropdown');
            if (!dropdown) return;

            const query = (val || '').trim().toLowerCase();
            if (!query) {
                dropdown.classList.add('hidden');
                return;
            }

            const matches = [];
            SCHOOL_ZONES.forEach(z => {
                const hit = z.title.toLowerCase().includes(query) ||
                    z.shortName.toLowerCase().includes(query) ||
                    z.keywords.some(k => k.toLowerCase().includes(query));
                if (hit) {
                    matches.push({
                        kind: 'zone',
                        zoneId: z.id,
                        title: z.title,
                        sub: `โรงเรียนบุญเหลือวิทยานุสรณ์ • พิกัด ${z.lat.toFixed(4)}, ${z.lon.toFixed(4)}`,
                        badge: 'โซนในโรงเรียน'
                    });
                }
            });

            Object.entries(LOCAL_SCHOOL_POIS).forEach(([cat, items]) => {
                items.forEach(poi => {
                    if (poi.name.toLowerCase().includes(query) || poi.detail.toLowerCase().includes(query)) {
                        matches.push({
                            kind: 'poi',
                            lat: poi.lat,
                            lon: poi.lon,
                            title: poi.name,
                            sub: poi.detail,
                            badge: POI_META[cat]?.label || 'จุดสำคัญ'
                        });
                    }
                });
            });

            currentSuggestions = matches.slice(0, 7);
            if (currentSuggestions.length === 0) {
                dropdown.classList.add('hidden');
                return;
            }

            dropdown.innerHTML = currentSuggestions.map((item, idx) => `
                <button type="button" onclick="selectAutocompleteItem(${idx})" class="w-full px-4 py-2.5 text-left hover:bg-slate-50 dark:hover:bg-slate-800/80 flex items-center justify-between gap-2 transition">
                    <div class="min-w-0">
                        <p class="text-xs sm:text-sm font-medium text-slate-900 dark:text-slate-100 truncate">${escapeHtml(item.title)}</p>
                        <p class="text-[11px] text-slate-500 dark:text-slate-400 truncate">${escapeHtml(item.sub)}</p>
                    </div>
                    <span class="text-[10px] px-2 py-0.5 rounded-full bg-sky-500/10 text-sky-700 dark:text-sky-300 shrink-0">${escapeHtml(item.badge)}</span>
                </button>
            `).join('');
            dropdown.classList.remove('hidden');
        }

        function selectAutocompleteItem(idx) {
            const item = currentSuggestions[idx];
            const dropdown = document.getElementById('autocompleteDropdown');
            const input = document.getElementById('searchInput');
            if (dropdown) dropdown.classList.add('hidden');
            if (!item) return;

            if (input) input.value = item.title;
            updateClearBtnVisibility();

            if (item.kind === 'zone') {
                loadZoneData(item.zoneId);
            } else {
                inspectCustomSchoolCoordinate(item.lat, item.lon, false, item.title);
            }
        }

        function performSearch() {
            const input = document.getElementById('searchInput');
            const dropdown = document.getElementById('autocompleteDropdown');
            if (dropdown) dropdown.classList.add('hidden');

            const query = input ? input.value.trim().toLowerCase() : '';
            if (!query) {
                loadZoneData('overview');
                return;
            }

            const matchedZone = SCHOOL_ZONES.find(z =>
                z.title.toLowerCase().includes(query) ||
                z.shortName.toLowerCase().includes(query) ||
                z.keywords.some(k => k.toLowerCase().includes(query))
            );
            if (matchedZone) {
                loadZoneData(matchedZone.id);
                return;
            }

            for (const items of Object.values(LOCAL_SCHOOL_POIS)) {
                const matchedPoi = items.find(p => p.name.toLowerCase().includes(query) || p.detail.toLowerCase().includes(query));
                if (matchedPoi) {
                    inspectCustomSchoolCoordinate(matchedPoi.lat, matchedPoi.lon, false, matchedPoi.name);
                    return;
                }
            }

            showToast('ไม่พบชื่อโซนที่ค้นหา กรุณาเลือกจากรายการจุดสำคัญในโรงเรียนด้านล่าง', 'info', 3000);
        }

        // =========================================================================
        // 6. CAMPUS ZONE ANALYSIS & REAL-TIME WEATHER / PM2.5 ENGINE
        // =========================================================================
        function findNearestSchoolZone(lat, lon) {
            let closest = SCHOOL_ZONES[0];
            let minDist = Infinity;
            for (const z of SCHOOL_ZONES) {
                const d = haversineMeters(lat, lon, z.lat, z.lon);
                if (d < minDist) {
                    minDist = d;
                    closest = z;
                }
            }
            return { zone: closest, distMeters: Math.round(minDist) };
        }

        function inspectCustomSchoolCoordinate(lat, lon, isFromGps = false, customLabel = null) {
            const numLat = Number(lat);
            const numLon = Number(lon);
            if (!Number.isFinite(numLat) || !Number.isFinite(numLon)) return;

            const { zone, distMeters } = findNearestSchoolZone(numLat, numLon);
            const distFromCenter = Math.round(haversineMeters(SCHOOL_CENTER.lat, SCHOOL_CENTER.lon, numLat, numLon));

            let title = customLabel;
            if (!title) {
                if (distFromCenter <= 350) {
                    title = distMeters <= 35
                        ? zone.title
                        : `จุดตรวจสอบใกล้${zone.shortName} (ห่าง ${distMeters} ม.)`;
                } else {
                    title = `จุดภายนอกโรงเรียน (ห่าง รร.บุญเหลือวิทยานุสรณ์ ${(distFromCenter / 1000).toFixed(2)} กม.)`;
                }
            }

            const customZone = {
                ...zone,
                id: distMeters <= 35 ? zone.id : `custom_${numLat.toFixed(5)}_${numLon.toFixed(5)}`,
                title,
                lat: numLat,
                lon: numLon,
                distFromSchoolCenter: distFromCenter
            };

            analyzeSchoolZone(customZone, { zoom: isFromGps ? 17 : 18 });
        }

        function loadZoneData(zoneId, options = {}) {
            const found = SCHOOL_ZONES.find(z => z.id === zoneId) || SCHOOL_ZONES[0];
            const dropdown = document.getElementById('schoolZoneSelect');
            if (dropdown) dropdown.value = found.id;
            analyzeSchoolZone({
                ...found,
                distFromSchoolCenter: Math.round(haversineMeters(SCHOOL_CENTER.lat, SCHOOL_CENTER.lon, found.lat, found.lon))
            }, options);
        }

        function updateMapPosition(lat, lon, zoom = 17, animate = true) {
            if (!map || typeof L === 'undefined') return;
            const numLat = Number(lat);
            const numLon = Number(lon);
            if (!Number.isFinite(numLat) || !Number.isFinite(numLon)) return;

            try {
                map.invalidateSize(false);
                const size = map.getSize();
                if (!size || size.x === 0 || size.y === 0 || !animate) {
                    map.setView([numLat, numLon], zoom, { animate: false });
                } else {
                    map.flyTo([numLat, numLon], zoom, { duration: 0.7 });
                }

                if (marker) marker.setLatLng([numLat, numLon]);
                if (zoneRadiusCircle) zoneRadiusCircle.setLatLng([numLat, numLon]);
            } catch (err) {}
        }

        function analyzeSchoolZone(zone, options = {}) {
            if (!zone) return;
            const lat = Number.isFinite(Number(zone.lat)) ? Number(zone.lat) : SCHOOL_CENTER.lat;
            const lon = Number.isFinite(Number(zone.lon)) ? Number(zone.lon) : SCHOOL_CENTER.lon;

            currentZone = { ...zone, lat, lon };
            updateMapPosition(lat, lon, options.zoom || 17, !options.initial);

            const hour = new Date().getHours();
            const isSchoolRush = (hour >= 7 && hour <= 8) || (hour >= 15 && hour <= 17);

            const s = { ...zone.scores };
            if (isSchoolRush && (zone.zoneType === 'gate_road' || zone.zoneType === 'perimeter_road')) {
                s.transit = Math.max(70, s.transit - 8);
            }

            const overall = Math.round((s.safety * 0.28) + (s.env * 0.20) + (s.infra * 0.20) + (s.transit * 0.16) + (s.green * 0.16));

            // Header & Overall Score Card
            const locTitleEl = document.getElementById('locTitle');
            const locSubEl = document.getElementById('locSub');
            const overallScoreEl = document.getElementById('overallScore');
            const livabilityGradeEl = document.getElementById('livabilityGrade');
            const riskBadgeEl = document.getElementById('riskBadge');

            if (locTitleEl) locTitleEl.textContent = zone.title;
            if (locSubEl) {
                const distText = Number.isFinite(zone.distFromSchoolCenter) && zone.distFromSchoolCenter > 0
                    ? ` • ห่างจุดศูนย์กลางโรงเรียน ${zone.distFromSchoolCenter} ม.`
                    : '';
                locSubEl.textContent = `${SCHOOL_CENTER.subdistrict}${distText} • พิกัด: ${lat.toFixed(5)}, ${lon.toFixed(5)}`;
            }
            if (overallScoreEl) overallScoreEl.textContent = overall;

            let gradeText = 'น่าเรียนและปลอดภัยดี (B+)';
            if (overall >= 88) gradeText = 'น่าเรียนและปลอดภัยดีเยี่ยม (A+)';
            else if (overall >= 80) gradeText = 'น่าเรียนและปลอดภัยดีมาก (A)';
            if (livabilityGradeEl) livabilityGradeEl.textContent = gradeText;

            if (riskBadgeEl) {
                if (overall >= 82) {
                    riskBadgeEl.textContent = 'ปลอดภัย / เสี่ยงต่ำ';
                    riskBadgeEl.className = 'px-3 py-1 rounded-full text-xs font-bold bg-emerald-500/15 text-emerald-700 dark:text-emerald-400 border border-emerald-500/30 whitespace-nowrap';
                } else {
                    riskBadgeEl.textContent = 'เฝ้าระวังการจราจร';
                    riskBadgeEl.className = 'px-3 py-1 rounded-full text-xs font-bold bg-amber-500/15 text-amber-700 dark:text-amber-400 border border-amber-500/30 whitespace-nowrap';
                }
            }

            // Mini Indicators
            const quickSafetyEl = document.getElementById('quickSafety');
            const quickEnvEl = document.getElementById('quickEnv');
            const quickConvEl = document.getElementById('quickConvenience');
            if (quickSafetyEl) quickSafetyEl.textContent = `${s.safety >= 85 ? 'ดีมาก' : 'ดี'} (${s.safety}%)`;
            if (quickEnvEl) quickEnvEl.textContent = `${s.env >= 82 ? 'ดีมาก' : 'ปานกลาง'} (${s.env}%)`;
            if (quickConvEl) quickConvEl.textContent = `${s.infra >= 88 ? 'ดีเยี่ยม' : 'ดี'} (${s.infra}%)`;

            // Dimension Header Scores
            const setTxt = (id, txt) => { const el = document.getElementById(id); if (el) el.textContent = txt; };
            const setW = (id, pct) => { const el = document.getElementById(id); if (el) el.style.width = `${pct}%`; };

            setTxt('scoreSafety', `${s.safety}/100`);
            setTxt('scoreEnv', `${s.env}/100`);
            setTxt('scoreInfra', `${s.infra}/100`);
            setTxt('scoreTransit', `${s.transit}/100`);
            setTxt('scoreGreen', `${s.green}/100`);

            // Dimension 1
            setTxt('cctvStatus', zone.cctv);
            setW('cctvBar', s.safety);
            setTxt('crossingStatus', zone.crossing);
            setW('crossingBar', zone.zoneType === 'perimeter_road' ? 78 : 90);
            setW('lightBar', 88);

            // Dimension 2
            setTxt('floodStatus', zone.flood);
            setW('floodBar', zone.zoneType === 'perimeter_road' ? 74 : 88);

            // Dimension 3
            setTxt('hospitalDist', zone.hospital);
            setW('hospitalBar', s.infra);
            setTxt('policeDist', zone.police);
            setW('policeBar', 90);

            // Dimension 4
            setTxt('transitAccess', zone.transit);
            setW('transitBar', s.transit);
            setTxt('trafficStatus', isSchoolRush ? 'หนาแน่น (ช่วงรับ-ส่งนักเรียน)' : 'คล่องตัวปานกลาง');
            setW('trafficBar', isSchoolRush ? 62 : 82);
            setTxt('walkStatus', zone.walk);
            setW('walkBar', zone.zoneType === 'perimeter_road' ? 76 : 90);

            // Dimension 5
            setTxt('greenSpaceDesc', zone.greenDesc);

            // Update Satellite HUD Coordinates
            const satCoordsEl = document.getElementById('satLiveCoords');
            if (satCoordsEl) {
                satCoordsEl.textContent = `${lat.toFixed(5)}, ${lon.toFixed(5)}`;
            }

            // Update Google Maps & Street View Links
            const gmUrl = `https://www.google.com/maps/search/?api=1&query=${lat},${lon}`;
            const svUrl = `https://www.google.com/maps/@?api=1&map_action=pano&viewpoint=${lat},${lon}`;
            ['googleMapsLink', 'mapOverlayGoogleMaps'].forEach(id => {
                const el = document.getElementById(id);
                if (el) el.href = gmUrl;
            });
            ['streetViewLink', 'mapOverlayStreetView'].forEach(id => {
                const el = document.getElementById(id);
                if (el) el.href = svUrl;
            });

            renderCommunitySection(currentZone);
            fetchRealTimeEnvironment(lat, lon, currentZone);
        }

        function renderAlerts(zone, live = {}) {
            const container = document.getElementById('alertBoxList');
            if (!container) return;

            const pm = Number(live.pmRounded ?? 23.5);
            const rain = Number(live.rainMm ?? 0);
            const temp = Number(live.tempC ?? 31);
            const alerts = [];

            if (rain > 0) {
                alerts.push({
                    type: 'amber',
                    icon: 'cloud-rain',
                    title: `ตรวจพบฝนตกบริเวณโรงเรียน (${rain} มม./ชม.)`,
                    desc: 'แนะนำให้นักเรียนหลบฝนในอาคารเรียนหรือหอประชุม และระวังพื้นลื่นบริเวณทางเดินและทางม้าลาย'
                });
            } else {
                alerts.push({
                    type: 'emerald',
                    icon: 'sun',
                    title: `สภาพอากาศในโรงเรียนปลอดโปร่ง (${Math.round(temp)}°C)`,
                    desc: 'ไม่พบฝนตกในเขตโรงเรียนบุญเหลือวิทยานุสรณ์ สามารถเรียนและทำกิจกรรมได้ตามปกติ'
                });
            }

            if (pm > 37.5) {
                alerts.push({
                    type: 'rose',
                    icon: 'wind',
                    title: `ค่าฝุ่น PM2.5 สูงกว่าเกณฑ์มาตรฐาน (${pm} µg/m³)`,
                    desc: 'แนะนำให้สวมหน้ากากอนามัยป้องกันฝุ่น PM2.5 และลดกิจกรรมกลางแจ้งที่สนามกีฬา'
                });
            } else {
                alerts.push({
                    type: 'sky',
                    icon: 'wind',
                    title: `คุณภาพอากาศในโรงเรียนอยู่ในเกณฑ์ดี (${pm} µg/m³)`,
                    desc: 'ค่าฝุ่น PM2.5 อยู่ในระดับปลอดภัยสำหรับกิจกรรมหน้าเสาธงและวิชาพลศึกษา'
                });
            }

            alerts.push({
                type: 'indigo',
                icon: 'info',
                title: `คำแนะนำเฉพาะจุด: ${zone ? zone.shortName : 'โรงเรียนบุญเหลือวิทยานุสรณ์'}`,
                desc: zone ? zone.tip : 'เขตปลอดภัยรอบสถานศึกษารัศมี 300 เมตร ครอบคลุมแนว ถ.สุรนารายณ์'
            });

            const colorStyles = {
                amber: 'bg-amber-50/80 dark:bg-amber-500/10 border-amber-200/80 dark:border-amber-500/20 text-amber-800 dark:text-amber-300',
                rose: 'bg-rose-50/80 dark:bg-rose-500/10 border-rose-200/80 dark:border-rose-500/20 text-rose-800 dark:text-rose-300',
                emerald: 'bg-emerald-50/80 dark:bg-emerald-500/10 border-emerald-200/80 dark:border-emerald-500/20 text-emerald-800 dark:text-emerald-300',
                sky: 'bg-sky-50/80 dark:bg-sky-500/10 border-sky-200/80 dark:border-sky-500/20 text-sky-800 dark:text-sky-300',
                indigo: 'bg-indigo-50/80 dark:bg-indigo-500/10 border-indigo-200/80 dark:border-indigo-500/20 text-indigo-800 dark:text-indigo-300'
            };

            container.innerHTML = alerts.map(a => `
                <div class="p-3 rounded-xl border ${colorStyles[a.type] || colorStyles.sky} flex items-start gap-3">
                    <i data-lucide="${escapeHtml(a.icon)}" class="w-4 h-4 mt-0.5 shrink-0"></i>
                    <div class="text-xs">
                        <p class="font-semibold">${escapeHtml(a.title)}</p>
                        <p class="opacity-85 mt-0.5">${escapeHtml(a.desc)}</p>
                    </div>
                </div>
            `).join('');

            refreshIcons();
        }

        function startRealTimeEngine() {
            if (liveClockTimer) clearInterval(liveClockTimer);
            if (autoSyncTimer) clearInterval(autoSyncTimer);

            const updateClock = () => {
                const now = new Date();
                const timeStr = now.toLocaleTimeString('th-TH', {
                    hour: '2-digit',
                    minute: '2-digit',
                    second: '2-digit'
                }) + ' น.';
                const dateStr = now.toLocaleDateString('th-TH', {
                    day: '2-digit',
                    month: 'short',
                    year: 'numeric'
                });

                const clockEl = document.getElementById('liveClockText');
                if (clockEl) clockEl.textContent = timeStr;

                const satClockEl = document.getElementById('satLiveClock');
                if (satClockEl) satClockEl.textContent = `${dateStr} ${timeStr}`;
            };
            updateClock();
            liveClockTimer = setInterval(updateClock, 1000);

            autoSyncTimer = setInterval(() => {
                if (!autoRefreshEnabled) return;
                nextRefreshSeconds -= 1;
                const countdownEl = document.getElementById('autoSyncCountdown');
                if (nextRefreshSeconds <= 0) {
                    nextRefreshSeconds = 30;
                    if (countdownEl) countdownEl.textContent = 'กำลังซิงค์ข้อมูล...';
                    if (currentZone) {
                        fetchRealTimeEnvironment(currentZone.lat, currentZone.lon, currentZone);
                    }
                    if (isLiveRadarActive) {
                        refreshLiveRadarLayer(true);
                    }
                } else if (countdownEl) {
                    countdownEl.textContent = `ซิงค์อัตโนมัติใน ${nextRefreshSeconds} วิ`;
                }
            }, 1000);
        }

        async function manualRefreshNow() {
            const icon = document.getElementById('manualRefreshIcon');
            if (icon) icon.classList.add('animate-spin');
            nextRefreshSeconds = 30;
            if (currentZone) {
                await fetchRealTimeEnvironment(currentZone.lat, currentZone.lon, currentZone);
                showToast('อัปเดตสภาพอากาศและค่าฝุ่น PM2.5 ในโรงเรียนเรียบร้อยแล้ว', 'check-circle', 2200);
            }
            setTimeout(() => {
                if (icon) icon.classList.remove('animate-spin');
            }, 600);
        }

        async function fetchRealTimeEnvironment(lat, lon, zone) {
            const numLat = Number(lat) || SCHOOL_CENTER.lat;
            const numLon = Number(lon) || SCHOOL_CENTER.lon;

            let pm25 = null;
            let tempC = null;
            let isDay = null;
            let rainMm = 0;
            let humidity = null;
            let windKmh = null;
            let elevationM = null;

            try {
                const weatherUrl = `https://api.open-meteo.com/v1/forecast?latitude=${numLat}&longitude=${numLon}&current=temperature_2m,is_day,relative_humidity_2m,precipitation,rain,wind_speed_10m&timezone=Asia%2FBangkok`;
                const airUrl = `https://air-quality-api.open-meteo.com/v1/air-quality?latitude=${numLat}&longitude=${numLon}&current=pm2_5,us_aqi&timezone=Asia%2FBangkok`;
                const elevUrl = `https://api.open-meteo.com/v1/elevation?latitude=${numLat}&longitude=${numLon}`;

                const [weatherRes, airRes, elevRes] = await Promise.allSettled([
                    fetch(weatherUrl),
                    fetch(airUrl),
                    fetch(elevUrl)
                ]);

                if (weatherRes.status === 'fulfilled' && weatherRes.value.ok) {
                    const wData = await weatherRes.value.json();
                    if (wData && wData.current) {
                        tempC = wData.current.temperature_2m;
                        isDay = wData.current.is_day;
                        humidity = wData.current.relative_humidity_2m;
                        rainMm = Number(wData.current.rain ?? wData.current.precipitation ?? 0);
                        windKmh = wData.current.wind_speed_10m;
                    }
                }

                if (airRes.status === 'fulfilled' && airRes.value.ok) {
                    const aData = await airRes.value.json();
                    if (aData && aData.current) {
                        pm25 = aData.current.pm2_5;
                    }
                }

                if (elevRes.status === 'fulfilled' && elevRes.value.ok) {
                    const eData = await elevRes.value.json();
                    if (Array.isArray(eData.elevation) && eData.elevation.length > 0) {
                        elevationM = Number(eData.elevation[0]);
                    }
                }
            } catch (e) {}

            if (pm25 === null || !Number.isFinite(Number(pm25))) pm25 = 23.5;
            if (tempC === null || !Number.isFinite(Number(tempC))) tempC = 31.0;

            const pmRounded = Math.round(Number(pm25) * 10) / 10;
            const pmStatusEl = document.getElementById('pmStatus');
            const pmBarEl = document.getElementById('pmBar');
            const mapCardPmEl = document.getElementById('mapCardPm');
            const activityStatusEl = document.getElementById('outdoorActivityStatus');
            const activityBarEl = document.getElementById('outdoorActivityBar');

            let pmLabel = 'ดีมาก';
            let pmScore = 92;
            let pmBarColor = 'bg-emerald-500 dark:bg-emerald-400';
            let pmTextColor = 'font-medium text-emerald-600 dark:text-emerald-400';

            if (pmRounded > 75) {
                pmLabel = 'มีผลต่อสุขภาพ';
                pmScore = 40;
                pmBarColor = 'bg-rose-500 dark:bg-rose-400';
                pmTextColor = 'font-medium text-rose-600 dark:text-rose-400';
            } else if (pmRounded > 37.5) {
                pmLabel = 'เริ่มมีผลกระทบ';
                pmScore = 62;
                pmBarColor = 'bg-amber-500 dark:bg-amber-400';
                pmTextColor = 'font-medium text-amber-600 dark:text-amber-400';
            } else if (pmRounded > 25) {
                pmLabel = 'ปานกลาง';
                pmScore = 78;
                pmBarColor = 'bg-sky-500 dark:bg-sky-400';
                pmTextColor = 'font-medium text-sky-600 dark:text-sky-400';
            }

            if (pmStatusEl) {
                pmStatusEl.textContent = `${pmRounded} µg/m³ (${pmLabel})`;
                pmStatusEl.className = pmTextColor;
            }
            if (pmBarEl) {
                pmBarEl.style.width = `${pmScore}%`;
                pmBarEl.className = `${pmBarColor} h-full rounded-full transition-all duration-500`;
            }
            if (mapCardPmEl) {
                mapCardPmEl.textContent = `PM2.5: ${pmRounded} µg/m³ • ${Math.round(tempC)}°C`;
            }

            // Outdoor School Activity Guidance (Flag Ceremony / PE Class)
            if (activityStatusEl && activityBarEl) {
                if (rainMm > 0) {
                    activityStatusEl.textContent = 'งดกิจกรรมกลางแจ้ง (มีฝนตก ใช้หอประชุม)';
                    activityStatusEl.className = 'font-medium text-amber-600 dark:text-amber-400';
                    activityBarEl.style.width = '55%';
                    activityBarEl.className = 'bg-amber-500 dark:bg-amber-400 h-full rounded-full';
                } else if (pmRounded > 37.5) {
                    activityStatusEl.textContent = 'ลดเวลาทำกิจกรรมกลางแจ้ง & สวมหน้ากาก';
                    activityStatusEl.className = 'font-medium text-amber-600 dark:text-amber-400';
                    activityBarEl.style.width = '60%';
                    activityBarEl.className = 'bg-amber-500 dark:bg-amber-400 h-full rounded-full';
                } else {
                    activityStatusEl.textContent = 'ทำกิจกรรมกลางแจ้งได้ตามปกติ';
                    activityStatusEl.className = 'font-medium text-emerald-600 dark:text-emerald-400';
                    activityBarEl.style.width = '92%';
                    activityBarEl.className = 'bg-emerald-500 dark:bg-emerald-400 h-full rounded-full';
                }
            }

            const elevEl = document.getElementById('elevationInfoText');
            if (elevEl) {
                const elevVal = Number.isFinite(elevationM) ? Math.round(elevationM * 10) / 10 : 178.5;
                elevEl.textContent = `${elevVal} ม. รทก. (พื้นที่ดอน ต.โคกสูง)`;
            }

            const liveTempEl = document.getElementById('liveWeatherTemp');
            if (liveTempEl) liveTempEl.textContent = `${Number(tempC).toFixed(1)}°C`;

            const liveRainEl = document.getElementById('liveWeatherRain');
            if (liveRainEl) liveRainEl.textContent = rainMm > 0 ? `ฝนตก ${rainMm} มม./ชม.` : 'ไม่มีฝน';

            const liveHumidityEl = document.getElementById('liveWeatherHumidity');
            if (liveHumidityEl && humidity !== null) liveHumidityEl.textContent = `ความชื้น ${Math.round(humidity)}%`;

            const liveWindEl = document.getElementById('liveWeatherWind');
            if (liveWindEl && windKmh !== null) liveWindEl.textContent = `ลม ${Math.round(windKmh)} กม./ชม.`;

            // Update Real-Time Day / Evening / Night Status on Satellite HUD
            const satDayNightEl = document.getElementById('satDayNightBadge');
            if (satDayNightEl) {
                const hr = new Date().getHours();
                if (isDay === 0 || (isDay === null && (hr >= 19 || hr < 6))) {
                    satDayNightEl.textContent = 'กลางคืน (Nighttime)';
                    satDayNightEl.className = 'inline-flex items-center gap-1 text-indigo-300 font-medium';
                } else if (hr >= 16 && hr <= 18) {
                    satDayNightEl.textContent = 'ช่วงเย็น / ใกล้ค่ำ';
                    satDayNightEl.className = 'inline-flex items-center gap-1 text-orange-300 font-medium';
                } else {
                    satDayNightEl.textContent = 'กลางวัน (แสงแดดจริง)';
                    satDayNightEl.className = 'inline-flex items-center gap-1 text-amber-300 font-medium';
                }
            }

            const lastUpdatedEl = document.getElementById('lastUpdatedText');
            if (lastUpdatedEl) {
                const nowStr = new Date().toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit', second: '2-digit' });
                lastUpdatedEl.textContent = `อัปเดตข้อมูลล่าสุด: วันนี้ ${nowStr} น.`;
            }

            renderAlerts(zone, { pmRounded, rainMm, tempC });
        }

        // =========================================================================
        // 7. COMMUNITY VOTING, RATING & REVIEWS ENGINE (LOCALSTORAGE PERSISTENCE)
        // =========================================================================
        let communityStore = {};
        let selectedStarRating = 5;
        let hoveredStarRating = 0;
        const selectedReviewTags = new Set();

        const STAR_LABELS = {
            1: '1 ดาว - ควรปรับปรุง',
            2: '2 ดาว - พอใช้',
            3: '3 ดาว - ปานกลาง',
            4: '4 ดาว - น่าเรียน/ปลอดภัยดี',
            5: '5 ดาว - ดีเยี่ยม/ปลอดภัยมาก'
        };

        function loadCommunityStore() {
            try {
                const raw = localStorage.getItem(STORAGE_KEY);
                if (raw) {
                    const parsed = JSON.parse(raw);
                    if (parsed && typeof parsed === 'object') {
                        communityStore = parsed;
                    }
                }
            } catch (e) {}
        }

        function saveCommunityStore() {
            try {
                localStorage.setItem(STORAGE_KEY, JSON.stringify(communityStore));
            } catch (e) {}
        }

        function getZoneStoreKey(zone) {
            return zone && zone.id ? zone.id : 'overview';
        }

        function ensureZoneCommunityData(zone) {
            const key = getZoneStoreKey(zone);
            if (!communityStore[key]) {
                const isRoad = zone && (zone.zoneType === 'gate_road' || zone.zoneType === 'perimeter_road');
                communityStore[key] = {
                    upVotes: isRoad ? 28 : 34,
                    downVotes: isRoad ? 5 : 2,
                    userVote: null,
                    reviews: [
                        {
                            author: 'ครูเวรประจำวัน',
                            stars: 5,
                            tags: isRoad ? ['ทางม้าลายชัดเจน', 'จุดรับ-ส่งปลอดภัย'] : ['ร่มรื่นน่าเรียน', 'โรงอาหารสะอาด'],
                            comment: isRoad
                                ? `บริเวณ${zone.shortName} มีครูเวรคอยอำนวยความสะดวกช่วงเช้าและเย็น ขอความร่วมมือผู้ปกครองจอดรับ-ส่งตามจุดที่กำหนด`
                                : `พื้นที่${zone ? zone.shortName : 'ภายในโรงเรียน'} สะอาด ปลอดภัย มีกล้อง CCTV และแสงสว่างเพียงพอตลอดวัน`,
                            time: 'ข้อมูลพื้นฐานโรงเรียน',
                            helpful: 8,
                            helpfulVoted: false
                        },
                        {
                            author: 'ตัวแทนสภานักเรียน',
                            stars: 4,
                            tags: isRoad ? ['ระวังรถเร็วหน้าโรงเรียน'] : ['ร่มรื่นน่าเรียน'],
                            comment: isRoad
                                ? 'ถนนสุรนารายณ์หน้าโรงเรียนมีรถวิ่งค่อนข้างเร็ว แนะนำให้เพื่อนๆ ข้ามทางม้าลายเป็นกลุ่มทุกครั้งครับ'
                                : 'บรรยากาศร่มรื่น น่าเรียน และเดินเชื่อมต่อระหว่างอาคารได้สะดวกมากครับ',
                            time: 'ข้อมูลพื้นฐานโรงเรียน',
                            helpful: 5,
                            helpfulVoted: false
                        }
                    ]
                };
                saveCommunityStore();
            }
            return communityStore[key];
        }

        function renderStarButtonsVisual(activeCount) {
            const displayCount = Math.max(1, Math.min(5, Number(activeCount) || 5));
            const labelEl = document.getElementById('starRatingLabel');
            if (labelEl) labelEl.textContent = STAR_LABELS[displayCount] || '';

            const container = document.getElementById('starRatingContainer');
            if (!container) return;

            container.querySelectorAll('.review-star-btn').forEach(btn => {
                const ratingVal = Number(btn.getAttribute('data-rating'));
                const svg = btn.querySelector('svg');
                const isSelected = ratingVal <= selectedStarRating;
                btn.setAttribute('aria-checked', ratingVal === selectedStarRating ? 'true' : 'false');

                if (!svg) return;
                if (ratingVal <= displayCount) {
                    svg.setAttribute('fill', '#f59e0b');
                    svg.setAttribute('stroke', '#f59e0b');
                    svg.style.transform = ratingVal === displayCount ? 'scale(1.12)' : 'scale(1)';
                } else {
                    svg.setAttribute('fill', 'none');
                    svg.setAttribute('stroke', '#94a3b8');
                    svg.style.transform = 'scale(1)';
                }
                btn.classList.toggle('bg-amber-500/10', isSelected);
            });
        }

        function setReviewStar(rating) {
            selectedStarRating = Math.max(1, Math.min(5, Number(rating) || 5));
            hoveredStarRating = 0;
            renderStarButtonsVisual(selectedStarRating);
        }

        function hoverReviewStar(rating) {
            hoveredStarRating = Math.max(1, Math.min(5, Number(rating) || 0));
            renderStarButtonsVisual(hoveredStarRating || selectedStarRating);
        }

        function resetReviewStarHover() {
            hoveredStarRating = 0;
            renderStarButtonsVisual(selectedStarRating);
        }

        function initStarRatingControls() {
            const container = document.getElementById('starRatingContainer');
            if (!container) return;

            container.addEventListener('click', (e) => {
                const btn = e.target.closest('.review-star-btn');
                if (!btn) return;
                e.preventDefault();
                const rating = Number(btn.getAttribute('data-rating'));
                if (rating >= 1 && rating <= 5) setReviewStar(rating);
            });

            container.addEventListener('mouseover', (e) => {
                const btn = e.target.closest('.review-star-btn');
                if (!btn) return;
                const rating = Number(btn.getAttribute('data-rating'));
                if (rating >= 1 && rating <= 5) hoverReviewStar(rating);
            });

            container.addEventListener('mouseleave', () => {
                resetReviewStarHover();
            });

            renderStarButtonsVisual(selectedStarRating);
        }

        function toggleReviewTag(btn, tag) {
            if (selectedReviewTags.has(tag)) {
                selectedReviewTags.delete(tag);
                btn.classList.remove('bg-sky-600', 'text-white', 'border-sky-600');
                btn.classList.add('bg-white', 'dark:bg-slate-800', 'text-slate-600', 'dark:text-slate-300');
            } else {
                selectedReviewTags.add(tag);
                btn.classList.remove('bg-white', 'dark:bg-slate-800', 'text-slate-600', 'dark:text-slate-300');
                btn.classList.add('bg-sky-600', 'text-white', 'border-sky-600');
            }
        }

        function voteAreaSentiment(direction) {
            if (!currentZone) return;
            const data = ensureZoneCommunityData(currentZone);

            if (data.userVote === direction) {
                if (direction === 'up') data.upVotes = Math.max(0, data.upVotes - 1);
                if (direction === 'down') data.downVotes = Math.max(0, data.downVotes - 1);
                data.userVote = null;
                showToast('ยกเลิกการโหวตจุดนี้แล้ว', 'info', 1800);
            } else {
                if (data.userVote === 'up') data.upVotes = Math.max(0, data.upVotes - 1);
                if (data.userVote === 'down') data.downVotes = Math.max(0, data.downVotes - 1);

                if (direction === 'up') data.upVotes += 1;
                if (direction === 'down') data.downVotes += 1;
                data.userVote = direction;
                showToast(
                    direction === 'up' ? 'บันทึกโหวต "ปลอดภัย / น่าเรียน" เรียบร้อยแล้ว' : 'บันทึกโหวต "ควรปรับปรุง/เฝ้าระวัง" เรียบร้อยแล้ว',
                    'check-circle',
                    2200
                );
            }

            saveCommunityStore();
            renderCommunitySection(currentZone);
        }

        function submitAreaReview() {
            if (!currentZone) return;
            const nameInput = document.getElementById('reviewerNameInput');
            const commentInput = document.getElementById('reviewCommentInput');

            const commentText = commentInput ? commentInput.value.trim() : '';
            if (!commentText) {
                showToast('กรุณาพิมพ์ความคิดเห็นสั้นๆ เกี่ยวกับโซนนี้ก่อนส่ง', 'alert-circle', 2500);
                if (commentInput) commentInput.focus();
                return;
            }

            const data = ensureZoneCommunityData(currentZone);
            const authorName = (nameInput && nameInput.value.trim()) ? nameInput.value.trim() : 'นักเรียน / ผู้ปกครอง';
            const nowStr = new Date().toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' }) + ' น.';

            const newReview = {
                author: authorName,
                stars: selectedStarRating,
                tags: Array.from(selectedReviewTags),
                comment: commentText,
                time: `วันนี้ ${nowStr}`,
                helpful: 1,
                helpfulVoted: true
            };

            if (selectedStarRating >= 4) data.upVotes += 1;
            else if (selectedStarRating <= 2) data.downVotes += 1;

            data.reviews.unshift(newReview);
            saveCommunityStore();

            if (commentInput) commentInput.value = '';
            selectedReviewTags.clear();
            document.querySelectorAll('.review-tag-chip').forEach(btn => {
                btn.classList.remove('bg-sky-600', 'text-white', 'border-sky-600');
                btn.classList.add('bg-white', 'dark:bg-slate-800', 'text-slate-600', 'dark:text-slate-300');
            });
            setReviewStar(5);

            renderCommunitySection(currentZone);
            showToast('บันทึกคะแนนและความคิดเห็นของคุณเรียบร้อยแล้ว!', 'check-circle', 2600);
        }

        function toggleReviewHelpful(idx) {
            if (!currentZone) return;
            const data = ensureZoneCommunityData(currentZone);
            const rev = data.reviews[idx];
            if (!rev) return;

            rev.helpfulVoted = !rev.helpfulVoted;
            rev.helpful = Math.max(0, (Number(rev.helpful) || 0) + (rev.helpfulVoted ? 1 : -1));
            saveCommunityStore();
            renderCommunitySection(currentZone);
        }

        function renderCommunitySection(zone) {
            if (!zone) return;
            const data = ensureZoneCommunityData(zone);

            const areaNameEl = document.getElementById('communityAreaName');
            if (areaNameEl) areaNameEl.textContent = zone.title;

            const upCountEl = document.getElementById('voteUpCount');
            const downCountEl = document.getElementById('voteDownCount');
            const upBtn = document.getElementById('voteUpBtn');
            const downBtn = document.getElementById('voteDownBtn');

            if (upCountEl) upCountEl.textContent = data.upVotes;
            if (downCountEl) downCountEl.textContent = data.downVotes;

            if (upBtn) {
                upBtn.classList.toggle('ring-2', data.userVote === 'up');
                upBtn.classList.toggle('ring-emerald-500', data.userVote === 'up');
            }
            if (downBtn) {
                downBtn.classList.toggle('ring-2', data.userVote === 'down');
                downBtn.classList.toggle('ring-rose-500', data.userVote === 'down');
            }

            const totalVotes = data.upVotes + data.downVotes;
            const totalReviews = data.reviews.length;
            const sumStars = data.reviews.reduce((acc, r) => acc + (Number(r.stars) || 5), 0);
            const avgStar = totalReviews > 0 ? (sumStars / totalReviews).toFixed(1) : '4.7';
            const positivePct = totalVotes > 0 ? Math.round((data.upVotes / totalVotes) * 100) : 91;
            const negativePct = 100 - positivePct;

            const avgRatingEl = document.getElementById('communityAvgRating');
            const totalCountEl = document.getElementById('communityTotalCount');
            const ratioEl = document.getElementById('communitySentimentRatio');
            const upPctEl = document.getElementById('sentimentUpPct');
            const downPctEl = document.getElementById('sentimentDownPct');
            const progressBarEl = document.getElementById('sentimentProgressBar');

            if (avgRatingEl) avgRatingEl.textContent = avgStar;
            if (totalCountEl) totalCountEl.textContent = `จาก ${totalVotes} โหวต • ${totalReviews} ความคิดเห็น`;
            if (ratioEl) ratioEl.textContent = `${positivePct}% แนะนำว่าปลอดภัย/น่าเรียน`;
            if (upPctEl) upPctEl.textContent = `ปลอดภัย/น่าเรียน ${positivePct}%`;
            if (downPctEl) downPctEl.textContent = `เฝ้าระวัง ${negativePct}%`;
            if (progressBarEl) progressBarEl.style.width = `${positivePct}%`;

            const listEl = document.getElementById('communityReviewsList');
            if (listEl) {
                listEl.innerHTML = data.reviews.map((rev, idx) => {
                    const safeAuthor = escapeHtml(rev.author || 'นักเรียน / ผู้ปกครอง');
                    const safeComment = escapeHtml(rev.comment || '');
                    const safeTime = escapeHtml(rev.time || '');
                    const starCount = Math.max(1, Math.min(5, Number(rev.stars) || 5));

                    const starsHtml = Array.from({ length: 5 }, (_, i) => {
                        const filled = i < starCount;
                        return `<svg viewBox="0 0 24 24" class="w-3.5 h-3.5" fill="${filled ? '#f59e0b' : 'none'}" stroke="${filled ? '#f59e0b' : '#94a3b8'}" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"></polygon></svg>`;
                    }).join('');

                    const tagsHtml = (Array.isArray(rev.tags) && rev.tags.length > 0)
                        ? `<div class="flex flex-wrap gap-1 mt-2">${rev.tags.map(t => `<span class="px-2 py-0.5 rounded-full text-[10px] bg-sky-500/10 text-sky-700 dark:text-sky-300 border border-sky-500/20">${escapeHtml(t)}</span>`).join('')}</div>`
                        : '';

                    return `
                        <div class="p-3.5 rounded-xl bg-slate-50 dark:bg-slate-900/50 border border-slate-200/80 dark:border-slate-700/40 space-y-1.5">
                            <div class="flex items-center justify-between gap-2">
                                <div class="flex items-center gap-2">
                                    <span class="w-7 h-7 rounded-full bg-gradient-to-tr from-sky-500 to-indigo-500 text-white text-xs font-bold flex items-center justify-center shrink-0">
                                        ${escapeHtml((rev.author || 'น').charAt(0))}
                                    </span>
                                    <div>
                                        <p class="text-xs font-semibold text-slate-900 dark:text-white">${safeAuthor}</p>
                                        <div class="flex items-center gap-0.5 mt-0.5">${starsHtml}</div>
                                    </div>
                                </div>
                                <span class="text-[11px] text-slate-400">${safeTime}</span>
                            </div>
                            <p class="text-xs text-slate-600 dark:text-slate-300 leading-relaxed pt-1">${safeComment}</p>
                            ${tagsHtml}
                            <div class="pt-1.5 flex justify-end">
                                <button type="button" onclick="toggleReviewHelpful(${idx})" class="inline-flex items-center gap-1 px-2 py-0.5 rounded-lg text-[11px] ${rev.helpfulVoted ? 'bg-sky-500/15 text-sky-700 dark:text-sky-300 font-medium' : 'text-slate-500 dark:text-slate-400 hover:bg-slate-200/60 dark:hover:bg-slate-800'} transition">
                                    <i data-lucide="thumbs-up" class="w-3 h-3"></i>
                                    <span>เห็นด้วย / มีประโยชน์ (${Number(rev.helpful) || 0})</span>
                                </button>
                            </div>
                        </div>
                    `;
                }).join('');
            }

            refreshIcons();
        }

        // =========================================================================
        // 8. SHARING, EMERGENCY COORDINATES & REPORT COPY UTILITIES
        // =========================================================================
        function copyTextSafe(text, successMsg) {
            const ta = document.createElement('textarea');
            ta.value = text;
            ta.style.position = 'fixed';
            ta.style.opacity = '0';
            document.body.appendChild(ta);
            ta.select();
            try {
                document.execCommand('copy');
                showToast(successMsg, 'check-circle', 3000);
            } catch (e) {
                showToast('ไม่สามารถคัดลอกอัตโนมัติได้ กรุณาลองใหม่อีกครั้ง', 'alert-circle', 2800);
            }
            document.body.removeChild(ta);
        }

        function shareCurrentArea() {
            if (!currentZone) return;
            const url = new URL(window.location.href.split('?')[0]);
            url.searchParams.set('zone', currentZone.id);
            url.searchParams.set('lat', currentZone.lat.toFixed(5));
            url.searchParams.set('lon', currentZone.lon.toFixed(5));
            copyTextSafe(url.toString(), `คัดลอกลิงก์แชร์ "${currentZone.shortName}" เรียบร้อยแล้ว`);
        }

        function copyEmergencyCoordinates() {
            if (!currentZone) return;
            const gmUrl = `https://www.google.com/maps/search/?api=1&query=${currentZone.lat},${currentZone.lon}`;
            const msg = [
                `[แจ้งพิกัดฉุกเฉิน / ขอความช่วยเหลือ]`,
                `สถานที่: โรงเรียนบุญเหลือวิทยานุสรณ์ (${SCHOOL_CENTER.subdistrict})`,
                `จุดที่อยู่: ${currentZone.title}`,
                `พิกัด GPS: ${currentZone.lat.toFixed(6)}, ${currentZone.lon.toFixed(6)}`,
                `แผนที่นำทาง: ${gmUrl}`
            ].join('\n');
            copyTextSafe(msg, 'คัดลอกพิกัดจุดเกิดเหตุในโรงเรียนพร้อมลิงก์แผนที่แล้ว');
        }

        function copyAreaSummaryReport() {
            if (!currentZone) return;
            const score = document.getElementById('overallScore')?.textContent || '-';
            const grade = document.getElementById('livabilityGrade')?.textContent?.trim() || '-';
            const pm = document.getElementById('pmStatus')?.textContent || '-';
            const activity = document.getElementById('outdoorActivityStatus')?.textContent || '-';
            const gmUrl = `https://www.google.com/maps/search/?api=1&query=${currentZone.lat},${currentZone.lon}`;

            const report = [
                `รายงานประเมินความปลอดภัยและสภาพแวดล้อม - โรงเรียนบุญเหลือวิทยานุสรณ์`,
                `จุดตรวจสอบ: ${currentZone.title}`,
                `คะแนนความปลอดภัย: ${score}/100 (${grade})`,
                `คุณภาพอากาศ (PM2.5): ${pm}`,
                `คำแนะนำกิจกรรมกลางแจ้ง: ${activity}`,
                `พิกัดแผนที่: ${gmUrl}`
            ].join('\n');
            copyTextSafe(report, 'คัดลอกรายงานสรุปจุดตรวจสอบเรียบร้อยแล้ว');
        }

        // =========================================================================
        // 8B. WEBSITE QR CODE GENERATOR & PRINTABLE PNG DOWNLOADER
        // =========================================================================
        let qrTargetMode = 'home';

        function getBaseWebsiteUrl() {
            const href = window.location.href || '';
            if (href.startsWith('http://') || href.startsWith('https://')) {
                return href.split('?')[0].split('#')[0];
            }
            return 'https://geosafety-boonluea.web.app/';
        }

        function buildQrUrlForMode(mode) {
            const baseUrl = getBaseWebsiteUrl();
            if (mode === 'zone' && currentZone) {
                try {
                    const u = new URL(baseUrl);
                    u.searchParams.set('zone', currentZone.id);
                    u.searchParams.set('lat', currentZone.lat.toFixed(5));
                    u.searchParams.set('lon', currentZone.lon.toFixed(5));
                    return u.toString();
                } catch (e) {
                    return baseUrl;
                }
            }
            return baseUrl;
        }

        function renderQrCodeGraphic(text) {
            const box = document.getElementById('qrCodeRenderBox');
            if (!box) return;
            box.innerHTML = '';

            const safeText = (text || '').trim() || getBaseWebsiteUrl();
            if (typeof QRCode !== 'undefined') {
                new QRCode(box, {
                    text: safeText,
                    width: 192,
                    height: 192,
                    colorDark: '#0f172a',
                    colorLight: '#ffffff',
                    correctLevel: QRCode.CorrectLevel.H
                });
            } else {
                box.innerHTML = '<p class="text-xs text-slate-500 p-4">ไม่สามารถโหลดตัวสร้าง QR Code ได้ กรุณาตรวจสอบอินเทอร์เน็ต</p>';
            }
        }

        function setQrTargetMode(mode) {
            qrTargetMode = mode === 'zone' ? 'zone' : 'home';
            const homeBtn = document.getElementById('qrModeHomeBtn');
            const zoneBtn = document.getElementById('qrModeZoneBtn');
            const captionEl = document.getElementById('qrZoneCaption');
            const inputEl = document.getElementById('qrUrlInput');

            if (homeBtn && zoneBtn) {
                if (qrTargetMode === 'home') {
                    homeBtn.className = 'py-2 px-3 rounded-lg bg-sky-600 text-white shadow-sm transition';
                    zoneBtn.className = 'py-2 px-3 rounded-lg text-slate-600 dark:text-slate-300 hover:bg-white/60 dark:hover:bg-slate-800 transition';
                } else {
                    zoneBtn.className = 'py-2 px-3 rounded-lg bg-sky-600 text-white shadow-sm transition';
                    homeBtn.className = 'py-2 px-3 rounded-lg text-slate-600 dark:text-slate-300 hover:bg-white/60 dark:hover:bg-slate-800 transition';
                }
            }

            if (captionEl) {
                captionEl.textContent = qrTargetMode === 'zone' && currentZone
                    ? `จุดตรวจสอบ: ${currentZone.shortName}`
                    : 'หน้าหลัก: โรงเรียนบุญเหลือวิทยานุสรณ์';
            }

            const targetUrl = buildQrUrlForMode(qrTargetMode);
            if (inputEl) inputEl.value = targetUrl;
            renderQrCodeGraphic(targetUrl);
        }

        function generateQrFromInput() {
            const inputEl = document.getElementById('qrUrlInput');
            if (!inputEl) return;
            renderQrCodeGraphic(inputEl.value.trim());
        }

        function openQrCodeModal() {
            const modal = document.getElementById('qrCodeModal');
            const noticeEl = document.getElementById('qrLocalHostNotice');
            if (!modal) return;

            const isLiveHttp = window.location.protocol === 'http:' || window.location.protocol === 'https:';
            if (noticeEl) {
                noticeEl.classList.toggle('hidden', isLiveHttp);
            }

            setQrTargetMode('home');
            modal.classList.remove('hidden');
            refreshIcons();
        }

        function closeQrCodeModal() {
            const modal = document.getElementById('qrCodeModal');
            if (modal) modal.classList.add('hidden');
        }

        function copyQrUrl() {
            const inputEl = document.getElementById('qrUrlInput');
            const urlVal = inputEl ? inputEl.value.trim() : getBaseWebsiteUrl();
            copyTextSafe(urlVal, 'คัดลอกลิงก์สำหรับ QR Code เรียบร้อยแล้ว');
        }

        function downloadQrCodePng() {
            const box = document.getElementById('qrCodeRenderBox');
            const inputEl = document.getElementById('qrUrlInput');
            const captionEl = document.getElementById('qrZoneCaption');
            if (!box) return;

            const sourceCanvas = box.querySelector('canvas');
            if (!sourceCanvas) {
                showToast('ไม่พบภาพ QR Code กรุณาลองเปิดหน้าต่างใหม่อีกครั้ง', 'alert-circle', 2500);
                return;
            }

            // Compose a clean printable school poster card onto an offscreen canvas
            const outCanvas = document.createElement('canvas');
            outCanvas.width = 640;
            outCanvas.height = 780;
            const ctx = outCanvas.getContext('2d');

            // White background card
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(0, 0, outCanvas.width, outCanvas.height);

            // Top Sky-Blue Header Banner
            ctx.fillStyle = '#0284c7';
            ctx.fillRect(0, 0, outCanvas.width, 130);

            ctx.fillStyle = '#ffffff';
            ctx.font = 'bold 28px Kanit, Inter, sans-serif';
            ctx.textAlign = 'center';
            ctx.fillText('GeoSafety & Livability', outCanvas.width / 2, 56);

            ctx.font = '500 22px Kanit, Inter, sans-serif';
            ctx.fillText('โรงเรียนบุญเหลือวิทยานุสรณ์', outCanvas.width / 2, 98);

            // QR Code Frame
            const qrSize = 400;
            const qrX = (outCanvas.width - qrSize) / 2;
            const qrY = 175;
            ctx.strokeStyle = '#cbd5e1';
            ctx.lineWidth = 4;
            ctx.strokeRect(qrX - 18, qrY - 18, qrSize + 36, qrSize + 36);
            ctx.drawImage(sourceCanvas, qrX, qrY, qrSize, qrSize);

            // Caption & Scan Instructions
            const zoneLabel = captionEl ? captionEl.textContent : 'โรงเรียนบุญเหลือวิทยานุสรณ์';
            ctx.fillStyle = '#0f172a';
            ctx.font = 'bold 22px Kanit, Inter, sans-serif';
            ctx.fillText(zoneLabel, outCanvas.width / 2, 645);

            ctx.fillStyle = '#475569';
            ctx.font = '18px Kanit, Inter, sans-serif';
            ctx.fillText('สแกน QR Code เพื่อตรวจสอบความปลอดภัยและค่าฝุ่น PM2.5 สด', outCanvas.width / 2, 685);

            const urlText = (inputEl ? inputEl.value.trim() : '').slice(0, 56);
            ctx.fillStyle = '#0284c7';
            ctx.font = '15px Inter, monospace';
            ctx.fillText(urlText, outCanvas.width / 2, 730);

            const link = document.createElement('a');
            link.download = `QR-GeoSafety-Boonluea-${qrTargetMode}.png`;
            link.href = outCanvas.toDataURL('image/png');
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);

            showToast('ดาวน์โหลดรูปภาพ QR Code (PNG) เรียบร้อยแล้ว!', 'download', 2800);
        }

        // =========================================================================
        // 9. GLOBAL EXPORTS & APP BOOTSTRAP
        // =========================================================================
        window.setMapLayer = setMapLayer;
        window.toggleLiveRadar = toggleLiveRadar;
        window.togglePOICategory = togglePOICategory;
        window.toggleMapFullscreen = toggleMapFullscreen;
        window.resetToSchoolCenter = resetToSchoolCenter;
        window.selectZoneByDropdown = selectZoneByDropdown;
        window.handleSearchInput = handleSearchInput;
        window.selectAutocompleteItem = selectAutocompleteItem;
        window.clearSearchInput = clearSearchInput;
        window.performSearch = performSearch;
        window.loadZoneData = loadZoneData;
        window.manualRefreshNow = manualRefreshNow;
        window.closeGpsHelpModal = closeGpsHelpModal;
        window.useCurrentLocation = useCurrentLocation;
        window.toggleLiveGpsTracking = toggleLiveGpsTracking;
        window.recalibrateGpsNow = recalibrateGpsNow;
        window.requestQuickLocationPermission = requestQuickLocationPermission;
        window.copyAppUrlForExternalBrowser = copyAppUrlForExternalBrowser;
        window.setReviewStar = setReviewStar;
        window.hoverReviewStar = hoverReviewStar;
        window.resetReviewStarHover = resetReviewStarHover;
        window.toggleReviewTag = toggleReviewTag;
        window.voteAreaSentiment = voteAreaSentiment;
        window.submitAreaReview = submitAreaReview;
        window.toggleReviewHelpful = toggleReviewHelpful;
        window.shareCurrentArea = shareCurrentArea;
        window.copyEmergencyCoordinates = copyEmergencyCoordinates;
        window.copyAreaSummaryReport = copyAreaSummaryReport;
        window.openQrCodeModal = openQrCodeModal;
        window.closeQrCodeModal = closeQrCodeModal;
        window.setQrTargetMode = setQrTargetMode;
        window.generateQrFromInput = generateQrFromInput;
        window.copyQrUrl = copyQrUrl;
        window.downloadQrCodePng = downloadQrCodePng;

        function bootstrapApp() {
            loadCommunityStore();
            refreshIcons();
            initStarRatingControls();
            populateSchoolZoneDropdown();

            const params = new URLSearchParams(window.location.search);
            const qZone = params.get('zone');
            const qLat = parseFloat(params.get('lat'));
            const qLon = parseFloat(params.get('lon'));

            if (qZone && SCHOOL_ZONES.some(z => z.id === qZone)) {
                const target = SCHOOL_ZONES.find(z => z.id === qZone);
                initMap(target.lat, target.lon);
                initGpsPermissionMonitor();
                startRealTimeEngine();
                loadZoneData(target.id, { initial: true, zoom: 17 });
            } else if (Number.isFinite(qLat) && Number.isFinite(qLon) && qLat >= 14.9 && qLat <= 15.2 && qLon >= 101.9 && qLon <= 102.3) {
                initMap(qLat, qLon);
                initGpsPermissionMonitor();
                startRealTimeEngine();
                inspectCustomSchoolCoordinate(qLat, qLon, false);
            } else {
                initMap(SCHOOL_CENTER.lat, SCHOOL_CENTER.lon);
                initGpsPermissionMonitor();
                startRealTimeEngine();
                loadZoneData('overview', { initial: true, zoom: 17 });
            }
        }

        if (document.readyState === 'loading') {
            document.addEventListener('DOMContentLoaded', bootstrapApp);
        } else {
            bootstrapApp();
        }
    </script>
</body>
</html>
