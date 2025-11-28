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
	 
	// 경우 5
	if (!result.Contains("ME1_RPM") && result.Contains("ME_RPM_ECC"))
{
    var dataIndex = result.IndexOf("ME_RPM_ECC");
    result[dataIndex] = "ME1_RPM";
}
}
```


### 2. 핵심 내용 / 개념 정리

`LOGGER_DATA_FIELD_NAME_CHANGER 테이블`
- 금일 기준 총 4건의 데이터만 존재
- 고유 키 값과 시작/종료일 날짜를 기반으로 컬럼명을 고친다.

