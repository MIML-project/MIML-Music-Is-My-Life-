# 🎵 MIML — Music Is My Life

자연어로 지금 기분을 말하면, AI가 어울리는 음악을 추천해주는 서비스.

> "요즘 좀 지쳐있어서 조용하고 따뜻한 노래 듣고 싶어"  
> → 에너지 낮고 어쿠스틱한 감성의 곡들을 추천

---

## 기술 스택

| 구분 | 기술 |
|------|------|
| Language | Kotlin |
| Framework | Spring Boot 3.x |
| Database | MySQL (AWS RDS) |
| 인증 | Firebase Authentication |
| 플레이리스트 저장 | Firestore |
| 배포 | AWS EC2 (Amazon Linux 2023, Java 17) |
| 외부 API | Spotify, Last.fm, OpenAI (GPT-4o-mini, text-embedding-3-small), SoundNet (RapidAPI) |

---

## 프로젝트 구조

```
backend/
└── src/main/kotlin/com/miml/backend/
    ├── client/         # 외부 API 클라이언트 (Spotify, OpenAI, Last.fm, SoundNet)
    ├── config/         # Firebase, Security, WebClient 설정
    ├── controller/     # REST API 엔드포인트
    ├── dto/            # 요청/응답 데이터 클래스
    ├── entity/         # JPA 엔티티 (Music, AudioFeatures, MusicTag, User 등)
    ├── repository/     # Spring Data JPA 레포지토리
    ├── Security/       # Firebase 인증 필터
    └── service/        # 핵심 비즈니스 로직

frontend/
├── App.js                        # 네비게이션 설정 (Stack + Bottom Tab)
├── firebase.js                   # Firebase 초기화
├── screens/                      # 화면 컴포넌트
│   ├── HomeScreen.js             # AI 채팅 기반 음악 추천 메인 화면
│   ├── LibraryScreen.js          # 보관함 (플레이리스트 목록)
│   ├── FeedScreen.js             # 커뮤니티 피드
│   ├── NowPlayingScreen.js       # 현재 재생 중인 곡 상세 화면
│   ├── PlaylistDetailScreen.js   # 플레이리스트 상세
│   ├── ProfileScreen.js          # 사용자 프로필 및 취향 분석
│   └── LoginScreen.js            # 로그인 화면
├── components/                   # 재사용 UI 컴포넌트
│   ├── NowPlayingBar.js          # 하단 미니 플레이어 바
│   ├── AddToPlaylistModal.js     # 플레이리스트 추가 모달
│   ├── SaveAsPlaylistModal.js    # 새 플레이리스트 저장 모달
│   └── Skeleton.js               # 로딩 스켈레톤 UI
└── context/                      # 전역 상태 관리
    ├── PlayerContext.js          # 재생 상태, 플레이리스트, 취향 프로필
    ├── AuthContext.js            # Firebase 인증 상태
    └── SpotifyContext.js         # Spotify 연동 및 재생 제어
```

---

## API 엔드포인트

모든 API는 Firebase Auth 토큰(`Authorization: Bearer <token>`) 필요.

| Method | Endpoint | 설명 |
|--------|----------|------|
| `POST` | `/api/recommend/smart` | 자연어 기반 음악 추천 |
| `POST` | `/api/feedback/satisfaction` | 좋아요/싫어요 피드백 |
| `GET` | `/api/user/me` | 유저 프로필 조회 |
| `POST` | `/api/spotify/import/liked` | Spotify 좋아요 곡 import |

---

## 로컬 실행 방법

### 백엔드

#### 1. 환경변수 설정

`.env` 파일을 생성하거나 환경변수로 아래 값들을 설정:

```env
DB_HOST=localhost
DB_USERNAME=your_db_username
DB_PASSWORD=your_db_password

SPOTIFY_CLIENT_ID=your_spotify_client_id
SPOTIFY_CLIENT_SECRET=your_spotify_client_secret

OPENAI_API_KEY=your_openai_api_key
LASTFM_API_KEY=your_lastfm_api_key
RAPIDAPI_KEY=your_rapidapi_key
```

#### 2. Firebase 설정

`backend/src/main/resources/firebase-service-account.json.example`을 복사해 `firebase-service-account.json`으로 저장한 뒤 실제 값 입력:

```bash
cp src/main/resources/firebase-service-account.json.example \
   src/main/resources/firebase-service-account.json
```

#### 3. 빌드 및 실행

```bash
cd backend
./gradlew bootRun
```

### 프론트엔드

#### 1. 백엔드 서버 주소 설정

아래 두 파일의 `BACKEND_URL`을 수정한다.

**`frontend/screens/HomeScreen.js`** (445번째 줄)
```js
const BACKEND_URL = 'http://백엔드_서버_IP:8080';
```

**`frontend/screens/FeedScreen.js`** (19번째 줄)
```js
const BACKEND_URL = 'http://백엔드_서버_IP:8080';
```

> 현재 기본값: `http://13.209.228.73:8080` (AWS EC2)

#### 2. 패키지 설치 및 실행

```bash
cd frontend
npm install
npx expo start --android
```

> Expo Go 앱 또는 Android 에뮬레이터 필요

---


## 프론트엔드

### 기술 스택

| 구분 | 기술 |
|------|------|
| Framework | React Native (Expo SDK 54) |
| Language | JavaScript (ES2022) |
| 상태 관리 | React Context API |
| 네비게이션 | React Navigation (Bottom Tab + Stack) |
| 인증 | Firebase Authentication |
| DB | Firebase Firestore (커뮤니티 게시물, 플레이리스트) |
| 음악 재생 | Spotify SDK 연동 |

### 주요 기능

- **AI 채팅 UI**: 자연어로 상황을 입력하면 백엔드 추천 API를 호출하여 곡 목록을 채팅 버블 형태로 표시
- **개인화 슬라이더**: `Now ↔ My Vibe` 슬라이더로 현재 기분과 평소 취향의 반영 비율(α) 실시간 조절
- **Spotify 연동 재생**: 추천 곡을 Spotify SDK로 즉시 재생, 미니 플레이어 상시 표시
- **보관함**: Liked Songs, Recently Played, 사용자 생성 플레이리스트 관리 (Firestore 동기화)
- **커뮤니티**: 감상 로그 작성 및 피드 조회, 게시물 데이터가 추천 알고리즘 가중치에 반영

