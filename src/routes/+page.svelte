<script>
  import { onMount } from 'svelte';

  const startDate = new Date('2026-11-02T00:00:00').getTime();
  const endDate = new Date('2027-02-26T00:00:00').getTime();

  let days = $state(0);
  let hours = $state(0);
  let minutes = $state(0);
  let seconds = $state(0);
  let workDays = $state(0);
  let progress = $state(0);
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
    } else {
      days = Math.floor(distance / (1000 * 60 * 60 * 24));
      hours = Math.floor((distance % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
      minutes = Math.floor((distance % (1000 * 60 * 60)) / (1000 * 60));
      seconds = Math.floor((distance % (1000 * 60)) / 1000);
      
      progress = Math.max(0, Math.min(100, (elapsed / totalDuration) * 100));

      // คำนวณวันทำงานจริง (หักวันอาทิตย์ออก)
      let sundays = 0;
      let cur = new Date(now);
      cur.setHours(0, 0, 0, 0); 
      let end = new Date(endDate);
      end.setHours(0, 0, 0, 0);
      
      while (cur < end) {
        if (cur.getDay() === 0) sundays++; // 0 คือวันอาทิตย์
        cur.setDate(cur.getDate() + 1);
      }
      workDays = days - sundays;
    }
  }

  onMount(() => {
    isMounted = true;
    calculateTime();
    const timer = setInterval(calculateTime, 1000);
    return () => clearInterval(timer); 
  });
</script>

<div class="relative min-h-screen flex flex-col items-center justify-center p-4 font-sans bg-black overflow-hidden">

  <!-- Glassmorphism Card -->
  <div class="relative z-10 bg-white/10 backdrop-blur-xl rounded-3xl shadow-2xl p-8 max-w-lg w-full text-center border border-white/20">
    <h1 class="text-3xl font-extrabold text-white mb-2 tracking-wide drop-shadow-md">Countdown สหกิจศึกษา</h1>
    <p class="text-white/70 mb-8 font-medium">บริษัทดลอัสจำกัด</p>
    
    {#if !isMounted}
      <div class="py-8 text-white/50 animate-pulse font-medium">กำลังเตรียมข้อมูล...</div>
    {:else if isFinished}
      <div class="text-3xl font-bold text-green-400 animate-bounce py-8 drop-shadow-md">
        อิสรภาพ 🎉
      </div>
    {:else}
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
          <span class="text-xs text-white/70 font-semibold tracking-wider">วันทำงานจริง (หยุดเฉพาะวันอาทิตย์)</span>
        </div>
        
        <div class="grid grid-cols-4 gap-2 sm:gap-3">
          <div class="bg-white/5 backdrop-blur-md border border-white/10 py-3 px-1 rounded-xl text-center shadow-inner">
            <div class="text-xl sm:text-2xl font-bold text-lime-300 drop-shadow-sm">{workDays}</div>
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
  <!-- ตัวของเหลวเรืองแสงด้านใน -->
  <div 
    class="relative h-full rounded-full bg-gradient-to-r from-red-500 via-rose-500 to-amber-500 transition-all duration-1000 ease-linear shadow-[0_0_25px_rgba(167,139,250,0.8),inset_0_2px_4px_rgba(255,255,255,0.6)] overflow-hidden" 
    style="width: {progress}%"
  >
    <!-- ลูกเล่นที่ 1: แสงแฟลชวิ่งผ่าน (Shimmer) -->
    <div class="absolute inset-0 bg-gradient-to-r from-transparent via-white/50 to-transparent -translate-x-full animate-[shimmer_2.5s_infinite]"></div>

    <!-- ลูกเล่นที่ 2: ลายเส้นแสงระลอกคลื่นวิ่งทับ (Pulse Wave) -->
    <div class="absolute inset-0 bg-[linear-gradient(90deg,transparent,rgba(255,255,255,0.3),transparent)] -translate-x-full animate-[wave_3s_ease-in-out_infinite]"></div>

    <!-- ลูกเล่นที่ 3: ไฮไลต์แสงสะท้อนผิวกระจกด้านบน -->
    <div class="absolute inset-x-0 top-0 h-[45%] bg-gradient-to-b from-white/70 via-white/25 to-transparent rounded-t-2xl"></div>
    
    <!-- เงาสะท้อนขอบล่างของของเหลวให้ดูนูน -->
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
</style>