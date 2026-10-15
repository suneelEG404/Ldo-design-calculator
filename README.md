<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Output-capacitorless LDO with spike detection: design calculator</title>
<style>
:root{--bg:#eef2f4;--panel:#fff;--ink:#14232e;--mute:#58697a;--line:#cdd7de;--acc:#0b6e79;--hl:#e3f1f2;--warn:#a5481a}
@media (prefers-color-scheme:dark){:root{--bg:#10191f;--panel:#17232b;--ink:#e6edf1;--mute:#93a5b3;--line:#2b3b46;--acc:#58c2cc;--hl:#17323a;--warn:#f0985f}}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--ink);font:16px/1.5 Georgia,"Times New Roman",serif}
main{max-width:860px;margin:0 auto;padding:24px 16px 48px}
h1{font-size:1.6rem;line-height:1.2;margin:0 0 6px}
h2{font-size:1.15rem;margin:0 0 10px;color:var(--acc)}
p.sub{margin:0 0 20px;color:var(--mute);max-width:62ch}
section{background:var(--panel);border:1px solid var(--line);border-radius:6px;padding:16px;margin-bottom:16px}
.grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(190px,1fr));gap:10px}
label{display:flex;flex-direction:column;font:13px/1.3 system-ui,sans-serif;color:var(--mute);gap:3px}
input{font:15px system-ui,sans-serif;padding:6px 8px;border:1px solid var(--line);border-radius:4px;background:var(--bg);color:var(--ink);width:100%}
input:focus-visible{outline:2px solid var(--acc);outline-offset:1px}
table{width:100%;border-collapse:collapse;font:14px/1.4 system-ui,sans-serif}
td{padding:7px 4px;border-bottom:1px solid var(--line);vertical-align:top}
td:first-child{width:56%}
td:last-child{text-align:right;font-weight:600;font-variant-numeric:tabular-nums;white-space:nowrap}
tr.key td:last-child{background:var(--hl);color:var(--acc)}
.f{font:12.5px system-ui,sans-serif;color:var(--mute);display:block}
.note{font:13px/1.45 system-ui,sans-serif;color:var(--mute);margin:10px 0 0}
.warn{color:var(--warn)}
.paper{font:12px system-ui,sans-serif;color:var(--mute)}
</style>
</head>
<body>
<main>
<h1>Output-capacitorless LDO with direct voltage-spike detection</h1>
<p class="sub">Hand-calculation worksheet for the topology in Or &amp; Leung, IEEE JSSC, vol. 45, no. 2, Feb. 2010. Defaults reproduce the paper's 1 V / 0.8 V / 66.7 mA case in AMS 0.35 µm. Change any value and every result updates. Only the MP size, CUP/CDN and RUP/RDN come from the paper; the other results are estimates to refine in simulation.</p>

<section>
<h2>Specification and process</h2>
<div class="grid" id="in-spec"></div>
</section>

<section>
<h2>1. Power transistor MP (triode at dropout)</h2>
<table id="t1"></table>
<p class="note">Paper's MP: 30000 µm / 0.35 µm (W/L = 85,700).</p>
</section>

<section>
<h2>2. Slew-rate limit and required boost</h2>
<div class="grid" id="in-sr"></div>
<table id="t2" style="margin-top:10px"></table>
</section>

<section>
<h2>3. Error-amplifier devices MC1 and MC3</h2>
<div class="grid" id="in-mc"></div>
<table id="t3" style="margin-top:10px"></table>
</section>

<section>
<h2>4. Spike-detection devices MUP1 / MDN1</h2>
<div class="grid" id="in-sp"></div>
<table id="t4" style="margin-top:10px"></table>
<p class="note">Square law overestimates in weak inversion. Use the result as an upper bound and tune the mirror ratios in simulation. MDN3 carries 2·IBIAS, so make MDN3:MDN2 = 2:1.</p>
</section>

<section>
<h2>5. High-pass network and bias generator</h2>
<div class="grid" id="in-hp"></div>
<table id="t5" style="margin-top:10px"></table>
<p class="note">Keep the corner below the loop bandwidth BWL (about 200 kHz to 1 MHz for a 100 mA LDO). Paper: 100 kHz corner, C = 3 pF, R = 530 kΩ, CB = 2 pF.</p>
</section>

<p class="paper">Process constants (µCox, Cox) are assumed typical values, not taken from the paper.</p>
</main>

<script>
const D={
 vin:1,vo:0.8,io:66.7,vthp:0.65,vthn:0.5,kp:58,kn:170,cox:4.5,cov:0.3,L:0.35,vg:0.18,
 ib:14,dvg:0.4,trec:1,
 mcwl:20,ibias:1,
 mnwl:28.6,dv:0.2,vovn:0.03,
 fc:100,cc:3,vsgb:0.6
};
const F={
 vin:["VIN (V)"],vo:["VO (V)"],io:["IO(max) (mA)"],vthp:["|Vthp| (V)"],vthn:["Vthn (V)"],
 kp:["µpCox (µA/V²)"],kn:["µnCox (µA/V²)"],cox:["Cox (fF/µm²)"],cov:["Overlap cap (fF/µm of W)"],L:["MP length L (µm)"],
 vg:["MP gate voltage at max load (V)"],
 ib:["Bias current into MP gate (µA)"],dvg:["Gate swing needed (V)"],trec:["Target slew time (µs)"],
 mcwl:["MC1/MC3 W/L"],ibias:["IBIAS (µA)"],
 mnwl:["MUP1/MDN1 W/L"],dv:["Output spike ΔV (V)"],vovn:["MUP1/MDN1 steady Vov (V)"],
 fc:["Corner fc (kHz)"],cc:["CUP = CDN (pF)"],vsgb:["MB source-gate voltage VSG,B (V)"]
};
const groups={"in-spec":["vin","vo","io","vthp","vthn","kp","kn","cox","cov","L"],
 "in-sr":["vg","ib","dvg","trec"],"in-mc":["mcwl","ibias"],"in-sp":["mnwl","dv","vovn"],"in-hp":["fc","cc","vsgb"]};
const V={...D};
for(const g in groups){
 const el=document.getElementById(g);
 groups[g].forEach(k=>{
  const l=document.createElement("label");
  l.textContent=F[k][0];
  const i=document.createElement("input");
  i.type="number";i.step="any";i.value=D[k];i.id="i-"+k;
  i.addEventListener("input",()=>{const n=parseFloat(i.value);if(!isNaN(n)){V[k]=n;calc();}});
  l.appendChild(i);el.appendChild(l);
 });
}
const fmt=(x,d=3)=>{if(!isFinite(x))return"n/a";const a=Math.abs(x);
 if(a>=1e4)return x.toLocaleString("en",{maximumFractionDigits:0});
 return Number(x.toPrecision(d)).toString();};
function rows(id,list){
 document.getElementById(id).innerHTML=list.map(r=>
  `<tr class="${r[3]?"key":""}"><td>${r[0]}<span class="f">${r[1]}</span></td><td class="${r[4]||""}">${r[2]}</td></tr>`).join("");
}
function calc(){
 const {vin,vo,io,vthp,kp,kn,cox,cov,L,vg,ib,dvg,trec,mcwl,ibias,mnwl,dv,vovn,fc,cc,vsgb}=V;
 // 1 power transistor
 const vsd=vin-vo, vsg=vin-vg, vov=vsg-vthp;
 const br=vov*vsd-vsd*vsd/2;
 const wl=(io*1e-3)/(kp*1e-6*br);
 const W=wl*L;
 const cg=W*L*cox*1e-15, cp=cg+W*cov*1e-15;
 const bad=(vov<=0||br<=0);
 rows("t1",[
  ["Dropout VSD","VIN − VO",fmt(vsd)+" V"],
  ["Source-gate voltage VSG","VIN − Vgate",fmt(vsg)+" V"],
  ["Overdrive Vov","VSG − |Vthp|",fmt(vov)+" V",0,bad?"warn":""],
  ["Triode bracket","Vov·VSD − VSD²/2",fmt(br*1e3)+" mV²·10⁻³"],
  ["Required W/L","IO / (µpCox · bracket)",bad?"invalid: no overdrive":fmt(wl,4),1],
  ["Width W","W/L × L",bad?"n/a":fmt(W,4)+" µm",1],
  ["Gate capacitance Cpar","W·L·Cox + W·Cov",bad?"n/a":fmt(cp*1e12,3)+" pF",1]
 ]);
 // 2 slew
 const sr=(ib*1e-6)/cp, t=cp*dvg/(ib*1e-6), ireq=cp*dvg/(trec*1e-6);
 rows("t2",[
  ["Slew rate with steady bias","I / Cpar",bad?"n/a":fmt(sr*1e-6)+" V/µs"],
  ["Time to swing the gate","Cpar·ΔVg / I",bad?"n/a":fmt(t*1e6)+" µs"],
  ["Minimum boost current","Cpar·ΔVg / target time",bad?"n/a":fmt(ireq*1e6)+" µA",1],
  ["Suggested peak boost","about 4× minimum",bad?"n/a":fmt(ireq*4e6)+" µA"]
 ]);
 // 3 MC
 const ibu=ibias*1e-6;
 const vovc=Math.sqrt(2*ibu/(kp*1e-6*mcwl));
 const vsgc=vthp+vovc;
 const gm=ibu/(1.3*0.0259);
 rows("t3",[
  ["Overdrive (square-law)","√(2I / (µpCox·W/L))",fmt(vovc*1e3)+" mV"],
  ["Source-gate voltage VSG,C","|Vthp| + Vov",fmt(vsgc)+" V"],
  ["VSET for VREF = VO","VO − VSG,C3",fmt(vo-vsgc)+" V",1],
  ["gm (weak-inversion estimate)","I / (n·VT), n = 1.3",fmt(gm*1e6)+" µS"]
 ]);
 // 4 spike devices
 const cgn=20*L*cox*1e-15; // assumes W = 20 µm gate load for MUP1/MDN1
 const ratio=(cc*1e-12)/(cc*1e-12+cgn);
 const di=kn*1e-6*mnwl*(vovn+dv*ratio/2)*dv*ratio;
 rows("t4",[
  ["Gate capacitance of MUP1/MDN1","assumes W = 20 µm",fmt(cgn*1e15)+" fF"],
  ["Coupling ratio","C / (C + Cgate)",fmt(ratio*100,3)+" %"],
  ["Coupled gate step","ΔV × ratio",fmt(dv*ratio*1e3)+" mV"],
  ["Extra current ΔI (paper Eq. 2)","µnCox·(W/L)·(Vov + ΔV/2)·ΔV",fmt(di*1e6)+" µA",1],
  ["Margin vs. minimum boost",bad?"":"ΔI / minimum boost",bad?"n/a":fmt(di/ireq)+"×",0,(!bad&&di<ireq)?"warn":""]
 ]);
 // 5 HP and bias gen
 const R=1/(2*Math.PI*fc*1e3*cc*1e-12);
 const rb=(vo-vsgb)/ibu;
 rows("t5",[
  ["RUP = RDN","1 / (2π·fc·C)",fmt(R/1e3,3)+" kΩ",1],
  ["Corner check","paper: 100 kHz, below BWL",fmt(fc)+" kHz"],
  ["RB in bias generator","(VO − VSG,B) / IBIAS",fmt(rb/1e3,3)+" kΩ",1],
  ["IBIAS at VO − 0.1 V (same RB)","tracks VO",fmt((vo-0.1-vsgb)/rb*1e6)+" µA"]
 ]);
}
calc();
</script>
</body>
</html>
