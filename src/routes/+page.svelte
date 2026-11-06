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

      const nowDateObj = new Date();
      const currentHour = nowDateObj.getHours();
      const currentMinute = nowDateObj.getMinutes();
      const currentSecond = nowDateObj.getSeconds();
      
      const totalSeconds = currentHour * 3600 + currentMinute * 60 + currentSecond;

      const time8_29_59 = 8 * 3600 + 29 * 60 + 59;   // 08:29:59
      const time11_59_59 = 11 * 3600 + 59 * 60 + 59; // 11:59:59
      const time12_59_59 = 12 * 3600 + 59 * 60 + 59; // 12:59:59
      const time17_29_59 = 17 * 3600 + 29 * 60 + 59; // 17:29:59

      const endWork = new Date();
      endWork.setHours(17, 30, 0, 0);

      if (totalSeconds < time8_29_59) {
        isBeforeWork = true;
        isAfterWork = false;
        workStatusText = 'ยังไม่ถึงเวลาเริ่มงาน (08:30 น.)';
        workStatusBadgeClass = 'text-amber-300 border-amber-500/30 bg-amber-500/10';
        dotColorClass = 'bg-amber-400'; 
      } else if (totalSeconds >= time8_29_59 && totalSeconds <= time11_59_59) {
        isBeforeWork = false;
        isAfterWork = false;
        workStatusText = 'ช่วงเวลาทำงาน (รอบเช้า)';
        workStatusBadgeClass = 'text-emerald-300 border-emerald-500/30 bg-emerald-500/10';
        dotColorClass = 'bg-emerald-400'; 
      } else if (totalSeconds > time11_59_59 && totalSeconds <= time12_59_59) {
        isBeforeWork = false;
        isAfterWork = false;
        workStatusText = 'ช่วงเวลาพักกินข้าว';
        workStatusBadgeClass = 'text-sky-300 border-sky-500/30 bg-sky-500/10';
        dotColorClass = 'bg-sky-400'; 
      } else if (totalSeconds > time12_59_59 && totalSeconds <= time17_29_59) {
        isBeforeWork = false;
        isAfterWork = false;
        workStatusText = 'ช่วงเวลาทำงาน (รอบบ่าย)';
        workStatusBadgeClass = 'text-emerald-300 border-emerald-500/30 bg-emerald-500/10';
        dotColorClass = 'bg-emerald-400'; 
      } else {
        isBeforeWork = false;
        isAfterWork = true;
        workStatusText = 'เลิกงานแล้ว พักผ่อนได้!';
        workStatusBadgeClass = 'text-rose-400 border-rose-500/30 bg-rose-500/10';
        dotColorClass = 'bg-rose-500'; 
      }

      const startWorkToday = new Date();
      startWorkToday.setHours(8, 30, 0, 0);
      const totalWorkMillis = endWork.getTime() - startWorkToday.getTime();

      if (nowDateObj < startWorkToday) {
        todayHoursLeft = 9;
        todayMinutesLeft = 0;
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

  let dotColorClass = $state('');
</script>

<!-- ล็อกหน้าจอเต็มพื้นที่ จัดกึ่งกลางพอดีเป๊ะทุกขนาดหน้าจอ ไร้ Scrollbar -->
<div class="fixed inset-0 w-screen h-screen flex flex-col items-center justify-center gap-3 sm:gap-4 py-3 px-3 sm:py-6 sm:px-6 bg-black overflow-hidden select-none font-sans text-neutral-200 max-w-xl mx-auto">
  
  <!-- ส่วนหัว -->
  <div class="text-center w-full">
    <h1 class="text-[10px] sm:text-xs uppercase tracking-[0.25em] text-neutral-500 font-semibold mb-0.5">Co-op Internship</h1>
    <p class="text-white text-sm sm:text-base font-medium tracking-tight">Donaus</p>
  </div>
  
  {#if !isMounted}
    <div class="text-neutral-600 animate-pulse text-sm font-light">Loading...</div>
  {:else if isFinished}
    <div class="text-2xl font-light text-emerald-400 animate-pulse">
      Freedom
    </div>
  {:else}
    
    <!-- เนื้อหาหลักจัดวางสัดส่วนกลางจอ -->
    <div class="w-full flex flex-col gap-2.5 sm:gap-3">

      <!-- ระยะเวลาสะสม -->
      <div class="bg-neutral-900/40 backdrop-blur-xl px-4 py-3 sm:py-4 rounded-2xl flex items-center justify-between text-left border border-white/10 shadow-[0_8px_30px_rgb(0,0,0,0.12)]">
        <div>
          <div class="text-[10px] sm:text-[11px] text-neutral-400 font-medium tracking-wide">ระยะเวลาสะสม</div>
          <div class="text-xs text-neutral-300 font-normal mt-0.5">ฝึกงานมาแล้วทั้งหมด</div>
        </div>
        <div class="flex items-baseline">
          <span class="text-4xl sm:text-6xl font-black text-white tracking-tight drop-shadow-md">{trainedDays}</span>
          <span class="text-xs sm:text-sm text-neutral-400 ml-2 font-medium">วัน</span>
        </div>
      </div>

      <!-- นับถอยหลังเลิกงานวันนี้ -->
      <div class="bg-neutral-900/40 backdrop-blur-xl px-4 py-3 sm:py-3.5 rounded-2xl text-left border border-white/10 shadow-[0_8px_30px_rgb(0,0,0,0.12)]">
        
        <!-- ป้ายสถานะการทำงานเด่นชัด -->
        <div class="mb-2.5">
          <div class="inline-flex items-center gap-2 px-3 py-1 rounded-full text-xs font-bold tracking-wide border border-white/15 bg-white/[0.08] backdrop-blur-md shadow-sm {workStatusBadgeClass}">
            <span class="w-2 h-2 rounded-full {dotColorClass} animate-pulse shadow-[0_0_8px_currentColor]"></span>
            {workStatusText}
          </div>
        </div>

        <div class="flex items-center justify-between mb-1.5">
          <div class="text-xs text-neutral-400 font-medium tracking-wide flex items-center gap-1.5">
            <span class="inline-block animate-spin-slow">⏳</span> นับถอยหลังเลิกงาน
          </div>
          <div class="text-xs text-neutral-400 font-medium">{todayWorkProgress.toFixed(1)}%</div>
        </div>

        <div class="mb-2.5 relative w-full bg-black/40 rounded-full h-1.5 overflow-hidden border border-white/5">
          <div class="relative h-full rounded-full bg-white transition-all duration-1000 ease-linear shadow-[0_0_8px_rgba(255,255,255,0.6)]" style="width: {todayWorkProgress}%"></div>
        </div>
        
        {#if isBeforeWork}
          <div class="text-xs font-light text-neutral-400 text-center py-1">เริ่มงาน 08:30 น.</div>
        {:else if isAfterWork}
          <div class="text-xs font-light text-emerald-400 text-center py-1">เสร็จสิ้นภารกิจวันนี้แล้ว</div>
        {:else}
          <div class="grid grid-cols-3 gap-2 text-center">
            <div class="bg-white/[0.03] backdrop-blur-md rounded-xl py-1.5 sm:py-2 border border-white/5">
              <span class="text-base sm:text-lg font-light text-white">{todayHoursLeft}</span>
              <span class="text-[10px] text-neutral-500 ml-0.5">ชม.</span>
            </div>
            <div class="bg-white/[0.03] backdrop-blur-md rounded-xl py-1.5 sm:py-2 border border-white/5">
              <span class="text-base sm:text-lg font-light text-white">{todayMinutesLeft}</span>
              <span class="text-[10px] text-neutral-500 ml-0.5">นาที</span>
            </div>
            <div class="bg-white/[0.03] backdrop-blur-md rounded-xl py-1.5 sm:py-2 border border-white/5">
              <span class="text-base sm:text-lg font-light text-white">{todaySecondsLeft}</span>
              <span class="text-[10px] text-neutral-500 ml-0.5">วินาที</span>
            </div>
          </div>
        {/if}
      </div>

      <!-- กล่องนับเวลาหลัก 4 ช่อง -->
      <div class="grid grid-cols-4 gap-2">
        {#each [{val: days, label: 'วัน'}, {val: hours, label: 'ชม.'}, {val: minutes, label: 'นาที'}, {val: seconds, label: 'วินาที'}] as item}
          <div class="bg-neutral-900/60 border border-neutral-800/40 py-2.5 sm:py-3 px-1 rounded-2xl text-center shadow-sm">
            <div class="text-lg sm:text-2xl font-light text-white tracking-tight">{item.val}</div>
            <div class="text-[10px] text-neutral-500 mt-0.5 font-medium">{item.label}</div>
          </div>
        {/each}
      </div>

      <!-- วันทำงานจริงเชิงลึก -->
      <div class="pt-2 border-t border-neutral-800/50 w-full text-left">
        <div class="text-[11px] text-neutral-400 font-medium tracking-wide mb-1.5 px-0.5">วันทำงานจริง (หยุดทุกวันอาทิตย์)</div>
        <div class="grid grid-cols-4 gap-2">
          <div class="bg-neutral-900/40 py-1.5 sm:py-2 rounded-xl text-center">
            <div class="text-xs sm:text-sm font-medium text-white">{workDays}</div>
            <div class="text-[9px] text-neutral-500">วันทำงาน</div>
          </div>
          <div class="bg-neutral-900/40 py-1.5 sm:py-2 rounded-xl text-center">
            <div class="text-xs sm:text-sm font-medium text-neutral-300">{hours}</div>
            <div class="text-[9px] text-neutral-500">ชั่วโมง</div>
          </div>
          <div class="bg-neutral-900/40 py-1.5 sm:py-2 rounded-xl text-center">
            <div class="text-xs sm:text-sm font-medium text-neutral-300">{minutes}</div>
            <div class="text-[9px] text-neutral-500">นาที</div>
          </div>
          <div class="bg-neutral-900/40 py-1.5 sm:py-2 rounded-xl text-center">
            <div class="text-xs sm:text-sm font-medium text-neutral-300">{seconds}</div>
            <div class="text-[9px] text-neutral-500">วินาที</div>
          </div>
        </div>
      </div>

    </div>

    <!-- ส่วนท้าย (ความคืบหน้าโครงการ) -->
    <div class="w-full text-center">
      <div class="mb-1.5">
        <div class="flex justify-between text-[11px] text-neutral-400 font-medium mb-1 px-0.5">
          <span>ไม่นานเกินรอ</span>
          <span class="text-white font-medium">{progress.toFixed(6)}%</span>
        </div>
        <div class="relative w-full bg-neutral-900 rounded-full h-5 overflow-hidden">
          <div class="relative h-full rounded-full bg-white transition-all duration-1000 ease-linear" style="width: {progress}%"></div>
        </div>
      </div>
      <div class="text-neutral-500 text-[10px] font-light tracking-wide mt-1">
        02.11.2026 — 26.02.2027
      </div>
    </div>
  {/if}
</div>

<style>
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