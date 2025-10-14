

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



## 4장 C# 객체지향 문법

p95
```c#
// 예제 4.1
class Book 
{
	string Title;
	decimal ISBN13;
	string Contents;
	string Author;
	int PageCount;
}
```

p96 
```c#
// 예제 4.2

using System;

namespace ConsoleApp1
{
	class Program
	{
		static void Main(string[] args) 
		{
			Book gulliver = new Book();
			
			// public 접근제어자가 없으면 client에서 속성 호출 불가
			string title = gulliver.Title;
			decimal isbn13 = gulliver.ISBN13;
			string contetns = gulliver.Contetns;
			string author = gulliver.Author;
			int pageCount = gulliver.PageCount;
		}
	}
	
	class Book
	{
		public string Title;
		public decimal ISBN13;
		public string Contents;
		public string Author;
		public int PageCount;
		
		public void Open()
		{
			Console.WriteLine("Book is opened");
		}
		
		public void Close()
		{
			Console.WriteLine("Book is closed");
		}
	}
}
```


p106

> 타입(class) = 속성 (field) + 행위 (method)


4.1.3 생성자

```c#
class Person 
{
	string name;
	
	public Person()
	{
		name = "홍길동";
		Console.WriteLine("생성자 호출");
	}
}
```

> [!info] 
> 생성자를 명시적으로 정의하지 않았다면 C# 컴파일러는 일부러 다음과 같은 빈 생성자를 클래스에 집어넣고 컴파일한다.

```c#
public Person() // 기본 생성자(default constructor)
{
}
```

> [!note] 주의할 점
> 개발자가 명시적으로 생성자를 정의한 경우 컴파일러는 기본 생성자를 추가하지 x
> ▶️ 파라미터를 주입받는 생성자를 정의해두면 기본 생성자는 생성되지 x 🔖


p109 예제 4.5 생성자를 여러 개 사용
```c#
class Book
{
	public string Title;
	public decimal ISBN13;
	public string Author;
	

	public Book(string title)
	{
		Title = title;
	}	
	
	public Book(string title, decimal isbn13)
	{
		Title = title;
		ISBN13 = isbn13;
	}
	
	public Book(string title, decimal isbn13, string author)
	{
		Title = title;
		ISBN13 = isbn13;
		Author = author;
	}
	
	// ..
}
```


4.1.4 종료자 

```c#
class Book
{
	public Book() // 기본 생성자
	{
	}
	
	~Book() // 종료자
	{
		// 자원 해제..
	}
}
```

>[!info] 
>종료자를 사용할 때는 정의하기에 앞서 신중하게 고민하고 판단해야 한다. 왜냐하면 GC 입장에서는 일반 참조 객체와는 달리 종료자가 정의된 클래스의 객체를 관리하려면 더 복잡한 과정을 거쳐야 하므로 성능 면에서 부하를 줄 수 있기 때문이다. 
>
>✅ 이 경우 기준은 하나다. 닷넷이 관리하지 안는 시스템 자원을 얻는 경우에만 종료자를 정의하라. 이런 경우는 아직 한번도 없었으므로 아직까지는 종료자를 정의해야 할 이유가 아무것도 없다. 나중에 네이티브 프로극램과의 협업을 다룰 때 이 부분에 대해 다시 이야기할 것이다. 그떄까지는 종료가 단지 메서드의 특별한 유형이라는 점만 기억해 두고 넘어가자.


4.1.5 정적 멤버, 인스턴스 멤버
- 어떤 타입을 실체화한 객체를 `인스턴스`라고 함
	- 또는 new 연산자를 거쳐 메모리에 할당된 객체라고 할 수 있다. 
- 바로 그 객체와 관련된 멤버를 인스턴스 멤버(instance member)라고 하며, 지금까지 설명한 필드, 메서드, 생성자는 모두 여기에 속함

```c#
class Person
{
	public string _name; // 인스턴스 필드
	
	public Person(string name) // 인스턴스 생성자
	{
		_name = name;
	}
	
	public void OutputYourName() // 인스턴스 메서드
	{
		Console.WriteLine(_name);
	}
}
```


4.1.5.1 정적 필드 
```c#
class Person
{
	static public int CountOfInstance; // static 예약어로 정적 필드 선언
	private string _name; // 인스턴스 필드
	
	public Person(string name) 
	{
		CountOfInstance ++;
		_name = name;
	}
}
```


4.1.5.2 정적 메서드 

```c#
class Person
{
	static int CountOfInstance; // private 정적 필드
	public string _name; // 인스턴스 필드
	
	public Person(string name) 
	{
		CountOfInstance ++;
		_name = name;
	}
	
	static public void OutputCount() // public 정적 메서드
	{
		Console.WriteLine(CountOfInstance); // 정적 메서드에서 정적 필드에 접근
	}
}
```
- 참고로 정적 메서드 안에서는 인스턴스 멤버에 접근할 수 없다는 특징이 있다.
	- ✅ 당연하게도 인스턴스 멤버 초기화가 안되었어서 할당된게 없으니 .. 
	- 정적 메서드는 new로 할당된 객체가 없는 상태에서도 호출되는 메서드라는 점을 생각하기

**Main 메서드**
> [!info] 진입점 (entry point)이라고도 하는데, C#은 다음과 같은 약속을 따르는 메서드를 최초로 실행될 메서드라고 규정한다
1. 메서드 이름은 반드시 Main
2. 정적 메서드 여야 한다
3. Main 메서드가 정의된 클래스의 이름은 제한이 없다. 하지만 2개 이상의 클래스에서 Main 메서드를 정의하고 있다면 C# 컴파일러에게 클래스를 지정해야 함 
4. Main 메서드의 반환값은 void 또는 int 만 허용됨 
5. Main 메서드의 매개변수는 없거나 string 배열만 허용됨

✅ 이 규칙을 만족하는 메서드를 정의하면 C# 컴파일러는 자동으로 그 메서드를 시작점으로 선택해 `EXE 파일`을 생성한다.

```c#
class Program
{
	static int Main(string[] args) 
	{
		return 0;
	}
}
```

```shell
> ConsoleApp1.exe

> echo %ERRORLEVEL%
0
```

실행 명령어에 공백 기준으로 값을 주면 Main 함수의 `args` 매개변수로 주입된다
```shell
> ConsoleApp1.exe Hello World
```


4.1.5.3 정적 생성자

```c#
class Person
{
	public string _name;
	
	public Person(string name)
	{
		_name = name;
		Console.WriteLine("ctro 실행");
	}
	
	static Person()
	{
		Console.WriteLine("cctor 실행");
	}
}

namespace ConsoleApp1
{
	class Program
	{
		static void Main(string[] args)
		{
			Person person1 = new Person("");
			Console.WriteLine("=====");
			Person person2 = new Person("");
		}
	}
}
```


```shell
cctor 실행
ctor 실행 
=====
ctor 실행
```

> [!note] 정적 생성자는 클래스의 어떤 멤버든 최초로 접근하는 시점에 단 한 번만 실행된다는 점을 기억하자. 정적 멤버를 처음 호출할 경우이거나 인스턴스 생성자를 통해 객체가 만들어지는 시점이 되면 그 어떤 코드보다도 우선적으로 실행된다.


4.1.6 네임스페이스
- 말그대로 "이름 공간"이라고 번역
- 일반적으로 수 많은 클래스를 분류하는 방법으로 사용되고 있다. 
	- ex. 먹는 배(Pear)와 타는 배(Ship)는 서로 다른 것을 지징한다.
	- 컴파일러는 그런 상황에서 적절한 의미 선택을 할 수 없으므로 이름 충돌(naming conflict)이라는 오류를 발생시킨다. 
	- namespace는 컴파일러에서 문맥 정보를 제공한다
- ..
- 그런데 현실적으로 보면 네임스페이스가 이름 충돌 떄문에 사용되는 경우는 많지 않다.
- 대신 **클래스의 소속을 구분하는 데 사용되는 것이 더 일반적**

예로통신과 관련된 클래스와 파일 조작을 위한 클래스를 만들어야 할 떄 다음과 같은 식으로 네임스페이스를 구분할 수 있다.
```c#
namespace Communication
{
	class Http
	{
	}
	
	class Ftp
	{
	}
}

namespace Disk.FileSystem
{
	class File
	{
	}
}
```

>[!info] namespace 경로를 모두 나열할 필요없이 최상단에 using 예약어를 사용하여 네임스페이스를 미리 선언해두면 객체 생성시 이를 생략해도 C# 컴파일러가 알아서 객체가 속한 네임스페이스를 찾아내어 오류 없이 컴파일한다.


### 4.2 캡슐화

p127
캡슐화가 잘 된 클래슨느 그것을 사용하는 입장에서도 편리하다는 장점이 있다. 클래스의 이름 자체에서 이미 제공되는 기능을 대략 파알할 수 있고, 외부로 제공해야 할 기능에 대해서만 정확하게 public으로 노출한다. 함수가 블랙박스였던 것처럼 클래스 역시 객체의 역할을 추상화한다.


**4.2.1 접근 제한자 (access modifier)** 🔖

| private            | -                                                                                                                      |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------- |
| protected          | 내부에서의 접근과 함께 파생 클래스에서만 접근 허용                                                                                           |
| public             | -                                                                                                                      |
| internal           | 동일한 어셈블리 내에서는 public에 준한 접근을 허용한다. 다른 어셈블리에서는 접근할 수 없다.                                                                |
| internal protected | 동일 어셈블리 내에서 정의된 클래스이거나 다른 어셈블리라면 파생 클래스인 경우에 한해 접근을 허용한다. (protected internal로도 지정 가능), 즉, internal 또는 protected 조건이다. |

일반 클래스 정의는 public, internal 만 사용될 수 있지만 클래스 내부에 정의되는 또 다른 클래스 (중첩 클래스)에는 5가지 접근 제한자를 모두 명시할 수 있다.

🧐 접근 제어자가 명시되지 않는 경우에 기본 제한자 정의
- class 정의에서 접근 제어자 생략한 경우 기본 `internal`로 설정
- class 내부의 멤버에 대해서는 `private`로 설정 ✅

> [!info]  OOP 관점에서 접근제어자는 정보 은닉에 중요한 역할을 한다


4.2.2 정보 은닉 (information hiding)

// public으로 멤버 변수 다 열어두면 어디서 수정한지 다 찾아서 처리해야 하므로.. 유지보수 💩 됨

> [!info] 정보 은닉의 기본 원칙 
> - 특별한 이유를 제외하고는 필드를 절대 public으로 선언하지 않는다. (그런데 그럴만한 특별한 이유가 과연 있을까?)
> - 접근이 필요할 때는 접근자/설정자 메서드를 만들어 외부에서 접근하는 경로를 클래스 개발자의 관리하에 둔다


4.2.3 프로퍼티(property)

> [!note] 
> 프로퍼티도 속성으로 번역되는데, 이 경우 객체지향에서 말하는 속성과 혼동될 수 있다. 즉, 객체 지향에서 말하는 속성 (attribute)은 C#에서 필드(field)에 해당하고, C#의 속성(property)는 접근자/설정자 메서드에 대한 편리한 구문에 해당한다. 경우에 따라 C#의 프로퍼티는 보통 public으로 되는 경우가 많아서 "공용 속성"이라고 구분해서 부르기도 한다

// getter/setter를 말하는듯 하다

```c# hl:5-9
class Circle
{
	double pi = 3.14;
	
	public double Pi
	{
		get { return pi; }
		set { pi = value; } // set 내부에서만 사용가능한 "value" 예약어
	}
}
```


>[!info] 외국 개발자들은 이를 두고 "syntactic sugar"라는 표현을 쓰기도 한다. 프로그래밍 세계에서 귀찮은 작업을 편리하게 만들어주는 요소라면 그것이 현실 세계의 설탕에 비유할 수 있지 않을까 하느 생각에서 이런 말을 만들어 낸 것 같다. (한글 의역은 "간편 표기법" 정도)

### 4.3 상속 (inheritance)
- `:` (콜론)을 이용해 상속 

```c# hl:9,20
public class Computer
{
	protected bool powerOn; // protected로 해야 자식 클래스에서 접근 가능
	public void Boot() { }
	public void Shutdown() { }
	public void Reset() { }
}

public class NoteBook : Computer
{
	bool fingerScan; // Notebook 타입에 해당하는 멤버만 추가
	public bool HashFingerScanDevice() { return fingerScan; }
	
	public void CLoseLid()
	{
		Shutdown(); // Notebook에서 추가된 메서드 내에서 부모의 메서드 호출
	}
}

public class Desktop : Computer 
{
}

// ..
```

상속을 의도적으로 박고 싶을때 `sealed` 예약어 사용
```c#
sealed class Pen
{
}

public class EletricPen: Pen  // 💩 컴파일 오류 발생
{
}
```

> C#은 단일 상속 (single inheritance)만 지원


4.3.1 형변환

```c#
Notebook noteBook = new Notebook();

Computer pc1 = noteBook; // 명시적 형변환 가능 
pc1.Boot();
pc1.Shutdown();
```

```c#
Computer pc = new Computer();
Notebook notebook = (Notebook) pc; //  명시적 형 변환, 컴파일은 가능

// 하지만 실행하면 오류 발생
```

> 명시적 형변환보다 암시적 형변환이 좀 더 자주 사용된다


예제4.11 배열 요소에서의 암시적 형변환 
```c#
Computer[] machines = new Computer[] {
	new Notebook(), new Desktop(), new Netbook() // 암시적 형변환
};

DeviceManager manager = new DeviceManager();
foreach(Computer device in machines) 
{
	manager.TurnOff(device);
}
```

> 🔖 foreach라고 명시하는거 부터 다르고, in 과 괄호 줄 바꿈, 눈에 띈다


4.3.1.1 as, is 연산자
```c#
Computer pc = new Computer();
Notebook notebook = pc as Notebook;

if(notebook != null) // 코드대로라면 if문 내부의 코드가 실행될 가능성은 x
{
	notebook.CloseLid();
}
```

>[!info] as는 형변환이 가능하면 지정된 타입의 인스턴스 값을 반환하고, 가능하지 않으면 null을 반환하기 때문에 null 반환 여부를 통해 형변환이 성공했는지 판단할 수 있다. 한 가지 기억해야 할 점은 as 연산자는 참조형 변수에 대해서만 적용할 수 있고, 참조형 타입으로의 체크만 가능하다

```c# hl:2,8
int n = 5;
if((n as string) != null) // 💩 컴파일 오류 발생
{
	Console.WriteLint("변수 n은 string 타입");
}

string txt = "text";
if((txt as int) != null) // 💩 컴파일 오류 발생
{
	Console.WriteLint("변수 txt는 int 타입")
}
```

✅ `as`가 형변환 결괏값을 반환하는 반면 `is` 연산자는 형변환의 가능성 여부를 불린형의 결과값으로 반환한다

```c# hl:2,8
int n = 5;
if(n is string)
{
	Console.WriteLint("변수 n은 string 타입");
}

string txt = "text";
if(txt is int)
{
	Console.WriteLint("변수 txt는 int 타입")
}
```


4.3.2 모든 타입의 조상: System.Object
- C# 컴파일러는 기본적으로 object 타입을 상속받는다고 가정하여 자동으로 코드를 생성한다
- 결국 **C#에서 정의되는 모든 클래스의 부모는 object가 된다**

// p144 그림 4.9 object로부터 파생된 타입 관계

```c#
public class Object
{
	public virtual bool Equals(Object obj);
	public virtual int GetHashCode();
	public Type GetType();
	public virtual string ToString();
}
```

4.3.2.1 ToString()

```c#
Program program = new Program();
Console.WriteLine(program.ToString()); // 경로.Program 출력

// C# 기본타입도 ToString() 호출 가능
// 이때 클래스 전체 이름이 아닌 담고 있는 값을 출력
int n = 500;
Console.WriteLine(n.ToString()); // 500
```


4.3.2.2 GetType
```c#
Computer computer = new Computer();
Type type = computer.GetType();

Console.WriteLine(type.FullName); // Computer
Console.WriteLine(type.IsClass);  // True
Console.WriteLine(type.IsArray);  // False
```
- `Type` 클래스의 프로퍼티 호출
- `GetType()`은 클래스의 인스턴스로부터 Type을 구한다
- 반면 클래스의 이름에서 곧바로 Type을 구하는 방법도 제공되는데 `typeof` 예약어를 사용하면 된다

```c#
Type type= typeof(double);
Console.WriteLine(type.FullName); // System.Double

Console.WriteLine(typeof(System.Int16).FullName); // System.Int16
```


4.3.2.3 Equals
- `값 형식`과 `참조 형식`에 대해 달라진다
	- `값 형식` : 인스턴스가 소유하고 있는 값을 대상으로 비교
	- `참조 형식` : 할당된 메모리 위치를 가리키는 식별자의 값이 같은지 비교
		- 객체의 경우 `힙에 할당된 데이터 주소를 가리키고 있는 스택 변수의 값`을 비교

```c#
string txt1 = new string(new char[] {'t', 'e', 'x', 't'});
string txt2 = new string(new char[] {'t', 'e', 'x', 't'});

Console.WriteLine(txt1.Equals(txt2)); // True
```

4.3.2.4 GetHashCode
- 특정 인스턴스를 고유 식별할 수 있는 4byte int 값을 반환한다
- object에서 정의된 GetHashCode는 참조 타입에 대해 기본 동작을 정의해 뒀는데, 생성된 참조형 타입의 인스턴스가 살아 있는 동안 닷넷 프레임워크 내부에서 그러한 인스턴스에 부여한 식별자 값을 반환하기 때문에 적어도 프로그램이 실행되는 중에 같은 타입의 다른 인스턴스와 GetHashCode 반환 값이 겹칠 가능성이 많지 않다. 

> 보통 재정의해서 사용하지 않나 싶은데 .. 


4.3.3 모든 배열의 조상 : System.Array


표4.4 Array 타입의 멤버

| 멤버       | 타입        | 설명                           |
| -------- | --------- | ---------------------------- |
| Rank     | 인스턴스 프로퍼티 | 배열 인스턴스의 차원(dimension) 수를 반환 |
| Length   | 인스턴스 프로퍼티 | 배열 인스턴스의 요소 수를 반환            |
| Sort     | 정적 메서드    | -                            |
| GetValue | 인스턴스 메서드  | 지정된 인덱스의 배열 요소 값을 반환         |
| Copy     | 정적 메서드    | 배열의 내용을 다른 배열에 복사한다          |


```c#
namespace TodoApi
{
    public class ConsoleApp1
    {
        private static void OutputArrayInfo(Array arr)
        {
            Console.WriteLine("배열의 차원 수 : " + arr.Rank);
            Console.WriteLine("배열의 요소 수 : " + arr.Length);
            Console.WriteLine();
        }

        private static void OutputArrayElements(string title, Array arr)
        {
            Console.WriteLine("[" + title + "]");
            for (int i = 0; i < arr.Length; i++)
            {
                Console.Write(arr.GetValue(i) + ",");
            }

            Console.WriteLine();
            Console.WriteLine();
        }

        static void Main(string[] args)
        {
            bool[,] boolArray = new bool[,] { { true, false }, { false, true } };
            OutputArrayInfo(boolArray);

            int[] intArray = new int[] { 5, 4, 3, 2, 1, 0 };
            OutputArrayInfo(intArray);

            OutputArrayElements("원본 intArray", intArray);
            Array.Sort(intArray);
            OutputArrayElements("정렬 후 intArray", intArray);

            int[] copyArray = new int[intArray.Length];
            Array.Copy(intArray, copyArray, intArray.Length);

            OutputArrayElements("intArray로부터 복사된 copyArray", copyArray);
        }
    }
}

```

```text
배열의 차원 수 : 2
배열의 요소 수 : 4

배열의 차원 수 : 1
배열의 요소 수 : 6

[원본 intArray]
5, 4, 3, 2, 1,0,

정렬후 
0, 1, 2, 3, 4, 5,

copyArray
0, 1, 2, 3, 4, 5
```


**4.3.4 this**✅✅
- 클래스 내부에 멤버변수에 접근했을때 this를 생략했다고 봐도 무방
- this 표현을 쓰고 안 쓰고는 개발자의 취향.
	- 어떤 개발자는 메서드 내에서 멤버 변수에 접근할 떄 그것이 멤버 변수임을 명확히 인식할 수 있게 this를 명시하기도 함

예제 4.15 this를 이용한 생성자 코드 재사용🔖

```c# hl:7,16,23
class Book
{
	string title;
	decimal isbn13;
	string author;
	
	public Book(string title) : this(title, 0)
	{
	}
	
	public Book(string title, decimal isbn13)
		: this(title, isbn13, string.Empty)
	{
	}
	
	public Book(string title, deciaml isbn13, string author)
	{
		this.title = title;
		this.isbn13 = isbn13;
		this.author = author;
	}
	
	public Book() : this(string.Empty, 0, string.Empty)
	{
	}
}
``` 


4.3.4.1 this와 인스턴스/정적 멤버의 관계 

```c#
class Book
{
	string title; // 인스턴스 필드
	static int count; // 정적 필드
	
	public Book(strin gtitle) // 인스턴스 생성자
	{
		this.title = title; // this로 인스턴스 필드 식별 가능
		this.Open(); // this로 인스턴스 메서드 식별 가능
		increment(); // 정적 메서드 사용 가능
	}
	
	void Open() // 인스턴스 메서드
	{
		Console.WriteLine(this.title); // 인스턴스 멤버 사용 가능
		Console.WriteLine(count); // 정적 멤버 사용 가능
	}
	
	public void Close()
	{
		Console.WriteLine(this.title + " 책을 덮는다");
	}
	
	static void Increment()
	{
		count ++; // 정적 필드 사용 가능
				  // 정적 메서드에서는 this가 없으므로 인스턴스 멤버 사용 x
	}
}
```

C# 컴파일러는 메서드 호출시 this를 인스턴스 메서드의 첫 번째 인자로 넘겨주는 식으로 구현하고 있따. 

```c# hl:6
Book book = new Book("");
book.Close();

// C# 컴파일러가 빌드한 후에 
Book book = new Book("");
book.Close(book);
```

즉 메서드에 해당 객체를 가리키는 인스턴스 변수를 인자로 넘기는 것이다. 그와 동시에 C# 컴파일러는 인스턴스 메서드도 다음과 같이 변환해서 컴파일한다. 

```c# hl:5
class Book
{
	// 생략 
	
	public void Close(Book this)
	{
		Console.WriteLine(this.title + " 책을 덮는다");
	}
}
```

> 컴파일 후에 this를 사용하는 곳은 저렇게 바꿔주나 보다 🔖

>[!tip] 마법처럼 보였던 this 식별자의 존재는 이처럼 컴파일러의 노력으로 빚어낸 결과다. 이 때문에 모든 인스턴스 메서드는 인자를 무조건 1개 이상 더 받게 돼 있으므로 내부에서 인스턴스 멤버에 접근할 일이 없다면 정적 메서드로 명시하는 것이 성능상 유리할 수 있다.


4.3.5 base🔖
- `this` 예약어가 클래스 인스턴스 자체를 가리키는 것과 달리`base` 예약어는 "부모 클래스"를 명시적으로 가르킴
- this 와 마찬가지로 부모 클래스의 멤버를 사용할 때 base 키워드가 생략 된것이나 다름없다고 보면 된다. 

```c#
public class Computer
{
	bool powerOn;
	
	public void Boot() { }
	public void Shutdown() { }
	public void Reset() { }
}

public class Notebook : Computer
{
	bool fingerScan;
	public bool HashFingerScanDevice() { return fingerScan; }
	
	public void CloseLid()
	{
		base.Shutdown(); // base 예약어를 명시
	}
}
```

```c# hl:13,17
class Book
{
	decimal isbn13;
	
	public Book(decimal isbn13)
	{
		this.isbn13 = isbn13;
	}
}

class EBook : Book
{
	public EBook() : base(0)
	{
	}
	
	public EBook(decimal isbn) : base(isbn)
	{
	}
}

```


### 4.4 다형성 (polymorphism)

```c#
class Mammal
{
	public void Move()
	{
		Console.WriteLine("이동한다");
	}
}

class Lion : Mammal
{
	public void Move()
	{
		Console.WriteLine("네 발로 움직인다");
	}
}

class Whale : Mammal
{
	public void Move()
	{
		Console.WriteLine("수영한다");
	}
}

class Human : Mammal
{
	public void Move()
	{
		Console.WriteLine("두 발로 움직인다");
	}
}
```


```c#
Lion lion = new Lion();
Mammal one = lion; // 부모 타입으로 형변환

one.Move(); // "이동한다" 출력 💩
```


> ✅ virtual과 override 예약어 명시하여 다형성 유지


```c# hl:3,11,19,27
class Mammal
{
	virtual public void Move()
	{
		Console.WriteLine("이동한다");
	}
}

class Lion : Mammal
{
	override public void Move()
	{
		Console.WriteLine("네 발로 움직인다");
	}
}

class Whale : Mammal
{
	override public void Move()
	{
		Console.WriteLine("수영한다");
	}
}

class Human : Mammal
{
	override public void Move()
	{
		Console.WriteLine("두 발로 움직인다");
	}
}
```

```c#
Lion lion = new Lion();
Mammal one = lion; // 부모 타입으로 형변환

one.Move(); // "네 발로 움직인다" 출력 

Human human = new Human();
Mamal two = human;
two.Move(); // "두 발로 움직인다" 출력
```


추가적으로 컴파일 경고를 없애려면 다음과 같이 할 수도 있다
```c#
class Lion : Mammal
{
	new public void Move() { } // 구현 생략
}

class Whale : Mammal
{
	new public void Move() { } // 구현 생략
}

class Human : Mammal
{
	new public void Move() { } // 구현 생략
}
```

>[!note] 부모와 자식 클래스에서 동일한 이름의 메서드를 사용하려면 두 가지 중 하나를 선택
>1. 메서드 오버라이드를 원하는가? 그렇다면 virtual/override를 사용하라 
>2. 단순히 자식 클래스에서 동일한 이름의 메서드가 필요했떤 것인가? 그렇다면 new를 사용하라


4.4.1.1 base를 이용한 메서드 재사용

```c# hl:13
public class Computer 
{
	virtual public void Boot()
	{
		Console.WriteLine("메인보드 켜기");
	}
}

public class Notebook : Computer
{
	override public void Boot()
	{
		base.Boot();
		Console.WriteLine("액정 화면 켜기");
	}
}
```


4.4.2 오버로드
- 메서드 시그니처(method signature)
	- 말 그대로 서명으로 번역
	- 일상 생활에서는 서명을 보고 그 주체가 누군인지를 판달할 수 있다.
	- 마찬가지로 메서드 시그니처는 어떤 메서드를 교유하게 규정할 수 있는 정보를 의미

> [!info] 메서드의 정의 분리
> - "이름", "반환 타입", "매개변수의 수", "개별 매개변수 타입 "
> - 위의 것들이 메서드의 "서명"이 된다
> - "메서드의 시그니처가 동일하다" == "메서드가 같다"로 해석

> 함수 이름만 같고 파라미터, 반환 타입/값이 다른 경우를 메서드 오버로드라고 함


4.4.2.2 연산자 오버로드 
```c#
public class Kilogram
{
	double mass;
	
	public Kilogram(double value)
	{
		this.mass = value;
	}
	
	public Kilogram Add(kilogram target) 
	{
		return new Kilogram(this.mass + target.mass);
	}
	
	public override string ToString() 
	{
		return mass + "kg";
	}
}
```


▶️ 처음보는 시그니처다..
```text
public static 타입 operator 연산자 (타입1 변수명1, 타입2 변수명2)
{
	// [타입]을 반환하는 코드
}
```

Kilogram의 + 연산자를 재정의하면 다음과 같다
```c# hl:5,14
public class Kilogram
{
	// 생략
	
	public static Kilogram operator +(Kilogram op1, Kilogram op2)
	{
		return new Kilogram(op1.mass + op2.mass);
	}
}

Kilogram kg1 = new Kilogram(5);
Kilogram kg2 = new Kilogram(15);

Kilogram kg3 = kg1 + kg2;
```
- `+` 연산자가 재정의된 메서드
- 이를 연산자 오버로드라고 한다

// p175 표 4.5 연산자에 따른 오버로드 가능 여부 (📸)


4.4.2.3 클래스 간의 형변환
```c#
public class Currency
{
	decimal money;
	public decimal Money { get { return money; } }
	
	public Currency(decimal money)
	{
		this.money = money;
	}
}

public class Won : Currency
{
	public Won(decimal money) : base(money) { }
	
	public override string ToString() 
	{
		return Money + "Won";
	}
}

public class Dollar : Currency
{
	public Dollar(decimal money) : base(money) { }
	
	public override string ToString() 
	{
		return Money + "Dollar";
	}
}

public class Yen : Currency
{
	public Yen(decimal money) : base(money) { }
	
	public override string ToString() 
	{
		return Money + "Yen";
	}
}
```


```c#
Won won = new Won(1000);
Dollar dollar = new Dollar(1);
Yen yen = new Yen(13);

won = yen; // 💩 컴파일 오류, 형변환 실패
```

> implicit(암시적), explicit(명시적) 예약어를 사용해서 해결 가능

```c# hl:5
public class Yen : Currency
{
	// Yen -> Won 대입
	static bpulic implicit operator Won(Yen yen)
	{
		return new Won(yen.Money * 13m); // m?
	}
}

Yen yen = new Yen(100);
Won won = yen; // implicit(암시적) 형변환 가능
Won won2 = (Won) yen; // explicit(명시적) 형변환 가능

Console.WriteLine(won); // 1300Won
```


```c# hl:3
public class Dollar : Currency 
{
	static bpulic explicit operator Won(Dollar dollar)
	{
		return new Won(dollar.Money * 1000m);
	}
}

Dollar dollar = new Dollar(1);
Won won = dollar; // implicit(암시적) 형변환 x (컴파일 오류)
Won won2 = (Won)dollar; // explicit(명시적) 형변환

Console.WriteLine(won2); // 1000Won
```


### 4.5 C#의 클래스 확장

4.5.1.1 중첩 클래스 
```c#
public class HardDisk
{
	class Platter // 접근 제어자 생략시 private로 지정됨
	{
	}
	
	class Head
	{
	}
	
	Platter[] platter;
	Head head;
}
```

4.5.1.2 추상 클래스 
- 추상 메서드는 private 선언 x
- 추상 클래스는 new로 인스턴스화 x
	- 만약 추사 을래스가 new를 통해 존재한다면 추상 메서드를 호출하는 경우 어떤 식으로 동작할지 예측할 수 없을 것이다. 추상 클래스에 반드시 추상 메서드가 포함돼 있어야 하는 것은 아니지만 그래도 여전히 추상 클래스는 new로 인스턴스화할 수 x

```c# hl:18,20
class Point
{
	int x, y;
	
	public Point(int x, int y)
	{
		this.x = x;
		this.y = y;
	}
	
	public override string ToString()
	{
		return "x : " + x + ", y : " + y; 
	}
}

// 추상 클래스
abstract class DrawingObject
{
	public abstract void Draw(); // 추상 메서드
	
	public void Move() { Console.WriteLine("Move"); }
}

class Line : DrawingObject 
{
	Point p1, p2;
	
	public Line(Point p1, Point p2)
	{
		this.p1 = p1;
		this.p2 = p2;
	}
	
	public override void Draw()
	{
		Console.WriteLine("Line : " + p1.ToString() + " ~ " + p2.ToString());
	}
}
```


4.5.1.3 델리게이트 (p183 ~ 200)


4.5.1.4 인터페이스 (p201 ~ 216)


4.5.1.5 구조체 (p216 ~ 232)


4.5.1.6 열거형 (p232)