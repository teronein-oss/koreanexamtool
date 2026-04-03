# koreanexamtool[korean-exam-seum-style_1.html](https://github.com/user-attachments/files/26456549/korean-exam-seum-style_1.html)
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Seum Teachers Lab - 국어 문제 출제 도구</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; }
        .tab-button { transition: all 0.2s; }
        .tab-button.active { background: white; border-bottom: 3px solid #3b82f6; font-weight: bold; }
        .type-item { transition: all 0.2s; cursor: pointer; }
        .type-item:hover { background: #f3f4f6; }
        .type-item.selected { background: #fee2e2; border-left: 4px solid #ef4444; }
        .dropdown-header { cursor: pointer; }
        .dropdown-content { max-height: 0; overflow: hidden; transition: max-height 0.3s ease; }
        .dropdown-content.open { max-height: 1000px; }
        .rotate-icon { transition: transform 0.3s; }
        .rotate-icon.open { transform: rotate(90deg); }
    </style>
</head>
<body class="bg-gray-100">
    <!-- Header -->
    <div class="bg-gradient-to-r from-gray-900 to-gray-800 text-white px-6 py-4 shadow-lg">
        <div class="flex items-center justify-between max-w-7xl mx-auto">
            <div class="flex items-center gap-3">
                <h1 class="text-2xl font-bold">Seum<span class="text-red-500">Teachers Lab</span></h1>
                <span class="text-sm text-gray-400">국어 문제 출제 도구</span>
            </div>
            <div class="flex items-center gap-4">
                <select id="providerSelect" onchange="switchProvider()" class="bg-gray-700 text-white px-4 py-2 rounded border border-gray-600">
                    <option value="claude">Claude API</option>
                    <option value="gemini">Gemini API</option>
                </select>
                <select id="modelSelect" class="bg-gray-700 text-white px-4 py-2 rounded border border-gray-600">
                    <option value="claude-sonnet-4-20250514">Claude Sonnet 4</option>
                    <option value="claude-opus-4-20250514">Claude Opus 4</option>
                </select>
                <input type="password" id="apiKeyInput" placeholder="Claude API Key" 
                    class="bg-gray-700 text-white px-4 py-2 rounded border border-gray-600 w-64">
                <button onclick="saveApiKey()" class="bg-blue-600 hover:bg-blue-700 px-4 py-2 rounded">저장</button>
                <button onclick="showGuide()" class="bg-green-600 hover:bg-green-700 px-4 py-2 rounded">사용 가이드</button>
            </div>
        </div>
    </div>

    <!-- Tab Menu -->
    <div class="bg-white border-b shadow-sm">
        <div class="max-w-7xl mx-auto px-6">
            <div class="flex gap-2">
                <button class="tab-button px-6 py-4 active" onclick="switchTab('prompt')">
                    ⚙️ 유형별 프롬프트 설정
                </button>
                <button class="tab-button px-6 py-4" onclick="switchTab('input')">
                    📝 지문 입력
                </button>
                <button class="tab-button px-6 py-4" onclick="switchTab('output')">
                    📄 문항 출력
                </button>
            </div>
        </div>
    </div>

    <!-- Main Content -->
    <div class="max-w-7xl mx-auto p-6">
        <div class="flex gap-6">
            <!-- Left Sidebar -->
            <div class="w-80 bg-white rounded-lg shadow-lg p-4">
                <h3 class="font-bold text-gray-800 mb-4 pb-3 border-b">문제 유형 목록</h3>
                <div id="typeList"></div>
            </div>

            <!-- Main Panel -->
            <div class="flex-1">
                <!-- Tab 1: 유형별 프롬프트 설정 -->
                <div id="promptTab" class="tab-content bg-white rounded-lg shadow-lg p-6">
                    <div id="promptEditor">
                        <div class="text-center text-gray-500 py-20">
                            <div class="text-6xl mb-4">⚙️</div>
                            <h3 class="text-xl font-bold mb-2">유형을 선택하세요</h3>
                            <p>왼쪽에서 문제 유형을 선택하면 프롬프트를 편집할 수 있습니다</p>
                        </div>
                    </div>
                </div>

                <!-- Tab 2: 지문 입력 -->
                <div id="inputTab" class="tab-content bg-white rounded-lg shadow-lg p-6 hidden">
                    <h2 class="text-2xl font-bold mb-6 pb-4 border-b">지문 입력 및 출제</h2>
                    
                    <div class="mb-6">
                        <label class="block font-bold text-gray-700 mb-2">지문 내용</label>
                        <textarea id="passageInput" rows="15" 
                            class="w-full px-4 py-3 border rounded-lg focus:ring-2 focus:ring-blue-500"
                            placeholder="출제할 지문을 입력하세요..."></textarea>
                    </div>

                    <div class="mb-6">
                        <label class="block font-bold text-gray-700 mb-3">출제할 유형 선택 (복수 선택 가능)</label>
                        <div id="typeCheckboxes" class="grid grid-cols-2 gap-3"></div>
                    </div>

                    <div class="mb-6 bg-yellow-50 border border-yellow-200 rounded-lg p-4">
                        <h4 class="font-bold text-yellow-800 mb-2">💡 추가 입력 필요</h4>
                        <div id="additionalInputs"></div>
                    </div>

                    <button onclick="generateProblems()" 
                        class="w-full bg-green-600 hover:bg-green-700 text-white font-bold py-4 rounded-lg text-lg shadow-lg">
                        🎯 선택한 유형으로 문제 생성하기
                    </button>
                </div>

                <!-- Tab 3: 문항 출력 -->
                <div id="outputTab" class="tab-content bg-white rounded-lg shadow-lg p-6 hidden">
                    <div class="flex items-center justify-between mb-6 pb-4 border-b">
                        <h2 class="text-2xl font-bold">생성된 문제</h2>
                        <div class="flex gap-2">
                            <button onclick="copyAllProblems()" class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded">
                                📋 전체 복사
                            </button>
                            <button onclick="downloadAllProblems()" class="bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded">
                                💾 다운로드
                            </button>
                            <button onclick="clearProblems()" class="bg-red-600 hover:bg-red-700 text-white px-4 py-2 rounded">
                                🗑️ 전체 삭제
                            </button>
                        </div>
                    </div>
                    <div id="problemsOutput"></div>
                </div>
            </div>
        </div>
    </div>

    <!-- Loading Overlay -->
    <div id="loadingOverlay" class="hidden fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
        <div class="bg-white rounded-lg p-8 text-center">
            <div class="animate-spin rounded-full h-16 w-16 border-b-2 border-blue-500 mx-auto mb-4"></div>
            <p class="text-gray-700 font-bold text-lg">문제 생성 중...</p>
            <p class="text-gray-500 text-sm mt-2" id="loadingStatus">준비 중...</p>
        </div>
    </div>

    <script>
        // 문제 유형 데이터 (PDF 분석 기반)
        const examTypes = {
            "독서_비문학": {
                icon: "📖",
                name: "독서 (비문학)",
                types: {
                    "글의전개방식": {
                        name: "글의 전개방식 및 구조 파악",
                        code: "structure",
                        points: "2점",
                        inputs: ["지문"],
                        prompt: `당신은 한국 고등학교 국어 교사입니다. 다음 지문을 읽고 글의 전개 방식 및 구조를 파악하는 문제를 출제하세요.

[지문]
{지문}

[출제 기준]
- 문제 유형: 글의 전개방식 및 구조 파악
- 핵심 포인트: 전개 방식 비교, 구조적 특징 분석, (가)와 (나) 대조
- 선택지 패턴: "~와 달리", "모두", "~하고 있다" 형식
- 주의사항: 지문에서 명확히 확인 가능한 구조만 언급

[출력 형식]
문제: (가)와 (나)에 대한 설명으로 가장 적절한 것은?

선택지:
① 
② 
③ 
④ 
⑤ 

정답: ②
해설: (각 선택지별 옳고 그른 이유 설명)`
                    },
                    "내용일치": {
                        name: "내용 일치 및 사실적 확인",
                        code: "factual",
                        points: "2점",
                        inputs: ["지문"],
                        prompt: `당신은 한국 고등학교 국어 교사입니다. 다음 지문의 내용을 정확히 이해했는지 확인하는 문제를 출제하세요.

[지문]
{지문}

[출제 기준]
- 문제 유형: 내용 일치 및 사실적 확인
- 핵심 포인트: 핵심 개념, 인물/이론 주장, 세부 내용
- 함정 요소: 과장, 왜곡, 누락, 혼동을 활용한 오답 선택지
- 문제 형식: "적절하지 않은 것은?" 

[출력 형식]
문제: 윗글에 대한 이해로 적절하지 않은 것은?

선택지:
① 
② 
③ 
④ 
⑤ 

정답: ③
해설: (각 선택지가 왜 적절한지/부적절한지 설명)`
                    },
                    "이유추론": {
                        name: "특정 구절의 이유 추론",
                        code: "reason",
                        points: "2점",
                        inputs: ["지문", "밑줄친 부분"],
                        prompt: `당신은 한국 고등학교 국어 교사입니다. 다음 지문에서 밑줄친 부분의 이유를 추론하는 문제를 출제하세요.

[지문]
{지문}

[밑줄친 부분]
{밑줄친 부분}

[출제 기준]
- 문제 유형: 이유 추론
- 핵심 포인트: 인과관계, 논리적 근거, 맥락 파악
- 선택지 구조: "~때문이다", "~생각했기 때문이다" 형식
- 주의사항: 지문에서 추론 가능한 논리적 근거만 제시

[출력 형식]
문제: ㉮의 이유를 추론한 내용으로 가장 적절한 것은?

선택지:
① 
② 
③ 
④ 
⑤ 

정답: ②
해설: (왜 해당 이유가 가장 적절한지 설명)`
                    },
                    "보기적용형": {
                        name: "<보기> 적용형",
                        code: "application",
                        points: "3점",
                        inputs: ["지문", "보기 상황"],
                        prompt: `당신은 한국 고등학교 국어 교사입니다. 다음 지문의 내용을 <보기>의 상황에 적용하는 문제를 출제하세요.

[지문]
{지문}

[<보기> 상황]
{보기 상황}

[출제 기준]
- 문제 유형: 보기 적용형 (3점)
- 핵심 포인트: 지문의 원리/개념을 보기 상황에 정확히 적용
- 검증: 지문 원리 ↔ 보기 상황 연결성 명확히
- 난이도: 3점 (복합적 사고 요구)

[출력 형식]
문제: (가), (나)를 바탕으로 <보기>에 대해 보인 반응으로 적절하지 않은 것은? [3점]

< 보 기 >
{보기 상황}

선택지:
① 
② 
③ 
④ 
⑤ 

정답: ④
해설: (각 선택지가 지문의 어떤 부분을 어떻게 적용했는지 설명)`
                    },
                    "어휘문맥": {
                        name: "어휘의 문맥적 의미",
                        code: "vocabulary",
                        points: "2점",
                        inputs: ["지문", "밑줄친 어휘"],
                        prompt: `당신은 한국 고등학교 국어 교사입니다. 다음 지문에서 밑줄친 어휘의 문맥적 의미를 파악하는 문제를 출제하세요.

[지문]
{지문}

[밑줄친 어휘]
{밑줄친 어휘}

[출제 기준]
- 문제 유형: 어휘의 문맥적 의미
- 핵심 포인트: 다의어, 문맥 의미, 유사 용례
- 오답 선택지: 동음이의어, 다른 의미의 문장
- 주의사항: 지문 속 문맥에서의 정확한 의미 파악

[출력 형식]
문제: ⓐ와 문맥상 의미가 가장 가까운 것은?

선택지: (각 선택지는 동일 어휘가 들어간 다른 문장)
① 
② 
③ 
④ 
⑤ 

정답: ③
해설: (지문 속 의미와 각 선택지 의미 비교 설명)`
                    }
                }
            },
            "문학_시가": {
                icon: "✍️",
                name: "문학 - 시가",
                types: {
                    "시어함축": {
                        name: "시어·시구의 함축적 의미",
                        code: "poetic_meaning",
                        points: "2점",
                        inputs: ["작품", "밑줄친 시어"],
                        prompt: `당신은 한국 고등학교 국어 교사입니다. 다음 시 작품에서 밑줄친 시어의 함축적 의미와 기능을 파악하는 문제를 출제하세요.

[작품]
{작품}

[밑줄친 시어]
{밑줄친 시어}

[출제 기준]
- 문제 유형: 시어·시구의 함축적 의미 및 기능
- 핵심 포인트: 상징, 비유, 화자 정서, 시적 기능
- 형식: ㄱ,ㄴ,ㄷ,ㄹ 중 옳은 것을 고르는 방식
- 주의사항: 과도한 해석 지양, 작품 내 근거 기반

[출력 형식]
문제: ㉠~㉤에 대한 적절한 설명만을 <보기>에서 있는 대로 고른 것은?

< 보 기 >
ㄱ. 
ㄴ. 
ㄷ. 
ㄹ. 

① ㄱ, ㄷ  ② ㄴ, ㄷ  ③ ㄴ, ㄹ
④ ㄱ, ㄴ, ㄹ  ⑤ ㄱ, ㄷ, ㄹ

정답: ④
해설: (각 보기가 왜 적절한지/부적절한지 설명)`
                    },
                    "외적준거": {
                        name: "외적 준거에 따른 감상",
                        code: "external_criteria",
                        points: "2점",
                        inputs: ["작품", "보기 해설"],
                        prompt: `당신은 한국 고등학교 국어 교사입니다. 다음 작품을 <보기>의 관점으로 감상하는 문제를 출제하세요.

[작품]
{작품}

[<보기> 해설]
{보기 해설}

[출제 기준]
- 문제 유형: 외적 준거에 따른 감상
- 핵심 포인트: 작품론, 시대/작가 배경, 문학사조
- 선택지 형식: "~를 참고하여", "~를 바탕으로"
- 주의사항: 보기 내용을 정확히 작품에 적용

[출력 형식]
문제: <보기>를 참고하여 (가)를 감상한 내용으로 적절하지 않은 것은?

< 보 기 >
{보기 해설}

선택지:
① 
② 
③ 
④ 
⑤ 

정답: ③
해설: (보기 내용이 작품의 어느 부분과 연결되는지 설명)`
                    },
                    "표현특징": {
                        name: "표현상 특징 (수사법)",
                        code: "expression",
                        points: "2점",
                        inputs: ["작품 (가)", "작품 (나)"],
                        prompt: `당신은 한국 고등학교 국어 교사입니다. 다음 두 작품의 표현상 특징을 비교하는 문제를 출제하세요.

[작품 (가)]
{작품 (가)}

[작품 (나)]
{작품 (나)}

[출제 기준]
- 문제 유형: 표현상 특징 비교
- 핵심 포인트: 수사법, 운율, 이미지, 어조
- 선택지 형식: "(가)는 (나)와 달리", "모두 ~하고 있다"
- 주의사항: 명확히 확인 가능한 표현 기법만 언급

[출력 형식]
문제: (가)와 (나)에 대한 설명으로 가장 적절한 것은?

선택지:
① 
② 
③ 
④ 
⑤ 

정답: ②
해설: (각 작품의 표현 기법 분석 및 비교)`
                    },
                    "작품비교": {
                        name: "작품 간의 공통점·차이점",
                        code: "comparison",
                        points: "2점",
                        inputs: ["작품", "보기 작품"],
                        prompt: `당신은 한국 고등학교 국어 교사입니다. 다음 작품과 <보기> 작품의 공통점과 차이점을 파악하는 문제를 출제하세요.

[작품]
{작품}

[<보기> 작품]
{보기 작품}

[출제 기준]
- 문제 유형: 작품 간 공통점·차이점 파악
- 핵심 포인트: 주제, 표현, 화자 태도, 시상 전개
- 검증: 두 작품의 연결고리 명확히
- 주의사항: 공통점과 차이점 모두 고려

[출력 형식]
문제: (다)와 <보기>를 감상한 내용으로 가장 적절한 것은?

< 보 기 >
{보기 작품}

선택지:
① 
② 
③ 
④ 
⑤ 

정답: ④
해설: (공통점과 차이점을 근거와 함께 설명)`
                    },
                    "화자정서": {
                        name: "화자의 정서와 태도",
                        code: "speaker_emotion",
                        points: "2점",
                        inputs: ["작품"],
                        prompt: `당신은 한국 고등학교 국어 교사입니다. 다음 작품에서 화자의 정서와 태도를 파악하는 문제를 출제하세요.

[작품]
{작품}

[출제 기준]
- 문제 유형: 화자의 정서와 태도
- 핵심 포인트: 정서 변화, 태도, 어조, 시적 자아
- 주의사항: 과도한 해석 지양, 작품 내 근거 기반
- 선택지: 작품에서 확인 가능한 화자의 정서와 태도만 제시

[출력 형식]
문제: (가)~(다)에 대한 감상으로 적절한 것은?

선택지:
① 
② 
③ 
④ 
⑤ 

정답: ⑤
해설: (화자의 정서와 태도를 작품의 어느 부분에서 확인할 수 있는지 설명)`
                    }
                }
            },
            "문학_서사": {
                icon: "📕",
                name: "문학 - 서사",
                types: {
                    "서술특징": {
                        name: "서술상 특징 (시점과 거리)",
                        code: "narrative",
                        points: "2점",
                        inputs: ["소설 지문"],
                        prompt: `당신은 한국 고등학교 국어 교사입니다. 다음 소설 지문의 서술상 특징을 파악하는 문제를 출제하세요.

[지문]
{소설 지문}

[출제 기준]
- 문제 유형: 서술상 특징
- 핵심 포인트: 시점, 서술자, 거리, 시간/공간 구성
- 선택지 예시: 1인칭 주인공, 전지적 작가, 과거-현재 교차, 내적 독백
- 주의사항: 지문에서 명확히 확인 가능한 서술 기법만 언급

[출력 형식]
문제: 윗글의 서술상 특징으로 가장 적절한 것은?

선택지:
① 
② 
③ 
④ 
⑤ 

정답: ③
해설: (서술 기법의 근거를 지문에서 제시하며 설명)`
                    },
                    "인물소재": {
                        name: "인물의 심리 및 소재의 기능",
                        code: "character",
                        points: "2점",
                        inputs: ["지문", "밑줄친 부분"],
                        prompt: `당신은 한국 고등학교 국어 교사입니다. 다음 지문에서 밑줄친 부분의 의미와 기능을 파악하는 문제를 출제하세요.

[지문]
{지문}

[밑줄친 부분]
{밑줄친 부분}

[출제 기준]
- 문제 유형: 인물 심리 및 소재 기능 파악
- 핵심 포인트: 인물 성격, 심리, 소재 상징, 배경 기능
- 검증: 지문 내 명확한 근거 필수
- 주의사항: 과도한 상징적 해석 지양

[출력 형식]
문제: ㉠~㉤에 대한 이해로 적절하지 않은 것은?

선택지:
① 
② 
③ 
④ 
⑤ 

정답: ②
해설: (각 밑줄친 부분의 의미와 기능을 지문 근거와 함께 설명)`
                    },
                    "작품론": {
                        name: "외적 준거 (<보기> 작품론)",
                        code: "work_theory",
                        points: "3점",
                        inputs: ["작품", "보기 작품론"],
                        prompt: `당신은 한국 고등학교 국어 교사입니다. 다음 작품을 <보기>의 관점에서 이해하는 문제를 출제하세요.

[작품]
{작품}

[<보기> 작품론]
{보기 작품론}

[출제 기준]
- 문제 유형: 외적 준거 활용 감상 (3점)
- 핵심 포인트: 사회적 맥락, 작가 의도, 문학사적 의의
- 형식: ㉠~㉤ 부분과 연결
- 난이도: 3점 (복합적 이해 요구)

[출력 형식]
문제: <보기>를 바탕으로 윗글에 대해 보인 반응으로 적절하지 않은 것은?

< 보 기 >
{보기 작품론}

선택지:
① 
② 
③ 
④ 
⑤ 

정답: ④
해설: (보기 내용을 작품에 적용한 각 선택지 분석)`
                    },
                    "대화특징": {
                        name: "대화의 특징 파악 [A][B]",
                        code: "dialogue",
                        points: "2점",
                        inputs: ["대화 부분 [A]", "대화 부분 [B]"],
                        prompt: `당신은 한국 고등학교 국어 교사입니다. 다음 대화 부분 [A]와 [B]의 특징을 파악하는 문제를 출제하세요.

[대화 부분 [A]]
{대화 부분 [A]}

[대화 부분 [B]]
{대화 부분 [B]}

[출제 기준]
- 문제 유형: 대화의 특징 파악
- 핵심 포인트: 말하기 방식, 설득 전략, 화행
- 범위: 특정 단락 내 분석
- 주의사항: [A]와 [B]의 비교 분석

[출력 형식]
문제: [A]와 [B]의 말하기 방식에 대한 설명으로 가장 적절한 것은?

선택지:
① 
② 
③ 
④ 
⑤ 

정답: ③
해설: (각 대화의 말하기 방식 특징 설명)`
                    }
                }
            },
            "화법과_작문": {
                icon: "💬",
                name: "화법과 작문",
                types: {
                    "말하기방식": {
                        name: "말하기 방식 및 전략 파악",
                        code: "speaking",
                        points: "2점",
                        inputs: ["발표문/대화문"],
                        prompt: `당신은 한국 고등학교 국어 교사입니다. 다음 발표문(또는 대화문)의 말하기 방식과 전략을 파악하는 문제를 출제하세요.

[발표문/대화문]
{발표문/대화문}

[출제 기준]
- 문제 유형: 말하기 방식 및 전략 파악
- 핵심 포인트: 청중 고려, 상호작용, 전략, 매체 활용
- 체크리스트: 질문, 예시, 자료, 비유, 강조
- 주의사항: 발표문/대화문에서 확인 가능한 방식만 언급

[출력 형식]
문제: 위 발표자의 말하기 방식으로 적절하지 않은 것은?

선택지:
① 
② 
③ 
④ 
⑤ 

정답: ②
해설: (각 선택지가 발표문의 어느 부분에서 확인되는지 설명)`
                    },
                    "자료활용": {
                        name: "자료 활용 및 조직 전략",
                        code: "material",
                        points: "2점",
                        inputs: ["발표문", "자료"],
                        prompt: `당신은 한국 고등학교 국어 교사입니다. 다음 발표자가 제시한 자료의 활용 방식을 파악하는 문제를 출제하세요.

[발표문]
{발표문}

[자료]
{자료}

[출제 기준]
- 문제 유형: 자료 활용 및 조직
- 핵심 포인트: 자료 적절성, ㉠㉡㉢ 위치, 해석
- 검증: 자료-발언 연결 논리 명확
- 주의사항: 자료와 발표 내용의 정확한 연결 관계

[출력 형식]
문제: 발표자의 자료 활용에 대한 설명으로 가장 적절한 것은?

선택지:
① 
② 
③ 
④ 
⑤ 

정답: ③
해설: (각 자료가 발표의 어느 부분을 뒷받침하는지 설명)`
                    },
                    "청중반응": {
                        name: "청중의 반응 및 수용 양상",
                        code: "audience",
                        points: "2점",
                        inputs: ["발표문", "청중 반응"],
                        prompt: `당신은 한국 고등학교 국어 교사입니다. 다음 발표를 듣고 청중이 보인 반응을 분석하는 문제를 출제하세요.

[발표문]
{발표문}

[<보기> 청중 반응]
{청중 반응}

[출제 기준]
- 문제 유형: 청중 반응 및 수용 양상
- 핵심 포인트: 이해도, 수용도, 비판적 듣기
- 학생 반응: 학생1, 학생2, 학생3 형식
- 주의사항: 발표 내용과 청중 반응의 정확한 연결

[출력 형식]
문제: 발표 내용을 바탕으로 할 때, <보기>에 나타난 학생들의 반응에 대한 이해로 가장 적절한 것은?

< 보 기 >
학생 1: ...
학생 2: ...
학생 3: ...

선택지:
① 
② 
③ 
④ 
⑤ 

정답: ③
해설: (청중 반응과 발표 내용의 연결 관계 설명)`
                    },
                    "쓰기계획": {
                        name: "쓰기 계획의 반영 여부",
                        code: "writing_plan",
                        points: "2점",
                        inputs: ["초고", "작문 계획"],
                        prompt: `당신은 한국 고등학교 국어 교사입니다. 다음 초고에 작문 계획이 반영되었는지 파악하는 문제를 출제하세요.

[초고]
{초고}

[작문 계획]
{작문 계획}

[출제 기준]
- 문제 유형: 쓰기 계획 반영 여부
- 핵심 포인트: 계획 이행, 누락, 추가 필요
- 문제 형식: "반영되지 않은 것은?" 또는 "적절하지 않은 것은?"
- 주의사항: 초고와 계획의 정확한 대응 관계

[출력 형식]
문제: 초고에 반영된 글쓰기 계획으로 적절하지 않은 것은?

선택지:
① 
② 
③ 
④ 
⑤ 

정답: ④
해설: (각 계획이 초고의 어느 부분에 반영되었는지 설명)`
                    },
                    "자료활용작문": {
                        name: "자료 활용 및 조직 (작문)",
                        code: "writing_material",
                        points: "3점",
                        inputs: ["초고", "자료"],
                        prompt: `당신은 한국 고등학교 국어 교사입니다. 다음 자료를 활용하여 초고를 보완하는 문제를 출제하세요.

[초고]
{초고}

[자료]
{자료}

[출제 기준]
- 문제 유형: 자료 활용 및 조직 (3점)
- 핵심 포인트: 통계, 인터뷰, 논문 → 문단 보강
- 난이도: 3점
- 주의사항: 자료를 어느 문단에 어떻게 활용할지 구체적 제시

[출력 형식]
문제: <보기>는 학생이 초고를 보완하기 위해 추가로 수집한 자료이다. 자료의 활용 방안으로 적절하지 않은 것은? [3점]

< 보 기 >
ㄱ. 자료1
ㄴ. 자료2
ㄷ. 자료3

선택지:
① 
② 
③ 
④ 
⑤ 

정답: ②
해설: (각 자료를 어떻게 활용해야 하는지 설명)`
                    },
                    "고쳐쓰기": {
                        name: "고쳐쓰기 및 조건에 따른 표현",
                        code: "revision",
                        points: "2점",
                        inputs: ["초고", "조건"],
                        prompt: `당신은 한국 고등학교 국어 교사입니다. 다음 초고를 <조건>에 맞게 고쳐 쓰는 문제를 출제하세요.

[초고]
{초고}

[<조건>]
{조건}

[출제 기준]
- 문제 유형: 고쳐쓰기 및 조건 표현
- 핵심 포인트: 수사법, 문장 호응, 논리
- 조건 예시: 비유 사용, 대조 표현, ~을 강조
- 주의사항: 조건을 모두 충족하는 선택지 작성

[출력 형식]
문제: 다음 <조건>에 따라 초고의 [A]를 고쳐 쓴 것으로 가장 적절한 것은?

< 조 건 >
{조건}

선택지:
① 
② 
③ 
④ 
⑤ 

정답: ③
해설: (각 선택지가 조건을 충족하는지 분석)`
                    },
                    "통합": {
                        name: "대화-작문 통합",
                        code: "integrated",
                        points: "3점",
                        inputs: ["대화", "초고"],
                        prompt: `당신은 한국 고등학교 국어 교사입니다. 다음 대화 내용이 초고에 어떻게 반영되었는지 파악하는 문제를 출제하세요.

[(가) 대화]
{대화}

[(나) 초고]
{초고}

[출제 기준]
- 문제 유형: 대화-작문 통합 (3점)
- 핵심 포인트: 대화→글 반영, ⓐⓑ→문단 연결
- 구조: 대화 내용이 초고에 어떻게 반영되었는지
- 난이도: 3점

[출력 형식]
문제: (가)의 내용이 (나)에 제시된 양상으로 적절하지 않은 것은? [3점]

선택지:
① 
② 
③ 
④ 
⑤ 

정답: ④
해설: (대화 내용과 초고의 대응 관계 설명)`
                    }
                }
            }
        };

        // State
        let currentTab = 'prompt';
        let selectedCategory = null;
        let selectedType = null;
        let generatedProblems = [];

        // Initialize
        function init() {
            loadApiKey();
            renderTypeList();
            renderTypeCheckboxes();
        }

        // API Key
        function loadApiKey() {
            const savedClaude = localStorage.getItem('claude_api_key');
            const savedGemini = localStorage.getItem('gemini_api_key');
            const savedProvider = localStorage.getItem('api_provider') || 'claude';
            
            document.getElementById('providerSelect').value = savedProvider;
            switchProvider(savedProvider);
            
            if (savedClaude) localStorage.setItem('claude_api_key_temp', savedClaude);
            if (savedGemini) localStorage.setItem('gemini_api_key_temp', savedGemini);
        }

        function switchProvider(provider) {
            const selected = provider || document.getElementById('providerSelect').value;
            const modelSelect = document.getElementById('modelSelect');
            const apiKeyInput = document.getElementById('apiKeyInput');
            
            localStorage.setItem('api_provider', selected);
            
            if (selected === 'claude') {
                modelSelect.innerHTML = `
                    <option value="claude-sonnet-4-20250514">Claude Sonnet 4</option>
                    <option value="claude-opus-4-20250514">Claude Opus 4</option>
                `;
                apiKeyInput.placeholder = 'Claude API Key (sk-ant-...)';
                const saved = localStorage.getItem('claude_api_key_temp');
                if (saved) apiKeyInput.value = saved;
            } else {
                modelSelect.innerHTML = `
                    <option value="gemini-2.0-flash-exp">Gemini 2.0 Flash (Experimental)</option>
                    <option value="gemini-exp-1206">Gemini Exp 1206</option>
                    <option value="gemini-2.0-flash-thinking-exp-1219">Gemini 2.0 Flash Thinking</option>
                `;
                apiKeyInput.placeholder = 'Gemini API Key (AIza...)';
                const saved = localStorage.getItem('gemini_api_key_temp');
                if (saved) apiKeyInput.value = saved;
            }
        }

        function saveApiKey() {
            const provider = document.getElementById('providerSelect').value;
            const key = document.getElementById('apiKeyInput').value;
            
            if (!key) {
                alert('❌ API Key를 입력하세요.');
                return;
            }
            
            if (provider === 'claude') {
                localStorage.setItem('claude_api_key', key);
                localStorage.setItem('claude_api_key_temp', key);
            } else {
                localStorage.setItem('gemini_api_key', key);
                localStorage.setItem('gemini_api_key_temp', key);
            }
            
            alert(`✅ ${provider.toUpperCase()} API Key가 저장되었습니다.`);
        }

        function showGuide() {
            const guide = `
📚 API Key 발급 방법

[Claude API]
1. https://console.anthropic.com/ 접속
2. API Keys 메뉴에서 Create Key
3. sk-ant-로 시작하는 키 복사

[Gemini API]
1. https://aistudio.google.com/apikey 접속
2. Create API Key 클릭
3. AIza로 시작하는 키 복사

💡 Tip: 각 API를 번갈아 사용할 수 있습니다.
`;
            alert(guide);
        }

        // Tab Switch
        function switchTab(tab) {
            currentTab = tab;
            
            // Tab buttons
            document.querySelectorAll('.tab-button').forEach(btn => btn.classList.remove('active'));
            event.target.classList.add('active');
            
            // Tab contents
            document.querySelectorAll('.tab-content').forEach(content => content.classList.add('hidden'));
            document.getElementById(tab + 'Tab').classList.remove('hidden');
        }

        // Render Type List (Sidebar)
        function renderTypeList() {
            const container = document.getElementById('typeList');
            container.innerHTML = '';

            Object.keys(examTypes).forEach(categoryKey => {
                const category = examTypes[categoryKey];
                
                // Category Header
                const header = document.createElement('div');
                header.className = 'dropdown-header flex items-center justify-between p-3 bg-gray-100 rounded mb-2 hover:bg-gray-200';
                header.onclick = () => toggleCategory(categoryKey);
                header.innerHTML = `
                    <div class="flex items-center gap-2">
                        <span class="text-xl">${category.icon}</span>
                        <span class="font-bold text-sm">${category.name}</span>
                    </div>
                    <svg class="w-4 h-4 rotate-icon" id="arrow_${categoryKey}" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"></path>
                    </svg>
                `;
                
                // Dropdown
                const dropdown = document.createElement('div');
                dropdown.id = `dropdown_${categoryKey}`;
                dropdown.className = 'dropdown-content mb-3';
                
                Object.keys(category.types).forEach(typeKey => {
                    const type = category.types[typeKey];
                    const item = document.createElement('div');
                    item.className = 'type-item p-2 pl-4 mb-1 rounded';
                    item.id = `type_${categoryKey}_${typeKey}`;
                    item.onclick = (e) => {
                        e.stopPropagation();
                        selectType(categoryKey, typeKey);
                    };
                    item.innerHTML = `
                        <div class="flex justify-between items-center">
                            <span class="text-sm font-medium">${type.name}</span>
                            <span class="text-xs px-2 py-0.5 bg-gray-200 rounded">${type.points}</span>
                        </div>
                        <div class="text-xs text-gray-500 mt-1">${type.code}</div>
                    `;
                    dropdown.appendChild(item);
                });
                
                container.appendChild(header);
                container.appendChild(dropdown);
            });
        }

        function toggleCategory(categoryKey) {
            const dropdown = document.getElementById(`dropdown_${categoryKey}`);
            const arrow = document.getElementById(`arrow_${categoryKey}`);
            dropdown.classList.toggle('open');
            arrow.classList.toggle('open');
        }

        // Select Type
        function selectType(categoryKey, typeKey) {
            selectedCategory = categoryKey;
            selectedType = typeKey;
            
            const type = examTypes[categoryKey].types[typeKey];
            
            // Sidebar highlight
            document.querySelectorAll('.type-item').forEach(item => item.classList.remove('selected'));
            document.getElementById(`type_${categoryKey}_${typeKey}`).classList.add('selected');
            
            // Show prompt editor
            showPromptEditor(type);
        }

        // Show Prompt Editor
        function showPromptEditor(type) {
            const editor = document.getElementById('promptEditor');
            editor.innerHTML = `
                <div class="mb-6">
                    <div class="flex items-center justify-between mb-4 pb-3 border-b">
                        <div>
                            <h2 class="text-2xl font-bold text-gray-800">${type.name}</h2>
                            <p class="text-sm text-gray-500 mt-1">코드: ${type.code} | 배점: ${type.points}</p>
                        </div>
                        <button onclick="resetPrompt()" class="px-4 py-2 bg-gray-500 hover:bg-gray-600 text-white rounded">
                            🔄 기본값 복원
                        </button>
                    </div>
                </div>

                <div class="mb-4">
                    <label class="block font-bold text-gray-700 mb-2">필수 입력 항목</label>
                    <div class="flex gap-2">
                        ${type.inputs.map(inp => `<span class="px-3 py-1 bg-blue-100 text-blue-700 rounded text-sm">${inp}</span>`).join('')}
                    </div>
                </div>

                <div class="mb-4">
                    <label class="block font-bold text-gray-700 mb-2">메타프롬프트 (AI 출제 지시사항)</label>
                    <textarea id="promptTextarea" rows="20" 
                        class="w-full px-4 py-3 border rounded-lg font-mono text-sm">${type.prompt}</textarea>
                </div>

                <div class="flex gap-3">
                    <button onclick="savePrompt()" class="flex-1 bg-blue-600 hover:bg-blue-700 text-white py-3 rounded-lg font-bold">
                        💾 프롬프트 저장
                    </button>
                    <button onclick="testPrompt()" class="flex-1 bg-green-600 hover:bg-green-700 text-white py-3 rounded-lg font-bold">
                        🧪 테스트 출제
                    </button>
                </div>
            `;
        }

        function resetPrompt() {
            if (confirm('기본 프롬프트로 복원하시겠습니까?')) {
                const type = examTypes[selectedCategory].types[selectedType];
                document.getElementById('promptTextarea').value = type.prompt;
                alert('✅ 기본값으로 복원되었습니다.');
            }
        }

        function savePrompt() {
            const newPrompt = document.getElementById('promptTextarea').value;
            examTypes[selectedCategory].types[selectedType].prompt = newPrompt;
            
            // LocalStorage에 저장
            const storageKey = `prompt_${selectedCategory}_${selectedType}`;
            localStorage.setItem(storageKey, newPrompt);
            
            alert('✅ 프롬프트가 저장되었습니다.');
        }

        function testPrompt() {
            alert('🧪 테스트 출제 기능은 "지문 입력" 탭에서 사용하세요.');
            switchTab('input');
        }

        // Render Type Checkboxes
        function renderTypeCheckboxes() {
            const container = document.getElementById('typeCheckboxes');
            container.innerHTML = '';

            Object.keys(examTypes).forEach(categoryKey => {
                const category = examTypes[categoryKey];
                
                Object.keys(category.types).forEach(typeKey => {
                    const type = category.types[typeKey];
                    const div = document.createElement('div');
                    div.className = 'flex items-center gap-2 p-3 bg-gray-50 rounded border';
                    div.innerHTML = `
                        <input type="checkbox" id="check_${categoryKey}_${typeKey}" 
                            value="${categoryKey}:${typeKey}" 
                            class="type-checkbox w-4 h-4"
                            onchange="updateAdditionalInputs()">
                        <label for="check_${categoryKey}_${typeKey}" class="text-sm flex-1">
                            <span class="font-bold">${type.name}</span>
                            <span class="text-gray-500 text-xs ml-2">${type.points}</span>
                        </label>
                    `;
                    container.appendChild(div);
                });
            });
        }

        // Update Additional Inputs
        function updateAdditionalInputs() {
            const container = document.getElementById('additionalInputs');
            const checked = Array.from(document.querySelectorAll('.type-checkbox:checked'));
            
            if (checked.length === 0) {
                container.innerHTML = '<p class="text-gray-500 text-sm">선택한 유형이 없습니다</p>';
                return;
            }

            const additionalInputsNeeded = new Set();
            checked.forEach(cb => {
                const [cat, typ] = cb.value.split(':');
                const type = examTypes[cat].types[typ];
                type.inputs.forEach(inp => {
                    if (inp !== '지문') additionalInputsNeeded.add(inp);
                });
            });

            if (additionalInputsNeeded.size === 0) {
                container.innerHTML = '<p class="text-green-600 text-sm">✅ 지문만 입력하면 됩니다</p>';
                return;
            }

            container.innerHTML = '';
            additionalInputsNeeded.forEach(input => {
                const div = document.createElement('div');
                div.className = 'mb-3';
                div.innerHTML = `
                    <label class="block font-medium text-gray-700 mb-1 text-sm">${input}</label>
                    <textarea id="additional_${input.replace(/\s+/g, '_')}" rows="4" 
                        class="w-full px-3 py-2 border rounded text-sm"
                        placeholder="${input}을(를) 입력하세요..."></textarea>
                `;
                container.appendChild(div);
            });
        }

        // Generate Problems
        async function generateProblems() {
            const provider = document.getElementById('providerSelect').value;
            const apiKey = provider === 'claude' 
                ? localStorage.getItem('claude_api_key')
                : localStorage.getItem('gemini_api_key');
            
            if (!apiKey) {
                alert(`❌ ${provider.toUpperCase()} API Key를 먼저 입력하고 저장해주세요.`);
                return;
            }

            const passage = document.getElementById('passageInput').value.trim();
            if (!passage) {
                alert('❌ 지문을 입력하세요.');
                return;
            }

            const checked = Array.from(document.querySelectorAll('.type-checkbox:checked'));
            if (checked.length === 0) {
                alert('❌ 출제할 유형을 하나 이상 선택하세요.');
                return;
            }

            // Show loading
            document.getElementById('loadingOverlay').classList.remove('hidden');
            
            const problems = [];
            
            for (let i = 0; i < checked.length; i++) {
                const cb = checked[i];
                const [cat, typ] = cb.value.split(':');
                const type = examTypes[cat].types[typ];
                
                document.getElementById('loadingStatus').textContent = 
                    `${i + 1}/${checked.length} - ${type.name} 생성 중...`;

                try {
                    // Prepare inputs
                    const inputs = { '지문': passage };
                    type.inputs.forEach(inp => {
                        if (inp !== '지문') {
                            const id = `additional_${inp.replace(/\s+/g, '_')}`;
                            const elem = document.getElementById(id);
                            if (elem) inputs[inp] = elem.value.trim();
                        }
                    });

                    // Build prompt
                    let prompt = type.prompt;
                    Object.keys(inputs).forEach(key => {
                        prompt = prompt.replace(`{${key}}`, inputs[key] || '');
                    });

                    // Call API based on provider
                    let problemText;
                    if (provider === 'claude') {
                        problemText = await callClaudeAPI(apiKey, prompt);
                    } else {
                        problemText = await callGeminiAPI(apiKey, prompt);
                    }

                    problems.push({
                        type: type.name,
                        points: type.points,
                        content: problemText,
                        timestamp: new Date().toISOString()
                    });

                } catch (error) {
                    problems.push({
                        type: type.name,
                        points: type.points,
                        content: `❌ 오류 발생: ${error.message}`,
                        timestamp: new Date().toISOString()
                    });
                }
            }

            // Hide loading
            document.getElementById('loadingOverlay').classList.add('hidden');

            // Save and display
            generatedProblems = problems;
            displayProblems();
            switchTab('output');
        }

        // Claude API Call
        async function callClaudeAPI(apiKey, prompt) {
            const model = document.getElementById('modelSelect').value;
            
            const response = await fetch('https://api.anthropic.com/v1/messages', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'anthropic-version': '2023-06-01',
                    'x-api-key': apiKey
                },
                body: JSON.stringify({
                    model: model,
                    max_tokens: 4000,
                    messages: [{ role: 'user', content: prompt }]
                })
            });

            if (!response.ok) {
                const errorData = await response.json();
                throw new Error(errorData.error?.message || `Claude API Error: ${response.status}`);
            }

            const data = await response.json();
            return data.content[0].text;
        }

        // Gemini API Call
        async function callGeminiAPI(apiKey, prompt) {
            const model = document.getElementById('modelSelect').value;
            const url = `https://generativelanguage.googleapis.com/v1beta/models/${model}:generateContent?key=${apiKey}`;
            
            const response = await fetch(url, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    contents: [{
                        parts: [{
                            text: prompt
                        }]
                    }],
                    generationConfig: {
                        temperature: 0.7,
                        maxOutputTokens: 4000
                    }
                })
            });

            if (!response.ok) {
                const errorData = await response.json();
                throw new Error(errorData.error?.message || `Gemini API Error: ${response.status}`);
            }

            const data = await response.json();
            return data.candidates[0].content.parts[0].text;
        }

        // Display Problems
        function displayProblems() {
            const container = document.getElementById('problemsOutput');
            
            if (generatedProblems.length === 0) {
                container.innerHTML = `
                    <div class="text-center text-gray-500 py-20">
                        <div class="text-6xl mb-4">📝</div>
                        <h3 class="text-xl font-bold mb-2">생성된 문제가 없습니다</h3>
                        <p>"지문 입력" 탭에서 문제를 생성하세요</p>
                    </div>
                `;
                return;
            }

            container.innerHTML = generatedProblems.map((prob, idx) => `
                <div class="mb-6 bg-gray-50 rounded-lg p-6 border-l-4 border-blue-500">
                    <div class="flex items-center justify-between mb-4">
                        <div>
                            <h3 class="font-bold text-lg">${idx + 1}. ${prob.type}</h3>
                            <p class="text-sm text-gray-500">${prob.points} | ${new Date(prob.timestamp).toLocaleString('ko-KR')}</p>
                        </div>
                        <button onclick="copyProblem(${idx})" class="px-3 py-1 bg-blue-500 hover:bg-blue-600 text-white rounded text-sm">
                            📋 복사
                        </button>
                    </div>
                    <pre class="whitespace-pre-wrap font-sans text-sm leading-relaxed">${prob.content}</pre>
                </div>
            `).join('');
        }

        function copyProblem(idx) {
            const prob = generatedProblems[idx];
            const text = `${prob.type} (${prob.points})\n\n${prob.content}`;
            navigator.clipboard.writeText(text).then(() => {
                alert('✅ 클립보드에 복사되었습니다!');
            });
        }

        function copyAllProblems() {
            const text = generatedProblems.map((prob, idx) => 
                `[${idx + 1}] ${prob.type} (${prob.points})\n\n${prob.content}\n\n${'='.repeat(80)}\n`
            ).join('\n');
            navigator.clipboard.writeText(text).then(() => {
                alert('✅ 전체 문제가 클립보드에 복사되었습니다!');
            });
        }

        function downloadAllProblems() {
            const text = generatedProblems.map((prob, idx) => 
                `[${idx + 1}] ${prob.type} (${prob.points})\n생성 시각: ${new Date(prob.timestamp).toLocaleString('ko-KR')}\n\n${prob.content}\n\n${'='.repeat(80)}\n`
            ).join('\n');
            
            const blob = new Blob([text], { type: 'text/plain;charset=utf-8' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `국어문제_${new Date().toISOString().slice(0,10)}.txt`;
            a.click();
            URL.revokeObjectURL(url);
        }

        function clearProblems() {
            if (confirm('⚠️ 생성된 모든 문제를 삭제하시겠습니까?')) {
                generatedProblems = [];
                displayProblems();
            }
        }

        // Init
        init();
    </script>
</body>
</html>
