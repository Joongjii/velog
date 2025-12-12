<article>

  <hr />

  <h2>1. 제네릭 메서드란 무엇인가?</h2>

  <p>
    한 줄로 정리하면:
  </p>

  <blockquote>
    <strong>“메서드 단위에서 타입을 외부에서 받는 메서드”</strong>
  </blockquote>

  <p>
    제네릭 클래스는 <code>class MyClass&lt;T&gt;</code>처럼 <strong>클래스 전체</strong>가 타입 매개변수 T를 공유하지만,<br />
    제네릭 메서드는 <strong>특정 메서드 하나만</strong> 타입 매개변수를 가진다.
  </p>

  <p>기본 형태는 이렇게 생겼다:</p>

  <pre><code class="language-csharp">반환형 메서드이름&lt;T&gt;(매개변수들...)
{
    // T를 타입처럼 사용
}</code></pre>

  <p>예를 들어:</p>

  <pre><code class="language-csharp">static void PrintTwice&lt;T&gt;(T value)
{
    Console.WriteLine(value);
    Console.WriteLine(value);
}</code></pre>

  <p>사용 예:</p>

  <pre><code class="language-csharp">PrintTwice&lt;int&gt;(10);        // T = int
PrintTwice&lt;string&gt;("Hi");   // T = string
PrintTwice(3.14);            // T를 컴파일러가 double로 추론</code></pre>

  <ul>
    <li><code>PrintTwice&lt;T&gt;</code> 가 제네릭 메서드</li>
    <li><code>&lt;T&gt;</code>가 <strong>형식 매개변수(Type Parameter)</strong></li>
    <li><code>PrintTwice&lt;int&gt;</code> 호출 시 <code>T</code>는 <code>int</code></li>
    <li>대부분은 <code>PrintTwice(3.14)</code>처럼 쓰면 <strong>컴파일러가 T를 자동으로 추론</strong>해 준다</li>
  </ul>

  <hr />

  <h2>2. 제네릭 “클래스”와 제네릭 “메서드” 차이</h2>

  <h3>2-1. 제네릭 클래스</h3>

  <pre><code class="language-csharp">class Box&lt;T&gt;
{
    public T Value;
}</code></pre>

  <pre><code class="language-csharp">var intBox = new Box&lt;int&gt;();
intBox.Value = 10;

var strBox = new Box&lt;string&gt;();
strBox.Value = "Hello";</code></pre>

  <ul>
    <li>클래스 수준에서 <code>T</code>를 받는다.</li>
    <li>클래스 안의 필드, 프로퍼티, 메서드 모두가 같은 <code>T</code>를 공유한다.</li>
  </ul>

  <h3>2-2. 제네릭 메서드</h3>

  <pre><code class="language-csharp">class Util
{
    public static void PrintTwice&lt;T&gt;(T value)
    {
        Console.WriteLine(value);
        Console.WriteLine(value);
    }
}</code></pre>

  <ul>
    <li><code>Util</code> 클래스 자체는 제네릭이 아니다.</li>
    <li><strong>메서드 <code>PrintTwice&lt;T&gt;</code>만</strong> 제네릭이다.</li>
    <li>각 호출마다 T를 다르게 줄 수 있다.</li>
  </ul>

  <h3>2-3. 클래스 제네릭 + 메서드 제네릭 같이 쓰는 경우</h3>

  <pre><code class="language-csharp">class Repository&lt;T&gt;
{
    public T GetById(int id)
    {
        // ...
        return default;
    }

    // 메서드만 따로 제네릭
    public U Convert&lt;U&gt;(T source)
    {
        // 여기서 U는 메서드 전용 타입 매개변수
        return default;
    }
}</code></pre>

  <ul>
    <li><code>T</code>는 클래스 전체에서 사용하는 타입</li>
    <li><code>U</code>는 <strong>Convert 메서드 전용 타입</strong></li>
    <li>둘은 서로 독립적이다 (이름만 다를 뿐 둘 다 타입 매개변수)</li>
  </ul>

  <hr />

  <h2>3. 제네릭 메서드 기본 문법 예제</h2>

  <h3>3-1. 가장 단순한 형태 – Echo</h3>

  <pre><code class="language-csharp">static T Echo&lt;T&gt;(T value)
{
    return value;
}</code></pre>

  <pre><code class="language-csharp">int a = Echo&lt;int&gt;(10);        // 10
string s = Echo("Hello");      // T를 string으로 자동 추론</code></pre>

  <ul>
    <li>어떤 타입이 오든 그대로 되돌려주는 메서드</li>
    <li>유틸리티/헬퍼 코드를 만들 때 자주 쓰는 패턴</li>
  </ul>

  <h3>3-2. 타입 매개변수 여러 개</h3>

  <pre><code class="language-csharp">static Dictionary&lt;TKey, TValue&gt; MakeDic&lt;TKey, TValue&gt;(TKey key, TValue value)
{
    var dic = new Dictionary&lt;TKey, TValue&gt;();
    dic[key] = value;
    return dic;
}</code></pre>

  <pre><code class="language-csharp">var dic1 = MakeDic&lt;int, string&gt;(1, "One");
var dic2 = MakeDic("ID", 100);   // 타입 추론 → TKey=string, TValue=int</code></pre>

  <hr />

  <h2>4. 제네릭 메서드 + 제약조건(where)</h2>

  <p>
    제네릭 클래스처럼, 제네릭 메서드도 <strong>형식 매개변수 제약조건</strong>을 걸 수 있다.
  </p>

  <h3>4-1. new() 제약 – 기본 생성자 필요</h3>

  <pre><code class="language-csharp">static T CreateAndLog&lt;T&gt;() where T : new()
{
    T obj = new T();
    Console.WriteLine(typeof(T).Name + " 생성됨");
    return obj;
}</code></pre>

  <pre><code class="language-csharp">class User
{
    public User() { }  // 기본 생성자
}

var user = CreateAndLog&lt;User&gt;();</code></pre>

  <ul>
    <li><code>where T : new()</code> 덕분에 <code>new T()</code> 사용 가능</li>
    <li>T는 반드시 public 기본 생성자를 가져야 한다</li>
  </ul>

  <h3>4-2. 인터페이스 제약</h3>

  <pre><code class="language-csharp">interface IHasName
{
    string Name { get; set; }
}

static void PrintName&lt;T&gt;(T obj) where T : IHasName
{
    Console.WriteLine(obj.Name);
}</code></pre>

  <pre><code class="language-csharp">class User : IHasName
{
    public string Name { get; set; }
}

PrintName(new User { Name = "홍길동" });</code></pre>

  <ul>
    <li>T가 어떤 구체 타입인지 몰라도, <code>IHasName</code>만 구현했다면 <code>Name</code> 속성을 안전하게 사용할 수 있다.</li>
  </ul>

  <hr />

  <h2>5. 제네릭 메서드, 실무에서 어디에 많이 쓰일까?</h2>

  <p>사실 이미 엄청 많이 쓰고 있다. 대표적으로:</p>

  <h3>5-1. 컬렉션/LINQ 확장 메서드</h3>

  <pre><code class="language-csharp">public static T[] ToArray&lt;T&gt;(this IEnumerable&lt;T&gt; source)
public static List&lt;T&gt; ToList&lt;T&gt;(this IEnumerable&lt;T&gt; source)
public static T FirstOrDefault&lt;T&gt;(this IEnumerable&lt;T&gt; source)
public static IEnumerable&lt;TResult&gt; Select&lt;TSource, TResult&gt;(
    this IEnumerable&lt;TSource&gt; source,
    Func&lt;TSource, TResult&gt; selector)</code></pre>

  <pre><code class="language-csharp">var list = new List&lt;int&gt; { 1, 2, 3 };
int first = list.FirstOrDefault();  // T = int</code></pre>

  <ul>
    <li>전부 제네릭 메서드이다.</li>
    <li>이 덕분에 <strong>하나의 메서드</strong>로 모든 타입의 리스트, 컬렉션을 다 다룰 수 있다.</li>
  </ul>

  <h3>5-2. Task / 비동기 라이브러리</h3>

  <pre><code class="language-csharp">var t = Task.FromResult(10);  // 사실은 Task.FromResult&lt;int&gt;(10)</code></pre>

  <p><code>Task&lt;TResult&gt;</code> 자체도 제네릭 타입이고, 그걸 생성하는 여러 API도 제네릭 메서드로 제공된다.</p>

  <hr />

  <h2>6. 제네릭 메서드를 쓰는 이유 (장점)</h2>

  <h3>1) 코드 재사용성</h3>
  <p>
    타입만 다른 동일한 로직을 여러 버전으로 만들 필요 없이,  
    <strong>한 번만 정의하고 모든 타입에 쓸 수 있다.</strong>
  </p>

  <h3>2) 타입 안정성</h3>
  <p>
    <code>object</code> + 캐스팅으로 처리하는 대신,  
    제네릭으로 타입을 받으면 <strong>컴파일 타임 타입 체크</strong>를 받을 수 있다.
  </p>

  <h3>3) 박싱/언박싱 감소</h3>
  <p>
    값 형식(<code>int</code>, <code>double</code> 등)을 <code>object</code>에 담았다 뺐다 하면 박싱/언박싱이 생기는데,  
    제네릭을 쓰면 이런 오버헤드가 줄어든다.
  </p>

  <hr />

  <h2>7. 대표적인 예: Swap 제네릭 메서드</h2>

  <pre><code class="language-csharp">static void Swap&lt;T&gt;(ref T a, ref T b)
{
    T temp = a;
    a = b;
    b = temp;
}</code></pre>

  <pre><code class="language-csharp">int x = 10, y = 20;
Swap(ref x, ref y);             // T = int

string s1 = "A", s2 = "B";
Swap(ref s1, ref s2);           // T = string</code></pre>

  <ul>
    <li><code>int</code>용 Swap, <code>string</code>용 Swap을 따로 만들 필요가 없다.</li>
    <li><strong>하나의 Swap&lt;T&gt;</strong>로 모든 타입을 처리할 수 있다.</li>
  </ul>

  <hr />

</article>