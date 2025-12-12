<article>



  <h2>1. dynamic vs reflection – 코드 스타일 / 사용성</h2>

  <h3>1) dynamic</h3>
  <ul>
    <li>마치 <strong>정적 타입인 것처럼 점(.) 찍고 바로 사용</strong>할 수 있다.</li>
    <li>문법이 깔끔하고 짧다.</li>
    <li>내부적으로는 <strong>런타임 바인딩</strong>(reflection + 캐시 등)을 사용해서 어떤 멤버를 호출할지 결정한다.</li>
  </ul>



  <h3>2) reflection</h3>
  <ul>
    <li><code>Type</code>, <code>MethodInfo</code>, <code>PropertyInfo</code> 같은 타입을 직접 다룬다.</li>
    <li>코드가 길어지고, 멤버 이름을 문자열로 다루기 때문에 <strong>오타</strong>에 취약하다.</li>
    <li>그 대신 <strong>더 저수준 제어가 가능</strong>하다. (멤버 나열, 특정 조건으로 필터링 등)</li>
  </ul>


  <hr />

  <h2>2. dynamic vs reflection – 기능 / 유연성 측면</h2>

  <h3>1) dynamic이 어울리는 경우</h3>
  <ul>
    <li><strong>“이 멤버가 있다는 걸 안다”</strong>는 전제 하에, 그냥 편하게 호출하고 싶을 때</li>
    <li>JSON / COM / 스크립트 객체처럼, <strong>“형태는 대충 아는데 타입 선언이 귀찮은”</strong> 경우</li>
  </ul>

  <p>예:</p>
  <pre><code class="language-csharp">dynamic json = JsonConvert.DeserializeObject(jsonText);

<p>Console.WriteLine(json.name);
Console.WriteLine(json.address.city);</code></pre></p>
  <p>
    DTO를 정식으로 만들기 전, 프로토타입/테스트 단계에서 빠르게 다뤄볼 때도 많이 사용한다.
  </p>

  <h3>2) reflection이 어울리는 경우</h3>
  <ul>
    <li>멤버 이름을 <strong>미리 모르는 상태에서</strong>, 런타임에 타입 구조를 탐색해야 할 때</li>
    <li>대표적인 사용 예:
      <ul>
        <li>DTO 자동 매핑 (프로퍼티 이름대로 값을 복사)</li>
        <li>객체의 모든 속성을 읽어서 로그 출력</li>
        <li>특정 <code>Attribute</code>가 붙은 멤버만 골라서 처리</li>
      </ul>
    </li>
    <li>dynamic보다 <strong>더 로우레벨</strong> 도구라서, “멤버가 무엇이 있는지”를 발견하는 작업에 적합하다.</li>
  </ul>

  <p>예:</p>
  <pre><code class="language-csharp">Type type = obj.GetType();
foreach (var prop in type.GetProperties())
{
    var value = prop.GetValue(obj);
    Console.WriteLine($"{prop.Name} = {value}");
}</code></pre>

  <hr />

  <h2>3. 성능 관점 </h2>

  <ul>
    <li><strong>정적 호출보다 둘 다 느리다</strong>는 점은 공통이다.</li>
    <li><code>dynamic</code>은 내부적으로 한 번 바인딩한 결과를 캐시하는 등 최적화가 들어가 있지만, 여전히 정적 호출보다는 비싸다.</li>
    <li><code>reflection</code>은 개발자가 잘못 쓰면 훨씬 비싸질 수 있다.
      <ul>
        <li>예: 루프 안에서 매번 <code>GetMethod</code>, <code>GetProperty</code> 호출</li>
      </ul>
    </li>
    <li>실무에서는 보통:
      <ul>
        <li>핵심 로직 / 성능이 중요한 반복문 안에서는 둘 다 피하고,</li>
        <li><strong>초기화, 설정, 매핑, 플러그인 로딩</strong> 같은 “바깥쪽”에서만 사용하는 경우가 많다.</li>
      </ul>
    </li>
  </ul>

  <hr />

  <h2>4. 에러가 언제 터지느냐 (에러 시점)</h2>

  <h3>1) dynamic</h3>
  <ul>
    <li>멤버가 없거나 시그니처가 맞지 않으면 → <strong>런타임에 <code>RuntimeBinderException</code></strong> 발생</li>
    <li>컴파일은 일단 다 통과한다.</li>
  </ul>

  <pre><code class="language-csharp">dynamic d = "Hello";
d.NonExistMethod(); // 컴파일 OK, 실행 시 RuntimeBinderException</code></pre>

  <h3>2) reflection</h3>
  <ul>
    <li>멤버를 찾을 때 <code>null</code> 여부를 체크해서, 예외를 직접 제어할 수 있다.</li>
    <li>단, 잘못된 방식으로 <code>Invoke</code>하면 여전히 런타임 예외가 발생한다.</li>
  </ul>

  <pre><code class="language-csharp">var mi = obj.GetType().GetMethod("Run");
if (mi != null)
{
    mi.Invoke(obj, null); // 시그니처가 맞지 않으면 여기서 예외
}
else
{
    Console.WriteLine("Run 메서드가 없습니다.");
}</code></pre>

  <hr />

  <h2>5. 함께 정리: dynamic / object / var</h2>

  <p>
    dynamic과 reflection을 이해할 때 자주 같이 언급되는 키워드가 <code>var</code>, <code>object</code>라서,
    간단하게 비교해 두면 기억하기 좋다.
  </p>

  <ul>
    <li><strong><code>var</code></strong>
      <ul>
        <li>그냥 <strong>컴파일 타임 타입 추론</strong>이다.</li>
        <li>한 번 타입이 결정되면 일반 변수와 똑같이 타입이 고정된다.</li>
        <li>타입 안전하고, 제일 많이 쓰는 기본 문법.</li>
      </ul>
    </li>
    <li><strong><code>object</code></strong>
      <ul>
        <li>모든 타입의 최상위 타입.</li>
        <li>형변환 + 박싱/언박싱이 필요할 수 있다.</li>
        <li>제네릭이 없던 시절 컬렉션에서 많이 쓰였던 방식 (예: <code>ArrayList</code>).</li>
      </ul>
    </li>
    <li><strong><code>dynamic</code></strong>
      <ul>
        <li>타입 체크를 <strong>런타임</strong>으로 미룬다.</li>
        <li>문법은 편하지만, 타입 안정성과 성능을 조금 희생한다.</li>
        <li>JSON/COM/스크립트/실험용 코드 등에 적당하다.</li>
      </ul>
    </li>
  </ul>

  <hr />




</article>