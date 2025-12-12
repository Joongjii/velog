<article>

  <hr />

  <h2>1. 한 줄 정리부터</h2>

  <blockquote>
    <strong>
      <code>await</code>로 기다리는 비동기 메서드에서 예외가 발생하면,<br />
      그 예외는 <code>await</code>가 있는 줄에서 터지는 것처럼 동작하고,<br />
      그 주변의 <code>try / catch</code>로 잡을 수 있다.
    </strong>
  </blockquote>

  <p>
    즉, “비동기라서 예외 처리가 완전 다르다”가 아니라,<br />
    <strong>“예외가 <code>await</code> 줄에서 터진다고 생각하면 된다”</strong> 정도로 이해하면 편하다.
  </p>

  <hr />

  <h2>2. 동기 코드 vs 비동기 코드 비교</h2>

  <h3>2-1. 동기 코드에서의 <code>try / catch</code></h3>

  <pre><code class="language-csharp">try
{
    DoSomething();  // 여기서 예외 나면 바로 catch로 감
}
catch (Exception ex)
{
    Console.WriteLine("에러 발생: " + ex.Message);
}</code></pre>

  <p>
    <code>DoSomething()</code> 안에서 예외가 발생하면,<br />
    바로 위의 <code>try / catch</code> 블록으로 예외가 전파된다.
  </p>

  <h3>2-2. 비동기 코드에서의 <code>await</code> + <code>try / catch</code></h3>

  <pre><code class="language-csharp">try
{
    await DoSomethingAsync();  // 이 줄에서 Task가 완료될 때 예외가 있으면 여기서 throw
}
catch (Exception ex)
{
    Console.WriteLine("에러 발생: " + ex.Message);
}</code></pre>

  <p>
    비동기 메서드 안에서 예외가 나면:
  </p>

  <ol>
    <li>예외가 바로 밖으로 튀어나오는 게 아니라, <strong>Task 안에 저장된다</strong>.</li>
    <li>나중에 그 Task를 <code>await</code>할 때, <strong>그 줄에서 예외를 다시 던진다(throw)</strong>.</li>
    <li>그래서 <code>await</code>를 둘러싼 <code>try / catch</code>에서 예외를 잡을 수 있다.</li>
  </ol>

  <p>
    결국 “예외는 <strong><code>await</code> 줄에서</strong> 터진다”고 보면 된다.
  </p>

  <hr />

  <h2>3. 기본 예제로 흐름 이해하기</h2>

  <pre><code class="language-csharp">static async Task Main()
{
    try
    {
        Console.WriteLine("요청 시작");
        string result = await DownloadDataAsync();  // 여기에서 예외가 다시 throw 됨
        Console.WriteLine("결과: " + result);
    }
    catch (Exception ex)
    {
        Console.WriteLine("예외 잡음: " + ex.Message);
    }

    Console.WriteLine("메인 계속 실행");
}

static async Task&lt;string&gt; DownloadDataAsync()
{
    // 예를 들어 HttpClient 호출 같은 비동기 작업이라고 가정
    await Task.Delay(1000); // 네트워크 대기라고 생각

    // 일부러 에러 발생
    throw new InvalidOperationException("다운로드 중 오류 발생!");
}</code></pre>

  <p>실행 흐름을 찬찬히 따라가면:</p>

  <ol>
    <li><code>DownloadDataAsync()</code>를 호출 → <code>Task&lt;string&gt;</code>를 반환</li>
    <li><code>await</code>가 이 Task가 끝날 때까지 기다린다.</li>
    <li><code>DownloadDataAsync</code> 내부에서 <code>throw</code> 발생.</li>
    <li>예외 정보가 Task 안에 저장된다.</li>
    <li>Task 완료 후 <code>await</code> 지점으로 돌아올 때, 그 예외를 다시 던진다.</li>
    <li>바깥의 <code>try / catch</code>에서 이 예외를 잡는다.</li>
  </ol>

  <p>
    그래서 “예외는 DownloadDataAsync 안에서 났지만,<br />
    실제로 잡는 위치는 <strong><code>await DownloadDataAsync()</code> 줄의 <code>catch</code></strong>라고 보면 된다.
  </p>

  <hr />

  <h2>4. 예외를 어디서 잡을지에 따른 패턴</h2>

  <h3>4-1. async 메서드 안에서 직접 처리</h3>

  <pre><code class="language-csharp">static async Task DownloadAndPrintAsync()
{
    try
    {
        string result = await DownloadDataAsync();
        Console.WriteLine("결과: " + result);
    }
    catch (Exception ex)
    {
        Console.WriteLine("DownloadAndPrintAsync 내부에서 처리: " + ex.Message);
    }
}</code></pre>

  <ul>
    <li>이 경우, 예외는 <code>DownloadAndPrintAsync</code> 안에서 끝까지 처리된다.</li>
    <li>호출하는 쪽에서는 예외가 올라오지 않는다.</li>
  </ul>

  <h3>4-2. 호출하는 쪽(상위)에서 처리</h3>

  <pre><code class="language-csharp">static async Task DownloadAndPrintAsync()
{
    // 여기서는 예외를 처리하지 않고 위로 올림
    string result = await DownloadDataAsync(); 
    Console.WriteLine("결과: " + result);
}

static async Task Main()
{
    try
    {
        await DownloadAndPrintAsync();
    }
    catch (Exception ex)
    {
        Console.WriteLine("Main에서 예외 처리: " + ex.Message);
    }
}</code></pre>

  <ul>
    <li>여기서는 <code>DownloadAndPrintAsync</code> 안에서 예외 처리 없이 그대로 던진다.</li>
    <li>최상위 (<code>Main</code> 또는 UI 레이어)에서 한 번에 예외를 처리하는 구조.</li>
  </ul>

  <p>
    결국 <strong>“예외를 어디 레벨에서 책임지고 처리할 것인가?”</strong>를 설계하는 문제다.
  </p>

  <hr />

  <h2>5. <code>await</code>를 안 쓰면 어떻게 되나? (중요 포인트)</h2>

  <pre><code class="language-csharp">// ❌ 이렇게 하면 예외가 try/catch에서 안 잡힐 수 있음
try
{
    Task t = DoSomethingAsync();  // Task만 만든 상태, await 안 함
    // 다른 작업...
}
catch (Exception ex)
{
    Console.WriteLine("여기선 예외 못 잡음");
}</code></pre>

  <ul>
    <li><code>await</code>를 하지 않으면, 예외가 **Task 안에서만** 발생하고 호출 스택으로 올라오지 않는다.</li>
    <li>나중에 <code>t.Wait()</code>나 <code>t.Result</code>를 사용할 때 예외가 튀어나오는데,<br />
        이때는 <code>AggregateException</code> 같은 형태로 감싸져서 나와서 더 복잡해진다.</li>
  </ul>

  <p>
    그래서 <strong>“비동기 예외를 <code>try / catch</code>로 제대로 처리하고 싶으면 반드시 <code>await</code> 해야 한다”</strong>라고 생각하면 편하다.
  </p>

  <hr />

  <h2>6. 여러 비동기 작업을 동시에 기다릴 때 (Task.WhenAll)</h2>

  <p>
    여러 개 Task를 동시에 돌려놓고 한 번에 기다릴 때도 패턴은 똑같다.
  </p>

  <pre><code class="language-csharp">try
{
    Task t1 = DoWorkAsync(1);
    Task t2 = DoWorkAsync(2);
    Task t3 = DoWorkAsync(3);

    await Task.WhenAll(t1, t2, t3);  // 여기서 여러 Task 예외가 한 번에 표출
}
catch (Exception ex)
{
    Console.WriteLine("WhenAll에서 예외 잡음: " + ex.Message);
}</code></pre>

  <ul>
    <li><code>Task.WhenAll</code>에 들어간 Task 중 하나라도 예외가 나면,</li>
    <li><code>await Task.WhenAll(...)</code> 줄에서 예외가 터진다.</li>
    <li>역시 그 줄을 <code>try / catch</code>로 감싸면 예외를 잡을 수 있다.</li>
    <li>실제로는 여러 예외가 한 번에 발생할 수도 있어서 내부적으로 <code>AggregateException</code>에 담기기도 한다.<br />
        처음에는 “여러 개 Task → <code>WhenAll</code>에서 한 번에 터진다” 정도만 기억해 두면 충분하다.</li>
  </ul>

  <hr />

  <h2>7. 자주 쓰는 패턴 정리</h2>

  <h3>패턴 1) async 메서드 안에서 try / catch</h3>

  <pre><code class="language-csharp">static async Task DoSomethingSafeAsync()
{
    try
    {
        await DoSomethingDangerousAsync();
    }
    catch (Exception ex)
    {
        // 여기서 로그 찍고 끝내기
        Console.WriteLine("내부 처리: " + ex.Message);
    }
}</code></pre>

  <h3>패턴 2) async 메서드는 그냥 던지고, 호출자에서 처리</h3>

  <pre><code class="language-csharp">static async Task DoSomethingAsync()
{
    await DoSomethingDangerousAsync(); // 예외를 위로 던짐
}

static async Task Main()
{
    try
    {
        await DoSomethingAsync();
    }
    catch (Exception ex)
    {
        Console.WriteLine("최상위에서 처리: " + ex.Message);
    }
}</code></pre>

  <p>
    상황에 따라 “어디서 예외를 책임질지”를 선택해서 이 두 패턴을 섞어 쓰게 된다.
  </p>

  <hr />

</article>