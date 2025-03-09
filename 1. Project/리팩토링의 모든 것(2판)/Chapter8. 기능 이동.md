
### 8.1 함수 옮기기 
예시1
```text
trackSummary(..)
- calculateDistance()
- distance(..)
- radians(..)
- calculateTime()

# 리팩터링1.
trackSummary(..)
- calculateDistance()
- distance(..)
- radians(..)
- calculateTime()

top_calculateDistance(..) 최상위로 분리


# 리팩터링2. distacne와 radians 옮기기
trackSummary(..)
- calculateDistance()
	- distance(..)
	- radians(..)
- calculateTime()

top_calculateDistance(..) 최상위로 분리
	- distance(..)
	- radians(..)

# 리팩터링3. 
- calculateDistance()에서 top_calculateDistance() 호출
- 이상 없으면 calculateDistance() 제거
- top_calculateDistance() 직접 호출하도록 변경
- top_calculateDistance() > totalDistance() renaming
- 
```

예시1의 경우 자바스크립트의 함수 중첩 사용하나 자바는 지원하지 않는다.
그래서 **메서드 분리**로 선언하여 구현하였다.

포인트는 함수 중첩을 사용해 함수 옮기기를 하는데 있어 의존하는게 없다면 최상위로 뽑는것으로 보인다

> 중첩 함수를 사용하다보면 숨겨진 데이터끼리 상호 의존하기가아주 쉬우니 중첩함수는 되도록 만들지 말자. p285



```java

public class GpsTrackSummary {  
    private final List<Waypoint> points;  
  
    public GpsTrackSummary(List<Waypoint> points) {  
        this.points = new ArrayList<>(points);  
    }  
  
    public TrackSummary trackSummary() {  
        double totalTime = calculateTime();  
        double pace = totalTime / 60 / totalDistance();  
        return new TrackSummary(totalTime, totalDistance(), pace);  
    }  
  
    private double totalDistance() {  
        double result = 0;  
        for(int i = 1; i < points.size(); i++) {  
            result += points.get(i - 1).distance(points.get(i));  
        }  
        return result;  
    }  
  
    private double calculateTime() {  
        if (points.isEmpty()) {  
            return 0;  
        }  
  
        Waypoint last = points.get(points.size() - 1);  
        Waypoint first = points.get(0);  
        return last.diffTimestamp(first);  
    }  
}


public class Waypoint {  
    private final double latitude;  
    private final double longitude;  
    private final double timestamp;  
  
    public Waypoint(double latitude, double longitude, double timestamp) {  
        this.latitude = latitude;  
        this.longitude = longitude;  
        this.timestamp = timestamp;  
    }  
  
    public double distance(Waypoint other) {  
        int earthRadius = 6371;  
        double dLat = radians(other.latitude - this.latitude);  
        double dLon = radians(other.longitude - this.longitude);  
  
        double a = Math.pow(Math.sin(dLat / 2), 2) + Math.cos(radians(other.latitude))  
                * Math.cos(radians(this.latitude)) * Math.pow(Math.sin(dLon / 2), 2);  
        double c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));  
  
        return earthRadius * c;  
    }  
  
    private double radians(double degrees) {  
        return degrees * Math.PI / 180;  
    }  
  
    public double diffTimestamp(Waypoint other) {  
        return this.timestamp - other.timestamp;  
    }

    // getter 생략
}

```

도움은 되지 않는 참고 주소
https://www.movable-type.co.uk/scripts/latlong.html


예2

```java
public class Account {  
    private final int dayOverdrawn;  
    private final AccountType type;  
  
    public Account(int dayOverdrawn, AccountType type) {  
        this.dayOverdrawn = dayOverdrawn;  
        this.type = type;  
    }  
  
    // 은행 이자 계산  
    public double bankCharge() {  
        double result = 4.5;  
        if(dayOverdrawn > 0) {  
            result += overdraftCharge();  
        }  
        return result;  
    }  
  
    // 초과 인출 이자 계산  
    private double overdraftCharge() {  
        if(type.isPremium()){  
            int baseCharge = 10;  
            if(dayOverdrawn <= 7) {  
                return baseCharge;  
            }  
  
            return baseCharge + (dayOverdrawn - 7) * 0.85;  
        }  
  
        return dayOverdrawn * 1.75;  
    }  
}


public class AccountType {  
    private final boolean premium;  
  
    public AccountType(boolean premium) {  
        this.premium = premium;  
    }  
  
    public boolean isPremium() {  
        return premium;  
    }
}
```
- Account의 overdraftCharge() 함수를 AccountType으로 옮긴다 
	- 이떄 dayOverdrawn은 Account에서 주입하도록한다

```java

public class AccountType {  
  private final boolean premium;  
  
  public AccountType(boolean premium) {  
    this.premium = premium;  
  }  
  
  private boolean isPremium() {  
    return premium;  
  }  
  
  // 초과 인출 이자 계산  
  public double overdraftCharge(int dayOverdrawn) {  
    if (isPremium()) {  
      int baseCharge = 10;  
      if (dayOverdrawn <= 7) {  
        return baseCharge;  
      }  
  
      return baseCharge + (dayOverdrawn - 7) * 0.85;  
    }  
  
    return dayOverdrawn * 1.75;  
  }  
}

```