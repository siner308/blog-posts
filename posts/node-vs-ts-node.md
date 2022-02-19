---
title:  "node와 ts-node의 CPU,RAM 사용량 비교"
subtitle: "ts-node를 production에서 사용하면 안되는 이유"
tags:
  - typescript
  - node
date:   2022-02-19
---
node와 ts-node의 차이가 어디에서 오는지 알아보기 위해 조사한 과정을 기록으로 남기고자 합니다.

특정 코드의 동작시간과 메모리 사용량을 각각 비교해보았습니다.

# 테스트 코드
테스트하고자 하는 코드는 소수의 개수를 반환하는 간단한 코드입니다.

```typescript
function isPrime(num: number): boolean {
  if (num < 2) return false;
  if (num == 2) return true;
  for (let i = 2; i < num; i++) {
    if (num % i == 0) return false;
  }
  return true;
}

function test(): void {
  const t0 = performance.now();
  const iterNum = 100000;
  const primes: number[] = [];

  for (let i = 2; i < iterNum; i++) {
    const progressRate: number = (i / iterNum) * 100;
    if (isPrime(i)) primes.push(i);
  }

  const t1 = performance.now();
}
```

<details>
<summary><span style="cursor: pointer; border-width: 0.1em; border-style: solid; border-color: black">전체 벤치마크 코드 (열어서 보기)</span></summary>

```typescript
interface MemoryUsage {
  rss: number;
  heapTotal: number;
  heapUsed: number;
  external: number;
  arrayBuffers: number;
}

function isPrime(num: number): boolean {
  if (num < 2) return false;
  if (num == 2) return true;
  for (let i = 2; i < num; i++) {
    if (num % i == 0) return false;
  }
  return true;
}

function printUsage(metrics: { progress: number; usage: MemoryUsage }[], key: string) {
  console.log(key);
  let prev: MemoryUsage = metrics[0].usage;
  metrics.forEach((curr) => {
    const { progress, usage } = curr;
    console.log(`[${progress.toString().padStart(3, '0')}%]${usage[key]}(${usage[key] - prev[key] >= 0 ? '+' : ''}${usage[key] - prev[key]})`);
    prev = curr.usage;
  });
  console.log('\n');
}

function test(): void {
  const t0 = performance.now();

  let progress: number = 0;
  const iterNum = 100000;
  const primes: number[] = [];
  const metrics: { progress: number, usage: MemoryUsage }[] = [];
  for (let i = 2; i < iterNum; i++) {
    const progressRate: number = (i / iterNum) * 100;
    if (progress <= progressRate) {
      metrics.push({ progress, usage: process.memoryUsage() });
      progress += 10;
    }
    if (isPrime(i)) primes.push(i);
  }
  metrics.push({ progress, usage: process.memoryUsage() });
  const t1 = performance.now();

  console.log('=============================');
  console.log(`소수는 ${primes.length}개 입니다.`);
  console.log((t1 - t0) + 'ms 걸렸습니다.');
  console.log('=============================');

  printUsage(metrics, 'rss');
  printUsage(metrics, 'heapTotal');
  printUsage(metrics, 'heapUsed');
  printUsage(metrics, 'external');
  printUsage(metrics, 'arrayBuffers');
}

test();
```
</details>

# 결과
좌측(또는 위)이 node, 우측(또는 아래)이 ts-node입니다.

## 시간
node와 ts-node 모두 출력된 결과에선 별 차이가 없었고, 반복문을 10만에서 100만회로 늘려서 다시 시도해보아도 0.3%정도의 오차율만 보였습니다.

ts-node의 경우 ts-node 패키지를 실행하는 시간으로 인해 실제로는 수 초 이상의 시간이 더 소요되었습니다. 하지만 이러한 컴파일 타임은 서버와 같이 장시간 동작하는 프로세스에서는 의미없는 차이라고 느껴집니다.

<div style="display: flex">
<img width="50%" alt="스크린샷 2022-02-19 오후 4 22 24" src="https://user-images.githubusercontent.com/34048253/154791289-9d567493-c04c-41ca-b21d-30361287f442.png">
<img width="50%" alt="스크린샷 2022-02-19 오후 4 23 46" src="https://user-images.githubusercontent.com/34048253/154791334-074cc7c0-d2d3-4e13-a414-468aa2035bad.png">
</div>

## Resident Set Size (RSS)
RSS의 경우 7~8배 정도의 차이가 있었습니다.
또한 node의 rss는 코드가 진행됨에따라 변화했지만, ts-node는 변화가 거의 없었습니다.
node는 최소한의 메모리만을 사용하도록 최적화가 되어있는 반면, ts-node는 한번에 쫙 땡겨서 쓴다는 느낌을 받았습니다.

<div style="display: flex">
<img width="206" alt="스크린샷 2022-02-19 오후 4 22 38" src="https://user-images.githubusercontent.com/34048253/154791298-784cbe7f-6f98-499f-be9e-ff2887b222f7.png">

<img width="191" alt="스크린샷 2022-02-19 오후 4 24 00" src="https://user-images.githubusercontent.com/34048253/154791343-f48a19c6-457a-4409-ac39-fae2bf0ef4bf.png">
</div>

## heapTotal

heapTotal의 경우 20에서 최대 26배까지 차이가 있었습니다. ts-node의 경우 typescript를 해석하기 위해 컴파일러를 메모리에 로드하는데, 이 때문에 메모리의 차이가 크게 발생하는 것으로 보입니다.

```typescript
// ./node_modules/ts-node/index.ts
function loadCompiler(name: string | undefined, relativeToPath: string) {
  const projectLocalResolveHelper =
    createProjectLocalResolveHelper(relativeToPath);
  const compiler = projectLocalResolveHelper(name || 'typescript', true);
  const ts: TSCommon = attemptRequireWithV8CompileCache(require, compiler);
  return { compiler, ts, projectLocalResolveHelper };
}
```

<div style="display: flex">
<img width="193" alt="스크린샷 2022-02-19 오후 4 22 52" src="https://user-images.githubusercontent.com/34048253/154791307-21a0e4d1-6ebd-417d-99ea-3735b68e6182.png">
<img width="158" alt="스크린샷 2022-02-19 오후 4 24 10" src="https://user-images.githubusercontent.com/34048253/154791349-1070f20d-840e-43e5-ad99-7214cb39e975.png">
</div>

## heapUsed

heapUsed의 경우 24~27배정도의 차이가 있었습니다.
눈에 띄었던 점은, node의 경우 30% 구간을 통과할때 heapUsed가 60만에서 70만정도 감소했지만, ts-node의 경우엔 계속해서 증가하기만 하는 차이를 보였습니다. 이는 반복해서 테스트를 돌려도 같은 결과가 나타났습니다.

<div style="display: flex">
<img width="183" alt="스크린샷 2022-02-19 오후 4 23 03" src="https://user-images.githubusercontent.com/34048253/154791313-ee720cb4-16a0-4482-aa8a-f78815a84916.png">
<img width="198" alt="스크린샷 2022-02-19 오후 4 24 20" src="https://user-images.githubusercontent.com/34048253/154791353-25cdd0d4-97c6-4c5c-89e1-d90b9f69cb61.png">
</div>

## external

external은 V8에서 관리하는 객체로써, JavaScript에 바인딩된 C++ 객체의 메모리 사용량을 나타냅니다.
external또한 5배정도의 차이가 있었습니다.

<div style="display: flex">
<img width="148" alt="스크린샷 2022-02-19 오후 4 23 15" src="https://user-images.githubusercontent.com/34048253/154791324-900cf6ff-6645-4bbe-bec2-9dad6ba7fcd9.png">
<img width="147" alt="스크린샷 2022-02-19 오후 4 24 36" src="https://user-images.githubusercontent.com/34048253/154791359-98f8a107-38a2-4c8d-9f5b-a68c94aaa84a.png">
</div>

## arrayBuffers

arrayBuffers는 약 4배정도의 차이를 보였습니다.

<div style="display: flex">
<img width="126" alt="스크린샷 2022-02-19 오후 4 23 24" src="https://user-images.githubusercontent.com/34048253/154791330-8fb6d0a1-88b3-4844-bccb-574370102aaa.png">
<img width="131" alt="스크린샷 2022-02-19 오후 4 24 45" src="https://user-images.githubusercontent.com/34048253/154791362-57389610-7c49-489f-b4cb-3a1cd4c72336.png">
</div>

# 결론
ts-node는 js파일을 생성하지 않는 대신, typescript 컴파일러를 메모리에 올려서 동작시켜야 하기 때문에 heap 메모리의 낭비가 있으므로, 자원이 한정적인 real-world(production)에서는 사용하지 않는것이 무조건 좋습니다.
