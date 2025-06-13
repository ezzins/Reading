<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>오늘의 독서 서비스</title>
  <style>
    body { font-family: sans-serif; margin: 0; display: flex; height: 100vh; }
    .sidebar { width: 220px; background-color: #4CAF50; color: white; display: flex; flex-direction: column; align-items: center; padding-top: 50px; }
    .sidebar button { background-color: transparent; border: none; color: white; font-size: 20px; margin: 20px 0; cursor: pointer; transition: 0.3s; }
    .sidebar button:hover { font-weight: bold; }
    .content { flex-grow: 1; padding: 50px; background-color: #f7f7f7; }
    .hidden { display: none; }
    .question { margin: 20px; }
    .result { margin-top: 30px; font-size: 20px; }
  </style>
</head>
<body>

  <div class="sidebar">
    <button onclick="showPage('emotion')">오늘의 감정 도서</button>
    <button onclick="showPage('weather')">오늘의 날씨 도서</button>
    <button onclick="showPage('sns')">독서 SNS</button>
    <button onclick="showPage('mystash')">MY 문장함</button>
  </div>

  <div class="content">

    <!-- 감정 도서 -->
    <div id="emotion-page" class="hidden">
      <h1>오늘의 감정 도서</h1>
      <div id="survey">
        <div class="question">
          <p>Q1. 현재 감정은?</p>
          <button onclick="answer(1,'우울')">우울</button>
          <button onclick="answer(1,'안정')">안정</button>
        </div>
        <div class="question hidden" id="q2">
          <p>Q2. 반응 성향은?</p>
          <button onclick="answer(2,'연결')">연결</button>
          <button onclick="answer(2,'혼자')">혼자</button>
        </div>
        <div class="question hidden" id="q3">
          <p>Q3. 자극 선호는?</p>
          <button onclick="answer(3,'자극')">자극</button>
          <button onclick="answer(3,'잔잔')">잔잔</button>
        </div>
      </div>
      <div id="emotion-result" class="result hidden"></div>
    </div>

    <!-- 날씨 도서 -->
    <div id="weather-page" class="hidden">
      <h1>오늘의 날씨 도서</h1>
      <div id="weather-text">날씨 정보를 불러오는 중...</div>
      <div id="weather-recommend" class="result"></div>
    </div>

    <!-- SNS -->
    <div id="sns-page" class="hidden">
      <h1>독서 SNS</h1>
      <h3>#읽는 중</h3>
      <input id="reading-book" placeholder="읽는 책 제목 입력">
      <button onclick="postReading()">등록</button>
      <p id="reading-result"></p>

      <h3>감상로그</h3>
      <input id="review" placeholder="한 문장 감상 입력">
      <button onclick="postReview()">등록</button>
      <p id="review-result"></p>
    </div>

    <!-- MY 문장함 -->
    <div id="mystash-page" class="hidden">
      <h1>MY 문장함</h1>
      <p>스크랩한 문장:</p>
      <ul id="stash-list"></ul>
    </div>

    <!-- 메인 페이지 -->
    <div id="default-page">
      <h1>📚 오늘의 독서 서비스에 오신 걸 환영합니다!</h1>
      <p>왼쪽 메뉴에서 시작해보세요.</p>
    </div>

  </div>

  <script>
    // 페이지 이동
    function showPage(page) {
      ['emotion', 'weather', 'sns', 'mystash', 'default'].forEach(p => {
        document.getElementById(p + '-page').classList.add('hidden');
      });
      document.getElementById(page + '-page').classList.remove('hidden');

      if(page === 'weather') getWeather();
      if(page === 'mystash') loadStash();
    }

    // 감정 설문
    let answers = {};
    function answer(q, ans) {
      answers[q] = ans;
      if(q === 1) document.getElementById("q2").classList.remove("hidden");
      if(q === 2) document.getElementById("q3").classList.remove("hidden");
      if(q === 3) showEmotionResult();
    }

    function showEmotionResult() {
      const combo = `${answers[1]}-${answers[2]}-${answers[3]}`;
      let text = "";
      let book = "";

      switch(combo) {
        case '우울-연결-잔잔':
          text = '“빛나지 않아도 괜찮아, 거기 있는 것만으로도 누군가에게는 위로가 되니까.”';
          book = "(공감과 위로)";
          break;
        case '우울-연결-자극':
          text = '“가장 어두운 밤도 끝나고 해는 떠오른다”';
          book = "레미제라블 - 빅토르 위고";
          break;
        case '안정-혼자-잔잔':
          text = '“부족하지도 넘치지도 않게 나다운 삶, 그래서 괜찮은 삶”';
          book = "(몰입과 집중)";
          break;
        case '우울-혼자-자극':
          text = '“끝이 없는 어둠도 결국 빛을 위한 준비일 뿐이다.”';
          book = "(변화와 돌파구)";
          break;
        default:
          text = '추천 결과 없음';
      }

      document.getElementById("survey").classList.add("hidden");
      document.getElementById("emotion-result").classList.remove("hidden");
      document.getElementById("emotion-result").innerHTML = text + "<br>" + book + 
        '<br><button onclick="saveStash(`'+text+'`)">스크랩</button>';
    }

    // 날씨 API
    function getWeather() {
      document.getElementById("weather-text").innerText = "날씨 정보를 불러오는 중...";

      const today = new Date();
      const year = today.getFullYear();
      const month = (today.getMonth() + 1).toString().padStart(2, '0');
      const day = today.getDate().toString().padStart(2, '0');
      const hour = today.getHours();

      const getBaseTime = (h) => {
        if (h < 2) return '2300';
        const baseTimes = ['0200','0500','0800','1100','1400','1700','2000','2300'];
        return baseTimes[Math.floor(h / 3)];
      };

      const base_date = year + month + day;
      const base_time = getBaseTime(hour);
      const serviceKey = '88OQfs5Tc45txi4rxRQBYJcjy5h9qX%2Fvhnsl2%2BnxP8BUY8wfjMzISoCq4kPZ6Oy2FOHZ21yfmgWw5YfqMdfdow%3D%3D';
      const nx = '60', ny = '127';
      const url = `https://apis.data.go.kr/1360000/VilageFcstInfoService_2.0/getVilageFcst?serviceKey=${serviceKey}&pageNo=1&numOfRows=1000&dataType=JSON&base_date=${base_date}&base_time=${base_time}&nx=${nx}&ny=${ny}`;

      fetch(url).then(res=>res.json()).then(data=>{
        const items = data.response.body.items.item;
        const sky = items.find(i => i.category === 'SKY');
        const pty = items.find(i => i.category === 'PTY');
        let text = '', recommend = '';

        if (pty && ['1','2','5','6'].includes(pty.fcstValue)) {
          text = '☔ 비 오는 날';
          recommend = '“빗소리처럼 조용히 스며드는 책 속으로.”';
        } else if (pty && ['3','7'].includes(pty.fcstValue)) {
          text = '❄️ 눈 오는 날';
          recommend = '“포근한 담요처럼 따뜻한 이야기를 읽어보세요.”';
        } else if (sky && sky.fcstValue === '1') {
          text = '☀️ 맑은 날';
          recommend = '“햇살 아래 펼쳐보는 인생의 페이지.”';
        } else {
          text = '⛅ 흐린 날';
          recommend = '“구름처럼 차분한 독서를 즐겨요.”';
        }

        document.getElementById("weather-text").innerText = text;
        document.getElementById("weather-recommend").innerHTML = recommend + 
          '<br><button onclick="saveStash(`'+recommend+'`)">스크랩</button>';
      }).catch(err=>{
        document.getElementById("weather-text").innerText = "날씨 정보를 불러올 수 없습니다.";
      });
    }

    // SNS 기능
    function postReading() {
      const val = document.getElementById("reading-book").value;
      document.getElementById("reading-result").innerText = "📖 지금 읽는 중: " + val;
    }
    function postReview() {
      const val = document.getElementById("review").value;
      document.getElementById("review-result").innerText = "📝 감상로그: " + val;
    }

    // MY 문장함 스크랩
    function saveStash(txt) {
      let stash = JSON.parse(localStorage.getItem("mystash") || "[]");
      stash.push(txt);
      localStorage.setItem("mystash", JSON.stringify(stash));
      alert("스크랩 완료!");
    }

    function loadStash() {
      let stash = JSON.parse(localStorage.getItem("mystash") || "[]");
      document.getElementById("stash-list").innerHTML = stash.map(t => `<li>${t}</li>`).join('');
    }
  </script>

</body>
</html>
