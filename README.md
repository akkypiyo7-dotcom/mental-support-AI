# mental-support-AI
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>お黴ちゃん AI（無料デモ）</title>
  <style>
    * { box-sizing: border-box; }
    body { margin: 0; font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Hiragino Kaku Gothic ProN", Meiryo, sans-serif; background: #f7f7fb; color: #222; }
    header { position: sticky; top: 0; backdrop-filter: blur(6px); background: rgba(255,255,255,0.8); border-bottom: 1px solid #eee; padding: 12px 16px; display:flex; align-items:center; gap:12px; }
    .avatar { width: 40px; height: 40px; border-radius: 50%; background: #fff; display:grid; place-items:center; border:1px solid #ddd; }
    .avatar svg { width: 22px; height: 22px; }
    .title { font-weight: 700; }

    main { max-width: 900px; margin: 0 auto; padding: 16px; }
    .card { background: #fff; border: 1px solid #eee; border-radius: 16px; box-shadow: 0 6px 20px rgba(0,0,0,0.06); overflow: hidden; }
    .hero { display:grid; grid-template-columns: 320px 1fr; gap: 16px; padding: 16px; }
    @media (max-width: 820px){ .hero { grid-template-columns:1fr; } }

    /* キャラクター領域 */
    .stage { position: relative; background: linear-gradient(#fefefe, #f2f5ff); border:1px solid #eee; border-radius: 16px; height: 320px; display:grid; place-items:center; overflow:hidden; }
    .char { position: relative; width: 160px; height: 160px; }
    .body { position:absolute; inset: 0; background:#ffe; border:2px solid #222; border-radius: 50% 50% 45% 45%/55% 55% 45% 45%; }
    .eye { position:absolute; top: 60px; width: 14px; height: 14px; background:#222; border-radius:50%; }
    .eye.left { left: 52px; }
    .eye.right{ right: 52px; }
    .arm { position:absolute; width: 80px; height: 14px; background:#222; border-radius: 7px; left: 120px; top: 110px; transform-origin: 0 50%; }
    .arm.wave { animation: wave 1.6s ease-in-out infinite; }
    @keyframes wave { 0%{ transform: rotate(0deg);} 25%{ transform: rotate(25deg);} 50%{ transform: rotate(0deg);} 75%{ transform: rotate(25deg);} 100%{ transform: rotate(0deg);} }

    .blanket { position:absolute; bottom:-140px; left:50%; transform:translateX(-50%); width: 220px; height: 140px; background:#cfe3ff; border:2px solid #222; border-radius: 18px; transition: bottom 0.8s ease; }
    .blanket.on { bottom: 10px; }

    .controls { display:flex; gap:8px; flex-wrap:wrap; }
    .controls button, .controls select { padding:10px 14px; border-radius: 12px; border:1px solid #ddd; background:#fff; cursor:pointer; }
    .controls button:hover { background:#f6f6ff; }

    /* チャット */
    .chat { border-top: 1px dashed #eee; padding: 12px 16px; display:flex; flex-direction:column; gap:12px; }
    .row { display:flex; gap:8px; align-items:flex-end; }
    textarea { flex:1; min-height:48px; padding:12px; border-radius:12px; border:1px solid #ddd; resize:vertical; }
    .send { padding: 12px 16px; border-radius: 12px; border: none; background: #222; color: #fff; cursor: pointer; }

    .log { max-height: 360px; overflow:auto; display:flex; flex-direction:column; gap:10px; padding: 0 4px; }
    .msg { display:flex; gap:10px; }
    .bubble { padding:10px 12px; border-radius: 14px; max-width: 70%; }
    .user .bubble { background:#eaf7ff; margin-left:auto; }
    .ai .bubble { background:#f4f4f7; }

    .notice { font-size: 12px; color:#666; }
    .badge { font-size: 12px; padding:2px 8px; border-radius:999px; border:1px solid #ddd; }
  </style>
</head>
<body>
  <header>
    <div class="avatar" title="お黴ちゃん">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="9"/><path d="M9 10h.01M15 10h.01"/><path d="M8 15c1.333 1 2.667 1 4 0"/></svg>
    </div>
    <div class="title">お黴ちゃん AI（無料・ブラウザ完結）</div>
    <div style="margin-left:auto" class="badge" id="engine-badge">Engine: ルールベース</div>
  </header>

  <main>
    <div class="card">
      <div class="hero">
        <!-- キャラクター舞台 -->
        <div class="stage">
          <div class="char" id="char">
            <div class="body"></div>
            <div class="eye left"></div>
            <div class="eye right"></div>
            <div class="arm" id="arm"></div>
            <div class="blanket" id="blanket"></div>
          </div>
        </div>
        <!-- コントロール＆説明 -->
        <div style="display:flex; flex-direction:column; gap:12px;">
          <div class="controls">
            <button id="btn-wave">手を振る</button>
            <button id="btn-blanket">布団をかぶる/外す</button>
            <select id="ai-engine">
              <option value="rules">AI: ルールベース（無料）</option>
              <option value="webllm">AI: WebLLM（端末内, 無料）</option>
            </select>
          </div>
          <div class="notice">
            ■ 完全無料で動く方針：<br>
            ・チャットは <b>ルールベース</b> を既定で使用（サンプル応答）<br>
            ・端末が対応していれば <b>WebLLM</b> を選択すると、ブラウザ内で軽量LLMが動きます（サーバ不要/課金不要）<br>
            ・あとでAPI課金を許容できるようになったら、OpenAI等のAPI呼び出しを追加実装できます
          </div>
          <div class="log" id="log"></div>
          <div class="chat">
            <div class="row">
              <textarea id="inp" placeholder="お話してね（無料デモ）"></textarea>
              <button class="send" id="send">送信</button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <p class="notice" style="margin-top:12px">
      使い方：この1ファイルを <b>index.html</b> として保存 → GitHub Pages にアップロード → そのまま公開できます。<br>
      画像（あなたの手描きキャラ）を使いたい場合は、.body や .eye などのCSSを画像に置き換えてください（背景透過PNG推奨）。
    </p>
  </main>

  <!-- WebLLM（必要な時だけ動的ロード） -->
  <script>
    const state = { engine: 'rules', webllm: null, session: null };

    const logEl = document.getElementById('log');
    function pushMsg(role, text){
      const wrap = document.createElement('div');
      wrap.className = 'msg ' + (role === 'user' ? 'user' : 'ai');
      const bubble = document.createElement('div');
      bubble.className = 'bubble';
      bubble.textContent = text;
      wrap.appendChild(bubble);
      logEl.appendChild(wrap);
      logEl.scrollTop = logEl.scrollHeight;
    }

    // ルールベース簡易応答（無料デモ用）
    function ruleReply(input){
      const s = input.trim();
      if(!s) return "";
      const lowers = s.toLowerCase();
      if(lowers.includes('hello') || s.includes('こんにちは')) return 'こんにちは！お黴ちゃんだよ。今日は何を話す？';
      if(s.includes('つかれ')||s.includes('疲れ')) return 'おつかれさま…深呼吸いっかいしよ。3秒吸って、4秒止めて、6秒はいてみよう。';
      if(s.includes('子')||s.includes('育児')) return '育児は毎日が実験だよね。一緒に小さい成功を探そう。';
      if(s.includes('不安')||s.includes('こわ')) return '不安を言葉にできたの、もう半分進んでる合図だよ。';
      return 'うんうん、聞いてるよ。もう少し詳しく教えて？';
    }

    // WebLLM を必要時にロード
    async function ensureWebLLM(){
      if(state.webllm) return state.webllm;
      const script = document.createElement('script');
      script.src = 'https://unpkg.com/@mlc-ai/web-llm/dist/index.min.js';
      await new Promise(r => { script.onload = r; document.head.appendChild(script); });
      state.webllm = window.webllm;
      return state.webllm;
    }

    async function initWebLLMSession(){
      const webllm = await ensureWebLLM();
      if(state.session) return state.session;
      const engine = new webllm.MLCEngine();
      // 軽量モデルを選択（端末性能により初回ロードに時間がかかることがあります）
      await engine.reload({ model: 'Llama-3-8B-Instruct-q4f32_MLC' }).catch(()=>{});
      state.session = engine;
      return state.session;
    }

    async function webllmReply(input){
      const engine = await initWebLLMSession();
      try{
        const out = await engine.chat.completions.create({
          messages: [{role:'system', content:'You are a kind, supportive assistant named Okabi-chan.'},{role:'user', content: input}],
          temperature: 0.6,
          stream: false,
        });
        const txt = out?.choices?.[0]?.message?.content || '…（応答なし）';
        return txt;
      }catch(e){
        return '（WebLLMの初期化に失敗しました。ルールベース応答に切り替えます）\n' + ruleReply(input);
      }
    }

    // UI 配線
    document.getElementById('send').addEventListener('click', async ()=>{
      const inp = document.getElementById('inp');
      const text = inp.value;
      if(!text.trim()) return;
      pushMsg('user', text);
      inp.value='';
      let reply = '';
      if(state.engine === 'rules') reply = ruleReply(text);
      else reply = await webllmReply(text);
      pushMsg('ai', reply);
    });

    document.getElementById('ai-engine').addEventListener('change', async (e)=>{
      state.engine = e.target.value === 'webllm' ? 'webllm' : 'rules';
      document.getElementById('engine-badge').textContent = 'Engine: ' + (state.engine==='webllm' ? 'WebLLM（端末内）' : 'ルールベース');
      if(state.engine==='webllm'){
        pushMsg('ai','（WebLLMを初期化中…初回は数十秒かかることがあります）');
        await initWebLLMSession();
        pushMsg('ai','準備できたよ！話しかけてね');
      }
    });

    // アニメ用ボタン
    const arm = document.getElementById('arm');
    document.getElementById('btn-wave').addEventListener('click', ()=>{
      arm.classList.toggle('wave');
      setTimeout(()=> arm.classList.remove('wave'), 2600);
    });
    const blanket = document.getElementById('blanket');
    document.getElementById('btn-blanket').addEventListener('click', ()=>{
      blanket.classList.toggle('on');
    });
  </script>
</body>
</html>
