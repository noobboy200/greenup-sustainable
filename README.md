import React, { useEffect, useRef, useState } from "react";
import { motion, useReducedMotion } from "framer-motion";
import { Menu, X, BookOpen, Layers, Users } from "lucide-react";

/**
 * Minimal, working page with: 
 * - Usable dark/light toggle (persists to localStorage)
 * - Global Warming horizontal timeline (milestones incl. 1.2°C fixed)
 * - Basic Nav + Hero + Announcements + ESG Timeline
 * - Source & update meta under warming axis
 *
 * Self-contained: no external UI kit imports required.
 */

// ----------------------
// CONFIG
// ----------------------
const CONFIG = {
  brand: { name: "永續教育平台", logoText: "ESG Talent Hub", tagline: "永續教育訓練｜ESG 解決方案｜綠領人才媒合" },
  nav: [
    { label: "首頁", href: "#" },
    { label: "最新公告", href: "#announcements" },
    { label: "台灣綠領現況", href: "#highlights" },
    { label: "永續時程", href: "#timeline" },
    { label: "全球升溫", href: "#warming" },
    { label: "解決方案", href: "#solutions" },
    { label: "常見問題", href: "#faq" },
    { label: "聯絡我們", href: "#contact" }
  ],
  announcements: [
    { title: "最新課程上線", date: "2025-09-15", content: "永續報告書撰寫實務班開放報名" },
    { title: "合作計畫", date: "2025-09-01", content: "與上市櫃公司展開碳盤查人才培訓合作" }
  ],
  highlights: [
    { title: "綠領職缺需求", value: "每月 21,000+", desc: "台灣每月平均有 2.1 萬綠領職缺" },
    { title: "企業參與率", value: "僅 7%", desc: "目前僅約 7% 企業招募綠領人才" },
    { title: "法規驅動", value: "2026 年起", desc: "上市櫃陸續需碳盤查與揭露" }
  ],
  esgTimeline: [
    { year: 2023, rows: [
      { target: "上市櫃 ≥100 億", level: "個體公司", scopes: "S1,S2", assurance: "否", plan: "否" },
      { target: "高排放產業", level: "個體公司", scopes: "S1,S2", assurance: "否", plan: "否" }
    ]},
    { year: 2024, rows: [ { target: "≥100 億/高排放", level: "個體公司", scopes: "S1,S2", assurance: "是(個體)", plan: "否" } ] },
    { year: 2025, rows: [ { target: "≥100 億", level: "合併報表", scopes: "S1,S2", assurance: "否", plan: "是(需揭露)" }, { target: "50–100 億", level: "個體公司", scopes: "S1,S2", assurance: "否", plan: "否" } ] },
    { year: 2026, rows: [ { target: "50–100 億", level: "合併報表", scopes: "S1,S2", assurance: "否", plan: "是(需揭露)" }, { target: "<50 億", level: "個體公司", scopes: "S1,S2", assurance: "否", plan: "否" } ] },
    { year: 2027, rows: [ { target: ">=100 億", level: "合併報表", scopes: "S1,S2", assurance: "是(合併)", plan: "-" }, { target: "50–100 億", level: "個體公司", scopes: "S1,S2", assurance: "是(個體)", plan: "-" }, { target: "<50 億", level: "合併報表", scopes: "S1,S2", assurance: "否", plan: "是(需揭露)" } ] },
    { year: 2028, rows: [ { target: "50–100 億", level: "合併報表", scopes: "S1,S2", assurance: "是(合併)", plan: "-" }, { target: "<50 億", level: "個體公司", scopes: "S1,S2", assurance: "是(個體)", plan: "-" } ] },
    { year: 2029, rows: [ { target: "<50 億", level: "合併報表", scopes: "S1,S2", assurance: "是(合併)", plan: "-" } ] }
  ],
  faq: [
    { q: "是否有線上課程?", a: "提供線上與線下混成" },
    { q: "是否提供企業內訓?", a: "可客製專屬訓練方案" },
    { q: "人才媒合是否收費?", a: "對人才免費，企業依方案計費" }
  ],
  contact: { email: "info@esgtalenthub.com", phone: "+886 2-1234-5678", address: "台北市信義區松高路100號" },
  // 🔧 Added to fix runtime: CONFIG.warmingMeta used by the page under WarmingAxis
  warmingMeta: {
    lastUpdated: "2025-09-19",
    sources: [
      { label: "WMO: State of the Global Climate 2023", url: "https://library.wmo.int/records/item/62763-state-of-the-global-climate-2023" },
      { label: "IPCC AR6 Synthesis Report (SPM)", url: "https://www.ipcc.ch/report/ar6/syr/" },
      { label: "IPCC SR1.5", url: "https://www.ipcc.ch/sr15/" }
    ],
    cadence: "每月檢視一次官方發布與監測機構彙整（遇重大更新會提前）",
    note: "目前頁面採前端靜態資料呈現；接上 API 後可自動拉取最新監測值（例如 HadCRUT、NOAA、ERA5）。"
  }
} as const;

// ----------------------
// Utils
// ----------------------
function useDarkMode() {
  const [mounted, setMounted] = useState(false);
  const [isDark, setIsDark] = useState<boolean>(() => {
    if (typeof window === "undefined") return false;
    const fromLS = localStorage.getItem("theme");
    if (fromLS === "dark") return true;
    if (fromLS === "light") return false;
    return window.matchMedia?.("(prefers-color-scheme: dark)").matches ?? false;
  });
  useEffect(() => { setMounted(true); }, []);
  useEffect(() => {
    if (!mounted) return;
    const root = document.documentElement;
    if (isDark) {
      root.classList.add("dark");
      root.style.colorScheme = 'dark';
      localStorage.setItem("theme", "dark");
    } else {
      root.classList.remove("dark");
      root.style.colorScheme = 'light';
      localStorage.setItem("theme", "light");
    }
  }, [isDark, mounted]);
  return { isDark, setIsDark, mounted };
}
const onKeyToggle = (e: React.KeyboardEvent, toggle: () => void) => { if (e.key === "Enter" || e.key === " ") { e.preventDefault(); toggle(); } };
const popoverAlignClass = (pct: number) => (pct < 10 ? "left-0 translate-x-0" : pct > 90 ? "right-0 translate-x-0" : "left-1/2 -translate-x-1/2");

// ----------------------
// Nav
// ----------------------
function Nav({ items }: { items: { label: string; href: string }[] }) {
  const [open, setOpen] = useState(false);
  return (
    <nav className="sticky top-0 z-50 backdrop-blur bg-white/70 dark:bg-zinc-900/70 border-b border-zinc-200 dark:border-zinc-800">
      <div className="w-full mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex items-center justify-between h-16">
          <a href="#" className="font-bold text-xl tracking-tight"><span className="bg-clip-text text-transparent bg-gradient-to-r from-green-600 to-blue-500">{CONFIG.brand.logoText}</span></a>
          <div className="hidden md:flex items-center gap-6">
            {items.map(it => (
              <a key={it.href} href={it.href} className="text-sm text-zinc-700 hover:text-zinc-900 dark:text-zinc-300 dark:hover:text-white">{it.label}</a>
            ))}
          </div>
          <button className="md:hidden p-2" onClick={() => setOpen(o => !o)} aria-label="開啟選單">{open ? <X/> : <Menu/>}</button>
        </div>
        {open && (
          <div className="md:hidden pb-4 flex flex-col gap-3">
            {items.map(it => <a key={it.href} href={it.href} className="text-sm text-zinc-700 dark:text-zinc-300">{it.label}</a>)}
          </div>
        )}
      </div>
    </nav>
  );
}

// ----------------------
// Hero
// ----------------------
const Hero = () => (
  <section className="relative overflow-hidden">
    <div className="absolute inset-0 -z-10 bg-gradient-to-br opacity-30 blur-3xl from-green-300 via-emerald-300 to-blue-300 dark:from-green-600/40 dark:via-emerald-600/40 dark:to-blue-600/40" />
    <div className="w-full mx-auto px-4 sm:px-6 lg:px-8 py-16 lg:py-24 grid lg:grid-cols-2 gap-10 items-center">
      <div>
        <motion.h1 initial={{opacity:0,y:10}} animate={{opacity:1,y:0}} transition={{duration:0.5}} className="font-extrabold tracking-tight" style={{ fontSize: "clamp(1.75rem, 4vw, 3rem)" }}>{CONFIG.brand.tagline}</motion.h1>
        <p className="mt-4 text-lg text-zinc-600 dark:text-zinc-300 max-w-xl">提供完整的永續教育服務、ESG 解決方案與綠領人才媒合。</p>
        <div className="mt-6 flex flex-wrap gap-3">
          <a href="#solutions" className="px-4 py-2 rounded-2xl bg-gradient-to-r from-green-600 to-blue-500 text-white">探索課程</a>
          <a href="#solutions" className="px-4 py-2 rounded-2xl border border-zinc-300 bg-white text-black hover:bg-zinc-100 dark:bg-white dark:text-black dark:hover:bg-zinc-200">了解 ESG 解決方案</a>
        </div>
      </div>
      <motion.div initial={{opacity:0,y:20}} animate={{opacity:1,y:0}} transition={{duration:0.6,delay:0.1}}>
        <div className="aspect-video rounded-2xl border border-zinc-200 dark:border-zinc-800 overflow-hidden shadow-xl">
          <img loading="lazy" src="[https://sparkling-sound-34b8.hongrui169.workers.dev/ChatGPT Image 2025年9月20日 下午11_42_56 (1).png]" alt="教育平台示意" className="w-full h-full object-cover" />
        </div>
      </motion.div>
    </div>
  </section>
);

// ----------------------
// 2D Rotating Globe (Announcements visual)
// ----------------------
function Globe2D({ className = "w-48 h-48 md:w-64 md:h-64" }: { className?: string }) {
  const texture = "https://upload.wikimedia.org/wikipedia/commons/thumb/8/80/Equirectangular_projection_SW.jpg/1024px-Equirectangular_projection_SW.jpg";
  return (
    <div className={className} aria-label="Rotating 2D world map" role="img">
      <div className="relative w-full h-full rounded-full overflow-hidden shadow-xl border border-zinc-200 dark:border-zinc-800">
        <div className="absolute inset-0" style={{ backgroundImage:`url(${texture})`, backgroundRepeat:"repeat-x", backgroundSize:"200% 100%", animation:"globe-pan 30s linear infinite" }} />
        <div className="absolute inset-0 pointer-events-none bg-[radial-gradient(ellipse_at_30%_30%,rgba(255,255,255,0.25),rgba(0,0,0,0)_40%),radial-gradient(ellipse_at_70%_70%,rgba(0,0,0,0.25),rgba(0,0,0,0)_40%)]" />
      </div>
      <style>{`@keyframes globe-pan{from{background-position:0% 50%}to{background-position:-100% 50%}}`}</style>
    </div>
  );
}

// ----------------------
// ESG Timeline (compact)
// ----------------------
function YearTable({ item }: { item: any }){
  if(!item?.rows?.length) return null;
  return (
    <div className="mt-2">
      <div className="grid grid-cols-5 text-xs font-medium bg-zinc-50 dark:bg-zinc-800/60 rounded-lg">
        {['適用對象','揭露層級','盤查範疇','是否確信','目標/策略/行動'].map(h=> <div key={h} className="px-2 py-1">{h}</div>)}
      </div>
      <div className="divide-y divide-zinc-200 dark:divide-zinc-800">
        {item.rows.map((r: any, i: number) => (
          <div key={i} className="grid grid-cols-5 text-[12px]">{['target','level','scopes','assurance','plan'].map((k)=>(<div key={k} className="px-2 py-1">{r[k]}</div>))}</div>
        ))}
      </div>
    </div>
  );
}

function HorizontalTimeline(){
  const data = CONFIG.esgTimeline;
  const [activeYear, setActiveYear] = useState<number|null>(null);
  const [hoverYear, setHoverYear] = useState<number|null>(null);
  useEffect(()=>{
    const onAnyClick=()=>setHoverYear(null); const onEsc=(e:KeyboardEvent)=>{ if(e.key==="Escape") setActiveYear(null); };
    window.addEventListener("click", onAnyClick); window.addEventListener("keydown", onEsc);
    return ()=>{ window.removeEventListener("click", onAnyClick); window.removeEventListener("keydown", onEsc); };
  },[]);
  const start=2023, end=2029, range=Math.max(1,end-start); const now=new Date(); const nowYear=now.getFullYear()+now.getMonth()/12;
  const progressPct=`${Math.max(0,Math.min(100,((nowYear-start)/range)*100))}%`;
  const toPctNum=(y:number)=>((y-start)/range)*100;
  return (
    <section id="timeline" className="py-16" aria-roledescription="timeline" aria-label="ESG 時程">
      <div className="w-full mx-auto px-4 sm:px-6 lg:px-8">
        <h2 className="text-2xl font-bold text-center">永續報告書單一水平時間軸</h2>
        <p className="mt-3 text-center text-sm text-zinc-600 dark:text-zinc-300">以下為[上市櫃公司]之時程示意, 各年於 8/31 前申報; 非上市櫃/特定產業規範可能不同, 請以主管機關最新公告為準。</p>
        <div className="relative mt-10 pt-20 pb-24">
          <div className="relative h-3 rounded-full bg-zinc-200 dark:bg-zinc-800 overflow-visible shadow-inner">
            <div className="relative h-full rounded-l-full origin-left bg-gradient-to-r from-green-600 via-emerald-500 to-blue-500" style={{width:progressPct}} />
            {Array.from({length:end-start+1}).map((_,i)=>{ const y=start+i; const item=data.find(d=>d.year===y); const isActive=activeYear===y; const showHover=hoverYear===y; const pct=toPctNum(y); const align=(pct<10?"left-0 translate-x-0":pct>90?"right-0 translate-x-0":"left-1/2 -translate-x-1/2");
              return (
                <div key={y} className="absolute top-1/2 -translate-y-1/2 group cursor-pointer" style={{left:`${pct}%`}} role="button" tabIndex={0} aria-expanded={isActive} aria-label={`年份 ${y}`} onClick={(e)=>{e.stopPropagation(); setActiveYear(p=>p===y?null:y);}} onKeyDown={(e)=>onKeyToggle(e,()=>setActiveYear(p=>p===y?null:y))} onMouseEnter={()=>setHoverYear(y)} onMouseLeave={()=>setHoverYear(v=>v===y?null:v)}>
                  <div className={`bg-zinc-400 dark:bg-zinc-500 group-hover:bg-emerald-500 w-0.5 h-8 mx-auto transition`} />
                  <div className={`absolute top-full mt-2 ${align} text-xs font-medium whitespace-nowrap`}><span className={`px-1.5 py-0.5 rounded-md transition-colors md:group-hover:bg-emerald-100 md:group-hover:text-emerald-700 md:dark:group-hover:bg-emerald-900/40 md:dark:group-hover:text-emerald-200`}>{y}</span></div>
                  {item && (
                    <div className={`absolute bottom-full mb-3 w-[320px] sm:w-[420px] transition-opacity duration-200 z-40 ${align} ${isActive?"opacity-0 md:opacity-100":showHover?"opacity-100":"opacity-0 pointer-events-none"}`}>
                      <div className="relative z-20 rounded-2xl border border-zinc-200 dark:border-zinc-800 bg-white dark:bg-zinc-900 shadow-xl p-4">
                        <div className="absolute left-1/2 translate-x-[-50%] -bottom-2 w-3 h-3 rotate-45 bg-white dark:bg-zinc-900 border-b border-r border-zinc-200 dark:border-zinc-800" />
                        <div className="font-semibold mb-2">{y} 年要求</div>
                        <YearTable item={item} />
                      </div>
                    </div>
                  )}
                </div>
              );
            })}
            {/* NOW marker */}
            <div className="absolute top-1/2 -translate-y-1/2 -translate-x-1/2 z-30" style={{left:progressPct}}>
              <div className="relative flex flex-col items-center"><div className="w-2.5 h-2.5 rounded-full bg-white border-2 border-blue-500 shadow"/><div className="w-0.5 h-8 bg-gradient-to-b from-green-600 to-blue-500"/><div className="text-xs mt-1">現在 / Now</div></div>
            </div>
          </div>
        </div>
        {activeYear!==null && (
          <div className="relative z-20 mt-6" onClick={(e)=>e.stopPropagation()}>
            {(()=>{ const item=data.find(d=>d.year===activeYear); if(!item) return null; return (
              <div className="rounded-2xl border border-zinc-200 dark:border-zinc-800 bg-white dark:bg-zinc-900 shadow-xl p-5">
                <div className="flex items-center justify-between"><div className="text-lg font-semibold">{activeYear} 年要求</div><button className="text-sm text-zinc-500 hover:text-zinc-800 dark:hover:text-zinc-200" onClick={()=>setActiveYear(null)}>關閉</button></div>
                <div className="mt-3"><YearTable item={item}/></div>
              </div>
            ); })()}
          </div>
        )}
      </div>
    </section>
  );
}

// ----------------------
// Global Warming Axis (with 1.2°C interaction fix)
// ----------------------
function WarmingAxis({ baselineLabel = "1850-1900", currentWarmingC = 1.3, maxScale = 2.0 }: { baselineLabel?: string; currentWarmingC?: number; maxScale?: number }) {
  const prefersReducedMotion = useReducedMotion();
  const milestones = [
    { level:0.5, color:"neutral", title:"全球升溫 0.5°C 已發生", bullets:["高溫日、熱浪增多","部分珊瑚白化","降水型態可偵測改變"], sources:[{label:"IPCC AR6 SYR SPM", url:"https://www.ipcc.ch/report/ar6/syr/"}]},
    { level:0.8, color:"neutral", title:"全球升溫 0.8°C 已發生", bullets:["極端降雨機率上升","冰川/海冰持續衰退","颱風降雨量跡象"], sources:[{label:"IPCC AR6 SYR SPM", url:"https://www.ipcc.ch/report/ar6/syr/"}]},
    { level:1.0, color:"neutral", title:"全球升溫 1.0°C 已發生", bullets:["熱浪/乾旱/強降雨上升","海平面較前工業上升 ~20cm","野火季延長"], sources:[{label:"IPCC AR6 SYR SPM", url:"https://www.ipcc.ch/report/ar6/syr/"}]},
    { level:1.2, color:"highlight", title:"全球升溫 1.2°C 已發生", bullets:["極端高溫創新高","沿海洪患頻率提升","熱相關健康風險上升"], sources:[{label:"IPCC AR6 SYR SPM", url:"https://www.ipcc.ch/report/ar6/syr/"}]},
    { level:1.5, color:"red", title:"全球升溫 1.5°C 預計影響", bullets:["極端熱浪顯著增加","生態失衡與珊瑚高風險","沿海鹽化與基礎設施壓力"], sources:[{label:"IPCC SR1.5", url:"https://www.ipcc.ch/sr15/"}]},
    { level:1.8, color:"neutral", title:"全球升溫 1.8°C 預計影響", bullets:["乾旱與強降雨更劇","糧水安全壓力增加","生態服務退化"], sources:[{label:"IPCC AR6 SYR SPM", url:"https://www.ipcc.ch/report/ar6/syr/"}]},
    { level:2.0, color:"neutral", title:"全球升溫 2.0°C 預計影響", bullets:["極端事件更強","珊瑚喪失範圍更大","經濟損失與脆弱族群風險上升"], sources:[{label:"IPCC SR1.5", url:"https://www.ipcc.ch/sr15/"}]},
  ] as const;
  const [active,setActive]=useState<number|null>(null);
  const [hovered,setHovered]=useState<number|null>(null);
  const rootRef = useRef<HTMLDivElement|null>(null);

  useEffect(()=>{
    const onClick=(e:MouseEvent)=>{ if(!rootRef.current) return; if(!rootRef.current.contains(e.target as Node)) { setActive(null); setHovered(null); } };
    const onEsc=(e:KeyboardEvent)=>{ if(e.key==="Escape") setActive(null); };
    window.addEventListener("click",onClick); window.addEventListener("keydown",onEsc);
    return ()=>{ window.removeEventListener("click",onClick); window.removeEventListener("keydown",onEsc); };
  },[]);

  const pct=Math.max(0,Math.min(100,(1.0*currentWarmingC/maxScale)*100));
  const toPctNum=(v:number)=>Math.max(0,Math.min(100,(v/maxScale)*100));
  const fillDur=prefersReducedMotion?0:3.2; const sheenDur=prefersReducedMotion?0:3.6;

  return (
    <section id="warming" className="py-16" aria-roledescription="timeline" aria-label="全球升溫">
      <div className="w-full mx-auto px-4 sm:px-6 lg:px-8">
        <h2 className="text-2xl font-bold text-center">自基準年至今的全球升溫</h2>
        <p className="mt-1 text-center text-xs text-zinc-500 dark:text-zinc-400">基準: {baselineLabel} 平均 (0°C)</p>
        <div ref={rootRef} className="relative mt-8 mb-10">
          <div className="h-3 rounded-full bg-zinc-200 dark:bg-zinc-800 overflow-visible shadow-inner" />
          <motion.div className="absolute left-0 top-0 h-3 rounded-l-full origin-left bg-gradient-to-r from-sky-500 via-amber-500 to-rose-600"
            initial={{width:"0%"}} whileInView={{width:`${pct}%`}} viewport={{once:true,amount:0.4}} transition={{duration:fillDur,ease:[0.22,1,0.36,1]}}>
            <motion.div className="absolute top-0 bottom-0 left-0 w-20 bg-white/50 blur-sm mix-blend-overlay"
              initial={{x:"-20%",opacity:0}} whileInView={{x:"110%",opacity:prefersReducedMotion?0:[0,1,0]}} viewport={{once:true,amount:0.5}} transition={{duration:sheenDur,ease:"easeInOut"}}/>
          </motion.div>

          <div className="absolute -top-6 -translate-x-1/2 text-center" style={{left:`${pct}%`}}>
            <div className="w-0.5 h-8 bg-zinc-500 mx-auto" />
            <div className="mt-1 text-xs font-medium">現在 / Now ~ {currentWarmingC.toFixed(1)}°C</div>
          </div>

          {milestones.map((m,idx)=>{
            const open = active===idx || hovered===idx;
            const pctNum = toPctNum(m.level);
            const align = popoverAlignClass(pctNum);
            const colorBar = m.color === "red" ? "bg-rose-500" : m.color === "highlight" ? "bg-amber-500" : "bg-zinc-500";
            const pill = m.color === "red" ? "bg-rose-600/15 text-rose-600 border-rose-500/40" : m.color === "highlight" ? "bg-amber-500/15 text-amber-700 border-amber-500/40" : "bg-white/70 dark:bg-zinc-900/70 text-zinc-700 dark:text-zinc-200 border-zinc-300 dark:border-zinc-700";
            return (
              <div key={m.level} className="absolute top-1/2 -translate-y-1/2 -translate-x-1/2" style={{left:`${pctNum}%`}}>
                <div className="group relative z-20 flex flex-col items-center cursor-pointer" role="button" tabIndex={0} aria-expanded={open}
                  onKeyDown={(e)=>onKeyToggle(e,()=>setActive(v=>v===idx?null:idx))}
                  onClick={(e)=>{e.stopPropagation(); setActive(v=>v===idx?null:idx);}}
                  onMouseEnter={()=>setHovered(idx)} onMouseLeave={()=>setHovered(v=>v===idx?null:v)}>
                  <span className={`mb-1 text-[11px] px-1.5 py-0.5 rounded-md border ${pill}`}>{m.level}°C</span>
                  <div className={`${colorBar} w-0.5 h-8 group-hover:scale-105 transition-transform`} />
                  <div className={`absolute bottom-full mb-3 w-[280px] sm:w-[340px] z-30 transition-opacity duration-200 ${align} ${open?"opacity-100":"opacity-0 pointer-events-none"}`}>
                    <div className="relative rounded-2xl border border-zinc-200 dark:border-zinc-800 bg-white dark:bg-zinc-900 shadow-xl p-4">
                      <div className="absolute left-1/2 translate-x-[-50%] -bottom-2 w-3 h-3 rotate-45 bg-white dark:bg-zinc-900 border-b border-r border-zinc-200 dark:border-zinc-800" />
                      <div className="text-sm font-semibold">{m.title}</div>
                      <ul className="mt-2 list-disc pl-5 text-xs text-zinc-600 dark:text-zinc-300 space-y-1">{m.bullets.map((b,i)=>(<li key={i}>{b}</li>))}</ul>
                      <div className="mt-3 text-[11px] text-zinc-500 dark:text-zinc-400">來源: {m.sources?.map((s:any,i:number)=>(<a key={i} className="underline" href={s.url} target="_blank" rel="noreferrer">{s.label}{i < (m.sources?.length||0)-1 ? '；' : ''}</a>))}</div>
                    </div>
                  </div>
                </div>
              </div>
            );
          })}
        </div>
      </div>
    </section>
  );
}

// ----------------------
// Page
// ----------------------
export default function Site() {
  const { isDark, setIsDark, mounted } = useDarkMode();
  const orgJsonLd = { "@context":"https://schema.org", "@type":"EducationalOrganization", name:CONFIG.brand.name, url:"https://example.com", description:CONFIG.brand.tagline };
  return (
    <div className={isDark ? 'dark' : ''}>
      <div className="min-h-screen bg-white text-zinc-900 dark:bg-zinc-950 dark:text-zinc-50">
        <script type="application/ld+json" dangerouslySetInnerHTML={{ __html: JSON.stringify(orgJsonLd) }} />
        <Nav items={CONFIG.nav} />
        <main>
          <Hero />
          <section id="announcements" className="py-14 bg-zinc-50 dark:bg-zinc-900/50">
            <div className="w-full mx-auto px-4 sm:px-6 lg:px-8">
              <h2 className="text-2xl font-bold text-center">最新公告</h2>
              <div className="mt-8 grid md:grid-cols-5 gap-8 items-start">
                <div className="md:col-span-2 flex justify-center"><Globe2D /></div>
                <div className="md:col-span-3 space-y-4">
                  {CONFIG.announcements.map((a,i)=> (
                    <div key={i} className="rounded-2xl border border-zinc-200 dark:border-zinc-800 p-5 bg-white dark:bg-zinc-900">
                      <div className="flex items-center justify-between"><div className="font-semibold">{a.title}</div><div className="text-xs text-zinc-500">{a.date}</div></div>
                      <p className="mt-2 text-sm text-zinc-600 dark:text-zinc-300">{a.content}</p>
                    </div>
                  ))}
                </div>
              </div>
            </div>
          </section>
          <section id="highlights" className="py-14">
            <div className="w-full mx-auto px-4 sm:px-6 lg:px-8">
              <h2 className="text-2xl font-bold text-center">台灣綠領人才現況</h2>
              <div className="mt-8 grid sm:grid-cols-2 lg:grid-cols-3 gap-6">
                {CONFIG.highlights.map((h,i)=>(
                  <div key={i} className="rounded-2xl border border-zinc-200 dark:border-zinc-800 p-6 text-center">
                    <div className="text-3xl font-bold bg-clip-text text-transparent bg-gradient-to-r from-green-600 to-blue-500">{h.value}</div>
                    <div className="mt-2 font-medium">{h.title}</div>
                    <div className="mt-1 text-sm text-zinc-600 dark:text-zinc-300">{h.desc}</div>
                  </div>
                ))}
              </div>
            </div>
          </section>
          <HorizontalTimeline />
          <WarmingAxis />
          {/* 資料來源與更新說明 */}
          <section aria-label="warming-meta" className="mt-[-8px] mb-10">
            <div className="max-w-3xl mx-auto px-4 sm:px-6 lg:px-8">
              <div className="rounded-xl border border-zinc-200 dark:border-zinc-800 bg-white dark:bg-zinc-900 p-4 text-sm">
                <div className="font-medium">資料來源 & 更新說明</div>
                <div className="mt-1 text-zinc-600 dark:text-zinc-300">
                  里程碑內容與升溫估計參考：
                  {(CONFIG.warmingMeta?.sources||[]).map((s: any,i: number)=> (
                    <span key={s.url}>
                      {" "}<a href={s.url} target="_blank" rel="noreferrer" className="underline">{s.label}</a>{i<(CONFIG.warmingMeta?.sources||[]).length-1?"；":"。"}
                    </span>
                  ))}
                </div>
                {CONFIG.warmingMeta && (
                  <>
                    <div className="mt-1 text-zinc-600 dark:text-zinc-300">更新頻率：{CONFIG.warmingMeta.cadence}（最後更新：{CONFIG.warmingMeta.lastUpdated}）。</div>
                    <div className="mt-1 text-zinc-500 dark:text-zinc-400 text-xs">{CONFIG.warmingMeta.note}</div>
                  </>
                )}
              </div>
            </div>
          </section>
          <section id="solutions" className="py-14 bg-zinc-50 dark:bg-zinc-900/50">
            <div className="w-full mx-auto px-4 sm:px-6 lg:px-8">
              <h2 className="text-2xl font-bold text-center">我們的服務</h2>
              <div className="mt-10 grid sm:grid-cols-2 lg:grid-cols-3 gap-6">
                {[
                  {title:'永續教育訓練', desc:'課程涵蓋報告書、碳盤查、策略與內稽', badge:'課程', Icon: BookOpen, grad:'from-emerald-500 to-green-600'},
                  {title:'整合性 ESG 解決方案', desc:'盤查→揭露→驗證，一站式接軌國際', badge:'方案', Icon: Layers, grad:'from-sky-500 to-blue-600'},
                  {title:'綠領人才媒合', desc:'培訓＋證照＋就業媒合，縮短缺口', badge:'媒合', Icon: Users, grad:'from-amber-500 to-orange-600'}
                ].map((s,i)=> (
                  <div key={i} className="rounded-2xl border border-zinc-200 dark:border-zinc-800 p-6 bg-white dark:bg-zinc-900">
                    {/* 標識區：圓形圖示 + 類別小標 */}
                    <div className="flex items-center gap-3">
                      <div className={`w-11 h-11 rounded-xl bg-gradient-to-br ${s.grad} text-white grid place-items-center shadow`}>
                        <s.Icon className="w-5 h-5"/>
                      </div>
                      <span className="inline-flex items-center px-2.5 py-0.5 rounded-full text-[11px] font-medium border border-zinc-200 dark:border-zinc-700 text-zinc-700 dark:text-zinc-200 bg-zinc-50 dark:bg-zinc-800/60">{s.badge}</span>
                    </div>
                    <div className="mt-3 font-semibold text-base">{s.title}</div>
                    <div className="mt-1 text-sm text-zinc-600 dark:text-zinc-300 leading-relaxed">{s.desc}</div>
                  </div>
                ))}
              </div>
            </div>
          </section>
          <section id="faq" className="py-14">
            <div className="w-full mx-auto px-4 sm:px-6 lg:px-8">
              <h2 className="text-2xl font-bold text-center">常見問題</h2>
              <div className="mt-6 divide-y divide-zinc-200 dark:divide-zinc-800 rounded-2xl border border-zinc-200 dark:border-zinc-800 overflow-hidden">
                {CONFIG.faq.map((f,i)=>(
                  <details key={i} className="group p-6">
                    <summary className="flex items-center justify-between cursor-pointer list-none"><span className="font-medium">{f.q}</span><span className="transition group-open:rotate-45">+</span></summary>
                    <p className="mt-3 text-zinc-600 dark:text-zinc-300">{f.a}</p>
                  </details>
                ))}
              </div>
            </div>
          </section>
          <section id="contact" className="py-14">
            <div className="w-full mx-auto px-4 sm:px-6 lg:px-8">
              <h2 className="text-2xl font-bold text-center">聯絡我們</h2>
              <div className="mt-6 grid md:grid-cols-5 gap-6 items-start">
                <div className="md:col-span-3">
                  <form onSubmit={(e)=>{e.preventDefault(); alert('已送出');}} className="rounded-2xl border border-zinc-200 dark:border-zinc-800 p-6">
                    <div className="grid sm:grid-cols-2 gap-4">
                      <input className="w-full rounded-xl border border-zinc-300 dark:border-zinc-700 bg-white dark:bg-zinc-900 p-3" placeholder="姓名" required/>
                      <input type="email" className="w-full rounded-xl border border-zinc-300 dark:border-zinc-700 bg-white dark:bg-zinc-900 p-3" placeholder="Email" required/>
                    </div>
                    <input className="w-full mt-4 rounded-xl border border-zinc-300 dark:border-zinc-700 bg-white dark:bg-zinc-900 p-3" placeholder="公司 / 單位"/>
                    <textarea rows={5} className="w-full mt-4 rounded-xl border border-zinc-300 dark:border-zinc-700 bg-white dark:bg-zinc-900 p-3" placeholder="訊息內容"/>
                    <button className="mt-4 px-4 py-2 rounded-2xl bg-gradient-to-r from-green-600 to-blue-500 text-white">送出</button>
                  </form>
                </div>
                <div className="md:col-span-2 space-y-3 text-sm">
                  <div className="flex items-center gap-2">📧 {CONFIG.contact.email}</div>
                  <div className="flex items-center gap-2">📞 {CONFIG.contact.phone}</div>
                  <div className="flex items-center gap-2">📍 {CONFIG.contact.address}</div>
                </div>
              </div>
            </div>
          </section>
        </main>
        <footer className="py-10 border-t border-zinc-200 dark:border-zinc-800 text-center text-sm text-zinc-600 dark:text-zinc-400">
          © {new Date().getFullYear()} {CONFIG.brand.name}. All rights reserved.
        </footer>
      </div>
    </div>
  );
}

// ----------------------
// Smoke tests
// ----------------------
try {
  console.assert(typeof Nav === 'function', '[Smoke] Nav defined');
  console.assert(typeof WarmingAxis === 'function', '[Smoke] WarmingAxis defined');
  console.assert(!!CONFIG.warmingMeta && Array.isArray(CONFIG.warmingMeta.sources) && CONFIG.warmingMeta.sources.length>0, '[Smoke] warmingMeta.sources present');
} catch {}
