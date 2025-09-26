

### 3.1 기본 자료형
p37 기본 타입 정수형 표

p39 닷넷의 기본 타입에 대한 C#의 별칭 (표 3.2)
```c#
[Fact]
public void 닷넷_기본타입에대한_씨샵별칭()
{
    System.Int32 n1 = 50;
    System.Int32 n2;
    n2 = 100;

    System.Int32 sum = n1 + n2;

    Assert.Equal(150, sum);
}
```
- int를 쓸지 System.Int32를 쓸지 개발자의 자유
	- 보통은 편리하다는 이유로 C# 별칭을 사용

p39 

**3.1.2 실수형 기본 타입** (표 3.3)

> 크기순 : float < double < decimal


p40 

**3.1.3 문자열 기본 타입** (표3.4)

> char, string만 있다.

- `\t` : 탭
- `\n` : 개행문자 (줄바꿈)
- `\` : 이스케이프 시퀀스라 부름
	- `@` 문자를 문자열 앞에 붙이면 내부에 있는 `\`를 이스케이프 시퀀스라 간주하지 않고 순수한 문자로 취급함
	- ex. `string text = @"\tHello\nWorld"` ▶️ 출력 `\tHello\nWorld`

p44

**3.1.4 불린(boolean)형 기본 타입** (표3.5)
> bool 타입만 있다.


>[!note] C#은 정적 타입 언어이기 때문에 반드시 자료형을 명시해야 한다

```c#
int n = 50;
string text = "Hello World";
bool isNumeric = false;

char ch = '\\'; //▶️ 출력시 \ 하나만 출력
char ch = '\u2023'; // 유니코드 문자의 번호를 16진수로 명시
char ch = '\t'; // TAB을 문자로 표현
char ch = '\n'; // 개행(NEW LINE)을 문자로 표현

float f = 5.2f;
double d= 10.5;
decimal money = 200.099m;
```

🔖**추가로 알아보면 좋은거**
- 유니코드, UTF-8, UTF-16, UTF-32
- 자주 사용되는 이스케이프 시퀀스


### 3.2 형변환
- Type Conversion, Casting
- `암시적 변환`과 `명시적 변환`으로 구분

3.2.1 암시적 변환 
```c#
byte b = 250;
short s = b;
```

3.2.2 명시적 변환
```c#
ushort u = 65;
char c = (char)u;
```

> 암시적 변환시 오버플로우나 언더플로우 조심

### 3.3 기본 문법 요소

3.3.1 예약어, 키워드 
- sbyte, byte, short, ushort, int, uint, long, ulong
- float, double, decimal
- char, string
- bool

3.3.2 식별자 
굵게 표시한 부분이 명명할 수 있는 식별자에 해당하며 자유럽게 변환 가능하다

```c# hl:3,5,7,9
using System;

namspace ConsoleApp1
{
	class Program 
	{
		static void Main(string[] args) 
		{
			string text = "Hello World";
			Console.WriteLine(text);
		}
	}
}
```


3.3.3 리터럴 
- 한글로 "문자 상의, 문자 그대로의"의미
- 프로그래밍 언어에서 마땅히 번역할 단어가 없어 영어 발음 그대로 쓰는것이 보통 
- 굳이 번역하자면 "소스 코드에 포함된 값"

```c#
string test = "Hello World";
int n = 5;
char ch = 'N';
bool result = true;
```
- 오른쪽에 있는 값이 모두 리터럴에 해당 


3.3.4.1 두 가지 저장소: 스택과 힙

> C/C++ 언어 등으로 프로그램을 만들면 메모리 할당과 해제를 반드시 쌍으로 맞춰야만 했다. 
> 반면 C# 프로그램이 동작하는 관리 환경의 경우 개발자는 오직 할당만 하고 해제는 관리 환경 내의 특정 구성 요소가 담당한다. 그것을 `가비지 수집기`라고 한다

> 이때 string은 `참조 형식`에 속한다


3.3.4.2 값 형식을 가리키는 변수
"값 형식"을 가리키는 변순의 경우 "값" 자체가 스택 영역에 할당되고 변수는 그 메모리를 가르키는 프로그램 내의 식별자다.

> [!note] 값 형식
> sbyte, byte, char, short, ushort, int, uint, long, ulong, float, double, decimal, bool 이 있다


3.3.4.3 참조 형식을 가리키는 변수
🔖p52 다시 읽기

> 초기화 되지 않은 모든 참조형 변수는 null 값을 가진다
> 참조형 변수가 더는 사용되지 않음을 명시하기 위해 null을 할당하기도 한다


3.3.4.4 기본값 

```c#
bool result; // false
int n; // 0
string text // null
```

3.3.5 상수
- `const` 예약어를 사용
	- 값이 절대 바뀌지 않는다는 의미에서 상수(constant)
- 컴파일러 입장에서 상수는 반드시 컴파일 할때 값이 결정돼야 한다

```c#
int n = Math.Max(0, 5); // 프로그램 실행시 n의 값이 결정된다 

const int maxN = Math.Max(0, 5); // Max 호출 후 값이 결정되어서 컴파일 오류 발생💩
```
컴파일러 입장에서 어떤 값을 대입해야 할지 컴파일 시점에 알 수 x


3.3.6 연산자, 문장 부호 
- `;` : 한 구문의 끝을 컴파일러에게 알리는 문장 부호
- `=` : 대입 연산자(assignment operator)


### 3.4 배열

```c#
int[] products = new int[5]; 
string[] names = new string[1000];
```

> 배열의 값을 별도로 힙에 할당하고 있다. 즉, 배열도 참조 형식에 속한다.

> 배열은 동일한 타입의 공간을 지정된 수만큼 메모리에 연속적으로 할당한다

```c#
int[] products = new int[5] {1, 2, 3, 4, 5}; // 배열의 요소 개수 지정

또는 

int[] products = new int[] {1, 2, 3, 4, 5}; // 배열의 요소 개수 미지정
```


```c#
string text = "Hello World";
char ch1 = text[0]; // "H"
char ch2 = text[1]; // "e"
```


3.4.1 다차원 배열

```c#
int[,] arr2 = new int[10, 5]; // 2차원 배열 , 10행 5열
int[,,] arr3 = new int[8, 3, 10]; // 3차원 배열
```
- 인덱스는 0번부터

```c#
int[,] arr2 = new int[2, 3] { // 2행 3열
	{1, 2, 3},
	{4, 5, 6}
};

int[,,] arr3 = new int[2, 3, 4] {
	{
		{1, 2, 3, 4},
		{5, 6, 7, 8},
		{9, 10, 11, 12}
	},
	{
		{13, 14, 15, 16},
		{17, 18, 19, 20},
		{21, 22, 23, 24}
	},
}
```


3.4.2 가변 배열 (jagged array)
```c#
int[][] arr = new int[5][]; // 2차원 가변 배열
arr[0] = new int[10];
arr[1] = new int[9];
arr[2] = new int[8];
arr[3] = new int[3];
arr[4] = new int[5];
```

다차원 배열이 콤마를 이용해 차수를 구분하는 반면, 가변 배열은 각 차수마다 대괄호를 사용한다
// 그림3.12 (p63)


### 3.5 제어문

3.5.1 선택문(selection statements)

3.5.1.1 관계 연산자, 논리 연산자 
// 표 사진 찍기

베타적 논리합 연산자 (XOR): `^` 
부정 연산자 (NOT) : `!`

3.5.1.2 if 문
```text
if(조건식)
	구문;
	
또는

if(조건식) 구문;

또는 

if(조건식)
{
	//
}
else if(조건식)
{
	//
}
else 
{
	//
}
```


> 삼항 연산자도 가능 
> ▶️ (조건식) ? 표현식1 : 표현식2;


3.5.1.3 switch 문

```text
switch(인스턴스)
{
	case 상수식1:
		구문;
		break;
	// ..
	default: 
		구문
		break;		
}
```


3.5.2 반복문 (p74)
- 증감 연산자 : `++`, `--`
	- 후위 표기법 : `n++;`
	- 전위 표기법 : `++n;`

```c#
int n = 50;
Console.WriteLine(n++); // 50 

n = 50;
Console.WriteLine(++n); // 51
```

```c#
int n = 50;
int result;

result = n++; // 50

n = 50;
result = ++n; // 51

n = 50;
result = n--; // 50

n = 50;
result = --n; // 49
```

표3.12 복합 대입 연산자 
- `+=`, `-=`, `*=`, `/=`, `%=`

```c#
int n = 50;
n += 5; // 55

n = 50;
n -= 5; // 45

n = 50;
n *= 5; // 250

n = 50;
n /= 5; // 10

n = 50;
n %= 5; // 0
```

3.5.2.2 for 문 (p78)

```text
for(초기화; 조건식; 반복문)
	구문;
	
또는 

for(초기화; 조건식; 반복문) 구문;
```

```c#
int n;
for(n = 1; n <= 9; n++) 
{
	Console.WriteLine(n); // 1 ~ 9까지 출력
}
```

> 특이하게도 초기화, 조건식, 반복문이 모두 선택 사항이라서 각 부분에 해당하는 코드를 제거 가능

```c#
int n = 1;
for(; n <= 9; n++) 
	Console.WriteLine(n);
	
또는

int n = 1;
for(; ; n++) 
{
	if(n > 9) break;
	
	Console.WriteLine(n);
}	

또는 
int n = 1;
for(; ;)
{
	if(n > 9) break; 
	
	Console.WriteLine(n);
	n ++;
}
	
```


3.5.2.3 중첩 루프 (생략)

3.5.2.4 foreach 문🔖

```text
foreach(표현식 요소의 자료형 변수명 in 표현식)
	구문;
	
또는

foreach(표현식 요소의 자료형 변수명 in 표현식) 구문;
```


```c#
int[] arr = new int[] {1, 2, 3, 4, 5};

foreach(int el in arr) 
{
	Console.WriteLine(el);
}
```


3.5.2.5 while 문 (동일해서 생략)


3.5.3 점프문(jump statements) 🧐
- `break`, `continue`, `goto`, `return`, `throw` 예약어가 있다