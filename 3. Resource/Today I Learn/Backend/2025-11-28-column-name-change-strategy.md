# 📌 오늘 배운 것 (Today I Learned) 
## 날짜 
`2025년 11월 28일`

## 카테고리
`#CSharp`, `#Strategy Pattern`, `#Abstraction`, `#Refactoring`

## 주제: 
### 1. 문제 상황 또는 학습 배경
- CSVReader를 통해 파일 읽었을때 컬럼 헤드 네이밍이 변경되야 하는 상황이 발생 
- hw 장비를 직접 설치해 수집을 하다보니, 직접 변경하기 어렵고, 그래서 서버단에서 처리하기로 한 것으로 추측
- 그런데 그 수가 늘어날 수록 if 조건문, while문과 같은 형태로 계속 수정이 발생하여 유지보수/관리가 어려워짐
- 이를 개선하고 사용하는 컬럼에 대해 익숙해지기 위해 리팩터링을 권장함 

> [!note] 251128
> 비슷한 이름을 가진 컬럼이 여러개 있는데, 거기에 우선순위에 따라 사용하는게 달라져서 아래와 같이 Name Change 메서드가 생기게 되었다는 듯 함.

```cs
 private async Task<List<string>> ChangeStandardFieldName(string shipKey, string[] filedHeaders, DateTime dateTime)
 {
     var result = filedHeaders.ToList(); // csv 컬럼 헤더 정보
     var headerChangeList = await loggerDataFieldNameChangerRepository.GetByShipKeyAsync(shipKey);

     foreach (var item in headerChangeList)
     {
         if (item.START_DATE <= dateTime && item.END_DATE > dateTime && result.Any(d => d == item.WRONG_FIELD_NAME))
         {
             var dataIndex = result.IndexOf(item.WRONG_FIELD_NAME);
             result[dataIndex] = item.FIELD_NAME;
         }
     }

	 // 경우1
	while (result.Any(s => Regex.IsMatch(s, "ME_CYL[0-9]_PCO_OUTLET_TEMP")))
 {
     var dataIndex = result.FindIndex(s => Regex.IsMatch(s, "ME_CYL[0-9]_PCO_OUTLET_TEMP"));
     var tempArray = result[dataIndex].Split('_');
     tempArray[0] = "ME1";
     result[dataIndex] = string.Join("_", tempArray);
 }
	
	// 경우2
	while (result.Any(s => Regex.IsMatch(s, "ME[0-9]_CYL_EXH_GAS_OUTLET_TEMP")))
	 {
	     var dataIndex = result.FindIndex(s => Regex.IsMatch(s, "ME[0-9]_CYL_EXH_GAS_OUTLET_TEMP"));
	     var tempArray = result[dataIndex].Split('_');
	     var number = tempArray[0].Last();
	     tempArray[0] = "ME1";
	     tempArray[1] = "CYL" + number;
	     result[dataIndex] = string.Join("_", tempArray);
	 }
	 
	 // 경우3
	 if (result.Contains("NO1_GE_TC_EXH_INLET_TEMP"))
	{
	    var dataIndex = result.IndexOf("NO1_GE_TC_EXH_INLET_TEMP");
	    result[dataIndex] = "GE1_TC1_EXH_INLET_TEMP";
	}
	
	// 경우4
	 if (!result.Contains("TIME_STAMP") && result.Contains("UTC"))
	 {
	     var dataIndex = result.IndexOf("UTC");
	     result[dataIndex] = "TIME_STAMP";
	 }
	 
	// 경우5
	if (result.Contains("ME1_RPM") && result.Contains("ME1_RPM_ECC"))
	{
	    var dataIndex = result.IndexOf("ME1_RPM_ECC");
	    var dataIndex2 = result.IndexOf("ME1_RPM");
	    result[dataIndex] = "ME1_RPM";
	    result[dataIndex2] = "ME1_RPM_ECC";
	}
}
```


### 2. 핵심 내용 / 개념 정리

`LOGGER_DATA_FIELD_NAME_CHANGER 테이블`
- 금일 기준 총 4건의 데이터만 존재
- 고유 키 값과 시작/종료일 날짜를 기반으로 컬럼명을 고친다.

>[!note] 타입 컬럼 정보를 추가, 5개 경우에 대한 전략 패턴을 생성해 둔 다음에 디비 조회하면서 초기화해버리면 되지 않나 싶다.
- 아니면 .. 이미 전략 5개를 초기화해서 컬렉션에 담은 다음에 타입에 따라 전략을 선택해, 주입하여 변경하면 되지 않나 싶다.

## Gemini 답변

고객님의 접근 방식은 **매우 훌륭합니다!** 👍

**DB 데이터로 규칙을 관리**하고, **규칙 타입에 따라 적절한 구현체(Strategy)**를 생성하여 **합성(Composition)**을 통해 파이프라인으로 처리하는 방식은, 이 레거시 코드를 **확장성과 유연성**을 갖춘 현대적인 구조로 리팩터링하는 **가장 이상적인 방법**입니다.

이 접근 방식을 구체화하고, 세 가지 규칙을 데이터베이스로 어떻게 옮겨 관리할 수 있는지 설명해 드릴게요.

---

## 1. 📊 DB 테이블 구조 및 규칙 정의

하드코딩된 조건문들을 $\text{DB}$ 테이블로 옮기기 위해, 모든 규칙을 아우르는 **단일 테이블**을 설계합니다. 핵심은 각 규칙이 **어떤 타입으로 동작할지** 지정하는 필드입니다.

### $\text{HeaderRewriteRule}$ 테이블 설계 (예시)

|**필드명**|**데이터 타입**|**설명**|
|---|---|---|
|**`RULE_TYPE`**|$\text{String}$|**규칙의 동작 방식**을 정의하는 키 (예: `REGEX`, `SINGLE_REPLACE`, `DOUBLE_SWAP`).|
|**`SHIP_KEY`**|$\text{String}$|적용 선박 키 ($\text{NULL}$이면 모든 선박에 적용).|
|**`START_DATE`**|$\text{DateTime}$|규칙 적용 시작 날짜 ($\text{NULL}$이면 항상 적용).|
|**`WRONG_PATTERN`**|$\text{String}$|변경 전 $\text{Key}$ 또는 **정규식 패턴** ($\text{ME\_CYL\[0-9\]\_...}$ 또는 $\text{NO3\_GE\_TC\_...}$).|
|**`TARGET_KEY_1`**|$\text{String}$|변경 후 목표 $\text{Key}$ (또는 $\text{Regex}$ 치환 $\text{Pattern}$).|
|**`WRONG_PATTERN_2`**|$\text{String}$|$\text{DOUBLE\_SWAP}$ 시 두 번째 잘못된 $\text{Key}$ (선택 사항).|
|**`TARGET_KEY_2`**|$\text{String}$|$\text{DOUBLE\_SWAP}$ 시 두 번째 목표 $\text{Key}$ (선택 사항).|

## 2. 🧪 규칙 타입별 $\text{DB}$ 데이터 매핑

기존의 하드코딩된 로직을 위 테이블에 저장하면 다음과 같이 깔끔하게 정리됩니다.

|**RULE_TYPE**|**WRONG_PATTERN**|**TARGET_KEY_1**|**WRONG_PATTERN_2**|**TARGET_KEY_2**|
|---|---|---|---|---|
|**`REGEX`**|`ME_CYL[0-9]_PCO_OUTLET_TEMP`|`ME1`|$\text{NULL}$|$\text{NULL}$|
|**`SINGLE_REPLACE`**|`NO3_GE_TC_EXH_INLET_TEMP`|`GE3_TC1_EXH_INLET_TEMP`|$\text{NULL}$|$\text{NULL}$|
|**`DOUBLE_SWAP`**|`ME1_RPM`|`ME1_RPM_ECC`|`ME1_RPM_ECC`|`ME1_RPM`|

## 3. 🛡️ 구현체 생성 및 합성 (전략 패턴과 팩토리 패턴)

### A. 1단계: 규칙 구현체 정의 (전략 패턴)

모든 규칙은 $\text{DB}$에서 가져온 데이터 (`HeaderRuleData`)를 기반으로 동작하는 $\text{IHeaderRewriteRule}$ 인터페이스를 구현합니다.

C#

```
public interface IHeaderRewriteRule {
    bool Apply(List<string> headers, HeaderRuleData data);
}

// 1. 정규식 규칙 구현체
public class RegexRewriteRule : IHeaderRewriteRule { /* ... Regex 로직 구현 ... */ }

// 2. 단건 교체 규칙 구현체
public class SingleReplaceRule : IHeaderRewriteRule { /* ... Contains/IndexOf 로직 구현 ... */ }

// 3. 더블 스왑 규칙 구현체
public class DoubleSwapRule : IHeaderRewriteRule { /* ... Contains 2개 로직 구현 ... */ }
```

### B. 2단계: 규칙 팩토리 (Factory Pattern)

$\text{DB}$에서 조회한 `HeaderRuleData` 객체의 `RULE_TYPE` 필드를 보고, **적절한 구현체를 생성**하는 팩토리를 만듭니다.

C#

```
public class HeaderRuleFactory
{
    public IHeaderRewriteRule CreateRule(HeaderRuleData data)
    {
        return data.RULE_TYPE switch
        {
            "REGEX" => new RegexRewriteRule(),
            "SINGLE_REPLACE" => new SingleReplaceRule(),
            "DOUBLE_SWAP" => new DoubleSwapRule(),
            _ => throw new ArgumentException($"Unknown rule type: {data.RULE_TYPE}")
        };
    }
}
```

### C. 3단계: 파이프라인 합성 (Composition)

최종 $\text{HeaderRewriteService}$는 다음 단계를 거칩니다.

1. **Repository 호출:** $\text{DB}$에서 `HeaderRuleData` 목록을 가져옵니다.
    
2. **Factory 사용:** 목록의 각 데이터(`HeaderRuleData`)에 대해 $\text{Factory}$를 사용해 `IHeaderRewriteRule` 구현체를 생성합니다.
    
3. **파이프라인 실행:** 생성된 모든 구현체를 컬렉션으로 모아 $\text{Apply}$ 메서드를 순서대로 호출합니다.
    

**이 구조를 사용하면, 새로운 유형의 헤더 변경 규칙이 생겨도 $\text{DB}$에 $\text{RULE_TYPE}$과 데이터를 추가하고, 해당 타입에 맞는 $\text{IHeaderRewriteRule}$ 구현체만 만들면 됩니다. 기존의 서비스 코드는 전혀 건드릴 필요가 없습니다.**