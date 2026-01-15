<article>


  <hr />

  <h2>1. LINQ 한 줄 정의</h2>

  <blockquote>
    <strong>LINQ = C# 코드 안에서 컬렉션(배열, List, DB데이터 등)을 SQL처럼 깔끔하게 필터/정렬/변환/집계하는 기능</strong>
  </blockquote>

  <p>원래는 이런 식으로 썼던 코드를,</p>

  <pre><code class="language-csharp">int[] numbers = { 3, 5, 4, 2, 6, 7, 8 };
List&lt;int&gt; evens = new List&lt;int&gt;();

foreach (int n in numbers)
{
    if (n % 2 == 0)
        evens.Add(n);
}

evens.Sort();  // 오름차순 정렬
</code></pre>

  <p>LINQ로는 이렇게 쓸 수 있다.</p>

  <pre><code class="language-csharp">int[] numbers = { 3, 5, 4, 2, 6, 7, 8 };

var evens = numbers
    .Where(n =&gt; n % 2 == 0)  // 짝수만 필터
    .OrderBy(n =&gt; n)         // 오름차순 정렬
    .ToList();               // List&lt;int&gt;로 변환
</code></pre>

  <ul>
    <li><code>Where</code> : 조건 필터링</li>
    <li><code>OrderBy</code> : 정렬</li>
    <li><code>ToList</code> : 리스트로 변환</li>
    <li><code>n =&gt; n % 2 == 0</code> : 조건을 나타내는 람다식 (Func&lt;int,bool&gt;)</li>
  </ul>

  <p>결과는 똑같지만, LINQ 버전이 훨씬 읽기 쉽다.  
    <strong>“무슨 일을 하는지”가 코드에 그대로 드러나는 게 장점</strong>이다.
  </p>

  <hr />

  <h2>2. LINQ는 어디에 쓸 수 있을까?</h2>

  <p>대표적으로 다음과 같은 곳에 쓸 수 있다.</p>

  <ul>
    <li><code>int[]</code>, <code>string[]</code> 같은 배열</li>
    <li><code>List&lt;T&gt;</code>, <code>Dictionary&lt;K,V&gt;</code> 같은 컬렉션</li>
    <li>DB에서 가져온 <code>IQueryable&lt;T&gt;</code> (Entity Framework 같은 ORM)</li>
    <li><code>IEnumerable&lt;T&gt;</code> 를 구현한 대부분의 타입</li>
  </ul>

  <p>즉, “여러 개의 데이터가 모여 있는 것”에 대해서 거의 다 쓸 수 있다고 보면 된다.</p>

  <hr />

  <h2>3. 메서드 문법(Method Syntax) 기본 패턴</h2>

  <p>LINQ를 가장 많이 쓰는 방식은 <strong>메서드 체인 스타일</strong>이다.</p>

  <pre><code class="language-csharp">컬렉션
    .Where(조건)
    .Select(변환)
    .OrderBy(정렬기준)
    .GroupBy(그룹기준)
    .ToList();
</code></pre>

  <h3>3-1. Where – 조건 필터링</h3>

  <pre><code class="language-csharp">int[] numbers = { 3, 5, 4, 2, 6, 7, 8 };

// 짝수만 가져오기
var evens = numbers
    .Where(n =&gt; n % 2 == 0)
    .ToList();
</code></pre>

  <ul>
    <li><code>Where</code>는 <code>Func&lt;T, bool&gt;</code> 타입의 함수를 받는다.</li>
    <li><code>n =&gt; n % 2 == 0</code>은 “n이 짝수면 true”를 반환하는 람다식이다.</li>
    <li>결과적으로 짝수만 필터링된 리스트를 얻게 된다.</li>
  </ul>

  <h3>3-2. Select – 모양 변환</h3>

  <p>데이터를 다른 형태로 바꾸고 싶을 때 사용한다.</p>

  <pre><code class="language-csharp">int[] numbers = { 1, 2, 3, 4, 5 };

var squares = numbers
    .Select(n =&gt; n * n)  // n을 n의 제곱으로 변환
    .ToList();
</code></pre>

  <p>문자열로 바꾸는 것도 가능하다.</p>

  <pre><code class="language-csharp">var texts = numbers
    .Select(n =&gt; $"숫자: {n}")
    .ToArray();
</code></pre>

  <ul>
    <li><code>Select</code>의 인수 타입: <code>Func&lt;TSource, TResult&gt;</code></li>
    <li>입력 타입과 출력 타입이 달라도 된다 (int → string 등)</li>
  </ul>

  <h3>3-3. OrderBy / OrderByDescending – 정렬</h3>

  <pre><code class="language-csharp">int[] numbers = { 3, 5, 4, 2, 6, 7, 8 };

var sortedAsc = numbers
    .OrderBy(n =&gt; n)         // 오름차순
    .ToList();

var sortedDesc = numbers
    .OrderByDescending(n =&gt; n) // 내림차순
    .ToList();
</code></pre>

  <p>문자열도 똑같이 쓸 수 있다.</p>

  <pre><code class="language-csharp">string[] names = { "Kim", "Park", "Lee", "Choi" };

var sortedNames = names
    .OrderBy(n =&gt; n)   // 이름 오름차순
    .ToList();
</code></pre>

  <h3>3-4. GroupBy – 그룹핑 (거래처별, 품목별 등 묶기)</h3>

  <p>실무에서 자주 쓰이는 “거래처별 매출집계”, “품목별 수량합계” 같은 작업에 사용한다.</p>

  <pre><code class="language-csharp">class Order
{
    public string CustomerCode { get; set; }
    public decimal Amount { get; set; }
}

List&lt;Order&gt; orders = ...; // 주문 리스트라고 가정

var sumByCustomer = orders
    .GroupBy(o =&gt; o.CustomerCode)   // 거래처코드별로 그룹핑
    .Select(g =&gt; new
    {
        Customer = g.Key,           // 그룹 키 = CustomerCode
        Total    = g.Sum(o =&gt; o.Amount),
        Count    = g.Count()
    })
    .ToList();
</code></pre>

  <p>정리하면:</p>

  <ul>
    <li><code>GroupBy(o =&gt; o.CustomerCode)</code> : CustomerCode가 같은 주문끼리 하나의 그룹이 된다.</li>
    <li><code>g.Key</code> : 해당 그룹의 거래처 코드</li>
    <li><code>g.Sum(o =&gt; o.Amount)</code> : 그 거래처의 주문금액 총합</li>
    <li><code>g.Count()</code> : 그 거래처의 주문 건수</li>
  </ul>

  <hr />

  <h2>4. 쿼리 문법(Query Syntax) – SQL처럼 쓰는 스타일</h2>

  <p>LINQ는 다음처럼 SQL 비슷한 문법으로도 쓸 수 있다.</p>

  <pre><code class="language-csharp">int[] numbers = { 3, 5, 4, 2, 6, 7, 8 };

var evens =
    from n in numbers
    where n % 2 == 0
    orderby n
    select n;

var list = evens.ToList();
</code></pre>

  <p>위 코드는 내부적으로는 다음과 같은 메서드 체인으로 바뀐다.</p>

  <pre><code class="language-csharp">var evens = numbers
    .Where(n =&gt; n % 2 == 0)
    .OrderBy(n =&gt; n);
</code></pre>

  <p>
    둘 다 똑같이 동작하니, 편한 스타일을 선택해서 쓰면 된다.<br />
    보통은 <strong>메서드 문법(점 찍고 이어 쓰는 방식)</strong>을 더 많이 사용한다.
  </p>

  <hr />

  <h2>5. LINQ와 델리게이트 / 람다 / Func / Predicate 관계</h2>

  <p>
    LINQ 메서드를 이해하려면, “이 메서드가 어떤 타입의 함수를 인수로 받는지”를 보면 된다.
  </p>

  <h3>5-1. Where의 시그니처(개념)</h3>

  <pre><code class="language-csharp">public static IEnumerable&lt;TSource&gt; Where&lt;TSource&gt;(
    this IEnumerable&lt;TSource&gt; source,
    Func&lt;TSource, bool&gt; predicate
)
</code></pre>

  <ul>
    <li><code>predicate</code> : <code>TSource</code> 하나를 받아서 <code>bool</code>을 반환하는 함수</li>
    <li>우리가 쓰는 <code>n =&gt; n % 2 == 0</code> 같은 람다식은 실제로 <code>Func&lt;int,bool&gt;</code> 타입이다.</li>
  </ul>

  <h3>5-2. List&lt;T&gt;.Find와 비교해 보기</h3>

  <pre><code class="language-csharp">public T Find(Predicate&lt;T&gt; match)</code></pre>

  <p>
    <code>Predicate&lt;T&gt;</code>는 사실상 <code>Func&lt;T,bool&gt;</code>와 같은 형태다(입력 T, 출력 bool).<br />
    그래서 다음 둘은 역할이 거의 같다.
  </p>

  <ul>
    <li><code>List&lt;T&gt;.Find(n =&gt; n % 2 == 0)</code> → <code>Predicate&lt;T&gt;</code> 사용</li>
    <li><code>Where(n =&gt; n % 2 == 0)</code> → <code>Func&lt;T,bool&gt;</code> 사용</li>
  </ul>

  <p>
    결국 LINQ는 <strong>“함수를 인수로 받아서 컬렉션을 처리하는 라이브러리”</strong>이고,
    그 함수는 대부분 <strong>람다식</strong>으로 작성한다.
  </p>

  <hr />

  <h2>6. 실무 느낌 예제: 주문 리스트 처리</h2>

  <p>아래는 “주문(Order) 리스트”를 LINQ로 처리하는 실무형 예제다.</p>

  <pre><code class="language-csharp">using System;
using System.Collections.Generic;
using System.Linq;

namespace LambdaExample
{
    class Order
    {
        public int Id { get; set; }              // 주문번호
        public string CustomerCode { get; set; } // 거래처 코드
        public string ItemCode { get; set; }     // 품목 코드
        public DateTime OrderDate { get; set; }  // 주문일자
        public decimal Amount { get; set; }      // 주문금액
        public string Status { get; set; }       // 상태: "완료", "취소" 등
    }

    class Program
    {
        static void Main()
        {
            List&lt;Order&gt; orders = GetSampleOrders();

            // 1) 완료 상태이면서 50만원 이상인 주문만 조회
            var highValueOrders = orders
                .Where(o =&gt; o.Status == "완료" &amp;&amp; o.Amount &gt;= 500_000m)
                .OrderByDescending(o =&gt; o.Amount)
                .ToList();

            // 2) 거래처별 매출 합계 (취소 제외)
            var sumByCustomer = orders
                .Where(o =&gt; o.Status == "완료")
                .GroupBy(o =&gt; o.CustomerCode)
                .Select(g =&gt; new
                {
                    CustomerCode = g.Key,
                    TotalAmount  = g.Sum(o =&gt; o.Amount),
                    OrderCount   = g.Count()
                })
                .OrderByDescending(x =&gt; x.TotalAmount)
                .ToList();

            // 3) 공통 합계 함수 + 조건을 Func&lt;Order,bool&gt;로 전달
            decimal c001Total = GetTotalAmount(
                orders,
                o =&gt; o.CustomerCode == "C001" &amp;&amp; o.Status == "완료"
            );

            decimal c002CancelTotal = GetTotalAmount(
                orders,
                o =&gt; o.CustomerCode == "C002" &amp;&amp; o.Status == "취소"
            );
        }

        static decimal GetTotalAmount(List&lt;Order&gt; orders, Func&lt;Order, bool&gt; predicate)
        {
            return orders
                .Where(predicate)       // 조건을 람다식으로 받아서 필터
                .Sum(o =&gt; o.Amount);    // 금액 합계
        }

        static List&lt;Order&gt; GetSampleOrders()
        {
            return new List&lt;Order&gt;
            {
                new Order { Id = 1, CustomerCode = "C001", ItemCode = "ITEM-01", OrderDate = new DateTime(2025, 1, 10), Amount = 300_000m, Status = "완료" },
                new Order { Id = 2, CustomerCode = "C001", ItemCode = "ITEM-02", OrderDate = new DateTime(2025, 1, 15), Amount = 800_000m, Status = "완료" },
                new Order { Id = 3, CustomerCode = "C002", ItemCode = "ITEM-03", OrderDate = new DateTime(2025, 2, 01), Amount = 150_000m, Status = "취소" },
                new Order { Id = 4, CustomerCode = "C002", ItemCode = "ITEM-01", OrderDate = new DateTime(2025, 2, 20), Amount = 1_200_000m, Status = "완료" },
                new Order { Id = 5, CustomerCode = "C003", ItemCode = "ITEM-02", OrderDate = new DateTime(2025, 3, 05), Amount = 500_000m, Status = "완료" },
                new Order { Id = 6, CustomerCode = "C003", ItemCode = "ITEM-03", OrderDate = new DateTime(2025, 3, 10), Amount = 50_000m,  Status = "완료" },
            };
        }
    }
}
</code></pre>

  <p>여기서 LINQ가 하는 일은 크게 세 가지다.</p>

  <ol>
    <li><strong>Where</strong>로 조건에 맞는 주문만 필터링</li>
    <li><strong>GroupBy + Select</strong>로 거래처별 집계</li>
    <li><strong>Func&lt;Order,bool&gt; + Where</strong>로 공통 합계 함수에 조건을 주입</li>
  </ol>

  <hr />

</article>