<!doctype html> <!-- HTML5 -->
<html lang="ko"> <!-- 한국어 -->

<head> <!-- head -->
  <meta charset="utf-8" /> <!-- UTF-8 -->
  <meta name="viewport" content="width=device-width, initial-scale=1" /> <!-- 모바일 -->
  <title>짐 체크</title> <!-- 제목 -->

  <style>
    /* CSS */
    :root {
      --bg: #000;
      --card: rgba(255, 255, 255, .08);
      --border: rgba(255, 255, 255, .18);
      --text: rgba(255, 255, 255, .92);
      --muted: rgba(255, 255, 255, .70);
    }

    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      min-height: 100vh;
      background: var(--bg);
      color: var(--text);
      font-family: system-ui, -apple-system, "Apple SD Gothic Neo", "Noto Sans KR", sans-serif;
      display: grid;
      place-items: center;
      padding: 16px;
    }

    .wrap {
      width: min(560px, 94vw);
      display: grid;
      gap: 12px;
    }

    .header {
      display: flex;
      align-items: baseline;
      justify-content: space-between;
      gap: 12px;
    }

    .title {
      font-size: 18px;
      font-weight: 800;
      letter-spacing: .5px;
    }

    .sub {
      font-size: 12px;
      color: var(--muted);
    }

    .card {
      background: var(--card);
      border: 1px solid var(--border);
      border-radius: 18px;
      padding: 14px;
      backdrop-filter: blur(10px);
    }

    .section-title {
      font-size: 12px;
      color: var(--muted);
      margin: 0 0 10px;
    }

    .row {
      display: grid;
      grid-template-columns: 1fr auto;
      gap: 10px;
      padding: 10px 6px;
      border-bottom: 1px solid rgba(255, 255, 255, .10);
      align-items: center;
    }

    .row:last-child {
      border-bottom: none;
    }

    .name {
      font-size: 15px;
      font-weight: 750;
    }

    .right {
      display: flex;
      align-items: center;
      gap: 10px;
    }

    .count {
      font-variant-numeric: tabular-nums;
      font-size: 18px;
      font-weight: 800;
      min-width: 40px;
      text-align: right;
    }

    .btns {
      display: flex;
      gap: 8px;
    }

    button {
      border: 1px solid rgba(255, 255, 255, .22);
      background: rgba(255, 255, 255, .10);
      color: var(--text);
      border-radius: 14px;
      padding: 10px 12px;
      font-size: 16px;
      font-weight: 800;
      cursor: pointer;
      min-width: 44px;
      transition: transform .08s ease;
    }

    button:active {
      transform: translateY(1px);
    }

    .footer {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
    }

    .ghost {
      background: rgba(255, 255, 255, .06);
    }

    .danger {
      background: rgba(255, 120, 120, .16);
    }

    .add {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
      margin-top: 10px;
    }
    /* ✅ select(선택창) 안 글씨 */
  .add select { 
   color: rgba(255,255,255,0.92);  /* 선택된 값 글자색 */
   background: rgba(255,255,255,0.06); /* 배경 유지 */
}

/* ✅ 드롭다운 목록(option) 글씨/배경 (특히 중요) */
  .add select option {
  color: #000;          /* 옵션 글자색: 검정(가독성 최고) */
  background: #fff;     /* 옵션 배경: 흰색 */
}
    .add input,
    .add select {
      width: 100%;
      padding: 12px;
      border-radius: 14px;
      border: 1px solid rgba(255, 255, 255, .18);
      background: rgba(255, 255, 255, .06);
      color: var(--text);
    }

    .add input::placeholder {
      color: rgba(255, 255, 255, .45);
    }
  </style>
  수량체크앱
  
</head>

<body>
  <div class="wrap">
    <div class="header">
      <div class="title">짐 체크</div>
      <div class="sub">내복 / 팬티 / 양말 (저장됨)</div>
    </div>

    <div class="card">
      <div class="section-title">현재 보유 수량</div>
      <div id="list"></div> <!-- JS가 렌더링할 영역 -->

      <div class="section-title" style="margin-top:14px;">항목 추가</div>
      <div class="add">
        <select id="catSel"> <!-- ✅ 카테고리 고정 3개 -->
          <option value="내복">내복</option>
          <option value="팬티">팬티</option>
          <option value="양말">양말</option>
        </select>
        <input id="nameInput" placeholder="예: 1707 두꺼운 양말 (thick)" />
      </div>
      <div style="margin-top:10px;">
        <button id="addBtn" class="ghost" style="width:100%;">+ 항목 추가</button>
      </div>
    </div>

    <div class="footer">
      <button id="resetAllBtn" class="danger">전체 0으로</button>
      <button id="exportBtn" class="ghost">백업(JSON)</button>
    </div>
  </div>

  <script>
    const KEY = "inventory_v3_fixedCats"; // ✅ localStorage 저장 키(버전)

    let items = []; // ✅ 기본 항목 없음(사용자가 직접 추가)

    let counts = {}; // ✅ id -> 수량 저장 객체

    const $list = document.getElementById("list");
    const $resetAllBtn = document.getElementById("resetAllBtn");
    const $exportBtn = document.getElementById("exportBtn");
    const $catSel = document.getElementById("catSel");
    const $nameInput = document.getElementById("nameInput");
    const $addBtn = document.getElementById("addBtn");

    function clamp0(n) { return n < 0 ? 0 : n; } // 음수 방지

    function save() { // ✅ 데이터 저장(안 날아가게 하는 핵심)
      localStorage.setItem(KEY, JSON.stringify({ items, counts })); // items와 counts를 함께 저장
    }

    function load() { // ✅ 데이터 불러오기(다음에 들어와도 유지)
      const raw = localStorage.getItem(KEY); // 저장된 값 읽기
      if (!raw) { save(); return; } // 처음이면 빈 상태 저장만 해둠
      try {
        const obj = JSON.parse(raw); // JSON 파싱
        if (Array.isArray(obj.items)) items = obj.items; // items 복원
        if (obj.counts && typeof obj.counts === "object") counts = obj.counts; // counts 복원
        // 혹시 items는 있는데 counts 키가 빠진 경우 0으로 채움
        for (const it of items) {
          if (typeof counts[it.id] !== "number") counts[it.id] = 0;
        }
      } catch {
        // 데이터가 깨졌으면 초기화(드물지만 대비)
        items = [];
        counts = {};
        save();
      }
    }

    function render() { // 화면 렌더링
      $list.innerHTML = ""; // 기존 내용 삭제

      // ✅ 고정 카테고리 순서대로 출력(원하는 순서 유지)
      const cats = ["내복", "팬티", "양말"];

      for (const cat of cats) { // 카테고리 반복
        const group = document.createElement("div"); // 그룹 컨테이너
        group.style.marginBottom = "10px";

        const h = document.createElement("div"); // 카테고리 제목
        h.className = "section-title";
        h.textContent = cat;
        group.appendChild(h);

        const filtered = items.filter(x => x.cat === cat); // 해당 카테고리 항목만

        if (filtered.length === 0) { // 항목이 없으면 안내 문구
          const empty = document.createElement("div");
          empty.style.color = "rgba(255,255,255,0.55)";
          empty.style.fontSize = "12px";
          empty.style.padding = "6px 6px 12px";
          empty.textContent = "등록된 항목 없음";
          group.appendChild(empty);
          $list.appendChild(group);
          continue;
        }

        for (const it of filtered) { // 항목 반복
          const row = document.createElement("div"); // row 생성
          row.className = "row";

          const left = document.createElement("div"); // 왼쪽(이름)
          left.innerHTML = `<div class="name">${it.label}</div>`; // 항목명 표시

          const right = document.createElement("div"); // 오른쪽(수량+버튼)
          right.className = "right";

          const countEl = document.createElement("div"); // 수량 표시
          countEl.className = "count";
          countEl.textContent = String(counts[it.id] ?? 0);

          const btns = document.createElement("div"); // 버튼 묶음
          btns.className = "btns";

          const minus = document.createElement("button"); // - 버튼
          minus.textContent = "-";
          minus.addEventListener("click", () => { // 클릭 시
            counts[it.id] = clamp0((counts[it.id] ?? 0) - 1); // 1 감소
            save(); // 저장
            render(); // 다시 렌더
          });

          const plus = document.createElement("button"); // + 버튼
          plus.textContent = "+";
          plus.addEventListener("click", () => { // 클릭 시
            counts[it.id] = (counts[it.id] ?? 0) + 1; // 1 증가
            save(); // 저장
            render(); // 다시 렌더
          });

          btns.appendChild(minus); // - 추가
          btns.appendChild(plus); // + 추가

          right.appendChild(countEl); // 수량 추가
          right.appendChild(btns); // 버튼 추가

          row.appendChild(left); // row에 왼쪽
          row.appendChild(right); // row에 오른쪽
          group.appendChild(row); // 그룹에 row
        }

        $list.appendChild(group); // 전체 리스트에 그룹 추가
      }
    }

    function makeId(cat, label) { // ✅ 입력값으로 고유 id 생성(중복 방지)
      const base = `${cat}_${label}`.toLowerCase().replace(/[^a-z0-9가-힣]+/g, "_").replace(/^_|_$/g, "");
      let id = base; // 기본 id
      let n = 2; // 중복시 suffix 숫자
      while (items.some(x => x.id === id)) { id = `${base}_${n++}`; } // 중복이면 _2, _3...
      return id; // 고유 id 반환
    }

    $addBtn.addEventListener("click", () => { // 항목 추가 버튼
      const cat = $catSel.value.trim(); // 선택한 카테고리
      const label = $nameInput.value.trim(); // 입력한 항목 이름
      if (!label) { alert("항목 이름을 입력해줘"); return; } // 빈 입력 방지

      const id = makeId(cat, label); // id 생성
      items.push({ id, cat, label }); // ✅ 선택한 cat에 항목 추가
      counts[id] = 0; // 수량 0으로 시작
      $nameInput.value = ""; // 입력창 비우기
      save(); // 저장(안 날아가게)
      render(); // 화면 갱신
    });

    $resetAllBtn.addEventListener("click", () => { // 전체 0으로 버튼
      for (const it of items) counts[it.id] = 0; // 모든 항목 0
      save(); // 저장
      render(); // 갱신
    });

    $exportBtn.addEventListener("click", () => { // 백업 버튼
      const data = JSON.stringify({ items, counts }, null, 2); // 보기 좋은 JSON 백업 데이터
      const w = window.open("", "_blank"); // 새 탭 열기
      w.document.write(`<pre style="white-space:pre-wrap;word-break:break-word;">${data}</pre>`); // JSON 보여주기
      w.document.title = "inventory_backup.json"; // 탭 제목
    });

    load(); // ✅ 시작할 때 저장된 데이터 복원
    render(); // ✅ 화면 출력
    save(); // ✅ KEY가 없으면 초기 상태라도 저장(안정용)
  </script>
</body>

</html>
