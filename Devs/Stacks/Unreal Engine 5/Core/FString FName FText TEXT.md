---
tags: []
related: 
---

## 1. FString, FName, FText, TEXT 매크로 개요

언리얼 엔진은 용도에 따라 최적화된 세 가지 문자열 클래스와 하나의 매크로를 제공합니다.

|구분|FString|FName|FText|TEXT|
|---|---|---|---|---|
|**설명**|동적 문자열 (가변)|불변 식별자|현지화 텍스트|L""과 동일한 와이드 문자열 리터럴 매크로|
|**주요 특징**|- 문자열 수정, 조합, 검색 등 다양한 조작 가능 - 런타임에 생성 및 변경|- 변경 불가능 (Immutable) - 해시 테이블에 저장되어 빠른 비교 가능 - 대소문자 구분 안 함 (Case-insensitive)|- 텍스트 현지화(Localization) 지원 - 런타임에 최종 텍스트 결정|- 컴파일 타임에 유니코드(UTF-16) 문자열로 변환|
|**주요 용도**|- 플레이어 이름, 채팅 메시지 등 동적으로 변하는 텍스트 - 파일 경로, 로그 메시지 등 조작이 필요한 문자열|- 액터 이름, 태그, 소켓 이름, 머티리얼 파라미터 이름 등 고유 식별자|- UI 텍스트 (버튼, 라벨 등) - 플레이어에게 보여지는 모든 텍스트|- 모든 문자열 리터럴을 감싸는 데 사용|
|**메모리**|문자열 길이에 따라 동적 할당 (상대적으로 무거움)|고유 문자열을 중앙 테이블에 저장 후 포인터와 해시 값만 가짐 (가벼움)|현지화 키와 데이터를 포함 (상대적으로 무거움)|컴파일된 바이너리에 포함|
|**성능**|조작은 유연하지만, 비교 연산은 상대적으로 느림|비교 연산이 매우 빠름 (해시 값 비교)|현지화 조회 과정에서 오버헤드 발생 가능|성능에 직접적인 영향은 없으나, 모든 문자열 리터럴에 사용해야 함|

---

## 2. 각 타입별 상세 설명

### 가. FString (동적 문자열)

`FString`은 C++의 `std::string`과 유사한 동적 문자열 클래스입니다. 문자열을 합치거나, 자르거나, 특정 문자를 찾는 등 다양한 조작이 필요할 때 사용됩니다.

- **헤더**: `Containers/UnrealString.h`
- **특징**:
    - **가변성(Mutable)**: 언제든지 내용을 수정할 수 있습니다.
    - **풍부한 API**: 문자열 조작을 위한 다양한 함수(`Append`, `Contains`, `Split`, `ToUpper` 등)를 제공합니다.
    - **연산자 오버로딩**: `+`, `+=`, `==` 등의 연산자를 통해 직관적인 문자열 처리가 가능합니다.

### 나. FName (고유 식별자)

`FName`은 애셋이나 오브젝트의 이름처럼 게임 내에서 고유 식별자로 사용되는 불변 문자열입니다.

- **헤더**: `UObject/NameTypes.h`
- **특징**:
    - **불변성(Immutable)**: 한번 생성되면 내용을 변경할 수 없습니다.
    - **빠른 비교**: 내부적으로 해시 테이블을 사용하여 문자열 비교가 매우 빠릅니다. 실제로는 문자열 자체가 아닌, 해시 값을 비교하기 때문입니다.
    - **메모리 효율성**: 동일한 문자열은 메모리에 단 한 번만 저장되고, `FName`은 이를 가리키는 방식으로 동작하여 메모리 중복을 방지합니다.
    - **대소문자 미구분**: 기본적으로 대소문자를 구분하지 않고 비교합니다. (예: "MyActor"와 "myactor"는 같은 `FName`으로 취급)

### 다. FText (현지화 텍스트)

`FText`는 사용자에게 표시되는 모든 텍스트를 처리하기 위한 클래스입니다. 다국어 지원(현지화) 기능이 내장되어 있어, 게임을 여러 언어로 출시할 때 필수적입니다.

- **헤더**: `Internationalization/Text.h`
- **특징**:
    - **현지화 지원**: 동일한 코드로 다양한 언어의 텍스트를 표시할 수 있습니다.
    - **안전한 포매팅**: 숫자나 날짜 등을 문자열에 포함시킬 때, 언어권 문화에 맞는 형식(예: 1,000 vs 1.000)으로 안전하게 변환합니다.
    - **불변성(Immutable)**: 런타임에 직접적인 문자열 조작이 불가능하며, 이는 현지화 시스템의 안정성을 보장하기 위함입니다.

### 라. TEXT 매크로

`TEXT()`는 문자열 리터럴을 유니코드(UTF-16, `TCHAR` 타입)로 인코딩해주는 매크로입니다. 언리얼 엔진은 내부적으로 유니코드를 사용하므로, 코드에 사용되는 모든 문자열 리터럴은 이 매크로로 감싸야 합니다.

- **예시**: `TEXT("Hello World")`
- **동작**: 컴파일러 설정에 따라 `L"Hello World"` (와이드 문자열) 또는 `"Hello World"` (ANSI 문자열)로 변환됩니다. 언리얼 엔진은 기본적으로 유니코드를 사용하므로 거의 항상 `L""` 로 변환됩니다.

---

## 3. 모든 것에 의한 초기화 방법

각 문자열 타입은 다른 타입이나 리터럴로부터 다양한 방법으로 초기화될 수 있습니다.

### 가. FString 초기화

```cpp
// 1. TEXT 매크로를 사용한 리터럴로 초기화
FString StrFromLiteral = TEXT("This is a string literal.");

// 2. 다른 FString으로 초기화 (복사 생성자)
FString StrFromOtherFString = StrFromLiteral;

// 3. FName으로 초기화
FName MyName = FName("MyObjectName");
FString StrFromFName = MyName.ToString();

// 4. FText로 초기화
FText MyText = FText::FromString(TEXT("UI Text"));
FString StrFromFText = MyText.ToString();

// 5. C-Style 문자열 (const TCHAR*)로 초기화
const TCHAR* CStyleString = TEXT("C-Style");
FString StrFromCStyle = CStyleString;

// 6. FString::Printf를 사용한 포맷팅 초기화
int32 Score = 100;
FString StrFromPrintf = FString::Printf(TEXT("Player Score: %d"), Score);

// 7. 빈 문자열로 초기화
FString EmptyStr1;
FString EmptyStr2 = TEXT("");
```

### 나. FName 초기화

```cpp
// 1. TEXT 매크로를 사용한 리터럴로 초기화 (생성자)
FName NameFromLiteral = FName(TEXT("PlayerCharacter"));

// 2. 다른 FName으로 초기화 (복사 생성자)
FName NameFromOtherFName = NameFromLiteral;

// 3. FString으로 초기화
FString StrForName = TEXT("MyActorTag");
FName NameFromFString = FName(*StrForName); // FString을 TCHAR*로 변환하여 생성자에 전달

// 4. C-Style 문자열 (const TCHAR*)로 초기화
const TCHAR* CStyleName = TEXT("SocketName");
FName NameFromCStyle = FName(CStyleName);

// 5. FName()을 사용한 빈 이름(None)으로 초기화
FName NoneName; // 기본 생성자는 "None"
FName NoneName2 = FName(); // "None"
FName NoneName3 = NAME_None; // "None"을 나타내는 전역 상수

// 6. 숫자를 추가하여 고유 이름 생성
FName UniqueName = FName(TEXT("Item"), 123); // 결과: "Item_123"
```

### 다. FText 초기화

```cpp
// 1. TEXT 매크로를 사용한 리터럴로 초기화 (FText::FromString 사용)
// 주의: FromString은 현지화되지 않는 텍스트에 사용해야 합니다.
FText TextFromLiteral = FText::FromString(TEXT("This is not for localization."));

// 2. 현지화를 위한 매크로 사용 (가장 일반적인 방법)
// NSLOCTEXT(네임스페이스, 키, 기본값_리터럴)
FText LocalizedText = NSLOCTEXT("MyGameNamespace", "WelcomeMessage", "Welcome to My Game!");
// LOCTEXT(네임스페이스, 키) - 네임스페이스가 이미 정의된 경우 사용
#define LOCTEXT_NAMESPACE "MyGameNamespace"
FText LocalizedText2 = LOCTEXT("Greeting", "Hello!");
#undef LOCTEXT_NAMESPACE

// 3. 다른 FText로 초기화 (복사 생성자)
FText TextFromOtherFText = LocalizedText;

// 4. FString으로 초기화
FString StrForText = TEXT("Dynamic Text");
FText TextFromFString = FText::FromString(StrForText);

// 5. 숫자를 텍스트로 변환 (현지화 문화권에 맞게 변환됨)
int32 Health = 100;
FText TextFromNumber = FText::AsNumber(Health);

// 6. FText::Format을 사용한 포맷팅 초기화 (안전한 방식)
FText FormatPattern = LOCTEXT("PlayerStatusFormat", "{PlayerName} has {Score} points.");
FFormatNamedArguments Args;
Args.Add(TEXT("PlayerName"), FText::FromString(TEXT("Gemini")));
Args.Add(TEXT("Score"), FText::AsNumber(10000));
FText FormattedText = FText::Format(FormatPattern, Args);

// 빈 텍스트로 초기화
FText EmptyText1;
FText EmptyText2 = FText::GetEmpty();
```

---

## 4. 핵심 요약 및 권장 사용법

1. **사용자에게 보여줄 텍스트는 무조건 `FText`를 사용하세요.** 현지화는 물론, 숫자나 날짜 형식도 안전하게 처리해줍니다. `NSLOCTEXT` 또는 `LOCTEXT` 매크로 사용을 습관화하는 것이 좋습니다.
2. **오브젝트나 애셋의 고유 식별자, 태그 등에는 `FName`을 사용하세요.** 메모리 효율이 좋고 비교가 매우 빠르므로 성능에 유리합니다.
3. **문자열을 동적으로 조작하거나 로그 출력, 파일 경로 처리 등 내부적인 문자열 처리가 필요할 때는 `FString`을 사용하세요.**
4. **코드에 들어가는 모든 문자열 리터럴은 `TEXT()` 매크로로 감싸주세요.** 이는 언리얼 엔진의 유니코드 환경과의 호환성을 위해 필수적입니다.

이 세 가지 문자열 클래스의 용도를 명확히 구분하고 올바른 상황에 사용하는 것이 언리얼 엔진에서 효율적이고 안정적인 코드를 작성하는 핵심입니다.
