<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>RFID UHF Anti-Flapping & Dwell-Time Intelligent System</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome for Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Chakra+Petch:wght@300;400;600;700&display=swap');
        
        body { 
            font-family: 'Chakra Petch', sans-serif;
            background-color: #0b0f19; /* Deep space dark blue */
            color: #f1f5f9;
        }
        
        /* สัญญาณคลื่นวิทยุกระจาย */
        .antenna-wave {
            animation: rfid-wave 2s infinite linear;
        }
        @keyframes rfid-wave {
            0% { transform: scale(1); opacity: 0.8; }
            50% { opacity: 0.4; }
            100% { transform: scale(3); opacity: 0; }
        }

        /* ล้อรถเข็นหมุน */
        .wheel-spin {
            animation: spin 1s infinite linear;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        /* สำหรับตารางเลื่อนได้ */
        .console-scrollbar::-webkit-scrollbar {
            width: 6px;
        }
        .console-scrollbar::-webkit-scrollbar-track {
            background: #0f172a;
        }
        .console-scrollbar::-webkit-scrollbar-thumb {
            background: #334155;
            border-radius: 3px;
        }
    </style>
</head>
<body class="p-4 md:p-6 min-h-screen">

    <div class="max-w-7xl mx-auto space-y-6">
        
        <!-- Header Console -->
        <div class="bg-slate-950 border-2 border-emerald-500/40 p-6 rounded-2xl shadow-[0_0_30px_rgba(16,185,129,0.15)] flex flex-col md:flex-row justify-between items-start md:items-center gap-4">
            <div>
                <div class="flex items-center gap-3">
                    <span class="flex h-3.5 w-3.5 relative">
                        <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-emerald-400 opacity-75"></span>
                        <span class="relative inline-flex rounded-full h-3.5 w-3.5 bg-emerald-500"></span>
                    </span>
                    <h1 class="text-3xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-emerald-400 via-cyan-400 to-blue-500 tracking-wider">
                        RFID ANTI-FLAPPING CONSOLE V2
                    </h1>
                </div>
                <p class="text-slate-400 text-sm mt-1">
                    ระบบป้องกันการสแกนซ้ำซ้อนและคำนวณเวลาจอดแช่ (15s Lockout & Dwell-Time Filtering Engine)
                </p>
            </div>
            
            <div class="flex flex-wrap items-center gap-3">
                <button id="btn-sound" onclick="toggleSound()" class="px-4 py-2 bg-slate-900 hover:bg-slate-800 border border-slate-700 rounded-lg flex items-center gap-2 text-sm transition">
                    <i id="sound-icon" class="fa-solid fa-volume-high text-emerald-400"></i>
                    <span id="sound-text">เปิดเสียงระบบ</span>
                </button>
                <div class="px-4 py-2 bg-slate-900 border border-emerald-500/20 rounded-lg text-sm text-right">
                    <span class="text-slate-400 block text-xs">LOCKOUT TIMEOUT</span>
                    <span class="text-emerald-400 font-mono font-bold"><i class="fa-solid fa-clock-rotate-left mr-1"></i>120 วินาที (2 นาที)</span>
                </div>
            </div>
        </div>

        <!-- Dashboard Statistics (แก้ไข Grid ให้รองรับการกดดูประวัติสแกนสะสมอย่างสวยงาม) -->
        <div class="grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-5 gap-4">
            <div onclick="openModal('ALL')" class="bg-slate-950 border border-slate-800 p-4 rounded-xl cursor-pointer hover:bg-slate-900 hover:border-emerald-500 transition-all group">
                <span class="text-slate-500 text-xs block group-hover:text-emerald-400 transition-colors">รถเข็นจดทะเบียน</span>
                <span id="stat-total" class="text-2xl font-bold text-slate-100 font-mono">0</span>
                <span class="text-[10px] text-emerald-500/70 block mt-1"><i class="fa-solid fa-hand-pointer mr-1"></i>คลิกดูรายชื่อ 20 คัน</span>
            </div>
            <div onclick="openModal('IN')" class="bg-slate-950 border border-slate-800 p-4 rounded-xl cursor-pointer hover:bg-slate-900 hover:border-blue-500 transition-all group">
                <span class="text-slate-500 text-xs block group-hover:text-blue-400 transition-colors">อยู่ในสโตร์ (IN)</span>
                <span id="stat-in" class="text-2xl font-bold text-blue-400 font-mono">0</span>
                <span class="text-[10px] text-blue-500/70 block mt-1"><i class="fa-solid fa-store mr-1"></i>คลิกดูรายชื่อ</span>
            </div>
            <div onclick="openModal('OUT')" class="bg-slate-950 border border-slate-800 p-4 rounded-xl cursor-pointer hover:bg-slate-900 hover:border-amber-500 transition-all group">
                <span class="text-slate-500 text-xs block group-hover:text-amber-400 transition-colors">อยู่นอกสโตร์ (OUT)</span>
                <span id="stat-out" class="text-2xl font-bold text-amber-500 font-mono">0</span>
                <span class="text-[10px] text-amber-500/70 block mt-1"><i class="fa-solid fa-warehouse mr-1"></i>คลิกดูรายชื่อ</span>
            </div>
            <div onclick="openModal('STATIONARY')" class="bg-slate-950 border border-slate-800 p-4 rounded-xl cursor-pointer hover:bg-slate-900 hover:border-yellow-500 transition-all group">
                <span class="text-slate-500 text-xs block group-hover:text-yellow-400 transition-colors">จอดแช่/อยู่ที่เดิม</span>
                <span id="stat-stationary" class="text-2xl font-bold text-yellow-500 font-mono">0</span>
                <span class="text-[10px] text-yellow-500/70 block mt-1"><i class="fa-solid fa-hourglass-half mr-1"></i>คลิกดูรายชื่อค้าง</span>
            </div>
            <div onclick="openModal('SCANS')" class="bg-slate-950 border border-slate-800 p-4 rounded-xl col-span-2 sm:col-span-1 cursor-pointer hover:bg-slate-900 hover:border-purple-500 transition-all group">
                <span class="text-slate-500 text-xs block group-hover:text-purple-400 transition-colors">สแกนสะสม (รวมบล็อก)</span>
                <span id="stat-scans" class="text-2xl font-bold text-purple-400 font-mono">0</span>
                <span class="text-[10px] text-cyan-500/70 block mt-1"><i class="fa-solid fa-bolt mr-1"></i>คลิกดูประวัติสแกน</span>
            </div>
        </div>

        <!-- Main Visualizer Arena -->
        <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
            
            <!-- Left 2 Columns: Visual Track and Controller -->
            <div class="lg:col-span-2 space-y-6">
                <!-- Visual Simulation Track -->
                <div class="bg-slate-950 border border-slate-800 rounded-2xl p-6 relative overflow-hidden shadow-inner">
                    <div class="absolute top-3 left-3 bg-slate-900 border border-slate-800 text-xs text-cyan-400 px-3 py-1 rounded-full font-mono uppercase">
                        <i class="fa-solid fa-eye mr-1"></i> Visual Live Stream Tracker
                    </div>

                    <!-- RFID Ceiling Antenna Representation -->
                    <div class="absolute top-0 left-1/2 transform -translate-x-1/2 flex flex-col items-center">
                        <div class="w-32 h-5 bg-slate-800 rounded-b-lg border-x-2 border-b-2 border-emerald-500 shadow-[0_0_20px_rgba(16,185,129,0.3)] z-20 flex items-center justify-center">
                            <span class="w-2 h-2 rounded-full bg-red-500 animate-pulse" id="antenna-led"></span>
                        </div>
                        <div class="text-[9px] font-mono text-emerald-400 mt-1 uppercase tracking-widest bg-slate-950 px-2 rounded-full border border-slate-800">UHF-CEILING-ANT-1</div>
                        
                        <!-- Waves of UHF RF -->
                        <div class="relative w-full flex justify-center mt-2">
                            <span class="absolute w-24 h-24 rounded-full bg-emerald-500/10 border border-emerald-500/20 antenna-wave"></span>
                            <span class="absolute w-24 h-24 rounded-full bg-emerald-500/10 border border-emerald-500/20 antenna-wave" style="animation-delay: 0.6s;"></span>
                            <span class="absolute w-24 h-24 rounded-full bg-emerald-500/10 border border-emerald-500/20 antenna-wave" style="animation-delay: 1.2s;"></span>
                        </div>
                    </div>

                    <!-- Track Lanes -->
                    <div class="h-64 mt-16 relative bg-slate-900/40 rounded-xl border border-slate-800 flex items-center justify-between px-10">
                        
                        <!-- Zone A: OUTSIDE -->
                        <div class="w-1/3 flex flex-col items-center justify-center text-center">
                            <div class="w-12 h-12 rounded-full bg-slate-800/80 flex items-center justify-center border border-slate-700 mb-2">
                                <i class="fa-solid fa-warehouse text-slate-400 text-xl"></i>
                            </div>
                            <span class="text-xs font-bold text-slate-500 uppercase tracking-widest">Zone A: Outside (จุดจอดด้านนอก)</span>
                        </div>

                        <!-- Zone B: SCANNING FIELD -->
                        <div id="rfid-field" class="w-1/3 h-full border-x-2 border-dashed border-emerald-500/20 bg-emerald-500/5 flex flex-col justify-end pb-4 items-center relative transition-colors duration-200">
                            <span class="text-[10px] text-emerald-400/80 font-mono tracking-widest bg-slate-950 px-2 py-0.5 rounded border border-emerald-500/20 absolute top-4">UHF BEAM AREA (920-925 MHz)</span>
                            <!-- RSSI Signal Meter Bar -->
                            <div class="w-28 bg-slate-950 border border-slate-800 rounded p-1.5 text-center font-mono">
                                <div class="text-[8px] text-slate-500">UHF ANTENNA RSSI</div>
                                <div id="rssi-bar" class="h-1.5 w-0 bg-gradient-to-r from-red-500 via-yellow-400 to-emerald-400 rounded transition-all duration-100"></div>
                                <div id="rssi-val" class="text-[9px] text-slate-400 mt-0.5">-120 dBm (Idle)</div>
                            </div>
                        </div>

                        <!-- Zone C: INSIDE -->
                        <div class="w-1/3 flex flex-col items-center justify-center text-center">
                            <div class="w-12 h-12 rounded-full bg-blue-950/80 flex items-center justify-center border border-blue-800 mb-2">
                                <i class="fa-solid fa-store text-blue-400 text-xl"></i>
                            </div>
                            <span class="text-xs font-bold text-blue-400 uppercase tracking-widest">Zone B: Store (ด้านในห้าง)</span>
                        </div>

                        <!-- Live moving carts render zone -->
                        <div id="simulation-cart-lane" class="absolute inset-x-0 top-1/2 transform -translate-y-1/2 h-24 pointer-events-none relative">
                            <!-- Moving carts will be dynamically injected here with CSS positioning -->
                        </div>

                    </div>
                </div>

                <!-- Simulation Controls Panel -->
                <div class="bg-slate-950 border border-slate-800 rounded-2xl p-6">
                    <h2 class="text-lg font-bold text-slate-200 mb-4 flex items-center gap-2">
                        <i class="fa-solid fa-sliders text-emerald-400"></i> แผงควบคุมบอร์ดจำลองกลยุทธ์ (Anti-Flapping Simulator Panel)
                    </h2>
                    
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                        <!-- Auto Engine -->
                        <div class="bg-slate-900 border border-slate-800 p-4 rounded-xl flex flex-col justify-between">
                            <div>
                                <span class="font-bold text-sm block text-slate-300">AUTO PATROL CARTS</span>
                                <p class="text-xs text-slate-500 mt-1">ปล่อยรถวิ่งผ่านตามปกติทีละคันอัตโนมัติ (วิ่งผ่านฟิลด์แล้วผ่านเลย)</p>
                            </div>
                            <div class="mt-4">
                                <button id="btn-auto" onclick="toggleAutoMode()" class="w-full py-2.5 bg-blue-600 hover:bg-blue-500 text-white font-bold rounded-lg transition text-sm flex items-center justify-center gap-2">
                                    <i class="fa-solid fa-play"></i> เริ่มโหมดวิ่งอัตโนมัติ
                                </button>
                            </div>
                        </div>

                        <!-- Manual Spigot Test (Normal) -->
                        <div class="bg-slate-900 border border-slate-800 p-4 rounded-xl flex flex-col justify-between">
                            <div>
                                <span class="font-bold text-sm block text-slate-300">MANUAL RUN-THROUGH</span>
                                <p class="text-xs text-slate-500 mt-1">จำลองผลักรถเข็นวิ่งข้ามเขตทันที (ไม่มีการหยุดแช่)</p>
                            </div>
                            <div class="mt-4 grid grid-cols-2 gap-2">
                                <button onclick="triggerManualCart('ENTER')" class="bg-blue-600/20 hover:bg-blue-600/40 text-blue-300 border border-blue-500/30 py-2 text-xs font-bold rounded-lg transition">
                                    <i class="fa-solid fa-sign-in-alt mr-1"></i> ดันเข้า (IN)
                                </button>
                                <button onclick="triggerManualCart('EXIT')" class="bg-amber-600/20 hover:bg-amber-600/40 text-amber-300 border border-amber-500/30 py-2 text-xs font-bold rounded-lg transition">
                                    <i class="fa-solid fa-sign-out-alt mr-1"></i> ดันออก (OUT)
                                </button>
                            </div>
                        </div>

                        <!-- Manual DWELL / HOLD Test (THE CORE HIGHLIGHT!) -->
                        <div class="bg-slate-900 border border-emerald-500/30 p-4 rounded-xl flex flex-col justify-between shadow-[0_0_15px_rgba(16,185,129,0.05)]">
                            <div>
                                <span class="font-bold text-sm text-emerald-400 flex items-center gap-1">
                                    <i class="fa-solid fa-hand-holding-hand"></i> DWELL & HOLD SIMULATION
                                </span>
                                <p class="text-xs text-slate-400 mt-1 font-sans">**จำลองรถเข็นมาจอดแช่อยู่ใต้เครื่องสแกนเลย** เพื่อทดสอบตัวกรองป้องกันการตีสถานะเพี้ยน!</p>
                            </div>
                            <div class="mt-3">
                                <button onclick="triggerStationaryDwellSim()" class="w-full py-2.5 bg-gradient-to-r from-emerald-600 to-teal-600 hover:from-emerald-500 hover:to-teal-500 text-white font-bold rounded-lg transition text-xs flex items-center justify-center gap-2 shadow">
                                    <i class="fa-solid fa-anchor"></i> นำมาจอดแช่และสแกนซ้ำๆ
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Right Column: Live Devices Status -->
            <div class="space-y-6">
                <!-- Lockout logic explaining -->
                <div class="bg-slate-950 border border-slate-800 rounded-2xl p-6">
                    <h2 class="text-base font-bold text-slate-200 mb-3 flex items-center gap-2">
                        <i class="fa-solid fa-shield-halved text-emerald-400"></i> อัลกอริทึมคุมสถานะ (Anti-Flapping Flow)
                    </h2>
                    <ul class="space-y-2 text-xs text-slate-400">
                        <li class="flex items-start gap-2">
                            <span class="w-1.5 h-1.5 rounded-full bg-emerald-400 mt-1.5"></span>
                            <span><strong>สแกนครั้งแรก:</strong> สลับสถานะ (IN / OUT) และล็อกปุ่มเป็นคูลดาวน์ 2 นาที (120s) ทันที</span>
                        </li>
                        <li class="flex items-start gap-2">
                            <span class="w-1.5 h-1.5 rounded-full bg-emerald-400 mt-1.5"></span>
                            <span><strong>สแกนซ้ำภายใน 2 นาที:</strong> ระบบบล็อกสถานะ ไม่ให้เปลี่ยนเป็นทิศตรงข้าม บันทึกเป็น <strong class="text-yellow-400">"STATIONARY"</strong></span>
                        </li>
                        <li class="flex items-start gap-2">
                            <span class="w-1.5 h-1.5 rounded-full bg-emerald-400 mt-1.5"></span>
                            <span><strong>สแกนซ้ำหลัง 2 นาที (แต่รถเข็นยังไม่เดินพ้น):</strong> ตีเป็นจอดแช่ต่อเนื่อง <strong class="text-orange-400">"STATIONARY (DWELL)"</strong> เพื่อรักษาความปลอดภัย</span>
                        </li>
                    </ul>
                </div>

                <!-- Dynamic Real-Time Carts Status Grid -->
                <div class="bg-slate-950 border border-slate-800 rounded-2xl p-6 flex flex-col max-h-[310px]">
                    <h2 class="text-md font-bold text-slate-200 mb-3 flex items-center justify-between">
                        <span class="flex items-center gap-2">
                            <i class="fa-solid fa-cart-shopping text-blue-500"></i> ตารางสถานะรถเข็นล่าสุด
                        </span>
                        <span class="text-[10px] bg-slate-900 text-slate-400 border border-slate-800 px-2 py-0.5 rounded font-mono">SQL Carts Table</span>
                    </h2>
                    <div class="overflow-y-auto flex-1 console-scrollbar">
                        <table class="w-full text-left font-mono text-xs">
                            <thead class="sticky top-0 bg-slate-950 text-slate-500 border-b border-slate-800 text-[10px]">
                                <tr>
                                    <th class="py-2">CART ID (UHF)</th>
                                    <th class="py-2 text-center">STATUS</th>
                                    <th class="py-2 text-right">LOCK STATE</th>
                                </tr>
                            </thead>
                            <tbody id="tbl-carts" class="divide-y divide-slate-900">
                                <!-- Dynamic rows -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>

        <!-- SQL Server Live Terminal Log Console -->
        <div class="bg-slate-950 border border-slate-800 rounded-2xl overflow-hidden shadow-2xl">
            <div class="bg-slate-900/80 px-6 py-3 border-b border-slate-800 flex justify-between items-center">
                <div class="flex items-center gap-2">
                    <span class="w-2.5 h-2.5 rounded-full bg-red-500"></span>
                    <span class="w-2.5 h-2.5 rounded-full bg-yellow-500"></span>
                    <span class="w-2.5 h-2.5 rounded-full bg-emerald-500"></span>
                    <span class="text-xs text-slate-400 font-mono ml-2"><i class="fa-solid fa-code text-emerald-400 mr-1"></i> live_mssql_transactions.sql</span>
                </div>
                <div class="flex items-center gap-3">
                    <button onclick="clearConsoleLog()" class="text-xs text-slate-500 hover:text-slate-300 font-mono"><i class="fa-solid fa-trash mr-1"></i> CLEAR LOGS</button>
                    <span class="text-xs bg-emerald-500/10 text-emerald-400 border border-emerald-500/20 px-2 py-0.5 rounded font-mono">MSSQL Server Express</span>
                </div>
            </div>
            <div id="console-log" class="p-4 bg-slate-950 h-60 font-mono text-[11px] overflow-y-auto console-scrollbar text-emerald-400 space-y-1">
                <div class="text-slate-500">[2026-07-02 15:00:00] SQL Server Engine ready and waiting for ESP32 post requests...</div>
                <div class="text-slate-500">[2026-07-02 15:00:00] Web RFID Antiflap logic is set to COOLDOWN = 120000ms.</div>
            </div>
        </div>

        <!-- Live Scan Logs (History Log with Cool-down Indicators) -->
        <div class="bg-slate-950 border border-slate-800 rounded-2xl p-6">
            <h2 class="text-md font-bold text-slate-200 mb-4 flex items-center justify-between">
                <span class="flex items-center gap-2">
                    <i class="fa-solid fa-database text-cyan-400"></i> บันทึกข้อมูลธุรกรรมประวัติการอ่านสัญญาณ (CartLogs Historian Table)
                </span>
                <span class="text-xs text-slate-500">เรียงจากใหม่ -> เก่า</span>
            </h2>
            <div class="overflow-x-auto">
                <table class="w-full text-left font-mono text-xs">
                    <thead>
                        <tr class="bg-slate-900 text-slate-400 border-b border-slate-800 text-[11px]">
                            <th class="p-3">LOG_ID</th>
                            <th class="p-3">TAG_ID (CartID)</th>
                            <th class="p-3">SCAN TYPE</th>
                            <th class="p-3">SQL EFFECT</th>
                            <th class="p-3">UHF RSSI</th>
                            <th class="p-3">COOLDOWN STATUS</th>
                            <th class="p-3">SCAN TIME</th>
                        </tr>
                    </thead>
                    <tbody id="tbl-history" class="divide-y divide-slate-800/40">
                        <tr id="empty-history-row">
                            <td colspan="7" class="p-8 text-center text-slate-600">
                                <i class="fa-solid fa-circle-info mr-2 text-slate-700"></i> ระบบอยู่ในโหมดพร้อมทำงาน รอการเหนี่ยวนำสัญญาณสแกน
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>

    </div>

    <!-- Modal System สำหรับกดดูรายชื่อรถเข็นแบบ Real-time -->
    <div id="cart-list-modal" class="fixed inset-0 bg-black/70 backdrop-blur-sm hidden z-[60] flex items-center justify-center opacity-0 transition-opacity duration-300">
        <div class="bg-slate-900 border-2 border-emerald-500/40 w-full max-w-xl rounded-2xl shadow-2xl p-6 transform scale-95 transition-transform duration-300 overflow-hidden flex flex-col max-h-[85vh]">
            <div class="flex justify-between items-center mb-4 border-b border-slate-700 pb-3">
                <h3 id="modal-title" class="text-xl font-bold text-emerald-400 flex items-center gap-2">
                    <i class="fa-solid fa-list-ul"></i> รายชื่อรถเข็น
                </h3>
                <button onclick="closeModal()" class="text-slate-400 hover:text-white transition">
                    <i class="fa-solid fa-xmark text-xl"></i>
                </button>
            </div>
            <div class="overflow-y-auto console-scrollbar pr-2 flex-1 min-h-[350px]">
                <table class="w-full text-left font-mono text-sm">
                    <thead id="modal-table-header" class="sticky top-0 bg-slate-900 text-slate-500 border-b border-slate-700 text-xs z-10">
                        <!-- หัวตารางจะถูกวาดตามหมวดหมู่โดยอัตโนมัติด้วย JavaScript -->
                    </thead>
                    <tbody id="modal-cart-list" class="divide-y divide-slate-800/60">
                        <!-- รายชื่อถูกแทรกด้วย JS -->
                    </tbody>
                </table>
            </div>
            <div class="mt-4 pt-3 border-t border-slate-700 text-right">
                <button onclick="closeModal()" class="px-6 py-2 bg-slate-800 hover:bg-slate-700 text-white rounded-lg text-sm font-bold transition">ปิดหน้าต่าง</button>
            </div>
        </div>
    </div>

    <!-- Alert Toast System -->
    <div id="toast" class="fixed bottom-5 right-5 bg-slate-900 border-2 border-emerald-500/30 text-slate-100 px-6 py-4 rounded-xl shadow-2xl transform transition-all translate-y-20 opacity-0 duration-300 flex items-center gap-4 z-50">
        <div id="toast-icon-box" class="p-2 bg-emerald-500/20 text-emerald-400 rounded-lg">
            <i class="fa-solid fa-satellite-dish animate-pulse"></i>
        </div>
        <div>
            <div id="toast-title" class="font-bold text-sm text-white">📡 RFID DETECTED</div>
            <div id="toast-body" class="text-xs text-slate-400"></div>
        </div>
    </div>

    <!-- Web Logic Control Scripts -->
    <script>
        // --- เปลี่ยน URL มาเป็น Backend Node.js (SQL Server) ---
        const LOCAL_API_URL = 'http://localhost:3000/api/scan'; 

        // --- 1. ข้อมูลรถเข็นในระบบพร้อมตัวระบุสถานะเวลา (State) ---
        let dbCarts = [];
        // สร้างรถเข็น 20 คัน เริ่มต้นเป็นอยู่นอกสโตร์ (OUT) ทั้งหมด เพื่อให้การสแกนครั้งแรกเป็นการเข้า (ENTER) เสมอ
        for(let i = 1; i <= 20; i++) {
            let num = i.toString().padStart(3, '0');
            dbCarts.push({ 
                id: `CART-UHF-X${num}`, 
                status: 'OUT', // ทุกคันเริ่มต้นจอดด้านนอก
                lastScannedTime: 0, 
                isStationary: false, 
                cooldownRemaining: 0 
            });
        }

        let dbLogs = [];
        let logCounter = 1;
        let isAutoMode = false;
        let simSpeed = 2; 
        let soundEnabled = false;
        let autoTimer = null;
        let cooldownInterval = null;

        // --- 2. เสียงบี๊บ RFID (Web Audio API) ---
        function toggleSound() {
            soundEnabled = !soundEnabled;
            const btn = document.getElementById('btn-sound');
            const icon = document.getElementById('sound-icon');
            const text = document.getElementById('sound-text');
            if (soundEnabled) {
                icon.className = "fa-solid fa-volume-high text-emerald-400";
                text.innerText = "ปิดเสียงระบบ";
                btn.className = "px-4 py-2 bg-slate-900 hover:bg-slate-800 border border-slate-700 rounded-lg flex items-center gap-2 text-sm transition text-emerald-400";
                playBeep(900, 0.08);
            } else {
                icon.className = "fa-solid fa-volume-xmark text-slate-400";
                text.innerText = "เปิดเสียงระบบ";
                btn.className = "px-4 py-2 bg-slate-900 hover:bg-slate-800 border border-slate-700 rounded-lg flex items-center gap-2 text-sm transition";
            }
        }

        function playBeep(frequency, duration) {
            if (!soundEnabled) return;
            try {
                const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                const oscillator = audioCtx.createOscillator();
                const gainNode = audioCtx.createGain();
                
                oscillator.connect(gainNode);
                gainNode.connect(audioCtx.destination);
                oscillator.type = 'sine';
                oscillator.frequency.value = frequency;
                gainNode.gain.setValueAtTime(0.08, audioCtx.currentTime);
                gainNode.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + duration);
                
                oscillator.start(audioCtx.currentTime);
                oscillator.stop(audioCtx.currentTime + duration);
            } catch (e) {
                console.log("Audio API failed", e);
            }
        }

        // --- 3. อัปเดตคูลดาวน์ 120 วินาทีเรียลไทม์ (ทุก 200ms เพื่อความลื่นไหล) ---
        function startCooldownTracker() {
            if (cooldownInterval) clearInterval(cooldownInterval);
            cooldownInterval = setInterval(() => {
                const now = Date.now();
                let updated = false;
                
                dbCarts.forEach(cart => {
                    if (cart.lastScannedTime > 0) {
                        const elapsed = now - cart.lastScannedTime;
                        const remaining = Math.max(0, 120000 - elapsed);
                        const newSec = Math.ceil(remaining / 1000);
                        
                        if (cart.cooldownRemaining !== newSec) {
                            cart.cooldownRemaining = newSec;
                            updated = true;
                        }
                    }
                });
                if (updated) {
                    renderAllData();
                    if (currentModalFilter) updateModalList(); // อัปเดตเวลาลดลงใน Modal สดๆ
                }
            }, 200);
        }

        // --- 4. การจัดการแอนิเมชันโมเดลรถเข็นแบบสมจริง (Realistic Animated Cart Model) ---
        function spawnCartAnimation(cartId, direction, behavior = 'NORMAL') {
            const lane = document.getElementById('simulation-cart-lane');
            const cartEl = document.createElement('div');
            
            // สร้างโมเดลรถเข็นที่มีรายละเอียดมากขึ้น สไตล์ 3D-Neon
            cartEl.className = "absolute top-2 h-16 w-20 flex flex-col items-center justify-between bg-slate-900/90 border border-slate-700 rounded-lg text-slate-300 shadow-xl pointer-events-none transition-all ease-linear";
            cartEl.innerHTML = `
                <!-- ส่วนตัวโครงรถเข็นมีไฟ LED แดง/เขียว ตามล็อกคูลดาวน์ -->
                <div class="w-full flex justify-between items-center px-1.5 pt-1">
                    <span id="led-${cartId}" class="w-2.5 h-2.5 rounded-full bg-red-500 animate-pulse"></span>
                    <span class="text-[8px] font-mono font-bold bg-slate-800 px-1 rounded text-cyan-400">${cartId}</span>
                </div>
                
                <!-- ตะกร้ารถเข็นเหล็กสไตล์มินิมอล -->
                <div class="relative w-11/12 h-6 border-b border-x border-slate-500/50 rounded-b flex items-center justify-center bg-slate-800/40">
                    <i class="fa-solid fa-cart-shopping text-sm text-slate-400"></i>
                    <!-- แสดงสัญลักษณ์แม่กุญแจและเวลาหากล็อก 120 วิอยู่ -->
                    <div id="cart-lock-visual-${cartId}" class="hidden absolute inset-0 bg-red-900/80 rounded flex items-center justify-center gap-1 text-[9px] text-white font-bold animate-pulse">
                        <i class="fa-solid fa-lock text-[8px]"></i>
                        <span id="cart-lock-time-${cartId}">120s</span>
                    </div>
                </div>

                <!-- ฐานล้อคู่และเพลารถเข็น ล้อจะหมุนเมื่อรถเข็นเลื่อน -->
                <div class="w-full flex justify-around px-2 pb-1">
                    <div class="w-3.5 h-3.5 rounded-full border border-slate-400 bg-slate-950 flex items-center justify-center">
                        <div class="w-1.5 h-1.5 bg-slate-400 rounded-full wheel-spin"></div>
                    </div>
                    <div class="w-3.5 h-3.5 rounded-full border border-slate-400 bg-slate-950 flex items-center justify-center">
                        <div class="w-1.5 h-1.5 bg-slate-400 rounded-full wheel-spin"></div>
                    </div>
                </div>
            `;

            let startPos, endPos, targetStopPos = 50;
            if (direction === 'ENTER') {
                startPos = -15;
                endPos = 115;
            } else {
                startPos = 115;
                endPos = -15;
            }

            cartEl.style.left = `${startPos}%`;
            lane.appendChild(cartEl);

            const duration = 6500 / simSpeed; 
            const startTime = performance.now();
            let hasScanned = false;

            // ตรวจสอบคูลดาวน์ปัจจุบันเพื่อเลือกสีไฟ LED ของรถ
            const cartObj = dbCarts.find(c => c.id === cartId);
            const led = cartEl.querySelector(`#led-${cartId}`);
            if (cartObj && cartObj.cooldownRemaining > 0) {
                led.className = "w-2.5 h-2.5 rounded-full bg-red-500 animate-pulse";
            } else {
                led.className = "w-2.5 h-2.5 rounded-full bg-emerald-400 animate-pulse";
            }

            function animate(timestamp) {
                const elapsed = timestamp - startTime;
                const progress = Math.min(elapsed / duration, 1);
                let currentPos = startPos + (endPos - startPos) * progress;

                // กรณีจอดแช่ (Dwell simulation) ให้หยุดวิ่งที่ 50%
                if (behavior === 'DWELL' && currentPos >= 45 && currentPos <= 55) {
                    currentPos = 50; 
                    cartEl.style.left = `50%`;
                    
                    // ปิดล้อหมุนขณะจอดนิ่ง
                    cartEl.querySelectorAll('.wheel-spin').forEach(w => w.classList.remove('wheel-spin'));
                    
                    // ดึงคลื่นสัญญาณวิทยุและเริ่มทดสอบสแกนรัวๆ
                    triggerDwellScanPollingLoop(cartId, direction, cartEl);
                    return; 
                }

                cartEl.style.left = `${currentPos}%`;

                // การตรวจจับสัญญาณสแกนคลื่น UHF (ช่วงตำแหน่ง 35% - 65%)
                if (currentPos >= 35 && currentPos <= 65) {
                    const centerOffset = Math.abs(50 - currentPos); 
                    const rssi = -42 - Math.round(centerOffset * 3); 
                    updateRSSIMeter(rssi, true);

                    if (!hasScanned && Math.abs(50 - currentPos) < 4) {
                        hasScanned = true;
                        // ยิงข้อมูลเข้าโมดูลวิเคราะห์สถานะ
                        evaluateRFIDSignal(cartId, direction, rssi);
                    }
                } else {
                    updateRSSIMeter(-120, false);
                }

                if (progress < 1) {
                    requestAnimationFrame(animate);
                } else {
                    cartEl.remove();
                    updateRSSIMeter(-120, false);
                }
            }

            requestAnimationFrame(animate);
        }

        // --- 5. ลูปพิเศษจำลองการส่งสแกนรัวๆ กรณีที่จอดหยุดนิ่งใต้เซนเซอร์ ---
        function triggerDwellScanPollingLoop(cartId, direction, cartElement) {
            let scanCounts = 0;
            const maxDwellScans = 6; // สแกนทั้งหมด 6 รอบ รอบละ 2.5 วินาที เพื่อทดสอบป้องกันการข้ามสถานะ
            const led = cartElement.querySelector(`#led-${cartId}`);
            const visualLock = cartElement.querySelector(`#cart-lock-visual-${cartId}`);
            const visualLockTime = cartElement.querySelector(`#cart-lock-time-${cartId}`);
            
            logSQLServer(`-- [DWELL SCENARIO] ${cartId} parked directly under the UHF sensor beam --`);
            
            function scanCycle() {
                if (scanCounts >= maxDwellScans) {
                    // ปล่อยให้วิ่งออกไปนอกพื้นที่หลังจากจอดแช่จนครบการเทส
                    logSQLServer(`-- [DWELL SCENARIO COMPLETED] ${cartId} is resuming travel --`);
                    visualLock.classList.add('hidden');
                    resumeTravel(cartElement, direction);
                    return;
                }

                scanCounts++;
                const rssi = -41 - Math.round(Math.random() * 5); // สัญญาณเข้มแรงตลอดเวลาเพราะจอดใต้เสา
                updateRSSIMeter(rssi, true);
                
                // ดำเนินการวิเคราะห์พฤติกรรมการสแกน
                evaluateRFIDSignal(cartId, direction, rssi);

                // อัปเดตรูปแม่กุญแจที่ตัวโมดูลรถเข็นให้เห็นว่าระบบกำลังบล็อก
                const cartObj = dbCarts.find(c => c.id === cartId);
                if (cartObj && cartObj.cooldownRemaining > 0) {
                    led.className = "w-2.5 h-2.5 rounded-full bg-red-500 animate-pulse";
                    visualLock.classList.remove('hidden');
                    visualLockTime.innerText = `${cartObj.cooldownRemaining}s`;
                } else {
                    led.className = "w-2.5 h-2.5 rounded-full bg-emerald-500 animate-pulse";
                    visualLock.classList.add('hidden');
                }

                // สแกนซ้ำครั้งถัดไปในอีก 2.5 วินาที
                setTimeout(scanCycle, 2500);
            }

            scanCycle();
        }

        // ตัวประคองการวิ่งต่อหลังจากจอดแช่
        function resumeTravel(cartElement, direction) {
            const startTime = performance.now();
            const duration = 2500;
            let startPos = 50;
            let endPos = direction === 'ENTER' ? 115 : -15;

            // หมุนล้อกลับมาใหม่เพื่อความสวยงาม
            cartElement.querySelectorAll('.w-1.5').forEach(w => w.classList.add('wheel-spin'));

            function step(timestamp) {
                const elapsed = timestamp - startTime;
                const progress = Math.min(elapsed / duration, 1);
                const currentPos = startPos + (endPos - startPos) * progress;
                cartElement.style.left = `${currentPos}%`;

                if (progress < 1) {
                    requestAnimationFrame(step);
                } else {
                    cartElement.remove();
                    updateRSSIMeter(-120, false);
                }
            }
            requestAnimationFrame(step);
        }

        // --- 6. หัวใจสำคัญ: อัลกอริทึมกรองข้อมูลสแกน (Core Algorithm) ---
        function evaluateRFIDSignal(cartId, actionType, rssi) {
            const now = Date.now();
            const cart = dbCarts.find(c => c.id === cartId);
            
            if (!cart) return;

            const timeDiff = now - cart.lastScannedTime;

            // เคสที่ 1: ตรวจจับครั้งแรก หรือพ้นกำหนดคูลดาวน์ 120 วินาทีไปแล้ว
            if (cart.lastScannedTime === 0 || timeDiff >= 120000) {
                
                // ตรวจสอบเพิ่มเติมว่าถ้าเคยสแกนไปแล้วและจอดแช่เดิมนานกว่า 120 วิ และยังอยู่จุดเดิม 
                // (ให้มองว่าเป็นประวัติการจอดแช่ต่อเนื่อง DWELL ดีกว่าจะไปกลับสถานะซ้ำซ้อน)
                if (cart.lastScannedTime > 0 && cart.isStationary) {
                    registerDwellEvent(cart, rssi, now, `STATIONARY (DWELL OVER 120s)`);
                } else {
                    // การทำงานปกติ เปลี่ยนแปลงสถานะ (ENTER <-> EXIT)
                    executeStateChange(cart, actionType, rssi, now);
                }
            } 
            // เคสที่ 2: สแกนซ้ำรวดเร็วภายใน 120 วินาที (ติดล็อก Cooldown)
            else {
                cart.isStationary = true;
                registerDwellEvent(cart, rssi, now, `STATIONARY (BLOCK FLAPPING)`);
            }

            renderAllData();
        }

        // จัดการเปลี่ยนสถานะจริงๆ (เมื่อตรวจสอบแล้วว่าไม่ใช่สแกนซ้ำซ้อน)
        function executeStateChange(cart, actionType, rssi, timestamp) {
            const prevStatus = cart.status;
            cart.status = actionType === 'ENTER' ? 'IN' : 'OUT';
            cart.lastScannedTime = timestamp;
            cart.cooldownRemaining = 120;
            cart.isStationary = false;

            playBeep(1100, 0.08); // เสียงสแกนปกติสำเร็จ
            showToast(cart.id, cart.status, `${rssi} dBm`, false);

            // ส่งข้อมูล SQL Server Log
            logSQLServer(`-- [SQL INGEST] RFID Scan validated for ${cart.id} --`);
            logSQLServer(`BEGIN TRANSACTION;`);
            logSQLServer(`UPDATE Carts SET CurrentStatus = '${cart.status}', LastUpdated = GETDATE() WHERE CartID = '${cart.id}';`);
            
            const currentLogId = logCounter++;
            const newLog = {
                logId: currentLogId,
                cartId: cart.id,
                action: cart.status,
                effect: `STATUS CHANGED (${prevStatus} -> ${cart.status})`,
                rssi: `${rssi} dBm`,
                cooldown: `2m Lock Active 🔒`,
                time: new Date(timestamp)
            };
            dbLogs.unshift(newLog);

            // ส่งข้อมูลไปยัง Node.js Backend เพื่อเซฟลง SQL Server จริงๆ
            sendToBackend(newLog);

            logSQLServer(`INSERT INTO CartLogs (CartID, ReaderLocation, ActionType, RSSI) VALUES ('${cart.id}', 'GATE_A_CEILING', '${cart.status}', '${rssi} dBm');`);
            logSQLServer(`COMMIT; -- SUCCESS: Record updated successfully.`);
            logSQLServer(` `);
        }

        // บันทึกกิจกรรมการหยุดอยู่ที่เดิม (Dwell/Stationary) เพื่อไม่ให้สลับสถานะ
        function registerDwellEvent(cart, rssi, timestamp, reasonType) {
            playBeep(500, 0.05); // เสียงบี๊บเตือนความถี่ต่ำแสดงว่า "ตรวจพบการจอดแช่"
            showToast(cart.id, `STATIONARY`, `${rssi} dBm`, true);

            logSQLServer(`-- [ALGORITHM BLOCKED] ${cart.id} scanned again under ceiling sensor within cooldown --`);
            logSQLServer(`-- Reason: [${reasonType}] - State remains '${cart.status}' to prevent flip-flop --`);
            
            const currentLogId = logCounter++;
            const newLog = {
                logId: currentLogId,
                cartId: cart.id,
                action: 'STATIONARY',
                effect: `REJECTED CHANGE (Remain ${cart.status})`,
                rssi: `${rssi} dBm`,
                cooldown: `Lock Active (${cart.cooldownRemaining}s)`,
                time: new Date(timestamp)
            };
            dbLogs.unshift(newLog);
            
            // ส่งข้อมูลไปยัง Node.js Backend เพื่อเซฟลง SQL Server จริงๆ
            sendToBackend(newLog);
            
            logSQLServer(`-- No SQL Update performed on Carts table. Safety filter active.`);
            logSQLServer(` `);
        }

        // --- 7. เครื่องปรับแต่งและเรนเดอร์ UI คอนโซล ---
        function updateRSSIMeter(rssi, isActive) {
            const bar = document.getElementById('rssi-bar');
            const val = document.getElementById('rssi-val');
            const rfidField = document.getElementById('rfid-field');
            const antLED = document.getElementById('antenna-led');

            if (isActive) {
                rfidField.className = "w-1/3 h-full border-x-2 border-dashed border-emerald-500 bg-emerald-500/10 flex flex-col justify-end pb-4 items-center relative transition-colors duration-200";
                antLED.className = "w-2.5 h-2.5 rounded-full bg-emerald-500 animate-ping";
                const percentage = Math.min(100, Math.max(0, ((rssi + 120) / 80) * 100));
                bar.style.width = `${percentage}%`;
                val.innerText = `${rssi} dBm`;
                val.className = "text-[9px] text-emerald-400 mt-0.5 font-bold";
            } else {
                rfidField.className = "w-1/3 h-full border-x-2 border-dashed border-emerald-500/20 bg-emerald-500/5 flex flex-col justify-end pb-4 items-center relative transition-colors duration-200";
                antLED.className = "w-2.5 h-2.5 rounded-full bg-red-500";
                bar.style.width = "0%";
                val.innerText = "-120 dBm (Idle)";
                val.className = "text-[9px] text-slate-500 mt-0.5";
            }
        }

        function renderAllData() {
            const total = dbCarts.length;
            const ins = dbCarts.filter(c => c.status === 'IN').length;
            const stationary = dbCarts.filter(c => c.isStationary).length;
            const scans = dbLogs.length;

            document.getElementById('stat-total').innerText = total;
            document.getElementById('stat-in').innerText = ins;
            document.getElementById('stat-stationary').innerText = stationary;
            document.getElementById('stat-out').innerText = total - ins; // จะอัปเดต Element OUT ที่เพิ่งใส่ไปใหม่ได้อย่างราบรื่น
            document.getElementById('stat-scans').innerText = scans;

            // 1. วาดตารางสถานะ Carts Table
            const tblCarts = document.getElementById('tbl-carts');
            tblCarts.innerHTML = '';
            
            dbCarts.forEach(cart => {
                const tr = document.createElement('tr');
                tr.className = "hover:bg-slate-900 border-b border-slate-900";
                
                const badge = cart.status === 'IN' 
                    ? '<span class="px-2 py-0.5 rounded-full text-[10px] bg-blue-500/10 text-blue-400 border border-blue-500/30 font-bold">IN-STORE</span>'
                    : '<span class="px-2 py-0.5 rounded-full text-[10px] bg-slate-800 text-slate-400 border border-slate-700 font-bold">OUTSIDE</span>';
                
                let lockBadge = '';
                if (cart.cooldownRemaining > 0) {
                    lockBadge = `<span class="px-2 py-0.5 rounded text-[10px] bg-red-950 text-red-400 border border-red-900/30 font-bold"><i class="fa-solid fa-lock mr-1 text-[8px]"></i>LOCK: ${cart.cooldownRemaining}s</span>`;
                } else {
                    lockBadge = `<span class="text-emerald-500 text-[11px]"><i class="fa-solid fa-unlock"></i> Ready</span>`;
                }

                tr.innerHTML = `
                    <td class="py-3 font-bold text-slate-300 flex items-center gap-2">
                        <i class="fa-solid fa-cart-flatbed text-slate-500 text-[10px]"></i> ${cart.id}
                    </td>
                    <td class="py-3 text-center">${badge}</td>
                    <td class="py-3 text-right">${lockBadge}</td>
                `;
                tblCarts.appendChild(tr);
            });

            // 2. วาดตารางประวัติ Log
            const tblHistory = document.getElementById('tbl-history');
            const emptyRow = document.getElementById('empty-history-row');
            
            if (dbLogs.length > 0) {
                if (emptyRow) emptyRow.style.display = 'none';
                tblHistory.innerHTML = '';
                
                dbLogs.forEach((log, index) => {
                    const tr = document.createElement('tr');
                    let bgClass = "hover:bg-slate-900/40";
                    if (index === 0) {
                        bgClass = log.action === 'STATIONARY' ? "bg-amber-500/5 hover:bg-amber-500/10 border-l-2 border-amber-500" : "bg-emerald-500/5 hover:bg-emerald-500/10 border-l-2 border-emerald-500";
                    }

                    tr.className = `border-b border-slate-900 ${bgClass}`;
                    
                    let actBadge = '';
                    if (log.action === 'ENTER' || log.action === 'IN') {
                        actBadge = '<span class="text-blue-400 font-bold"><i class="fa-solid fa-arrow-right-to-bracket mr-1"></i> ENTER (ขาเข้า)</span>';
                    } else if (log.action === 'EXIT' || log.action === 'OUT') {
                        actBadge = '<span class="text-slate-400 font-bold"><i class="fa-solid fa-arrow-right-from-bracket mr-1"></i> EXIT (ขาออก)</span>';
                    } else {
                        actBadge = '<span class="text-yellow-500 font-bold"><i class="fa-solid fa-hourglass-half mr-1"></i> STATIONARY (แช่เดิม)</span>';
                    }

                    tr.innerHTML = `
                        <td class="p-3 text-slate-500">#${log.logId}</td>
                        <td class="p-3 font-bold text-slate-300">${log.cartId}</td>
                        <td class="p-3">${actBadge}</td>
                        <td class="p-3 text-slate-400 font-sans">${log.effect}</td>
                        <td class="p-3 text-cyan-400">${log.rssi}</td>
                        <td class="p-3 text-slate-500">${log.cooldown}</td>
                        <td class="p-3 text-slate-500 text-[10px]">${formatTime(log.time)}</td>
                    `;
                    tblHistory.appendChild(tr);
                });
            }
        }

        // --- 8. แสดง Popup ด้านล่างขวาเสมือนจริง ---
        function showToast(cartId, action, rssi, isBlocked) {
            const toast = document.getElementById('toast');
            const toastBody = document.getElementById('toast-body');
            const toastTitle = document.getElementById('toast-title');
            const toastIconBox = document.getElementById('toast-icon-box');

            if (isBlocked) {
                toastTitle.innerText = "🛑 ANTI-FLAP SYSTEM ACTIVE";
                toastBody.innerHTML = `พบรถเข็นคันเดิม: <span class="text-yellow-400 font-bold">${cartId}</span><br>จอดแช่เดิมใต้เซนเซอร์ | ทำการบล็อกการสลับสถานะ!`;
                toastIconBox.className = "p-2 bg-yellow-500/20 text-yellow-500 rounded-lg";
                toast.className = "fixed bottom-5 right-5 bg-slate-900 border-2 border-yellow-500/50 text-slate-100 px-6 py-4 rounded-xl shadow-2xl transform transition-all duration-300 flex items-center gap-4 z-50";
            } else {
                toastTitle.innerText = "📡 UHF RFID TAG VALIDATED";
                toastBody.innerHTML = `รหัสสินค้า: <span class="text-emerald-400 font-bold">${cartId}</span><br>เปลี่ยนตำแหน่งสำเร็จ (${action}) | กำลังส่ง: ${rssi}`;
                toastIconBox.className = "p-2 bg-emerald-500/20 text-emerald-400 rounded-lg";
                toast.className = "fixed bottom-5 right-5 bg-slate-900 border-2 border-emerald-500/50 text-slate-100 px-6 py-4 rounded-xl shadow-2xl transform transition-all duration-300 flex items-center gap-4 z-50";
            }
            
            toast.style.transform = 'translateY(0)';
            toast.style.opacity = '1';
            
            setTimeout(() => {
                toast.style.transform = 'translateY(20px)';
                toast.style.opacity = '0';
            }, 2300);
        }

        function formatTime(d) {
            return d.toLocaleTimeString('th-TH') + '.' + d.getMilliseconds().toString().padStart(3, '0');
        }

        function formatSQLDate(d) {
            return d.toISOString().replace('T', ' ').substring(0, 19);
        }

        // --- ฟังก์ชันสำหรับส่งข้อมูลไปยัง Node.js (SQL Server Backend) ---
        function sendToBackend(logData) {
            fetch(LOCAL_API_URL, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    cartId: logData.cartId,
                    action: logData.action,
                    effect: logData.effect,
                    rssi: logData.rssi,
                    time: logData.time.toISOString()
                })
            }).then(response => response.json())
              .then(data => {
                  console.log(`[SQL Server] Data Synced:`, data);
              }).catch(err => {
                  console.error("[SQL Server] Connection Failed (ตรวจสอบว่ารัน server.js หรือยัง):", err);
              });
        }

        function logSQLServer(message) {
            const consoleLog = document.getElementById('console-log');
            const logLine = document.createElement('div');
            
            if (message.startsWith('-- [ALGORITHM')) {
                logLine.className = "text-yellow-500 font-bold";
            } else if (message.startsWith('--')) {
                logLine.className = "text-cyan-500 font-semibold";
            } else if (message.includes('BEGIN') || message.includes('COMMIT')) {
                logLine.className = "text-purple-400";
            } else {
                logLine.className = "text-emerald-400";
            }
            
            logLine.innerText = `[${new Date().toLocaleTimeString('th-TH')}] ${message}`;
            consoleLog.appendChild(logLine);
            consoleLog.scrollTop = consoleLog.scrollHeight;
        }

        function clearConsoleLog() {
            document.getElementById('console-log').innerHTML = '<div class="text-slate-500">[Log cleared. MSSQL is listening...]</div>';
        }

        // --- 9. ตัวปล่อยการจำลองอัตโนมัติและแมนนวล ---
        function toggleAutoMode() {
            isAutoMode = !isAutoMode;
            const btn = document.getElementById('btn-auto');
            if (isAutoMode) {
                btn.innerHTML = `<i class="fa-solid fa-stop"></i> หยุดโหมดวิ่งอัตโนมัติ`;
                btn.className = "w-full py-2.5 bg-red-600 hover:bg-red-500 text-white font-bold rounded-lg transition text-sm flex items-center justify-center gap-2";
                logSQLServer(`-- Auto Patrol Started. Spawning random cart flow --`);
                startAutoLoop();
            } else {
                btn.innerHTML = `<i class="fa-solid fa-play"></i> เริ่มโหมดวิ่งอัตโนมัติ`;
                btn.className = "w-full py-2.5 bg-blue-600 hover:bg-blue-500 text-white font-bold rounded-lg transition text-sm flex items-center justify-center gap-2";
                logSQLServer(`-- Auto Patrol Stopped --`);
                if (autoTimer) clearTimeout(autoTimer);
            }
        }

        function startAutoLoop() {
            if (!isAutoMode) return;
            const randomCart = dbCarts[Math.floor(Math.random() * dbCarts.length)];
            const action = randomCart.status === 'OUT' ? 'ENTER' : 'EXIT';
            
            spawnCartAnimation(randomCart.id, action, 'NORMAL');
            
            const nextDelay = (13000 / simSpeed) * (0.8 + Math.random() * 0.4);
            autoTimer = setTimeout(startAutoLoop, nextDelay);
        }

        function triggerManualCart(action) {
            const targetStatus = action === 'ENTER' ? 'OUT' : 'IN';
            const candidateCart = dbCarts.find(c => c.status === targetStatus);
            
            if (candidateCart) {
                spawnCartAnimation(candidateCart.id, action, 'NORMAL');
            } else {
                const randomCart = dbCarts[Math.floor(Math.random() * dbCarts.length)];
                spawnCartAnimation(randomCart.id, action, 'NORMAL');
            }
        }

        // ตัวสำคัญ: จำลองจอดแช่
        function triggerStationaryDwellSim() {
            const randomCart = dbCarts[Math.floor(Math.random() * dbCarts.length)];
            // เลือกขี่ไปฝั่งไหนก็ได้ แต่เมื่อถึงใจกลางแล้วจะหยุดจอดแช่
            const action = randomCart.status === 'OUT' ? 'ENTER' : 'EXIT';
            
            spawnCartAnimation(randomCart.id, action, 'DWELL');
        }

        // --- 10. ระบบ Modal สำหรับคลิกดูรายชื่อรถเข็น (แยกหมวดหมู่) ---
        let currentModalFilter = null;

        function openModal(filterType) {
            currentModalFilter = filterType;
            const modal = document.getElementById('cart-list-modal');
            const title = document.getElementById('modal-title');
            
            // เปลี่ยนชื่อหัวข้อ Modal ตามที่กดมา
            if (filterType === 'ALL') title.innerHTML = '<i class="fa-solid fa-list-ul"></i> รถเข็นจดทะเบียนทั้งหมด (20 คัน)';
            else if (filterType === 'IN') title.innerHTML = '<i class="fa-solid fa-store text-blue-400"></i> รถเข็นที่อยู่ในสโตร์ (IN)';
            else if (filterType === 'OUT') title.innerHTML = '<i class="fa-solid fa-warehouse text-amber-500"></i> รถเข็นที่อยู่นอกสโตร์ (OUT)';
            else if (filterType === 'STATIONARY') title.innerHTML = '<i class="fa-solid fa-hourglass-half text-yellow-500"></i> รถเข็นที่ตรวจพบจอดแช่ (STATIONARY)';
            else if (filterType === 'SCANS') title.innerHTML = '<i class="fa-solid fa-bolt text-purple-400"></i> ประวัติการคำขอสแกนสะสมทั้งหมด (รวมบล็อก)';
            
            updateModalList();
            
            modal.classList.remove('hidden');
            setTimeout(() => {
                modal.classList.remove('opacity-0');
                modal.querySelector('div').classList.remove('scale-95');
                modal.querySelector('div').classList.add('scale-100');
            }, 10);
        }

        function closeModal() {
            const modal = document.getElementById('cart-list-modal');
            modal.classList.add('opacity-0');
            modal.querySelector('div').classList.remove('scale-100');
            modal.querySelector('div').classList.add('scale-95');
            
            setTimeout(() => {
                modal.classList.add('hidden');
                currentModalFilter = null;
            }, 300);
        }

        function updateModalList() {
            if (!currentModalFilter) return;
            
            const thead = document.getElementById('modal-table-header');
            const tbody = document.getElementById('modal-cart-list');
            tbody.innerHTML = '';
            
            if (currentModalFilter === 'SCANS') {
                // เปลี่ยนรูปแบบหัวตารางให้เหมาะกับประวัติการสแกนสะสม
                thead.innerHTML = `
                    <tr>
                        <th class="py-2">เวลาสแกน</th>
                        <th class="py-2">CART ID</th>
                        <th class="py-2 text-center">ประเภทกิจกรรม</th>
                        <th class="py-2 text-right">ผลลัพธ์ระบบ/SQL Effect</th>
                    </tr>
                `;

                if (dbLogs.length === 0) {
                    tbody.innerHTML = '<tr><td colspan="4" class="py-6 text-center text-slate-500 bg-slate-900/50">ยังไม่มีประวัติบันทึกการส่งข้อมูลเข้ามาในระบบ</td></tr>';
                    return;
                }

                dbLogs.forEach(log => {
                    const tr = document.createElement('tr');
                    tr.className = "border-b border-slate-800/80 hover:bg-slate-800/50 transition-colors text-xs";
                    
                    let actBadge = '';
                    if (log.action === 'ENTER' || log.action === 'IN') {
                        actBadge = '<span class="text-blue-400 font-bold"><i class="fa-solid fa-arrow-right-to-bracket mr-1"></i> ENTER</span>';
                    } else if (log.action === 'EXIT' || log.action === 'OUT') {
                        actBadge = '<span class="text-slate-400 font-bold"><i class="fa-solid fa-arrow-right-from-bracket mr-1"></i> EXIT</span>';
                    } else {
                        actBadge = '<span class="text-yellow-500 font-bold"><i class="fa-solid fa-hourglass-half mr-1"></i> STATIONARY</span>';
                    }

                    tr.innerHTML = `
                        <td class="py-3 text-slate-400 font-mono">${formatTime(log.time)}</td>
                        <td class="py-3 font-bold text-slate-300 font-mono">${log.cartId}</td>
                        <td class="py-3 text-center font-sans">${actBadge}</td>
                        <td class="py-3 text-right text-slate-300 font-mono text-[11px]">${log.effect}</td>
                    `;
                    tbody.appendChild(tr);
                });

            } else {
                // เปลี่ยนรูปแบบหัวตารางสำหรับการแสดงรถเข็นปกติ
                thead.innerHTML = `
                    <tr>
                        <th class="py-2">CART ID</th>
                        <th class="py-2 text-center">STATUS</th>
                        <th class="py-2 text-right">LOCK TIME (120s)</th>
                    </tr>
                `;

                // กรองข้อมูลตามหมวดหมู่ที่คลิก
                let filteredCarts = dbCarts;
                if (currentModalFilter === 'IN') filteredCarts = dbCarts.filter(c => c.status === 'IN');
                else if (currentModalFilter === 'OUT') filteredCarts = dbCarts.filter(c => c.status === 'OUT');
                else if (currentModalFilter === 'STATIONARY') filteredCarts = dbCarts.filter(c => c.isStationary);
                
                if (filteredCarts.length === 0) {
                    tbody.innerHTML = '<tr><td colspan="3" class="py-6 text-center text-slate-500 bg-slate-900/50">ไม่พบรถเข็นในสถานะนี้</td></tr>';
                    return;
                }

                // เรียงให้คันที่ติด Lock ขึ้นมาก่อน
                filteredCarts.sort((a, b) => b.cooldownRemaining - a.cooldownRemaining);

                filteredCarts.forEach(cart => {
                    const tr = document.createElement('tr');
                    tr.className = "border-b border-slate-800/80 hover:bg-slate-800/50 transition-colors";
                    
                    const badge = cart.status === 'IN' 
                        ? '<span class="px-2 py-0.5 rounded-full text-[10px] bg-blue-500/10 text-blue-400 border border-blue-500/30 font-bold">IN</span>'
                        : '<span class="px-2 py-0.5 rounded-full text-[10px] bg-slate-800 text-slate-400 border border-slate-700 font-bold">OUT</span>';
                    
                    let lockBadge = '';
                    if (cart.cooldownRemaining > 0) {
                        lockBadge = `<span class="px-2 py-1 rounded text-[11px] bg-red-950/80 text-red-400 border border-red-900/30 font-bold tracking-wider"><i class="fa-solid fa-lock mr-1"></i>${cart.cooldownRemaining}s</span>`;
                    } else {
                        lockBadge = `<span class="text-emerald-500/70 text-[11px]"><i class="fa-solid fa-check"></i> พร้อมใช้งาน</span>`;
                    }

                    tr.innerHTML = `
                        <td class="py-3 font-bold text-slate-300 flex items-center gap-2">
                            <i class="fa-solid fa-cart-flatbed text-slate-500 text-[10px]"></i> ${cart.id}
                            ${cart.isStationary ? '<span class="text-yellow-500 text-[10px] ml-1" title="จอดแช่"><i class="fa-solid fa-hourglass-half"></i></span>' : ''}
                        </td>
                        <td class="py-3 text-center">${badge}</td>
                        <td class="py-3 text-right pr-2">${lockBadge}</td>
                    `;
                    tbody.appendChild(tr);
                });
            }
        }

        window.onload = function() {
            renderAllData();
            startCooldownTracker();
        };

    </script>
</body>
</html>
