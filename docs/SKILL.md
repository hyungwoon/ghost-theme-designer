---
name: ghost-theme-designer
description: Casper Ghost 테마 디자인 수정 스킬. /ghost-theme-designer 커맨드로 호출합니다. CSS/HBS 수정 → 빌드 → ZIP 전체 파이프라인을 실행합니다.
---

# Ghost Theme Designer

Casper Ghost 테마(v5.9.0) CSS/디자인 수정 전용 스킬입니다.

**저장소**: https://github.com/hyungwoon/ghost-theme-designer

## 활성화 시점

- Ghost 테마 색상, 폰트, 레이아웃 변경 요청
- 포스트 카드, 헤더, 네비게이션 등 UI 수정
- 다크모드 스타일 변경
- 새 CSS 규칙 또는 HBS 파티얼 추가
- Ghost Admin 커스텀 설정(색상, 폰트) 활용한 스타일링

## 워크플로우

```
요청 수신
    ↓
복잡도 판단
    ├── 간단 (색상/여백 조정) → 즉시 수정
    └── 복잡 (레이아웃/다중파일) → 분석 후 수정
                                   ↓
                        yarn test (자동 빌드 + gscan 검증)
                                   ↓
                         yarn zip (dist/casper.zip 생성)
                                   ↓
                      완료 보고 (변경 내용 + 파일명)
```

**개발 중 확인:**
- `yarn dev` 실행 후 http://localhost:3000 에서 실시간 확인
- HBS 수정 → 자동 livereload
- CSS 수정 → 자동 PostCSS 컴파일

## 복잡도 기준

**즉시 수정 (간단):**
- 색상 변경 (`--color-*` 변수 수정)
- 폰트 크기/두께 조정
- 여백(margin/padding) 조정
- 단일 요소 스타일 변경

**분석 후 수정 (복잡):**
- 레이아웃 구조 변경 (그리드, flex 방향)
- 여러 파일에 걸친 수정
- 새 컴포넌트/파티얼 생성
- 반응형 브레이크포인트 추가/수정
- 네비게이션 레이아웃 변경

## 핵심 규칙

1. `assets/css/screen.css` 수정 (소스), `assets/built/` 직접 수정 금지
2. Casper 기존 클래스명 보존 — 기존 CSS 셀렉터 변경하지 않기
3. 색상 수정 시 `html.dark-mode` + `html.auto-color` 모두 업데이트
4. 수정 후 반드시 `yarn build && yarn test && yarn zip` 실행
5. gscan 오류 발생 시 ZIP 생성 전에 반드시 수정

## 자주 쓰는 CSS 셀렉터

```css
/* 네비게이션 */
.gh-head { }                    /* 상단 네비바 */
.gh-head-logo { }               /* 로고 */
.gh-head-menu .nav a { }        /* 메뉴 링크 */
.gh-head-button { }             /* Subscribe 버튼 */

/* 홈 커버 */
.site-header-content { }        /* 커버 영역 */
.site-title { }                 /* 사이트 제목 */
.site-description { }           /* 사이트 설명 */

/* 포스트 피드 */
.post-feed { }                  /* 포스트 그리드 */
.post-card { }                  /* 포스트 카드 */
.post-card-title { }            /* 카드 제목 */
.post-card-excerpt { }          /* 카드 발췌 */

/* 개별 포스트 */
.article-title { }              /* 포스트 제목 */
.gh-content { }                 /* 포스트 본문 */
.gh-canvas { }                  /* 콘텐츠 그리드 */
.article-byline { }             /* 저자 정보 */

/* 푸터 */
.site-footer { }                /* 사이트 푸터 */
.footer-cta { }                 /* 구독 CTA */
```

## CSS 변수 레퍼런스

```css
:root {
    --color-darkgrey: #15171A;
    --color-midgrey: #738a94;
    --color-lightgrey: #f1f1f1;
    --color-secondary-text: #979797;
    --color-border: #e1e1e1;
    --color-wash: #e5eff5;
    --color-darkmode: #151719;
    --ghost-accent-color: /* Ghost Admin 설정 */;
    --font-sans: -apple-system, BlinkMacSystemFont, ...;
    --font-serif: Georgia, Times, serif;
    --gh-font-heading: /* 사용자 선택 */;
    --gh-font-body: /* 사용자 선택 */;
}
```

## 사용 예제

**요청:** "헤더 배경색을 진청색으로 변경해줘"

**실행:**
1. `assets/css/screen.css` Section 3에서 `.gh-head` 색상 수정
2. `html.dark-mode` 및 `html.auto-color` 모두 업데이트
3. `yarn dev`로 확인
4. `yarn test` → `yarn zip` 실행

**완료 보고:**
```
✅ 수정 완료
- 변경 파일: assets/css/screen.css
- 변경 내용: 헤더 배경색을 진청색(#1a3a52)으로 변경 (다크모드/자동 모드 포함)
- 검증: yarn test ✓ 통과
- 배포: dist/casper.zip 생성 완료
  → Ghost Admin > Design > Theme > Upload 에서 업로드하세요
```

## 빌드 명령어

```bash
cd /Users/hyungwoonlee/Downloads/casper

# 개발 중 실시간 확인 (livereload)
yarn dev

# 최종 검증 및 빌드
yarn test          # gulp build + gscan 검사 (자동 실행)
yarn test:ci       # CI 환경 검증 (--fatal --verbose)

# 배포용 패키징
yarn zip           # dist/casper.zip 생성
```

**빌드 파이프라인:**
- CSS: `assets/css/screen.css` → PostCSS (import → color-mod → autoprefixer → cssnano) → `assets/built/screen.css`
- JS: `assets/js/*.js` → concat → uglify → `assets/built/casper.js`
- **주의:** `assets/built/` 직접 편집 금지, 항상 소스 파일 수정
