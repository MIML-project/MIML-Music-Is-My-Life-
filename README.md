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

## 핵심 기능

### 1. 음악 데이터 파이프라인
- Spotify에서 사용자의 좋아요 곡을 일괄 import
- SoundNet(RapidAPI)으로 각 곡의 음향 수치(energy, happiness, danceability, acousticness, tempo) 분석 및 저장
- Last.fm으로 아티스트별 장르 태그 수집
- OpenAI `text-embedding-3-small`로 태그 벡터 생성

### 2. AI 추천 알고리즘 (`SmartRecommendationService`)
1. GPT-4o-mini가 자연어 입력을 분석해 오디오 피처 수치 + 장르 필터 추출
2. 유저 청취 프로필과 지정 비율(profileRatio)로 블렌딩
3. 오디오 범위 필터 + 태그 임베딩 코사인 유사도로 후보 풀 추출
4. 장르 pre-filter (사용자가 장르를 명시한 경우에만 적용)
5. 오디오 거리 점수 → 태그 커버리지 점수 순으로 최종 순위 결정
6. 아티스트 중복 제한(최대 2곡/아티스트) + 가중 샘플링

### 3. 피드백 기반 프로필 학습
- 좋아요/싫어요 피드백으로 유저의 오디오 피처 프로필 점진적 업데이트
- 무드 피드백으로 세부 분위기 선호도 반영

---

## API 엔드포인트

모든 API는 Firebase Auth 토큰(`Authorization: Bearer <token>`) 필요.

| Method | Endpoint | 설명 |
|--------|----------|------|
| `POST` | `/api/recommend/smart` | 자연어 기반 음악 추천 |
| `POST` | `/api/feedback/satisfaction` | 좋아요/싫어요 피드백 |
| `POST` | `/api/feedback/mood` | 무드 피드백 |
| `GET` | `/api/user/me` | 유저 프로필 조회 |
| `POST` | `/api/spotify/import/liked` | Spotify 좋아요 곡 import |
| `GET` | `/api/hello` | 서버 상태 확인 (인증 불필요) |

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

백엔드 서버 주소가 변경된 경우 아래 두 파일의 `BACKEND_URL`을 수정하세요.

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

## 데이터베이스 스키마

주요 테이블:

| 테이블 | 설명 |
|--------|------|
| `music` | 곡 메타데이터 (title, artist, album, spotifyId, albumImageUrl) |
| `audio_features` | 음향 수치 (energy, happiness, danceability, acousticness, tempo) |
| `music_tags` | 커뮤니티 태그 + voteCount |
| `artist_genres` | Last.fm 장르 데이터 |
| `users` | Firebase UID 기반 유저 + 오디오 피처 프로필 |

플레이리스트는 Firestore에 스냅샷으로 저장 (MySQL 변경에 독립적).

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

