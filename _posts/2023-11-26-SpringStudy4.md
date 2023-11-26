---
layout: single
title: "4주차 스프링부트 스터디 미션"


---

<br>

# 피드백 후 과제 수정하기

<br><br>

###### 저번주에 만든 게시판 엔티티를 C(reate), R(ead), U(pdate), D(elete)로 구현하는 과제를 수행했고 코어님에게 피드백 받은것을 수정하는 시간을 가졌다.

<br>

<br>

**내가 받은 피드백**

1.  날짜 자동으로 받아오기
2.  soft delete를 jpa 성격에 맞게 구현하기
3. 계속 보면서 짜잘한거 수정하기

<br>

<br>

<br>

#### 1. 날짜 자동으로 받아오기

---



```java
   @CreatedDate
   @Column(name = "post_date", nullable = false, updatable = false)
   private LocalDateTime postDate;

   @LastModifiedDate
   @Column(name = "update_date", nullable = false)
   private LocalDateTime updateDate;
```

<br>

위에 코드는 3주차 Board 엔티티 코드의 일부분이다. 이렇게 해도 오류는 안나지만 포스트맨에서 내가 직접 데이터를 입력 해줘야한다는 문제점이 있다.

<br>

그래서 내가 수정한 점은 다음과 같다

- 시간을 받아오는 기본 엔티티 생성
- @EntityListeners 이용하기 

<br>

**BaseTimeEntity.java**

```java
package com.example.springbasic.Board;

import lombok.Getter;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;
import jakarta.persistence.EntityListeners;
import jakarta.persistence.MappedSuperclass;
import java.time.LocalDateTime;

@Getter
@EntityListeners(value = {AuditingEntityListener.class})
@MappedSuperclass
public abstract class BaseTimeEntity {

    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}


```

<br>

따로 시간을 받는 엔티티를 만들어서 Board.java가 상속받게 했다.  
그리고 @EntityListeners(value = {AuditingEntityListener.class}) 를 이용하여 생성일과 수정일을 자동으로 관리하게 된다

<br>

여기서 JPA Auditing은 Spring Data JPA에서 시간에 대해서 자동으로 값을 넣어주는 기능입니다. 도메인을 영속성 컨텍스트에 저장하거나 조회를 수행한 후에 update를 하는 경우 매번 시간 데이터를 입력하여 주어야 하는데, audit을 이용하면 자동으로 시간을 매핑하여 데이터베이스의 테이블에 넣어주게 됩니다.

![SmartSelectImage_2023-11-26-17-11-02](C:\Users\tmdrl\OneDrive\사진\Screenshots\SmartSelectImage_2023-11-26-17-11-02.png)

출처: https://webcoding-start.tistory.com/53

<br>

<br>

위에서 말한대로 보드 엔티티에 베이스를 상속하고 포스트맨에서 데이터를 조회해보니까 creatAt, updateAt 모두 null 값이 나와서 당황했다. 이 문제를 찾아보니까 의외로 간단하게 해결됐다.

<br>

```java
package com.example.springbasic;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;


@SpringBootApplication
@EnableJpaAuditing // 이거 넣어줘야 @EntityListeners가 돌아감
public class SpringbasicApplication {

	public static void main(String[] args) {

		SpringApplication.run(SpringbasicApplication.class, args);
	}

}
```

<br>

메인 프로그래에 @EnableJpaAuditing을 붙여줘야 오류가 생기자 않는다.   
@EnableJpaAuditing을 붙여야 Auditing이 가능해져서 자동으로 값이 들어온다고 한다. 그리고 의존성 추가와 responseDto도 수정해줘야한다.

<br>

```java
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
```

<br>

<br>

**BoardResponseDto.java**

```java
package com.example.springbasic.Board;

import lombok.Getter;

import java.time.LocalDateTime;

@Getter
public class BoardResponseDto {

    private Long postNumber;
    private String title;
    private int views;
    private String category;
    private Long memberId;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;

    public BoardResponseDto(Board board) {
        this.createdAt = board.getCreatedAt();
        this.updatedAt = board.getUpdatedAt();
        this.postNumber = board.getPostNumber();
        this.title = board.getTitle();
        this.views = board.getViews();
        this.category = board.getCategory();
        this.memberId = board.getMemberId();
    }

}
```

<br>

<br>

수정을 다 하고 포스트맨으로 데이터 입력 후 조회하면 아래 사진과 같이 나온다.

![SmartSelectImage_2023-11-26-16-39-02](C:\Users\tmdrl\OneDrive\사진\Screenshots\SmartSelectImage_2023-11-26-16-39-02.png)



첫번째 문제 해결!!!!

<br>

<br><br>

<br>

#### soft delete 수정하기

---

기존 코드를 보면 보드 엔티티에 isDeleted가 boolean 타입으로 초기에 false로 초기화 했었다. 그래서 delete 작업을 수행하면 true로 바뀌게 된다.  
그래서 데이터 조회를 하면 isDeleted에 true로만 출력되고 제대로 sofr delete를 구현하지 못했다.

![SmartSelectImage_2023-11-20-03-20-44](C:\Users\tmdrl\OneDrive\사진\Screenshots\SmartSelectImage_2023-11-20-03-20-44.png)

<br><br>

내가 코드에서 수정한 점은 다음과 같다.

-  @Where(clause = "is_deleted is null") 추가하기
-  boolean 대신에 Boolean 사용하기

<br><br>

일단 boolean은 true, false 두 가지 값만 가질수 있지만 Boolean은 null도 허용되서 총 3가지 값을 가질 수 있다.   
그래서 isDeleted를 Boolean값으로 해주고 @Where(clause = "is_deleted is null") 추가해줬다.

<br>

@Where 어노테이션은 보통 삭제되지 않은 엔터티만을 조회할 때 사용된다. 그래서 clause = "is_deleted is null"을 통해 

is_deleted가 null일때만 조회가 된다. 그러면 우리는 soft delete구현하기 위해서는 삭제작업 중 is_deleted를 null이 아니게 하면된다.

<br>

<br>

다음은 수정된 코드이다.  
**Board.java**

```java
package com.example.springbasic.Board;


import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import org.hibernate.annotations.Where;


@Getter
@Entity
@Builder
@AllArgsConstructor
@NoArgsConstructor
@Where(clause = "is_deleted is null")
@Table(name = "boards")
public class Board extends BaseTimeEntity{


    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long postNumber;

    @Column(name = "title", nullable = false, length = 255)
    private String title;

    @Column(name = "views", nullable = false)
    private int views;

    @Column(name = "category", length = 50)
    private String category;


    @Column(name = "member_id")
    private Long memberId;

    @Column(name = "is_deleted")
    private Boolean isDeleted;

    public void softDelete() {
        this.isDeleted = true;
    }


    public void update(String title, int views, String category, Long memberId) {
        this.title = title;
        this.views = views;
        this.category = category;
        this.memberId = memberId;
    }

}
```

<br>

**BoardSeviceImpl.java**의 일부

```
    @Override
    @Transactional
    public Long deleteBoard(Long id) {
        Board board = boardRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Board not found with id: " + id));

        // Soft delete 수행
        board.softDelete();

        return board.getPostNumber();
    }
}
```

<br>

여기서 삭제작업을 하면 isDeleted가 true가 되면서 더 이상 @Where에 조회되않는다.

<br>

포스트맨에서 실행해서 확인해보겠다.  

![SmartSelectImage_2023-11-26-16-40-15](C:\Users\tmdrl\OneDrive\사진\Screenshots\SmartSelectImage_2023-11-26-16-40-15.png)

<br>

삭제하기 전에 데이터이다.  이제 postNumber가 1인 데이터를 삭제해보자

<br>

![SmartSelectImage_2023-11-26-16-41-02](C:\Users\tmdrl\OneDrive\사진\Screenshots\SmartSelectImage_2023-11-26-16-41-02.png)

![SmartSelectImage_2023-11-26-16-41-34](C:\Users\tmdrl\OneDrive\사진\Screenshots\SmartSelectImage_2023-11-26-16-41-34.png)

<br>

postNumber가 1인 데이터가 조회되지 않는다. 그럼 실제로 데이터베이스에 데이터가 남아있는지 확인해보자

<br>

![Screenshot_2023-11-26-16-42-54](C:\Users\tmdrl\OneDrive\사진\Screenshots\Screenshot_2023-11-26-16-42-54.png)

<br>

아직 데이터가 남아있는 것을 확인할 수 있다.  
두번째 문제 해결!!

<br>

<br>

<br>

마지막 문제들은 디테일한 부분이다  
먼저 @Autowired 대신에 생성자 주입방식을 사용하는 것이다. 

<br>

생성자 주입 방식을 사용하면 이점이 있다. 

-  **의존성 주입이 명시적:** 생성자를 통한 주입은 클래스의 의존성이 명시적으로 드러난다
-  **불변성 확보:** 생성자 주입을 사용하면 필드를 final로 선언할 수 있어 불변성을 확보할 수 있다.
-  **테스트 용이성:** 생성자 주입은 테스트 시 모의 객체(Mock)를 주입하기가 더 편리하다.

<br>

<br>

나는 서비스구현 코드에서  @Autowired를 사용했는데 이 부분을 수정했다.

```java
//@Autowired 보다는 생성자 주입 방식으로
    public BoardServiceImpl(BoardRepository boardRepository) {
        this.boardRepository = boardRepository;
    }
```

<br>

이외에 내가 수정한 부분은 시간을 자동으로 받아오니까 requestDto에서 날짜 입력받는 부분을 삭제한거와 짜잘한 부분들이다

수정을 하는 시간을 가지면서 내가 짠 코드가 얼마나 빈약한지 알게 되었고 앞으로 탄탄하게 실력을 쌓아가야겠다고 다짐 했다.