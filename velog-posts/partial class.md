<article>
<hr />

  <h2>1. <code>partial class</code> 한 줄 정의</h2>

  <blockquote>
    <strong>partial class = “하나의 클래스를 여러 파일로 나눠서 정의할 수 있게 해주는 문법”</strong>
  </blockquote>

  <p>
    보통은 “클래스 하나 = 파일 하나”처럼 생각하기 쉬운데, C#에서는 하나의 클래스를 여러 .cs 파일에 나눠서 적을 수 있다.<br />
    나눠져 있는 코드들은 컴파일할 때 <strong>컴파일러가 전부 합쳐서 하나의 클래스</strong>로 만들어 준다.
  </p>

  <h3>간단 예제</h3>

  <p>예를 들어, 아래처럼 두 파일이 있다고 해 보자.</p>

  <h4><code>Person.cs</code></h4>

  <pre><code class="language-csharp">public partial class Person
{
    public string Name { get; set; }

    public void SayHello()
    {
        Console.WriteLine($"안녕, 나는 {Name}");
    }
}</code></pre>

  <h4><code>Person.Address.cs</code></h4>

  <pre><code class="language-csharp">public partial class Person
{
    public string Address { get; set; }

    public void PrintAddress()
    {
        Console.WriteLine($"주소: {Address}");
    }
}</code></pre>

  <p>
    이 두 파일은 컴파일될 때 이렇게 하나의 <code>Person</code> 클래스로 합쳐진 것처럼 동작한다.
  </p>

  <pre><code class="language-csharp">public class Person
{
    public string Name { get; set; }
    public void SayHello() { ... }

    public string Address { get; set; }
    public void PrintAddress() { ... }
}</code></pre>

  <p>
    개발자는 파일을 나눠서 관리하지만, 프로그램이 실행될 때는 <strong>그냥 한 클래스</strong>라고 생각하면 된다.
  </p>

  <hr />

  <h2>2. 왜 <code>partial class</code>를 쓰나?</h2>

  <p>“그냥 한 파일에 다 쓰면 되지, 왜 쪼개서 쓰지?” 라는 생각이 들 수 있다. 대표적인 이유는 세 가지 정도다.</p>

  <h3>2-1. 자동 생성 코드와 내가 짠 코드를 분리하려고</h3>

  <p>
    실무에서 <strong>가장 많이 쓰이는 이유</strong>는 바로 이거다.
  </p>

  <ul>
    <li>WinForms, WPF, ASP.NET WebForms 같은 UI 디자이너를 쓰면</li>
    <li>버튼, 텍스트박스 같은 컨트롤을 배치하는 순간 디자이너가 자동으로 코드를 만들어 준다.</li>
  </ul>

  <p>예를 들어 WinForms에서 폼 하나를 만들면 이런 구조를 많이 보게 된다.</p>

  <h4><code>Form1.cs</code> (내가 작성하는 코드)</h4>

  <pre><code class="language-csharp">public partial class Form1 : Form
{
    public Form1()
    {
        InitializeComponent(); // 디자이너가 만든 설정 코드 호출
    }

    private void button1_Click(object sender, EventArgs e)
    {
        MessageBox.Show("클릭!");
    }
}</code></pre>

  <h4><code>Form1.Designer.cs</code> (디자이너가 자동 생성하는 코드)</h4>

  <pre><code class="language-csharp">partial class Form1
{
    private System.Windows.Forms.Button button1;

    private void InitializeComponent()
    {
        this.button1 = new System.Windows.Forms.Button();
        // 위치, 크기, 텍스트, 이벤트 핸들러 연결 등 자동 생성 코드
    }
}</code></pre>

  <p>
    두 파일 모두 <code>partial class Form1</code>로 선언되어 있기 때문에, 둘이 합쳐져서 하나의 <code>Form1</code> 클래스가 된다.
  </p>

  <p>
    이렇게 나누면 좋은 점:
  </p>

  <ul>
    <li><strong>디자이너가 건드리는 코드</strong>와 <strong>내가 직접 쓰는 코드</strong>를 파일 수준에서 분리할 수 있다.</li>
    <li>디자이너가 <code>.Designer.cs</code> 파일을 자동으로 고쳐도, 내 코드가 섞이지 않는다.</li>
    <li>내가 실수로 디자이너 코드 부분을 지워서 폼이 깨지는 일을 줄일 수 있다.</li>
  </ul>

  <h3>2-2. 클래스가 너무 커서 논리적으로 쪼개고 싶을 때</h3>

  <p>
    어떤 클래스가 기능이 많아지면 줄 수가 엄청나게 길어질 수 있다. 예를 들어 <code>OrderService</code>에
  </p>

  <ul>
    <li>주문 조회</li>
    <li>주문 생성</li>
    <li>주문 취소</li>
    <li>정산 처리</li>
    <li>로그 기록</li>
  </ul>

  <p>등이 한꺼번에 들어 있으면 유지보수가 힘들다.</p>

  <p>이럴 때 <code>partial</code>로 파일을 역할별로 나눌 수 있다.</p>

  <h4><code>OrderService.Query.cs</code></h4>

  <pre><code class="language-csharp">public partial class OrderService
{
    public Order GetOrder(int id) { ... }

    public List&lt;Order&gt; GetOrdersByCustomer(string customerCode) { ... }
}</code></pre>

  <h4><code>OrderService.Command.cs</code></h4>

  <pre><code class="language-csharp">public partial class OrderService
{
    public void CreateOrder(Order order) { ... }

    public void CancelOrder(int id) { ... }
}</code></pre>

  <p>
    이런 식으로 나누면:
  </p>

  <ul>
    <li>“조회 관련 코드만 보고 싶다” → <code>.Query.cs</code>만 열면 됨</li>
    <li>“등록/수정 관련 코드만 보고 싶다” → <code>.Command.cs</code>만 보면 됨</li>
    <li>파일이 나뉘어 있어서 검색/관리/리뷰할 때 훨씬 편하다.</li>
  </ul>

  <h3>2-3. 여러 사람이 한 클래스를 동시에 작업할 때</h3>

  <p>
    팀 작업을 할 때, 한 클래스에 기능이 몰려 있으면 한 파일에서 충돌이 많이 날 수 있다.
  </p>

  <ul>
    <li>A 개발자: 조회 기능 작업</li>
    <li>B 개발자: 등록/수정 기능 작업</li>
  </ul>

  <p>
    이런 상황에서 파일을 나눠 두면, 서로 다른 파일에서 작업할 수 있어서 충돌을 줄일 수 있다.
  </p>

  <hr />

  <h2>3. <code>partial class</code>의 규칙과 주의사항</h2>

  <h3>3-1. 이름이 같아야 한다</h3>

  <pre><code class="language-csharp">public partial class Person { ... }
public partial class Person { ... }  // OK, 같은 클래스의 다른 부분

public partial class PersonEx { ... } // 이름이 다르면 완전히 다른 클래스</code></pre>

  <h3>3-2. 같은 namespace 안에 있어야 한다</h3>

  <pre><code class="language-csharp">// 파일 1
namespace MyApp.Models
{
    public partial class Person { ... }
}

// 파일 2
namespace MyApp.Models   // 네임스페이스가 같아야 합쳐진다
{
    public partial class Person { ... }
}</code></pre>

  <h3>3-3. 접근 제한자(public, internal 등)는 일관되게</h3>

  <p>
    보통 한 파일에만 <code>public partial class</code>를 적고, 나머지는 <code>partial class</code>만 적어도 컴파일은 잘 된다.<br />
    하지만 가독성 측면에서는 <strong>가능하면 같은 접근제한자로 맞춰주는 편이 좋다</strong>.
  </p>

  <h3>3-4. 실행될 때는 그냥 “한 클래스”일 뿐</h3>

  <p>
    <strong>중요한 포인트</strong>는, partial은 어디까지나 “코드 작성 편의를 위한 문법”이라는 점이다.
  </p>

  <ul>
    <li>런타임(실행 시점)에는 “부분1, 부분2” 같은 개념이 없다.</li>
    <li>그냥 <strong>하나의 <code>Person</code> 클래스</strong>만 존재한다.</li>
    <li>partial인지 아닌지 구분할 수 있는 정보도 별로 중요하지 않다.</li>
  </ul>

  <hr />

  <h2>4. 콘솔 프로젝트에서 직접 써 보는 간단 예제</h2>

  <p>실제로 partial class가 어떻게 동작하는지, 아주 간단한 예제로 확인해 보자.</p>

  <h3><code>Person.Basic.cs</code></h3>

  <pre><code class="language-csharp">public partial class Person
{
    public string Name { get; set; }
    public int Age { get; set; }

    public void PrintBasicInfo()
    {
        Console.WriteLine($"이름: {Name}, 나이: {Age}");
    }
}</code></pre>

  <h3><code>Person.Job.cs</code></h3>

  <pre><code class="language-csharp">public partial class Person
{
    public string JobTitle { get; set; }

    public void PrintJobInfo()
    {
        Console.WriteLine($"직업: {JobTitle}");
    }
}</code></pre>

  <h3><code>Program.cs</code></h3>

  <pre><code class="language-csharp">using System;

class Program
{
    static void Main()
    {
        Person p = new Person();
        p.Name = "홍길동";
        p.Age = 30;
        p.JobTitle = "개발자";

        p.PrintBasicInfo(); // 이름, 나이 출력
        p.PrintJobInfo();   // 직업 출력

        Console.ReadKey();
    }
}</code></pre>

  <p>
    비록 파일은 3개로 나눠져 있지만, 실제로는 그냥 <code>Person</code>이라는 클래스 한 개가 있는 것처럼 자연스럽게 동작한다.
  </p>

  <hr />

  <h2>5. 보너스: partial method 간단 맛보기</h2>

  <p>
    <code>partial class</code> 안에서는 <strong><code>partial method</code></strong>도 쓸 수 있다.
    이건 자동 생성 코드와 후처리 코드를 나눌 때 유용하다.
  </p>

  <h3>파일 1</h3>

  <pre><code class="language-csharp">public partial class Person
{
    partial void OnCreated();

    public Person()
    {
        OnCreated(); // 구현이 있으면 호출, 없으면 컴파일 시 제거됨
    }
}</code></pre>

  <h3>파일 2</h3>

  <pre><code class="language-csharp">public partial class Person
{
    partial void OnCreated()
    {
        Console.WriteLine("Person 객체가 생성되었습니다.");
    }
}</code></pre>

  <p>
    구현을 해 두면 생성자에서 자동으로 호출되고,<br />
    구현을 안 하면 컴파일러가 그냥 이 호출 자체를 지워 버린다.
  </p>

  <p>
    이건 조금 고급 기능에 가까우니까,  
    “partial class 안에 partial method라는 것도 있다” 정도만 알고 넘어가도 충분하다.
  </p>

  <hr />

  <h2>6. 정리</h2>

  <ul>
    <li><strong>partial class</strong>는 하나의 클래스를 여러 파일로 나누기 위한 문법이다.</li>
    <li>컴파일 시에 여러 파일의 partial 정의들을 합쳐서 하나의 클래스가 된다.</li>
    <li>주로:
      <ul>
        <li>자동 생성 코드와 직접 작성한 코드를 분리할 때</li>
        <li>클래스가 너무 커서 기능별로 파일을 나누고 싶을 때</li>
        <li>여러 개발자가 한 클래스를 나눠 작업할 때 사용된다.</li>
      </ul>
    </li>
    <li>이름, namespace가 동일해야 같은 클래스의 partial로 합쳐질 수 있다.</li>
    <li>실행 시점에는 그냥 일반 클래스와 완전히 똑같이 취급된다.</li>
  </ul>

  <p>
    앞으로 프로젝트에서 <code>Form1.Designer.cs</code>, <code>SomeWindow.g.cs</code> 같은 파일을 보게 되면,
    “아, 이게 바로 partial class로 나눠진 자동 생성 코드구나” 하고 이해하면 된다.
  </p>
</article>