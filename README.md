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
  if면 IDE 재실행 해보기
```
### MSW library 설치
```
npm install msw --save-dev
```
### MockServiceWorker 생성
```
npx msw init public/ --save
```
### browser
```
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
### handler
```
import { http } from 'msw'
export const handlers = [ ...
```
### server
```
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

### 테스트 설정
1. `vitest.config.ts` 설정:
```typescript
import { defineConfig } from 'vitest/config'
import path from 'path'

export default defineConfig({
  test: {
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
})
```

2. `src/test/setup.ts` 설정:
```typescript
import { cleanup } from '@testing-library/react'
import { afterEach } from 'vitest'

afterEach(() => {
  cleanup()
})
```

### 테스트 실행 스크립트
```json
{
  "scripts": {
    "test": "vitest",
    "test:coverage": "vitest run --coverage",
    "test:ui": "vitest --ui"
  }
}
```

```
npm test
```

### 주요 테스트 기능
1. Vitest
   - 빠른 테스트 실행
   - React 19 호환성
   - Watch 모드 지원
   - UI 모드 지원

2. MSW 테스트 통합
   - API 모킹
   - 테스트 격리
   - 핸들러 재설정

3. 테스트 커버리지
   - 코드 커버리지 리포트
   - HTML, JSON, 텍스트 형식 지원
