## Java 8 이전의 날짜, 시간 API의 문제

- Date 클래스는 직관적이지 못하며 자체적으로 시간대 정보를 알지 못함.
- 또한 여러 메서드를 deprecated 시키고 등장한 Calendar 클래스 또한 쉽게 에러를 일으키는 설계 문제를 갖고 있다
- Date와 Calendar 두 가지 클래스가 등장하면서 개발자들에게 혼란만 가중됨.
- 날짜와 시간을 파싱하는데 등장한 DateFormat은 Date에만 지원되었으며, thread-safe하지 못한다.
- Date와 Calendar는 모두 가변 클래스로 유지보수가 어렵다.
## **그래서 등장한 API들**

- java.time 패키지는 LocalDate, LocalTime, LocalDateTime, Instant, Duration, Period 등 새로운 날짜와 시간에 관련된 클래스를 제공한다.
### **LocalDate와 LocalTime**

`LocalDate` 인스턴스는 시간을 제외한 날짜를 표현하는 불변 객체다.

- LocalDate 객체는 어떤 시간대 정보도 포함하지 않는다.
- 정적 팩토리 메서드 of으로 LocalDate 인스턴스를 만들 수 있다.

```java
LocalDate date = LocalDate.of(2020, 12, 22); // of
int year = date.getYear();
Month month = date.getMonth();
int day = date.getDayOfMonth();
LocalDate now = LocalDate.now(); // 시스템 시계의 정보를 이용해서 현재 날짜 정보
```

LocalDate가 제공하는 get 메서드에 TemporalField를 전달해서 정보를 얻는 방법도 있다.

- TemporalField는 시간 관련 객체에서 어떤 필드의 값에 접근할지 정의하는 인터페이스이다.
- `ChronoField`는 TemporalField의 구현체이며 ChronoField의 열거자 요소를 이용해서 원하는 정보를 쉽게 얻을 수 있다.

```java
// public int get(TemporalField field)
int year = date.get(ChronoField.YEAR);
```

시간에 대한 정보는 `LocalTime` 클래스로 표현할 수 있다.

- LocalTime도 정적 메서드 of로 인스턴스를 만들 수 있다.

```java
LocalTime time = LocalTime.of(13, 45, 20); // 13:45:20
int hour = time.getHour();
int minute = time.getMinute();
int second = time.getSecond();
```

- parse 메서드를 통해 날짜와 시간 문자열로 LocalDate와 LocalTime의 인스턴스를 만들 수 있다.

```java
LocalDate date = LocalDate.parse("2020-12-22");
LocalTime time = LocalTime.parse("13:45:20");`
```

#### **날짜와 시간 조합**

`LocalDateTime`은 LocalDate와 LocalTime을 쌍으로 갖는 복합 클래스다.

- 날짜와 시간을 모두 표현할 수 있으며 정적 메서드 of로 인스턴스 생성도 가능하다.
- toLocalDate, toLocalTime 메서드로 LocalDate, LocalTime 인스턴스 추출 가능함.

```java
LocalDateTime dateTime = LocalDateTime.of(2020, Month.DECEMBER, 22, 13, 45, 20);
LocalDateTime dateTime2 = LocalDateTime.of(date, time);

LocalDateTime dateTime2 = date.atTime(13, 45, 20);
LocalDateTime dateTime2 = time.atDate(date);

LocalDate date = dateTime.toLocalDate();
LocalTime time = dateTime2.toLocalTime();
```

### **Instant 클래스 : 기계의 날짜와 시간**

- java.time.Instant 클래스에서는 기계적인 관점에서 시간을 표현한다.
- 유닉스 에포크 시간(Unix epoch time) (1970년 1월 1일 0시 0분 0초 UTC)을 기준으로 특정 지점까지의 `시간을 초`로 표현한다.
- 팩토리 메서드 `ofEpochSecond`에 초를 넘겨주어 인스턴스를 생성할 수 있다.
- Instant 클래스는 `나노초`의 정밀도를 제공하며 오버로드된 `ofEpochSecond` 메서드 버전에서는 두 번째 인수를 이용해서 나노초 단위로 시간을 보정할 수 있다.
- Instant 클래스도 사람이 확인할 수 있도록 시간을 표현해주는 정적 팩토리 메서드 now를 제공한다. 하지만 사람이 읽을 수 있는 시간정보는 제공하지 않는다.

### Duration과 Period

- 지금까지 살펴본 모든 클래스는 `Temporal` 인터페이스를 구현하는데, Temporal 인터페이스는 특정 시간을 모델링하는 객체의 값을 어떻게 읽고 조작할지 정의한다.
- `Duration` 클래스를 사용하면 두 시간 객체 사이의 지속시간을 만들 수 있다.
    - `Duration.between(Temporal startInclusive, Temporal endExclusive` : 정적 팩토리 메서드를 사용하면 두 시간 객체 사이의 지속시간을 만들 수 있다.
    - Duration 클래스는 초와 나노초로 시간 단위를 표현함으로 between 메서드에 LocalDate를 전달할 수 없다.
- 년, 월, 일로 시간을 표현할 때는 `Period` 클래스를 사용하자.
    - Period 클래스의 팩토리 메서드 `between(LocalDate startDateInclusive, LocalDate endDateExclusive` 을 이용하면 두 LocalDate의 차이를 확인할 수 있다.

살펴본 클래스는 불변으로, 함수형 프로그래밍, 스레드 안정성과 도메인 모델의 일관성을 유지하는데 용이하다!
### **날짜 조정, 파싱, 포매팅**

- 날짜나 시간 인스턴스에 시간을 더해야 하는 상황이나 시간 포맷터를 만드는 방법이 필요할 수 있다.
- withAttribute 메서드를 사용하면 일부 속성이 수정된 상태의 `새로운 객체`를 반환받을 수 있다.

```java
LocalDate date1 = LocalDate.of(2017, 9, 21);
LocalDate date2 = date1.withYear(2011); // 2011-09-21
LocalDate date2 = date2.withDayOfMonth(25); // 2011-09-25
LocalDate date2 = date3.with(ChronoField.MONTH_OF_YEAR, 2); // 2011-02-25

LocalDate date1 = LocalDate.of(2017, 9, 21);
LocalDate date2 = date1.plusWeeks(1); // 2017-09-28
LocalDate date2 = date2.minusYear(6); // 2011-09-28
LocalDate date2 = date3.plus(6, ChronoUnit.MONTHS); // 2012-03-28
```

### **TemporalAdjusters**

- `복잡한 날짜 조정기능`이 필요할 때 `with` 메서드에 `TemporalAdjuster`를 전달하는 방법으로 문제를 해결할 수 있다.
- 날짜와 시간 API는 다양한 상황에서 사용할 수 있도록 다양한 TemporalAdjuster 팩토리 메서드를 제공한다.
- 필요한 기능이 존재하지 않으면 커스텀 TemporalAdjuster 를 구현하여 사용할 수 있다.

```java
LocalDate date1 = LocalDate.of(2021, 9, 6); // (월)
LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY)); // 2021-09-12
```

## **날짜와 시간 객체 출력과 파싱**

- 날짜와 시간 관련 작업에서 포매팅과 파싱은 필수적이며, java.time.format 패키지가 지원한다.
- 정적 팩토리 메서드와 상수를 이용해서 손쉽게 포매터를 만들 수 있다.

```java
LocalDate date = LocalDate.of(2014, 3, 18);
String s1 = date.format(DateTimeFormatter.BASIC_ISO_DATE); // 20140318
String s2 = date.format(DateTimeFormatter.ISO_LOCAL_DATE); // 2014-03-18
```

```java
LocalDate date1 = LocalDate.parse("20140318", DateTimeFormatter.BASIC_ISO_DATE);
LocalDate date1 = LocalDate.parse("2014-03-18", DateTimeFormatter.ISO_LOCAL_DATE);
```

- 기존 java.util.DateFormat 클래스와 달리 모든 DateTileFormatter는 스레드에서 안전하게 사용할 수 있는 클래스이며, 특정 패턴으로 포매터를 만들 수 있는 정적 팩토리 메서드도 제공한다.

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPatterm("dd/MM/yyyy");
LocalDate date = LocalDate.of(2014, 3, 18);
String formattedDate = date1.format(formatter);
LocalDate date2 = LocalDate.parse(formattedDate, formatter);
```

- LocalDate의 format 메서드는 요청 형식의 패턴에 해당하는 문자열을 생성한다.
- 그리고 정적 메서드 parse는 같은 포매터를 적용해서 생성된 문자열을 파싱함으로써 다시 날짜를 생성한다.
- 또한, `DateTimeFormatterBuilder` 클래스를 이용하면 원하는 포매터를 직접 만들 수 있다.

## **다양한 시간대와 캘린더 활용 방법**

- 새로운 날짜와 시간 API의 큰 편리함 중 하나는 시간대(timezone)를 간단하게 처리할 수 있다는 점이다.
- 기존의 java.util.TimeZone을 대체할 수 있는 `java.time.ZoneId` 클래스가 새롭게 등장했다.
- ZoneId를 이용하면 서머타임 같은 복잡한 사항이 자동으로 처리된다.
- 또한 ZoneId는 `불변` 클래스다.

### **시간대 사용하기**

- 표준이 같은 지역을 묶어서 시간대(time zone) 규칙 집합을 정의한다.
- ZoneRules 클래스에는 약 40개 정도의 시간대가 있다.
- ZoneId의 getRules()를 이용해서 해당 시간대의 규정을 획득할 수 있다.

```java
ZoneId romeZone = ZoneId.of("Europe/Rome");
```

- 지역 ID는 '{지역}/{도시}' 형식으로 이루어 진다.
- 지역집합 정보는 IANA Time Zone Database 에서 제공하는 정보를 사용한다.
- getDefault() 메서드를 이용하면 기존의 TimeZone 객체를 ZoneId 객체로 변환할 수 있다.

```java
ZoneId zoneId = TimeZone.getDefault().toZoneId();
```

- ZoneId는 LocalDate, LocalTime, LocalDateTime과 같이 ZonedDateTime 인스턴스로 변환할 수 있다.
- ZonedDateTime은 지정한 시간대에 상대적인 시점을 표현한다. (ZonedDateTime = LocalDateTime + 타임존/시차)

```java
LocalDate date = LocalDate.of(2014, 13, 18);
ZonedDateTime zdt = date.atStartOfDay(romeZone);`
```

### ZoneId vs ZoneOffset
- ZoneId은 타임존, ZoneOffset은 시차를 나타낸다.
- ZoneOffset는 UTC 기준으로 고정된 시간 차이를 양수나 음수로 나타내는 반면, ZoneId는 이 시간 차이를 타임존 코드로 나타낸다.

```java
ZoneOffset seoulZoneOffset = ZoneOffset.of("+09:00");
System.out.println("+0900 Time = " + ZonedDateTime.now(seoulZoneOffset));
ZoneId seoulZoneId = ZoneId.of("Asia/Seoul");
System.out.println("Seoul Time = " + ZonedDateTime.now(seoulZoneId));
```
