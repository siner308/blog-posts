---
title: "칩 아키텍처 생태계 완전 정복"
subtitle: "ARM · x86 · RISC-V, 그리고 로열티의 흐름"
tags:
    - hardware
    - investment
date: 2026-06-02
image: https://raw.githubusercontent.com/siner308/blog-posts/master/assets/chip-isa-ecosystem-thumbnail.png
---

<style>
.chip-isa .tag.arm { background: rgba(0,184,212,0.15); color: #00b8d4; border: 1px solid #00b8d4; }
.chip-isa .tag.x86 { background: rgba(247,129,102,0.15); color: #f78166; border: 1px solid #f78166; }
.chip-isa .tag.riscv { background: rgba(126,231,135,0.15); color: #7ee787; border: 1px solid #7ee787; }
</style>

<div class="html-doc chip-isa">

<header>
  <p class="sub" style="color:var(--muted);font-size:1rem">개발자의 <code>x86_64</code> / <code>arm64</code> 빌드 타겟이 실제로 무엇을 의미하는지부터, Apple·Tesla·Qualcomm·AWS의 칩 전략과 ARM Holdings가 돈을 버는 구조까지 — 2026-06-02 학습 자료</p>
</header>

<div class="callout">
<strong>✓ 교차 검증됨:</strong> 이 자료의 핵심 사실들은 독립 모델(OpenAI Codex)의 웹 검색 팩트체크를 거쳐, <strong>SEC EDGAR(ARM FY2026 20-F)·Reuters·RISC-V 연차보고서·SHD Group 보고서</strong> 등 1차 출처로 수치를 교정했습니다(2026-06-02). 교정 내역은 각 섹션과 말미 "검증 노트"에 표기.
</div>

<div class="panel">
  <strong>목차</strong>
  <div class="toc">
    <p><a href="#s1">1. 가장 큰 그림 — ISA란 무엇인가</a></p>
    <p><a href="#s2">2. 세 가지 ISA 비교</a></p>
    <p><a href="#s25">2.5 ARM 아키텍처 기술 심층 — 왜 빠르고 전력 효율적인가</a></p>
    <p><a href="#s3">3. ARM의 비즈니스 모델 — 라이선스 vs 로열티</a></p>
    <p><a href="#s4">4. 칩 회사 지도 — 누가 무엇을 따르나</a></p>
    <p><a href="#s5">5. 로열티는 실제로 어떻게 흐르나</a></p>
    <p><a href="#s6">6. RISC-V — 무료 오픈소스 ISA의 위협</a></p>
    <p><a href="#s65">6.5 RISC-V 채택 현황 — 실데이터 추적</a></p>
    <p><a href="#s66">6.6 중국 RISC-V 전선 — 심층 추적</a></p>
    <p><a href="#s7">7. 종합 — 투자자 관점의 함의</a></p>
    <p><a href="#s8">8. 용어 사전</a></p>
  </div>
</div>

<h2 id="s1">1. 가장 큰 그림 — ISA란 무엇인가</h2>

<p><strong>ISA(Instruction Set Architecture, 명령어 집합 구조)</strong>는 "CPU가 알아듣는 언어의 규격"입니다. 소프트웨어(기계어)와 하드웨어(트랜지스터) 사이의 <em>계약서</em>예요. <code>ADD</code>, <code>LOAD</code>, <code>STORE</code> 같은 명령어가 비트 단위로 어떻게 인코딩되는지를 정의합니다.</p>

<p>당신이 <code>GOARCH=arm64 go build</code> 또는 <code>docker build --platform linux/arm64</code>를 실행하면, 컴파일러는 소스코드를 <strong>ARM ISA 규격의 기계어</strong>로 번역합니다. 그 바이너리는 x86 CPU에서는 한 글자도 못 읽습니다 — 언어가 다르기 때문입니다. 이게 빌드 타겟이 갈리는 근본 이유입니다.</p>

<figure>
<svg viewBox="0 0 760 300" role="img" aria-label="소프트웨어와 하드웨어 사이의 ISA 계층 다이어그램">
  <rect x="180" y="20" width="400" height="44" rx="8" fill="#1c2330" stroke="#30363d"/>
  <text x="380" y="47" fill="#e6edf3" font-size="16" text-anchor="middle">소스 코드 (C, Go, Rust, Swift…)</text>
  <text x="380" y="84" fill="#9098a3" font-size="13" text-anchor="middle">↓ 컴파일러 (gcc / clang / go) — 타겟 ISA 선택</text>
  <rect x="120" y="100" width="240" height="44" rx="8" fill="rgba(0,184,212,0.12)" stroke="#00b8d4"/>
  <text x="240" y="121" fill="#00b8d4" font-size="14" text-anchor="middle" font-weight="600">arm64 기계어</text>
  <text x="240" y="138" fill="#9098a3" font-size="11" text-anchor="middle">ARM ISA 규격</text>
  <rect x="400" y="100" width="240" height="44" rx="8" fill="rgba(247,129,102,0.12)" stroke="#f78166"/>
  <text x="520" y="121" fill="#f78166" font-size="14" text-anchor="middle" font-weight="600">x86_64 기계어</text>
  <text x="520" y="138" fill="#9098a3" font-size="11" text-anchor="middle">x86 ISA 규격</text>
  <line x1="60" y1="170" x2="700" y2="170" stroke="#d2a8ff" stroke-width="2" stroke-dasharray="6 4"/>
  <text x="380" y="186" fill="#d2a8ff" font-size="12" text-anchor="middle" font-weight="600">━━ ISA = 소프트웨어 ↔ 하드웨어 계약서 ━━</text>
  <rect x="120" y="206" width="240" height="60" rx="8" fill="#161b22" stroke="#00b8d4"/>
  <text x="240" y="231" fill="#e6edf3" font-size="13" text-anchor="middle">ARM ISA 구현 칩</text>
  <text x="240" y="251" fill="#9098a3" font-size="11" text-anchor="middle">Apple M, Graviton, 스냅드래곤</text>
  <rect x="400" y="206" width="240" height="60" rx="8" fill="#161b22" stroke="#f78166"/>
  <text x="520" y="231" fill="#e6edf3" font-size="13" text-anchor="middle">x86 ISA 구현 칩</text>
  <text x="520" y="251" fill="#9098a3" font-size="11" text-anchor="middle">Intel Core, AMD Ryzen</text>
  <line x1="240" y1="64" x2="240" y2="100" stroke="#9098a3" stroke-width="1.5" marker-end="url(#a)"/>
  <line x1="520" y1="64" x2="520" y2="100" stroke="#9098a3" stroke-width="1.5" marker-end="url(#a)"/>
  <line x1="240" y1="144" x2="240" y2="206" stroke="#00b8d4" stroke-width="1.5" marker-end="url(#a)"/>
  <line x1="520" y1="144" x2="520" y2="206" stroke="#f78166" stroke-width="1.5" marker-end="url(#a)"/>
  <defs>
    <marker id="a" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto">
      <path d="M0,0 L8,4 L0,8 Z" fill="#9098a3"/>
    </marker>
  </defs>
</svg>
<figcaption>같은 소스코드라도 컴파일 타겟 ISA에 따라 전혀 다른 기계어가 나오고, 서로 다른 칩에서만 실행된다.</figcaption>
</figure>

<div class="callout">
<strong>핵심 통찰:</strong> ISA는 "설계도(blueprint)"이지 "칩(물건)"이 아닙니다. 누군가는 ISA를 <em>소유</em>하고(설계도 주인), 누군가는 그 ISA를 <em>구현한 칩</em>을 만듭니다(공장). 이 둘이 분리될 수 있다는 점이 ARM 비즈니스 모델 전체의 출발점입니다.
</div>

<h2 id="s2">2. 세 가지 ISA 비교</h2>

<div class="table-scroll"><table>
<thead>
<tr><th>구분</th><th><span class="tag x86">x86 / x86-64</span></th><th><span class="tag arm">ARM</span></th><th><span class="tag riscv">RISC-V</span></th></tr>
</thead>
<tbody>
<tr><td>소유 주체</td><td>Intel + AMD (양자 크로스 라이선스)</td><td>ARM Holdings (Softbank 자회사)</td><td><strong>누구도 소유 안 함</strong> — RISC-V International (비영리 재단)</td></tr>
<tr><td>설계 철학</td><td>CISC (복잡 명령어)</td><td>RISC (간결 명령어)</td><td>RISC (간결 명령어, 모듈식)</td></tr>
<tr><td>라이선스</td><td>사실상 폐쇄 — 진입장벽 매우 높음 (현재 Intel·AMD 양강, VIA 등 군소)</td><td>유료 라이선스 (ARM이 허가)</td><td><strong>무료·오픈소스</strong> — 로열티 0</td></tr>
<tr><td>주력 영역</td><td>데스크톱·노트북·서버(전통)</td><td>모바일(지배적) → 서버·PC·차로 확장 중</td><td>임베디드·MCU → 데이터센터·AI 가속기로 부상</td></tr>
<tr><td>전력 효율</td><td>상대적으로 낮음(고성능 지향)</td><td>높음(모바일 DNA)</td><td>구현에 따라 다름(설계 자유도 높음)</td></tr>
<tr><td>대표 칩</td><td>Intel Core, AMD Ryzen/EPYC</td><td>Apple M, AWS Graviton, 스냅드래곤</td><td>SiFive, Alibaba T-Head, Tenstorrent</td></tr>
</tbody>
</table></div>

<p class="src">출처: Wikipedia "x86-64"(AMD가 1999~2000 x86 64비트 확장 개발), "RISC-V"(오픈 표준 ISA, 로열티 없이 구현 가능), "Arm Holdings", 2026-06-02 확인.</p>

<div class="callout warn">
<strong>RISC vs CISC 한 줄 요약:</strong> CISC(x86)는 "명령어 하나가 많은 일을 한다"(복잡하지만 코드가 짧음), RISC(ARM·RISC-V)는 "명령어를 단순하게 쪼개고 많이 쓴다"(단순해서 전력·설계 효율↑). 모바일·데이터센터가 전력 효율을 중시하면서 RISC 계열이 약진했습니다.
</div>

<h2 id="s25">2.5 ARM 아키텍처 기술 심층 — 왜 빠르고 전력 효율적인가</h2>

<p>앞 절은 "RISC가 단순하다"고만 했습니다. 여기서는 <strong>실제 하드웨어 엔지니어링 수준</strong>에서, ARM(AArch64)이 x86-64 대비 무엇이 다르고 그 차이가 왜 성능·전력 우위로 이어지는지 짚습니다. 개발자가 체감하는 "arm 빌드"의 밑바닥입니다.</p>

<h3>① 고정 길이 명령어 — 디코더 병렬화의 열쇠</h3>
<p>AArch64 명령어는 <strong>전부 정확히 4바이트(32비트) 고정</strong>입니다. x86은 <strong>1~15바이트 가변 길이</strong>입니다. 사소해 보이지만 이게 현대 CPU 성능의 핵심 병목을 가릅니다.</p>

<figure>
<svg viewBox="0 0 760 250" role="img" aria-label="고정 길이 vs 가변 길이 명령어 디코드 비교">
  <text x="20" y="28" fill="#00b8d4" font-size="14" font-weight="700">ARM: 고정 4바이트 — 명령어 경계를 즉시 안다</text>
  <g>
    <rect x="20" y="40" width="110" height="34" rx="5" fill="rgba(0,184,212,0.15)" stroke="#00b8d4"/><text x="75" y="62" fill="#e6edf3" font-size="12" text-anchor="middle">명령 1</text>
    <rect x="135" y="40" width="110" height="34" rx="5" fill="rgba(0,184,212,0.15)" stroke="#00b8d4"/><text x="190" y="62" fill="#e6edf3" font-size="12" text-anchor="middle">명령 2</text>
    <rect x="250" y="40" width="110" height="34" rx="5" fill="rgba(0,184,212,0.15)" stroke="#00b8d4"/><text x="305" y="62" fill="#e6edf3" font-size="12" text-anchor="middle">명령 3</text>
    <rect x="365" y="40" width="110" height="34" rx="5" fill="rgba(0,184,212,0.15)" stroke="#00b8d4"/><text x="420" y="62" fill="#e6edf3" font-size="12" text-anchor="middle">명령 4</text>
    <rect x="480" y="40" width="110" height="34" rx="5" fill="rgba(0,184,212,0.15)" stroke="#00b8d4"/><text x="535" y="62" fill="#e6edf3" font-size="12" text-anchor="middle">명령 5</text>
    <rect x="595" y="40" width="110" height="34" rx="5" fill="rgba(0,184,212,0.15)" stroke="#00b8d4"/><text x="650" y="62" fill="#e6edf3" font-size="12" text-anchor="middle">명령 6</text>
  </g>
  <text x="20" y="98" fill="#7ee787" font-size="11.5">→ 6개 디코더를 한 번에 가동 (8-wide도 쉬움). 경계 계산 불필요.</text>
  <text x="20" y="138" fill="#f78166" font-size="14" font-weight="700">x86: 가변 1~15바이트 — 앞을 풀어야 다음 경계를 안다</text>
  <g>
    <rect x="20" y="150" width="60" height="34" rx="5" fill="rgba(247,129,102,0.15)" stroke="#f78166"/><text x="50" y="172" fill="#e6edf3" font-size="11" text-anchor="middle">1B</text>
    <rect x="85" y="150" width="150" height="34" rx="5" fill="rgba(247,129,102,0.15)" stroke="#f78166"/><text x="160" y="172" fill="#e6edf3" font-size="11" text-anchor="middle">7바이트</text>
    <rect x="240" y="150" width="90" height="34" rx="5" fill="rgba(247,129,102,0.15)" stroke="#f78166"/><text x="285" y="172" fill="#e6edf3" font-size="11" text-anchor="middle">3B</text>
    <rect x="335" y="150" width="200" height="34" rx="5" fill="rgba(247,129,102,0.15)" stroke="#f78166"/><text x="435" y="172" fill="#e6edf3" font-size="11" text-anchor="middle">11바이트</text>
    <rect x="540" y="150" width="60" height="34" rx="5" fill="rgba(247,129,102,0.15)" stroke="#f78166"/><text x="570" y="172" fill="#e6edf3" font-size="11" text-anchor="middle">2B</text>
  </g>
  <text x="20" y="208" fill="#ffb454" font-size="11.5">→ "명령 2가 어디서 끝나지?"를 먼저 풀어야 명령 3을 찾음. 디코더 폭 확장이 비싸고 전력 먹음.</text>
  <text x="20" y="232" fill="#9098a3" font-size="11">그래서 Apple Firestorm(M1) 코어는 8-wide 디코드, 고성능 x86은 통상 4~6-wide에 머문다.</text>
</svg>
<figcaption>고정 길이의 진짜 가치는 "디코드 병렬화". 매 사이클 더 많은 명령을 동시에 해석할수록 IPC(사이클당 명령)가 올라간다. 출처: Wikipedia "AArch64"(A64 4-byte fixed), "x86-64". 2026-06-03.</figcaption>
</figure>

<h3>② x86-64 vs AArch64 — 엔지니어링 스펙 대조</h3>
<div class="table-scroll"><table>
<thead><tr><th>항목</th><th><span class="tag x86">x86-64</span></th><th><span class="tag arm">AArch64</span></th><th>의미</th></tr></thead>
<tbody>
<tr><td>범용 레지스터</td><td>16개</td><td><strong>31개</strong></td><td>레지스터 압박↓ → 메모리 접근↓ → 빠름·저전력. 컴파일러 최적화 여지↑</td></tr>
<tr><td>명령어 길이</td><td>가변 1~15B</td><td><strong>고정 4B</strong></td><td>디코더 병렬화 용이 (위 ①)</td></tr>
<tr><td>주소 지정 모드</td><td>다수·복잡</td><td>load-store (메모리는 load/store만)</td><td>연산은 레지스터에서만 → 파이프라인 단순·예측 가능</td></tr>
<tr><td>메모리 일관성</td><td><strong>strong (TSO)</strong></td><td><strong>weak (relaxed)</strong></td><td>★ 아래 ③ — 개발자에게 직접 영향</td></tr>
<tr><td>벡터(SIMD)</td><td>SSE/AVX(고정 폭)</td><td>NEON + <strong>SVE</strong>(가변 폭)</td><td>SVE는 하드웨어 벡터 길이에 코드가 자동 적응</td></tr>
<tr><td>디코드 폭(대표 코어)</td><td>~4~6-wide</td><td>Apple Firestorm <strong>8-wide</strong></td><td>IPC 우위의 직접 원천</td></tr>
</tbody>
</table></div>
<p class="src">출처: Wikipedia "X86-64"(GPR 8→16), "AArch64"(31 general-purpose 64-bit registers, A64 fixed 4-byte, SVE), "Apple M1"(Firestorm), "Memory ordering"(x86 strongest, ARM weak). 2026-06-03 확인.</p>

<h3>③ 메모리 일관성 모델 — 개발자가 실제로 데이는 지점</h3>
<div class="callout warn">
<p>x86은 <strong>strong ordering(TSO)</strong>: 멀티스레드에서 메모리 쓰기 순서가 대체로 보존됩니다. ARM은 <strong>weak ordering</strong>: CPU가 성능을 위해 메모리 연산 순서를 자유롭게 재배열합니다.</p>
<p><strong>왜 중요한가:</strong> x86에서 "어쩌다 잘 돌던" 락 없는(lock-free) 멀티스레드 코드가 <strong>ARM에서 깨질 수 있습니다.</strong> ARM은 명시적 <em>메모리 배리어</em>가 필요하기 때문입니다. 실제로 Apple은 Rosetta 2(x86→ARM 변환) 성능을 위해 <strong>하드웨어에 TSO 모드를 따로 넣었습니다</strong> — x86 바이너리를 ARM에서 정확히 돌리려면 strong ordering을 흉내 내야 했기 때문입니다.</p>
<p class="src">출처: Wikipedia "Memory ordering" — "x86-64 processors have the strongest memory order ... [ARM] weak". 2026-06-03.</p>
</div>

<h3>④ 이 모든 게 합쳐지는 곳 — 성능/와트(Performance per Watt)</h3>
<p>단순 디코드 + 많은 레지스터 + load-store는 <strong>같은 일을 더 적은 트랜지스터·전력으로</strong> 하게 만듭니다 — 이것이 통념상 ARM의 최대 경쟁력입니다. <strong>(단, "ISA가 효율의 원천"이라는 이 통념에는 강한 학술적 반론이 있습니다 — 아래 ⑨ 필독.)</strong></p>
<ul>
<li><strong>모바일을 평정한 이유</strong>: 배터리 = 전력 효율이 전부. ARM의 DNA가 여기서 자랐습니다.</li>
<li><strong>Apple M 시리즈 충격</strong>: 5nm(TSMC) + 8-wide 디코드로, 노트북에서 데스크톱급 성능을 <strong>팬 없이/저발열</strong>로 냄. "성능/와트"가 x86 노트북을 압도.</li>
<li><strong>데이터센터 전환의 진짜 이유</strong>: 서버는 전기요금·냉각비가 TCO의 큰 축. AWS Graviton이 같은 성능을 더 낮은 전력으로 내면 곧 비용 절감. 이게 하이퍼스케일러가 ARM 자체 서버칩을 만드는 기술적 동기입니다.</li>
</ul>

<h3>⑤ ARM 아키텍처의 장 / 단점 (기술적 근거 기반)</h3>
<div class="twocol">
<div class="panel">
<p><strong>✅ 강점</strong></p>
<ul>
<li>성능/와트 우위 (고정 디코드·load-store·많은 레지스터의 합)</li>
<li>디코더 병렬화로 높은 IPC 천장 (Apple이 입증)</li>
<li>모듈식·확장식 ISA (SVE 등 — 코드가 하드웨어 벡터 폭에 자동 적응)</li>
<li>모바일에서 수십 년 축적된 성숙한 소프트웨어 생태계 + 거대한 출하 기반</li>
</ul>
</div>
<div class="panel">
<p><strong>⚠️ 약점·도전</strong></p>
<ul>
<li><strong>weak memory model</strong> → 기존 x86 코드 이식 시 미묘한 동시성 버그. 변환·검증 비용.</li>
<li>x86의 <strong>레거시 하위호환</strong>(수십 년 윈도/엔터프라이즈 SW)을 단번에 못 따라감 — 데스크톱/엔터프라이즈 전환이 느린 이유.</li>
<li>고성능 코어 설계는 여전히 어려움 — ARM ISA를 써도 Apple급 코어를 아무나 못 만듦(설계 역량이 변수).</li>
<li><strong>같은 RISC 장점을 RISC-V가 무료로 제공</strong> → ISA 자체의 차별성이 비용 앞에서 약화 (앞 6장 참조).</li>
</ul>
</div>
</div>

<h3>⑥ 기술 → 경쟁력 → 전망 연결</h3>
<div class="panel">
<p><strong>기술적 해자의 본질:</strong> ARM의 경쟁력은 "ISA가 마법이라서"가 아니라 <strong>① 성능/와트 우위 + ② 거대한 생태계·출하 기반의 전환비용</strong>입니다. ISA 설계 우위(고정 디코드 등)는 RISC-V도 RISC라 상당 부분 공유하므로, <strong>장기 해자는 ISA 자체보다 생태계</strong>에 있습니다.</p>
<p><strong>전망 (기술 관점):</strong></p>
<ul>
<li><strong>순풍</strong>: AI 데이터센터·PC가 전력 효율을 점점 중시 → ARM의 성능/와트 DNA가 모바일을 넘어 서버·PC로 확장 (Graviton, RTX Spark가 증거).</li>
<li><strong>역풍</strong>: RISC-V가 "같은 RISC 이점 + 무료 + 지정학 중립"으로 하단(임베디드)부터 잠식. 고성능 영역은 ARM 생태계가 당분간 방어하나, 시간은 오픈 진영 편일 수 있음.</li>
<li><strong>핵심 변수</strong>: ARM이 ISA 라이선서를 넘어 CSS(통합 설계)로 부가가치를 올리려 할수록 고객(Qualcomm 등)과 충돌 — 기술 우위를 비즈니스로 전환하는 과정에서 생태계 신뢰를 잃을 위험.</li>
</ul>
</div>

<h3>⑦ ARM이 직접 파는 코어 제품군 — Cortex와 Neoverse</h3>
<p>지금까지 "ARM ISA"를 얘기했지만, ARM은 ISA뿐 아니라 <strong>완성된 코어 설계(Cortex/Neoverse)</strong>도 팝니다(앞 3장의 "코어 IP 라이선스"). Apple처럼 ISA만 받아 자체 설계하는 곳을 빼면, 대부분의 칩은 이 코어들을 가져다 씁니다. 제품군이 시장을 그대로 반영합니다:</p>
<div class="table-scroll"><table>
<thead><tr><th>제품군</th><th>용도</th><th>대표 코어 (세대)</th><th>탑재 예</th></tr></thead>
<tbody>
<tr><td><strong>Cortex-X</strong></td><td>모바일 <em>최고성능</em> 코어</td><td>X1→X4→<strong>X925</strong>(2024~)</td><td>스냅드래곤, MediaTek Dimensity, Google Tensor, <strong>Nvidia GB10(RTX Spark)</strong></td></tr>
<tr><td><strong>Cortex-A</strong></td><td>모바일 <em>효율</em> 코어 (big.LITTLE의 LITTLE)</td><td>A520 / A720 / A725(2024~25)</td><td>거의 모든 안드로이드 AP</td></tr>
<tr><td><strong>Neoverse V</strong></td><td>서버 <em>최고성능</em> (Cortex-X 파생)</td><td>V1(Zeus)→V2(Demeter)→V3(Poseidon)</td><td>AWS Graviton3(V1), Graviton4·Nvidia Grace·Google Axion(V2)</td></tr>
<tr><td><strong>Neoverse N</strong></td><td>서버 <em>균형</em>(성능/전력/면적)</td><td>N1→N2</td><td>AWS Graviton2(N1), MS Azure Cobalt 100(N2), Alibaba Yitian 710</td></tr>
<tr><td><strong>Neoverse E</strong></td><td>엣지·네트워킹 (저전력 처리량)</td><td>E1→E2</td><td>5G 인프라, 엣지 장비</td></tr>
</tbody>
</table></div>
<p class="src">출처: Wikipedia "ARM Neoverse"(V1=Graviton3, V2=Graviton4/Grace/Axion, N1=Graviton2, N2=Cobalt 100/Yitian 710; V/N/E 시리즈 구분), "ARM Cortex-A"(Cortex-X1~X925, A520/A720/A725 세대), 2026-06-03 확인.</p>
<div class="callout">
<strong>연결 고리:</strong> 앞 6.6장에서 본 Nvidia "RTX Spark(N1X)" PC칩이 바로 ARM <strong>Cortex-X925</strong> 기반입니다 — ARM이 <em>직접 설계한</em> 최고성능 모바일 코어가 Nvidia를 통해 PC로 올라온 것. 즉 ARM은 "ISA만 빌려주는 회사"가 아니라 "완성 코어를 파는 회사"로서도 PC·서버 확장의 직접 수혜자입니다(코어 IP는 ISA 라이선스보다 로열티 단가가 높음, 앞 3장).
</div>

<h3>⑧ 정량 벤치마크 — 숫자로 보는 경쟁력</h3>
<div class="table-scroll"><table>
<thead><tr><th>비교</th><th>수치</th><th>출처</th></tr></thead>
<tbody>
<tr><td>AWS Graviton vs x86 (가성비)</td><td>comparable x86 대비 <strong>최대 ~40% 가성비 우위</strong>, 비용 <strong>~20% 저렴</strong> (Graviton3/4 일반화)</td><td>AWS 공식 (ec2/graviton)</td></tr>
<tr><td>Graviton4 vs Intel Xeon 5세대</td><td>compute-heavy 워크로드 <strong>최대 40% 낮은 TCO</strong></td><td>AWS 공식</td></tr>
<tr><td>Intel Lunar Lake vs ARM(노트북)</td><td>Intel은 "Arm 능가 배터리" 주장(최대 20.1h). 독립 테스트는 "<strong>근접하나 Apple/Qualcomm을 아직 못 넘음</strong>"</td><td>Tom's Hardware / XDA / PCWorld, 2024~25</td></tr>
<tr><td>Apple M vs x86 (효율 본질)</td><td>피크 전력은 비슷해도 M은 "<strong>race to idle</strong>"(빨리 끝내고 잠듦)로 총 에너지↓ → 실사용 배터리 우위</td><td>독립 리뷰 (XDA 등)</td></tr>
<tr><td>효율 정량 (Cinebench 2024 단일코어 points/W)</td><td>Apple <strong>M4 Pro 11.1</strong> vs Intel Core Ultra 258V <strong>5.36</strong> vs AMD Ryzen AI HX370 <strong>3.64</strong> → M이 약 <strong>2배</strong></td><td>Notebookcheck, 2024~26</td></tr>
<tr><td>멀티코어 효율 (points/W)</td><td>Apple M3 <strong>28.3</strong> &gt; Intel 258V 17.7~19.3 ≈ AMD HX370 19.7. <strong>단, AMD를 15W로 제한하면 25.2</strong>로 역전 — "전력 타겟이 ISA보다 중요"의 실증</td><td>Notebookcheck</td></tr>
</tbody>
</table></div>
<p class="src">출처: <a href="https://aws.amazon.com/ec2/graviton/">AWS EC2 Graviton</a> (Graviton3 = Graviton2 대비 +25%, Graviton4 = +30%); <a href="https://www.tomshardware.com/pc-components/cpus/intel-launches-lunar-lake-claims-arm-beating-battery-life-worlds-fastest-mobile-cpu-cores">Tom's Hardware (Lunar Lake)</a>; <a href="https://www.notebookcheck.net/Intel-Lunar-Lake-CPU-analysis-The-Core-Ultra-7-258V-s-multi-core-performance-is-disappointing-but-its-everyday-efficiency-is-good.893405.0.html">Notebookcheck (Lunar Lake 효율 분석)</a>. Claude WebSearch + Codex web-research 교차 확인, 2026-06-03.</p>

<h3>⑨ ⚠️ 강력한 반론 — "ISA는 효율과 거의 무관하다"</h3>
<div class="callout warn">
<p>앞 ①~⑥에서 "고정 디코드·load-store 같은 <strong>ISA 설계</strong>가 ARM 효율의 원천"이라 했습니다. 그런데 이를 <strong>정면으로 반박하는 신뢰도 높은 연구</strong>가 있고, 학습 자료의 정직성을 위해 명시합니다.</p>
<p><strong>2013 Wisconsin "Power Struggles" 연구</strong>(MIPS·ARM·x86 7개 플랫폼 실측)의 결론: <strong>"ISA가 RISC냐 CISC냐는 사실상 무관하다. ARM과 x86의 전력·성능 차이는 ISA가 아니라 <em>서로 다른 설계 목표·마이크로아키텍처·공정 노드</em>에서 온다. 어느 ISA도 본질적으로 더 효율적이지 않다."</strong> 반도체 분석 매체 Chips and Cheese도 "ARM or x86? ISA Doesn't Matter"로 같은 결론.</p>
<p>업계 당사자도 같은 말을 합니다 — Intel 기술마케팅 VP Robert Hallock(2024): "ARM vs x86 전력 논쟁은 <strong>'잘못된 전제(false premise)'</strong>다. ISA가 전력을 좌우하지 않는다."</p>
<p class="src">출처: <a href="https://research.cs.wisc.edu/vertical/papers/2013/hpca13-isa-power-struggles.pdf">Wisconsin HPCA 2013 "Power Struggles"</a>; <a href="https://chipsandcheese.com/p/arm-or-x86-isa-doesnt-matter">Chips and Cheese "ISA Doesn't Matter"</a>; Intel VP Hallock 발언(inkl, 2024). Claude WebSearch + Codex web-research 교차 확인, 2026-06-03.</p>
</div>

<div class="panel">
<p><strong>그래서 진실은 어디에? (종합 판단)</strong></p>
<ul>
<li>ISA 설계(고정 디코드 등)는 <strong>디코더 구현을 쉽게 해주는 이점은 실재</strong>하지만, 그것 하나로 효율 격차 전체를 설명하지 못함. 학계 컨센서스는 "ISA 기여는 작고, <strong>공정(TSMC 우위) + 마이크로아키텍처 + 설계 투자</strong>가 지배적"입니다.</li>
<li><strong>Apple M의 충격은 ISA가 아니라 "Apple의 설계 역량 + TSMC 최신 공정 독점"의 합작</strong>이라는 해석이 더 정확합니다. 같은 ARM ISA를 써도 평범한 칩은 평범합니다(앞 ⑤ "고성능 코어 설계는 어렵다"와 연결).</li>
<li><strong>Intel Lunar Lake가 ARM에 근접</strong>한 것이 이 반론의 실증입니다 — x86이라는 "불리한 ISA"로도 최신 공정·설계를 쏟으면 ARM급 효율에 거의 도달했습니다. 더 결정적으로, <strong>AMD Ryzen을 15W로 제한하면 효율이 25.2 points/W로 뛰어 Intel을 역전</strong>합니다(⑧ 표) — 같은 x86 칩의 효율이 전력 타겟만으로 크게 변한다는 것은, 효율을 좌우하는 게 ISA가 아니라 <strong>설정·설계 포인트</strong>임을 보여줍니다.</li>
</ul>
<p><strong>투자 관점의 함의 (중요):</strong> "ARM = ISA가 본질적으로 우월"이 약화되면, ARM의 진짜 해자는 <strong>ISA 우위가 아니라 ⑥에서 말한 "생태계 + 출하 기반"</strong>으로 좁혀집니다. 동시에 이 반론은 <strong>RISC-V에도 같은 논리</strong>로 적용됩니다 — RISC-V가 "RISC라서 좋다"가 아니라, 결국 누가 좋은 코어를 설계하느냐의 싸움. 즉 ARM의 위협도 기회도 ISA 자체보다 "생태계·설계 역량·공정 접근"에서 갈립니다.</p>
</div>

<h2 id="s3">3. ARM의 비즈니스 모델 — 라이선스 vs 로열티</h2>

<p>ARM Holdings는 <strong>칩을 단 한 개도 만들지 않습니다.</strong> 설계 IP(지식재산권)를 팔 뿐입니다. 수익은 두 종류로 들어옵니다:</p>

<figure>
<svg viewBox="0 0 760 340" role="img" aria-label="ARM의 라이선스와 로열티 수익 구조">
  <rect x="280" y="20" width="200" height="56" rx="10" fill="rgba(0,184,212,0.15)" stroke="#00b8d4" stroke-width="2"/>
  <text x="380" y="44" fill="#00b8d4" font-size="16" text-anchor="middle" font-weight="700">ARM Holdings</text>
  <text x="380" y="64" fill="#9098a3" font-size="11" text-anchor="middle">ISA·코어 IP 설계도 소유</text>
  <rect x="60" y="140" width="300" height="150" rx="10" fill="#161b22" stroke="#30363d"/>
  <text x="210" y="166" fill="#d2a8ff" font-size="15" text-anchor="middle" font-weight="700">① 라이선스 (License)</text>
  <text x="210" y="192" fill="#e6edf3" font-size="12.5" text-anchor="middle">설계도 접근권 — 선불·고정</text>
  <text x="210" y="214" fill="#9098a3" font-size="11.5" text-anchor="middle">"이 설계도 볼 권리를 사겠다"</text>
  <text x="210" y="240" fill="#9098a3" font-size="11.5" text-anchor="middle">1회성 / 계약 기간 단위</text>
  <text x="210" y="262" fill="#7ee787" font-size="11.5" text-anchor="middle">규모는 작지만 진입 장벽 역할</text>
  <rect x="400" y="140" width="300" height="150" rx="10" fill="#161b22" stroke="#30363d"/>
  <text x="550" y="166" fill="#ffb454" font-size="15" text-anchor="middle" font-weight="700">② 로열티 (Royalty)</text>
  <text x="550" y="192" fill="#e6edf3" font-size="12.5" text-anchor="middle">칩 출하 시마다 — 종량·반복</text>
  <text x="550" y="214" fill="#9098a3" font-size="11.5" text-anchor="middle">"칩 팔릴 때마다 통행료 내라"</text>
  <text x="550" y="240" fill="#9098a3" font-size="11.5" text-anchor="middle">칩당 (가격의 약 1~2%, 통념)</text>
  <text x="550" y="262" fill="#7ee787" font-size="11.5" text-anchor="middle">★ 진짜 성장 엔진 (복리 누적)</text>
  <line x1="340" y1="76" x2="210" y2="140" stroke="#d2a8ff" stroke-width="1.5" marker-end="url(#b)"/>
  <line x1="420" y1="76" x2="550" y2="140" stroke="#ffb454" stroke-width="1.5" marker-end="url(#b)"/>
  <defs>
    <marker id="b" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto">
      <path d="M0,0 L8,4 L0,8 Z" fill="#9098a3"/>
    </marker>
  </defs>
  <text x="380" y="320" fill="#9098a3" font-size="12" text-anchor="middle">공장·재료비 없음 → 매출총이익률 97.5% (출처: stockanalysis.com, 2026-06-01)</text>
</svg>
<figcaption>라이선스는 "입장료", 로열티는 "통행료". 칩이 세상에 많이 깔릴수록 로열티가 복리로 쌓이는 구조.</figcaption>
</figure>

<h3>라이선스의 3가지 등급 (중요)</h3>
<p>같은 "ARM 라이선스"라도 무엇을 사느냐에 따라 로열티 단가가 다릅니다. 이게 칩 회사들의 전략을 가릅니다.</p>

<div class="table-scroll"><table>
<thead><tr><th>라이선스 유형</th><th>무엇을 사는가</th><th>자유도</th><th>로열티</th><th>대표 사용자</th></tr></thead>
<tbody>
<tr><td><strong>아키텍처(ISA) 라이선스</strong></td><td>명령어 규격만 — 코어는 직접 설계</td><td>최고</td><td>낮음(하지만 라이선스료 자체는 비쌈)</td><td>Apple, Qualcomm(Oryon), Nvidia</td></tr>
<tr><td><strong>코어 IP 라이선스</strong></td><td>ARM이 설계한 완제품 코어(Cortex)</td><td>중간</td><td>중간</td><td>대부분의 중소 칩 설계사</td></tr>
<tr><td><strong>CSS (Compute Subsystem)</strong></td><td>코어+캐시+인터커넥트 통합 블록</td><td>낮음</td><td><strong>높음</strong></td><td>최근 ARM이 미는 고부가 영역</td></tr>
</tbody>
</table></div>

<p class="src">출처: Wikipedia "Arm Holdings" — "architectural licence is more costly than a regular Arm core licence"; ARM이 칩 가격 기준에서 <strong>디바이스 가치 기준</strong> 로열티로 전환 중. 2026-06-02 확인.</p>

<div class="callout">
<strong>왜 Apple은 비싼 아키텍처 라이선스를 살까?</strong> 아키텍처 라이선스는 ISA만 받고 코어를 처음부터 직접 설계합니다. 라이선스료는 비싸지만 <em>칩당 로열티는 낮고</em>, 무엇보다 자기만의 최적화(Apple M 시리즈의 압도적 전력효율)가 가능합니다. 즉 "ARM 언어는 쓰되, 칩은 내 방식대로" 만드는 것 — 규모가 큰 회사일수록 이쪽이 유리합니다.
</div>

<h2 id="s4">4. 칩 회사 지도 — 누가 무엇을 따르나</h2>

<figure>
<svg viewBox="0 0 760 420" role="img" aria-label="칩 회사들이 어떤 ISA를 따르는지 분류">
  <rect x="20" y="20" width="230" height="380" rx="12" fill="rgba(0,184,212,0.06)" stroke="#00b8d4"/>
  <text x="135" y="48" fill="#00b8d4" font-size="17" text-anchor="middle" font-weight="700">ARM ISA</text>
  <text x="135" y="68" fill="#9098a3" font-size="11" text-anchor="middle">유료 라이선스</text>
  <g font-size="12.5" fill="#e6edf3">
    <rect x="38" y="84" width="194" height="42" rx="7" fill="#161b22" stroke="#30363d"/>
    <text x="48" y="103">Apple — M/A 시리즈</text><text x="48" y="119" fill="#9098a3" font-size="10.5">아키텍처 라이선스, 자체 코어</text>
    <rect x="38" y="134" width="194" height="42" rx="7" fill="#161b22" stroke="#30363d"/>
    <text x="48" y="153">AWS — Graviton</text><text x="48" y="169" fill="#9098a3" font-size="10.5">자체 서버칩, 클라우드 전용</text>
    <rect x="38" y="184" width="194" height="42" rx="7" fill="#161b22" stroke="#30363d"/>
    <text x="48" y="203">Qualcomm — 스냅드래곤</text><text x="48" y="219" fill="#9098a3" font-size="10.5">모바일 AP, Oryon 자체코어</text>
    <rect x="38" y="234" width="194" height="42" rx="7" fill="#161b22" stroke="#30363d"/>
    <text x="48" y="253">Nvidia — Grace, RTX Spark</text><text x="48" y="269" fill="#9098a3" font-size="10.5">서버 CPU·신규 PC칩(N1X)</text>
    <rect x="38" y="284" width="194" height="42" rx="7" fill="#161b22" stroke="#30363d"/>
    <text x="48" y="303">MS Cobalt / Google Axion</text><text x="48" y="319" fill="#9098a3" font-size="10.5">하이퍼스케일러 자체 서버칩</text>
    <rect x="38" y="334" width="194" height="50" rx="7" fill="#161b22" stroke="#30363d"/>
    <text x="48" y="353">삼성 Exynos · 미디어텍</text><text x="48" y="369" fill="#9098a3" font-size="10.5">모바일 AP (Cortex 코어)</text>
  </g>
  <rect x="265" y="20" width="230" height="380" rx="12" fill="rgba(247,129,102,0.06)" stroke="#f78166"/>
  <text x="380" y="48" fill="#f78166" font-size="17" text-anchor="middle" font-weight="700">x86 ISA</text>
  <text x="380" y="68" fill="#9098a3" font-size="11" text-anchor="middle">사실상 폐쇄 (2社)</text>
  <g font-size="12.5" fill="#e6edf3">
    <rect x="283" y="84" width="194" height="50" rx="7" fill="#161b22" stroke="#30363d"/>
    <text x="293" y="105">Intel</text><text x="293" y="122" fill="#9098a3" font-size="10.5">x86 원조 설계자, 직접 제조</text>
    <rect x="283" y="144" width="194" height="50" rx="7" fill="#161b22" stroke="#30363d"/>
    <text x="293" y="165">AMD</text><text x="293" y="182" fill="#9098a3" font-size="10.5">x86-64 확장 개발, 크로스 라이선스</text>
  </g>
  <text x="380" y="240" fill="#9098a3" font-size="11.5" text-anchor="middle" font-style="italic">신규 진입 거의 불가.</text>
  <text x="380" y="258" fill="#9098a3" font-size="11.5" text-anchor="middle" font-style="italic">두 회사가 서로의 특허를</text>
  <text x="380" y="276" fill="#9098a3" font-size="11.5" text-anchor="middle" font-style="italic">크로스 라이선스로 묶어</text>
  <text x="380" y="294" fill="#9098a3" font-size="11.5" text-anchor="middle" font-style="italic">제3자 진입을 차단.</text>
  <rect x="510" y="20" width="230" height="380" rx="12" fill="rgba(126,231,135,0.06)" stroke="#7ee787"/>
  <text x="625" y="48" fill="#7ee787" font-size="17" text-anchor="middle" font-weight="700">RISC-V</text>
  <text x="625" y="68" fill="#9098a3" font-size="11" text-anchor="middle">무료 · 오픈소스</text>
  <g font-size="12.5" fill="#e6edf3">
    <rect x="528" y="84" width="194" height="42" rx="7" fill="#161b22" stroke="#30363d"/>
    <text x="538" y="103">Tesla — FSD/Dojo 일부</text><text x="538" y="119" fill="#9098a3" font-size="10.5">자율주행·AI 학습 가속기</text>
    <rect x="528" y="134" width="194" height="42" rx="7" fill="#161b22" stroke="#30363d"/>
    <text x="538" y="153">Alibaba — T-Head 玄铁</text><text x="538" y="169" fill="#9098a3" font-size="10.5">중국 탈ARM 전략 핵심</text>
    <rect x="528" y="184" width="194" height="42" rx="7" fill="#161b22" stroke="#30363d"/>
    <text x="538" y="203">SiFive</text><text x="538" y="219" fill="#9098a3" font-size="10.5">RISC-V 코어 IP 전문</text>
    <rect x="528" y="234" width="194" height="42" rx="7" fill="#161b22" stroke="#30363d"/>
    <text x="538" y="253">Nvidia (GPU 내 MCU)</text><text x="538" y="269" fill="#9098a3" font-size="10.5">제어용 마이크로컨트롤러(대량)</text>
    <rect x="528" y="284" width="194" height="42" rx="7" fill="#161b22" stroke="#30363d"/>
    <text x="538" y="303">Western Digital, 구글</text><text x="538" y="319" fill="#9098a3" font-size="10.5">스토리지 컨트롤러 등</text>
    <rect x="528" y="334" width="194" height="50" rx="7" fill="#161b22" stroke="#30363d"/>
    <text x="538" y="353">Tenstorrent</text><text x="538" y="369" fill="#9098a3" font-size="10.5">AI 칩 스타트업 (짐 켈러)</text>
  </g>
</svg>
<figcaption>같은 회사가 여러 ISA를 동시에 쓰기도 한다(예: Nvidia는 ARM 기반 CPU + GPU 내부 제어엔 RISC-V). 출처: Wikipedia "RISC-V" 채택사 목록, "Apple silicon"(ARM architecture), 2026-06-02.</figcaption>
</figure>

<h3>Tesla와 Apple — 같은 "자체 칩"이지만 전략이 다르다</h3>
<div class="twocol">
<div class="panel">
<p><strong>Apple = ARM 위에서 최고를 만든다</strong></p>
<p>비싼 아키텍처 라이선스로 ARM ISA를 받되, 코어는 100% 자체 설계. M 시리즈의 전력효율은 여기서 나옵니다. ARM에 <em>로열티는 계속</em> 냅니다 — 단 "디바이스당"이 아니라 <strong>ARM 기반 칩 1개당</strong>입니다(보도상 칩당 $0.30 미만). Apple은 2040년 이후까지 가는 장기 계약을 ARM과 맺었습니다.</p>
<p class="src">교정(Codex 검증): "맥/아이폰 팔릴 때마다" → "칩당 로열티". 출처: MacRumors 2023-11-29(칩당 $0.30 미만), ARM IPO 공시.</p>
</div>
<div class="panel">
<p><strong>Tesla = 영역별로 ISA를 섞는다</strong></p>
<p>차량 자율주행(FSD) 칩과 Dojo AI 학습 가속기에서 ARM 코어와 RISC-V를 용도별로 혼용. 로열티 부담을 줄이고 설계 자유도를 극대화하려는 전형적 대형 설계사 전략입니다.</p>
</div>
</div>

<h2 id="s5">5. 로열티는 실제로 어떻게 흐르나</h2>

<p>구체적 예시로 따라가 봅시다. 당신이 안드로이드 폰을 한 대 산다고 할 때:</p>

<figure>
<svg viewBox="0 0 760 230" role="img" aria-label="스마트폰 한 대가 팔릴 때 로열티가 흐르는 경로">
  <rect x="20" y="90" width="130" height="56" rx="9" fill="#161b22" stroke="#7ee787"/>
  <text x="85" y="114" fill="#e6edf3" font-size="13" text-anchor="middle">소비자</text>
  <text x="85" y="132" fill="#9098a3" font-size="10.5" text-anchor="middle">폰 1대 구매</text>
  <rect x="200" y="90" width="130" height="56" rx="9" fill="#161b22" stroke="#30363d"/>
  <text x="265" y="108" fill="#e6edf3" font-size="12.5" text-anchor="middle">삼성/샤오미</text>
  <text x="265" y="125" fill="#9098a3" font-size="10.5" text-anchor="middle">폰 제조사(OEM)</text>
  <text x="265" y="139" fill="#9098a3" font-size="10.5" text-anchor="middle">스냅드래곤 칩 탑재</text>
  <rect x="380" y="90" width="130" height="56" rx="9" fill="#161b22" stroke="#f78166"/>
  <text x="445" y="108" fill="#e6edf3" font-size="12.5" text-anchor="middle">Qualcomm</text>
  <text x="445" y="125" fill="#9098a3" font-size="10.5" text-anchor="middle">칩 설계·판매</text>
  <text x="445" y="139" fill="#9098a3" font-size="10.5" text-anchor="middle">ARM ISA 사용</text>
  <rect x="560" y="90" width="180" height="56" rx="9" fill="rgba(0,184,212,0.15)" stroke="#00b8d4" stroke-width="2"/>
  <text x="650" y="114" fill="#00b8d4" font-size="13" text-anchor="middle" font-weight="700">ARM Holdings</text>
  <text x="650" y="132" fill="#9098a3" font-size="10.5" text-anchor="middle">칩당 로열티 수취</text>
  <line x1="150" y1="118" x2="200" y2="118" stroke="#9098a3" stroke-width="1.5" marker-end="url(#c)"/>
  <line x1="330" y1="118" x2="380" y2="118" stroke="#9098a3" stroke-width="1.5" marker-end="url(#c)"/>
  <line x1="510" y1="118" x2="560" y2="118" stroke="#ffb454" stroke-width="2" marker-end="url(#c)"/>
  <text x="535" y="80" fill="#ffb454" font-size="11" text-anchor="middle">통행료 약 1~2%</text>
  <text x="380" y="190" fill="#9098a3" font-size="12" text-anchor="middle">칩 한 개가 출하될 때마다 ARM은 단 한 번도 칩을 만지지 않고 로열티를 받는다.</text>
  <text x="380" y="210" fill="#7ee787" font-size="12" text-anchor="middle">→ 전 세계 연간 ~300억 개 ARM 칩 출하 = 거대한 복리 통행료 풀</text>
  <defs>
    <marker id="c" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto">
      <path d="M0,0 L8,4 L0,8 Z" fill="#9098a3"/>
    </marker>
  </defs>
</svg>
<figcaption>로열티는 가치사슬의 맨 위에서 칩 한 개당 떨어진다. 모바일을 넘어 서버·PC·차로 칩 종류가 늘수록 이 통행료 풀이 커진다.</figcaption>
</figure>

<div class="callout">
<strong>개발자 관점의 연결:</strong> 당신의 CI/CD에 <code>linux/arm64</code> 타겟이 추가되고, AWS에서 Graviton 인스턴스를 띄우고, 회사가 Apple Silicon 맥북을 지급할 때마다 — 그 끝에는 ARM의 로열티 미터기가 한 칸씩 돌아갑니다. <strong>"arm 빌드가 늘어나는 세상"이 곧 ARM의 매출 모수가 커지는 세상</strong>입니다. 이게 어제 본 PER 417을 시장이 정당화하려는 성장 논리의 본질입니다.
</div>

<h2 id="s6">6. RISC-V — 무료 오픈소스 ISA의 위협</h2>

<p>RISC-V(리스크-파이브)는 2010년 UC 버클리에서 시작된 <strong>완전 개방형 ISA</strong>입니다. ARM과 똑같은 RISC 철학을 따르지만 결정적 차이가 하나 있습니다 — <strong>로열티가 0</strong>입니다. 누구나 규격을 가져다 칩을 만들고 한 푼도 안 냅니다.</p>

<div class="table-scroll"><table>
<thead><tr><th></th><th><span class="tag arm">ARM</span></th><th><span class="tag riscv">RISC-V</span></th></tr></thead>
<tbody>
<tr><td>라이선스료 / 로열티</td><td>둘 다 있음</td><td><strong>둘 다 없음</strong></td></tr>
<tr><td>소프트웨어 생태계</td><td>성숙 (수십 년 축적)</td><td>발전 중 (격차 존재)</td></tr>
<tr><td>설계 자유도</td><td>ARM 규격 준수 의무</td><td>모듈식, 마음대로 확장 가능</td></tr>
<tr><td>지정학</td><td>영국/미국 영향권 — 수출 규제 노출</td><td><strong>중립 — 중국이 적극 채택</strong></td></tr>
<tr><td>현재 강점 영역</td><td>모바일·서버·PC 전방위</td><td>임베디드·MCU·AI 가속기·스토리지 컨트롤러</td></tr>
</tbody>
</table></div>

<div class="callout warn">
<strong>ARM에게 왜 위협인가 (Bear 논리):</strong>
<ul>
<li><strong>지정학적 탈출구:</strong> 미·중 갈등으로 중국이 ARM 라이선스 리스크를 느끼자, Alibaba(T-Head) 등이 RISC-V로 대거 이동. ARM 매출의 큰 축인 중국이 흔들립니다.</li>
<li><strong>비용 0의 유혹:</strong> 대형 설계 역량을 가진 회사(하이퍼스케일러, Tesla)는 "통행료 내느니 직접 RISC-V로"가 점점 현실적입니다.</li>
<li><strong>빌드 타겟의 신호:</strong> 개발 생태계에 <code>riscv64</code> 타겟이 늘기 시작하면 그게 ARM 해자(moat) 침식의 선행지표입니다.</li>
</ul>
</div>

<div class="callout">
<strong>그러나 — 반대 측면 (ARM 방어 논리):</strong> 소프트웨어 생태계는 하루아침에 안 옮겨집니다. 수십 년간 쌓인 ARM용 컴파일러·라이브러리·드라이버·검증된 코어가 거대한 전환비용(switching cost)을 만듭니다. RISC-V는 "성능이 덜 중요한 주변부(MCU·컨트롤러)"부터 먹어 들어오는 중이고, 고성능 모바일·서버 코어에서 ARM을 당장 대체하긴 어렵습니다. <strong>위협은 장기·구조적이지, 단기 실적 충격은 아닙니다.</strong>
</div>

<h2 id="s65">6.5 RISC-V 채택 현황 — 실데이터 추적 (2026-06-02)</h2>

<p>"위협이다"라는 말만으로는 부족합니다. 실제 데이터로 <strong>어디까지 왔는지</strong> 추적해 봅시다. 결론부터: RISC-V는 <strong>임베디드·MCU에서는 이미 주류, 데이터센터·AI 서버급으로 빠르게 올라오는 중, 고성능 모바일·PC에서는 아직 ARM 미위협</strong> 단계입니다.</p>

<h3>지표 1 — 생태계 멤버십 (가장 확실한 정량 데이터)</h3>
<div class="table-scroll"><table>
<thead><tr><th>지표</th><th>값</th><th>의미</th></tr></thead>
<tbody>
<tr><td>RISC-V International 회원사</td><td><strong>4,500+ (2025)</strong></td><td>2015년 출범 후 폭발적 성장</td></tr>
<tr><td>본부 소재</td><td><strong>스위스</strong> (2020 미국→스위스 이전)</td><td>★ 미 수출규제 회피 목적 — 지정학 중립성 확보</td></tr>
<tr><td>주요 멤버</td><td>Qualcomm, Nvidia, Google, Intel, MediaTek, Alibaba, SiFive 등</td><td>ARM의 고객이자 경쟁사가 동시에 RISC-V에도 베팅</td></tr>
</tbody>
</table></div>
<p class="src">출처: Wikipedia "RISC-V" — "more than 4,500 members as of 2025", 스위스 이전 "citing concerns over U.S. export controls". 2026-06-02 확인.</p>

<div class="callout warn">
<strong>주목할 신호 — 본부의 스위스 이전:</strong> RISC-V 재단이 미국에서 스위스로 본부를 옮긴 건 분명한 사실입니다. <strong>다만 그 동기는 논쟁적입니다</strong> — 복수 언론(Reuters 등)은 미 수출규제 우려를 보도했지만, RISC-V 공식 FAQ는 "특정 국가·기업·정부·사건 때문이 아니다"라고 명시적으로 반박합니다. 어느 쪽이든, 결과적으로 스위스의 중립성은 중국 등이 "미국이 끊을 수 없는 ISA"로 RISC-V를 택하는 환경을 만듭니다.
<br><span class="src">교정(Codex 검증): 단정 → 양론 병기. 출처: RISC-V FAQ(riscv.org/about/faq), Reuters 보도.</span>
</div>

<h3>지표 2 — 채택 영역별 성숙도 (실제 제품 기준)</h3>
<figure>
<svg viewBox="0 0 760 300" role="img" aria-label="RISC-V 채택 성숙도를 영역별로 표시한 막대 그래프">
  <text x="20" y="30" fill="#e6edf3" font-size="14" font-weight="600">RISC-V 침투 정도 (왼쪽=완료, 오른쪽=초기)</text>
  <g font-size="13" fill="#e6edf3">
    <text x="20" y="68">임베디드·MCU</text>
    <rect x="200" y="54" width="520" height="22" rx="4" fill="#1c2330"/>
    <rect x="200" y="54" width="490" height="22" rx="4" fill="#7ee787"/>
    <text x="455" y="70" fill="#0d1117" font-size="11" text-anchor="middle" font-weight="700">이미 주류 (ESP32, GD32V 등 대량 출하)</text>
    <text x="20" y="108">스토리지 컨트롤러</text>
    <rect x="200" y="94" width="520" height="22" rx="4" fill="#1c2330"/>
    <rect x="200" y="94" width="430" height="22" rx="4" fill="#7ee787"/>
    <text x="415" y="110" fill="#0d1117" font-size="11" text-anchor="middle" font-weight="700">광범위 (Western Digital 등)</text>
    <text x="20" y="148">AI 가속기</text>
    <rect x="200" y="134" width="520" height="22" rx="4" fill="#1c2330"/>
    <rect x="200" y="134" width="320" height="22" rx="4" fill="#ffb454"/>
    <text x="360" y="150" fill="#0d1117" font-size="11" text-anchor="middle" font-weight="700">빠르게 부상 (Tenstorrent, Tesla Dojo)</text>
    <text x="20" y="188">데이터센터 서버 CPU</text>
    <rect x="200" y="174" width="520" height="22" rx="4" fill="#1c2330"/>
    <rect x="200" y="174" width="250" height="22" rx="4" fill="#ffb454"/>
    <text x="325" y="190" fill="#0d1117" font-size="11" text-anchor="middle" font-weight="700">진입 단계 (Alibaba C930 64코어)</text>
    <text x="20" y="228">고성능 모바일 AP</text>
    <rect x="200" y="214" width="520" height="22" rx="4" fill="#1c2330"/>
    <rect x="200" y="214" width="90" height="22" rx="4" fill="#f78166"/>
    <text x="300" y="230" fill="#e6edf3" font-size="11" text-anchor="start" font-weight="600"> 초기 (ARM 우위 견고)</text>
    <text x="20" y="268">PC / 노트북</text>
    <rect x="200" y="254" width="520" height="22" rx="4" fill="#1c2330"/>
    <rect x="200" y="254" width="60" height="22" rx="4" fill="#f78166"/>
    <text x="270" y="270" fill="#e6edf3" font-size="11" text-anchor="start" font-weight="600"> 거의 없음 (생태계 미성숙)</text>
  </g>
</svg>
<figcaption>RISC-V는 "아래(임베디드)에서 위(고성능)로" 침투 중. ARM의 핵심 수익원인 모바일·PC는 아직 방어선 안. 출처: Wikipedia "RISC-V" Implementations 섹션 실제 제품 목록, 2026-06-02.</figcaption>
</figure>

<h3>지표 3 — 영역별 실제 제품 (정성 증거)</h3>
<div class="table-scroll"><table>
<thead><tr><th>영역</th><th>실제 RISC-V 제품 / 기업</th><th>ARM 위협도</th></tr></thead>
<tbody>
<tr><td>임베디드·IoT</td><td>Espressif ESP32-C(중국), GigaDevice GD32V MCU 시리즈(2019~)</td><td><span class="tag riscv">이미 잠식</span></td></tr>
<tr><td>AI 가속기</td><td>Tenstorrent(짐 켈러), Tesla Dojo/FSD 일부, 다수 AI 칩의 제어 코어</td><td><span class="tag riscv">빠른 부상</span></td></tr>
<tr><td>데이터센터 CPU</td><td>Alibaba T-Head <strong>Xuantie C930</strong>(서버급), <strong>VitalStone V100</strong>(64코어 서버칩), Ventana, SiFive P870</td><td><span class="tag x86">진입 시작</span></td></tr>
<tr><td>GPU 내부 제어</td><td>Nvidia가 자사 GPU의 마이크로컨트롤러로 RISC-V 채택(연 수십억 개 규모)</td><td><span class="tag riscv">조용히 대량</span></td></tr>
<tr><td>고성능 모바일/PC</td><td>아직 상용 주력 제품 부재</td><td><span class="tag arm">ARM 견고</span></td></tr>
</tbody>
</table></div>
<p class="src">출처: Wikipedia "RISC-V" — Alibaba DAMO Xuantie C930 "server-grade", VitalStone V100 "up to 64 RISC-V cores", Tenstorrent·SiFive P870·Ventana datacenter-class 코어 언급. 2026-06-02 확인.</p>

<h3>지표 4 — 출하량 전망</h3>
<div class="callout">
<p>독립 검증(Codex)으로 실제 1차 출처 수치를 확보했습니다:</p>
<ul>
<li><strong>2030년까지 RISC-V 칩 160억 개+ 전망</strong> (RISC-V International, 2023-11-07)</li>
<li><strong>2030년 RISC-V SoC 출하 침투율 22.3%</strong> (SHD Group 2024 요약 보고서)</li>
<li>최신 갱신치: <strong>2031년 210억 개+</strong> (SHD Group 2025)</li>
</ul>
<p>단, 핵심은 방향성입니다: <strong>총 출하량의 대부분은 여전히 저가 임베디드</strong>이고, ARM의 고부가 로열티 풀(모바일·서버 고성능 코어)을 직접 잠식하는 건 아직 초기입니다. <strong>"개수(160억 개)"와 "로열티 가치"를 반드시 구분</strong>해야 합니다 — 임베디드 160억 개의 로열티 합이 고성능 코어 소수의 로열티보다 작을 수 있습니다.</p>
<p class="src">출처: RISC-V International "More than 16 billion RISC-V chips forecasted by 2030"(2023-11-07), SHD Group RISC-V Market Analysis 2024 Abridged Report. Codex 웹검증, 2026-06-02.</p>
</div>

<h3>종합 — ARM 해자 침식 단계 진단</h3>
<div class="panel">
<ul>
<li><strong>이미 일어난 일:</strong> 저가 임베디드·MCU·스토리지 컨트롤러·GPU 내부 제어는 RISC-V가 상당 부분 가져감. 단 이 영역은 ARM에게도 로열티 단가가 낮은 주변부.</li>
<li><strong>지금 일어나는 일:</strong> 데이터센터 서버급(Alibaba 64코어)·AI 가속기로 올라오는 중. 여기가 ARM의 신성장 영역(서버 로열티)과 정면 충돌하는 전선.</li>
<li><strong>아직 안 일어난 일:</strong> 고성능 모바일 AP·PC. ARM의 핵심 캐시카우는 소프트웨어 생태계 전환비용 덕에 방어선 안.</li>
<li><strong>가속 변수:</strong> 중국. ARM 라이선스 지정학 리스크 때문에 Alibaba 등이 RISC-V로 국가적 이동 중 — ARM 중국 매출(전체의 큰 축)에 구조적 위협.</li>
</ul>
<p><strong>투자 결론 연결:</strong> RISC-V는 ARM의 <em>단기 실적</em>을 흔드는 요인은 아니지만, <em>장기 멀티플(PER 417)을 정당화하는 "ARM이 영원히 표준"이라는 전제</em>를 침식합니다. 특히 서버·중국 전선의 진행 속도가 ARM 강세 시나리오의 핵심 모니터링 포인트입니다.</p>
</div>

<h2 id="s66">6.6 중국 RISC-V 전선 — 심층 추적 (2026-06-02)</h2>

<p>RISC-V 위협 중에서도 <strong>중국 전선</strong>은 성격이 다릅니다. 다른 곳의 RISC-V 채택이 "비용 절감"이라면, 중국의 채택은 <strong>국가 안보·기술 주권 차원의 전략</strong>입니다. 시장 논리가 아니라 정치 논리로 움직이기 때문에 더 빠르고 더 비가역적입니다.</p>

<div class="callout warn">
<strong>한 줄 요약:</strong> 중국은 미국이 ARM·x86을 "언제든 끊을 수 있는 무기"로 보고, 끊을 수 없는 RISC-V로 <strong>국가 차원의 CPU 아키텍처 대이동</strong>을 진행 중입니다. ARM 매출의 큰 축인 중국이 구조적으로 이탈하는 시나리오입니다.
</div>

<h3>왜 중국인가 — 인과 사슬</h3>
<figure>
<svg viewBox="0 0 760 250" role="img" aria-label="중국이 RISC-V로 이동하는 인과 사슬">
  <rect x="20" y="30" width="170" height="64" rx="9" fill="#161b22" stroke="#f78166"/>
  <text x="105" y="56" fill="#f78166" font-size="13" text-anchor="middle" font-weight="600">미국 수출규제</text>
  <text x="105" y="74" fill="#9098a3" font-size="10.5" text-anchor="middle">첨단 칩·EDA·</text>
  <text x="105" y="88" fill="#9098a3" font-size="10.5" text-anchor="middle">ISA 접근 차단 위협</text>
  <rect x="230" y="30" width="170" height="64" rx="9" fill="#161b22" stroke="#ffb454"/>
  <text x="315" y="50" fill="#ffb454" font-size="13" text-anchor="middle" font-weight="600">ARM·x86 = 리스크</text>
  <text x="315" y="68" fill="#9098a3" font-size="10.5" text-anchor="middle">"미국이 통제 가능한</text>
  <text x="315" y="82" fill="#9098a3" font-size="10.5" text-anchor="middle">외국 IP에 의존 불가"</text>
  <rect x="440" y="30" width="170" height="64" rx="9" fill="#161b22" stroke="#7ee787"/>
  <text x="525" y="50" fill="#7ee787" font-size="13" text-anchor="middle" font-weight="600">RISC-V = 해법</text>
  <text x="525" y="68" fill="#9098a3" font-size="10.5" text-anchor="middle">오픈·무료·스위스 재단</text>
  <text x="525" y="82" fill="#9098a3" font-size="10.5" text-anchor="middle">→ 끊을 수 없음</text>
  <rect x="620" y="30" width="120" height="64" rx="9" fill="rgba(126,231,135,0.12)" stroke="#7ee787" stroke-width="2"/>
  <text x="680" y="52" fill="#7ee787" font-size="12" text-anchor="middle" font-weight="700">국가 차원</text>
  <text x="680" y="68" fill="#7ee787" font-size="12" text-anchor="middle" font-weight="700">대이동</text>
  <text x="680" y="84" fill="#9098a3" font-size="10" text-anchor="middle">정부 주도</text>
  <line x1="190" y1="62" x2="230" y2="62" stroke="#9098a3" stroke-width="1.5" marker-end="url(#d)"/>
  <line x1="400" y1="62" x2="440" y2="62" stroke="#9098a3" stroke-width="1.5" marker-end="url(#d)"/>
  <line x1="610" y1="62" x2="620" y2="62" stroke="#7ee787" stroke-width="2" marker-end="url(#d)"/>
  <text x="380" y="150" fill="#e6edf3" font-size="13" text-anchor="middle" font-weight="600">결정적 증거 2가지</text>
  <rect x="60" y="162" width="320" height="66" rx="8" fill="#1c2330" stroke="#30363d"/>
  <text x="220" y="183" fill="#e6edf3" font-size="11" text-anchor="middle">전략 방향: 서방 소유 ISA(ARM·x86)</text>
  <text x="220" y="199" fill="#e6edf3" font-size="11" text-anchor="middle">의존도 축소 → RISC-V 장려</text>
  <text x="220" y="217" fill="#9098a3" font-size="10" text-anchor="middle">("대부분 전환"은 단정 못함 — 정책 장려까지 확인)</text>
  <rect x="400" y="165" width="320" height="60" rx="8" fill="#1c2330" stroke="#30363d"/>
  <text x="560" y="188" fill="#e6edf3" font-size="11.5" text-anchor="middle">2025-03: 중국, RISC-V 칩 사용</text>
  <text x="560" y="206" fill="#e6edf3" font-size="11.5" text-anchor="middle">전국 촉진 정책 준비 (Reuters 단독)</text>
  <defs>
    <marker id="d" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto">
      <path d="M0,0 L8,4 L0,8 Z" fill="#9098a3"/>
    </marker>
  </defs>
</svg>
<figcaption>출처(Codex 교차검증): Reuters "China to publish policy to boost RISC-V chip use nationwide"(2025-03-04). ※ Wikipedia의 "CPU 아키텍처 대부분 전환" 표현은 과장으로 판정 — 검증된 사실은 "정책 장려·서방 ISA 의존 축소"까지. 2026-06-02.</figcaption>
</figure>

<h3>중국 RISC-V 기업·칩 지도 (실제 제품)</h3>
<div class="table-scroll"><table>
<thead><tr><th>기업 (소재)</th><th>대표 제품</th><th>영역</th><th>의미</th></tr></thead>
<tbody>
<tr><td><strong>Alibaba T-Head / DAMO Academy</strong> (항저우)</td><td>Xuantie <strong>C910</strong>(2019, 오픈소스) → C920 → <strong>C930</strong>(서버급 CPU <em>설계/IP</em>)</td><td>서버 CPU IP</td><td>중국 RISC-V의 기함. ※ C930은 양산 칩이 아니라 CPU 설계/IP — 실제 양산칩 예는 V100</td></tr>
<tr><td><strong>SpacemiT</strong> (항저우)</td><td>Key Stone K1(8코어 앱프로세서), <strong>VitalStone V100</strong>(64코어 서버칩)</td><td>서버·앱</td><td>OpenC910 기반 64코어 — 데이터센터 정조준</td></tr>
<tr><td><strong>GigaDevice</strong> (베이징)</td><td>GD32V 시리즈 MCU (RV32IMAC, 2019~)</td><td>임베디드</td><td>중국 MCU 시장에서 이미 대량 출하</td></tr>
<tr><td><strong>Espressif</strong> (상하이)</td><td>ESP32-C3(RISC-V MCU), ESP32-S2(RISC-V 보조코어)</td><td>IoT/임베디드</td><td>전 세계 IoT에 깔린 중국산 RISC-V</td></tr>
<tr><td><strong>StarFive</strong></td><td>SiFive 코어 IP 중화권 독점 유통 → 자체 구현</td><td>전방위</td><td>중국 RISC-V 생태계 게이트웨이</td></tr>
</tbody>
</table></div>
<p class="src">출처: Wikipedia "RISC-V" Implementations 섹션 — Xuantie C910/C920/C930, SpacemiT VitalStone V100 "up to 64 RISC-V cores ... based on the OpenC910 project", GigaDevice GD32V, Espressif ESP32-C3/S2. 2026-06-02 확인.</p>

<h3>ARM 매출에 대한 함의</h3>
<div class="callout">
<p>ARM에게 중국은 단순한 한 개 시장이 아닙니다. <strong>독립 검증(Codex)으로 ARM FY2026 20-F(SEC)에서 실제 수치를 확보</strong>했습니다:</p>
<ul>
<li><strong>Arm China = 전체 매출의 ~16% (FY2026)</strong></li>
<li>PRC(중국 본토) 매출 = ~19% (FY2025) — 직접 매출 + Arm China 관계 매출 포함</li>
</ul>
<p>즉 "~20%"는 역사적으론 맞지만 <strong>최신 Arm China 집중도는 16%</strong>입니다. 게다가 ARM 중국 사업은 <strong>"Arm China"라는 별도 합작법인</strong>으로 운영돼 지배구조마저 복잡합니다(과거 경영권 분쟁 이력).</p>
<p>중국이 국가 차원에서 RISC-V로 이동하면 ARM은 ① 신규 라이선스 상실 ② 기존 로열티 풀 잠식 ③ 가장 빠르게 성장하던 시장의 구조적 이탈을 동시에 겪습니다 — 매출의 1/6이 걸린 전선입니다.</p>
<p class="src">출처: ARM FY2026 20-F (SEC EDGAR, 2026-05 제출). Codex 웹검증, 2026-06-02.</p>
</div>

<h3>균형 — 중국 RISC-V의 한계 (Devil's Advocate)</h3>
<div class="panel">
<ul>
<li><strong>제조 병목:</strong> RISC-V 설계는 자유롭지만, 그걸 첨단 공정(7nm 이하)으로 <em>제조</em>하려면 여전히 TSMC/ASML 생태계가 필요. 미국 제재는 ISA가 아니라 <strong>제조 단계</strong>에서 중국을 누르고 있음. RISC-V로 설계해도 최신 공정 생산은 별개 문제.</li>
<li><strong>소프트웨어 생태계:</strong> 서버·PC급에서 RISC-V 소프트웨어 성숙도는 ARM에 한참 못 미침. C930이 나와도 운영체제·컴파일러·라이브러리 최적화가 ARM 수준에 도달하려면 수년.</li>
<li><strong>성능 격차:</strong> 고성능 코어 설계 노하우는 단기간에 따라잡기 어려움. 현재 중국 RISC-V 서버칩은 ARM Neoverse·Apple급 성능과 격차.</li>
<li><strong>그러나 — 시간은 중국 편:</strong> 위 한계들은 "속도"의 문제지 "방향"의 문제가 아님. 국가가 보조금·내수시장·인력을 쏟아부으면 격차는 좁혀짐. ARM에게 중국 전선은 <em>지금의 실적</em>이 아니라 <em>5~10년 후 시장 구조</em>의 문제.</li>
</ul>
</div>

<h3>투자자 모니터링 포인트</h3>
<div class="callout warn">
중국 전선의 진행 속도를 가늠할 선행지표:
<ul>
<li>ARM 분기 실적의 <strong>중국 매출 비중 추이</strong> (현재 ~16% — 추가 감소 시작 = 이탈 가시화)</li>
<li>Alibaba C930 같은 <strong>서버급 RISC-V의 실제 데이터센터 배치</strong> 규모</li>
<li>중국 정부 RISC-V <strong>"가이던스"의 의무화 여부</strong> (권고 → 강제 전환 시 급가속)</li>
<li>개발 생태계에 <strong><code>riscv64</code> 타겟·중국발 커밋 증가</strong> (소프트웨어 성숙의 선행 신호)</li>
</ul>
</div>

<h2 id="s7">7. 종합 — 투자자 관점의 함의</h2>

<div class="panel">
<p><strong>ARM Holdings = "arm64로 빌드되는 세상의 통행료 징수자"</strong></p>
<ul>
<li><strong>강세 논리:</strong> 모바일(지배 완료) → 서버(Graviton/Cobalt/Axion) → PC(Nvidia N1X) → 차(자율주행)로 ARM 칩이 깔리는 영역이 계속 확장. 로열티는 칩 출하량에 비례해 복리로 누적. 매출총이익률 97.5%의 IP 통행료 모델.</li>
<li><strong>약세 논리:</strong> ① RISC-V가 무료·지정학 중립으로 해자를 장기 침식 ② ARM이 CSS로 고객 영역까지 침범하며 갈등(FTC 반독점 조사, 퀄컴 소송) ③ 무엇보다 <strong>PER 417·PS 76의 가격이 이 모든 확장을 이미 선반영</strong>.</li>
<li><strong>기술 트렌드는 진짜, 가격은 미래를 과도하게 당겨씀</strong> — 이게 앞선 분석의 "관망" 결론의 근거입니다. arm64 전환이라는 거대한 구조적 추세는 사실이지만, 주가는 그 추세가 <em>완벽하게</em> 실현될 미래를 현재가에 이미 반영해 두었습니다.</li>
</ul>
</div>

<h2 id="s8">8. 용어 사전</h2>
<div class="table-scroll"><table>
<tbody>
<tr><td><strong>ISA</strong></td><td>Instruction Set Architecture. CPU가 알아듣는 명령어 규격. ARM/x86/RISC-V가 각각 다른 ISA.</td></tr>
<tr><td><strong>arm64 / aarch64</strong></td><td>ARM의 64비트 ISA. 같은 것의 다른 이름(Apple은 arm64, 리눅스는 aarch64로 부름).</td></tr>
<tr><td><strong>x86-64 / amd64</strong></td><td>x86의 64비트 확장. AMD가 개발해서 amd64라고도 부름.</td></tr>
<tr><td><strong>RISC / CISC</strong></td><td>간결 명령어 / 복잡 명령어 설계 철학. ARM·RISC-V는 RISC, x86은 CISC.</td></tr>
<tr><td><strong>IP (코어)</strong></td><td>Intellectual Property. 여기선 ARM이 파는 CPU 설계 자산(Cortex 등).</td></tr>
<tr><td><strong>아키텍처 라이선스</strong></td><td>ISA 규격만 받고 코어는 직접 설계할 권리. 비싸지만 자유도 최고(Apple·Qualcomm·Nvidia).</td></tr>
<tr><td><strong>CSS</strong></td><td>Compute Subsystem. ARM이 코어+주변 블록을 통합 제공하는 고부가 패키지.</td></tr>
<tr><td><strong>로열티</strong></td><td>칩이 출하될 때마다 ISA 소유자에게 내는 칩당 사용료. ARM의 핵심 반복 매출.</td></tr>
<tr><td><strong>OEM</strong></td><td>Original Equipment Manufacturer. 칩을 사다가 완제품(폰·노트북)을 만드는 회사.</td></tr>
<tr><td><strong>해자 (Moat)</strong></td><td>경쟁사가 못 넘는 진입장벽. ARM의 해자는 성숙한 소프트웨어 생태계와 전환비용.</td></tr>
</tbody>
</table></div>

<hr>
<div class="callout">
<strong>검증 노트 (Claude 초안 → Codex 독립 팩트체크, 2026-06-02):</strong>
<p>이 자료는 처음 Wikipedia 단일 출처로 작성된 뒤, OpenAI Codex의 독립 웹검색으로 교차 검증·교정됐습니다. 주요 교정:</p>
<ul>
<li><strong>1차 출처로 확정:</strong> ARM 매출총이익 97.5%·중국 매출 16%(FY2026 20-F), RISC-V 멤버 4,500+(연차보고서), 출하 전망 160억 개@2030(RISC-V Int'l/SHD).</li>
<li><strong>교정된 오류:</strong> ARM 6/1 등락 +19%→<strong>종가 +15.73%</strong>(장중 고가 $421); Nvidia 칩 "N1X"→공식명 <strong>RTX Spark</strong>; Apple 로열티 "디바이스당"→<strong>칩당</strong>; x86 "진입 거의 불가"→과장(VIA 존재); 스위스 이전 동기→<strong>양론 병기</strong>; 중국 "CPU 대부분 전환"→과장(정책 장려까지); C930 "칩"→<strong>설계/IP</strong>.</li>
<li><strong>여전히 불확실:</strong> 로열티 "칩 가격의 1~2%"는 통념일 뿐 ARM 공시엔 보편 요율 없음. 2026년 시점 데이터(주가·RTX Spark)는 작성 환경과 검증 모델의 지식 시점 차로 일부 한계 존재.</li>
</ul>
</div>
<p class="src">학습 자료 작성: 2026-06-02 · 사실관계 출처: Wikipedia(RISC-V, Arm Holdings, x86-64, Apple silicon), stockanalysis.com(ARM 재무, 2026-06-01). 본 자료는 교육 목적이며 투자 권유가 아닙니다. 수치·전략은 작성 시점 기준이며 변동될 수 있습니다.</p>

</div>
