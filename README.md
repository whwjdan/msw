# MSW(Mock Service Worker) 설정 가이드

## 1. 프로젝트 구조
```bash
src/
├── mocks/                  # MSW(Mock Service Worker) 관련 설정
│   ├── browser.ts          # MSW 브라우저 설정
│   ├── handlers.ts         # API 모킹 핸들러
│   ├── config.ts           # MSW 사용 설정
│   └── server.ts           # 테스트용 MSW 서버 설정
├── test/                   # 테스트 환경 관련 설정
│   └── setup.ts            # 테스트 환경 설정
└── app/                    # 애플리케이션 주요 코드
    └── layout.tsx          # MSW 초기화
```
## 2. 주요 파일 설정

### `src/mocks/config.ts`
```typescript
export const MSW_CONFIG = {
  USE_MSW: {
    PROJECTS: true,     // /api/projects API는 MSW 사용
    ART_STYLES: true,   // /api/art-styles API는 MSW 사용
  }
} as const;
```
- 각 API별로 MSW 사용 여부를 설정
- `true`: MSW로 모킹
- `false`: 실제 서버 사용

### `src/mocks/browser.ts`
```typescript
import { handlers } from './handlers'

export const setUpMocks = async () => {
  if (typeof window !== 'undefined') {
    const { setupWorker } = require("msw/browser");
    const worker = setupWorker(...handlers)
    
    return worker.start({
      onUnhandledRequest: 'bypass',
    });
  }
}
```
- 브라우저 환경에서 MSW worker 설정
- 개발 환경에서만 동작

### `src/app/layout.tsx`
```typescript
async function enableMocking() {
  if (process.env.NODE_ENV !== 'development') {
    return;
  }
  const { setUpMocks } = await import('@/src/mocks/browser');
  return setUpMocks();
}
```
- 개발 환경에서 MSW 초기화
- 서버 컴포넌트에서 안전하게 MSW 설정

## 3. 사용 방법

### 개발 환경에서 MSW 사용
1. `config.ts`에서 모킹할 API 설정:
```typescript
USE_MSW: {
  PROJECTS: true,    // 이 API는 MSW로 모킹
  ART_STYLES: false  // 이 API는 실제 서버 사용
}
```

2. `handlers.ts`에서 모킹 응답 정의:
```typescript
export const handlers = [
  if (!MSW_CONFIG.USE_MSW.PROJECTS) {
  http.get('/api/projects', () => {
    return HttpResponse.json({
      // 모킹할 데이터
    });
  }),
  // 다른 핸들러들...
];
```
2-1. 해당 코드로 핸들러에서 모킹 사용여부 결정
```
if (!MSW_CONFIG.USE_MSW.PROJECTS) {
```

3. 개발 서버 실행:
```bash
npm run dev
```

### 테스트 환경에서 MSW 사용
1. 테스트 파일에서 MSW 서버 설정:
```typescript
import { setupServer } from 'msw/node'
import { handlers } from './mocks/handlers'

const server = setupServer(...handlers)

beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())
```

2. 테스트 실행:
```bash
npm test
```

## 4. 주요 기능

### 1. 선택적 API 모킹
- `config.ts`를 통해 각 API별로 MSW 사용 여부 설정 가능
- 실제 서버와 MSW를 혼합 사용 가능

### 2. 개발 환경 전용
- MSW는 개발 환경에서만 동작
- 프로덕션 환경에서는 자동으로 비활성화

### 3. 테스트 지원
- Vitest와 통합하여 테스트 환경에서 API 모킹 가능
- 테스트별로 핸들러 재설정 가능

## 5. 확인 방법

### 개발 환경
- 브라우저 개발자 도구의 Network 탭에서 요청 확인
- MSW 로그는 Console 탭에서 확인 가능

### 테스트 환경
- 테스트 실행 시 MSW 서버 상태 로그 확인
- 테스트 케이스에서 모킹된 응답 검증

## 6. 주의사항

1. MSW 설정 변경 후에는 개발 서버 재시작 필요
2. `config.ts`의 설정이 `handlers.ts`의 구현과 일치해야 함
3. 테스트 환경에서는 `server.ts`의 설정이 사용됨
