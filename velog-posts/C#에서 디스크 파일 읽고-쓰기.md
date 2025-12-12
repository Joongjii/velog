<article>

  <ul>
    <li><code>File.ReadAllText</code> / <code>WriteAllText</code> 같은 간단 방법</li>
    <li><code>FileStream</code>, <code>StreamReader</code>, <code>StreamWriter</code> 같은 스트림 기반 방식</li>
    <li><code>BinaryReader</code>, <code>BinaryWriter</code>를 사용하는 이진 파일</li>
    <li><code>Seek</code>을 이용한 임의 접근(Random Access)</li>
    <li><code>ReadAsync</code>, <code>WriteAsync</code>를 이용한 비동기 파일 I/O</li>
    <li><code>Path</code>, <code>Directory</code> 등 경로/디렉터리 관리</li>
  </ul>

  <hr />

  <h2>1. 가장 쉬운 파일 처리 – <code>File</code> 클래스 (한 번에 읽고/쓰기)</h2>

  <p>
    작은 텍스트 파일을 빠르게 읽고 쓸 때는 <code>System.IO.File</code> 클래스의
    <code>ReadAllXXX</code>, <code>WriteAllXXX</code> 메서드를 쓰는 게 가장 편하다.
  </p>

  <h3>1-1. 텍스트 전체를 한 번에 쓰기 / 읽기</h3>

  <pre><code class="language-csharp">using System;
using System.IO;

class Program
{
    static void Main()
    {
        string path = "test.txt";

        // 쓰기
        File.WriteAllText(path, "첫 줄\n두 번째 줄");

        // 읽기
        string text = File.ReadAllText(path);
        Console.WriteLine(text);
    }
}
</code></pre>

  <ul>
    <li><code>WriteAllText</code> : 문자열 전체를 파일에 한 번에 기록</li>
    <li><code>ReadAllText</code> : 파일 전체를 한 번에 읽어 문자열로 반환</li>
  </ul>

  <h3>1-2. 줄 단위로 한 번에 쓰기 / 읽기</h3>

  <pre><code class="language-csharp">using System;
using System.IO;

class Program
{
    static void Main()
    {
        string[] lines = { "Line1", "Line2", "Line3" };
        File.WriteAllLines("lines.txt", lines);

        string[] readLines = File.ReadAllLines("lines.txt");
        foreach (var line in readLines)
        {
            Console.WriteLine(line);
        }
    }
}
</code></pre>

  <p>
    이런 방식은 <strong>파일 크기가 크지 않을 때</strong> “간단하게 읽고/쓰기” 좋다.
  </p>

  <hr />

  <h2>2. 스트림 기반 파일 처리 – <code>FileStream</code> + <code>StreamReader</code>/<code>StreamWriter</code></h2>

  <p>
    조금 더 본격적으로 파일을 다루려면 <strong>스트림(Stream)</strong> 기반으로 접근한다.
  </p>

  <ul>
    <li><code>FileStream</code> : 파일을 바이트 단위로 읽고/쓰는 기본 스트림</li>
    <li><code>StreamReader</code> : 텍스트를 편하게 읽기</li>
    <li><code>StreamWriter</code> : 텍스트를 편하게 쓰기</li>
  </ul>

  <h3>2-1. 텍스트 파일 읽기 – <code>StreamReader</code></h3>

  <pre><code class="language-csharp">using System;
using System.IO;

class Program
{
    static void Main()
    {
        using (FileStream fs = new FileStream("test.txt", FileMode.Open, FileAccess.Read))
        using (StreamReader reader = new StreamReader(fs))  // 기본 인코딩
        {
            string all = reader.ReadToEnd();   // 파일 전체 읽기
            Console.WriteLine(all);
        } // using 끝나면 자동으로 파일 닫힘
    }
}
</code></pre>

  <h3>2-2. 텍스트 파일 쓰기 – <code>StreamWriter</code></h3>

  <pre><code class="language-csharp">using System;
using System.IO;

class Program
{
    static void Main()
    {
        using (FileStream fs = new FileStream("test.txt", FileMode.Create, FileAccess.Write))
        using (StreamWriter writer = new StreamWriter(fs))
        {
            writer.WriteLine("첫 줄");
            writer.WriteLine("둘째 줄");
        } // Flush + Close 자동
    }
}
</code></pre>

  <h3>2-3. 왜 <code>using</code>을 써야 할까?</h3>

  <p>
    <code>FileStream</code>은 운영체제의 <strong>파일 핸들</strong>을 잡고 있다.
    파일을 열어만 놓고 닫지 않으면:
  </p>

  <ul>
    <li>다른 프로그램이나 코드에서 파일을 사용하지 못할 수 있고</li>
    <li>OS 자원이 낭비된다</li>
  </ul>

  <p>
    그래서 <code>using</code> 블록을 사용해서 <code>Dispose()</code>를 자동으로 호출하는 것이 기본 패턴이다.
  </p>

  <hr />

  <h2>3. 이진 파일 처리 – <code>BinaryReader</code> / <code>BinaryWriter</code></h2>

  <p>
    텍스트가 아니라 <strong>숫자, 바이트, 구조체</strong> 같은 데이터를 파일에 저장하고 싶다면
    <code>BinaryReader</code> / <code>BinaryWriter</code>를 사용한다.
  </p>

  <h3>3-1. 바이너리 쓰기</h3>

  <pre><code class="language-csharp">using System;
using System.IO;

class Program
{
    static void Main()
    {
        using (FileStream fs = new FileStream("data.bin", FileMode.Create, FileAccess.Write))
        using (BinaryWriter bw = new BinaryWriter(fs))
        {
            bw.Write(123);          // int
            bw.Write(3.14f);        // float
            bw.Write(true);         // bool
            bw.Write("Hello");      // string
        }
    }
}
</code></pre>

  <h3>3-2. 바이너리 읽기</h3>

  <pre><code class="language-csharp">using System;
using System.IO;

class Program
{
    static void Main()
    {
        using (FileStream fs = new FileStream("data.bin", FileMode.Open, FileAccess.Read))
        using (BinaryReader br = new BinaryReader(fs))
        {
            int i = br.ReadInt32();
            float f = br.ReadSingle();
            bool b = br.ReadBoolean();
            string s = br.ReadString();

            Console.WriteLine($"{i}, {f}, {b}, {s}");
        }
    }
}
</code></pre>

  <p>
    중요한 포인트는 <strong>읽는 순서가 쓰는 순서와 반드시 같아야 한다</strong>는 것.
    이건 일종의 “파일 포맷 / 프로토콜”을 설계하는 느낌으로 이해하면 된다.
  </p>

  <hr />

  <h2>4. 임의 접근(Random Access) – <code>FileStream.Seek</code></h2>

  <p>
    파일을 처음부터 쭉 읽는 게 아니라, <strong>파일의 중간으로 점프해서</strong> 읽거나 쓰고 싶을 때가 있다.
    이때 <code>FileStream.Seek</code>를 사용한다.
  </p>

  <pre><code class="language-csharp">using System;
using System.IO;

class Program
{
    static void Main()
    {
        using (FileStream fs = new FileStream("data.bin", FileMode.Open, FileAccess.Read))
        using (BinaryReader br = new BinaryReader(fs))
        {
            // 파일의 8바이트 지점으로 이동 (0 기반 오프셋)
            fs.Seek(8, SeekOrigin.Begin);

            int value = br.ReadInt32();  // 8바이트 이후의 int 값
            Console.WriteLine(value);
        }
    }
}
</code></pre>

  <ul>
    <li><code>Seek(offset, origin)</code></li>
    <li><code>origin</code>에는 <code>SeekOrigin.Begin</code>, <code>SeekOrigin.Current</code>, <code>SeekOrigin.End</code> 등이 들어간다.</li>
  </ul>

  <p>
    고정 길이 레코드로 저장된 바이너리 파일을 다루거나,
    대용량 파일에서 특정 위치만 빠르게 읽고 싶을 때 유용한 방식이다.
  </p>

  <hr />

  <h2>5. 비동기 파일 I/O – <code>ReadAsync</code> / <code>WriteAsync</code> / <code>File.*Async</code></h2>

  <p>
    UI가 멈추지 않게 하거나, 서버에서 많은 요청을 처리할 때는
    <strong>비동기 I/O</strong>를 사용하는 게 좋다.
    C#에서는 <code>async/await</code>와 함께 <code>File.*Async</code>, <code>FileStream.ReadAsync</code>, <code>WriteAsync</code> 등을 사용한다.
  </p>

  <h3>5-1. 고수준 비동기 – <code>File.ReadAllTextAsync</code>, <code>WriteAllTextAsync</code></h3>

  <pre><code class="language-csharp">using System;
using System.IO;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        string path = "test.txt";

        await File.WriteAllTextAsync(path, "Hello Async File!");

        string text = await File.ReadAllTextAsync(path);
        Console.WriteLine(text);
    }
}
</code></pre>

  <h3>5-2. 스트림 기반 비동기 – <code>FileStream</code>의 <code>ReadAsync</code> / <code>WriteAsync</code></h3>

  <pre><code class="language-csharp">using System;
using System.IO;
using System.Text;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        byte[] buffer = Encoding.UTF8.GetBytes("Async Stream Example");

        // 쓰기
        using (FileStream fs = new FileStream(
            "async.bin",
            FileMode.Create,
            FileAccess.Write,
            FileShare.None,
            bufferSize: 4096,
            useAsync: true))   // 비동기 사용 설정
        {
            await fs.WriteAsync(buffer, 0, buffer.Length);
        }

        // 읽기
        using (FileStream fs = new FileStream(
            "async.bin",
            FileMode.Open,
            FileAccess.Read,
            FileShare.Read,
            bufferSize: 4096,
            useAsync: true))
        {
            byte[] readBuffer = new byte[buffer.Length];
            int bytesRead = await fs.ReadAsync(readBuffer, 0, readBuffer.Length);

            string text = Encoding.UTF8.GetString(readBuffer, 0, bytesRead);
            Console.WriteLine(text);
        }
    }
}
</code></pre>

  <p>
    비동기 파일 I/O를 사용하면, I/O를 기다리는 동안 스레드를 블로킹하지 않아서
    <strong>UI 응답성</strong>과 <strong>서버 처리량</strong>이 좋아지는 효과가 있다.
  </p>

  <hr />

  <h2>6. 경로/디렉터리 다루기 – <code>Path</code>, <code>Directory</code>, <code>FileInfo</code></h2>

  <p>
    파일 작업을 할 때는 파일만이 아니라 <strong>경로, 폴더</strong> 관리도 함께 필요하다.
  </p>

  <h3>6-1. <code>Path</code> – 문자열 경로 다루기</h3>

  <pre><code class="language-csharp">using System;
using System.IO;

class Program
{
    static void Main()
    {
        string folder = @"C:\Temp";
        string fileName = "test.txt";

        string fullPath = Path.Combine(folder, fileName); // C:\Temp\test.txt

        string dir = Path.GetDirectoryName(fullPath);     // C:\Temp
        string name = Path.GetFileName(fullPath);         // test.txt
        string ext = Path.GetExtension(fullPath);         // .txt

        Console.WriteLine(fullPath);
        Console.WriteLine(dir);
        Console.WriteLine(name);
        Console.WriteLine(ext);
    }
}
</code></pre>

  <h3>6-2. <code>Directory</code> – 폴더 생성 / 파일 목록</h3>

  <pre><code class="language-csharp">using System;
using System.IO;

class Program
{
    static void Main()
    {
        // 폴더 생성
        Directory.CreateDirectory(@"C:\Temp\Logs");

        // 특정 폴더의 .txt 파일 목록
        string[] files = Directory.GetFiles(@"C:\Temp", "*.txt");
        foreach (var f in files)
        {
            Console.WriteLine(f);
        }
    }
}
</code></pre>

  <h3>6-3. <code>FileInfo</code> / <code>DirectoryInfo</code> – 객체 지향 버전</h3>

  <pre><code class="language-csharp">using System;
using System.IO;

class Program
{
    static void Main()
    {
        var info = new FileInfo("test.txt");
        Console.WriteLine(info.FullName);
        Console.WriteLine(info.Length);
        Console.WriteLine(info.LastWriteTime);
    }
}
</code></pre>

  <hr />

  <h2>7. 언제 어떤 방식을 쓰면 좋을까?</h2>

  <table border="1" cellpadding="6" cellspacing="0">
    <thead>
      <tr>
        <th>상황</th>
        <th>추천 API</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>작은 텍스트 파일을 한 번에 읽고/쓰기</td>
        <td><code>File.ReadAllText</code>, <code>File.WriteAllText</code></td>
      </tr>
      <tr>
        <td>줄 단위로 간단하게 읽고/쓰기</td>
        <td><code>File.ReadAllLines</code>, <code>File.WriteAllLines</code></td>
      </tr>
      <tr>
        <td>큰 파일을 조금씩 읽거나, 반복해서 처리</td>
        <td><code>FileStream</code> + <code>StreamReader</code>/<code>StreamWriter</code></td>
      </tr>
      <tr>
        <td>숫자/바이너리 구조 저장/복원</td>
        <td><code>FileStream</code> + <code>BinaryReader</code>/<code>BinaryWriter</code></td>
      </tr>
      <tr>
        <td>파일 중간으로 점프해서 읽기/쓰기</td>
        <td><code>FileStream.Seek</code></td>
      </tr>
      <tr>
        <td>UI 멈추지 않게 비동기로 파일 처리</td>
        <td><code>File.*Async</code>, <code>FileStream.ReadAsync</code>/<code>WriteAsync</code></td>
      </tr>
      <tr>
        <td>경로 조합, 확장자, 폴더 관리</td>
        <td><code>Path</code>, <code>Directory</code>, <code>FileInfo</code>, <code>DirectoryInfo</code></td>
      </tr>
    </tbody>
  </table>

  <hr />

</article>