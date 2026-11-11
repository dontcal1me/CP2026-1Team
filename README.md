# CP2026-1Team

# 냉장고 식재료 관리 프로그램 (Fridge Manager)

> 냉장고 속 식재료의 유통기한을 관리하고, 먼저 먹어야 할 재료를 알려 주며, 남은 재료로 만들 수 있는 요리를 추천하는 C언어 프로그램
> **콘솔 버전(필수)**을 먼저 완성하고, 같은 로직을 재사용해 **GUI 버전(도전)**으로 확장합니다.

- 과목: 비판적사고 (홍서빈 교수님)
- 팀명: `팀명 입력`
- 개발 언어: C
- 개발 환경: Visual Studio 2022 / Windows 10 이상
- GUI: Win32 API (Windows 기본 제공, 추가 라이브러리 설치 없음)

---

## 목차

1. [프로젝트 소개](#1-프로젝트-소개)
2. [프로젝트 목표](#2-프로젝트-목표)
3. [주요 기능](#3-주요-기능)
4. [설계 원칙: 로직과 화면 분리](#4-설계-원칙-로직과-화면-분리)
5. [모듈 구조](#5-모듈-구조)
6. [핵심 자료구조](#6-핵심-자료구조)
7. [데이터 파일 형식](#7-데이터-파일-형식)
8. [개발 환경 설정](#8-개발-환경-설정)
9. [빌드 및 실행 방법](#9-빌드-및-실행-방법)
10. [실행 화면 예시](#10-실행-화면-예시)
11. [팀원 및 역할](#11-팀원-및-역할)
12. [협업 규칙](#12-협업-규칙)
13. [참고 자료](#13-참고-자료)

---

## 1. 프로젝트 소개

냉장고에 무엇이 있는지, 언제까지 먹어야 하는지 잊어버려서 음식을 버리는 일이 자주 생깁니다.
이 프로그램은 식재료를 등록해 두면 **오늘 날짜 기준으로 남은 일수를 자동 계산**하고,
**유통기한이 임박한 재료를 먼저 알려 주며**, **그 재료로 만들 수 있는 요리를 추천**합니다.
또한 버린 식재료를 기록하고 통계로 보여 주어, 사용자가 자신의 소비 습관을 데이터로 돌아볼 수 있도록 돕습니다.

---

## 2. 프로젝트 목표

### 필수 목표 (콘솔 버전)
- F1~F6 기능을 콘솔 프로그램으로 모두 완성
- 데이터 파일 저장·불러오기로 프로그램을 껐다 켜도 정보 유지
- 로직 모듈과 화면 모듈을 분리한 구조로 구현

### 도전 목표 (GUI 버전)
콘솔 버전이 완성된 뒤 단계별로 진행합니다. 로직 모듈은 **수정 없이 그대로** 재사용합니다.

| 단계 | 내용 | 난이도 |
|:---:|---|:---:|
| G1 | 프로그램 시작 시 유통기한 알림을 Windows 팝업 창(`MessageBoxW`)으로 표시 | 하 |
| G2 | 메인 창에 식재료 목록 표(ListView)와 기능 버튼 배치, 임박순 정렬 표시 | 중 |
| G3 | 식재료 등록·수정 입력 창, 삭제·수량 변경 버튼 | 중 |
| G4 | 요리 추천 창(추천 목록 + "요리하기" 버튼으로 자동 차감), 레시피 추가 창 | 상 |
| G5 | 폐기 통계 창, 임박 재료는 목록에서 빨간색·주황색으로 강조 | 상 |

> GUI는 G1부터 순서대로 진행하며, 일정이 부족하면 완성된 단계까지만 제출합니다. 콘솔 버전은 항상 실행 가능한 상태로 유지합니다.

---

## 3. 주요 기능

| 번호 | 기능 | 설명 | 로직 모듈 | 우선순위 |
|:---:|---|---|---|:---:|
| F1 | 식재료 등록 | 이름, 유통기한(YYYY-MM-DD), 수량, 보관 방법(냉장/냉동) 입력 | `food` | 필수 |
| F2 | 유통기한 임박순 자동 정렬 | 오늘 날짜를 자동으로 가져와 D-day 계산 후 남은 일수가 적은 순으로 정렬 | `expiry` | 필수 |
| F3 | 요리 추천 | 미리 등록된 레시피와 보유 재료를 비교해 추천, 사용자 정의 레시피 추가 | `recipe` | 필수 |
| F4 | 시작 시 임박 알림 | 실행 직후 "유통기한 지난 재료 N개, 2일 이내 재료 M개" 경고 표시 | `expiry` | 필수 |
| F5 | 식재료 수정·삭제·수량 관리 | 정보 수정, 삭제, 수량 변경 / 요리 선택 시 사용한 재료 자동 차감 | `food`, `recipe` | 필수 |
| F6 | 폐기 기록 통계 | 폐기한 재료를 기록하고 총 폐기 수, 보관 방법별 비율, 가장 많이 버린 재료 출력 | `stats` | 필수 |
| G1~G5 | GUI 버전 | 위 기능을 Windows 창 화면으로 제공 | `ui_gui` | 도전 |

### 기능 상세

**F2. D-day 계산**
- `time.h`의 `time()`, `localtime_s()`로 오늘 날짜를 가져옵니다.
- 유통기한 날짜를 `struct tm`으로 바꾼 뒤 `mktime()`과 `difftime()`으로 남은 일수를 계산합니다.
- 표기 예: `D-3`, `D-DAY`, `D+2 (기한 지남)`

**F3. 요리 추천 점수**
- 보유한 재료 1개당 +10점
- 유통기한 2일 이내 재료를 사용하면 +5점
- 필요한 재료를 모두 가지고 있으면 +20점
- 점수 순으로 정렬하고, 필요 재료의 절반 이상을 가진 요리만 추천하며, 부족한 재료도 함께 보여 줍니다.

**F4. 시작 알림**
- 콘솔: 테두리 박스 형태의 경고 화면 → 아무 키나 누르면 메인 메뉴로 이동
- GUI(G1): Windows 경고 팝업 창

**F5. 자동 차감**
- 추천 목록에서 요리를 선택하면 확인을 받은 뒤 레시피에 적힌 수량만큼 재료를 차감합니다.
- 수량이 0이 된 재료는 목록에서 자동 삭제됩니다.

**F6. 폐기 기록**
- 식재료를 삭제할 때 폐기로 기록할지 선택합니다.
- "기한 지난 재료 일괄 폐기" 기능으로 기한이 지난 재료를 한꺼번에 폐기 처리합니다.

---

## 4. 설계 원칙: 로직과 화면 분리

같은 기능을 콘솔과 GUI 두 화면에서 쓰기 위해, 프로그램을 **로직 계층**과 **화면 계층**으로 나눕니다.

```
┌──────────────────────┐    ┌──────────────────────┐
│  화면 계층 (콘솔)     │    │  화면 계층 (GUI)      │
│  main.c              │    │  main_gui.c          │
│  ui_console.c        │    │  ui_gui.c            │
│  printf / scanf_s    │    │  Win32 API           │
└──────────┬───────────┘    └──────────┬───────────┘
           │      같은 함수를 호출       │
           └──────────────┬─────────────┘
                          ▼
┌──────────────────────────────────────────────────┐
│  로직 계층 (공통)                                 │
│  food.c  expiry.c  recipe.c  stats.c  fileio.c   │
│  ※ printf / scanf / Win32 API 사용 금지           │
└──────────────────────────────────────────────────┘
```

### 규칙
1. **로직 모듈에서는 화면 입출력을 하지 않습니다.** 값은 매개변수로 받고, 결과는 반환값이나 포인터로 돌려줍니다.
2. 성공·실패는 `ResultCode`로 반환하고, 어떤 메시지를 보여 줄지는 화면 모듈이 결정합니다.
3. 화면 모듈은 로직 모듈의 함수만 호출하며, 구조체 내부를 직접 고치지 않습니다.

```c
/* 잘못된 예: 화면과 로직이 섞여 있어 GUI에서 재사용 불가 */
void addFood(Fridge *f) {
    printf("이름: ");
    scanf("%s", ...);
}

/* 좋은 예: 로직만 담당하므로 콘솔과 GUI 모두에서 그대로 호출 */
ResultCode addFood(Fridge *f, const Food *item);
```

---

## 5. 모듈 구조

하나의 솔루션 안에 **콘솔 프로젝트**와 **GUI 프로젝트** 두 개를 두고, 로직 모듈 소스 파일은 두 프로젝트가 함께 사용합니다.

```
fridge-manager/
├── README.md
├── .gitignore
├── FridgeManager.sln         # 솔루션 파일 (이 파일을 열어서 작업)
├── FridgeConsole/
│   └── FridgeConsole.vcxproj # 콘솔 프로젝트 (필수)
├── FridgeGUI/
│   └── FridgeGUI.vcxproj     # GUI 프로젝트 (도전)
├── src/
│   ├── common.h              # 공통 구조체, 열거형, 상수, ResultCode
│   │
│   │   ── 로직 계층 (두 프로젝트 공통) ──
│   ├── food.h / food.c            # F1 등록, F5 수정·삭제·수량 변경·검색
│   ├── expiry.h / expiry.c        # F2 날짜 계산·D-day·정렬, F4 임박 개수 계산
│   ├── recipe.h / recipe.c        # F3 추천 점수 계산·레시피 추가, F5 자동 차감
│   ├── stats.h / stats.c          # F6 폐기 기록 추가·일괄 폐기·통계 계산
│   ├── fileio.h / fileio.c        # 모든 데이터 파일 저장·불러오기
│   │
│   │   ── 화면 계층 (콘솔 프로젝트, 필수) ──
│   ├── main.c                     # 콘솔 진입점: 데이터 불러오기 → 알림 → 메뉴 → 저장
│   ├── ui_console.h / ui_console.c  # 메뉴·목록·입력 화면 출력
│   ├── util.h / util.c            # 안전한 콘솔 입력, 입력 버퍼 정리
│   │
│   │   ── 화면 계층 (GUI 프로젝트, 도전) ──
│   ├── main_gui.c                 # GUI 진입점 (wWinMain)
│   └── ui_gui.h / ui_gui.c        # 창, 버튼, ListView, 입력 대화상자
└── data/
    ├── fridge.txt            # 식재료 목록
    ├── recipes.txt           # 레시피 목록 (기본 + 사용자 추가)
    └── waste.txt             # 폐기 기록
```

| 프로젝트 | 포함하는 파일 |
|---|---|
| `FridgeConsole` | `main.c`, `ui_console.c`, `util.c` + 로직 모듈 5개 |
| `FridgeGUI` | `main_gui.c`, `ui_gui.c` + 로직 모듈 5개 |

### 모듈별 주요 함수 (초안)

**로직 계층**

| 모듈 | 함수 | 역할 |
|---|---|---|
| `food` | `void initFridge(Fridge *f)` / `void freeFridge(Fridge *f)` | 동적 배열 초기화·해제 |
| | `ResultCode addFood(Fridge *f, const Food *item)` | 식재료 추가 (같은 이름·유통기한이면 수량 합치기) |
| | `ResultCode updateFood(Fridge *f, int idx, const Food *item)` | 식재료 정보 수정 |
| | `ResultCode removeFood(Fridge *f, int idx)` | 식재료 삭제 |
| | `ResultCode useFood(Fridge *f, int idx, int amount)` | 수량 차감, 0이면 삭제 |
| | `int findFood(const Fridge *f, const char *name)` | 이름으로 검색, 없으면 -1 |
| `expiry` | `Date getToday(void)` | 오늘 날짜 반환 |
| | `int daysLeft(Date expire)` | 남은 일수 계산 (지났으면 음수) |
| | `int isValidDate(Date d)` | 날짜 유효성 검사 (윤년, 월별 일수) |
| | `void sortByExpiry(Fridge *f)` | 임박순 정렬 (`qsort`) |
| | `void countAlerts(const Fridge *f, int *expired, int *soon)` | 기한 지난 개수, 2일 이내 개수 계산 |
| `recipe` | `int recommend(const Fridge *f, const RecipeBook *rb, Recommendation *out, int max)` | 추천 결과를 점수순으로 채우고 개수 반환 |
| | `ResultCode cookRecipe(Fridge *f, const Recipe *rc)` | 레시피 재료 자동 차감 (부족하면 `RC_NOT_ENOUGH`) |
| | `ResultCode addRecipe(RecipeBook *rb, const Recipe *rc)` | 사용자 레시피 추가 |
| `stats` | `ResultCode addWaste(WasteList *w, const Food *item, Date today)` | 폐기 기록 추가 |
| | `int discardExpired(Fridge *f, WasteList *w)` | 기한 지난 재료 일괄 폐기, 폐기 개수 반환 |
| | `void computeWasteStats(const WasteList *w, WasteStats *out)` | 통계 계산 |
| `fileio` | `ResultCode loadFridge / saveFridge` | `fridge.txt` 읽기·쓰기 |
| | `ResultCode loadRecipes / saveRecipes` | `recipes.txt` 읽기·쓰기 |
| | `ResultCode loadWaste / saveWaste` | `waste.txt` 읽기·쓰기 |

**화면 계층**

| 모듈 | 함수 | 역할 |
|---|---|---|
| `ui_console` | `void showStartupAlert(const Fridge *f)` | 콘솔 경고 박스 출력 |
| | `void runMainMenu(AppData *app)` | 메인 메뉴 반복 |
| | `void screenAddFood / screenFoodList / screenEditFood ...` | 각 기능 화면 |
| `util` | `int inputInt(const char *msg, int min, int max)` | 범위 검사가 있는 정수 입력 |
| | `int inputDate(const char *msg, Date *d)` | `YYYY-MM-DD` 형식 검사 후 입력 |
| | `void clearInputBuffer(void)` | 입력 버퍼 비우기 |
| `ui_gui` | `void guiShowStartupAlert(const Fridge *f)` | G1: 경고 팝업 |
| | `LRESULT CALLBACK MainWndProc(...)` | G2: 메인 창 메시지 처리 |
| | `void guiRefreshList(HWND list, const Fridge *f)` | G2: ListView 갱신 |
| | `INT_PTR CALLBACK FoodDlgProc(...)` | G3: 등록·수정 대화상자 |
| | `void guiShowRecommend / guiShowStats ...` | G4·G5: 추천·통계 창 |

> 주의: `common.h`와 로직 모듈의 함수 선언(`.h`)은 구현을 시작하기 전에 팀 전체가 확정합니다. 확정 후 변경할 때는 반드시 팀원에게 공유합니다.

---

## 6. 핵심 자료구조

```c
/* common.h */
#ifndef COMMON_H
#define COMMON_H

#define NAME_LEN     30
#define MAX_ING      10
#define ALERT_DAYS   2      /* 임박 기준: 2일 이내 */

typedef enum {
    RC_OK = 0,
    RC_NOT_FOUND,     /* 대상 없음 */
    RC_INVALID,       /* 잘못된 입력값 */
    RC_NOT_ENOUGH,    /* 재료 부족 */
    RC_NO_MEMORY,     /* 메모리 할당 실패 */
    RC_FILE_ERROR     /* 파일 열기·쓰기 실패 */
} ResultCode;

typedef enum { FRIDGE = 0, FREEZER = 1 } Storage;   /* 냉장, 냉동 */

typedef struct {
    int year, month, day;
} Date;

typedef struct {
    char    name[NAME_LEN];
    Date    expire;
    int     quantity;
    Storage storage;
} Food;

typedef struct {
    Food *items;      /* 동적 배열 */
    int   count;
    int   capacity;   /* 가득 차면 realloc으로 2배 확장 */
} Fridge;

typedef struct {
    char name[NAME_LEN];
    char ingredients[MAX_ING][NAME_LEN];
    int  amounts[MAX_ING];    /* 재료별 필요 수량 */
    int  ingCount;
    int  isCustom;            /* 0: 기본 레시피, 1: 사용자 추가 */
} Recipe;

typedef struct {
    Recipe *items;
    int     count;
    int     capacity;
} RecipeBook;

typedef struct {
    int  recipeIdx;           /* RecipeBook 안의 인덱스 */
    int  matched;             /* 보유한 재료 수 */
    int  score;               /* 추천 점수 */
    int  canCook;             /* 1이면 바로 요리 가능 */
    int  missingCount;
    char missing[MAX_ING][NAME_LEN];   /* 부족한 재료 이름 */
} Recommendation;

typedef struct {
    char    name[NAME_LEN];
    int     quantity;
    Storage storage;
    Date    expire;
    Date    discarded;        /* 폐기한 날짜 */
} WasteRecord;

typedef struct {
    WasteRecord *items;
    int          count;
    int          capacity;
} WasteList;

typedef struct {
    int  totalRecords;        /* 폐기 횟수 */
    int  totalQuantity;       /* 폐기한 재료 총 수량 */
    int  fridgeCount;         /* 냉장 재료 폐기 횟수 */
    int  freezerCount;        /* 냉동 재료 폐기 횟수 */
    int  thisMonth;           /* 이번 달 폐기 횟수 */
    char topName[NAME_LEN];   /* 가장 많이 버린 재료 */
    int  topCount;
} WasteStats;

typedef struct {              /* 화면 모듈이 들고 다니는 전체 데이터 */
    Fridge     fridge;
    RecipeBook recipes;
    WasteList  waste;
} AppData;

#endif
```

### 사용하는 C언어 핵심 개념

| 개념 | 활용 위치 |
|---|---|
| 제어문 | 메인 메뉴 반복(`while`, `switch`), 재료 매칭 반복문, D-day 조건 분기, GUI 메시지 분기 |
| 함수 분리(모듈화) | 로직 계층·화면 계층 분리, 기능별 `.c` / `.h` 파일 분리 |
| 배열과 문자열 | 레시피 재료 목록, `strcmp`로 재료 이름 비교, `strstr`로 검색 |
| 포인터 | 구조체를 포인터로 전달해 수정, 결과를 포인터 매개변수로 반환, 콜백 함수 포인터(`qsort`, `WndProc`) |
| 구조체 | `Food`, `Recipe`, `Recommendation`, `WasteRecord`, `WasteStats`, `AppData` |
| 파일 입출력 | `fridge.txt`, `recipes.txt`, `waste.txt` 저장·불러오기 |
| 동적 메모리 할당 | `malloc` / `realloc` / `free`로 목록 크기 자동 확장 |
| 기타 | `time.h` 날짜 계산, `enum` 상태값, `qsort` 정렬, Win32 API(GUI) |

---

## 7. 데이터 파일 형식

- 인코딩: **ANSI(EUC-KR, 코드 페이지 949)**로 저장합니다. (8.3 한글 처리 참고)
- 공백으로 구분된 텍스트 파일이며, 재료 이름에는 공백을 넣지 않습니다. (예: `돼지 고기`는 안 되고 `돼지고기`로 입력)

**`data/fridge.txt`**: `이름 유통기한 수량 보관(0=냉장, 1=냉동)`
```
계란 2026-11-12 6 0
대파 2026-11-10 1 0
돼지고기 2026-12-20 2 1
두부 2026-11-11 1 0
```

**`data/recipes.txt`**: `요리이름 사용자추가여부 재료개수 재료1 수량1 재료2 수량2 ...`
```
계란말이 0 2 계란 3 대파 1
계란볶음밥 0 3 계란 2 밥 1 대파 1
김치찌개 0 4 김치 1 돼지고기 1 두부 1 대파 1
된장찌개 0 4 된장 1 두부 1 애호박 1 양파 1
```

**`data/waste.txt`**: `이름 수량 보관 유통기한 폐기일`
```
우유 1 0 2026-11-05 2026-11-08
상추 1 0 2026-11-03 2026-11-08
```

---

## 8. 개발 환경 설정

### 8.1 설치 (팀원 전원)

1. **Visual Studio 2022 Community** 설치
2. 설치 화면의 워크로드에서 **"C++를 사용한 데스크톱 개발"** 선택
   (C 컴파일러와 Win32 API 개발 도구가 함께 설치됩니다.)

### 8.2 솔루션과 프로젝트 만들기 (팀장이 1회 진행 후 GitHub에 올림)

**콘솔 프로젝트 (FridgeConsole)**
1. 새 프로젝트 → **빈 프로젝트** → 이름 `FridgeConsole`, 솔루션 이름 `FridgeManager`
2. 솔루션 탐색기 → "소스 파일" 우클릭 → 추가 → 기존 항목 → `src/`의 콘솔용 파일과 로직 모듈 `.c` 파일 추가
3. "헤더 파일"에 `src/`의 `.h` 파일 추가
4. 파일 확장자는 반드시 **`.c`**로 합니다. (`.cpp`이면 C++로 컴파일됩니다.)

**GUI 프로젝트 (FridgeGUI, 도전 단계에서 추가)**
1. 솔루션 우클릭 → 추가 → 새 프로젝트 → **빈 프로젝트** → 이름 `FridgeGUI`
2. `main_gui.c`, `ui_gui.c`와 로직 모듈 `.c` 파일을 기존 항목으로 추가
   (로직 파일은 복사하지 않고 `src/`의 같은 파일을 두 프로젝트가 함께 참조합니다.)
3. 프로젝트 속성 → 링커 → 시스템 → **하위 시스템: Windows (/SUBSYSTEM:WINDOWS)**
4. 프로젝트 속성 → 고급 → **문자 집합: 유니코드 문자 집합 사용**
5. 공용 컨트롤(ListView) 라이브러리는 `ui_gui.c` 상단에서 연결합니다.
   ```c
   #include <windows.h>
   #include <commctrl.h>
   #pragma comment(lib, "comctl32.lib")
   ```

**두 프로젝트 공통 설정** (구성: **모든 구성**, 플랫폼: **모든 플랫폼**으로 선택 후 설정)
- C/C++ → 전처리기 → 전처리기 정의에 `_CRT_SECURE_NO_WARNINGS` 추가
  (`scanf`, `fopen` 사용 시 나오는 보안 경고 C4996 해제)
- 디버깅 → 작업 디렉터리를 **`$(SolutionDir)`**로 변경
  (실행할 때 솔루션 폴더의 `data/`를 찾을 수 있도록 함)

### 8.3 한글 처리

한국어 Windows의 콘솔은 기본적으로 **코드 페이지 949(EUC-KR)**를 사용합니다. 아래처럼 맞추면 별도 설정 없이 콘솔에서 한글 입력과 출력이 정상 동작합니다.

| 파일 종류 | 저장 인코딩 | 이유 |
|---|---|---|
| 소스 파일 (`.c`, `.h`) | **유니코드(서명 있는 UTF-8)** | GitHub에서도 한글 주석이 깨지지 않음. Visual Studio가 문자열을 실행 시 949로 자동 변환 |
| 데이터 파일 (`.txt`) | **ANSI (EUC-KR)** | 콘솔 입력과 같은 인코딩이라 `strcmp` 비교가 정확함 |
| `README.md` | UTF-8 | GitHub 표시용 |

- 소스 파일 인코딩 변경: 파일 → 다른 이름으로 저장 → 저장 버튼 옆 화살표 → **인코딩하여 저장** → "유니코드(서명 있음)(UTF-8) - 코드 페이지 65001"
- 데이터 파일 인코딩 변경: 메모장에서 열기 → 다른 이름으로 저장 → 인코딩 **ANSI**
- 프로젝트 속성에 `/utf-8` 옵션은 **추가하지 않습니다.** (추가하면 콘솔 한글 출력이 깨집니다.)

**GUI에서의 한글**: Win32 API는 유니코드(`wchar_t`) 함수(`...W`)를 사용합니다. 데이터 파일에서 읽은 문자열은 화면에 표시하기 전에 변환합니다.
```c
/* ui_gui.c: ANSI(949) → 유니코드 변환 (로직 데이터를 화면에 표시할 때) */
void ansiToWide(const char *src, wchar_t *dst, int dstLen) {
    MultiByteToWideChar(CP_ACP, 0, src, -1, dst, dstLen);
}

/* ui_gui.c: 유니코드 → ANSI(949) 변환 (입력창 값을 로직에 넘길 때) */
void wideToAnsi(const wchar_t *src, char *dst, int dstLen) {
    WideCharToMultiByte(CP_ACP, 0, src, -1, dst, dstLen, NULL, NULL);
}
```

### 8.4 `.gitignore`

Visual Studio가 만드는 빌드 결과물과 개인 설정 파일은 GitHub에 올리지 않습니다.
```
.vs/
x64/
Debug/
Release/
*.user
*.suo
*.obj
*.pdb
*.ilk
*.exe
```
> `.sln`과 `.vcxproj`, `.vcxproj.filters` 파일은 반드시 커밋해야 팀원이 같은 프로젝트 구성을 받을 수 있습니다.

---

## 9. 빌드 및 실행 방법

1. GitHub 저장소를 클론한 뒤 `FridgeManager.sln`을 Visual Studio로 엽니다.
2. 솔루션 탐색기에서 실행할 프로젝트를 우클릭 → **시작 프로젝트로 설정**
   - 콘솔 버전: `FridgeConsole`
   - GUI 버전: `FridgeGUI`
3. `Ctrl + F5` (디버깅하지 않고 시작) 또는 `F5` (디버깅 시작)

> 실행 시 솔루션 폴더 안에 `data/` 폴더가 있어야 합니다. 작업 디렉터리 설정(8.2)을 확인하세요.

---

## 10. 실행 화면 예시

### 10.1 콘솔 버전 (필수)

**프로그램 시작 시 알림**
```
╔══════════════════════════════════════╗
║           [ 유통기한 알림 ]           ║
╠══════════════════════════════════════╣
║  오늘 날짜: 2026-11-09               ║
║                                      ║
║  유통기한 지난 재료: 1개             ║
║   - 우유 (D+1)                       ║
║  2일 이내 임박 재료: 3개             ║
║   - 대파 (D-1)                       ║
║   - 두부 (D-2)                       ║
║   - 계란 (D-2)                       ║
╚══════════════════════════════════════╝
      아무 키나 누르면 메뉴로 이동합니다
```

**메인 메뉴**
```
============ 냉장고 관리 ============
 1. 식재료 등록
 2. 식재료 목록 (임박순)
 3. 식재료 수정 / 삭제 / 수량 변경
 4. 오늘 뭐 먹지? (요리 추천)
 5. 레시피 추가
 6. 기한 지난 재료 일괄 폐기
 7. 폐기 기록 통계
 0. 저장 후 종료
=====================================
선택 >
```

**식재료 목록 (임박순)**
```
번호  이름        수량  보관   유통기한     남은 기간
-------------------------------------------------
 1   우유         1   냉장   2026-11-08   D+1 (지남)
 2   대파         1   냉장   2026-11-10   D-1
 3   두부         1   냉장   2026-11-11   D-2
 4   계란         6   냉장   2026-11-11   D-2
 5   돼지고기     2   냉동   2026-12-20   D-41
```

**요리 추천**
```
===== 오늘의 추천 요리 =====
1. 계란말이     [재료 2/2] [바로 가능]  (임박 재료: 계란, 대파)
2. 계란볶음밥   [재료 2/3]  부족: 밥
3. 김치찌개     [재료 3/4]  부족: 김치

만들 요리 번호를 입력하세요 (0: 돌아가기) > 1
계란 3개, 대파 1개를 사용합니다. 만들까요? (y/n) > y
[완료] 재료가 차감되었습니다. (계란 6 → 3, 대파 1 → 0, 대파 삭제)
```

**폐기 기록 통계**
```
=========== 폐기 기록 통계 ===========
총 폐기 횟수       : 7회 (재료 9개)
냉장 / 냉동 비율   : 냉장 86% / 냉동 14%
가장 많이 버린 재료: 우유 (3회)
이번 달 폐기 횟수  : 4회
--------------------------------------
[팁] 냉장 재료 폐기가 많아요. 장보기 양을 조금 줄여 보세요!
```

### 10.2 GUI 버전 (도전)

**G1. 시작 알림 팝업**
```
┌─ 유통기한 알림 ────────────────────┐
│                                    │
│  (!)  유통기한 지난 재료: 1개       │
│       2일 이내 임박 재료: 3개       │
│                                    │
│                         [ 확인 ]   │
└────────────────────────────────────┘
```
```c
/* G1 구현 예시 (ui_gui.c) */
void guiShowStartupAlert(const Fridge *f) {
    int expired, soon;
    wchar_t msg[200];
    countAlerts(f, &expired, &soon);          /* 로직 함수 재사용 */
    if (expired == 0 && soon == 0) return;
    swprintf(msg, 200, L"유통기한 지난 재료: %d개\n2일 이내 임박 재료: %d개", expired, soon);
    MessageBoxW(NULL, msg, L"유통기한 알림", MB_OK | MB_ICONWARNING);
}
```

**G2~G3. 메인 창**
```
┌─ 냉장고 관리 ─────────────────────────────────────────────┐
│ ┌──────────────────────────────────────────┐ ┌──────────┐ │
│ │ 이름     │ 수량 │ 보관 │ 유통기한   │ 남은 │ │ 식재료   │ │
│ │──────────┼──────┼──────┼────────────┼──────│ │  등록    │ │
│ │ 우유     │  1   │ 냉장 │ 2026-11-08 │ D+1  │ ├──────────┤ │
│ │ 대파     │  1   │ 냉장 │ 2026-11-10 │ D-1  │ │ 수정     │ │
│ │ 두부     │  1   │ 냉장 │ 2026-11-11 │ D-2  │ ├──────────┤ │
│ │ 계란     │  6   │ 냉장 │ 2026-11-11 │ D-2  │ │ 삭제     │ │
│ │ 돼지고기 │  2   │ 냉동 │ 2026-12-20 │ D-41 │ ├──────────┤ │
│ │          │      │      │            │      │ │ 요리 추천│ │
│ │          │      │      │            │      │ ├──────────┤ │
│ │          │      │      │            │      │ │ 폐기 통계│ │
│ └──────────────────────────────────────────┘ └──────────┘ │
│  전체 5개 · 기한 지남 1개 · 임박 3개                       │
└────────────────────────────────────────────────────────────┘
```

**G3. 등록 대화상자**
```
┌─ 식재료 등록 ───────────────────┐
│  이름      [ 계란             ] │
│  유통기한  [ 2026-11-20       ] │
│  수량      [ 6   ]              │
│  보관      (●) 냉장  ( ) 냉동   │
│                                 │
│              [ 등록 ] [ 취소 ]  │
└─────────────────────────────────┘
```

**G4. 요리 추천 창**
```
┌─ 오늘 뭐 먹지? ────────────────────────────────┐
│  요리         │ 재료  │ 상태                     │
│───────────────┼───────┼──────────────────────────│
│  계란말이     │ 2/2   │ 바로 가능 (임박 재료)    │
│  계란볶음밥   │ 2/3   │ 부족: 밥                 │
│  김치찌개     │ 3/4   │ 부족: 김치               │
│                                                 │
│        [ 요리하기 ]  [ 레시피 추가 ]  [ 닫기 ]   │
└─────────────────────────────────────────────────┘
```

---

## 11. 팀원 및 역할

5명이 **개발 모듈 1개 이상 + 개발 외 업무 1개**씩 맡도록 나눕니다. 모든 팀원이 직접 코드를 작성하고 커밋하며, 문서·발표·테스트·일정 관리도 한 명에게 몰리지 않게 분배합니다.

### 11.1 역할 요약

| 이름 | 학번 | 역할 | 담당 모듈 / 업무 |
|---|---|---|---|
| | | 팀장 · 통합 · 일정 관리 | `common.h`, `main.c`, `ui_console`(메뉴 뼈대), 솔루션 구성, GitHub 관리·최종 병합 / GUI: `main_gui.c`, G2 메인 창 |
| | | 구현 · 테스트 | `food` (F1, F5), `util` / GUI: G3 등록·수정 대화상자 |
| | | 구현 · 발표 자료 | `expiry` (F2, F4) / GUI: G1 시작 알림 팝업, G5 임박 색상 표시 |
| | | 구현 · 발표 | `recipe` (F3, 자동 차감), 기본 레시피 데이터 / GUI: G4 요리 추천 창 |
| | | 구현 · 문서 | `stats` (F6), `fileio` / GUI: G5 폐기 통계 창 |

### 11.2 팀원별 상세 업무

**팀원 1: 팀장 · 통합 · 일정 관리**
- 개발 (필수)
  - `common.h` 구조체·열거형·상수 초안 작성, 팀 회의에서 확정
  - `main.c`: 데이터 불러오기 → 시작 알림 → 메인 메뉴 → 저장 후 종료 흐름
  - `ui_console.c`의 메인 메뉴와 화면 전환 뼈대 작성
- 개발 (GUI 도전): `main_gui.c`(wWinMain), G2 메인 창(ListView, 버튼 배치)
- 개발 외 업무
  - Visual Studio 솔루션·프로젝트 구성, `.gitignore` 작성, GitHub 저장소 개설 및 팀원 Collaborator 초대
  - Pull Request 검토 후 `main` 병합, 버전 태그(v1.0, v2.0) 지정
  - 주간 회의 진행, WBS 일정 관리, 회의록 작성
- 산출물: 통합된 소스 코드, 회의록, 일정표

**팀원 2: 구현 · 테스트**
- 개발 (필수)
  - `food.c`: 식재료 등록, 수정, 삭제, 수량 변경, 검색, 동적 배열 확장(`realloc`)
  - `util.c`: `inputInt`, `inputDate`, `clearInputBuffer` 등 잘못된 입력을 막는 입력 함수
  - 식재료 등록·수정·삭제 콘솔 화면 함수
- 개발 (GUI 도전): G3 식재료 등록·수정 대화상자
- 개발 외 업무
  - 테스트 시나리오 작성 (정상 입력, 잘못된 날짜, 빈 목록, 수량 0, 파일 없음 등)
  - 통합 테스트 진행 및 버그를 GitHub Issues에 등록
- 산출물: 테스트 시나리오, 테스트 결과서

**팀원 3: 구현 · 발표 자료**
- 개발 (필수)
  - `expiry.c`: 오늘 날짜 가져오기, D-day 계산, 날짜 유효성 검사(윤년·월별 일수), 임박순 정렬(`qsort`), 임박 개수 계산
  - 시작 알림 박스 화면, 식재료 목록(임박순) 콘솔 화면 함수
- 개발 (GUI 도전): G1 시작 알림 팝업, G5 목록에서 임박 재료 색상 강조
- 개발 외 업무
  - 발표 자료(PPT) 제작: 프로젝트 소개, 설계 구조도, 기능 시연 순서, 역할 분담, 느낀 점
  - 프로그램 흐름도 작성 (제안보고서 3.3 항목)
- 산출물: 발표 자료, 흐름도

**팀원 4: 구현 · 발표**
- 개발 (필수)
  - `recipe.c`: 추천 점수 계산, 부족 재료 목록, 추천 정렬, 요리 시 재료 자동 차감, 사용자 레시피 추가
  - 기본 레시피 데이터(`recipes.txt`) 20개 이상 작성
  - 요리 추천·레시피 추가 콘솔 화면 함수
- 개발 (GUI 도전): G4 요리 추천 창, 레시피 추가 창
- 개발 외 업무
  - 최종 발표 및 데모 시연 진행 (시연용 샘플 데이터 준비 포함)
  - 예상 질문과 답변 정리
- 산출물: 레시피 데이터 파일, 시연 시나리오

**팀원 5: 구현 · 문서**
- 개발 (필수)
  - `fileio.c`: 식재료·레시피·폐기 기록 파일 저장 및 불러오기, 파일이 없을 때 처리
  - `stats.c`: 폐기 기록 추가, 기한 지난 재료 일괄 폐기, 통계 계산
  - 폐기 통계 콘솔 화면 함수
- 개발 (GUI 도전): G5 폐기 통계 창
- 개발 외 업무
  - 제안보고서, 결과보고서 작성 및 취합 (각 팀원 담당 부분을 받아서 정리)
  - README 관리
- 산출물: 제안보고서, 결과보고서, README

### 11.3 다 함께 하는 업무

| 업무 | 방식 |
|---|---|
| `common.h`·함수 선언 확정 | 착수 첫 주 회의에서 5명이 함께 결정 |
| 코드 리뷰 | 다른 팀원의 Pull Request를 1명 이상 확인 후 병합 |
| 보고서 담당 부분 작성 | 각자 맡은 모듈의 설계·구현 내용은 본인이 작성해 문서 담당에게 전달 |
| 발표 준비 | 각자 맡은 기능의 시연 방법과 설명을 발표 담당에게 전달, 리허설 1회 이상 함께 진행 |

### 11.4 모듈 간 의존 관계 (작업 순서 참고)

```
common.h (팀원 1, 전원 확정)
   │
   ├── food (팀원 2) ──────┐
   ├── expiry (팀원 3) ────┤
   ├── fileio (팀원 5) ────┼──> recipe (팀원 4: food 함수 사용)
   │                       └──> stats (팀원 5: food, expiry 함수 사용)
   │
   └── main.c / ui_console (팀원 1: 모든 모듈 연결)
```

- `food`, `expiry`, `fileio`는 서로 독립적이라 **동시에** 개발할 수 있습니다.
- `recipe`와 `stats`는 `food` 함수를 사용하므로, `food.h` 선언이 확정되면 바로 시작하고 실제 연결은 `food` 구현 후 테스트합니다.
- `ui_console.c`는 여러 명이 수정하므로, 파일 안에서 담당자별 구역을 주석으로 나눠 충돌을 줄입니다.
  ```c
  /* ===== [팀원 2] 식재료 등록·수정·삭제 화면 ===== */
  /* ===== [팀원 3] 시작 알림·목록 화면 ===== */
  /* ===== [팀원 4] 요리 추천 화면 ===== */
  /* ===== [팀원 5] 폐기 통계 화면 ===== */
  ```

---

## 12. 협업 규칙

### 브랜치
- `main`: 항상 실행 가능한 코드만 유지
- `feature/모듈명`: 각자 담당 모듈 개발 (예: `feature/recipe`, `feature/gui-alert`)
- 작업이 끝나면 Pull Request를 올리고, 팀원 1명 이상 확인 후 `main`에 병합

### 진행 순서
1. `common.h`와 로직 모듈 `.h` 함수 선언 확정
2. 로직 모듈 구현 + 콘솔 화면 구현 → **콘솔 버전 v1.0 태그**
3. 콘솔 버전이 완성된 뒤 GUI G1 → G5 순서로 진행 → **GUI 버전 v2.0 태그**

### 커밋 메시지
```
[태그] 작업 내용

예)
[feat] 요리 추천 점수 계산 기능 추가
[feat] GUI 시작 알림 팝업 구현
[fix] D-day 계산 시 월말 오류 수정
[docs] README 실행 방법 수정
[refactor] 입력 함수 util 모듈로 분리
```

### 코딩 규칙
- **로직 모듈(`food`, `expiry`, `recipe`, `stats`, `fileio`)에서는 `printf`, `scanf`, Win32 API를 쓰지 않습니다.**
- 함수 이름은 camelCase (`addFood`), 상수는 대문자 (`MAX_ING`)
- 구조체를 함수에 넘길 때는 포인터 사용, 수정하지 않으면 `const` 붙이기
- `malloc` / `realloc` 후 반드시 `NULL` 검사, 종료 전 `free`
- 소스 파일은 UTF-8(서명 있음), 데이터 파일은 ANSI로 저장
- `.vcxproj` 파일을 수정(파일 추가·속성 변경)했다면 커밋 메시지에 꼭 적어 충돌을 줄입니다.

---

## 13. 참고 자료
- 수업 교재: `교재명 / 저자 / 출판사`
- C 표준 라이브러리 `time.h`: https://en.cppreference.com/w/c/chrono
- `qsort`: https://en.cppreference.com/w/c/algorithm/qsort
- Visual Studio에서 C 프로그램 만들기 (Microsoft Learn): https://learn.microsoft.com/ko-kr/cpp/build/walkthrough-compile-a-c-program-on-the-command-line
- Win32 API 시작하기 (Microsoft Learn): https://learn.microsoft.com/ko-kr/windows/win32/learnwin32/learn-to-program-for-windows
- ListView 컨트롤 (Microsoft Learn): https://learn.microsoft.com/ko-kr/windows/win32/controls/list-view-controls-overview
