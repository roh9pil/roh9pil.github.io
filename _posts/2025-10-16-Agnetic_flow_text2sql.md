<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>LLM 기반 Text2SQL 기술 요약</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;500;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Noto Sans KR', sans-serif;
            background-color: #f0f2f5;
        }
        .card {
            background-color: white;
            border-radius: 12px;
            padding: 24px;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
            transition: transform 0.2s ease-in-out, box-shadow 0.2s ease-in-out;
        }
        .card:hover {
            transform: translateY(-5px);
            box-shadow: 0 8px 20px rgba(0, 0, 0, 0.12);
        }
        .header-gradient {
            background: linear-gradient(135deg, #4F46E5 0%, #818CF8 100%);
        }
        .icon {
            width: 48px;
            height: 48px;
        }
        .flow-arrow {
            position: relative;
            text-align: center;
            margin: 24px 0;
        }
        .flow-arrow::after {
            content: '';
            position: absolute;
            left: 50%;
            top: 100%;
            transform: translateX(-50%);
            width: 2px;
            height: 24px;
            background-color: #d1d5db;
        }
        .step-number {
            background-color: #4F46E5;
            color: white;
            border-radius: 50%;
            width: 32px;
            height: 32px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: 700;
            margin-right: 16px;
        }
        .tag {
            display: inline-block;
            background-color: #e0e7ff;
            color: #4338ca;
            padding: 4px 12px;
            border-radius: 9999px;
            font-size: 0.875rem;
            font-weight: 500;
        }
    </style>
</head>
<body class="p-4 md:p-8">
    <div class="max-w-4xl mx-auto">
        <!-- Header -->
        <header class="text-center mb-12">
            <div class="inline-block p-4 rounded-xl header-gradient mb-4">
                <svg class="w-12 h-12 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 20l4-16m4 4l4 4-4 4M6 16l-4-4 4-4"></path></svg>
            </div>
            <h1 class="text-4xl font-bold text-gray-800">LLM 기반 Text2SQL 기술 개요</h1>
            <p class="text-lg text-gray-600 mt-2">자연어 질문을 데이터베이스 언어(SQL)로 변환하는 기술의 모든 것</p>
        </header>

        <!-- Section 1: Challenges -->
        <section class="mb-12">
            <h2 class="text-2xl font-bold text-gray-800 mb-6 text-center">🤔 Text2SQL, 왜 어려울까요?</h2>
            <div class="grid md:grid-cols-3 gap-6">
                <div class="card text-center">
                    <div class="flex justify-center mb-4">
                        <svg class="icon text-red-500" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M7.5 8.25h9m-9 3H12m-9.75 1.51c0 1.6 1.123 2.994 2.707 3.227 1.129.166 2.27.293 3.423.379.35.026.67.21.865.501L12 21l2.755-4.133a1.14 1.14 0 01.865-.501 48.17 48.17 0 003.423-.379c1.584-.233 2.707-1.626 2.707-3.228V6.741c0-1.602-1.123-2.995-2.707-3.228A48.394 48.394 0 0012 3c-2.392 0-4.744.175-7.043.513C3.373 3.746 2.25 5.14 2.25 6.741v6.018z" /></svg>
                    </div>
                    <h3 class="font-bold text-lg mb-2">맥락 전달의 어려움</h3>
                    <p class="text-gray-600 text-sm">스키마, 컬럼 의미 등 명시적 정보와 데이터의 정확한 의미, 비즈니스 용어 같은 암묵적 정보를 함께 전달하기 어렵습니다.</p>
                </div>
                <div class="card text-center">
                    <div class="flex justify-center mb-4">
                        <svg class="icon text-yellow-500" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M9.879 7.519c1.171-1.025 3.071-1.025 4.242 0 1.172 1.025 1.172 2.687 0 3.712-.203.179-.43.326-.67.442-.745.361-1.45.999-1.45 1.827v.75M21 12a9 9 0 11-18 0 9 9 0 0118 0zm-9 5.25h.008v.008H12v-.008z" /></svg>
                    </div>
                    <h3 class="font-bold text-lg mb-2">자연어 의도 파악</h3>
                    <p class="text-gray-600 text-sm">같은 질문도 표현 방식이 다양하고, 질문 자체가 모호하여 사용자의 진짜 의도를 정확히 파악하기 힘듭니다.</p>
                </div>
                <div class="card text-center">
                    <div class="flex justify-center mb-4">
                        <svg class="icon text-blue-500" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M6.75 7.5l3 2.25-3 2.25m4.5 0h3m-9 8.25h13.5A2.25 2.25 0 0021 18V6a2.25 2.25 0 00-2.25-2.25H5.25A2.25 2.25 0 003 6v12a2.25 2.25 0 002.25 2.25z" /></svg>
                    </div>
                    <h3 class="font-bold text-lg mb-2">기술적 복잡성</h3>
                    <p class="text-gray-600 text-sm">Join, 서브쿼리, 집계 함수 등 복잡한 SQL 구문을 다뤄야 하며, 데이터베이스 방언(dialect) 차이도 고려해야 합니다.</p>
                </div>
            </div>
        </section>

        <!-- Section 2: Core Process -->
        <section class="mb-12">
            <h2 class="text-2xl font-bold text-gray-800 mb-8 text-center">💡 기본 Text2SQL 시스템의 핵심 3단계</h2>
            <div class="relative">
                <div class="border-l-2 border-dashed border-indigo-300 absolute h-full left-4 md:left-1/2 md:-translate-x-1/2"></div>
                <div class="space-y-10">
                    <div class="flex items-start md:items-center">
                        <div class="step-number flex-shrink-0">1</div>
                        <div class="ml-4 md:ml-0 md:w-5/12 md:text-right md:pr-8">
                            <div class="tag mb-2">분석</div>
                            <h3 class="font-bold text-lg text-gray-800">자연어 질문 분석</h3>
                            <p class="text-gray-600">사용자의 질문을 분석해 핵심 요소와 의도를 추출하고, 구조화된 형식으로 변환합니다.</p>
                        </div>
                    </div>
                     <div class="flex items-start md:items-center md:flex-row-reverse">
                        <div class="step-number flex-shrink-0">2</div>
                        <div class="ml-4 md:ml-0 md:w-5/12 md:text-left md:pl-8">
                             <div class="tag mb-2">생성</div>
                            <h3 class="font-bold text-lg text-gray-800">SQL 생성</h3>
                            <p class="text-gray-600">추출된 세부 사항을 바탕으로, 데이터베이스가 이해할 수 있는 유효한 SQL 구문으로 매핑합니다.</p>
                        </div>
                    </div>
                    <div class="flex items-start md:items-center">
                        <div class="step-number flex-shrink-0">3</div>
                        <div class="ml-4 md:ml-0 md:w-5/12 md:text-right md:pr-8">
                             <div class="tag mb-2">실행 및 반환</div>
                            <h3 class="font-bold text-lg text-gray-800">데이터베이스 쿼리</h3>
                            <p class="text-gray-600">AI가 생성한 SQL을 실제 데이터베이스에서 실행하고, 검색된 결과를 사용자에게 반환합니다.</p>
                        </div>
                    </div>
                </div>
            </div>
        </section>

        <!-- Section 3: Methodologies -->
        <section class="mb-12">
            <h2 class="text-2xl font-bold text-gray-800 mb-6 text-center">🛠️ LLM 성능을 높이는 2가지 핵심 방법론</h2>
            <div class="grid md:grid-cols-2 gap-6">
                <div class="card">
                    <h3 class="font-bold text-xl mb-3">1. 프롬프트 엔지니어링</h3>
                    <p class="text-gray-600 mb-4">LLM이 질문을 더 똑똑하게 이해하도록 유도하는 기술입니다. 데이터베이스 구조, 예시 질문과 답변, 시스템 설명 등을 프롬프트에 잘 설계해서 넣습니다.</p>
                    <div class="bg-gray-100 p-4 rounded-lg text-sm text-gray-700">
                        <p><strong>입력:</strong> 사용자 질문, 테이블/스키마 정의, 샘플 쿼리/결과</p>
                        <p class="my-2"><strong>프로세스:</strong> 잘 설계된 프롬프트 → LLM</p>
                        <p><strong>출력:</strong> SQL 반환 → 검증 → 쿼리 실행 → 결과</p>
                    </div>
                </div>
                <div class="card">
                    <h3 class="font-bold text-xl mb-3">2. 파인튜닝 (미세조정)</h3>
                    <p class="text-gray-600 mb-4">기존 LLM을 실제 업무 데이터(질의/정답)로 추가 학습시켜, 특정 데이터베이스와 도메인에 특화된 모델로 만드는 과정입니다.</p>
                    <div class="bg-gray-100 p-4 rounded-lg text-sm text-gray-700">
                        <p><strong>입력:</strong> 사용자 질문, 테이블/스키마 정의</p>
                        <p class="my-2"><strong>프로세스:</strong> 프롬프트 → <span class="font-bold text-indigo-600">Fine-tuned LLM</span></p>
                        <p><strong>출력:</strong> SQL 반환 → 검증 → 쿼리 실행 → 결과</p>
                    </div>
                </div>
            </div>
        </section>

        <!-- Section 4: Agentic Workflow -->
        <section>
            <h2 class="text-2xl font-bold text-gray-800 mb-2 text-center">🚀 새로운 패러다임: Agentic Workflow</h2>
            <p class="text-center text-gray-600 mb-8">단일 LLM이 아닌, 각자 역할을 가진 '전문가 에이전트'들이 협업하여 문제를 해결합니다.</p>
            <div class="card">
                <h3 class="font-bold text-xl mb-4">전체 아키텍처 흐름</h3>
                <div class="text-center text-gray-700 font-medium p-4 bg-gray-50 rounded-lg flex flex-wrap justify-center items-center gap-x-4 gap-y-2">
                    <span>사용자</span>
                    <span class="text-gray-400">→</span>
                    <span>클라이언트</span>
                    <span class="text-gray-400">→</span>
                    <span>웹서버</span>
                    <span class="text-gray-400">→</span>
                    <span class="font-bold text-indigo-600">AI 에이전트 서버</span>
                    <span class="text-gray-400">→</span>
                     <span class="flex items-center">LLM <span class="text-xs text-gray-500 ml-1">(Claude)</span></span>
                    <span class="text-gray-400">→</span>
                    <span>RAG & DB</span>
                    <span class="text-gray-400">→</span>
                    <span>결과</span>
                </div>

                <h3 class="font-bold text-xl mt-8 mb-4">에이전트들의 협업 방식</h3>
                 <div class="grid md:grid-cols-3 gap-4 text-center">
                    <div class="p-4 bg-indigo-50 rounded-lg">
                        <h4 class="font-bold">1. Question Agent</h4>
                        <p class="text-sm text-gray-600">질문을 받고, 의도/모호성/복잡도를 분석하여 정제하고 분할합니다.</p>
                    </div>
                    <div class="flex items-center justify-center">
                      <svg class="w-6 h-6 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 8l4 4m0 0l-4 4m4-4H3"></path></svg>
                    </div>
                    <div class="p-4 bg-purple-50 rounded-lg">
                        <h4 class="font-bold">2. Query Agent(s)</h4>
                        <p class="text-sm text-gray-600">분할된 각 질문에 대해 SQL을 생성, 실행하고 결과를 저장합니다.</p>
                    </div>
                     <div class="md:col-span-3 flex items-center justify-center my-2">
                         <svg class="w-6 h-6 text-gray-400 transform rotate-90 md:rotate-0" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 8l4 4m0 0l-4 4m4-4H3"></path></svg>
                    </div>
                     <div class="p-4 bg-green-50 rounded-lg md:col-start-2">
                        <h4 class="font-bold">3. Answer Agent</h4>
                        <p class="text-sm text-gray-600">각 Query Agent의 결과를 종합하여 최종 답변(텍스트, 차트 등)을 생성합니다.</p>
                    </div>
                </div>
            </div>
        </section>

    </div>
</body>
</html>

