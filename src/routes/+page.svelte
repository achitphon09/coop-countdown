<script>
  import { onMount } from 'svelte';

  const startDate = new Date('2026-11-02T00:00:00').getTime();
  const endDate = new Date('2027-02-26T00:00:00').getTime();

  let days = $state(0);
  let hours = $state(0);
  let minutes = $state(0);
  let seconds = $state(0);
  let workDays = $state(0);
  let trainedDays = $state(0);
  let progress = $state(0);

  // ตัวแปรสำหรับสถานะและนับถอยหลังรายวัน
  let todayWorkProgress = $state(0);
  let todayHoursLeft = $state(0);
  let todayMinutesLeft = $state(0);
  let todaySecondsLeft = $state(0);
  let workStatusText = $state('');
  let workStatusBadgeClass = $state('');
  let isAfterWork = $state(false);
  let isBeforeWork = $state(false);

  let isFinished = $state(false);
  let isMounted = $state(false);

  function calculateTime() {
    const now = new Date().getTime();
    const distance = endDate - now;
    
    const totalDuration = endDate - startDate;
    const elapsed = now - startDate;

    if (distance < 0) {
      isFinished = true;
      progress = 100;
      days = 0; hours = 0; minutes = 0; seconds = 0; workDays = 0;
      trainedDays = Math.floor(totalDuration / (1000 * 60 * 60 * 24));
    } else {
      days = Math.floor(distance / (1000 * 60 * 60 * 24));
      hours = Math.floor((distance % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
      minutes = Math.floor((distance % (1000 * 60 * 60)) / (1000 * 60));
      seconds = Math.floor((distance % (1000 * 60)) / 1000);
      
      trainedDays = Math.max(0, Math.floor(elapsed / (1000 * 60 * 60 * 24)));
      progress = Math.max(0, Math.min(100, (elapsed / totalDuration) * 100));

      let sundays = 0;
      let cur = new Date(now);
      cur.setHours(0, 0, 0, 0); 
      let end = new Date(endDate);
      end.setHours(0, 0, 0, 0);
      
      while (cur < end) {
        if (cur.getDay() === 0) sundays++;
        cur.setDate(cur.getDate() + 1);
      }
      workDays = days - sundays;

      // --- จัดการช่วงเวลาทำงานรายวัน ---
      const nowDateObj = new Date();
      const currentHour = nowDateObj.getHours();
      const currentMinute = nowDateObj.getMinutes();
      const timeValue = currentHour * 60 + currentMinute; // แปลงเป็นนาทีเพื่อเทียบเช่วงเวลาได้ง่าย

      const startWorkMin = 8 * 60;       // 08:00
      const lunchStartMin = 12 * 60 + 1; // 12:01
      const lunchEndMin = 13 * 60;       // 13:00
      const afternoonStartMin = 13 * 60 + 1; // 13:01
      const endWorkMin = 17 * 60 + 30;   // 17:30

      const endWork = new Date();
      endWork.setHours(17, 30, 0, 0);

      if (timeValue < startWorkMin) {
        isBeforeWork = true;
        isAfterWork = false;
        workStatusText = 'ยังไม่ถึงเวลาเริ่มงาน (08:00 น.)';
        workStatusBadgeClass = 'bg-amber-500/20 text-amber-300 border-amber-500/30';
      } else if (timeValue >= startWorkMin && timeValue <= 12 * 60) {
        isBeforeWork = false;
        isAfterWork = false;
        workStatusText = 'ช่วงเวลาทำงาน (รอบเช้า)';
        workStatusBadgeClass = 'bg-blue-500/20 text-blue-300 border-blue-500/30';
      } else if (timeValue >= lunchStartMin && timeValue <= lunchEndMin) {
        isBeforeWork = false;
        isAfterWork = false;
        workStatusText = 'ช่วงเวลาพักกินข้าว';
        workStatusBadgeClass = 'bg-emerald-500/20 text-emerald-300 border-emerald-500/30';
      } else if (timeValue >= afternoonStartMin && timeValue <= endWorkMin) {
        isBeforeWork = false;
        isAfterWork = false;
        workStatusText = 'ช่วงเวลาทำงาน (รอบบ่าย)';
        workStatusBadgeClass = 'bg-indigo-500/20 text-indigo-300 border-indigo-500/30';
      } else {
        isBeforeWork = false;
        isAfterWork = true;
        workStatusText = 'ช่วงเวลาเลิกงานแล้ว พักผ่อนได้!';
        workStatusBadgeClass = 'bg-green-500/20 text-green-300 border-green-500/30';
      }

      // คำนวณนับถอยหลังถึง 17:30 น.
      const startWorkToday = new Date();
      startWorkToday.setHours(8, 0, 0, 0);
      const totalWorkMillis = endWork.getTime() - startWorkToday.getTime();

      if (nowDateObj < startWorkToday) {
        todayHoursLeft = 9;
        todayMinutesLeft = 30;
        todaySecondsLeft = 0;
        todayWorkProgress = 0;
      } else if (nowDateObj > endWork) {
        todayHoursLeft = 0;
        todayMinutesLeft = 0;
        todaySecondsLeft = 0;
        todayWorkProgress = 100;
      } else {
        const timeLeftToday = endWork.getTime() - nowDateObj.getTime();
        const elapsedToday = nowDateObj.getTime() - startWorkToday.getTime();

        todayHoursLeft = Math.floor((timeLeftToday % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
        todayMinutesLeft = Math.floor((timeLeftToday % (1000 * 60 * 60)) / (1000 * 60));
        todaySecondsLeft = Math.floor((timeLeftToday % (1000 * 60)) / 1000);

        todayWorkProgress = Math.max(0, Math.min(100, (elapsedToday / totalWorkMillis) * 100));
      }
    }
  }

  onMount(() => {
    isMounted = true;
    calculateTime();
    const timer = setInterval(calculateTime, 1000);
    return () => clearInterval(timer); 
  });
</script>

<div class="relative min-h-screen flex flex-col items-center justify-center p-4 bg-black overflow-hidden">

  <!-- Glassmorphism Card -->
  <div class="relative z-10 bg-white/10 backdrop-blur-xl rounded-3xl shadow-2xl p-8 max-w-lg w-full text-center border border-white/20">
    <h1 class="text-lg sm:text-xl font-extrabold text-white mb-2 tracking-wide drop-shadow-md">Countdown สหกิจศึกษา</h1>
    <p class="text-white/70 mb-8 font-medium">บริษัทดลอัสจำกัด</p>
    
    {#if !isMounted}
      <div class="py-8 text-white/50 animate-pulse font-medium">กำลังเตรียมข้อมูล...</div>
    {:else if isFinished}
      <div class="text-3xl font-bold text-green-400 animate-bounce py-8 drop-shadow-md">
        อิสรภาพ
      </div>
    {:else}
      
      <!-- ส่วนแสดงจำนวนวันที่ฝึกไปแล้ว -->
      <div class="mb-3 bg-zinc-800/90 backdrop-blur-xl px-4 py-3 rounded-2xl shadow-[0_10px_25px_rgba(0,0,0,0.5),inset_0_2px_4px_rgba(255,255,255,0.2),inset_0_-3px_6px_rgba(0,0,0,0.5)] border border-white/10 flex items-center justify-between text-left">
  <div>
    <div class="text-xs text-zinc-400 font-semibold tracking-wider uppercase">ระยะเวลาสะสม</div>
    <div class="text-md text-white/90 font-medium mt-0.5">ฝึกงานมาแล้วทั้งหมด</div>
  </div>
  
  <div class="flex items-baseline bg-zinc-900/90 px-3.5 py-1.5 rounded-xl border border-white/10 shadow-[0_4px_10px_rgba(0,0,0,0.4),inset_0_1px_2px_rgba(255,255,255,0.15),inset_0_-2px_4px_rgba(0,0,0,0.6)]">
    <span class="text-5xl sm:text-6xl font-black text-white drop-shadow-sm tracking-tight">{trainedDays}</span>
    <span class="text-xs text-zinc-400 ml-1.5 font-bold">วัน</span>
  </div>
</div>

<!-- ส่วนนับถอยหลังเลิกงานวันนี้ พร้อมแบนเนอร์บอกช่วงเวลา -->
     <div class="mb-5 bg-gradient-to-b from-zinc-800/90 to-zinc-900/90 backdrop-blur-xl px-4 py-3 rounded-2xl shadow-[0_12px_30px_rgba(0,0,0,0.6),inset_0_2px_4px_rgba(255,255,255,0.25),inset_0_-4px_8px_rgba(0,0,0,0.6)] border border-white/15 text-left">
        
  <!-- แบนเนอร์บอกช่วงเวลา (Claymorphism Badge นูนเด่น) -->
  <div class="mb-2.5 px-3 py-1.5 rounded-xl border border-white/20 text-xs font-bold text-center tracking-wide shadow-[0_4px_12px_rgba(0,0,0,0.4),inset_0_1px_3px_rgba(255,255,255,0.3)] {workStatusBadgeClass}">
    {workStatusText}
  </div>

  <!-- หัวข้อและเปอร์เซ็นต์ -->
  <div class="flex items-center justify-between mb-1.5 px-0.5">
    <div class="text-[11px] text-zinc-300 font-semibold tracking-wider flex items-center gap-1">
      <span class="inline-block animate-spin-slow">⏳</span> นับถอยหลังเลิกงาน
    </div>
    <div class="text-[11px] text-amber-400 font-bold bg-black/40 px-2 py-0.5 rounded-full border border-white/10 shadow-inner">
      {todayWorkProgress.toFixed(1)}%
    </div>
  </div>

  <!-- Progress Bar สไตล์หลอดแก้ว Claymorphism -->
  <div class="mb-3 relative w-full bg-zinc-950/90 rounded-full h-3 overflow-hidden border border-white/20 shadow-[inset_0_3px_8px_rgba(0,0,0,0.9),0_2px_4px_rgba(0,0,0,0.3)]">
    <div 
      class="relative h-full rounded-full bg-gradient-to-r from-red-500 via-rose-500 to-amber-400 transition-all duration-1000 ease-linear shadow-[inset_0_1px_2px_rgba(255,255,255,0.6)] overflow-hidden" 
      style="width: {todayWorkProgress}%"
    >
      <div class="absolute inset-0 bg-gradient-to-r from-transparent via-white/40 to-transparent -translate-x-full animate-[shimmer_2.5s_infinite]"></div>
    </div>
  </div>
  
  {#if isBeforeWork}
    <div class="text-xs font-bold text-amber-300 py-2 text-center bg-black/30 rounded-xl border border-white/10 shadow-inner">
      เริ่มงาน 08:00 น. นะอย่าลืม
    </div>
  {:else if isAfterWork}
    <div class="text-xs font-bold text-green-400 py-2 text-center bg-black/30 rounded-xl border border-white/10 shadow-inner">
      วันนี้เสร็จงานแล้ว กลับบ้านดิรอไร
    </div>
  {:else}
    <!-- กล่องตัวเลขชั่วโมง นาที วินาที แบบ Inset นูนลึก -->
    <div class="grid grid-cols-3 gap-2 text-center">
      <div class="bg-zinc-950/80 rounded-xl py-1.5 px-1 border border-white/10 shadow-[0_4px_10px_rgba(0,0,0,0.5),inset_0_2px_4px_rgba(0,0,0,0.8),inset_0_-1px_2px_rgba(255,255,255,0.1)]">
        <span class="text-lg font-black text-white drop-shadow-[0_2px_4px_rgba(0,0,0,0.5)]">{todayHoursLeft}</span>
        <span class="text-[9px] text-zinc-400 ml-0.5 font-medium">ชม.</span>
      </div>
      <div class="bg-zinc-950/80 rounded-xl py-1.5 px-1 border border-white/10 shadow-[0_4px_10px_rgba(0,0,0,0.5),inset_0_2px_4px_rgba(0,0,0,0.8),inset_0_-1px_2px_rgba(255,255,255,0.1)]">
        <span class="text-lg font-black text-white drop-shadow-[0_2px_4px_rgba(0,0,0,0.5)]">{todayMinutesLeft}</span>
        <span class="text-[9px] text-zinc-400 ml-0.5 font-medium">นาที</span>
      </div>
      <div class="bg-zinc-950/80 rounded-xl py-1.5 px-1 border border-white/10 shadow-[0_4px_10px_rgba(0,0,0,0.5),inset_0_2px_4px_rgba(0,0,0,0.8),inset_0_-1px_2px_rgba(255,255,255,0.1)]">
        <span class="text-lg font-black text-white drop-shadow-[0_2px_4px_rgba(0,0,0,0.5)]">{todaySecondsLeft}</span>
        <span class="text-[9px] text-zinc-400 ml-0.5 font-medium">วินาที</span>
      </div>
    </div>
  {/if}
</div>

      <!-- กล่องนับเวลาหลัก (รวมวันหยุด) -->
      <div class="grid grid-cols-4 gap-3 sm:gap-4">
      
        <div class="relative group bg-gradient-to-br from-white/10 to-white/5 backdrop-blur-xl border border-white/10 border-t-white/30 border-l-white/30 py-4 px-2 rounded-2xl shadow-[0_8px_32px_rgba(0,0,0,0.5)] hover:-translate-y-1 transition-all duration-300 overflow-hidden">
          <div class="absolute inset-0 bg-gradient-to-br from-white/20 to-transparent opacity-0 group-hover:opacity-100 transition-opacity duration-300"></div>
          <div class="relative z-10 text-3xl sm:text-4xl font-extrabold text-transparent bg-clip-text bg-gradient-to-b from-white to-white/40 drop-shadow-lg">{days}</div>
          <div class="relative z-10 text-xs sm:text-sm text-white/50 mt-1 font-medium tracking-wider">วัน</div>
        </div>

        <div class="relative group bg-gradient-to-br from-white/10 to-white/5 backdrop-blur-xl border border-white/10 border-t-white/30 border-l-white/30 py-4 px-2 rounded-2xl shadow-[0_8px_32px_rgba(0,0,0,0.5)] hover:-translate-y-1 transition-all duration-300 overflow-hidden">
          <div class="absolute inset-0 bg-gradient-to-br from-white/20 to-transparent opacity-0 group-hover:opacity-100 transition-opacity duration-300"></div>
          <div class="relative z-10 text-3xl sm:text-4xl font-extrabold text-transparent bg-clip-text bg-gradient-to-b from-white to-white/40 drop-shadow-lg">{hours}</div>
          <div class="relative z-10 text-xs sm:text-sm text-white/50 mt-1 font-medium tracking-wider">ชั่วโมง</div>
        </div>

        <div class="relative group bg-gradient-to-br from-white/10 to-white/5 backdrop-blur-xl border border-white/10 border-t-white/30 border-l-white/30 py-4 px-2 rounded-2xl shadow-[0_8px_32px_rgba(0,0,0,0.5)] hover:-translate-y-1 transition-all duration-300 overflow-hidden">
          <div class="absolute inset-0 bg-gradient-to-br from-white/20 to-transparent opacity-0 group-hover:opacity-100 transition-opacity duration-300"></div>
          <div class="relative z-10 text-3xl sm:text-4xl font-extrabold text-transparent bg-clip-text bg-gradient-to-b from-white to-white/40 drop-shadow-lg">{minutes}</div>
          <div class="relative z-10 text-xs sm:text-sm text-white/50 mt-1 font-medium tracking-wider">นาที</div>
        </div>

        <div class="relative group bg-gradient-to-br from-white/10 to-white/5 backdrop-blur-xl border border-white/10 border-t-white/30 border-l-white/30 py-4 px-2 rounded-2xl shadow-[0_8px_32px_rgba(0,0,0,0.5)] hover:-translate-y-1 transition-all duration-300 overflow-hidden">
          <div class="absolute inset-0 bg-gradient-to-br from-white/20 to-transparent opacity-0 group-hover:opacity-100 transition-opacity duration-300"></div>
          <div class="relative z-10 text-3xl sm:text-4xl font-extrabold text-transparent bg-clip-text bg-gradient-to-b from-white to-white/40 drop-shadow-lg">{seconds}</div>
          <div class="relative z-10 text-xs sm:text-sm text-white/50 mt-1 font-medium tracking-wider">วินาที</div>
        </div>
      </div>

      <!-- วันทำงานจริง (ไม่รวมวันอาทิตย์) -->
      <div class="mt-6 pt-5 border-t border-white/10 w-full text-left">
        <div class="flex items-center space-x-2 mb-3 px-1">
          <span class="text-xs text-white/70 font-semibold tracking-wider">วันทำงานจริง (หยุดทุกวันอาทิตย์)</span>
        </div>
        
        <div class="grid grid-cols-4 gap-2 sm:gap-3">
          <div class="bg-white/5 backdrop-blur-md border border-white/10 py-3 px-1 rounded-xl text-center shadow-inner">
            <div class="text-xl sm:text-2xl font-bold text-green-300 drop-shadow-sm">{workDays}</div>
            <div class="text-[10px] text-white/50 mt-1 font-medium tracking-wider">วันทำงาน</div>
          </div>
          <div class="bg-white/5 backdrop-blur-md border border-white/10 py-3 px-1 rounded-xl text-center shadow-inner">
            <div class="text-xl sm:text-2xl font-bold text-white/90 drop-shadow-sm">{hours}</div>
            <div class="text-[10px] text-white/50 mt-1 font-medium tracking-wider">ชั่วโมง</div>
          </div>
          <div class="bg-white/5 backdrop-blur-md border border-white/10 py-3 px-1 rounded-xl text-center shadow-inner">
            <div class="text-xl sm:text-2xl font-bold text-white/90 drop-shadow-sm">{minutes}</div>
            <div class="text-[10px] text-white/50 mt-1 font-medium tracking-wider">นาที</div>
          </div>
          <div class="bg-white/5 backdrop-blur-md border border-white/10 py-3 px-1 rounded-xl text-center shadow-inner">
            <div class="text-xl sm:text-2xl font-bold text-white/90 drop-shadow-sm">{seconds}</div>
            <div class="text-[10px] text-white/50 mt-1 font-medium tracking-wider">วินาที</div>
          </div>
        </div>
      </div>

      

      <!-- Progress Bar  -->
      <div class="mt-8 text-left">
        <div class="flex justify-between text-sm text-white/80 font-bold mb-3 px-1">
          <span class="tracking-wider">ไม่นานเกินรอ</span>
          <span class="text-white drop-shadow-md text-base">{progress.toFixed(6)}%</span>
        </div>
        
        <!-- Progress Bar แนวหลอดแก้วไซเบอร์พังก์สุดล้ำ -->
        <div class="relative w-full bg-black/50 backdrop-blur-xl rounded-full h-12 overflow-hidden border border-white/25 shadow-[inset_0_4px_16px_rgba(0,0,0,0.9),0_8px_32px_rgba(0,0,0,0.4)]">
          <div 
            class="relative h-full rounded-full bg-gradient-to-r from-red-500 via-rose-500 to-amber-500 transition-all duration-1000 ease-linear shadow-[0_0_25px_rgba(167,139,250,0.8),inset_0_2px_4px_rgba(255,255,255,0.6)] overflow-hidden" 
            style="width: {progress}%"
          >
            <div class="absolute inset-0 bg-gradient-to-r from-transparent via-white/50 to-transparent -translate-x-full animate-[shimmer_2.5s_infinite]"></div>
            <div class="absolute inset-0 bg-[linear-gradient(90deg,transparent,rgba(255,255,255,0.3),transparent)] -translate-x-full animate-[wave_3s_ease-in-out_infinite]"></div>
            <div class="absolute inset-x-0 top-0 h-[45%] bg-gradient-to-b from-white/70 via-white/25 to-transparent rounded-t-2xl"></div>
            <div class="absolute inset-x-0 bottom-0 h-[20%] bg-gradient-to-t from-black/20 to-transparent"></div>
          </div>
        </div>
      </div>
      
      <div class="mt-6 inline-flex items-center justify-center space-x-2 bg-white/5 border border-white/10 px-4 py-2 rounded-full text-xs text-white/70 font-medium backdrop-blur-sm">
        <span>2 พ.ย. 2569</span>
        <span class="text-white/40">→</span>
        <span>26 ก.พ. 2570</span>
      </div>
    {/if}
  </div>
</div>

<style>
  @keyframes shimmer {
    100% { transform: translateX(200%); }
  }
  @keyframes wave {
    0% { transform: translateX(-100%); }
    100% { transform: translateX(200%); }
  }

  @keyframes shimmer {
    100% { transform: translateX(200%); }
  }

  /* อนิเมชั่นหมุนช้าๆ สำหรับนาฬิกาทราย */
  @keyframes spinSlow {
    0% { transform: rotate(0deg); }
    25% { transform: rotate(15deg); }
    75% { transform: rotate(-15deg); }
    100% { transform: rotate(0deg); }
  }

  .animate-spin-slow {
    display: inline-block;
    animation: spinSlow 3s ease-in-out infinite;
  }
</style>