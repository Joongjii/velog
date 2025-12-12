<article>


  <pre><code class="language-csharp">int* p = stackalloc int[10];</code></pre>


  <pre><code class="language-csharp">Span&lt;int&gt; buffer = stackalloc int[10];</code></pre>

  <p>
    처음 보면 “이게 대체 뭐지?” 싶은데, 한 번 잡고 가면 <br />  
    <strong>스택/힙 구조, 고성능 코드</strong> 이해에 꽤 도움이 되는 키워드다.
  </p>

  <hr />

  <h2>1. <code>stackalloc</code> 한 줄 정의</h2>

  <blockquote>
    <strong><code>stackalloc</code>은 “힙(heap)이 아니라 <u>스택(stack)</u>에 작은 배열(버퍼)을 직접 만들어 쓰자”는 키워드</strong>다.
  </blockquote>

  <p>
    보통 우리는 이렇게 배열을 만든다.
  </p>

  <pre><code class="language-csharp">int[] arr = new int[10];</code></pre>

  <ul>
    <li>이 배열은 <strong>힙(Heap)</strong>에 만들어지고</li>
    <li>사용이 끝나면 <strong>가비지 컬렉터(GC)</strong>가 언젠가 치워 준다.</li>
  </ul>

  <p>
    반면 <code>stackalloc</code>을 쓰면:
  </p>

  <pre><code class="language-csharp">int* p = stackalloc int[10]; // 스택에 int 10개짜리 버퍼 생성</code></pre>

  <ul>
    <li><strong>스택(Stack)</strong>에 <code>int</code> 10개짜리 버퍼가 만들어지고</li>
    <li>해당 메서드/블록을 빠져나가면, 스택 프레임이 정리되면서 <strong>자동으로 사라진다</strong>.</li>
  </ul>

  <p>
    즉, 한 마디로 요약하면:
  </p>

  <blockquote>
    “잠깐 쓸 <strong>작은 배열</strong>을 힙 대신 스택에 올려서 <strong>빠르게 쓰고 한 번에 정리하자</strong>”
  </blockquote>

  <hr />

  <h2>2. 왜 굳이 써야 할까?</h2>

  <p>
    일반적인 업무 코드에서는 <code>stackalloc</code>을 자주 쓰지 않는다.  
    하지만 다음과 같은 상황에서는 꽤 유용하다.
  </p>

  <h3>2-1. 고성능 코드 (자주 호출되는 함수)</h3>

  <ul>
    <li>예: 초당 수십만 번 호출되는 파싱/인코딩 함수</li>
    <li>매번 <code>new byte[1024]</code>를 만들어서 쓰면:
      <ul>
        <li>힙에 계속 배열이 생기고</li>
        <li>GC가 자주 돌면서 성능에 부담이 된다.</li>
      </ul>
    </li>
    <li><code>stackalloc</code>으로 스택에 버퍼를 만들면:
      <ul>
        <li>할당/해제가 <strong>엄청 빠르다</strong> (스택 포인터만 위아래로 움직이는 수준)</li>
      </ul>
    </li>
  </ul>

  <h3>2-2. 네이티브/Interop 작업</h3>

  <ul>
    <li>C, C++ DLL(P/Invoke)을 호출할 때 임시 버퍼가 필요할 수 있다.</li>
    <li>이때 <code>byte*</code>, <code>int*</code> 같은 포인터 버퍼를 <code>stackalloc</code>으로 만들고 바로 네이티브 함수에 넘길 수 있다.</li>
  </ul>

  <h3>2-3. <code>Span&lt;T&gt;</code>와 함께 쓰는 고성능 문자열/버퍼 처리</h3>

  <ul>
    <li>예: <code>Span&lt;byte&gt; buffer = stackalloc byte[256];</code></li>
    <li>문자열 파싱, 인코딩/디코딩, 바이너리 처리 등에서 작은 임시 버퍼를 자주 쓸 때 성능에 도움이 된다.</li>
  </ul>

  <p>
    정리하면:
  </p>

  <blockquote>
    <strong>“작고, 일시적이고, 성능이 중요한 버퍼”</strong>가 필요한 경우 → <code>stackalloc</code> 후보
  </blockquote>

  <hr />

  <h2>3. 기본 문법 – 두 가지 스타일</h2>

  <h3>3-1. 포인터 기반 (unsafe 코드)</h3>

  <p>
    가장 기본 형태는 C 스타일 포인터와 함께 쓰는 것이다.
  </p>

  <pre><code class="language-csharp">unsafe
{
    int* p = stackalloc int[5];  // 스택에 int 5개짜리 버퍼 생성

    for (int i = 0; i &lt; 5; i++)
    {
        p[i] = i * 10;
    }

    for (int i = 0; i &lt; 5; i++)
    {
        Console.WriteLine(p[i]);
    }
}</code></pre>

  <ul>
    <li><code>unsafe</code> 문맥이 필요하다.</li>
    <li><code>int*</code>, <code>byte*</code> 같은 포인터 연산을 직접 쓸 수 있다.</li>
    <li>C 코드와 거의 비슷한 느낌이지만, 그만큼 실수하면 위험할 수 있다.</li>
  </ul>

  <h3>3-2. <code>Span&lt;T&gt;</code> 기반 (요즘 추천 스타일)</h3>

  <p>
    C# 7.2+에서는 <code>Span&lt;T&gt;</code>와 함께 쓰면 <strong>unsafe 없이</strong>도 <code>stackalloc</code>을 사용할 수 있다.
  </p>

  <pre><code class="language-csharp">using System;

class Program
{
    static void Main()
    {
        Span&lt;int&gt; buffer = stackalloc int[5]; // 스택에 int 5개

        for (int i = 0; i &lt; buffer.Length; i++)
        {
            buffer[i] = i * 10;
        }

        foreach (var x in buffer)
        {
            Console.WriteLine(x);
        }
    }
}</code></pre>

  <ul>
    <li><code>Span&lt;T&gt;</code>는 “연속된 메모리 블록”을 안전하게 다루기 위한 타입이다.</li>
    <li><code>stackalloc</code>으로 만든 버퍼를 <code>Span</code>으로 감싸면:
      <ul>
        <li>포인터를 직접 안 써도 되고</li>
        <li>범위 체크(인덱스 검사)도 해 줘서 더 안전하다.</li>
      </ul>
    </li>
  </ul>

  <hr />

  <h2>4. 스택에 만든다는 건 어떤 의미인가?</h2>

  <p>
    “스택에 만든다”는 말은 단순히 위치만 다른 게 아니라,  
    <strong>수명, 속도, 크기 제한</strong>에서 차이가 난다는 뜻이다.
  </p>

  <h3>4-1. 수명(lifetime)이 짧다</h3>

  <ul>
    <li>스택에 올린 버퍼는 <strong>해당 메서드/블록 안에서만 유효</strong>하다.</li>
    <li>메서드에서 빠져나가는 순간 스택 프레임이 정리되면서 메모리가 같이 날아간다.</li>
    <li>그래서 <strong>바깥으로 주소/참조를 내보내면 절대 안 된다.</strong></li>
  </ul>

  <p>예를 들어, 이런 코드는 위험하다.</p>

  <pre><code class="language-csharp">unsafe int* MakeBuffer()
{
    int* p = stackalloc int[10];
    return p; // ❌ 함수가 끝나면 p가 가리키는 메모리는 이미 사라진 상태
}</code></pre>

  <h3>4-2. 할당/해제가 매우 빠르다</h3>

  <ul>
    <li>스택은 단순히 “위로 쌓았다가, 한 번에 내려오는” 구조다.</li>
    <li>버퍼를 만들 때 스택 포인터를 조금 올리고, 함수 끝날 때 한 번에 내려온다.</li>
    <li>힙 할당 + GC 부담에 비해 <strong>엄청 가볍다</strong>.</li>
  </ul>

  <h3>4-3. 크기를 너무 크게 잡으면 위험하다</h3>

  <ul>
    <li>스택은 용량이 제한적이다.</li>
    <li><code>stackalloc byte[256]</code> 정도는 괜찮지만,  
      <code>stackalloc byte[1024 * 1024]</code> 이런 건 <strong>스택 오버플로우 위험</strong>이 있다.</li>
    <li>일반적으로 <strong>수십~수백 개 원소, 몇 KB 수준 버퍼</strong>에 사용하는 것이 적당하다.</li>
  </ul>

  <hr />

  <h2>5. <code>new</code> 배열과 비교 – heap vs stack</h2>

  <p>같은 <code>int</code> 100개라도, 다음 둘은 완전히 다르게 동작한다.</p>

  <pre><code class="language-csharp">// 1) 힙에 생성
int[] arr = new int[100];

// 2) 스택에 생성
Span&lt;int&gt; buf = stackalloc int[100];</code></pre>

  <table border="1" cellpadding="6" cellspacing="0">
    <thead>
      <tr>
        <th>구분</th>
        <th><code>new int[100]</code> (Heap)</th>
        <th><code>stackalloc int[100]</code> (Stack)</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>메모리 위치</td>
        <td>힙(Heap)</td>
        <td>스택(Stack)</td>
      </tr>
      <tr>
        <td>수명</td>
        <td>변수를 더 이상 참조하지 않을 때, GC가 수거</td>
        <td>현재 메서드/블록 끝날 때 자동 소멸</td>
      </tr>
      <tr>
        <td>할당 비용</td>
        <td>상대적으로 비쌈 (힙 관리 + GC 부담)</td>
        <td>매우 빠름 (스택 포인터 이동 수준)</td>
      </tr>
      <tr>
        <td>크기 제한</td>
        <td>사실상 메모리 한도까지 가능</td>
        <td>크게 잡으면 스택 오버플로우 위험</td>
      </tr>
      <tr>
        <td>사용 난이도</td>
        <td>가장 일반적인 방식, 안전</td>
        <td>잘못 쓰면 위험 → 주의 필요</td>
      </tr>
    </tbody>
  </table>

  <hr />

  <h2>6. 사용할 때 주의할 점</h2>

  <h3>6-1. 버퍼 크기를 과하게 크게 잡지 말 것</h3>

  <ul>
    <li>수백 ~ 몇 KB 정도는 보통 괜찮지만,</li>
    <li>수 MB급 버퍼를 <code>stackalloc</code>으로 만들면 스택 오버플로우 가능성이 크다.</li>
  </ul>

  <h3>6-2. 수명(scope) 벗어난 참조 금지</h3>

  <ul>
    <li><code>stackalloc</code> 버퍼의 주소/참조를
      <ul>
        <li>필드에 저장하거나</li>
        <li>리턴 값으로 내보내거나</li>
        <li>비동기/스레드로 넘기는 것</li>
      </ul>
      전부 위험하다.</li>
    <li>현재 메서드/블록 안에서만 쓰고 끝내야 한다.</li>
  </ul>

  <h3>6-3. 진짜 필요한 곳에서만 사용</h3>

  <ul>
    <li>일반적인 비즈니스 로직(ERP 화면, CRUD, 간단 계산)에서는 그냥 <code>new</code> 배열을 쓰는 것이 더 낫다.</li>
    <li><code>stackalloc</code>은 <strong>“성능/저수준 작업용 특수 도구”</strong> 정도로 생각하는 게 좋다.</li>
  </ul>

  <hr />

  <h2>7. 언제 써볼 만한지 예시</h2>

  <ul>
    <li>초당 수십만 번 호출되는 문자열 파싱 함수에서 임시 버퍼가 필요할 때</li>
    <li>바이너리 데이터를 빠르게 처리하는 인코더/디코더 구현 시</li>
    <li>P/Invoke로 C 함수에 임시 버퍼 포인터를 넘겨야 할 때</li>
    <li><code>Span&lt;byte&gt;</code>로 문자열 조작/파싱을 최적화할 때</li>
  </ul>

  <p>
    그 외 대부분의 “일반적인” 코드에서는 <code>stackalloc</code> 없이도 충분하다.
  </p>

  <hr />

  <h2>8. 한 줄 요약</h2>

  <blockquote>
    <strong><code>stackalloc</code> = “잠깐 쓸 작은 배열을 힙이 아니라 <u>스택에 직접 만들어서</u>  
    엄청 빠르게 쓰고, 메서드 끝나면 흔적도 없이 같이 날려버리는 키워드”</strong>
  </blockquote>


</article>