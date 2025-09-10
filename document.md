# Meal Roulette - 기술명세서

## 📋 프로젝트 개요

**Meal Roulette**는 TheMealDB API를 활용한 실시간 레시피 웹애플리케이션입니다. 단일 HTML 파일로 구성되어 있으며, 모바일 우선 반응형 디자인과 한국어 UI를 제공합니다.

### 🎯 주요 목표
- 실시간 레시피 검색 및 추천
- 사용자 친화적인 인터페이스
- 오프라인 캐싱을 통한 성능 최적화
- 접근성 고려한 웹 표준 준수

## 🏗️ 아키텍처

### 기술 스택
- **Frontend**: HTML5, CSS3, Vanilla JavaScript
- **API**: TheMealDB REST API
- **스타일링**: CSS Grid, Flexbox, CSS Variables
- **폰트**: Google Fonts (Noto Sans KR)
- **저장소**: localStorage (브라우저)

### 파일 구조
```
project/
├── index.html          # 메인 애플리케이션 파일
└── document.md         # 기술명세서
```

## 🔌 API 명세

### 기본 설정
- **Base URL**: `https://www.themealdb.com/api/json/v1/1/`
- **응답 형식**: JSON
- **캐시 정책**: 30분 (브라우저 메모리)

### 사용 엔드포인트

#### 1. 카테고리 목록 조회
```http
GET /list.php?c=list
```
**응답 예시**:
```json
{
  "meals": [
    {"strCategory": "Beef"},
    {"strCategory": "Chicken"},
    {"strCategory": "Dessert"}
  ]
}
```

#### 2. 레시피 검색
```http
GET /search.php?s={query}
```
**파라미터**:
- `s`: 검색어 (필수)

#### 3. 카테고리별 레시피 조회
```http
GET /filter.php?c={category}
```
**파라미터**:
- `c`: 카테고리명 (필수)

#### 4. 랜덤 레시피 조회
```http
GET /random.php
```

#### 5. 레시피 상세 조회
```http
GET /lookup.php?i={id}
```
**파라미터**:
- `i`: 레시피 ID (필수)

## 🎨 UI/UX 설계

### 디자인 시스템

#### 색상 팔레트
```css
:root {
  --primary-color: #ff6b6b;      /* 메인 브랜드 색상 */
  --secondary-color: #4ecdc4;    /* 보조 색상 */
  --accent-color: #45b7d1;       /* 강조 색상 */
  --text-primary: #2c3e50;       /* 주요 텍스트 */
  --text-secondary: #7f8c8d;     /* 보조 텍스트 */
  --bg-primary: #ffffff;         /* 배경색 */
  --bg-secondary: #f8f9fa;       /* 보조 배경 */
  --border-color: #e9ecef;       /* 테두리 색상 */
}
```

#### 다크 모드 색상
```css
[data-theme="dark"] {
  --text-primary: #ecf0f1;
  --text-secondary: #bdc3c7;
  --bg-primary: #2c3e50;
  --bg-secondary: #34495e;
  --bg-card: #34495e;
  --border-color: #4a5f7a;
}
```

### 반응형 브레이크포인트
- **모바일**: < 768px
- **태블릿**: 768px - 1024px
- **데스크톱**: > 1024px

### 컴포넌트 구조

#### 1. 헤더 (Header)
- 로고 및 브랜딩
- 테마 토글 버튼
- 스티키 포지셔닝

#### 2. 검색 섹션 (Search Section)
- 검색 입력창 (디바운스 400ms)
- 카테고리 드롭다운
- 랜덤 추천 버튼

#### 3. 레시피 그리드 (Recipe Grid)
- CSS Grid 레이아웃
- 카드 기반 디자인
- 호버 효과 및 애니메이션

#### 4. 상세 페이지 (Detail Page)
- 해시 라우팅 (`#/meal/{id}`)
- 이미지 갤러리
- 재료 체크리스트
- 조리 방법 단계별 표시

## ⚙️ 핵심 기능

### 1. 검색 시스템
```javascript
// 디바운스 구현
function handleSearch(event) {
    clearTimeout(searchTimeout);
    searchTimeout = setTimeout(() => {
        const query = event.target.value.trim();
        if (query.length >= 2) {
            searchMeals(query);
        }
    }, 400);
}
```

### 2. 캐싱 시스템
```javascript
// 30분 캐시 구현
const CACHE_DURATION = 30 * 60 * 1000;
const cache = new Map();

async function fetchWithCache(url) {
    const cached = cache.get(url);
    if (cached && Date.now() - cached.timestamp < CACHE_DURATION) {
        return cached.data;
    }
    // API 호출 및 캐시 저장
}
```

### 3. 재시도 로직
```javascript
// 지수 백오프 재시도
async function fetchWithRetry(url, maxRetries = 3) {
    for (let i = 0; i < maxRetries; i++) {
        try {
            const response = await fetch(url);
            if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
            return await response.json();
        } catch (error) {
            if (i === maxRetries - 1) throw error;
            await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
        }
    }
}
```

### 4. 재료 파싱
```javascript
// strIngredient1-20, strMeasure1-20 파싱
function parseIngredients(meal) {
    const ingredients = [];
    for (let i = 1; i <= 20; i++) {
        const ingredient = meal[`strIngredient${i}`];
        const measure = meal[`strMeasure${i}`];
        
        if (ingredient && ingredient.trim()) {
            ingredients.push({
                id: i,
                name: ingredient.trim(),
                measure: (measure || '').trim()
            });
        }
    }
    return ingredients;
}
```

## 💾 데이터 관리

### localStorage 구조

#### 즐겨찾기 (Favorites)
```javascript
// 저장 형식
localStorage.setItem('favorites', JSON.stringify(['52772', '52773']));
```

#### 최근 본 항목 (Recent Meals)
```javascript
// 저장 형식 (최대 20개)
localStorage.setItem('recentMeals', JSON.stringify([
    { idMeal: '52772', strMeal: 'Teriyaki Chicken Casserole', ... },
    { idMeal: '52773', strMeal: 'Chicken Handi', ... }
]));
```

#### 테마 설정 (Theme)
```javascript
// 저장 형식
localStorage.setItem('theme', 'dark'); // 'light' | 'dark'
```

## 🚀 성능 최적화

### 1. 이미지 최적화
- **지연 로딩**: `loading="lazy"` 속성 사용
- **폴백 이미지**: SVG 기반 에러 처리
- **적절한 크기**: CSS로 이미지 크기 제어

### 2. 네트워크 최적화
- **캐싱**: 30분 메모리 캐시
- **재시도**: 지수 백오프 알고리즘
- **디바운스**: 검색 입력 400ms 지연

### 3. 렌더링 최적화
- **CSS Grid**: 효율적인 레이아웃
- **CSS Variables**: 테마 전환 최적화
- **애니메이션**: GPU 가속 활용

## ♿ 접근성 (Accessibility)

### ARIA 라벨
```html
<button class="theme-toggle" onclick="toggleTheme()" aria-label="테마 변경">
<input type="text" class="search-input" aria-label="레시피 검색">
<button class="favorite-btn" aria-label="즐겨찾기 추가">
```

### 키보드 네비게이션
- **Tab 순서**: 논리적인 포커스 순서
- **포커스 표시**: 명확한 시각적 피드백
- **키보드 이벤트**: Enter, Escape 키 지원

### 스크린 리더 지원
- **시맨틱 HTML**: 적절한 태그 사용
- **alt 텍스트**: 모든 이미지에 설명 제공
- **sr-only 클래스**: 화면에만 표시되는 텍스트

## 🔧 설정 및 커스터마이징

### API 교체 방법
1. `API_BASE_URL` 상수 수정
2. API 응답 형식에 맞게 데이터 파싱 로직 수정
3. 필요한 경우 새로운 엔드포인트 추가

```javascript
// API 설정 변경 예시
const API_BASE_URL = 'https://your-api.com/v1/';
```

### 테마 커스터마이징
```css
:root {
  --primary-color: #your-color;    /* 메인 색상 변경 */
  --secondary-color: #your-color;  /* 보조 색상 변경 */
  --radius: 8px;                   /* 모서리 둥글기 조정 */
}
```

## 📱 배포 가이드

### GitHub Pages
1. GitHub 저장소 생성
2. `index.html` 파일 업로드
3. Settings > Pages에서 Source를 "Deploy from a branch" 선택
4. Branch를 "main"으로 설정
5. 자동 배포 완료

### Cloudflare Pages
1. Cloudflare 대시보드에서 Pages 프로젝트 생성
2. `index.html` 파일 업로드 또는 GitHub 저장소 연결
3. 자동 배포 설정
4. 커스텀 도메인 연결 (선택사항)

### 로컬 개발
```bash
# 간단한 HTTP 서버 실행
python -m http.server 8000
# 또는
npx serve .
```

## 🐛 에러 처리

### 네트워크 에러
- **재시도 로직**: 최대 3회 재시도
- **사용자 알림**: 토스트 메시지로 에러 표시
- **폴백 UI**: 빈 상태 화면 표시

### API 에러
- **HTTP 상태 코드**: 적절한 에러 메시지
- **타임아웃**: 10초 후 재시도
- **캐시 활용**: 오프라인 상태에서 캐시된 데이터 사용

### 브라우저 호환성
- **ES6+ 지원**: 모던 브라우저 대상
- **폴리필**: 필요시 추가 고려
- **Graceful Degradation**: 기능별 폴백 제공

## 📊 성능 지표

### 목표 성능
- **First Contentful Paint**: < 1.5초
- **Largest Contentful Paint**: < 2.5초
- **Cumulative Layout Shift**: < 0.1
- **First Input Delay**: < 100ms

### 최적화 전략
- **코드 분할**: 단일 파일이지만 모듈화된 구조
- **이미지 최적화**: 적절한 크기와 포맷
- **캐싱**: 브라우저 및 메모리 캐시 활용
- **압축**: Gzip/Brotli 압축 권장

## 🔮 향후 개선 사항

### 기능 확장
- [ ] 사용자 계정 시스템
- [ ] 레시피 평가 및 리뷰
- [ ] 식단 계획 기능
- [ ] 영양 정보 표시
- [ ] 다국어 지원

### 기술 개선
- [ ] PWA (Progressive Web App) 변환
- [ ] Service Worker 도입
- [ ] 오프라인 모드 지원
- [ ] 푸시 알림
- [ ] 백그라운드 동기화

### 성능 최적화
- [ ] 이미지 WebP 포맷 지원
- [ ] Critical CSS 인라인
- [ ] 리소스 프리로딩
- [ ] 번들 크기 최적화

## 📝 라이선스

이 프로젝트는 MIT 라이선스 하에 배포됩니다.

## 🤝 기여 가이드

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

**문서 버전**: 1.0.0  
**최종 업데이트**: 2024년 1월  
**작성자**: Meal Roulette Development Team
