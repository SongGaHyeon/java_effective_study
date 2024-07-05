# Item 10.equals는 일반 규약을 지켜 재정의하라


### "equals를 재정의하지 않는 것이 최선이다."
- 아래에 있는 경우에는 equals 를 재정의 할 필요가 없다


1. 각 인스턴스가 본질적으로 고유하다
    - 싱글톤(패턴)을 만들었다고 가정할때, object는 그 자체로 고유하다. (클래스 인스턴스가 하나)
    - enum을 사용하였을 때 , (enum은 근본적으로 하나만 존재) . 굳이 enum에 equals 사용 불필요 


2. 인스턴스의 '논리적 동치성'을 검사할 필요가 없다.
    - 두개의 화폐(1000원 지폐)가 있다고 할때, 두개가 실제로 같지 않다. (같은 거 두개가 본질적으로 같은 것이 아님). 
    - 값이 같냐 ? -> 같다, 하지만 엄연히 다른 돈 
    - "object가 제공하는 기본적인 equals는 객체의 동일성을 비교한다" - 논리적인 동치성을 검사해야하는지 유무를 살피기
    - (ex) 대표적 예시. 문자열의 "Hello"와 "Hello"가 같은지 비교할 필요 X (드물다)
    

3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 적절하다
    - 부모 클래스에 이미 equals가 정의되어 있는 경우 (재정의할 필요가 없다)
    - (ex) List와 Set은 상위 클래스인 abstractList, abstractMap에 equals가 이미 구현되어 있다. 

4. 클래스가 private 이거나 package-private이고 equals 메서드를 호출할 일이 없다.
    

### "그럼 equals를 정의해야하는 경우?"
- equals를 정의할 땐, 일반 규약을 지켜 재정의해야한다.


1. 반사성 : a랑 a 오브젝트 있으면 (거울을 봤을때) 같다고 해야함


~~~
A.equals(A) == true
~~~


2. 대칭성 : a equals b는, b equals a 와 같아야한다. (결과가 같아야한다)


~~~java
import java.util.ArrayList;
import java.util.List;
import java.util.Objects;

public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString) {
            return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
        }
        if (o instanceof String) {
            return s.equalsIgnoreCase((String) o);
        }
        //권장되지 않는 코드
        return false;
    }

    public static void main(String[] args) {
        CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
        String polish = "polish"; //CaseInsensitiveString은 String을 이용할 수 있겠지만, String은 CaseInsensitiveString을 모른다
        System.out.println(cis.equals(polish)); //true

        List<CaseInsensitiveString> list = new ArrayList<>();
        list.add(cis);
        System.out.println(list.contains(polish)); //들어있지 않다고 나온다 (대칭성 위반이라서)
    }
}
~~~


    - 여기서 cis.equals(polish) 는 true, polish.equals(cis)는 false가 된다.
    - 서로 다른 두 객체를 비교할 때, 한쪽에서는 "같다"고 하지만, 반대쪽에서는 "다르다"고 하기 때문에 문제가 됨
    - 이 문제를 피하기 위해서는 CaseInsensitiveString 클래스가 오직 CaseInsensitiveString 객체와만 비교를 수행하도록 equals 메서드를 수정


~~~
public static void main(String[] args) {
        CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
        //아래처럼 같은 CaseInsensitiveString의 객체를 만든다
        CaseInsensitiveString cis2 = new CaseInsensitiveString("polish");
        String polish = "polish"; 
        System.out.println(cis.equals(polish)); 

        //다음과 같이 확인 (대칭성 확인)
        System.out.println(cis.equals(cis2));
        System.out.println(cis2.equals(cis));

        List<CaseInsensitiveString> list = new ArrayList<>();
        list.add(cis);
        System.out.println(list.contains(polish)); //들어있지 않다고 나온다 (대칭성 위반이라서)
    }
~~~


3. 추이성 : A와 B가 같고, B와 C가 같으면 A와 C가 같다


~~~
import java.util.Objects;

public final class TransitivityViolation {
    private final String s;

    public TransitivityViolation(String s) {
        this.s = Objects.requireNonNull(s);
    }

    @Override
    public boolean equals(Object o) {
        if (o instanceof TransitivityViolation) {
            TransitivityViolation other = (TransitivityViolation) o;
            // 대소문자 구분 없이 비교
            return s.equalsIgnoreCase(other.s);
        }
        if (o instanceof String) {
            // 대소문자 구분하지 않고, 길이가 같다면 true
            return s.equalsIgnoreCase((String) o) && s.length() == ((String) o).length();
        }
        return false;
    }

    public static void main(String[] args) {
        TransitivityViolation a = new TransitivityViolation("hello");
        String b = "HELLO";
        TransitivityViolation c = new TransitivityViolation("HELLO");

        System.out.println(a.equals(b)); // true
        // TransitivityViolation의 equals 메서드는 b가 String 타입임을 인식하고 대소문자 구분 없이 비교, 길이도 같아서 true 반환

        System.out.println(b.equals(c)); // false
        //String의 equals 메서드가 호출

        System.out.println(a.equals(c)); // true
        //길이가 같아서 true
    }
}
~~~


- equals 메서드에서 비교 로직을 일관되게 유지해야 함 (객체 타입에 관계없이 동일한 방식으로 문자열 비교)


~~~java
import java.util.Objects;

public final class ConsistentEquals {
    private final String s;

    public ConsistentEquals(String s) {
        this.s = Objects.requireNonNull(s);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true; // 동일 객체 참조
        if (o == null || getClass() != o.getClass()) return false; // 동일 클래스 확인
        ConsistentEquals that = (ConsistentEquals) o;
        return s.equalsIgnoreCase(that.s); // 대소문자 구분 없이 문자열 비교
    }

    @Override
    public int hashCode() {
        return Objects.hash(s.toLowerCase()); // equals와 일관되게 하기 위해 toLowerCase 사용
    }

    public static void main(String[] args) {
        ConsistentEquals a = new ConsistentEquals("hello");
        ConsistentEquals b = new ConsistentEquals("HELLO");
        ConsistentEquals c = new ConsistentEquals("hello");

        System.out.println(a.equals(b)); // true
        System.out.println(b.equals(c)); // true
        System.out.println(a.equals(c)); // true
    }
}
~~~


- 안전한 코드


~~~java
import java.awt.Color;
import java.util.Objects;

public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    public Point asPoint() {
        return point;
    }

    @Override
    public boolean equals(Object o) {
        //자기자신의 타입인지 확인하기
        if (!(o instanceof ColorPoint)) {
            return false;
        }
        //자신이 가진 field를 확인
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }

    @Override
    public int hashCode() {
        return 31 * point.hashCode() + color.hashCode();
    }
}

class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }

    @Override
    public int hashCode() {
        return 31 * x + y;
    }
}
~~~


- 테스트 코드


~~~
import java.awt.Color;
import java.util.HashSet;
import java.util.Set;

public class CounterPointTest {
    // 단위 원 안의 모든 점을 포함하도록 unitCircle을 초기화한다.
    private static final Set<Point> unitCircle = Set.of(
        new Point(1, 0), 
        new Point(-1, 0),
        new Point(0, 1), 
        new Point(0, -1)
    );

    public static boolean onUnitCircle(Point p) {
        return unitCircle.contains(p);
    }

    public static void main(String[] args) {
        Point p1 = new Point(1, 0);
        Point p2 = new 위에서쓴메서드의패키지명.Colorpoint(1, 0, Color.RED).asPoint();

        // true를 출력한다.
        System.out.println(onUnitCircle(p1));
        // true를 출력해야 하지만, Point의 equals가 getClass를 사용해 작성되었다면 그렇지 않다.
        System.out.println(onUnitCircle(p2));
    }
}
~~~


4. 일관성 : a equals b 첫번째 시도시의 결과가 두번째 시도한 a equals b와 같아야한다.


이를 위해서는 코드를 복잡하게 짜면 안된다. 예를 들어,
URL google1 = new URL(protocol:"https", host:"about.google", file:"/products/");
URL google2 = new URL(protocol:"https", host:"about.google", file:"/products/");
다음과 같이 짤 때 문제가 발생한다.


Virtual host인 경우 (일반적 도메인의 경우에도...), 도메인이 가리킨 실제 IP값이 바뀔 여지가 있음.
이렇게 쓰는 것이 그래서 별로 (일관성 위반 가능성이 있다)



### equals 구현 방법


1. == 연산자를 활용해서 자기 자신의 참조인지 확인
2. instanceof 연산자로 올바른 타입인지 확인
3. 입력된 값을 올바른 타입으로 형변환
4. 입력 객체와 자기 자신의 대응되는 핵심 필드가 일치하는지 확인


* 구글의 AutoValue, Lombok 사용
* IDE의 코드 생성 기능을 사용


~~~
//1번(반사성에 해당하기도 함)
if (this == o) {
    return true;
}

//2번
if (!(o instanceof Point))
    return false;

//3번 형변환 (type casting)
Point p = (Point)o;

//4번. 핵심필드만 비교 (부동소수점 영향 안받는 타입은 ==로 비교)
return p.x == x && p.y == y;
~~~


* 주의점 : 4번 할 때, 부동소수점 영향을 받는 Float이나 Double은 .compare()이용
* 주의점 : equals 재정의 시에 **반드시!!!!** hashCode도 재정의해야한다 (item11)





* * *

# item11.equals를 재정의하려거든 hashCode도 재정의하라

- hashCode란,
해시코드는 객체를 숫자로 표현한 것입니다. 이 숫자는 객체가 고유하게 가지고 있는 특징을 반영합니다. 해시코드는 해시 테이블 같은 자료구조에서 객체를 빠르게 찾거나 저장하는 데 사용됩니다.


### hashCode 규약 (해시 테이블에서 객체를 빠르고 효율적으로 찾기 위해)


1. equals 비교에 사용하는 정보가 변경되지 않았다면 hashCode는 매번 같은 값을 리턴해야함


2. ***두 객체에 대한 equals가 같다면, hashCode의 값도 같아야한다*** 


3. 두 객체에 대한 equals가 다르더라도, hashCode의 값은 같을 수 있지만 해시 테이블 ***성능을 고려해 다른 값을 리턴하는 것이 좋다.*** (같아도 되지만,,, 성능)


~~~java
PhoneNumber number1 = new PhoneNumber(123, 456, 7890)
PhoneNumber number2 = new PhoneNumber(123, 456, 7890)

//같은 인스턴스인데 다른 hashCode
System.out.println(number1.equals(number2));
System.out.println(number1.hashCode());
System.out.println(number2.hashCode())


map.put(number1, "song");
map.put(number2, "kim");

String s = map.get(new PhoneNumber(123, 456, 7890)); //각 객체를 넣으면 null값이 나오게 됨
//이 새로운 것에 대한 hashCode가 없어서. Bucket에 없어서 이런 문제 발생

//String s = map.get(number2); 이건 제대로 동작
System.out.println(s);

~~~
동작 방식 : hashMap에 넣을때, hashCode() 메서드를 실행해서 어느 버킷에 넣을지 정함
꺼낼 때도, key에 대한 hashCode 먼저 가져오고, 버킷에 들어있는 오브젝트 꺼내온다


~~~java
PhoneNumber number1 = new PhoneNumber(123, 456, 7890)
PhoneNumber number2 = new PhoneNumber(456, 789, 1111)

//같은 인스턴스인데 다른 hashCode
System.out.println(number1.equals(number2));
System.out.println(number1.hashCode());
System.out.println(number2.hashCode())
//전혀 다른 오브젝트인데 같은 hash값이 나오는 문제 발생!
//hash Colision
//linked 리스트와 같이 쭉 넣고 쓰이게 됨 (hash의 장점 사용 못함) - O(n)


map.put(number1, "song");
map.put(number2, "kim");

String s = map.get(new PhoneNumber(123, 456, 7890)); //각 객체를 넣으면 null값이 나오게 됨
//이 새로운 것에 대한 hashCode가 없어서. Bucket에 없어서 이런 문제 발생

//String s = map.get(number2); 이건 제대로 동작
System.out.println(s);
~~~


### 전형적으로 하는 hashCode 구현방법

~~~java
@Override public int hashCode() {
    int result = Short.hashCode(areaCode);  //1
    result = 31 * result + Short.hashCode(prefix); //2
    //이전의 값에 31 곱하고 해쉬코드 더해주기

    result = 31 * result + Short.hashCode(lineNum) //3
    return result;
}
~~~


~~~java
@Override
public int hashCode() {
    return Objects.hash(areaCode, prefix, lineNum);
}
~~~
같은 코드다.


* 왜 31 이냐?
    - 홀수로 해야 뒤에 0이 채워지는 현상이 발생안하고, 숫자가 밀리는 현상이 안발생한다
    - hashing 할 때, hash collision(충돌) 적게 나는 연구결과가 31 


1. 핵심 필드 하나의 값의 해쉬값을 계산해서(구해서) result 값을 초기화
Double 이면 Double.hashCode
Integer이면 Integer.hashCode


2. 기본 타입을 Type.hashCode , 참조 타입을 해당 필드의 hashCode, 배열은 모든 원소를 재귀적으로 위의 로직을 적용



이 메서드는 객체의 여러 필드를 조합하여 고유한 숫자(해시코드)를 만드는 과정임.
이 과정을 통해 같은 데이터를 가진 객체는 같은 해시코드를, 다른 데이터를 가진 객체는 다른 해시코드를 갖게 된다!







