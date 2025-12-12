<article>


  <hr />

  <h2>1. 먼저, “속성 패턴(Property Pattern)”이 뭔지부터</h2>

  <p>
    속성 패턴은 <strong>객체의 속성 값</strong>을 기준으로 매칭하는 패턴 매칭 문법이다.
  </p>

  <pre><code class="language-csharp">if (order is { Status: "Closed", Amount: &gt; 0 })
{
    Console.WriteLine("종료된 유효 주문");
}</code></pre>

  <p>위 코드는 이렇게 읽을 수 있다.</p>

  <ul>
    <li><code>order</code>라는 객체가 있고,</li>
    <li>그 안에 <code>Status</code> 속성이 있고 값이 <code>"Closed"</code>이고,</li>
    <li><code>Amount</code> 속성이 있고 값이 0보다 크면,</li>
    <li><strong>if 블록 안으로 들어간다</strong>.</li>
  </ul>

  <p>즉,</p>

  <pre><code class="language-csharp">order.Status == "Closed" &amp;&amp; order.Amount &gt; 0</code></pre>

  <p>
    를 패턴 매칭 문법으로 예쁘게 표현한 것이라고 보면 된다.<br />
    여기까지가 기본적인 <strong>속성 패턴</strong>이다.
  </p>

  <hr />

  <h2>2. 중첩된 객체일 때 기존 속성 패턴은 어떻게 생겼나?</h2>

  <p>
    이제 이런 모델을 가정해 보자.
  </p>

  <pre><code class="language-csharp">class Address
{
    public string City    { get; set; }
    public string Country { get; set; }
}

class Person
{
    public string Name    { get; set; }
    public Address Address { get; set; }
}</code></pre>

  <p>
    <code>Person</code> 안에 <code>Address</code>가 있고, 그 안에 다시 <code>City</code>가 있는 구조다.<br />
    이때 “서울에 사는 사람”을 체크하고 싶다고 하자.
  </p>

  <h3>2-1. C# 10 이전 스타일 (중첩 속성 패턴)</h3>

  <pre><code class="language-csharp">if (person is { Address: { City: "Seoul" } })
{
    Console.WriteLine("서울에 사는 사람");
}</code></pre>

  <p>읽어보면:</p>

  <ul>
    <li><code>person</code>은
      <ul>
        <li><code>Address</code>라는 속성을 가지고 있고,</li>
        <li>그 <code>Address</code> 속성 안에 <code>City</code>라는 속성이 있고,</li>
        <li>그 값이 <code>"Seoul"</code>이면 매칭된다.</li>
      </ul>
    </li>
  </ul>

  <p>
    동작은 이해되는데, <strong>괄호가 겹겹이 중첩</strong>되어 있어서 눈에 잘 안 들어오고,
    우리가 평소에 쓰는 <code>person.Address.City</code> 스타일과도 조금 거리가 있다.
  </p>

  <hr />

  <h2>3. 확장 속성 패턴(Extended Property Pattern) 등장</h2>

  <p>
    이런 불편함 때문에 C# 10부터 <strong>확장 속성 패턴</strong>이라는 문법이 추가되었다.<br />
    핵심 아이디어는 아주 단순하다.
  </p>

  <blockquote>
    “중첩 객체의 속성도 <code>Address.City</code>처럼 <strong>점(.)으로 이어서</strong> 쓸 수 있게 하자.”
  </blockquote>

  <h3>3-1. 같은 의미, 더 간단한 문법</h3>

  <pre><code class="language-csharp">// 옛날 방식
if (person is { Address: { City: "Seoul" } }) { ... }

// 확장 속성 패턴
if (person is { Address.City: "Seoul" }) { ... }</code></pre>

  <p>
    둘은 완전히 <strong>동일한 의미</strong>다.  
    다만 확장 속성 패턴은:
  </p>

  <ul>
    <li><code>Address</code> 속성 안으로 들어가서</li>
    <li>그 안에 있는 <code>City</code>를 바로 지정할 수 있도록</li>
  </ul>

  <p>“점(.)으로 경로를 확장해서 쓰는 속성 패턴” → Extended Property Pattern 이라고 부른다.</p>

  <hr />

  <h2>4. 확장 속성 패턴 예제 모음</h2>

  <h3>4-1. City + Country 동시에 체크하기</h3>

  <p>기존 방식:</p>

  <pre><code class="language-csharp">if (person is { Address: { City: "Seoul", Country: "KR" } })
{
    Console.WriteLine("한국 서울 거주자");
}</code></pre>

  <p>확장 속성 패턴으로 쓰면:</p>

  <pre><code class="language-csharp">if (person is { Address.City: "Seoul", Address.Country: "KR" })
{
    Console.WriteLine("한국 서울 거주자");
}</code></pre>

  <p>
    동작은 똑같지만, <code>Address.City</code>, <code>Address.Country</code>처럼  
    평소에 쓰던 프로퍼티 접근 방식과 거의 같아서 <strong>훨씬 읽기 편하다</strong>.
  </p>

  <h3>4-2. 속성 패턴 + switch 식</h3>

  <pre><code class="language-csharp">string Describe(Person p) =&gt; p switch
{
    { Address.City: "Seoul", Address.Country: "KR" }
        =&gt; "서울에 사는 한국인",

    { Address.Country: "KR" }
        =&gt; "한국에 사는 사람",

    { Address.Country: "US" }
        =&gt; "미국에 사는 사람",

    _   =&gt; "그 외"
};</code></pre>

  <p>
    원래라면 전부 <code>{ Address: { City: ..., Country: ... } }</code> 형태로 썼어야 한다.<br />
    확장 속성 패턴 덕분에, 우리가 평소에 생각하는
    <code>p.Address.City</code>, <code>p.Address.Country</code> 느낌을 그대로 패턴 안에서도 사용할 수 있게 된 것이다.
  </p>

  <hr />

  <h2>5. 확장 속성 패턴 정리</h2>

  <h3>5-1. 개념 요약</h3>

  <ul>
    <li><strong>기본 속성 패턴</strong>
      <ul>
        <li><code>obj is { Prop: value, OtherProp: &gt; 0 }</code></li>
        <li>→ 객체의 프로퍼티 값에 따라 매칭</li>
      </ul>
    </li>
    <li><strong>확장 속성 패턴</strong> (Extended Property Pattern)
      <ul>
        <li>중첩 객체에서도 <code>Address.City</code>처럼 점(.)으로 바로 접근</li>
        <li><code>obj is { Address.City: "Seoul" }</code></li>
        <li>→ <code>{ Address: { City: "Seoul" } }</code>와 동등한 표현</li>
      </ul>
    </li>
  </ul>

  <h3>5-2. 장점</h3>

  <ul>
    <li><strong>가독성 향상</strong>
      <ul>
        <li>중첩 중괄호를 줄여서 눈에 잘 들어온다.</li>
        <li>일반 C# 코드의 <code>obj.Address.City</code>와 사고방식이 거의 동일하다.</li>
      </ul>
    </li>
    <li><strong>복잡한 조건을 패턴 하나로 표현 가능</strong>
      <ul>
        <li>if-else로 늘어지던 조건을 switch + 패턴으로 깔끔하게 정리하기 좋다.</li>
      </ul>
    </li>
  </ul>

  <hr />

  <h2>6. 언제 써보면 좋을까?</h2>

  <ul>
    <li>DTO / ViewModel 등이 <strong>중첩 구조</strong>를 가질 때
      <ul>
        <li>예: <code>Order.Customer.Grade</code>, <code>Order.Shipping.Address.City</code> 등</li>
      </ul>
    </li>
    <li>상태 분기를 switch 식으로 깔끔하게 정리하고 싶을 때</li>
    <li>“조건문 지옥”이 되어 있는 코드를 리팩터링하면서
      <ul>
        <li><code>if (p.Address != null &amp;&amp; p.Address.City == "Seoul") ...</code></li>
        <li>같은 코드를 패턴 매칭 기반으로 바꿔보고 싶을 때</li>
      </ul>
    </li>
  </ul>

  <hr />

  <h2>7. 한 줄 요약</h2>

  <blockquote>
    확장 속성 패턴(Extended Property Pattern)은  
    <strong>“중첩 객체의 속성을 패턴 매칭에서 <code>Address.City</code>처럼 점(.)으로 편하게 표현하는 문법”</strong>이다.  
    기존 <code>{ Address: { City: ... } }</code> 를 더 읽기 쉬운 형태로 줄여준다고 생각하면 된다.
  </blockquote>

</article>