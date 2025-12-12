<article>

  <hr />

  <h2>1. 스레드(Thread)가 뭐야?</h2>

  <h3>1-1. 회사/직원 비유로 이해하기</h3>

  <p>
    먼저 비유부터 해 보자.
  </p>

  <ul>
    <li><strong>프로세스(Process)</strong> = 회사</li>
    <li><strong>스레드(Thread)</strong> = 회사 안에서 일하는 직원들</li>
  </ul>

  <p>
    어떤 프로그램(EXE)을 실행하면, 운영체제 입장에서는 <strong>프로세스 하나</strong>가 생긴다.<br />
    그리고 그 안에서 실제로 일을 하는 단위가 <strong>스레드</strong>다.
  </p>

  <p>
    예를 들어 엑셀을 켜면:
  </p>

  <ul>
    <li>엑셀 프로세스 1개</li>
    <li>그 안에
      <ul>
        <li>화면 그리는 스레드</li>
        <li>마우스/키보드 입력 받는 스레드</li>
        <li>자동 저장하는 스레드</li>
      </ul>
      같이 여러 스레드가 동시에 일을 할 수 있다.
    </li>
  </ul>

  <p>
    C# 콘솔 프로그램도 마찬가지다.
  </p>

  <ul>
    <li>프로그램이 실행되면 <strong>Main 스레드</strong> 하나로 시작한다.</li>
    <li>우리가 필요하다면 <code>Thread</code>를 만들어 스레드를 더 늘릴 수 있다.</li>
  </ul>

  <hr />

  <h2>2. C#에서 Thread 직접 만들어 보기</h2>

  <p>가장 기초적인 예제부터 보자.</p>

  <pre><code class="language-csharp">using System;
using System.Threading;

class Program
{
    static void Main()
    {
        Console.WriteLine("메인 스레드 시작");

        // 1) 새 스레드에서 실행할 작업 정의
        Thread t = new Thread(DoWork);

        // 2) 새 스레드 시작
        t.Start();

        // 3) 메인 스레드는 자기 할 일 계속
        for (int i = 0; i &lt; 5; i++)
        {
            Console.WriteLine($"[Main] i = {i}");
            Thread.Sleep(500); // 0.5초 쉬기
        }

        // 4) 새 스레드가 끝날 때까지 기다리고 싶으면 Join 사용
        t.Join();

        Console.WriteLine("메인 스레드 종료");
    }

    static void DoWork()
    {
        for (int i = 0; i &lt; 5; i++)
        {
            Console.WriteLine($"[Worker] i = {i}");
            Thread.Sleep(700); // 0.7초 쉬기
        }
    }
}
</code></pre>

  <h3>2-1. 이 코드에서 일어나는 일</h3>

  <ul>
    <li>프로그램 시작 → <strong>Main 스레드</strong> 시작</li>
    <li><code>new Thread(DoWork)</code> : 새 직원(스레드)을 한 명 더 채용</li>
    <li><code>t.Start()</code> : 그 직원에게 <code>DoWork</code> 일을 시킴</li>
    <li>Main 스레드는 <code>for</code> 루프를 돌면서 <code>[Main] ...</code> 출력</li>
    <li>새 스레드는 <code>DoWork</code> 안에서 <code>[Worker] ...</code> 출력</li>
  </ul>

  <p>
    콘솔에 <code>[Main] ...</code>, <code>[Worker] ...</code>가 섞여서 출력되면<br />
    두 개의 스레드가 동시에 일을 하고 있다는 뜻이다.
  </p>

  <ul>
    <li><code>Thread.Sleep(500)</code> : “이 스레드는 0.5초 동안 잠깐 쉰다”</li>
    <li><code>t.Join()</code> : “스레드 <code>t</code>가 끝날 때까지 여기서 기다린다”</li>
  </ul>

  <p>
    입문 단계에서는 <strong>“스레드를 직접 만들면 이렇게 여러 작업을 동시에 돌릴 수 있다”</strong> 정도만 이해하고 넘어가면 된다.
  </p>

  <hr />

  <h2>3. 스레드풀(ThreadPool)은 뭐야?</h2>

  <h3>3-1. 비유로 먼저 이해하기</h3>

  <p>
    방금 본 <code>new Thread(...)</code>는 이렇게 생각할 수 있다.
  </p>

  <blockquote>
    “작업 하나 처리하려고, 직원을 한 명 새로 채용한다.”
  </blockquote>

  <p>
    그런데 현실에서 이러면 비효율적이다.
  </p>

  <ul>
    <li>작업이 조금 생길 때마다 사람 채용 → 작업 끝나면 바로 해고</li>
    <li>채용/해고 비용이 너무 크다</li>
  </ul>

  <p>
    그래서 회사에서는 보통 이렇게 한다.
  </p>

  <blockquote>
    “기본 직원들을 몇 명 쭉 고용해 두고, 일이 생기면 그 직원들에게 나눠준다.”
  </blockquote>

  <p>
    이게 바로 <strong>스레드풀(Thread Pool)</strong> 개념이다.
  </p>

  <ul>
    <li>미리 만들어 놓은 스레드들의 “풀(pool)”이 있고</li>
    <li>우리는 “이 작업 좀 처리해줘”라고 <strong>일(작업)</strong>만 던져준다.</li>
    <li>풀 안에서 놀고 있는 스레드가 그 일을 가져다가 실행한다.</li>
  </ul>

  <p>
    .NET 내부에는 이미 <strong>ThreadPool</strong>이 준비되어 있고,  
    우리는 거기에 작업을 큐(queue)에 넣어 사용하면 된다.
  </p>

  <hr />

  <h2>4. ThreadPool 기본 사용 예제</h2>

  <pre><code class="language-csharp">using System;
using System.Threading;

class Program
{
    static void Main()
    {
        Console.WriteLine("메인 스레드 시작");

        // 스레드풀에 작업을 큐에 넣기
        ThreadPool.QueueUserWorkItem(DoWork);

        Console.WriteLine("메인 스레드는 계속 진행 중...");
        Console.ReadKey(); // 프로그램이 바로 종료되지 않게 대기
    }

    // 스레드풀용 콜백 메서드는 보통 object? 하나를 인수로 받는 시그니처를 가진다.
    static void DoWork(object? state)
    {
        Console.WriteLine(
            $"스레드풀 작업 시작, Thread ID = {Thread.CurrentThread.ManagedThreadId}");

        Thread.Sleep(1000); // 1초 정도 일하는 척
        Console.WriteLine("스레드풀 작업 끝");
    }
}
</code></pre>

  <h3>4-1. 포인트 정리</h3>

  <ul>
    <li><code>ThreadPool.QueueUserWorkItem(DoWork);</code>
      <ul>
        <li>“스레드풀에 이 메서드 실행할 작업 하나 넣어줘”라는 의미</li>
        <li>새 스레드를 직접 만드는 것이 아니라, 풀에 있는 스레드를 재사용한다.</li>
      </ul>
    </li>
    <li><code>DoWork(object? state)</code>
      <ul>
        <li>ThreadPool에서 호출할 메서드는 인자로 <code>object?</code> 하나 받는 형태가 기본</li>
        <li>지금 예제에서는 <code>state</code>를 사용하지 않으니까 <code>null</code> 그대로 둬도 된다.</li>
      </ul>
    </li>
  </ul>

  <hr />

  <h2>5. Thread 직접 생성 vs ThreadPool 비교</h2>

  <table border="1" cellpadding="6" cellspacing="0">
    <thead>
      <tr>
        <th>구분</th>
        <th><code>new Thread</code> 직접 생성</th>
        <th><code>ThreadPool</code> / <code>Task.Run</code></th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>스레드 생성 비용</td>
        <td>스레드 만들 때마다 비용 큼</td>
        <td>미리 만든 스레드를 재사용 → 효율적</td>
      </tr>
      <tr>
        <td>수명 관리</td>
        <td><code>Start</code>, <code>Join</code>, 종료 시점 직접 관리</td>
        <td>작업만 던지면 끝나면 알아서 반환</td>
      </tr>
      <tr>
        <td>개수 제한 관리</td>
        <td>너무 많이 만들면 OS에 부담</td>
        <td>.NET이 내부적으로 최대 스레드 수를 조절</td>
      </tr>
      <tr>
        <td>사용 난이도</td>
        <td>입문자에게는 다소 복잡</td>
        <td><code>Task.Run</code>, <code>async/await</code>로 비교적 쉬움</td>
      </tr>
      <tr>
        <td>요즘 권장 방식</td>
        <td>특수한 경우 아니면 잘 안 씀</td>
        <td>대부분 스레드풀 + Task / async 조합 사용</td>
      </tr>
    </tbody>
  </table>

  <p>
    실무에서는 보통:
  </p>

  <blockquote>
    <strong>직접 <code>Thread</code>를 만드는 경우는 특수한 상황이고,</strong><br />
    <strong>대부분은 스레드풀 + <code>Task</code> / <code>async/await</code> 조합으로 처리한다.</strong>
  </blockquote>

  <hr />

  <h2>6. 사실 <code>Task.Run</code>도 스레드풀 위에서 돈다</h2>

  <p>
    요즘 C# 코드에서는 <code>ThreadPool.QueueUserWorkItem</code> 대신
    <strong><code>Task.Run</code></strong>을 훨씬 더 자주 쓴다.
  </p>

  <pre><code class="language-csharp">using System;
using System.Threading;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        Console.WriteLine("메인 시작");

        // Task.Run 안의 코드는 스레드풀 스레드에서 실행된다.
        await Task.Run(() =>
        {
            Console.WriteLine(
                $"백그라운드 작업, Thread ID = {Thread.CurrentThread.ManagedThreadId}");
            Thread.Sleep(1000);
            Console.WriteLine("백그라운드 작업 끝");
        });

        Console.WriteLine("메인 종료");
    }
}
</code></pre>

  <h3>6-1. 여기서 알아둘 점</h3>

  <ul>
    <li><code>Task.Run(() =&gt; { ... })</code>는 내부적으로 <strong>스레드풀에 작업을 던지는 것</strong>이라고 생각하면 된다.</li>
    <li>우리는 “이 코드를 백그라운드에서 돌려줘” 정도의 느낌만 가지고 사용해도 된다.</li>
    <li><code>await</code>를 쓰면, 그 작업이 끝난 다음에 다시 이어서 실행할 수 있다.</li>
  </ul>

  <p>
    그래서 요약하면:
  </p>

  <ul>
    <li>과거: <code>new Thread</code> 또는 <code>ThreadPool.QueueUserWorkItem</code> 사용</li>
    <li>현재: 대부분 <code>Task</code> + <code>async/await</code> 사용  
      (내부에서는 결국 스레드풀이 일을 한다)</li>
  </ul>

  <hr />

  <h2>7. 실전 느낌으로 보는 간단 예제</h2>

  <p>
    화면(UI)을 가진 프로그램(윈폼, WPF 등)에서는<br />
    <strong>무거운 작업을 메인(UI) 스레드에서 돌리면 화면이 멈춘다.</strong><br />
    그래서 보통 스레드풀/백그라운드에서 작업을 돌리고, 결과만 가져온다.
  </p>

  <p>
    콘솔에서 비슷한 느낌만 보이는 예제를 하나 보자.
  </p>

  <pre><code class="language-csharp">using System;
using System.Threading;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        Console.WriteLine("프로그램 시작");

        // 무거운 작업을 스레드풀에서 실행
        Task&lt;long&gt; work = Task.Run(() =>
        {
            Console.WriteLine("무거운 작업 시작");
            Thread.Sleep(3000); // 3초 걸리는 작업이라고 가정
            Console.WriteLine("무거운 작업 끝");
            return 12345L; // 결과 예시
        });

        Console.WriteLine("메인은 다른 일 계속 할 수 있음...");

        // 여기서 결과를 기다림 (중간에 UI라면 다른 이벤트 처리 가능)
        long result = await work;

        Console.WriteLine($"작업 결과: {result}");
        Console.WriteLine("프로그램 종료");
    }
}
</code></pre>

  <h3>7-1. 실행 흐름</h3>

  <ol>
    <li><code>프로그램 시작</code> 출력</li>
    <li><code>무거운 작업 시작</code> 출력 (스레드풀 스레드)</li>
    <li><code>메인은 다른 일 계속 할 수 있음...</code> 출력 (메인 스레드)</li>
    <li>3초 후 <code>무거운 작업 끝</code> 출력</li>
    <li><code>await work</code> 이후 <code>작업 결과: 12345</code> 출력</li>
    <li><code>프로그램 종료</code> 출력</li>
  </ol>

  <p>
    중요한 건, <strong>무거운 작업 때문에 메인 흐름이 멈추지 않는다</strong>는 점이다.<br />
    이런 패턴이 UI 프로그램이나 서버 프로그램에서 매우 자주 사용된다.
  </p>

  <hr />

  <h2>8. 입문자가 꼭 기억하면 좋은 포인트</h2>

  <ul>
    <li>
      <strong>스레드(Thread)</strong>
      <ul>
        <li>프로그램 안에서 실제로 일을 하는 실행 흐름</li>
        <li>C# 프로그램은 기본적으로 Main 스레드 1개로 시작</li>
        <li><code>new Thread(...)</code> 로 직접 스레드를 만들 수 있지만, 요즘은 특별한 경우에만 사용</li>
      </ul>
    </li>
    <li>
      <strong>스레드풀(ThreadPool)</strong>
      <ul>
        <li>.NET이 미리 만들어 관리하는 “스레드 직원 풀”</li>
        <li><code>ThreadPool.QueueUserWorkItem</code> 또는 <code>Task.Run</code>으로 작업을 던지면, 풀 안의 스레드가 실행</li>
        <li>스레드를 직접 만드는 것보다 효율적이고 안전한 경우가 많다.</li>
      </ul>
    </li>
    <li>
      <strong>Task / async / await</strong>
      <ul>
        <li>스레드/스레드풀을 좀 더 쉽게 쓰도록 도와주는 고수준 도구</li>
        <li>내부적으로는 스레드풀이나 비동기 I/O를 활용</li>
        <li>입문자는 “비동기 코드는 Task + async/await로 작성한다”부터 익숙해지는 것이 좋다.</li>
      </ul>
    </li>
    <li>
      <strong>UI 프로그램에서는</strong>
      <ul>
        <li>무거운 작업을 메인(UI) 스레드에서 직접 돌리면 화면이 멈춘다.</li>
        <li>스레드풀/백그라운드에서 일을 돌리고, 결과만 UI 스레드로 가져오는 패턴을 사용한다.</li>
      </ul>
    </li>
  </ul>

  <hr />

  <h2>9. 이후 찾아보면 좋은 부분</h2>

  <p>스레드와 스레드풀 개념을 이해했다면, 다음 단계로 이런 것들을 공부해 보면 좋다.</p>

  <ul>
    <li><strong>동기화</strong> : <code>lock</code>, <code>Monitor</code>, <code>Interlocked</code> 등</li>
    <li><strong>스레드 안전(Thread-Safety)</strong> 개념</li>
    <li><strong>async/await</strong>가 스레드를 새로 만드는 게 아니라는 점 (I/O 비동기 개념)</li>
  </ul>


</article>