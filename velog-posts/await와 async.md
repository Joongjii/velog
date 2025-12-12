<article>

  <hr />

  <h2>1. 한 줄 정의</h2>

  <h3>async</h3>
  <blockquote>
    메서드나 람다 <strong>선언부</strong>에 붙이는 키워드.<br />
    “이 메서드는 비동기 스타일로 작성되었고, <code>Task</code> 또는 <code>Task&lt;T&gt;</code>를 반환할 거야” 라는 표시.
  </blockquote>

  <h3>await</h3>
  <blockquote>
    메서드/람다 <strong>몸통 안</strong>에서 쓰는 키워드(연산자).<br />
    “이 비동기 작업(<code>Task</code>)이 끝날 때까지 <em>비동기적으로</em> 기다렸다가,  
    끝나면 그 다음 줄부터 이어서 실행해 줘.” 라는 뜻.
  </blockquote>

  <hr />

  <h2>2. 어디에 쓰는 키워드인가?</h2>

  <h3>2-1. async는 ‘메서드 선언부’에 붙는다</h3>

  <pre><code class="language-csharp">async Task DoWorkAsync()
{
    // 여기 안에서 await 사용 가능
}

async Task&lt;int&gt; GetNumberAsync()
{
    await Task.Delay(1000);
    return 10;
}
</code></pre>

  <ul>
    <li><code>async</code>는 메서드 이름 앞에 붙는다.</li>
    <li>“이 메서드는 비동기 상태머신으로 변환된다”는 신호를 컴파일러에 주는 역할을 한다.</li>
    <li>반환 타입은 보통 <code>Task</code> / <code>Task&lt;T&gt;</code> / (특수한 경우) <code>void</code> 중 하나다.</li>
  </ul>

  <p>
    중요한 점 하나:  
    <strong><code>async</code>를 붙였다고 자동으로 새 스레드가 만들어지는 게 아니다.</strong><br />
    그냥 “비동기 로직을 담고 있고, 안에서 <code>await</code> 쓸 거야” 정도의 표지판에 가깝다.
  </p>

  <h3>2-2. await는 ‘메서드 몸통 안’에서만 쓴다</h3>

  <pre><code class="language-csharp">async Task TestAsync()
{
    await Task.Delay(1000); // OK
}

void Test()
{
    await Task.Delay(1000); // ❌ 컴파일 에러: async 메서드가 아니라서
}
</code></pre>

  <ul>
    <li><code>await</code>는 <strong>반드시 <code>async</code>가 붙은 메서드/람다 안</strong>에서만 사용할 수 있다.</li>
    <li><code>await someTask</code>는 “<code>someTask</code>가 끝날 때까지 비동기적으로 기다렸다가, 다시 여기로 돌아와서 아래 코드를 이어서 실행하자” 라는 의미다.</li>
    <li><code>Task&lt;T&gt;</code>를 await 하면, <code>T</code> 결과값을 바로 꺼내 쓸 수 있다.</li>
  </ul>

  <pre><code class="language-csharp">int result = await GetNumberAsync();  // GetNumberAsync가 Task&lt;int&gt;를 반환한다고 가정
</code></pre>

  <p>
    위 한 줄은 사실상:
  </p>

  <ol>
    <li><code>GetNumberAsync()</code>가 시작되고 <code>Task&lt;int&gt;</code> 하나가 만들어진다.</li>
    <li>그 Task가 완료될 때까지 비동기적으로 대기한다.</li>
    <li>완료되면 그 안의 <code>int</code> 값을 꺼내서 <code>result</code>에 넣는다.</li>
  </ol>

  <hr />

  <h2>3. 역할 관점에서 보는 async vs await</h2>

  <h3>3-1. async의 역할</h3>

  <ul>
    <li>이 메서드를 <strong>비동기 상태머신</strong>으로 변환하도록 컴파일러에게 지시한다.</li>
    <li>이 안에 <code>await</code>를 쓸 수 있게 해준다.</li>
    <li>반환 타입을 <code>Task</code> / <code>Task&lt;T&gt;</code> / <code>void</code> 중 하나로 맞추게 만든다.</li>
  </ul>

  <p>
    흔한 오해 중 하나가:
  </p>

  <blockquote>
    “async 붙이면 자동으로 다른 스레드에서 실행된다”
  </blockquote>

  <p>
    이건 아니다.  
    새 스레드를 만드는 건 <code>Task.Run</code>, <code>Thread</code>, <code>ThreadPool</code> 같은 애들이고,  
    <code>async</code>는 단지 비동기 흐름을 <strong>코드 구조</strong>로 표현하기 위한 키워드다.
  </p>

  <h3>3-2. await의 역할</h3>

  <ul>
    <li><code>Task</code> 또는 <code>Task&lt;T&gt;</code>가 <strong>완료될 때까지 비동기적으로 대기</strong>한다.</li>
    <li>대기하는 동안 <strong>스레드를 블록시키지 않는다</strong> (UI 프리징 방지의 핵심).</li>
    <li>작업이 끝나면 이어서 아래 줄 코드를 실행한다.</li>
    <li><code>Task&lt;T&gt;</code>를 await하면, 결과 <code>T</code>를 바로 받을 수 있다.</li>
  </ul>

  <pre><code class="language-csharp">async Task DemoAsync()
{
    Console.WriteLine("1");

    await Task.Delay(1000); // 1초 후, 다시 여기로 돌아와서

    Console.WriteLine("2"); // 이 줄부터 이어서 실행
}
</code></pre>

  <p>
    여기서 <code>await Task.Delay(1000)</code>를 <code>Thread.Sleep(1000)</code>으로 바꾸면  
    1초 동안 스레드가 진짜로 멈춰버린다.  
    <code>await</code>는 “기다린다”는 의미는 같지만, <strong>스레드를 점유하지 않는다는 점이 결정적으로 다르다</strong>.
  </p>

  <hr />

  <h2>4. async와 await의 관계</h2>

  <h3>4-1. async만 있고 await가 없을 때</h3>

  <pre><code class="language-csharp">async Task FooAsync()
{
    Console.WriteLine("Hello");
    // await 없음
}
</code></pre>

  <p>
    이런 코드는 사실상 동기 메서드와 거의 같다.  
    컴파일러가 내부적으로 Task를 감싸긴 하지만, 중간에 “끊기는 지점”이 없기 때문에  
    흐름은 그냥 위에서 아래로 쭉 실행된다.
  </p>

  <p>
    그래서 <strong>진짜 비동기의 느낌이 나는 지점은 결국 <code>await</code>를 만나는 순간부터</strong>라고 보면 된다.
  </p>

  <h3>4-2. await만 쓰고 async를 안 쓸 때</h3>

  <pre><code class="language-csharp">void Foo()
{
    await Task.Delay(1000); // ❌ 컴파일 에러
}
</code></pre>

  <p>
    이런 코드는 컴파일 자체가 안 된다.  
    <strong>await는 반드시 async 메서드 안에서만 사용 가능</strong>하기 때문이다.
  </p>

  <p>
    즉, 둘의 관계를 요약하면:
  </p>

  <ul>
    <li><code>await</code>는 <code>async</code> 없이 쓸 수 없다.</li>
    <li><code>async</code>는 <code>await</code> 없이 쓸 수는 있지만, 의미가 거의 없다.</li>
  </ul>

  <hr />

  <h2>5. 간단 예제로 전체 흐름 다시 보기</h2>

  <pre><code class="language-csharp">async Task&lt;int&gt; GetNumberAsync()
{
    await Task.Delay(1000); // 1초짜리 비동기 작업
    return 10;
}

async Task ShowAsync()
{
    Console.WriteLine("시작");

    int n = await GetNumberAsync();   // 여기서 비동기 대기 + 결과 받기

    Console.WriteLine("결과: " + n);
}
</code></pre>

  <ul>
    <li><code>GetNumberAsync</code>  
      → “나중에 <code>int</code>를 줄 비동기 메서드야” (반환 타입: <code>Task&lt;int&gt;</code>)</li>
    <li><code>await Task.Delay(1000)</code>  
      → 1초 뒤에 다시 아래 줄부터 이어서 실행</li>
    <li><code>int n = await GetNumberAsync();</code>  
      → <code>Task&lt;int&gt;</code>가 끝날 때까지 기다렸다가, 안의 <code>int</code> 값(<code>10</code>)을 꺼내서 <code>n</code>에 대입</li>
  </ul>

  <hr />

  <h2>6. 한 번에 요약</h2>

  <ul>
    <li><strong>async</strong>
      <ul>
        <li>메서드/람다 선언부에 붙는다.</li>
        <li>“이 메서드는 비동기 스타일로 작성됐다, 안에서 await 쓸 거다” 라는 표시.</li>
        <li>반환 타입은 <code>Task</code>, <code>Task&lt;T&gt;</code>, <code>void</code>(이벤트 핸들러) 중 하나.</li>
      </ul>
    </li>
    <li><strong>await</strong>
      <ul>
        <li>비동기 작업(<code>Task</code>, <code>Task&lt;T&gt;</code>) 앞에 붙는 연산자.</li>
        <li>해당 Task가 완료될 때까지 비동기적으로 기다린 뒤, 다음 줄부터 이어서 실행.</li>
        <li><code>Task&lt;T&gt;</code>라면 결과 <code>T</code>를 바로 꺼내서 변수에 넣어줌.</li>
      </ul>
    </li>
  </ul>

  <p>
    정말 짧게 말하면:
  </p>

  <blockquote>
    <strong>async</strong>는 “이 메서드는 비동기 모드로 작성할게” 라고 선언하는 스위치이고,<br />
    <strong>await</strong>는 “이 비동기 작업이 끝날 때까지 잠깐 맡겨 두고, 끝나면 여기서부터 다시 시작하자” 라고 말하는 실행 지점이다.
  </blockquote>


</article>