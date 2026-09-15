---
title: "칩 아키텍처(ISA) 생태계 정리"
subtitle: "arm64로 빌드하는 세상에서 ARM Holdings는 무엇으로 돈을 벌까"
tags:
    - hardware
    - investment
date: 2026-06-02
image: https://raw.githubusercontent.com/siner308/blog-posts/master/assets/chip-isa-ecosystem-thumbnail.png
---

이 글의 수치는 2026년 6월 2일 기준이고 투자 권유가 아닙니다. 초안은 Claude로 만들고 OpenAI Codex의 웹 검색으로 수치를 교차 확인한 학습 자료를 다시 정리한 것입니다.

# 서론

2026년 6월 1일에 ARM Holdings 주가가 하루에 15.73% 올랐습니다(장중 고가 421달러). PER 417이라는 숫자를 시장이 어떤 성장 논리로 정당화하는지 궁금했습니다. 매일 `GOARCH=arm64 go build`를 치고 AWS에서 Graviton 인스턴스를 띄우는데도 ARM이 정확히 무엇을 팔아서 돈을 버는지는 설명할 수 없었기 때문입니다.

그래서 빌드 타겟 `arm64`가 무엇을 가리키는지에서 시작해 ARM의 수익 구조와 RISC-V까지 따라가 봤습니다. 위키피디아로 먼저 훑고, 매출 비중 같은 숫자는 SEC 공시(ARM FY2026 20-F)와 Reuters 등 1차 출처로 다시 확인했습니다.

# ISA란 무엇인가

ISA(Instruction Set Architecture, 명령어 집합 구조)는 CPU가 알아듣는 명령어의 규격입니다. `ADD`, `LOAD`, `STORE` 같은 명령어가 비트 단위로 어떻게 인코딩되는지를 정하고, 소프트웨어(기계어)와 하드웨어(트랜지스터) 사이의 계약 역할을 합니다.

`GOARCH=arm64 go build`나 `docker build --platform linux/arm64`를 실행하면 컴파일러는 소스 코드를 ARM ISA 규격의 기계어로 번역합니다. 그 바이너리는 x86 CPU에서는 한 글자도 읽지 못합니다. 언어가 다르기 때문입니다. 빌드 타겟이 갈리는 이유가 이것입니다.

여기서 중요한 점은 ISA는 설계도이고 칩은 물건이라는 것입니다. 설계도를 소유하는 쪽과 그 설계도로 칩을 만드는 쪽이 분리될 수 있고, ARM의 사업은 전부 이 분리에서 시작합니다.

# 세 가지 ISA

| 구분 | x86 / x86-64 | ARM | RISC-V |
|------|--------------|-----|--------|
| 소유 | Intel과 AMD (크로스 라이선스) | ARM Holdings (Softbank 자회사) | 누구도 소유하지 않음. RISC-V International(비영리 재단)이 관리 |
| 설계 철학 | CISC | RISC | RISC, 모듈식 |
| 라이선스 | 사실상 폐쇄. Intel과 AMD 양강에 VIA 같은 군소 업체 | 유료 라이선스 | 무료, 오픈소스, 로열티 0 |
| 주력 영역 | 데스크톱, 노트북, 서버 | 모바일에서 서버, PC, 차로 확장 중 | 임베디드와 MCU에서 데이터센터, AI 가속기로 올라오는 중 |
| 대표 칩 | Intel Core, AMD Ryzen/EPYC | Apple M, AWS Graviton, 스냅드래곤 | SiFive, Alibaba T-Head, Tenstorrent |

CISC(x86)는 명령어 하나가 많은 일을 하고, RISC(ARM, RISC-V)는 단순한 명령어를 많이 씁니다. 모바일과 데이터센터가 전력 효율을 중시하면서 RISC 계열이 커졌습니다. x86-64 자체는 AMD가 1999년에서 2000년 사이에 만든 64비트 확장이라 amd64라고도 부릅니다. ([x86-64](https://en.wikipedia.org/wiki/X86-64))

# ARM이 효율적이라는 통념, 그리고 반론

처음에는 ARM의 전력 효율이 ISA 설계에서 온다고 정리했습니다. 근거는 이렇습니다.

- [AArch64](https://en.wikipedia.org/wiki/AArch64) 명령어는 전부 4바이트 고정 길이입니다. x86은 1바이트에서 15바이트까지 가변이라 앞 명령어를 풀어야 다음 명령어의 경계를 알 수 있습니다. 고정 길이면 디코더를 병렬로 넓히기 쉬워서 Apple Firestorm(M1) 코어는 8-wide 디코드를 했고, 고성능 x86은 보통 4에서 6-wide에 머물렀습니다.
- 범용 레지스터가 x86-64는 16개, AArch64는 31개입니다. 레지스터가 많으면 메모리 접근이 줄어듭니다.
- AArch64는 load-store 구조라 메모리는 load와 store로만 건드리고 연산은 레지스터에서만 합니다. 파이프라인이 단순해집니다.
- 벡터 연산은 x86이 SSE/AVX(고정 폭), ARM이 NEON과 SVE(가변 폭)입니다. SVE는 하드웨어 벡터 길이에 코드가 자동으로 맞춰집니다.

개발자가 직접 부딛히는 차이는 [메모리 일관성 모델](https://en.wikipedia.org/wiki/Memory_ordering)입니다. x86은 strong ordering(TSO)이라 멀티스레드에서 쓰기 순서가 대체로 보존되고, ARM은 weak ordering이라 CPU가 순서를 자유롭게 바꿉니다. x86에서 어쩌다 잘 돌던 lock-free 코드가 ARM에서 깨질 수 있는 이유입니다. Apple이 Rosetta 2로 x86 바이너리를 정확히 돌리기 위해 하드웨어에 TSO 모드를 따로 넣은 것도 이 때문입니다.

그런데 이 통념을 정면으로 반박하는 연구가 있었습니다. 2013년 위스콘신 대학의 ["Power Struggles"](https://research.cs.wisc.edu/vertical/papers/2013/hpca13-isa-power-struggles.pdf) 논문은 MIPS, ARM, x86 7개 플랫폼을 실측해서, ISA가 RISC냐 CISC냐는 전력과 성능에 사실상 무관하고 차이는 설계 목표와 마이크로아키텍처, 공정 노드에서 온다고 결론냈습니다. Chips and Cheese도 ["ARM or x86? ISA Doesn't Matter"](https://chipsandcheese.com/p/arm-or-x86-isa-doesnt-matter)에서 같은 결론을 냈고, Intel의 Robert Hallock은 2024년에 ARM 대 x86 전력 논쟁이 잘못된 전제라고 말했습니다(inkl, 2024).

숫자를 보면 반론 쪽이 설득력 있습니다.

| 비교 | 수치 | 출처 |
|------|------|------|
| Cinebench 2024 단일코어 points/W | Apple M4 Pro 11.1, Intel Core Ultra 258V 5.36, AMD Ryzen AI HX370 3.64 | [Notebookcheck](https://www.notebookcheck.net/Intel-Lunar-Lake-CPU-analysis-The-Core-Ultra-7-258V-s-multi-core-performance-is-disappointing-but-its-everyday-efficiency-is-good.893405.0.html) |
| 멀티코어 points/W | Apple M3 28.3, Intel 258V 17.7에서 19.3, AMD HX370 19.7. AMD를 15W로 제한하면 25.2 | [Notebookcheck](https://www.notebookcheck.net/Intel-Lunar-Lake-CPU-analysis-The-Core-Ultra-7-258V-s-multi-core-performance-is-disappointing-but-its-everyday-efficiency-is-good.893405.0.html) |
| AWS Graviton 대 x86 | 비슷한 x86 인스턴스 대비 가성비 최대 40% 우위, 비용 약 20% 저렴 | [AWS](https://aws.amazon.com/ec2/graviton/) |
| Graviton4 대 Intel Xeon 5세대 | compute-heavy 워크로드에서 최대 40% 낮은 TCO | [AWS](https://aws.amazon.com/ec2/graviton/) |
| Intel Lunar Lake | Intel은 ARM을 넘는 배터리(최대 20.1시간)를 주장. 독립 테스트는 근접했지만 Apple과 Qualcomm을 아직 못 넘었다고 평가 | [Tom's Hardware](https://www.tomshardware.com/pc-components/cpus/intel-launches-lunar-lake-claims-arm-beating-battery-life-worlds-fastest-mobile-cpu-cores), XDA, PCWorld |

같은 AMD 칩을 15W로 제한했을 때 효율이 Intel을 넘어선 것이 결정적이었습니다. 효율을 좌우하는 것이 ISA가 아니라 전력 타겟과 설계 포인트라는 뜻입니다. Apple M의 효율도 ISA보다는 Apple의 설계 역량과 TSMC 최신 공정을 먼저 쓸 수 있었던 점의 합작으로 보는 것이 더 정확해 보입니다. 같은 ARM ISA를 써도 평범한 칩은 평범합니다.

그래서 ARM의 경쟁력은 ISA 우위보다 생태계와 출하 기반이 만드는 전환 비용에 있다고 결론을 고쳤습니다. 이 결론은 뒤의 RISC-V 이야기에도 그대로 이어집니다. RISC-V가 RISC라서 좋은 것이 아니라, 결국 누가 좋은 코어를 설계하느냐의 싸움입니다.

# ARM의 비즈니스 모델

[ARM Holdings](https://en.wikipedia.org/wiki/Arm_Holdings)는 칩을 한 개도 만들지 않습니다. 설계 IP를 팔고 두 종류의 돈을 받습니다.

1. 라이선스. 설계도에 접근할 권리를 선불로 삽니다. 1회성이거나 계약 기간 단위고, 규모는 작지만 진입 장벽 역할을 합니다.
2. 로열티. 칩이 출하될 때마다 칩당 받습니다. 통념상 칩 가격의 1에서 2%라고 하는데, ARM 공시에는 보편 요율이 없어서 확인하지 못했습니다.

공장도 재료비도 없으니 매출총이익률이 97.5%입니다(stockanalysis.com, 2026-06-01). 칩이 세상에 많이 깔릴수록 로열티가 쌓이는 구조라서 성장 엔진은 라이선스가 아니라 로열티입니다.

같은 ARM 라이선스라도 무엇을 사느냐가 다릅니다.

| 라이선스 | 무엇을 사는가 | 자유도 | 로열티 | 대표 사용자 |
|----------|---------------|--------|--------|-------------|
| 아키텍처(ISA) 라이선스 | 명령어 규격만. 코어는 직접 설계 | 최고 | 낮음. 대신 라이선스료가 비쌈 | Apple, Qualcomm(Oryon), Nvidia |
| 코어 IP 라이선스 | ARM이 설계한 완성 코어(Cortex, Neoverse) | 중간 | 중간 | 대부분의 칩 설계사 |
| CSS(Compute Subsystem) | 코어와 캐시, 인터커넥트를 묶은 블록 | 낮음 | 높음 | 최근 ARM이 미는 영역 |

Apple이 비싼 아키텍처 라이선스를 사는 이유가 여기 있습니다. 라이선스료는 비싸도 칩당 로열티는 낮고, 코어를 처음부터 자기 방식으로 설계할 수 있습니다. 보도에 따르면 Apple이 내는 로열티는 칩당 0.30달러 미만이고, 2040년 이후까지 가는 장기 계약을 맺었다고 합니다(MacRumors, 2023-11-29). 처음엔 "맥이나 아이폰이 팔릴 때마다"라고 적었는데 확인해보니 디바이스당이 아니라 칩당이었습니다.

ARM이 직접 설계해서 파는 코어 제품군은 시장을 그대로 반영합니다. ([ARM Cortex-A](https://en.wikipedia.org/wiki/ARM_Cortex-A), [ARM Neoverse](https://en.wikipedia.org/wiki/ARM_Neoverse))

| 제품군 | 용도 | 대표 코어 | 탑재 예 |
|--------|------|-----------|---------|
| Cortex-X | 모바일 최고 성능 | X1에서 X4, X925(2024~) | 스냅드래곤, MediaTek Dimensity, Google Tensor, Nvidia GB10(RTX Spark) |
| Cortex-A | 모바일 효율 코어 | A520, A720, A725 | 거의 모든 안드로이드 AP |
| Neoverse V | 서버 최고 성능 | V1(Zeus), V2(Demeter), V3(Poseidon) | Graviton3(V1), Graviton4와 Nvidia Grace, Google Axion(V2) |
| Neoverse N | 서버 균형형 | N1, N2 | Graviton2(N1), Azure Cobalt 100(N2), Alibaba Yitian 710 |
| Neoverse E | 엣지, 네트워킹 | E1, E2 | 5G 인프라 |

Nvidia의 PC 칩 RTX Spark(N1X)가 Cortex-X925 기반입니다. ARM이 직접 설계한 모바일 최고 성능 코어가 Nvidia를 통해 PC로 올라온 것이고, 코어 IP는 ISA 라이선스보다 로열티 단가가 높아서 ARM에게는 더 좋은 장사입니다.

# 누가 어떤 ISA를 쓰나

- ARM: Apple(M과 A 시리즈. 아키텍처 라이선스로 자체 코어), AWS Graviton, Qualcomm 스냅드래곤(Oryon 자체 코어), Nvidia Grace와 RTX Spark, Microsoft Cobalt, Google Axion, 삼성 Exynos, MediaTek
- x86: Intel과 AMD. 두 회사가 서로의 특허를 크로스 라이선스로 묶고 있어서 제3자 진입이 사실상 막혀 있습니다. (VIA처럼 군소 업체가 있긴 합니다)
- RISC-V: Tesla(FSD와 Dojo 일부), Alibaba T-Head, SiFive, Nvidia(GPU 안의 제어용 MCU), Western Digital, Google(스토리지 컨트롤러 등), Tenstorrent

같은 회사가 여러 ISA를 동시에 씁니다. Nvidia는 CPU는 ARM으로 만들고 GPU 내부 제어에는 RISC-V를 씁니다. Tesla도 FSD 칩과 Dojo에서 ARM 코어와 RISC-V를 용도별로 섞습니다. 설계 역량이 큰 회사가 로열티 부담을 줄이면서 설계 자유도를 챙기는 방식으로 보입니다.

# 로열티는 어떻게 흐르나

안드로이드 폰 한 대를 산다고 가정해 봅시다.

1. 소비자가 삼성이나 샤오미 폰을 삽니다.
2. 폰 제조사는 Qualcomm에서 스냅드래곤 칩을 사서 넣었습니다.
3. Qualcomm은 그 칩을 ARM ISA로 설계했으니 칩 한 개당 ARM에 로열티를 냅니다.

ARM은 칩을 한 번도 만지지 않고 가치 사슬 맨 위에서 칩당 돈을 받습니다. 전 세계에서 연간 약 300억 개의 ARM 칩이 출하되니 이 풀은 큽니다. CI에 `linux/arm64` 타겟을 추가하고, Graviton 인스턴스를 띄우고, 회사가 Apple Silicon 맥북을 지급할 때마다 그 끝에서 ARM의 미터기가 한 칸씩 돕니다. arm 빌드가 늘어나는 세상이 곧 ARM 매출의 모수가 커지는 세상이라는 것. PER 417을 시장이 정당화하려는 논리가 이것입니다.

# RISC-V

[RISC-V](https://en.wikipedia.org/wiki/RISC-V)는 2010년 UC 버클리에서 시작된 개방형 ISA입니다. ARM과 같은 RISC 철학인데 누구나 규격을 가져다 칩을 만들어도 로열티를 내지 않습니다.

| | ARM | RISC-V |
|--|-----|--------|
| 라이선스료와 로열티 | 둘 다 있음 | 둘 다 없음 |
| 소프트웨어 생태계 | 성숙 | 발전 중 |
| 설계 자유도 | ARM 규격 준수 의무 | 모듈식, 자유롭게 확장 |
| 지정학 | 영국과 미국 영향권. 수출 규제에 노출 | 중립. 중국이 적극 채택 |
| 현재 강한 영역 | 모바일, 서버, PC | 임베디드, MCU, AI 가속기, 스토리지 컨트롤러 |

ARM에게 위협인 이유는 세 가지입니다. 미중 갈등으로 중국이 ARM 라이선스를 리스크로 보기 시작했고, 설계 역량이 있는 회사(하이퍼스케일러, Tesla)에게는 통행료를 내느니 RISC-V로 직접 만드는 쪽이 점점 현실적이고, 개발 생태계에 `riscv64` 타겟이 늘기 시작하면 그것이 ARM 해자가 깎이는 선행 지표가 됩니다.

반대로 소프트웨어 생태계는 하루아침에 옮겨지지 않습니다. 수십 년 쌓인 ARM용 컴파일러, 라이브러리, 드라이버, 검증된 코어가 전환 비용을 만듭니다. RISC-V는 성능이 덜 중요한 주변부부터 먹어 들어오는 중이고 고성능 모바일과 서버 코어에서 ARM을 당장 대체하기는 어렵습니다. 장기적이고 구조적인 위협이지 단기 실적 충격은 아니라고 봅니다.

## 어디까지 왔나

- RISC-V International 회원사는 2025년 기준 4,500곳이 넘습니다. Qualcomm, Nvidia, Google, Intel, MediaTek, Alibaba, SiFive처럼 ARM의 고객이자 경쟁사가 동시에 RISC-V에도 걸어두고 있습니다.
- 재단 본부는 2020년 미국에서 스위스로 옮겼습니다. Reuters 등은 미국 수출 규제 우려를 동기로 보도했지만 [RISC-V 공식 FAQ](https://riscv.org/about/faq/)는 특정 국가나 사건 때문이 아니라고 반박합니다. 어느 쪽이든 결과적으로 중국이 "미국이 끊을 수 없는 ISA"로 RISC-V를 택하기 좋은 환경이 됐습니다.
- 임베디드와 MCU는 이미 주류입니다(Espressif ESP32-C3, GigaDevice GD32V). 스토리지 컨트롤러도 Western Digital 등이 널리 씁니다. AI 가속기는 Tenstorrent와 Tesla Dojo로 빠르게 올라오고, 데이터센터 CPU는 Alibaba T-Head의 Xuantie C930(서버급 설계 IP), SpacemiT의 VitalStone V100(64코어 서버칩), Ventana, SiFive P870이 진입 단계입니다. 고성능 모바일 AP와 PC는 아직 상용 주력 제품이 없습니다.
- 출하 전망은 2030년까지 160억 개 이상(RISC-V International, 2023-11-07), 2030년 SoC 침투율 22.3%(SHD Group 2024), 2031년 210억 개 이상(SHD Group 2025)입니다.

숫자를 볼 때 개수와 로열티 가치를 구분해야 합니다. 총 출하량 대부분은 저가 임베디드라서, 임베디드 160억 개의 로열티 합이 고성능 코어 소수의 로열티보다 작을 수 있습니다. ARM의 고부가 로열티 풀(모바일과 서버의 고성능 코어)을 직접 잠식하는 단계는 아직 초기입니다.

## 중국

중국의 RISC-V 채택은 비용 절감이 아니라 기술 주권의 문제라서 성격이 다릅니다. 미국 수출 규제가 첨단 칩과 EDA, ISA 접근을 끊을 수 있다는 위협이 되자 ARM과 x86이 리스크가 됐고, 오픈이고 무료이며 스위스 재단이 관리하는 RISC-V가 해법이 됐습니다. 2025년 3월에는 중국 정부가 RISC-V 칩 사용을 전국적으로 촉진하는 정책을 준비 중이라는 Reuters 단독 보도가 있었습니다. 위키피디아에는 "CPU 아키텍처 대부분 전환"이라는 표현이 있는데, 확인된 사실은 정책 장려와 서방 ISA 의존 축소까지라서 그 표현은 과장으로 봤습니다.

중국 RISC-V 기업으로는 Alibaba T-Head(Xuantie C910, C920, C930), SpacemiT(Key Stone K1, VitalStone V100), GigaDevice(GD32V MCU), Espressif(ESP32-C3), StarFive(SiFive 코어 IP의 중화권 유통)가 있습니다. C930은 양산 칩이 아니라 CPU 설계 IP이고, 실제 양산칩 예는 OpenC910 기반 64코어인 V100입니다.

ARM 매출에서 Arm China는 FY2026 기준 약 16%입니다(ARM FY2026 20-F). PRC 본토 매출은 FY2025 기준 약 19%였습니다. "20%"는 역사적으로는 맞지만 최신 Arm China 집중도는 16%입니다. 게다가 ARM 중국 사업은 Arm China라는 별도 합작법인이라 지배구조도 복잡합니다(과거에 경영권 분쟁이 있었습니다). 중국이 국가 차원에서 RISC-V로 가면 ARM은 신규 라이선스를 잃고, 기존 로열티 풀이 잠식되고, 가장 빠르게 성장하던 시장이 구조적으로 빠져나가는 것을 동시에 겪습니다. 매출의 1/6이 걸린 전선입니다.

물론 한계도 있습니다. RISC-V로 설계해도 7nm 이하 첨단 공정으로 제조하려면 TSMC와 ASML 생태계가 필요하고, 미국 제재는 ISA가 아니라 제조 단계에서 중국을 누르고 있습니다. 서버와 PC급 RISC-V 소프트웨어 성숙도는 ARM에 한참 못 미치고, 고성능 코어 설계 노하우도 단기간에 따라잡기 어렵습니다. 다만 이것은 속도의 문제지 방향의 문제는 아닌 것으로 보입니다. ARM에게 중국은 지금 실적이 아니라 5년에서 10년 뒤 시장 구조의 문제입니다.

# 정리

ARM Holdings는 arm64로 빌드되는 세상의 통행료 징수자입니다. 모바일을 평정했고 서버(Graviton, Cobalt, Axion)와 PC(RTX Spark), 차로 칩이 깔리는 영역이 계속 넓어지고 있어서 로열티는 출하량에 비례해 쌓입니다. 매출총이익률 97.5%의 IP 사업입니다.

물론 RISC-V가 무료와 지정학 중립을 무기로 해자를 장기적으로 깎고 있고, ARM이 CSS로 고객 영역까지 들어가면서 갈등(FTC 반독점 조사, Qualcomm 소송)도 생겼습니다. 무엇보다 PER 417, PS 76이라는 가격이 이 확장을 이미 다 반영하고 있습니다. 기술 트렌드는 진짜지만 가격이 미래를 과하게 당겨 썼다고 판단해서 앞선 분석의 결론은 관망이었습니다.

앞으로 볼 지표는 ARM 분기 실적의 중국 매출 비중(현재 약 16%), 서버급 RISC-V의 실제 데이터센터 배치 규모, 중국 정부 RISC-V 가이던스의 의무화 여부, 그리고 개발 생태계의 `riscv64` 타겟 증가입니다.

ps. 로열티가 칩 가격의 1에서 2%라는 말은 통념일 뿐이고 ARM 공시에 보편 요율이 없어서 확인하지 못했습니다. 2026년 시점의 주가와 RTX Spark 관련 데이터는 검증에 쓴 모델의 지식 시점 차이로 한계가 있습니다.

# 용어

| 용어 | 뜻 |
|------|----|
| ISA | Instruction Set Architecture. CPU가 알아듣는 명령어 규격. ARM, x86, RISC-V가 각각 다른 ISA |
| arm64 / aarch64 | ARM의 64비트 ISA. 같은 것의 다른 이름. Apple은 arm64, 리눅스는 aarch64로 부름 |
| x86-64 / amd64 | x86의 64비트 확장. AMD가 개발해서 amd64라고도 부름 |
| RISC / CISC | 간결 명령어와 복잡 명령어 설계 철학. ARM과 RISC-V는 RISC, x86은 CISC |
| 코어 IP | ARM이 파는 CPU 설계 자산(Cortex, Neoverse) |
| 아키텍처 라이선스 | ISA 규격만 받고 코어는 직접 설계할 권리 |
| CSS | Compute Subsystem. 코어와 주변 블록을 묶어 파는 고부가 패키지 |
| 로열티 | 칩이 출하될 때마다 ISA 소유자에게 내는 칩당 사용료 |
| OEM | 칩을 사다가 완제품(폰, 노트북)을 만드는 회사 |
| 해자(moat) | 경쟁사가 넘기 어려운 진입 장벽 |

# references

- [Wikipedia: RISC-V](https://en.wikipedia.org/wiki/RISC-V)
- [Wikipedia: Arm Holdings](https://en.wikipedia.org/wiki/Arm_Holdings)
- [Wikipedia: x86-64](https://en.wikipedia.org/wiki/X86-64)
- [Wikipedia: AArch64](https://en.wikipedia.org/wiki/AArch64)
- [Wikipedia: Memory ordering](https://en.wikipedia.org/wiki/Memory_ordering)
- [Wikipedia: ARM Neoverse](https://en.wikipedia.org/wiki/ARM_Neoverse)
- [Wikipedia: ARM Cortex-A](https://en.wikipedia.org/wiki/ARM_Cortex-A)
- [Wikipedia: Apple silicon](https://en.wikipedia.org/wiki/Apple_silicon)
- [Wisconsin HPCA 2013: Power Struggles: Revisiting the RISC vs. CISC Debate](https://research.cs.wisc.edu/vertical/papers/2013/hpca13-isa-power-struggles.pdf)
- [Chips and Cheese: ARM or x86? ISA Doesn't Matter](https://chipsandcheese.com/p/arm-or-x86-isa-doesnt-matter)
- [AWS EC2 Graviton](https://aws.amazon.com/ec2/graviton/)
- [Tom's Hardware: Intel launches Lunar Lake](https://www.tomshardware.com/pc-components/cpus/intel-launches-lunar-lake-claims-arm-beating-battery-life-worlds-fastest-mobile-cpu-cores)
- [Notebookcheck: Intel Lunar Lake CPU analysis](https://www.notebookcheck.net/Intel-Lunar-Lake-CPU-analysis-The-Core-Ultra-7-258V-s-multi-core-performance-is-disappointing-but-its-everyday-efficiency-is-good.893405.0.html)
- [RISC-V International FAQ](https://riscv.org/about/faq/)
- RISC-V International, "More than 16 billion RISC-V chips forecasted by 2030" (2023-11-07)
- SHD Group, RISC-V Market Analysis 2024 Abridged Report
- Reuters, "China to publish policy to boost RISC-V chip use nationwide" (2025-03-04)
- MacRumors, Apple과 ARM의 로열티 계약 보도 (2023-11-29)
- ARM Holdings FY2026 Form 20-F (SEC EDGAR, 2026-05 제출)
- stockanalysis.com, ARM 재무 지표 (2026-06-01)
