<div align="center">

# 🛒 TypeScript 전자상거래 타입 모델링 실습

**정적 타입으로 런타임 오류를 미리 막는 전자상거래 데이터 모델 · 타입 가드 · 제네릭 저장소**

![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-5FA04E?style=for-the-badge&logo=nodedotjs&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white)
![Python](https://img.shields.io/badge/Python_3.13-3776AB?style=for-the-badge&logo=python&logoColor=white)
![uv](https://img.shields.io/badge/uv-DE5FE9?style=for-the-badge&logo=uv&logoColor=white)

</div>

<br>

## 📖 프로젝트 소개

VS Code에서 **Jupyter Notebook + tslab 커널**로 TypeScript 코드를 셀 단위로 작성하고, 실행 결과를 바로 확인하며 만든 실습 프로젝트입니다.

전자상거래 도메인(회원 · 상품)을 바탕으로 아래 내용을 구현했습니다.

| 노트북 | 주제 | 핵심 키워드 |
| --- | --- | --- |
| 📓 [`01_data_modeling.ipynb`](./01_data_modeling.ipynb) | 데이터 모델링 & 타입 안전 집계 함수 | `interface` `type` `리터럴 유니언` `선택적 프로퍼티` `??` |
| 📓 [`02_generics_typeguard.ipynb`](./02_generics_typeguard.ipynb) | 타입 가드 분기 & 제네릭 인메모리 저장소 | `in 좁히기` `never` `제네릭 제약` `private` |

> ✅ 두 노트북 모두 **`any`를 사용하지 않았고**, 모든 셀의 실행 결과가 파일에 저장되어 있습니다.

<br>

## 📌 목차

1. [문항 1 · 데이터 모델링 및 집계 유틸리티](#1%EF%B8%8F⃣-문항-1--데이터-모델링-및-집계-유틸리티)
2. [문항 2 · 타입 가드와 제네릭 저장소](#2%EF%B8%8F⃣-문항-2--타입-가드와-제네릭-저장소)
3. [타입 안정성 포인트](#-타입-안정성-포인트)
4. [기술 스택](#-기술-스택)
5. [폴더 구조](#-폴더-구조)
6. [실행 방법](#%EF%B8%8F-실행-방법)
7. [트러블슈팅](#-트러블슈팅)

<br>

---

## 1️⃣ 문항 1 · 데이터 모델링 및 집계 유틸리티

### 🧩 데이터 모델

리터럴 유니언 타입으로 **허용되는 값 자체를 제한**하고, 인터페이스로 객체 구조를 정의했습니다.

```typescript
type UserRole = "admin" | "customer" | "guest";
type Category = "electronics" | "clothing" | "books";

interface User {
  id: number;
  name: string;
  email: string;
  role: UserRole;        // 문자열이 아니라 3가지 역할 중 하나만 허용
  isActive: boolean;
}

interface Product {
  id: number;
  title: string;
  price: number;
  category: Category;
  stock: number;
  discountRate?: number; // 선택적 프로퍼티 (할인 없는 상품은 생략)
}
```

### 🔍 권한 기반 필터 함수

```typescript
function filterUsersByRole(users: User[], targetRole: UserRole): User[] {
  return users.filter((user) => user.role === targetRole);
}
```

- `targetRole`을 `UserRole`로 지정해서 `filterUsersByRole(users, "manager")` 같은 잘못된 호출은 **실행 전에 차단**됩니다.
- 콜백의 `user`는 따로 타입을 적지 않아도 `User`로 자동 추론됩니다.

### 💰 총 결제 금액 계산 함수

```typescript
function calculateTotalPrice(products: Product[]): number {
  let total = 0;

  for (const product of products) {
    const rate = product.discountRate ?? 0;   // undefined면 0 → 정가 적용
    total += product.price * (1 - rate);
  }
  return total;
}
```

`discountRate`는 `number | undefined`라서 그대로 계산하면 `NaN`이 될 수 있습니다. `?? 0`으로 기본값을 지정해 **할인 없는 상품도 안전하게 계산**합니다.

### 📊 실행 결과

| 상품 | 가격 | 할인율 | 결제 금액 |
| --- | ---: | :---: | ---: |
| 컴퓨터 | 10,000원 | 0.2 | 8,000원 |
| 티셔츠 | 20,000원 | 없음 → 0 | 20,000원 |
| **합계** | | | **28,000원** |

```
총 결제 금액: 28,000원
```

<br>

---

## 2️⃣ 문항 2 · 타입 가드와 제네릭 저장소

### 🔀 `in` 연산자로 타입 좁히기

`User | Product` 유니언을 받아, 각 타입에만 있는 프로퍼티로 분기합니다.

```typescript
function renderItemSummary(item: User | Product): string {
  if ("email" in item) {
    return `[회원] ${item.name} (${item.role})`;       // item: User
  } else if ("title" in item) {
    return `[상품] ${item.title} - ${item.price}원`;   // item: Product
  } else {
    const exhaustiveCheck: never = item;              // 처리하지 않은 타입이 있으면 컴파일 오류
    return exhaustiveCheck;
  }
}
```

```
[회원] 고객 (customer)
[상품] 컴퓨터 - 10000원
```

> 💡 **Exhaustive 검사**: 마지막 `else`에서 `item`은 `never` 타입이어야 합니다. 나중에 유니언에 새 타입(예: `Order`)이 추가되면 이 줄에서 컴파일 오류가 나서 **분기 누락을 바로 알 수 있습니다.**

### 🗄️ 제네릭 인메모리 저장소

```typescript
interface Identifiable {
  id: number;
}

class InMemoryStorage<T extends Identifiable> {
  private items: T[] = [];

  add(item: T): void {
    if (this.getById(item.id)) {
      console.warn(`id ${item.id}는 이미 존재하여 추가하지 않습니다`);
      return;
    }
    this.items.push(item);
  }

  getById(id: number): T | undefined {
    return this.items.find((item) => item.id === id);
  }

  getAll(): T[] {
    return [...this.items];
  }
}
```

| 설계 | 이유 |
| --- | --- |
| `T extends Identifiable` | 모든 항목에 `id`가 있음을 보장 → `item.id`를 안전하게 사용 |
| `private items` | 외부에서 내부 배열에 직접 접근·수정하지 못하도록 캡슐화 |
| `getAll()`이 복사본 반환 | 반환된 배열을 수정해도 원본 데이터는 그대로 유지 |
| `add()` 중복 id 검사 | 고유 키 규칙을 저장소 차원에서 지킴 |
| `getById()` → `T \| undefined` | 없는 id 조회 가능성을 타입으로 드러내 호출부에서 처리하도록 유도 |

### 🧪 다형성 인스턴스 검증

하나의 클래스로 **회원 저장소와 상품 저장소**를 모두 만들 수 있습니다.

```typescript
const userStorage = new InMemoryStorage<User>();
const productStorage = new InMemoryStorage<Product>();

userStorage.getById(2);     // 반환 타입: User | undefined
productStorage.getById(1);  // 반환 타입: Product | undefined
```

```
id 2는 이미 존재하여 추가하지 않습니다
id 2 회원: 고객
id 99 회원: undefined
id 1 상품: 컴퓨터
```

<br>

---

## 🛡️ 타입 안정성 포인트

| 포인트 | 적용 내용 |
| --- | --- |
| 🚫 **`any` 미사용** | 모든 변수 · 매개변수 · 반환값에 명시적 타입 지정 |
| 🎯 **리터럴 유니언** | `role: "manager"`처럼 정의되지 않은 값은 컴파일 단계에서 차단<br>`Type '"manager"' is not assignable to type 'UserRole'.` |
| ❓ **선택적 프로퍼티 방어** | `discountRate ?? 0`으로 `undefined` → `NaN` 계산 오류 방지<br>(`\|\|`는 `0`도 대체하므로 `??` 사용) |
| 🔀 **타입 좁히기** | `in` 연산자로 별도 래퍼 함수 없이 블록별 타입 자동 인식 |
| ✅ **Exhaustive 검사** | `never` 타입으로 유니언 분기 누락을 컴파일 단계에서 감지 |
| 🧬 **제네릭 제약** | `T extends Identifiable`로 저장 가능한 타입을 제한하면서도 재사용성 확보 |

<br>

## 🧰 기술 스택

| 분류 | 도구 | 버전 | 용도 |
| --- | --- | --- | --- |
| 언어 | TypeScript | tslab 내장 | 실습 언어 |
| 런타임 | Node.js | v24 | tslab 실행 |
| 커널 | tslab | 1.0.22 | Jupyter용 TypeScript 커널 |
| 노트북 | Jupyter | ≥ 1.1.1 | 커널 관리 및 노트북 실행 |
| 패키지 관리 | uv / npm | - | Python · Node 의존성 관리 |
| 에디터 | VS Code + Jupyter 확장 | - | 대화형 노트북 실행 |

<br>

## 📁 폴더 구조

```
frontend-ts/
├── 📓 01_data_modeling.ipynb        # 문항 1: 데이터 모델링 & 집계 유틸리티
├── 📓 02_generics_typeguard.ipynb   # 문항 2: 타입 가드 & 제네릭 저장소
├── 📄 README.md
├── 📦 package.json                  # npm 의존성 (tslab, typescript)
├── 📦 package-lock.json
├── 🐍 pyproject.toml                # Python 의존성 (jupyter)
├── 🐍 uv.lock
├── 🐍 .python-version
└── 🙈 .gitignore                    # .venv, node_modules 제외
```

<br>

## ▶️ 실행 방법

### 사전 준비

- [Node.js](https://nodejs.org/) v18 이상
- [uv](https://docs.astral.sh/uv/)
- [VS Code](https://code.visualstudio.com/) + [Jupyter 확장](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.jupyter)

### 설치

```bash
# 1. 저장소 클론
git clone https://github.com/yeji-jang/AX_frontend_TypeScript_practice.git
cd AX_frontend_TypeScript_practice

# 2. Python 가상환경 생성 + jupyter 설치
uv sync

# 3. npm 의존성 설치
npm install
npm install -g tslab

# 4. tslab 커널 등록
tslab install --python=.venv/Scripts/python.exe   # Windows
# tslab install --python=.venv/bin/python         # macOS / Linux

# 5. 커널 등록 확인 (tslab, jslab이 보이면 성공)
uv run jupyter kernelspec list
```

### 노트북 실행

1. VS Code에서 `.ipynb` 파일을 엽니다.
2. 오른쪽 위 **Select Kernel** → **Jupyter Kernel...** → **TypeScript** 를 선택합니다.
3. **Restart** → **Run All** 순서로 전체 셀을 실행합니다.

<br>

## 🔧 트러블슈팅

<details>
<summary><b>TypeScript 코드를 실행했는데 <code>SyntaxError: invalid syntax</code>가 나와요</b></summary>
<br>

Python 커널로 실행되고 있는 상태입니다. 오른쪽 위 커널 선택에서 **Jupyter Kernel... → TypeScript**로 바꿔 주세요.
</details>

<details>
<summary><b><code>Cannot find name 'User'</code> 오류가 나요</b></summary>
<br>

노트북은 **실행한 셀의 내용만** 기억합니다. 타입을 정의한 위쪽 셀을 먼저 실행하거나 **Run All**을 사용하세요.
노트북끼리는 커널이 따로 실행되므로, 02번 노트북에도 모델 정의가 따로 들어 있습니다.
</details>

<details>
<summary><b><code>Unexpected pending rebuildTimer</code> 오류가 나요</b></summary>
<br>

코드 문제가 아니라 tslab 내부 오류입니다. 커널 재시작 직후 셀을 너무 빠르게 연속 실행하면 발생할 수 있습니다.

1. **Restart** 후 2~3초 기다리기
2. 첫 번째 코드 셀만 먼저 실행
3. **Run All** 실행
</details>

<details>
<summary><b>커널 목록에 TypeScript가 보이지 않아요</b></summary>
<br>

- `uv run jupyter kernelspec list`로 `tslab`이 등록되어 있는지 확인하세요.
- `Ctrl+Shift+P` → **Developer: Reload Window**로 VS Code를 새로고침하세요.
</details>

<br>

<div align="center">

**Made with TypeScript 💙 & tslab**

</div>
