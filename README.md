# 🛒 TypeScript 전자상거래 타입 모델링 실습

![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-v24-5FA04E?style=flat-square&logo=nodedotjs&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-tslab-F37626?style=flat-square&logo=jupyter&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.13-3776AB?style=flat-square&logo=python&logoColor=white)
![uv](https://img.shields.io/badge/uv-package%20manager-DE5FE9?style=flat-square)

> VS Code + Jupyter Notebook + **tslab** 커널로 셀 단위로 타입을 정의하고 실행 결과를 바로 확인하며,
> 전자상거래 도메인의 데이터 모델링 · 타입 가드 · 제네릭 설계를 실습합니다.

<br>

## 📌 목차

- [학습 목표](#-학습-목표)
- [과제 구성](#-과제-구성)
- [기술 스택](#-기술-스택)
- [폴더 구조](#-폴더-구조)
- [환경 설정](#%EF%B8%8F-환경-설정)
- [실행 방법](#%EF%B8%8F-실행-방법)
- [제출 시 주의사항](#-제출-시-주의사항)

<br>

## 🎯 학습 목표

- 변수, 배열, 객체, 함수의 입출력에 정확한 타입을 지정하고 **`any` 없이** 타입 안정성을 확보한다.
- `interface`와 `type` 별칭의 차이를 이해하고 도메인에 맞는 데이터 구조를 설계한다.
- 유니언 타입과 **타입 가드**로 여러 형태의 데이터를 런타임 오류 없이 분기 처리한다.
- `class`와 **제네릭**을 결합해 재사용 가능한 공통 로직을 구현한다.

<br>

## 📚 과제 구성

### 1️⃣ `01_data_modeling.ipynb` · 회원·상품 데이터 모델링 및 집계 유틸리티

| 구분 | 내용 | 핵심 개념 |
| :---: | --- | --- |
| 필수 | `UserRole`, `Category` 타입 별칭과 `User`, `Product` 인터페이스 정의 | 문자열 리터럴 유니언, 선택적 프로퍼티 `?` |
| 필수 | `users: User[]`, `products: Product[]` 더미 데이터 생성 및 검증 | 배열 타입, 타입 검사 |
| 심화 | `filterUsersByRole(users, targetRole): User[]` | 함수 매개변수·반환 타입 |
| 심화 | `calculateTotalPrice(products): number` | `undefined` 안전 처리 |

### 2️⃣ `02_generics_typeguard.ipynb` · 타입 가드 분기 처리 및 제네릭 저장소

| 구분 | 내용 | 핵심 개념 |
| :---: | --- | --- |
| 필수 | `renderItemSummary(item: User \| Product): string` | `in` 연산자 타입 좁히기 |
| 심화 | `Identifiable` 인터페이스 선언 | 제네릭 제약 조건 |
| 심화 | `InMemoryStorage<T extends Identifiable>` 클래스 | `private` 캡슐화, 제네릭 클래스 |
| 심화 | `add` / `getById` / `getAll` 메서드, `User`·`Product` 인스턴스 검증 | 타입 추론, 다형성 |

<br>

## 🧰 기술 스택

| 분류 | 도구 | 버전 | 용도 |
| --- | --- | --- | --- |
| 언어 | TypeScript | tslab 내장 | 실습 언어 |
| 런타임 | Node.js | v24 | tslab 실행 |
| 커널 | tslab | 1.0.22 | Jupyter용 TypeScript 커널 |
| 노트북 | Jupyter | ≥ 1.1.1 | 커널 관리 및 노트북 실행 |
| Python | Python / uv | 3.13 | Jupyter 설치 및 가상환경 관리 |
| 에디터 | VS Code + Jupyter 확장 | - | 대화형 노트북 실행 |

<br>

## 📁 폴더 구조

```
frontend-ts/
├── 📓 01_data_modeling.ipynb        # 문항 1
├── 📓 02_generics_typeguard.ipynb   # 문항 2
├── 📄 README.md
├── 📦 package.json                  # npm 의존성 (tslab, typescript)
├── 📦 package-lock.json
├── 🐍 pyproject.toml                # Python 의존성 (jupyter)
├── 🐍 uv.lock
├── 🐍 .python-version
├── 🙈 .gitignore
├── .venv/                           # (제외) Python 가상환경
└── node_modules/                    # (제외) npm 패키지
```

<br>

## ⚙️ 환경 설정

### 사전 준비

- [Node.js](https://nodejs.org/) (v18 이상)
- [uv](https://docs.astral.sh/uv/)
- [VS Code](https://code.visualstudio.com/) + [Jupyter 확장](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.jupyter)

### 설치

```bash
# 1. Python 가상환경 생성 및 jupyter 설치
uv sync

# 2. npm 의존성 설치
npm install

# 3. tslab 전역 설치 (커널이 PATH에서 tslab 명령을 찾기 때문에 필요)
npm install -g tslab

# 4. tslab 커널 등록
tslab install --python=.venv/Scripts/python.exe   # Windows
# tslab install --python=.venv/bin/python         # macOS / Linux

# 5. 커널 등록 확인 (목록에 tslab, jslab이 보이면 성공)
uv run jupyter kernelspec list
```

<br>

## ▶️ 실행 방법

1. VS Code에서 `.ipynb` 파일을 엽니다.
2. 오른쪽 위 **Select Kernel** → **Jupyter Kernel...** → **TypeScript** 를 선택합니다.
3. **Run All** 로 전체 셀을 실행합니다.

> 💡 커널이 목록에 보이지 않으면 VS Code를 완전히 종료한 뒤 다시 실행하세요.

<br>

## 📦 제출 시 주의사항

- [ ] 모든 셀을 실행한 뒤 **출력 결과가 저장된 상태**로 `.ipynb` 파일을 저장했는가?
- [ ] 두 노트북이 프로젝트 **루트 경로**에 있는가?
- [ ] `.zip`으로 압축할 때 **`.venv/`를 제외**했는가? (필수)
- [ ] `node_modules/`도 제외했는가? (권장, `npm install`로 다시 설치 가능)
