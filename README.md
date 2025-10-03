<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Smart Exam Portal</title>
  <style>
    body {
      margin: 0;
      font-family: "Segoe UI", Arial, sans-serif;
      background: linear-gradient(135deg, #6a11cb, #2575fc);
      color: #fff;
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 30px;
    }
    .card {
      display: flex;
      gap: 20px;
      max-width: 1100px;
      width: 100%;
    }
    .quiz, .webcam {
      border-radius: 18px;
      box-shadow: 0 10px 35px rgba(0,0,0,0.25);
      padding: 20px;
      background: rgba(255,255,255,0.1);
      backdrop-filter: blur(16px);
      flex: 1;
    }
    .quiz {
      flex: 2;
    }
    h1 {
      margin-top: 0;
      font-size: 1.5rem;
      text-align: center;
      color: #fff;
    }
    #question {
      font-size: 1.2rem;
      margin: 15px 0;
      font-weight: 600;
    }
    .option {
      display: block;
      padding: 12px 16px;
      margin: 8px 0;
      border-radius: 12px;
      background: linear-gradient(135deg, #4facfe, #00f2fe);
      color: #fff;
      border: none;
      cursor: pointer;
      font-size: 1rem;
      transition: transform 0.2s, background 0.3s;
    }
    .option:hover {
      transform: scale(1.03);
      background: linear-gradient(135deg, #43e97b, #38f9d7);
    }
    .option.selected {
      border: 3px solid #fff;
      transform: scale(1.05);
    }
    button.primary {
      margin-top: 12px;
      width: 100%;
      padding: 12px;
      border-radius: 12px;
      border: none;
      font-size: 1.1rem;
      font-weight: bold;
      cursor: pointer;
      background: linear-gradient(135deg, #ff416c, #ff4b2b);
      color: white;
      transition: transform 0.2s, background 0.3s;
    }
    button.primary:hover {
      transform: scale(1.05);
      background: linear-gradient(135deg, #f7971e, #ffd200);
    }
    #result {
      margin-top: 15px;
      font-weight: bold;
      font-size: 1.1rem;
      text-align: center;
    }
    #timer {
      height: 12px;
      border-radius: 6px;
      background: rgba(255,255,255,0.2);
      overflow: hidden;
      margin-top: 10px;
    }
    #timerFill {
      height: 100%;
      background: linear-gradient(135deg, #f12711, #f5af19);
      width: 100%;
      transition: width 1s linear;
    }
    video {
      width: 100%;
      border-radius: 12px;
      background: #000;
      margin-top: 8px;
    }
    .warn {
      margin-top: 12px;
      font-weight: bold;
      font-size: 0.95rem;
      text-align: center;
    }
    .warncount {
      margin-top: 4px;
      text-align: center;
      font-size: 0.9rem;
    }
    .final {
      text-align: center;
      font-size: 1.5rem;
      color: #fff;
    }
  </style>
</head>
<body>
  <div class="card">
    <div class="quiz" id="quizBox">
      <h1>Smart Exam Portal</h1>
      <div id="question">Loading question…</div>
      <div id="options"></div>
      <div id="timer"><div id="timerFill"></div></div>
      <button class="primary" id="submitBtn">Submit / Next</button>
      <div id="result"></div>
    </div>

    <div class="webcam">
      <h1>Webcam Monitor</h1>
      <video id="video" autoplay muted playsinline></video>
      <div class="warn" id="warnText">Monitoring movement...</div>
      <div class="warncount" id="warnCount">Warnings: 0 / 5</div>
    </div>
  </div>

  <script>
  /********************** QUIZ **********************/
  const QUESTIONS = [
    {q:"What is the capital of India?", options:["Delhi","Mumbai","Kolkata","Chennai"], a:0},
    {q:"What is the national animal of India?", options:["Lion","Tiger","Elephant","Peacock"], a:1},
    {q:"Which is the largest planet?", options:["Earth","Jupiter","Mars","Venus"], a:1},
    {q:"HTML stands for?", options:["Hyper Trainer Marking Language","Hyper Text Markup Language","Hyper Text Making Language","High Text Machine Language"], a:1},
    {q:"CSS is used for?", options:["Structure","Data Storage","Styling","Programming"], a:2},
    {q:"Who is the Father of Computers?", options:["Albert Einstein","Charles Babbage","Isaac Newton","Alan Turing"], a:1},
    {q:"Which gas do plants absorb?", options:["Oxygen","Carbon Dioxide","Nitrogen","Hydrogen"], a:1},
    {q:"Shortcut for Copy?", options:["Ctrl + V","Ctrl + C","Ctrl + X","Ctrl + Z"], a:1},
    {q:"Who wrote the Indian National Anthem?", options:["Rabindranath Tagore","Mahatma Gandhi","Jawaharlal Nehru","Sarojini Naidu"], a:0},
    {q:"5 + 7 × 2 = ?", options:["19","24","26","17"], a:0}
  ];
  let idx=0, score=0, selected=null;
  let timerInterval, timeLeft=15;

  const questionEl=document.getElementById("question");
  const optionsEl=document.getElementById("options");
  const submitBtn=document.getElementById("submitBtn");
  const resultEl=document.getElementById("result");
  const timerFill=document.getElementById("timerFill");

  function renderQuestion(){
    selected=null;
    const q=QUESTIONS[idx];
    questionEl.textContent=`${idx+1}. ${q.q}`;
    optionsEl.innerHTML="";
    q.options.forEach((opt,i)=>{
      const b=document.createElement("button");
      b.className="option";
      b.innerText=String.fromCharCode(65+i)+". "+opt;
      b.onclick=()=>{selected=i;highlight(i)};
      optionsEl.appendChild(b);
    });
    resultEl.textContent="";
    startTimer();
  }
  function highlight(i){
    [...optionsEl.children].forEach((c,ci)=>c.classList.toggle("selected",ci===i));
  }
  function startTimer(){
    clearInterval(timerInterval);
    timeLeft=15;
    timerFill.style.width="100%";
    timerInterval=setInterval(()=>{
      timeLeft--;
      timerFill.style.width=(timeLeft/15*100)+"%";
      if(timeLeft<=0){clearInterval(timerInterval);autoSubmit();}
    },1000);
  }
  function autoSubmit(){
    if(selected===null){ idx++; if(idx<QUESTIONS.length) renderQuestion(); else showFinal(); }
    else submitBtn.click();
  }
  submitBtn.onclick=()=>{
    clearInterval(timerInterval);
    if(selected===null){ alert("Please select an option."); startTimer(); return; }
    if(selected===QUESTIONS[idx].a){ score++; resultEl.textContent=`✅ Correct! Score: ${score}/${idx+1}`; }
    else resultEl.textContent=`❌ Wrong! Score: ${score}/${idx+1}`;
    idx++;
    if(idx<QUESTIONS.length){ setTimeout(renderQuestion,1500); } else { setTimeout(showFinal,1500); }
  };
  function showFinal(){
    document.getElementById("quizBox").innerHTML=`<div class="final">🎉 Quiz Completed!<br>Final Score: ${score}/${QUESTIONS.length}</div>`;
  }
  renderQuestion();

  /******************** WEBCAM ********************/
  const video=document.getElementById("video");
  const warnText=document.getElementById("warnText");
  const warnCount=document.getElementById("warnCount");
  const canvas=document.createElement("canvas");
  const ctx=canvas.getContext("2d");
  let prevImage=null, warnings=0, lastWarn=0;
  const LIMIT=5, COOL=1500, STEP=8, TH=18;

  async function startCamera(){
    try{
      const stream=await navigator.mediaDevices.getUserMedia({video:{width:320,height:240}});
      video.srcObject=stream; await video.play();
      canvas.width=320; canvas.height=240;
      setTimeout(processFrame,800);
    }catch(e){warnText.textContent="⚠ Webcam not available";}
  }
  function addWarn(msg){
    if(Date.now()-lastWarn<COOL) return;
    lastWarn=Date.now(); warnings++;
    warnText.textContent=msg; warnCount.textContent=`Warnings: ${warnings}/${LIMIT}`;
    if(warnings>=LIMIT){
      document.body.innerHTML=`<h1 style="text-align:center;color:red;">❌ Exam Closed due to Malpractice</h1><p style="text-align:center;font-size:1.2rem;">Final Score: ${score}/${QUESTIONS.length}</p>`;
      video.srcObject.getTracks().forEach(t=>t.stop());
    }
  }
  function processFrame(){
    if(video.paused||video.ended){ setTimeout(processFrame,500); return; }
    ctx.drawImage(video,0,0,canvas.width,canvas.height);
    const curr=ctx.getImageData(0,0,canvas.width,canvas.height);
    if(prevImage){
      let total=0,count=0;
      for(let y=0;y<canvas.height;y+=2){
        for(let x=0;x<canvas.width;x+=STEP){
          const i=(y*canvas.width+x)*4;
          const lum=0.299*curr.data[i]+0.587*curr.data[i+1]+0.114*curr.data[i+2];
          const pl=0.299*prevImage.data[i]+0.587*prevImage.data[i+1]+0.114*prevImage.data[i+2];
          total+=Math.abs(lum-pl); count++;
        }
      }
      const avg=total/count;
      if(avg>TH) addWarn("⚠ Suspicious movement detected!");
      else if(warnings<LIMIT) warnText.textContent="✅ No suspicious movement.";
    }
    prevImage=ctx.getImageData(0,0,canvas.width,canvas.height);
    setTimeout(processFrame,1000);
  }
  startCamera();
  </script>
</body>
</html>
